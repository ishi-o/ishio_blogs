---
title: "PLG: 部署与飞书告警实践"
date: 2026-08-16T00:00:00.000Z
categories: [Tools & Utilities, Observability]
tags: [grafana, alerting, observability]
mathjax: true
---

<!-- placeholder -->
<!-- more -->

# `PLG`: 部署与飞书告警实践

本篇记录如何把 `PLG` 监控体系以 `git` + `helm` + `CI` 的方式部署进集群，并把告警送达飞书群，所有做法都是生产环境的通用玩法

## 监控栈的 `CI` 部署

### 整体形态

- 监控栈是一个 `umbrella chart`：聚合 `kube-prometheus-stack`(含 `Prometheus`/`Alertmanager`)+ `loki` + `promtail` + `grafana` 依赖，环境差异全部放在 `values-*.yaml` 中
- 部署不走手工命令，而是 `GitHub Actions` 在 `push` 到 `main` 时按路径触发，路径过滤保证只有监控配置变更才触发部署：

  ```yaml
  on:
    push:
      branches:
        - main
      paths:
        - charts/monitoring/**
        - charts/istio/**
        - charts/ingress/**
  ```

- 多集群、多产品通过 `matrix` 展开，每个条目指定 `chart`、命名空间、`values` 文件与 `kubeconfig` 上下文：

  ```yaml
  strategy:
    matrix:
      product:
        - name: monitoring
          namespace: monitoring
          values: values-example-prod.yaml
      cluster:
        - name: example-prod
          runner: example-runner # 自托管 runner，持有集群访问权
          context: example-prod
  ```

### 密钥注入

- 密钥永远不进 `git`：`values` 文件里写占位符，`CI` 用 `envsubst` 在发布时替换

  ```yaml
  # values 文件中的占位
  grafana:
    env:
      BOT_GRAFANA_WEBHOOK_TOKEN: $BOT_GRAFANA_WEBHOOK_TOKEN
  ```

  ```bash
  envsubst '$FEISHU_APP_ID $FEISHU_APP_SECRET $BOT_GRAFANA_WEBHOOK_TOKEN' \
    < charts/monitoring/values-example-prod.yaml > $values_file
  ```

- 简单值直接用 `--set` 注入(如 `Cloudflare` `Token`)，不必经过 `values` 文件

### 发布与自愈

- 统一使用 `helm upgrade --install --create-namespace`，并对"上一个操作未完成"的常见冲突做**自动回滚 + 重试**：

  ```bash
  for retry in {1..3}; do
    res=$(helm -n monitoring upgrade --install --create-namespace monitoring \
      ./charts/monitoring -f "$values_file" \
      --set cloudflare.apiToken=$CF_TOKEN 2>&1)
    [[ $? -eq 0 ]] && exit 0
    if [[ "$res" = *"another operation (install/upgrade/rollback) is in progress"* ]]; then
      helm -n monitoring rollback monitoring # 卡住的发布先回滚再重试
      continue
    fi
    sleep 10
  done
  exit 1
  ```

- `runner` 通过 `kubeconfig` `secret` 切换目标集群上下文，一个 `runner` 可以发布多个集群

  ```yaml
  - uses: azure/k8s-set-context@v4
    with:
      method: kubeconfig
      kubeconfig: ${{ secrets.KUBECONFIG }}
      context: ${{ matrix.cluster.context }}
  ```

## 告警即代码

### `Grafana` 统一告警

- 选择 `Grafana` 统一告警(`Unified Alerting`)而非 `Alertmanager` 作为告警出口：规则、联系点、通知策略都住在 `Grafana` 里，与仪表盘共享数据源与标签体系，`Alertmanager` 仅作为 `kube-prometheus-stack` 的兜底保留
- 告警规则不进 `Grafana` 界面手点，而是写成 `ConfigMap`，打上约定标签，由 `Grafana sidecar` 全命名空间监听并同步进 `Grafana`，改动走 `git` 评审：

  ```yaml
  kind: ConfigMap
  metadata:
    name: pod-status-alert
    namespace: monitoring
    labels:
      grafana_alert: "1" # sidecar 按该标签发现告警资源
  data:
    pod-status-alert.yaml: |
      apiVersion: 1
      ...
  ```

  ```yaml
  # values 中开启 sidecar 的告警同步，搜索全命名空间
  grafana:
    sidecar:
      alerts:
        enabled: true
        searchNamespace: ALL
  ```

- 同一文件描述四类资源：`groups`(规则)、`contactPoints`(联系点)、`policies`(通知策略)、`templates`(通知模板)

### 告警规则的结构

- 规则按 `group` 组织，`folder` 决定在 `Grafana` 中的目录归属，`interval` 是评估周期：

  ```yaml
  apiVersion: 1
  groups:
    - orgId: 1
      name: Kubernetes
      folder: Infra
      interval: 1m # 每分钟评估一次
      rules:
        - uid: a9a96a82-f6f5-4bfe-a18e-6f1ac6c9e72f
          title: 服务健康状态
          condition: 不健康服务增多 # 指向起决定作用的表达式 refId
          data:
            - refId: 不健康服务
              datasourceUid: prometheus # 查询 Prometheus
              model:
                expr: |
                  sum by (namespace, pod, phase) (
                    kube_pod_status_phase{namespace !~ 'staging|dev-.*', phase=~"Pending|Unknown|Failed"} == 1
                    and on (namespace, pod) ((time() - kube_pod_created) > 180)
                  )
            - refId: 不健康服务增多
              datasourceUid: __expr__ # 表达式阶段做阈值判断
              model:
                type: threshold
                expression: 不健康服务
                conditions:
                  - evaluator:
                      params: [0]
                      type: gt # 结果 > 0 即触发
          for: 5m # 持续满足 5 分钟才真正触发，抑制抖动
          noDataState: OK # 无数据视为正常
          annotations:
            summary: 命名空间 {{ $labels.namespace }} 下 Pod {{ $labels.pod }} 处于 {{ $labels.phase }} 状态
          labels:
            product: infra # 供通知模板与策略路由使用
  ```

- 注意三个工程细节：
  - `PromQL` 里先做过滤与降噪：排除 `staging` 等低价值命名空间、只看存活超过 `3` 分钟的 `Pod`
  - `for: 5m` + `noDataState: OK` 是防告警风暴的两道闸
  - `labels.product` 是整条告警链路的路由键，从 `values` 按产品开关(`alerts.<product>.enabled`)决定渲染哪些规则

## 飞书告警链路

### 联系点：`webhook` + 自建机器人

- `Grafana` 原生不支持飞书，与其等官方支持，不如加一层**自建转发服务**：联系点配置为普通 `webhook`，指向自建的机器人服务，由它把 `Grafana` 的告警 payload 转成飞书群消息
- 好处是格式完全自主、鉴权自己掌控、以后接任何新 `IM` 都只改这一个服务：

  ```yaml
  apiVersion: 1
  contactPoints:
    - orgId: 1
      name: 服务端告警群
      receivers:
        - uid: 80192ad5-0865-4075-a69a-eeccccbf82d8
          type: webhook
          settings:
            url: https://bot.example.com/grafana/webhook # 自建机器人
            httpMethod: POST
            authorization_scheme: Bearer # 出口鉴权，防止伪造告警
            authorization_credentials: $BOT_GRAFANA_WEBHOOK_TOKEN
  ```

- `Bearer Token` 通过环境变量注入 `Grafana`(`grafana.env`)，再由 `envsubst` 在 `CI` 阶段填充，全程不落 `git`

### 通知策略

- 策略决定"谁收到什么、多久提醒一次"，`repeat_interval` 是防轰炸的关键参数：

  ```yaml
  apiVersion: 1
  policies:
    - orgId: 1
      receiver: grafana-default-email # 兜底接收器
      group_by: ['...']
      repeat_interval: 5m # 未恢复告警每 5 分钟重复提醒
      routes:
        - receiver: 服务端告警群 # 全部告警转发到飞书群
          continue: true # 继续匹配后续路由，便于后续按产品分流
  ```

### 通知模板

- 模板用 `Grafana` 的 `Go template` 语法，自定义标题与正文，把关键信息(`product`/`application` 标签、当前数值、图表与静默链接)组织成飞书消息卡片的字段：

  ```yaml
  apiVersion: 1
  templates:
    - orgId: 1
      name: "example.title"
      template: |
        {{- if len .Alerts.Firing -}}
          🚨🚨🚨 有 {{ len .Alerts.Firing }} 条告警规则正在触发
        {{- end }}
        {{- if and (len .Alerts.Firing) (len .Alerts.Resolved) -}}; {{ end -}}
        {{- if len .Alerts.Resolved -}}
          ✅✅✅ 有 {{ len .Alerts.Resolved }} 条告警规则已经恢复
        {{- end }}
    - orgId: 1
      name: "example.message"
      template: |
        - 产品名称: {{ .Labels.product }}
        - 产品应用: {{ or (or .Labels.application .Labels.pod) "没有数据" }}
        - 告警规则: {{ .Labels.alertname }}
        - 当前数值: {{ range $k, $v := .Values }}{{ $k }}={{ $v }} {{ end }}
        {{- if .DashboardURL }}
        - 查看图表: [点击查看]({{ .DashboardURL }})
        {{- end }}
        {{- if .SilenceURL }}
        - 抑制告警: [点击抑制]({{ .SilenceURL }})
        {{- end }}
  ```

- `.Labels.application` 正是 `Prometheus` 抓取配置里 `relabel` 出来的服务名标签——**采集端打的标签决定通知端的信息量**，这是整条链路设计的一体性所在

## 飞书登录(`OAuth`)

- `Grafana` 的登录同样接入飞书，用 `generic_oauth` 对接飞书开放平台的 `OAuth` 端点，账号体系与公司 `IM` 统一：

  ```yaml
  grafana.ini:
    auth.generic_oauth:
      enabled: true
      name: 飞书
      client_id: $FEISHU_APP_ID # CI 注入
      client_secret: $FEISHU_APP_SECRET
      auth_url: https://passport.feishu.cn/suite/passport/oauth/authorize
      token_url: https://passport.feishu.cn/suite/passport/oauth/token
      api_url: https://passport.feishu.cn/suite/passport/oauth/userinfo
      login_attribute_path: name
      role_attribute_path: name == 'someone' && 'Admin' || 'Viewer' # 按用户名映射角色
  ```

- `role_attribute_path` 实现"白名单管理员、其余只读"，避免给全员开编辑权限

## 链路总结

  ```text
  Pod/Envoy 指标 ──> Prometheus(注解发现/relabel 打标) ──┐
  容器日志 ──> Promtail(CRI 解析) ──> Loki(S3/分级保留) ──┤
                                                        v
                                     Grafana(数据源/仪表盘/告警，全部 ConfigMap + sidecar 供给)
                                                        |
                              告警规则(interval/for/labels.product)
                                                        v
                              通知策略(repeat_interval) ─> 联系点(webhook + Bearer)
                                                        v
                                     自建机器人 ──> 飞书群卡片(title/message 模板)
  ```
