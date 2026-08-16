---
title: "PLG: Prometheus + Loki + Grafana 可观测体系"
date: 2026-08-16T00:00:00.000Z
categories:
  - Tools & Utilities
  - Container
tags:
  - beginner
mathjax: true
---

<!-- placeholder -->
<!-- more -->

# `PLG`

## 快速入门

### 简介

- **`PLG`** 指以 **`Prometheus` + `Loki` + `Grafana`** 为核心的可观测性体系，分别负责**指标**、**日志**与**可视化/告警**，采集日志还需要 **`Promtail`**，四者的关系：

  ```text
  应用/容器 ──(pull 指标)──> Prometheus ──┐
  应用/容器 ──(tail 日志)──> Promtail ──push──> Loki ──┐
                                                    ├──> Grafana(查询/仪表盘/告警)
  ```

- 相比 `ELK` 全文索引的重量级方案，`Loki` 只对**标签**建索引、日志正文压缩存储，以极低的成本换来"够用"的日志检索能力，与 `k8s` 的标签体系天然契合
- `Grafana` 是唯一面向用户的入口：指标与日志统一查询、仪表盘与告警规则统一管理，且全部资源支持**配置化供给**(`provisioning`)，可以像管理应用一样用 `git` + `helm` 管理监控体系

## `Prometheus`

### 简介

- **`pull` 模型**：`Prometheus` 主动周期性抓取(`scrape`)目标的 `/metrics` 端点，而非等待推送，天然适合 `k8s` 的自注册环境
- 数据以时序形式本地存储(`TSDB`)，配合 `PromQL` 做聚合查询
- 基本配置结构：`scrape_configs` 定义一组抓取任务(`job`)，每个任务通过**服务发现**找到目标，再通过 `relabel` 改写标签

### 服务发现与注解发现

- `k8s` 环境用 `kubernetes_sd_configs` 发现目标，`role` 可选 `pod`/`service`/`endpoints` 等
- 生产实践是**注解驱动**：应用只在 `Service`/`Pod` 上打 `prometheus.io/*` 注解即可被发现，零额外配置

  ```yaml
  additionalScrapeConfigs:
    - job_name: kubernetes-service-endpoints
      kubernetes_sd_configs:
        - role: endpoints
      relabel_configs:
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
          action: keep # 只保留声明了注解的 Service
          regex: true
        - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_port]
          action: replace
          target_label: __address__
          regex: (.+)(?::\d+);(\d+) # 用注解端口替换默认地址
          replacement: $1:$2
        - source_labels: [__meta_kubernetes_service_name]
          action: replace
          target_label: application # 服务名映射为 application 标签，告警信息直接可用
  ```

- `Istio` 网格的指标同样可以低成本接入：按容器端口名过滤，抓取 `sidecar` 暴露的 `Envoy` 指标

  ```yaml
  - job_name: envoy-stats
    metrics_path: /stats/prometheus
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_container_port_name]
        action: keep
        regex: .*-envoy-prom
  ```

### 存储

- 数据目录挂在 `PVC` 上即可持久化；重装时应**复用已有云盘**：`PVC` 通过 `volumeName` + `selector` 钉死既有的 `PV`，监控历史不因重装丢失

  ```yaml
  storageSpec:
    volumeClaimTemplate:
      spec:
        selector:
          matchLabels:
            app.kubernetes.io/name: monitoring
        storageClassName: monitoring
        volumeName: monitoring # 直接绑定已有 PV
        resources:
          requests:
            storage: 300Gi
  ```

## `Promtail`

### 简介

- `Promtail` 是 `Loki` 官方采集代理，以 `DaemonSet` 跑在每个节点，`tail` 容器日志文件并 `push` 给 `Loki`
- 采集时把 `k8s` 元数据(命名空间、`Pod` 名、标签)作为日志的标签附上，这就是 `Loki` 查询的主要维度

### 基本配置

  ```yaml
  config:
    clients:
      - url: http://loki-write-headless:3100/loki/api/v1/push # 推送地址
    snippets:
      pipelineStages:
        - cri: {} # 解析 CRI 日志格式(containerd 运行时的标准格式，拆出时间戳/级别/正文)
  ```

- `pipelineStages` 是加工管道，常用阶段：
  - `cri`/`docker`：解析容器运行时日志头
  - `regex`：正则提取字段，配合 `labels` 阶段提升为标签(标签基数要克制，高基数值不要做标签)
  - `timestamp`/`multiline`：修正时间戳、合并多行日志(如 `Java` 堆栈)
- 简单可扩展模式下的 `Loki` 有多个写入副本，推送地址用 `headless service`(`loki-write-headless`)让客户端负载均衡到所有 `write` 实例

## `Loki`

### 简介

- `Loki` 只索引标签，日志正文按块(`chunk`)压缩存储到对象存储，查询时按标签定位再过滤正文(`LogQL`)
- `LogQL` 速记：`{namespace="production", app=~"api.+"}` 选流，`| json` 解析正文，`| line_format` 格式化，指标查询再套聚合如 `rate(...[5m])`

### 部署模式

- `单体模式`(`单体` `SingleBinary`)：所有组件一个进程，小规模够用
- `简单可扩展模式`(`SimpleScalable`)：拆为 `read`/`write`/`backend` 三组，`read`/`write` 可独立扩容，日志量大时的性价比之选；注意两条数据面都要配持久化，且 `write` 的 `PVC` 开启随 `StatefulSet` 删除(`enableStatefulSetAutoDeletePVC`)

### 关键配置

- 存储与索引结构，对象存储放 `chunk`，`TSDB` 索引按天滚动：

  ```yaml
  loki:
    storage:
      type: s3
      bucketNames:
        chunks: loki-chunks # 日志块
        ruler: loki-ruler
        admin: loki-admin
      s3:
        s3: s3://loki
        endpoint: http://s3.internal # 兼容 S3 协议即可(各云厂商对象存储均可)
    schemaConfig:
      configs:
        - from: 2024-01-01
          store: tsdb
          object_store: s3
          schema: v13
          index:
            period: 24h
  ```

- 保留策略：全局保留 + **按流精细覆盖**，低价值环境短保留，成本立省

  ```yaml
  limits_config:
    retention_period: 744h # 默认保留 31 天
    retention_stream:
      - selector: '{namespace="staging"}'
        priority: 1
        period: 168h # staging 只留 7 天
      - selector: '{app=~".+job.+"}'
        priority: 2
        period: 168h # 任务类 Pod 日志也只留 7 天
  compactor:
    retention_enabled: true # 老版本用 tableManager，新版统一由 compactor 执行删除
  ```

## `Grafana`

### 数据源

- 数据源用 `provisioning` 配置，不做手工点击；`uid` 显式固定，仪表盘与告警规则都靠 `uid` 引用数据源

  ```yaml
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
        - name: Prometheus
          type: prometheus
          uid: prometheus
          url: http://monitoring-kube-prometheus-prometheus:9090
          isDefault: true
        - name: Loki
          type: loki
          uid: loki
          url: http://loki-read-headless:3100
          jsonData:
            timeout: 60
            maxLines: 1000 # 限制单次查询行数，防止误查打爆浏览器
  ```

### 仪表盘与配置即代码

- 仪表盘 `json` 放进 `ConfigMap`，打上标签(如 `grafana_dashboard: "1"`)，由 `Grafana` 的 **`sidecar`**(`k8s-sidecar`)监听全集群的同标签 `ConfigMap`，自动加载与更新
- 目录归类用注解指定，文件结构即目录结构：

  ```yaml
  kind: ConfigMap
  metadata:
    name: grafana-logs
    labels:
      grafana_dashboard: "1" # sidecar 按该标签发现
    annotations:
      k8s-sidecar-target-directory: "/tmp/dashboards/Logs" # 归入 Logs 目录
  data:
    logs.json: |
      { "title": "Logs", ... } # 仪表盘 JSON 全文
  ```

  ```yaml
  sidecar:
    dashboards:
      enabled: true
      provider:
        foldersFromFilesStructure: true # 按文件结构生成目录
    datasources:
      enabled: true
  ```

- 社区仪表盘(`grafana.com` 的 `gnetId`)可直接导入 `json` 后沉淀为 `ConfigMap`，再做定制
- 这套"标签 + `sidecar`"的模式同样用于**告警规则**，见下一篇的告警实践
