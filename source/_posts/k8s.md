---
title: "Kubernetes: 生产级的分布式集群部署"
date: 2026-04-20T00:00:00.000Z
categories:
  - Tools & Utilities
  - Container
tags:
  - beginner
mathjax: true
---

<!-- placeholder -->
<!-- more -->

# `Kubernetes`

## 快速入门

### 简介

- **`Kubernetes`**(简称 **`k8s`**)是一个开源的容器编排平台。相比于`Docker`，它能够在**集群级别**自动化地管理容器的部署、扩缩容和运维，但学习成本也相对更高
- 除了基础的容器调度与自愈能力，`k8s`原生支持**服务发现、负载均衡、滚动更新与配置管理**等服务治理，配合 **`Istio`** 等服务网格组件，可以**替代传统的`RPC`框架**，将流量治理、可观测性、安全等能力下沉至基础设施层
- [官方文档](https://kubernetes.io/docs)

### 工具介绍

- 对本地环境，`minikube` 提供了一种 `k8s` 的单节点集群方案
  - `minikube start --driver=docker`：用 `docker` 驱动启动一个 `minikube` 容器(集群)，让它接管 `docker`
  - `minikube status`：查看集群状态
  - `minikube stop`：停止集群
  - `minikube delete`：删除集群
  - `minikube dashboard`：快速开启一个由 `minikube` 管理的 `dashboard`(管理容器的可视化网页)
- `k3s` 是一个轻量级的 `k8s` 发行版，单个二进制文件即包含全部组件，内存占用约 `512MB`，适合低配机器、边缘设备与 `CI` 环境，也可作为本地方案的另一选择
  - `curl -sfL https://get.k3s.io | sh -`：一条命令安装并启动，自带 `containerd`、`Flannel` 网络、本地存储与 `Service LoadBalancer`，开箱即用
  - `k3s kubectl get node`：内置了 `kubectl`，无需单独安装
  - 凭证位于 `/etc/rancher/k3s/k3s.yaml`，让外部 `kubectl`/`helm` 使用需 `export KUBECONFIG=/etc/rancher/k3s/k3s.yaml`
  - `k3s-killall.sh` / `k3s-uninstall.sh`：停止/彻底卸载
  - `k3d` 在 `docker` 容器里运行 `k3s`，可以像 `minikube` 一样快速创建和销毁多节点集群，兼顾轻量与隔离
- `kubectl` 是和 `k8s` 集群交互的命令行工具
  - `kubectl [command] [kind] [name] [flags]` 是一般的命令格式，其中 `kind` 是 `k8s API` 定义的类型
  - `kubectl get pod/service/node`：获取 `pod`/`service`/`node` 等的简短状态
  - `kubectl logs [podname]`：查看日志
  - `kubectl exec -it [podname] -- /bin/sh`：进入容器(`--interactive --tty`)
  - `kubectl apply [-f x.yaml]...`：应用配置
- `helm` 是 `k8s` 的包管理器，替代了 `apply` 并提供更好的管理方式
  - `Chart`：一个完整应用所需的全部配置，在其目录下
    - `Chart.yaml`：一个 `Chart` 的定义
    - `values-*.yaml`：所有环境变量的定义，默认查找 `values.yaml` 文件
    - `templates/`：`go tmpl` 文件所在目录，定义应用的配置模板
    - `Chart.lock`：锁文件，由 `helm dependency build` 根据 `Chart.yaml` 所依赖的 `chart` 的版本生成
    - `charts/`：本 `chart` 依赖的子 `charts`
  - `Release`：一个 `Chart` 在集群中的运行实例
  - `Repo`：类似 `docker hub`，每个仓库有一个 `index.yaml`

## 核心字段

### `apiVersion`与`kind`

- `k8s`使用声明式管理，所有集群的状态通过`yaml`来定义
- **`apiVersion`**：`api`版本定义，格式为`GROUP/VERSION`，一般来说`v1`是稳定版本，但存在特例
  - 核心组(`v1`)：基础资源，包含`Pod`、`Service`、`ConfigMap`、`Secret`、`Namespace`
  - `apps/v1`：工作负载管理，包含`Deployment`、`StatefulSet`、`DaemonSet`、`ReplicaSet`
  - `batch/v1`：批处理任务，包含`Job`、`CronJob`
  - `networking.k8s.io/v1`：网络配置，包含`Ingress`、`NetworkPolicy`
  - `rbac.authorization.k8s.io/v1`：`RBAC`权限控制，包含`Role`、`RoleBinding`、`ClusterRole`、`ClusterRoleBinding`
  - `storage.k8s.io/v1`：存储管理，包含`StorageClass`、`VolumeAttachment`
  - `autoscaling/v2`：自动扩缩容，包含`HorizontalPodAutoscaler`(`v1`已废弃，生产使用`v2`)
  - `apiextensions.k8s.io/v1`：自定义资源，包含`CustomResourceDefinition`(CRD)，不常用
  - `networking.istio.io/v1`：`Istio` 网络配置，包含`VirtualService`、`DestinationRule`、`Gateway`、`ServiceEntry`
  - `security.istio.io/v1`：`Istio` 安全配置，包含`AuthorizationPolicy`、`PeerAuthentication`
  - `telemetry.istio.io/v1`：`Istio` 可观测性配置，包含`Telemetry`
  - 使用`kubectl explain [kind]`查看资源对应的`apiVersion`和字段说明
  - 使用`kubectl api-resources`查看所有资源及其所属`API`组
- **`kind`**：资源类型，如`Pod`、`Deployment`、`Service`
  - `Pod`：最小部署单元，包含一个或多个容器，适用于临时调试、手动运行任务
  - `Deployment`：管理无状态应用，支持滚动更新和回滚，最常用的控制器
  - `StatefulSet`：管理有状态应用(如数据库)，提供稳定的网络标识和持久存储
  - `DaemonSet`：在每个节点上运行一个 `Pod` 副本，适用于日志收集、监控代理
  - `ReplicaSet`：维护 `Pod` 副本数量，`Deployment` 底层依赖，通常不直接使用
  - `Job`：一次性任务，执行完成后退出
  - `CronJob`：定时任务，按 `Cron` 表达式周期性执行
  - `Service`：为 `Pod` 提供稳定的访问入口，支持负载均衡
  - `Ingress`：七层负载均衡，基于域名和路径路由到不同 `Service`
  - `ConfigMap`：存储非敏感配置数据，以键值对形式挂载到 `Pod`
  - `Secret`：存储敏感信息(密码、密钥)，`base64` 编码存储，本身并不加密也不安全，只是因为敏感信息通常存在特殊字符甚至是二进制数据，而`k8s`传输数据使用`json`，因此通常转为`base64`，而`Secret`则是在`ConfigMap`的基础上添加一层`base64`编码校验的资源类型
  - `StorageClass`：存储模板，定义动态供给方式和存储参数
  - `PVC(PersistentVolumeClaim)`：命名空间级别的存储声明，由用户创建，引用 `StorageClass` 动态供给
  - `PV(PersistentVolume)`：集群级别的存储资源，云环境通常由 `StorageClass` 动态创建
  - `NetworkPolicy`：网络策略，限制 `Pod` 间的网络通信
  - `LimitRange`：资源限制范围，限制 `Pod` 的资源使用上下限
  - `ResourceQuota`：资源配额，限制命名空间的总资源使用量
  - `HorizontalPodAutoscaler`：自动扩缩容，根据 CPU/内存使用率调整 `Pod` 副本数
  - `Namespace`：资源隔离，不同命名空间的资源相互独立
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: pod-1
  ```

### `metadata`

- **`metadata.name`**：适用于所有资源，资源名称在命名空间内唯一，DNS 解析时会用到(如 Service 的 `name.namespace.svc.cluster.local`)
- **`metadata.labels`**：适用于所有资源，键值对标签用于资源筛选和关联，Service 通过 labels 匹配 Pod，Deployment 通过 labels 管理 Pod
- **`metadata.namespace`**：适用于所有资源，指定资源所属命名空间，默认为 `default`，系统组件在 `kube-system`
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
    labels:
      app: nginx
    namespace: default
  spec:
    containers:
      - name: nginx
        image: nginx:1.21
  ```

### `spec`

- **`spec.selector`**：适用于 Deployment、StatefulSet、DaemonSet、Job、Service，选择器匹配 Pod 的 labels，决定控制器管理哪些 Pod，必须与 `spec.template.metadata.labels` 匹配
- **`spec.template`**：适用于 Deployment、StatefulSet、DaemonSet、Job、CronJob，Pod 模板定义要创建的 Pod 的完整配置，包含 `metadata.labels` 和 `spec.containers`
- **`spec.replicas`**：适用于 Deployment、StatefulSet，期望的 Pod 副本数量，默认为 1
- 示例：

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-deployment
  spec:
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
          - name: nginx
            image: nginx:1.21
  ```

### `containers` 与 `volumes`

- **`spec.containers`**：适用于 `Pod`、`Deployment`、`StatefulSet`、`DaemonSet`、`Job`、`CronJob`，容器列表，每个容器包含以下字段：
  - `name`：容器名称
  - `image`：镜像名称和标签
  - `ports.containerPort`：容器暴露的端口
  - `resources.requests`：最小资源需求(`CPU`、内存)，调度依据
  - `resources.limits`：最大资源限制(`CPU`、内存)，防止资源争抢
  - `volumeMounts`：存储卷挂载路径
  - `env`：环境变量配置
  - `livenessProbe`：存活探针，检测容器是否存活，失败则重启容器
    - `initialDelaySeconds`：首次探测延迟
    - `periodSeconds`：探测间隔
    - `failureThreshold`：失败阈值，允许的最大连续探测失败次数，超出此阈值将重启pod
    - `successThreshold`：成功阈值，将该pod视为健康的最小连续探测成功次数，存活探针的成功阈值必须是1
    - 探测方式：`httpGet`(指定 `path` 和 `port`)、`tcpSocket`(指定 `port`)、`exec`(指定 `command`)
  - `readinessProbe`：就绪探针，检测容器是否就绪，失败则从 `Service` 摘除。参数与 `livenessProbe` 相同，但失败后不会重启容器，只是暂时停止流量进入
- **`spec.volumes`**：适用于 `Pod`、`Deployment`、`StatefulSet`、`DaemonSet`、`Job`、`CronJob`，存储卷定义，常用类型：
  - `emptyDir`：临时存储，`Pod` 删除时数据丢失，适用于缓存、临时文件
  - `configMap`：挂载 `ConfigMap`，需指定 `name`，可选择挂载特定 `items` 或全部键值对。支持 `optional: true`(`ConfigMap` 不存在时不阻塞启动)和 `defaultMode`(文件权限，默认 `0644`)
  - `secret`：挂载 `Secret`，参数与 `ConfigMap` 类似，额外支持 `secretName` 指定 `Secret` 名称。常用于挂载 `TLS` 证书(`secret.items` 指定 `tls.crt` 和 `tls.key`)或数据库密码
  - `persistentVolumeClaim`：挂载 `PVC`，需指定 `claimName`，支持 `readOnly: true` 设置只读挂载。`PVC` 绑定的 `PV` 可以是静态供给(管理员预先创建)或动态供给(通过 `StorageClass` 自动创建)
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      - name: nginx
        image: nginx:1.21
        ports:
          - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
        volumeMounts:
          - name: config
            mountPath: /etc/config
            readOnly: true
          - name: cache
            mountPath: /var/cache
          - name: tls
            mountPath: /etc/tls
            readOnly: true
          - name: data
            mountPath: /var/data
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 30
          periodSeconds: 10
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 5
          failureThreshold: 3
          successThreshold: 1
    volumes:
      - name: config
        configMap:
          name: my-config
          defaultMode: 0644
      - name: cache
        emptyDir: {}
      - name: tls
        secret:
          secretName: my-tls-secret
          items:
            - key: tls.crt
              path: cert.pem
            - key: tls.key
              path: key.pem
      - name: data
        persistentVolumeClaim:
          claimName: my-pvc
          readOnly: false
  ```

### `Service` 独有字段

- **`spec.type`**：服务类型，可选值：
  - `ClusterIP`：默认值，仅在集群内部可访问
  - `NodePort`：通过节点端口暴露，范围 `30000-32767`，但问题在于限制端口且不支持域名，用户不方便访问且会暴露服务IP，仅适用于开发测试
  - `LoadBalancer`：配合云厂商负载均衡器使用，适用于生产环境的对外提供服务
  - `ExternalName`：映射到外部域名，相当于`DNS`层面的代理，方便内部服务访问外部第三方服务
- **`spec.externalTrafficPolicy`**：外部流量策略，仅 `NodePort`/`LoadBalancer` 有效
  - `Cluster`(默认)：流量经 `kube-proxy` 转发，可跨节点负载均衡，但会做一次 `SNAT`，丢失客户端真实 `IP`
  - `Local`：流量只转发到本节点的 `Pod`，保留客户端真实 `IP`，但本节点无 `Pod` 时流量会被丢弃，需配合探针摘除不健康节点
- **`spec.ports`**：端口配置列表，每个端口包含：
  - `port`：`Service` 暴露的端口，供集群内部访问
  - `targetPort`：`Pod` 容器端口
  - `nodePort`：节点端口(仅 `NodePort` 类型，范围 `30000-32767`)
  - `protocol`：协议，默认 `TCP`
  - `name`：端口名称，多端口时必须指定
- **`spec.clusterIP`**：虚拟 `IP` 地址，设置为 `None` 时为 `Headless Service`，无头服务是指，该`Service`不会做负载均衡，在`DNS`解析它管理的`pod`的地址时会直接指向其管理的`pod IP`，通常用于有状态服务，例如数据库服务等
- 示例：

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: my-service
  spec:
    type: NodePort
    selector:
      app: nginx
    ports:
      - port: 80
        targetPort: 80
        nodePort: 30080
  ```

### `Ingress` 独有字段

- **`spec.rules`**：路由规则列表，基于域名和路径转发到不同 `Service`
  - **`host`**：域名，如 `api.example.com`（可选，不指定则匹配所有域名）
  - **`http.paths`**：路径规则列表
    - **`path`**：匹配路径，如 `/api`、`/`
    - **`pathType`**：路径匹配类型
      - `Exact`：精确匹配
      - `Prefix`：前缀匹配
      - `ImplementationSpecific`：由 `Ingress Controller` 决定
    - **`backend`**：后端服务配置
      - **`service.name`**：`Service` 名称
      - **`service.port.number`**：`Service` 端口号
- **`spec.tls`**：`TLS` 证书配置，支持 `HTTPS`
- **`spec.ingressClassName`**：指定 `Ingress Controller` 的名称，对应 `IngressClass` 资源的名称，通常不需要自己创建
  - `nginx`：`Nginx Ingress Controller` 安装时自动创建
  - `istio`：`Istio Gateway` 安装时自动创建
- 示例：

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: my-ingress
  spec:
    ingressClassName: nginx
    rules:
      - host: api.example.com
        http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: api-service
                  port:
                    number: 80
    tls:
      - hosts:
          - api.example.com
        secretName: tls-secret
  ```

### `ConfigMap` 与 `Secret` 独有字段

- **`data`**：键值对配置数据，`Secret` 的值需 `base64` 编码
- **`stringData`**：仅 `Secret` 支持，明文写入自动转 `base64`
- **`type`**：仅 `Secret` 支持，类型标识
  - `Opaque`：默认
  - `kubernetes.io/dockerconfigjson`：镜像拉取凭证
  - `kubernetes.io/tls`：`TLS` 证书
- 示例：

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: my-config
  data:
    key1: value1
    key2: value2
  ```

### `PV` 与 `PVC` 独有字段

- **供给模式**：
  - 静态供给：管理员手动创建 `PV`，用户创建 `PVC` 绑定已有 `PV`，适用于本地存储、`NFS` 等无动态供给的场景
  - 动态供给：用户创建 `PVC` 并引用 `StorageClass`，`Provisioner` 自动创建 `PV` 并绑定，云环境推荐方式
- **`spec.capacity.storage`**：仅 `PV`，存储大小，如 `10Gi`
- **`spec.accessModes`**：访问模式
  - `ReadWriteOnce`：单节点读写
  - `ReadWriteMany`：多节点读写
  - `ReadOnlyMany`：多节点只读
- **`spec.storageClassName`**：存储类名称，用于 `PV` 和 `PVC` 绑定
- **`spec.resources.requests.storage`**：仅 `PVC`，请求的存储大小
- **`spec.volumeMode`**：卷模式
  - `Filesystem`：默认值，文件系统模式
  - `Block`：块设备模式
- **`spec.persistentVolumeReclaimPolicy`**：仅 `PV`，回收策略
  - `Retain`：保留数据，需手动清理
  - `Recycle`：回收数据，已废弃
  - `Delete`：删除存储资源
- 示例(动态供给)：

  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 10Gi
    storageClassName: standard # 引用 StorageClass 动态创建 PV
  ```

### `StorageClass` 独有字段

- **`provisioner`**：存储供给器，决定使用哪种存储后端
  - 云厂商：`kubernetes.io/aws-ebs`、`kubernetes.io/gce-pd`、`kubernetes.io/azure-disk`
  - 第三方：`nfs-client`、`rbd.csi.ceph.com`（Ceph）
  - 本地存储：`kubernetes.io/no-provisioner`
- **`parameters`**：存储参数，传递给供给器的配置
  - AWS EBS：`type: gp2`（存储类型）、`zone: us-east-1a`（可用区）、`fsType: ext4`
  - GCE PD：`type: pd-standard`、`replication-type: regional-pd`
  - Ceph RBD：`pool: rbd`、`cluster: ceph`
- **`reclaimPolicy`**：回收策略，`PVC` 删除时 `PV` 的处理方式
  - `Delete`：删除 `PV` 和底层存储（默认）
  - `Retain`：保留 `PV` 和数据，需手动清理
  - `Recycle`：已废弃，执行 `rm -rf /volume/*` 后重用
- **`volumeBindingMode`**：绑定模式，控制 `PV` 创建和绑定的时机
  - `Immediate`：立即创建并绑定（默认）
  - `WaitForFirstConsumer`：延迟到第一个 `Pod` 调度后再创建，确保 `PV` 与 `Pod` 在同一可用区
- **`allowVolumeExpansion`**：是否允许动态扩容（`true`/`false`）
  - 需要存储后端支持，AWS EBS、GCE PD 支持，NFS 不支持
- **`mountOptions`**：挂载选项，传递给 `mount` 命令
  - 例如：`["hard", "nfsvers=4.1", "noatime"]`（NFS 选项）
- **`allowedTopologies`**：限制 `PV` 创建的拓扑位置（如可用区）
  - 配合 `WaitForFirstConsumer` 使用，确保存储与 `Pod` 在同一区域
- 示例：

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: standard
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
  zone: us-east-1a
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
mountOptions:
  - debug
allowedTopologies:
  - matchLabelExpressions:
      - key: topology.kubernetes.io/zone
        values:
          - us-east-1a
          - us-east-1b
```

### `Deployment` 独有字段

- **`spec.strategy`**：更新策略
  - `RollingUpdate`：滚动更新，逐步替换 `Pod`
  - `Recreate`：重建，先删除旧 `Pod` 再创建新 `Pod`
- **`spec.strategy.rollingUpdate.maxSurge`**：滚动更新时最多超出期望副本数的数量，控制更新过程中可以多创建多少个新 `Pod`，可以是数字或百分比
- **`spec.strategy.rollingUpdate.maxUnavailable`**：滚动更新时最多不可用的副本数，控制更新过程中最多有多少个旧 `Pod` 可以不可用，可以是数字或百分比，设置为`0`同时使`maxSurge>0`可保证不停机更新
- **`maxSurge` 与 `maxUnavailable` 配合使用场景**：
  - `maxSurge: 25%` + `maxUnavailable: 0`：先创建新 `Pod`，确认就绪后再删除旧 `Pod`，零停机但资源占用高
  - `maxSurge: 0` + `maxUnavailable: 25%`：先删除旧 `Pod` 再创建新 `Pod`，资源占用低但会有短暂不可用
  - `maxSurge: 1` + `maxUnavailable: 1`：平衡更新速度和资源占用，最常用配置
- 示例：

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: my-deployment
  spec:
    replicas: 3
    strategy:
      type: RollingUpdate
      rollingUpdate:
        maxSurge: 1
        maxUnavailable: 0
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
          - name: nginx
            image: nginx:1.21
  ```

### `StatefulSet` 独有字段

- **`spec.serviceName`**：关联的 `Headless Service` 名称，用于 `Pod` 网络标识
- **`spec.podManagementPolicy`**：`Pod` 管理策略
  - `OrderedReady`：默认值，按顺序逐个启动或删除 `Pod`
  - `Parallel`：并行启动或删除 `Pod`
- **`spec.updateStrategy`**：更新策略
  - `RollingUpdate`：滚动更新
  - `OnDelete`：手动删除 `Pod` 后更新
- 示例：

  ```yaml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: my-statefulset
  spec:
    serviceName: my-headless-service
    replicas: 3
    selector:
      matchLabels:
        app: nginx
    template:
      metadata:
        labels:
          app: nginx
      spec:
        containers:
          - name: nginx
            image: nginx:1.21
  ```

### `Job` 与 `CronJob` 独有字段

- **`spec.completions`**：仅 `Job`，完成次数，默认为 1
- **`spec.parallelism`**：仅 `Job`，并行 `Pod` 数量，默认为 1
- **`spec.backoffLimit`**：仅 `Job`，失败重试次数，默认为 6
- **`spec.schedule`**：仅 `CronJob`，`Cron` 表达式，如 `0 * * * *`(每小时)
- **`spec.concurrencyPolicy`**：仅 `CronJob`，并发策略
  - `Allow`：允许并发
  - `Forbid`：禁止并发
  - `Replace`：替换旧任务
- **`spec.successfulJobsHistoryLimit`**：仅 `CronJob`，保留成功 `Job` 数量，默认为 3
- **`spec.failedJobsHistoryLimit`**：仅 `CronJob`，保留失败 `Job` 数量，默认为 1
- 示例：

  ```yaml
  apiVersion: batch/v1
  kind: CronJob
  metadata:
    name: my-cronjob
  spec:
    schedule: "0 * * * *"
    concurrencyPolicy: Forbid
    successfulJobsHistoryLimit: 3
    failedJobsHistoryLimit: 1
    jobTemplate:
      spec:
        template:
          spec:
            containers:
              - name: job
                image: busybox
                command: ["echo", "hello"]
            restartPolicy: OnFailure
  ```

### 健康检查字段

- **`spec.containers[].livenessProbe`**：适用于 `Pod`、`Deployment`、`StatefulSet`、`DaemonSet`、`Job`、`CronJob`，存活探针，检测容器是否存活，失败则重启容器
- **`spec.containers[].readinessProbe`**：适用于 `Pod`、`Deployment`、`StatefulSet`、`DaemonSet`、`Job`、`CronJob`，就绪探针，检测容器是否就绪，失败则从 `Service` 摘除
- **`spec.containers[].startupProbe`**：适用于 `Pod`、`Deployment`、`StatefulSet`、`DaemonSet`、`Job`、`CronJob`，启动探针，检测容器是否启动完成，用于慢启动应用
- 探测方式(三种探针通用)：
  - `httpGet`：`HTTP` 请求探测，需指定 `path` 和 `port`
  - `tcpSocket`：`TCP` 端口探测，需指定 `port`
  - `exec`：执行命令探测，需指定 `command`
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      - name: nginx
        image: nginx:1.21
        livenessProbe:
          httpGet:
            path: /health
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: /ready
            port: 80
          initialDelaySeconds: 5
          periodSeconds: 3
        startupProbe:
          httpGet:
            path: /health
            port: 80
          failureThreshold: 30
          periodSeconds: 10
  ```

### 节点调度字段

- **`spec.nodeSelector`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，简单的节点选择器，通过标签匹配节点
- **`spec.nodeName`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，直接指定节点名称，跳过调度器
- **`spec.tolerations`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，容忍度，允许 Pod 调度到有污点的节点
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    nodeSelector:
      disktype: ssd
    tolerations:
      - key: "key1"
        operator: "Equal"
        value: "value1"
        effect: "NoSchedule"
    containers:
      - name: nginx
        image: nginx:1.21
  ```

### 镜像拉取字段

- **`spec.containers[].imagePullPolicy`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，镜像拉取策略，可选值：
  - `Always`：每次都拉取镜像
  - `IfNotPresent`(默认)：本地不存在时才拉取
  - `Never`：从不拉取，只使用本地镜像
- **`spec.imagePullSecrets`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，拉取私有镜像的凭证，存储在 Secret 中
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    imagePullSecrets:
      - name: my-registry-secret
    containers:
      - name: nginx
        image: nginx:1.21
        imagePullPolicy: IfNotPresent
  ```

### 环境变量字段

- **`spec.containers[].env`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，环境变量列表，支持直接赋值和引用 ConfigMap/Secret
- **`spec.containers[].envFrom`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，从 ConfigMap 或 Secret 批量导入环境变量
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    containers:
      - name: nginx
        image: nginx:1.21
        env:
          - name: ENV_VAR_1
            value: "value1"
          - name: DB_PASSWORD
            valueFrom:
              secretKeyRef:
                name: my-secret
                key: password
        envFrom:
          - configMapRef:
              name: my-config
  ```

### 重启策略字段

- **`spec.restartPolicy`**：适用于 Pod，重启策略，可选值：
  - `Always`(默认)：容器退出后总是重启
  - `OnFailure`：容器异常退出时重启
  - `Never`：从不重启
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    restartPolicy: OnFailure
    containers:
      - name: nginx
        image: nginx:1.21
  ```

### 服务账户字段

- **`spec.serviceAccountName`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，指定 Pod 使用的 ServiceAccount，用于访问 API Server
- **`spec.automountServiceAccountToken`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，是否自动挂载 ServiceAccount Token，默认为 `true`
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    serviceAccountName: my-sa
    automountServiceAccountToken: false
    containers:
      - name: nginx
        image: nginx:1.21
  ```

### 初始化容器字段

- **`spec.initContainers`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，初始化容器列表，在主容器启动前按顺序执行，必须全部成功退出后主容器才会启动
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    initContainers:
      - name: init-db
        image: busybox
        command: ["sh", "-c", "echo Initializing..."]
    containers:
      - name: nginx
        image: nginx:1.21
  ```

### 优先级字段

- **`spec.priorityClassName`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，优先级类名，高优先级 Pod 抢占低优先级 Pod 资源
- **`spec.priority`**：适用于 Pod、Deployment、StatefulSet、DaemonSet、Job、CronJob，优先级数值，范围 1000000000 到 -2147483648
- 示例：

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-pod
  spec:
    priorityClassName: high-priority
    containers:
      - name: nginx
        image: nginx:1.21
  ```

## `Istio`

### 简介

- **`Istio`** 是最流行的**服务网格**(`Service Mesh`)实现，将流量治理、安全、可观测性等能力从业务代码中下沉到基础设施层，业务容器无感知
- 架构分为两层：
  - **控制面**(`istiod`)：负责配置分发(`Pilot`)、证书签发(`CA`)，是集群中的单一大脑
  - **数据面**(`Envoy`)：以 **`sidecar`** 形式注入到每个 `Pod`，通过 `iptables` 透明劫持出入流量，所有经过 `Pod` 的请求都会先经过 `Envoy` 代理
- 核心能力：
  - **流量治理**：按域名/路径/权重路由、灰度发布、重试、超时、熔断、会话保持
  - **安全**：服务间自动 `mTLS`、基于身份的访问控制(`AuthorizationPolicy`)
  - **可观测性**：自动生成访问日志、指标(`Metrics`)与调用链(`Tracing`)
- `Sidecar` 注入方式：
  - 命名空间标签：`kubectl label namespace [ns] istio-injection=enabled`，该命名空间下所有新建 `Pod` 都会被注入
  - `Pod` 级别：在 `spec.template.metadata.annotations` 中设置 `sidecar.istio.io/inject: "true"`(或 `"false"` 排除)
  - 注入发生在 `Pod` 创建时，已运行的 `Pod` 需要重建才能生效

### 安装

- 推荐使用 `helm` 安装，分为两个 `chart`，依赖顺序不能颠倒：

  ```bash
  helm repo add istio https://istio-release.storage.googleapis.com/charts
  kubectl create namespace istio-system
  helm install istio-base istio/base -n istio-system
  helm install istiod istio/istiod -n istio-system --wait
  ```

- `base` `chart` 定义了 `CRD` 与集群级别的角色等前置资源，`istiod` `chart` 部署控制面本身
- 常用配置(通过 `istiod` 的 `values` 覆盖 `meshConfig`)：

  ```yaml
  istiod:
    meshConfig:
      accessLogFile: /dev/stdout # 开启访问日志，输出到容器标准输出
      outboundTrafficPolicy:
        mode: ALLOW_ANY # 未注册外部服务的出站流量默认放行(默认 REGISTRY_ONLY 会拒绝)
    global:
      proxy:
        resources: # sidecar(Envoy)的资源限制，避免占用过多业务资源
          limits:
            cpu: 1000m
            memory: 1Gi
  ```

- `istioctl` 是 `Istio` 的命令行工具，常用命令：
  - `istioctl proxy-status`：查看所有 `sidecar` 与 `istiod` 的同步状态
  - `istioctl proxy-config route [pod]`：查看指定 `sidecar` 的实际路由表，排查路由不生效问题
  - `istioctl analyze`：静态分析集群内 `Istio` 配置的潜在冲突

### `Gateway`

- 入口网关，控制**哪些流量可以进入网格**，相当于传统架构的负载均衡器 + `Nginx` 的 `server` 块
- **`spec.selector`**：通过标签匹配承载流量的 `ingress gateway` `Pod`(通常部署在 `istio-system`)，一个网关 `Pod` 可被多个 `Gateway` 资源复用
- **`spec.servers`**：服务器列表，每个包含：
  - `port`：监听端口(`number`、`name`、`protocol`)
  - `hosts`：监听的域名列表
  - `tls`：`TLS` 配置，`httpsRedirect: true` 将 `HTTP` 重定向到 `HTTPS`；`mode: SIMPLE` 终止 `TLS`，证书通过 `credentialName` 引用 `Secret`
- 示例：

  ```yaml
  apiVersion: networking.istio.io/v1
  kind: Gateway
  metadata:
    name: my-gateway
  spec:
    selector:
      istio: ingress # 匹配 ingress gateway Pod 上的标签
    servers:
      - port:
          number: 80
          name: http
          protocol: HTTP
        tls:
          httpsRedirect: false
        hosts:
          - api.example.com
      - port:
          number: 443
          name: https
          protocol: HTTPS
        tls:
          mode: SIMPLE
          credentialName: example-tls # 引用同命名空间的 kubernetes.io/tls 类型 Secret
        hosts:
          - api.example.com
  ```

### `VirtualService`

- 路由规则，控制**流量在网格内如何转发**，相当于 `Nginx` 的 `location` 块，需同时被 `Gateway`(入口)或 `Service`(内部)引用
- **`spec.hosts`**：匹配的目标域名，网关场景为外部域名，内部场景为 `Service` 的 `FQDN`
- **`spec.gateways`**：绑定的 `Gateway` 名称，内部流量路由可省略(默认 `mesh`)
- **`spec.http`**：`HTTP` 路由规则列表，按顺序首次匹配生效：
  - `match`：匹配条件(`uri.prefix`、`headers`、`port` 等)
  - `route`：转发目标，`destination.host` 指向 `Service`，`destination.port.number` 指向端口
  - `weight`：多目标按权重分流，灰度发布的基础
  - `rewrite`、`timeout`、`retries`、`fault`(`注入延迟/中止做故障演练`)
- **`spec.tcp`**：`TCP` 路由，适用于非 `HTTP` 协议(如数据库)的端口转发
- 示例：

  ```yaml
  apiVersion: networking.istio.io/v1
  kind: VirtualService
  metadata:
    name: my-vs
  spec:
    hosts:
      - api.example.com
    gateways:
      - my-gateway
    http:
      - match:
          - uri:
              prefix: /v2
        route:
          - destination:
              host: api-v2.default.svc.cluster.local
              port:
                number: 80
      - route: # 兜底规则，其余流量走稳定版本
          - destination:
              host: api-v1.default.svc.cluster.local
              port:
                number: 80
            weight: 90
          - destination:
              host: canary.default.svc.cluster.local
              port:
                number: 80
            weight: 10 # 10% 灰度
  ```

### `DestinationRule`

- 目标策略，定义**流量到达目标 `Service` 后的转发策略**，与 `VirtualService` 配合使用
- **`spec.host`**：作用的 `Service`(需与 `VirtualService` 的 `route.destination.host` 一致)
- **`spec.trafficPolicy`**：流量策略：
  - `loadBalancer`：负载均衡算法，`consistentHash`(基于源 `IP`/指定 `Header` 的会话保持)、`leastRequest`、`random`
  - `connectionPool`：连接池限制(`tcp`/`http` 的最大连接数、并发请求数)
  - `outlierDetection`：熔断策略，连续错误次数(`consecutive5xxErrors`)达到阈值后摘除实例
  - `tls`：`ISTIO_MUTUAL` 开启双向 `mTLS`
- **`spec.subsets`**：将同一 `Service` 按 `Pod` 标签划分为不同子集(如 `v1`、`v2`)，供 `VirtualService` 按 `subset` 精确路由
- 示例：

  ```yaml
  apiVersion: networking.istio.io/v1
  kind: DestinationRule
  metadata:
    name: my-dr
  spec:
    host: api.default.svc.cluster.local
    trafficPolicy:
      loadBalancer:
        consistentHash:
          useSourceIp: true # 基于来源 IP 的会话保持
    subsets:
      - name: stable
        labels:
          version: stable
      - name: canary
        labels:
          version: canary
  ```

### `ServiceEntry`

- 将**网格外部的服务注册进网格**，让外部服务也能享受 `Istio` 的路由、熔断与 `mTLS` 能力，是最能体现“替代自建代理”的资源
- 典型场景：将外部 `IP`/域名伪装成集群内的 `Service`，内部服务直接通过 `Service` 域名访问外部数据库或第三方 `API`
- **`spec.hosts`**：对外暴露的虚拟域名，通常用 `[name].[namespace].svc.cluster.local` 让内部访问无感知
- **`spec.addresses`**：为该条目分配的虚拟 `IP`(可选)
- **`spec.ports`**：端口与协议(`HTTP`、`TCP`、`MONGO` 等)
- **`spec.resolution`**：地址解析方式，`STATIC`(端点 `IP` 固定)、`DNS`(按域名解析)
- **`spec.location`**：`MESH_EXTERNAL`(外部服务)或 `MESH_INTERNAL`
- **`spec.endpoints`**：实际后端端点列表(`address` + `ports`)
- 示例(将外部 `MongoDB` 伪装成集群内 `Service`)：

  ```yaml
  apiVersion: networking.istio.io/v1
  kind: ServiceEntry
  metadata:
    name: external-mongo
  spec:
    hosts:
      - external-mongo.default.svc.cluster.local # 内部用该域名访问
    ports:
      - number: 27017
        name: mongo
        protocol: MONGO
    resolution: STATIC
    location: MESH_EXTERNAL
    endpoints:
      - address: 10.0.0.10 # 外部 MongoDB 实际地址
        ports:
          mongo: 27017
  ```

### `AuthorizationPolicy`

- 访问控制策略，基于**工作负载身份**(而非 `IP`)决定谁可以访问谁，默认全部放行，一旦定义则未匹配的请求全部拒绝
- **`spec.selector`**：策略作用的 `Pod` 标签
- **`spec.action`**：`ALLOW`(白名单)、`DENY`(黑名单)、`CUSTOM`
- **`spec.rules[].from.source`**：请求来源，`ipBlocks`/`remoteIpBlocks`(`IP` 黑白名单)、`principals`(对端服务身份)、`namespaces`
- **`spec.rules[].to.operation`**：请求目标，`methods`、`paths`、`ports`
- 示例(网关层 `IP` 黑名单)：

  ```yaml
  apiVersion: security.istio.io/v1
  kind: AuthorizationPolicy
  metadata:
    name: ingress-policy
    namespace: istio-system
  spec:
    selector:
      matchLabels:
        istio: ingress
    action: DENY
    rules:
      - from:
          - source:
              remoteIpBlocks:
                - 1.2.3.4/32
  ```

### 证书管理

- `Gateway` 的 `HTTPS` 证书通常配合 **`cert-manager`** 自动签发与续期，形成 `ClusterIssuer` → `Certificate` → `Secret`(`kubernetes.io/tls`) → `Gateway.credentialName` 的链路
- `HTTP-01` 校验需要真实公网入口，通配符域名(`*.example.com`)必须使用 `DNS-01` 校验，此时需要一个 `DNS` 服务商的 `API` 凭证 `Secret`(如 `Cloudflare` 的 `Api Token`)
- 示例(`DNS-01` 通配符证书)：

  ```yaml
  apiVersion: cert-manager.io/v1
  kind: ClusterIssuer
  metadata:
    name: letsencrypt-dns
  spec:
    acme:
      server: https://acme-v02.api.letsencrypt.org/directory
      email: admin@example.com
      privateKeySecretRef:
        name: letsencrypt-key
      solvers:
        - dns01:
            cloudflare:
              apiTokenSecretRef:
                name: cloudflare-token
                key: api-token
  ```

## `Charts` 实践

以上能力在生产实践中通常都会沉淀为一个 `helm` 仓库，并按“基础设施 → 应用模板 → 代理组件”分层组织，这也是大家的普遍做法

### 总览

- **基础设施层**：
  - `base`：集群初始化，创建基础命名空间与镜像仓库凭证(`registry secret`)，并托管 `cert-manager` 依赖，作为其他所有 `chart` 的前置
  - `istio`：以 `umbrella chart` 封装官方 `base` + `istiod` 依赖，统一注入 `meshConfig`(访问日志格式、`ALLOW_ANY` 出站策略、`sidecar` 资源限制)，并创建带 `istio-injection` 标签的 `istio-system` 命名空间
  - `ingress`：封装官方 `gateway chart`(部署 `ingress gateway` `Pod`)，配套 `ClusterIssuer`、`Cloudflare DNS` 凭证、`AuthorizationPolicy` `IP` 黑名单等模板
  - `monitoring`：聚合 `kube-prometheus-stack` + `loki` + `promtail` + `grafana` + `grafana-mcp` 依赖，并模板化大量 `Grafana` 告警规则(证书过期、`Pod` 状态、`CPU`/内存、`JVM`、`WebSocket` 等)与数据源配置
- **应用模板层**：
  - `common`：最简无状态应用模板，仅 `Deployment` + `Service` + 镜像凭证
  - `spring-app`：功能最全的 `Spring` 应用模板，见下文
  - `spring-job`：一次性/定时任务模板，仅 `Job` + 密钥 + 存储，无 `Service` 与路由
- **代理组件层**(纯 `Istio` 资源，不包含任何业务镜像)：
  - `simple-proxy`：`L7` 反向代理，`Gateway` + `VirtualService` + `ServiceEntry` + `DestinationRule` 四件套，把外部域名流量转发到一组上游，可选 `consistentHash` 会话保持，替代自建 `Nginx`
  - `mongo-proxy`：`L4` 代理，`ServiceEntry` 将外部 `MongoDB` `IP` 注册为集群内 `Service`，`VirtualService` 的 `tcp` 规则把网关 `443` 端口转发到 `27017`
  - `mongodb`：`bitnami mongodb` 依赖 + `mongo-express` 管理界面，通过 `Istio` 网关暴露
- **环境特定层**：
  - `github-actions-cache-server`：自托管 `GitHub Actions` 缓存服务(`PVC` + `HPA` + `Ingress`)
  - `aliyun-pod-ip`：阿里云 `ack-extend-network-controller` 依赖，管理 `Pod` 弹性 `IP`

### 应用模板的设计要点

以 `spring-app` 为例，一个生产级应用 `chart` 通常包含：

- **工作负载**：`Deployment`(探针、资源限制)+ `HPA`(`CPU` 利用率自动扩缩)+ `PDB`(保证节点维护时的最小可用副本数)
- **流量入口**：`Service` + `Istio` 的 `Gateway`/`DestinationRule`(可选会话保持)，证书由 `cert-manager` 自动签发
- **安全**：
  - `ServiceAccount` + 最小化 `RBAC`，关闭 `automountServiceAccountToken`
  - `NetworkPolicy` 默认拒绝出站，仅放行 `DNS`，并显式屏蔽云厂商 `metadata` 网段(`169.254.0.0/16`、`100.64.0.0/10`)，防止应用被 `SSRF` 后窃取云凭证
  - 数据库密钥、`Cloudflare` `Token` 等以 `Secret` 模板管理
- **多云适配**：同一 `chart` 通过 `values` 切换对象存储挂载方案(`oss`/`gcs`/`cos`)与 `StorageClass`(按云厂商提供不同存储模板)
- **环境隔离**：默认注入 `nodeSelector: kingboat.io/environment={{ .Release.Namespace }}` 与对应容忍度，让命名空间与节点环境一一对应，用户可整体覆盖
- **生命周期钩子**：`post-install`/`post-upgrade` `Job` 执行数据库迁移(`schema` 变更)等前置任务

### 沉淀的最佳实践

- **`umbrella chart` 封装上游依赖**：不直接部署第三方 `chart`，而是以依赖(`dependencies` + `condition` 开关)的方式包装，统一版本与定制，升级时只需改一处版本号
- **版本统一为 `0.0.0`**：由 `CI` 在发布时统一注入版本号，避免手工维护，可回溯到具体 `commit`
- **用 `Istio` 资源替代传统代理**：外部 `MySQL`/`MongoDB`/第三方 `API` 的访问入口全部用 `ServiceEntry` + `VirtualService` 建模，业务方看到的是稳定的集群内域名，底层地址变更只需改 `chart` 的 `values`
- **可观测性即代码**：`Grafana` 数据源、告警规则、通知渠道全部模板化在 `monitoring` `chart` 中，随应用一起评审与发布
