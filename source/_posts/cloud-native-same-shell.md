---
title: "云原生项目都长同一张脸:从 cert-manager 源码看那些'用的时候看不见'的概念"
date: 2026-08-19
categories: [CloudNative, Kubernetes]
tags: [kubernetes, cert-manager, controller, 源码阅读]
---

平时使用 cert-manager,体验是这样的:apply 一个 `Certificate` YAML,几分钟后 `kubectl get secret` 里就多了一个装着 TLS 证书的 Secret。中间发生了什么?完全黑盒。

最近读了 cert-manager 的源码,发现一件更有意思的事:**这个"黑盒"内部的结构,和 ExternalDNS、Argo CD、Velero 这些项目几乎是同一张脸**。也就是说,云原生世界里存在一套隐藏的、统一的结构——用户永远看不见它,只有深入到源码层面才会撞上它。这篇文章就以 cert-manager 为例,把这张"脸"描出来。

<!-- more -->

## 一、用户视角:看不见的三个黑箱

先明确"看不见"到底指什么。日常使用一个 operator 类项目时,有三个东西被抽象掉了:

1. **谁在干活**。你以为在跟 `Certificate` 这个资源打交道,实际上是某个容器里的某个 Go 进程在 watch 它。用户不需要知道 `cmd/` 下有 `controller`、`webhook`、`cainjector` 三个二进制,更不需要知道 cainjector 是干嘛的。
2. **状态存在哪**。YAML 里你只写了三行 spec(`secretName`、`dnsNames`、`issuerRef`),但 reconcile 过程中 status 里被写入了 revision、conditions、lastFailureTime……这些字段是控制器之间**互相通信的机制**,和用户意图毫无关系。
3. **决策怎么做的**。证书什么时候该续期?为什么失败了在重试?这些逻辑藏在 policies 链和指数退避的设计文档里,对外只表现为一个 `Ready` condition。

这三样东西——控制器进程、status 通信、内部决策——恰恰是所有云原生项目**共享**的部分。

## 二、那张共享的脸:七个部件

打开 cert-manager 仓库,按目录对照:

```
cmd/            每个二进制一个入口(controller / webhook / cainjector)
pkg/apis/       CRD 类型定义 + 生成的 deepcopy / client
pkg/controller/ 一堆控制器,每个是一个 watch-reconcile 循环
internal/       非公开代码,新控制器都在这里
test/           unit / integration(envtest) / e2e(kind)
make/           共享 Makefile 逻辑(从 makefile-modules 仓库引入)
hack/           代码生成脚本
design/         设计文档
```

逐个说这些部件为什么是"通用件":

### 1. Controller 模式:watch → workqueue → reconcile

这是整个生态的内核。所有项目都是同一个循环:

```
informer 监听资源变化 → 变化塞进 workqueue → worker 取出 → reconcile → patch status
```

关键性质是 **level-based 而非 edge-based**:控制器不关心"发生了什么事件",只关心"当前状态和期望状态差多少"。所以 reconcile 必须幂等——同一项处理一百次结果相同。这个约束解释了源码里大量看似啰嗦的防御性代码。

cert-manager 里 `pkg/controller/certificates/` 甚至把这个模式用到了极致:证书管理被拆成六个各管一件事的控制器(`trigger` 判断要不要重发、`issuing` 编排、`requestmanager` 维护请求对象、`keymanager` 管私钥轮换……),它们互相不通信,全靠读写 `Certificate` 的 status 字段协作。**资源对象本身就是消息总线**——这是读这类源码前需要换掉的最大的脑子。

### 2. client-go + informer 缓存

所有项目共用 `k8s.io/client-go`。informer 在本地内存里维护一份资源的缓存(list-watch),控制器读资源走缓存而不打 API server。所以源码里到处是 `lister`(读缓存)和 `client`(写)两个对象并存——第一次见会觉得分裂,明白缓存模型后就成了本能。

### 3. CRD + 代码生成

`pkg/apis/certmanager/v1/` 里的类型定义带着 JSON tag 和 kubebuilder 注解,deepcopy、client、informer 全部生成(`zz_generated*.go`,文件头写着禁止手改)。改一个字段的真实流程是:改类型 → 跑 codegen → CRD yaml 重新生成 → 提交一大坨 diff。**API 即协议**,这是和普通 Go 仓库最大的工程差异。

### 4. 入口:Cobra + 多二进制

`cmd/` 下每个目录一个 main,全部用 cobra/pflag 组装命令行参数。operator 本体、webhook、辅助工具分开部署成不同的 Deployment——为什么 webhook 必须独立?因为 API server 调用它时主控制器可能还没起来,启动顺序决定了部署形态。这种"部署拓扑源于架构约束"的细节,只有读源码+部署清单才能拼出来。

### 5. 准入 Webhook

有 CRD 的项目几乎都有一个 validating webhook。cert-manager 的 `internal/webhook/` 用的是 `k8s.io/apiserver` 库而不是什么 web 框架——这类项目里没有 gin/echo,HTTP 服务是按 apiserver 的姿势写的。

### 6. 可观测性三件套

Prometheus 指标端点、logr 结构化日志、OpenTelemetry tracing。每家都有,只是指标名字不同。读源码时按 metric 名反查采集点,是定位代码路径的捷径。

### 7. 工程文化层

golangci-lint、每个文件头的 Apache license、DCO 签核、kind 起本地集群跑 e2e、`make verify` 作为 CI 门禁。换个云原生项目,这些名字一个不变。

## 三、脸下面的内脏:真正各不相同的部分

如果只有壳,这些项目就成了复制粘贴。它们的价值全在"内脏"里,而内脏**不可迁移**:

- **领域状态机**。cert-manager 的灵魂是 `Certificate → CertificateRequest → (Order → Challenge) → Secret` 这条资源管道;Argo CD 的灵魂是 git 与集群的同步状态机。这是每个项目唯一值得单独写深度笔记的部分。
- **插件机制**。cert-manager 的 issuer 注册表(`RegisterIssuer` + 工厂模式,`pkg/issuer/factory.go`)、ACME DNS provider 的 webhook 插件体系,都是各家自己的发明,没有标准。
- **框架代际**。cert-manager 是"老派"项目:很多控制器手写 workqueue(`pkg/controller`),新代码才迁往 controller-runtime(`internal/controller`)。而 kubebuilder 脚手架生成的新项目几乎纯 controller-runtime——你会看到 `Reconcile(ctx, req)` 签名和 `kubebuilder:rbac` 注解。心智模型一样,代码形状不同。

所以准确的说法是:**壳提供 60% 的可复用认知,剩下 40% 是每个项目的存在理由。**

## 四、验证方法

这套"壳"理论可以用一个半天实验检验:随便 clone 一个不熟悉的 operator 项目(比如 external-dns),只回答三个问题——

1. 入口在哪个 `main.go`?
2. 它 watch 哪些资源?
3. reconcile 往哪些 status 字段写东西?

如果半小时内能定位全部三个答案,说明壳已经内化,以后读任何云原生项目都可以直接跳过外壳、直奔领域状态机。

## 结语

云原生的抽象是分层的:用户层看到"apply YAML 拿到证书",运维层看到"几个 Deployment 在跑",只有源码层能看到那张所有项目共享的、由 controller 模式和 client-go 构成的脸。日常使用永远不需要这些知识——但一旦要 debug 一个卡在 `Ready=False` 的证书、或者给项目贡献代码,这张脸就是你唯一的地图。而学一次,到处能用,大概是读这类"无聊基础设施源码"最划算的地方。

> 源码版本:cert-manager master(@cd21f71)。
