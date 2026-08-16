---
title: "Kubernetes: 生产级配置技巧"
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

# 生产级配置技巧

以下都是生产环境中的普遍做法(见 `Kubernetes` 一文的「`Charts` 实践」)，多数解决的是文档不会明说、但线上一定会踩的坑

## 优雅停机

- `Pod` 收到终止信号后，`Service`/`Istio` 的端点列表传播存在延迟，若容器立即退出，仍会有少量请求被路由到已死的 `Pod`
- 解决方式是**让容器先睡一会儿再退出**，用 `preStop` 钩子桥接端点传播延迟，同时放宽 `terminationGracePeriodSeconds` 给应用留足清理时间

  ```yaml
  spec:
    terminationGracePeriodSeconds: 60 # 给 preStop + 应用关闭留足时间(默认仅 30s)
    containers:
      - name: app
        lifecycle:
          preStop:
            exec:
              command: ["sh", "-c", "sleep 15"] # 等待 LB/网格端点列表更新，期间继续处理存量请求
  ```

## `HPA` 与 `replicas` 的冲突

- `Deployment` 的 `spec.replicas` 与 `HPA` 会互相打架：`HPA` 扩容后，下一次 `apply` 会把副本数改回 `yaml` 里的值
- 解决方式是在模板层做互斥：**仅当 `HPA` 未启用时才渲染 `replicas` 字段**，启用 `HPA` 后副本数完全交给自动扩缩

  ```yaml
  spec:
    {{- if and (not .Values.hpa.enabled) (hasKey .Values "replicas") }}
    replicas: {{ .Values.replicas }}
    {{- end }}
  ```

## 配置变更也触发滚动重启

- 滚动更新只比较 `Pod` 模板哈希，若仅改了环境变量之外的间接配置(如依赖的配置中心)，`helm upgrade` 后 `Pod` 不会重建，容易造成“升级了但没生效”
- 解决方式是注入一个**每次渲染都变化的随机注解**，强制每次发布都滚动重启

  ```yaml
  spec:
    template:
      metadata:
        annotations:
          rollme: {{ randAlphaNum 5 | quote }} # 每次 helm upgrade 都变化，强制重建 Pod
  ```

## `Job` 与 `sidecar` 的死锁

- 被注入 `Istio` `sidecar` 的 `Job` 可能永远不结束：业务容器退出后，`Envoy` `sidecar` 仍在运行，`Pod` 无法到达 `Completed` 状态
- 解决方式是给任务类 `Pod` **显式关闭注入**

  ```yaml
  spec:
    template:
      metadata:
        annotations:
          sidecar.istio.io/inject: "false" # 数据库迁移等一次性 Job 不需要网格能力
    backoffLimit: 2 # 快速失败，避免反复重试污染数据
  ```

- 同理，`CronJob` 若只是访问数据库或外部 `API`，也应关闭注入以避免资源浪费与完成阻塞

## 探针的节奏控制

- 探针不是越灵敏越好：`periodSeconds` 过短 + 应用 `GC` 停顿会导致误判重启，形成**重启风暴**(流量涌入冷启动应用 → 更慢 → 更多失败)
- 生产实践：存活探针放宽到 `60s` 一次，就绪探针保持 `15s`，`timeoutSeconds: 10` 容忍慢响应
- `Spring Boot` 应用直接使用 `actuator` 的分层健康端点，避免把依赖检查混入存活探针

  ```yaml
  livenessProbe:
    httpGet:
      path: /actuator/health/liveness # 只关心进程是否活着
      port: 8080
    initialDelaySeconds: 10
    periodSeconds: 60 # 放宽节奏，避免 GC 停顿引发误杀
    timeoutSeconds: 10
  readinessProbe:
    httpGet:
      path: /actuator/health/readiness # 关心依赖是否就绪
      port: 8080
    initialDelaySeconds: 10
    periodSeconds: 15
    timeoutSeconds: 10
  ```

## 免注解的指标采集

- 不部署 `ServiceMonitor` 时，`Prometheus` 可通过 `Pod` 注解自动发现抓取目标，应用零配置接入监控

  ```yaml
  spec:
    template:
      metadata:
        annotations:
          prometheus.io/scrape: "true"
          prometheus.io/port: "8080"
          prometheus.io/path: "/actuator/prometheus"
  ```

## 容器感知的 `JVM` 调优

- `JVM` 默认按宿主机内存分配堆，在容器里会严重超配；同时 `CPU limit` 会被 `JVM` 识别为核心数，影响 `GC` 线程数
- 生产实践：`requests` 与 `limits` 设为相同值保证资源独占，再用环境变量覆盖 `JVM` 行为

  ```yaml
  resources:
    requests:
      cpu: "2"
      memory: 4Gi
    limits: # 与 requests 一致，避免 CPU 限流引发长尾延迟
      cpu: "2"
      memory: 4Gi
  env:
    - name: JAVA_TOOL_OPTIONS
      value: >-
        -XX:ActiveProcessorCount=2 -XX:MaxDirectMemorySize=128M
        -XX:InitialRAMPercentage=50 -XX:MinRAMPercentage=75 -XX:MaxRAMPercentage=75
  ```

  - `ActiveProcessorCount` 显式指定核心数，不受 `CPU` 配额干扰
  - `RAMPercentage` 按容器内存的百分比分配堆，替代写死的 `-Xmx`

## `helm hook` 编排发布顺序

- 私有镜像的拉取凭证必须在第一个 `Pod` 创建前就绪，否则首个 `Deployment` 会卡在 `ImagePullBackOff`
- 用 `pre-install`/`pre-upgrade` 钩子提前创建，`hook-weight` 保证其先于其他钩子资源执行；数据库迁移等任务则用 `post-install` 钩子跟上，`weight` 决定多个任务的先后

  ```yaml
  apiVersion: v1
  kind: Secret
  type: kubernetes.io/dockerconfigjson
  metadata:
    name: my-app-regcred
    annotations:
      "helm.sh/hook": pre-install,pre-upgrade # 在 Deployment 之前创建
      "helm.sh/hook-weight": "-5" # 数值越小越先执行
  data:
    .dockerconfigjson: {{ template "imagePullSecret" . }}
  ```

  ```yaml
  apiVersion: batch/v1
  kind: Job
  metadata:
    name: my-app-migration
    annotations:
      "helm.sh/hook": post-install,post-upgrade # 应用就绪后执行数据库迁移
      "helm.sh/hook-weight": "0"
  ```

- 凭证 `Secret` 需要在使用它的每个命名空间各放一份(如 `cert-manager` 拉取镜像的命名空间)，模板中渲染多份即可

## 对象存储挂载为本地盘

- 通过各云厂商的 `CSI` 驱动，把对象存储桶直接挂载为 `PV`，应用像读写本地目录一样访问 `OSS`/`GCS`，无需引入 `SDK`
- `GCS` 的方式最简单，注解即可启用 `GKE` 托管的 `gcsfuse` `sidecar`，还能单独限制 `sidecar` 资源

  ```yaml
  spec:
    template:
      metadata:
        annotations:
          gke-gcsfuse/volumes: "true" # 让 GKE 注入 gcsfuse sidecar
          gke-gcsfuse/cpu-limit: "250m"
          gke-gcsfuse/memory-limit: "256Mi"
  ```

- `OSS` 需要手写静态 `PV`，两个细节：**每个桶用独立的 `storageClassName`** 防止默认供给器抢占绑定；`PVC` 用 **`selector` 按标签钉死**要绑定的 `PV`

  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: my-app-oss-csi-pv-0
    labels:
      alicloud-pvname: my-app-oss-0 # 供 PVC selector 匹配
  spec:
    storageClassName: my-app-csi-oss-0 # 每桶唯一，避免被动态供给拦截
    accessModes:
      - ReadWriteMany
    persistentVolumeReclaimPolicy: Retain
    csi:
      driver: ossplugin.csi.alibabacloud.com
      volumeHandle: my-app-oss-csi-pv-0
      volumeAttributes:
        bucket: my-bucket
        url: oss-cn-hangzhou-internal.aliyuncs.com
        akId: "<access-key>"
        akSecret: "<secret-key>"
  ---
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-app-pvc-0
  spec:
    storageClassName: my-app-csi-oss-0
    accessModes:
      - ReadWriteMany
    resources:
      requests:
        storage: 10Gi # 对象存储不校验，仅作形式声明
    selector:
      matchLabels:
        alicloud-pvname: my-app-oss-0 # 按标签绑定指定 PV
  ```

- 有状态监控组件(如 `Prometheus`)重装时应保留数据：手写 `PV` 直接引用**已存在的云盘**(`volumeHandle` 填云盘 `ID`)，配合 `persistentVolumeReclaimPolicy: Retain`，删除 `chart` 重装后数据仍在

  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: monitoring
  spec:
    storageClassName: monitoring
    accessModes:
      - ReadWriteOnce
    capacity:
      storage: 500Gi
    csi:
      driver: udisk.csi.ucloud.cn
      volumeHandle: "<已有云盘 ID>" # 复用存量云盘
    persistentVolumeReclaimPolicy: Retain # PVC 删除后保留云盘与数据
  ```

## 出站默认拒绝与元数据防护

- 应用被 `SSRF` 攻击时，攻击者最先尝试的就是云厂商的元数据服务地址，拿到的是**临时云凭证**，可直接操纵云资源
- 生产实践：`NetworkPolicy` 默认拒绝所有出站，仅放行 `DNS`，并显式排除元数据网段

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: NetworkPolicy
  metadata:
    name: my-app
  spec:
    podSelector:
      matchLabels:
        app: my-app
    policyTypes:
      - Egress
    egress:
      - to: # 放行集群 DNS
          - namespaceSelector:
              matchLabels:
                kubernetes.io/metadata.name: kube-system
            podSelector:
              matchLabels:
                k8s-app: kube-dns
          ports:
            - protocol: UDP
              port: 53
      - to: # 放行其余公网，但屏蔽元数据网段
          - ipBlock:
              cidr: 0.0.0.0/0
              except:
                - 100.64.0.0/10 # 阿里云等 CG-NAT 网段(含 metadata 100.100.100.200)
                - 169.254.0.0/16 # link-local(含 AWS/GCP/Azure 169.254.169.254)
  ```

- 在此之上可为特殊角色开白名单，例如调试用的 `admin` shell `Pod` 只允许访问 `API Server` 端口，不开内网通行证

  ```yaml
  egress:
    - to: # 仅允许访问 apiserver
        - ipBlock:
            cidr: 10.0.0.0/8
      ports:
        - protocol: TCP
          port: 443
        - protocol: TCP
          port: 6443
  ```

## 应用内嵌调试终端

- 让应用持有专属 `ServiceAccount` 与最小 `RBAC`(创建 `Job`、`pods/exec`、读写 `Secret`)，即可在网页里一键拉起带完整工具链的调试 `Pod`，进容器排查线上问题
- 关键是**权限最小化**：`Role` 只授予单个命名空间内的少量动词，调试 `Pod` 再用 `NetworkPolicy` 隔离

  ```yaml
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    name: my-app-shell
  rules:
    - apiGroups: ["batch"]
      resources: ["jobs"]
      verbs: ["get", "list", "watch", "create", "delete"]
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "list", "watch"]
    - apiGroups: [""]
      resources: ["pods/exec"]
      verbs: ["create", "get"]
  ```

  ```yaml
  # 应用 Pod 将自身命名空间下发给调试 Pod，无需硬编码
  env:
    - name: TOOLS_SHELL_POD_NAMESPACE
      valueFrom:
        fieldRef: # Downward API：把 Pod 自身信息注入环境变量
          fieldPath: metadata.namespace
  ```

## 网关证书的域名数量上限

- `Istio` `Gateway` 的单个 `server` 块在证书 `SAN` 数量上有实际上限(约 `100` 个域名)，域名多时会直接报错
- 解决方式是**按 `99` 个一组拆分 `server` 块**，每组配独立的证书 `Secret`，模板层用循环批量生成

  ```yaml
  servers:
    - port:
        number: 443
        name: https
        protocol: HTTPS
      tls:
        mode: SIMPLE
        credentialName: my-app-tls-1 # 第一组证书
      hosts: # ≤ 99 个域名
        - a.example.com
    - port:
        number: 443
        name: https-2 # 端口相同，name 必须不同
        protocol: HTTPS
      tls:
        mode: SIMPLE
        credentialName: my-app-tls-2 # 第二组证书
      hosts:
        - b.example.com
  ```

## 入口层的最小暴露面

- 业务不需要的端点一律在网关层直接 `404`，而不是靠应用代码防御：管理端点(`/actuator/*`)只放行健康检查，文档路由(`/docs`)按环境开关

  ```yaml
  http:
    - match: # 禁用文档路由
        - uri:
            prefix: /docs
      directResponse: # 不经过后端直接返回
        status: 404
    - match: # 仅放行健康检查
        - uri:
            exact: /actuator/health
      route:
        - destination:
            host: my-app
            port:
              number: 80
    - match: # 其余管理端点全部 404
        - uri:
            prefix: /actuator
      directResponse:
        status: 404
    - route: # 正常业务流量
        - destination:
            host: my-app
            port:
              number: 80
  ```

- 多产品共用一套部署时，可在网关层按域名路由并**注入产品标识 `Header`**，后端无感知区分租户

  ```yaml
  http:
    - match:
        - headers:
            ":authority": # Envoy 的 Host 头
              exact: product-a.example.com
      route:
        - destination:
            host: my-app
            port:
              number: 80
          headers:
            request:
              set:
                X-Product: "a" # 后端读取该 Header 区分产品
  ```

## 全链路请求 `ID` 透传

- 外部客户端带来的 `X-Request-Id` 默认不会在网关处保留，导致跨服务的日志无法串联
- 用 `EnvoyFilter` 直接修改网关配置，让网关**透传外部请求 `ID`**，全链路日志与 `trace` 得以对齐

  ```yaml
  apiVersion: networking.istio.io/v1
  kind: EnvoyFilter
  metadata:
    name: ingress-preserve-request-id
  spec:
    workloadSelector:
      labels:
        istio: ingress # 作用于 ingress gateway Pod
    configPatches:
      - applyTo: NETWORK_FILTER
        match:
          context: GATEWAY
          listener:
            filterChain:
              filter:
                name: envoy.filters.network.http_connection_manager
        patch:
          operation: MERGE
          value:
            typed_config:
              "@type": type.googleapis.com/envoy.extensions.filters.network.http_connection_manager.v3.HttpConnectionManager
              preserve_external_request_id: true
  ```

## 环境即命名空间

- 用污点 + 容忍度把节点按环境隔离：`staging` 命名空间的发布只调度到 `staging` 节点，互不抢资源；命名空间统一打上 `istio-injection` 标签，新服务自动入网格
- 模板层以 `Release.Namespace` 推导默认值，用户不配置则自动隔离

  ```yaml
  # 节点打污点: kubectl taint node node1 kingboat.io/environment=production:NoSchedule
  tolerations:
    - key: kingboat.io/environment
      operator: Equal
      value: {{ .Release.Namespace | quote }} # 容忍度跟随命名空间，自动匹配环境节点
      effect: NoSchedule
  ```

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: staging
    labels:
      istio-injection: enabled # 命名空间内新建 Pod 自动注入 sidecar
  ```
