---
title: "Helm: Kubernetes 的包管理器"
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

# `Helm`

## 快速入门

### 简介

- **`Helm`** 是 `k8s` 官方推荐的包管理器，定位类似于 `apt`/`brew`：把一组相互关联的 `k8s` 资源(`Deployment`、`Service`、`Ingress`...)打包为 **`Chart`**，一条命令完成安装、升级与回滚
- 相比手写一堆 `yaml` 再 `kubectl apply`，`helm` 解决了三个问题：
  - **参数化**：同一份模板通过不同的 `values` 部署到不同环境，而不是复制出多份几乎相同的 `yaml`
  - **生命周期**：`Release` 记录每次发布的版本与清单，升级失败可一键回滚到任意历史版本
  - **复用**：第三方应用(`mongodb`、`prometheus`...)直接复用社区 `Chart`，不必自己从零编写
- [官方文档](https://helm.sh/docs)

### 核心概念

- **`Chart`**：一个应用完整的模板集合，即 `helm` 的"包"
- **`Release`**：`Chart` 的一次安装实例，同一 `Chart` 可以在同一集群安装多次，每次都是独立的 `Release`，靠 `Release` 名称区分
- **`Repo`**：存放 `Chart` 的仓库，类似 `docker hub`，每个仓库由一个 `index.yaml` 索引

## `Chart` 结构

```text
my-chart/
├── Chart.yaml          # Chart 的元信息与依赖声明
├── values.yaml         # 默认参数，渲染模板时的兜底值
├── values-prod.yaml    # 环境覆盖文件，可选
├── Chart.lock          # 依赖锁文件，由 dependency update 生成
├── charts/             # 依赖的子 Chart，由 dependency update 下载
├── templates/
│   ├── _helpers.tpl    # 命名模板(下划线开头不渲染为资源)，沉淀命名规范
│   ├── deployment.yaml
│   ├── service.yaml
│   └── NOTES.txt       # 安装成功后打印的使用说明
└── .helmignore         # 打包时忽略的文件，类似 .gitignore
```

- **`Chart.yaml`** 关键字段：

  ```yaml
  apiVersion: v2
  name: my-chart
  description: Helm Chart for my app
  type: application # application(可部署)/library(仅提供模板)
  version: 0.0.0 # Chart 自身版本，生产实践可固定为 0.0.0 交给 CI 注入
  appVersion: "1.0.0" # 内含应用的版本，仅作描述，不参与渲染
  dependencies: # 依赖的子 Chart
    - name: mongodb # 上游 Chart 名称
      version: 15.6.1 # 依赖版本
      repository: https://charts.bitnami.com/bitnami # 上游仓库
      alias: mongodb # 别名，同一依赖可按别名多次引入
      condition: mongodb.enabled # 根据 values 开关决定是否渲染
  ```

- 命名规范：`Chart` 名称小写字母与数字；`version` 与 `appVersion` 是两个独立概念，前者是模板的版本，后者是应用的版本

## 常用命令

### 安装与升级

- `helm install [release] [chart] -n [ns] --create-namespace -f values-prod.yaml`：安装一个 `Release`，命名空间不存在时自动创建
- `helm upgrade [release] [chart] -f values-prod.yaml`：升级已存在的 `Release`
  - `--install`：常与 `upgrade` 合用为 `helm upgrade --install`(可缩写 `-i`)，不存在则安装、存在则升级，是 `CI` 中的标准写法
  - `--atomic`：升级失败自动回滚到上一版本，生产发布必备
  - `--wait`：等待所有资源就绪才返回，配合探针保证发布串行化
  - `--timeout 10m`：`--wait` 的超时时间
- `helm uninstall [release] -n [ns]`：卸载 `Release`，删除其管理的所有资源(`Secret`/`PVC` 等是否保留取决于资源的安装顺序注解)

### 查询与回滚

- `helm list -n [ns] -a`：列出 `Release`，`-a` 包含已卸载的
- `helm history [release]`：查看发布历史与状态
- `helm rollback [release] [revision]`：回滚到指定版本，不指定版本则回退一个
- `helm get values [release]`：查看当前生效的 `values`；`helm get manifest [release]`：查看实际渲染进集群的清单，排查"模板到底渲染了什么"的第一入口
- `helm status [release]`：查看 `Release` 状态

### 本地调试

- `helm create [name]`：生成一个带示例模板的 `Chart` 骨架
- `helm lint`：静态检查 `Chart` 的规范问题
- `helm template [chart] -f values-prod.yaml`：本地渲染模板并打印结果，**不接触集群**，调试模板最常用的命令
  - `--show-only templates/deployment.yaml`：只看单个文件的渲染结果
  - `--debug`：渲染报错时输出完整堆栈
- `helm diff upgrade [release] [chart]`(需安装 `helm-diff` 插件)：升级前预览集群内清单与待应用清单的差异，`Code Review` 的好帮手

### 依赖与仓库

- `helm dependency update`：按 `Chart.yaml` 下载依赖到 `charts/` 并生成 `Chart.lock`
- `helm dependency build`：按 `Chart.lock` 锁定的版本下载，保证可复现构建，`CI` 中应使用此命令
- `helm repo add [name] [url]` / `helm repo update`：添加仓库并刷新索引
- `helm package [chart]`：打包为 `.tgz`；`helm push` 推送到仓库

## 模板语法

### 对象与取值

- 渲染上下文的顶层对象用 `.` 访问，常用的有：
  - `.Values`：合并后的参数(默认 `values.yaml` + `-f` 文件 + `--set`)
  - `.Release.Name` / `.Release.Namespace`：`Release` 的名称与命名空间，是资源命名的第一来源
  - `.Chart.Name` / `.Chart.AppVersion`：`Chart` 元信息
  - `.Files` / `.Capabilities`：访问 `Chart` 内文件、查询集群版本
- `{{ }}` 输出值，`{{- }}` 去掉前侧空白，`{{ -}}` 去掉后侧空白，常用于压缩渲染结果中的空行

  ```yaml
  metadata:
    name: {{ .Release.Name }} # 资源一律以 Release 命名，同一 Chart 可多实例部署
  ```

### 常用函数与管道

- 函数通过管道 `|` 串联，最常用的组合是 **`toYaml | nindent`**：把一段 `values` 结构原样转成 `yaml` 并整体缩进，是"参数块透传"的标准写法
- `nindent` 比 `indent` 多一个前置换行，避免值被拼接到上一行尾部，**优先使用 `nindent`**

  ```yaml
  tolerations:
    {{- .Values.tolerations | toYaml | nindent 8 }}
  ```

- 其他高频函数：
  - `quote`：加引号，数字类值(`cpu: 2`)必须加，否则被渲染为数字而校验失败
  - `default`：兜底默认值，`{{ .Values.replicas | default 1 }}`
  - `required`：缺失即报错并中断渲染，用于强制用户提供关键参数
  - `upper` / `replace`：转换 `values` 键名，如把 `feishu-app-id` 转成环境变量名 `FEISHU_APP_ID`
  - `randAlphaNum 5`：随机字符串，配合注解实现"每次发布强制滚动重启"
  - `printf`：格式化字符串，`{{ printf "%s-%s" .Release.Namespace .Release.Name }}`

### 控制结构

- `if/else if/else`：条件渲染，判空用 `not (empty .Values.x)` 而非直接判布尔(`0`、`""`、空列表都是假值，容易误判)

  ```yaml
  {{- if not (empty .Values.secrets) }}
  env:
    {{- range $key, $value := .Values.secrets }}
    - name: {{ $key | upper | replace "-" "_" }}
      valueFrom:
        secretKeyRef:
          name: {{ $.Release.Name }}-secrets
          key: {{ $key }}
    {{- end }}
  {{- end }}
  ```

- `range`：遍历列表或字典，`range $idx, $bucket := .Values.storage.buckets` 同时取下标与值
- `with`：切换作用域，`with .Values.gateway` 内部直接用 `.hosts`；作用域内访问顶层需用根对象 `$`(如 `$.Release.Name`)
- `hasKey`：判断键是否存在，`{{ if and (not .Values.hpa.enabled) (hasKey .Values "replicas") }}`

### 命名模板

- 复杂的命名规则(截断长度、拼接 `Chart` 名与 `Release` 名)沉淀在 `_helpers.tpl` 中，用 `define` 定义、`include` 调用

  ```yaml
  # templates/_helpers.tpl
  {{- define "my-chart.fullname" -}}
  {{- if .Values.fullnameOverride }}
  {{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
  {{- else }}
  {{- printf "%s-%s" .Release.Name (include "my-chart.name" .) | trunc 63 | trimSuffix "-" }}
  {{- end }}
  {{- end }}
  ```

- `include` 比 `template` 多了管道能力(可接 `| nindent`)，统一用 `include` 即可
- 资源间的引用(`Deployment` 引用 `Secret` 名、`PVC` 名)一律通过同一命名模板或同一 `printf` 表达式生成，避免改一处漏一处

## `values` 管理策略

- 覆盖优先级从低到高：`values.yaml` → `-f` 指定的文件(后者覆盖前者) → `--set`/`--set-string`
- `--set` 适合 `CI` 中注入动态值(镜像 `tag`、版本号)；复杂结构用文件管理，`--set` 写嵌套极易出错
- 数字与特殊字符类值用 `--set-string` 强制为字符串，避免类型漂移
- 环境差异组织为 `values-staging.yaml` / `values-prod.yaml`，公共默认值留在 `values.yaml`，覆盖文件只写差异项
- 未使用的参数显式置空(`~`)，模板中用 `if` 判断空值决定是否渲染，保持渲染输出最小化
- 敏感信息不进 `values` 文件与 `git`，由 `CI` 在发布时从密钥管理系统注入，模板中用 `secretKeyRef` 引用

## 发布编排(`Hooks`)

- `helm` 在发布生命周期的固定时点执行带 `helm.sh/hook` 注解的资源，解决"先有鸡还是先有蛋"的顺序问题：

  ```yaml
  metadata:
    annotations:
      "helm.sh/hook": pre-install,pre-upgrade # 在主资源之前执行
      "helm.sh/hook-weight": "-5" # 数值越小越先执行，默认 0
      "helm.sh/hook-delete-policy": before-hook-creation # 重复执行前先删旧 Job(默认行为)
  ```

- 常用钩子时点：
  - `pre-install`/`pre-upgrade`：主资源创建前，适合创建前置依赖，如私有镜像的拉取凭证(`Secret` 必须先于第一个 `Pod` 存在)
  - `post-install`/`post-upgrade`：主资源就绪后，适合数据库迁移、数据初始化等任务
  - `pre-delete`/`post-delete`：卸载前后，适合清理外部资源
- 钩子资源须是能"跑完"的类型(`Job`、`Secret` 等)，`Job` 类钩子配合 `restartPolicy: Never` + `backoffLimit` 快速失败

## 工程实践

- **`umbrella chart` 封装第三方依赖**：不直接 `install` 社区 `Chart`，而是声明为依赖并打上 `alias` 与 `condition`，统一在自家 `values` 里覆盖上游参数，升级只改一处版本号，定制与追踪都集中在一个仓库
- **版本交给 `CI`**：自建 `Chart` 的 `version` 固定为 `0.0.0`，由 `CI` 发布时注入，发布产物天然可回溯到 `commit`
- **一切命名源于 `Release`**：资源名、`Secret` 名、`PVC` 名统一用 `.Release.Name`/`.Release.Namespace` 生成，同一 `Chart` 即可在同一集群多实例部署
- **用命名空间表达环境**：模板以 `.Release.Namespace` 推导默认 `nodeSelector`/`tolerations` 等，`values` 不配置则自动按环境隔离，用户仅在需要时覆盖
- **互斥渲染避免冲突**：存在依赖互斥的字段(如 `HPA` 与 `spec.replicas`)在模板层做条件互斥，不依赖使用者自觉
- **发布命令标准化**：`CI` 统一使用 `helm upgrade --install --atomic --wait`，失败自动回滚，配合 `helm diff` 在合并请求中展示变更
