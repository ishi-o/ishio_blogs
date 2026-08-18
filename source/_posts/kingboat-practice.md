---
title: "Note: 实习项目-游戏后端"
categories:
  - Programming
  - System design
tags:
  - design
  - trick
mathjax: false
date: 2026-04-27T10:09:54.000Z
---

<!-- placeholder -->
<!-- more -->

# 游戏后端开发

一个多游戏复用的社交游戏后端：同一个 `Gradle` 多模块代码库，按 API、任务、运营后台和业务域拆成多个可独立启动的应用；每个正式部署通常绑定一个产品配置，另有 `all-in-one` profile 用于本地或特定环境下在同一个进程里按请求切换产品。产品级配置负责隔离数据库连接、RabbitMQ 队列、Redis 命名空间和文件目录，而不是让业务代码到处写 `if (product == ...)`。

```yaml
# 一份 per-product 配置 = 一个游戏的全部连接信息（AppConfiguration record 承接）
app:
  products:
    - id: game-a
      name: Game A
      external-url: https://...
      mongodb-uri: mongodb://...
      redis-uri: redis://...
      redis-discrepancy: game-a-prod
      rabbitmq:
        { host: ..., settlement-queue: q.game-a.prod.alliance-settlement }
      storage: { type: LOCAL, location: /data/game-a, cdn-url: https://... }
```

上面的 YAML 是为了说明配置边界的简化示意。仓库中的真实绑定入口是 `AppConfiguration` 的 `@ConfigurationProperties(prefix = "app")`，`ProductInfo` 里保存每个产品的 `mongodbUri`、`redisUri`、`redisDiscrepancy`、`Storage`、APNs 和可选的 RabbitMQ 配置。RabbitMQ 的队列名没有默认值：启用 broker 后却没有配置隔离队列，应用会在启动阶段失败，避免多个产品误消费同一个队列。

这里要区分两种部署方式。普通 API/ops 实例启动时只使用一套数据源，连接池和线程模型都比较简单；`all-in-one` 才会启用 `ProductContextFilter`，从 `X-Product` 请求头或 `productId` 参数设置 `ThreadLocal`，Mongo、Redis、Rabbit、APNs 和存储服务按上下文懒加载并缓存。上下文必须在请求结束时清理，否则 Servlet 线程复用会造成串产品，这是为什么过滤器使用 `try-with-resources` 包住 `ProductContext.setProduct()`。

从面试角度，这个设计的核心不是“一个进程连很多数据库”，而是把多租户隔离放到基础设施适配层：业务服务只拿 `MongoTemplate`、`RedisTemplate` 或 `StorageService`，不感知当前连接究竟属于哪个产品；同时生产环境可以用“一个产品一个 Pod”降低上下文误用风险，`all-in-one` 只承担开发便利和少量特殊部署需求。

## 技术选型

### 数据库

- 项目采用`Mongodb`数据库，与传统关系型数据库相比，它的优点在于：
  - 它拥有灵活的 `schema`，或者说它没有 `schema`，针对游戏后端这类业务以及客户端需求时常变化，特别是需要支持多个游戏的后端而言，需要非常灵活的 `schema`
    - 新增字段无需执行 `ALTER TABLE`，也就是不需要维护`schema`，同时也不需要数据迁移
    - 向后兼容性好，旧数据缺少新字段不影响读取
  - 拥有灵活的模式带来的好处不仅是灵活添加或删除字段，也意味着原本需要存储于多个关系表的数据可以内嵌存储在同一个 `MongoDB` 的集合里，减少许多关联查询；但内嵌并不保证更省存储，重复字段、文档更新放大和索引大小都要付成本。比如 `UserEvent` 的奖励结构不尽相同，但类型数量有限，因此把类型化附件放在同一个集合里，而不是为每种奖励单独建表
  - 针对非结构化的数据比如嵌套 `Map`，写入速度更快
    - `MongoDB` 天然支持嵌套对象，不用拆分成多表
    - 同时也适合日志、数据埋点、用户行为等数据格式不固定的场景
  - 天然支持水平扩展，适合高并发的写入场景，我们使用阿里云的托管服务，使用副本集保证高可用，同时对于大数据量的集合可以在控制台配置分片
- 那么代价是什么？实际上代价并不大，最大的问题在于多文档读写
  - `MongoDB` 原生支持单文档读写的原子性和隔离性，且大多数场景都是单文档的读写，因为在设计上一个游戏客户端只能修改单个用户数据
  - 对于多文档读写的场景，可以使用多文档事务，虽然性能并不强，但场景不多且调用不频繁，比如用户账号绑定、数据迁移、活动结算会涉及多文档
    - 注意多文档事务要求副本集部署：本地开发的容器也以**单节点副本集**启动（`mongod --replSet` + 健康检查里 `rs.initiate`），否则事务直接不可用；事务管理器按 `profile` 注册，本地一体式开发模式跳过

      ```yaml
      # docker-compose: 多文档事务要求副本集，本地也要单节点 replSet
      command: ["mongod", "--replSet", "rs0"]
      healthcheck:
        test: ["CMD", "mongosh", "--eval", "rs.initiate()"]
      ```

  - 对于少部分需要解决并发冲突单文档读写的场景，我们使用基于时间戳的校验，比如客户端数据同步，这部分数据是**在客户端计算并存储的**，用户可能是在断网的情况下游玩的，那在恢复网络之后，此时由客户端发起的同步数据的请求中，我们不能认为后来的请求就是最新的数据，而是进行时间戳的校验（见“客户端数据模型与同步”）

- 一些配套实践：
  - 同一份文档用 `JsonViews` 区分对客户端与运营后台的序列化视图，避免给客户端泄漏运营字段：

    ```java
    public interface JsonViews {
      interface Public {}      // 客户端可见
      interface Operational {} // 仅运营后台
      interface Hidden {}
    }

    @JsonView(JsonViews.Public.class)
    private String deviceId;
    @JsonView(JsonViews.Operational.class)
    private boolean bot; // 机器人标志绝不给客户端
    ```

  - **配置即数据**：`KeyValue`/`ObjectConfig` 集合存运营可在线编辑的 `JSON` 配置（匹配规则、积分规则、机器人参数等），每个 `key` 有代码里的枚举作为单一事实来源，加载时统一回退到代码默认值——改数值不需要发版：

    ```java
    public enum Key {
      ARENA_MATCHMAKING_CONFIG, ARENA_POINT_RULES_CONFIG,
      GAME_ITEM_FRAUD_THRESHOLDS_CONFIG, PUSH_RATE_LIMIT_CONFIG /* ... */
    }

    // 加载：配置文档 → 反序列化 → 任一步失败回退代码默认值
    public ArenaBotConfig loadBotConfig() {
      return objectConfigRepo.findByKeyAndName(Key.ARENA_BOT_CONFIG, "default")
          .map(doc -> parse(doc, ArenaBotConfig.class))
          .orElse(ArenaBotConfig.defaults());
    }
    ```

  - 排行类查询驱动索引设计：用户集合上大量 `(指标, createdDate)` 复合索引，排名查询模式决定了 `schema`

#### 从模型和访问路径看，为什么它适合这个项目

MongoDB 的选择最终落在“聚合边界”上，而不是落在数据库宣传页上的“灵活 schema”。例如 `UserEvent` 是用户收件箱：事件类型固定，但奖励、联盟战报、竞技场结果等附件结构不同；代码把这些附件定义成类型化 `record`，放在同一个集合里，再用部分索引约束真正需要唯一的事件。`ClientCustomData` 则更激进：每个 `(userId, key)` 只有一份不透明 JSON，服务端只抽取少数需要做鉴权、排行榜或风控的字段。

活动数据也是文档模型的真实用例。`GenericActivityUserData.data` 是 `Map<String, Object>`，排序字段可能是 `data.score`、`data.mergeScore` 或运营后来定义的字段；如果强行做关系表，需要为每个活动的每个指标建立结构或 EAV 表，查询、索引和类型转换都会变复杂。这里的代价是排序字段必须经过配置校验，不能把任意客户端传入的字段名直接拼进 Mongo 查询。

#### 单文档原子性、多文档事务和“最终一致”要分开讲

用户存档、点赞计数、节点状态这类操作尽量让一个文档承担一个业务原子边界：`$inc`、条件 `updateFirst`、`findAndModify` 比“查出 Java 对象后 save”更不容易覆盖并发更新。确实要改多份文档时，普通部署的 DAO 会注册 `MongoTransactionManager`；但 `all-in-one` profile 明确跳过这个 transaction manager，原因是多个产品连接是动态路由的，不能把一个事务错误地跨到另一个产品连接上。

活动结算还刻意没有把整个大榜单包进一个 Mongo 事务。源码注释直接说明：大榜单可能超过 `transactionLifetimeLimitSeconds`，而“创建奖励事件”和“标记进度已结算”都设计成可重试、幂等的步骤，因此采用小步骤重跑自愈。面试时应把它称为“数据库事务 + 幂等补偿的组合”，不要笼统说成“所有结算都由分布式事务保证”。

关系型数据库仍然有明确的优势：支付账务、强约束库存、复杂联表报表或需要大量 `JOIN` 的数据更适合 PostgreSQL/MySQL。这个项目没有选择它，主要是因为核心读写以用户、活动实例和事件文档为中心，且客户端数据结构变化快；不是因为 MongoDB 在所有场景都更快。

### 索引实践

- **唯一索引当并发守卫用**：对“只应发生一次”的写入（活动参与、回调处理、奖励发放），与其在代码里先查后插（查和插之间存在竞态窗口），不如直接建 `(activityId, userId, cycle)` 这类唯一复合索引——并发提交的重复文档在数据库层被拒绝，应用捕获 `DuplicateKeyException` 当幂等命中处理；查重逻辑和竞态窗口一起消失：

  ```java
  verifySignature(rawBody); // 推荐先验签，避免攻击者抢占幂等键
  try {
    callbackRepo.insert(CallbackRecord.of(outTradeNo, rawBody)); // outTradeNo 唯一
  } catch (DuplicateKeyException e) {
    return "success"; // 已处理过 → 幂等命中，直接应答
  }
  // ... 映射订单、创建交易
  ```

  这是推荐的安全顺序，不是所有渠道当前实现的真实顺序；EezPay 的现状和风险在“支付系统”一节单独说明。

- **部分索引（partial index）做轻量幂等**：机器人攻击日志在 `(playerUserId, seasonId, shadow, status)` 上建唯一**部分**索引，过滤条件 `status = SCHEDULED`——只对“待执行”的行强制唯一，已完成的历史记录不占唯一性空间，代码里的重挂/去重判断全部省掉：

  ```js
  db.arena_bot_attack_logs.createIndex(
    { playerUserId: 1, seasonId: 1, shadow: 1, status: 1 },
    { unique: true, partialFilterExpression: { status: "SCHEDULED" } },
  );
  // 重新武装时直接 insert：SCHEDULED 已存在则撞索引 = 已武装，跳过
  ```

- **配置驱动的动态索引**：通用活动的排序字段是运营配置的（`data.*` 路径），注解 `@CompoundIndex` 写不了运行期才知道的字段——于是在启动时为**每个活动动态创建**两个部分索引：等值前缀（`group`/`userId` + `cycle`）+ 该活动配置的排序字段 + `dateKey`，过滤条件限定 `activityId = 本活动`：

  ```java
  // 启动时对每个活动执行：索引 = 等值前缀 + 运营配置的排序字段 + dateKey
  var sort = activityService.buildSort(activity.getSortFields()); // data.score DESC...
  var index = new Index().on("cycle", ASC).on("userId", ASC);
  sort.forEach(o -> index.on(o.getProperty(), o.getDirection())); // 运行期才知道
  index.on("dateKey", ASC)
      .named("activity_daily_user_data_sort_" + activityId + "_user_sort_idx")
      .partial(PartialIndexFilter.of(Criteria.where("activityId").is(activityId)));
  if (!existingIndexNames.contains(indexName)) indexOps.createIndex(index); // 幂等
  ```

  - 每个活动一小片索引，而不是全体活动共用一个大复合索引；活动下线后理论上可以按活动清理这片索引，但当前启动器只负责 create-if-missing，不会自动删除旧索引
  - 索引名带活动 `id` 前缀，按名字 create-if-missing，启动器天然幂等
  - 建索引的 `CommandLineRunner` 挂 `@Profile("!all-in-one")`：正常部署在启动时创建，本地一体式开发跳过、交给 `auto-index-creation` 兜底
  - 代价是索引只增不删：活动中途改排序字段会生成新索引，旧索引留在那成了写放大——接受它，因为活动实例本身有快照语义，排序字段的变更频率很低

#### 索引不是“加上就快”，而是把业务不变量写进数据库

例如账号绑定的 `UserThirdPartyMapping` 有两条不同的唯一约束：同一用户同一身份类型只能有一条映射；同一种身份标识只能有一个 `root` 映射，非 root peer 可以共享它。它们分别用普通复合唯一索引和带 `root = true` 的部分唯一索引表达。这样并发绑定时，应用不必相信“查询结果仍然最新”，而是捕获 `DuplicateKeyException`，再判断是精确重复、已被别人抢先绑定，还是数据完整性异常。

通用活动还有一个容易被忽略的细节：`GenericActivityUserProgress` 的唯一索引是 `(activityId, userId, instanceCycle)`，而 `GenericActivityUserData` 的唯一索引是 `(activityId, userId, cycle)`。点赞接口必须按 `cycle` 查询，不能按 `instanceCycle` 查询后再 upsert，否则两个并发的首次点赞都可能看不到目标文档并同时尝试插入。即使如此，源码仍捕获一次 `DuplicateKeyException` 并重试 `$inc`，因为“唯一索引作为最后一道闸门”比假设并发不会发生更可靠。

动态活动日榜的索引则体现了“查询模式决定索引”：启动时读取活动的排序字段，分别生成按 `group + cycle + sortFields + dateKey` 和 `userId + cycle + sortFields + dateKey` 的索引，只对当前活动 `activityId` 做 partial filter。创建按索引名幂等，但不会自动删除旧排序索引，因此运营修改排序字段后要配套查看索引数量和写放大；这不是免费的动态能力。

### 缓存：`Redis`

- `key` 统一经 `RedisKeyProvider` 加产品配置里的 `discrepancy` 前缀拼接，多产品、多环境在共用一套 `Redis` 时依靠不同 discrepancy 隔离：

  ```java
  // 具体前缀由 app.products[].redis-discrepancy 配置，不在业务代码里硬编码
  keyProvider.withParts("lock", "season-matching", seasonId);
  // => "<configured-discrepancy>:lock:season-matching:123"
  ```

- 承担的角色：
  - `Spring Cache`：`JSON` 序列化 + 受限的多态类型白名单（只允许自家包和 `JDK` 类型，防反序列化攻击），短 `TTL`
  - `ZSET` 排行榜（周榜）：

    ```java
    redis.opsForZSet().add(weekKey, userId, grade); // 写入即上榜
    redis.expireAt(weekKey, nextMonday()); // 榜单随周期自动消失
    redis.opsForZSet().reverseRangeWithScores(weekKey, 0, 99); // top 100
    ```

  - `INCR`/`setIfAbsent` 计数与发号（顺序的 `VIP` 编号）
  - `SETNX` 分布式锁（赛季匹配：加锁后 double-check，幂等）：

    ```java
    var acquired = redis.opsForValue()
        .setIfAbsent(lockKey, lockValue, Duration.ofMinutes(5));
    if (Boolean.TRUE.equals(acquired)) {
      if (alreadyMatched(seasonId)) return; // double-check：等锁期间别人可能已做完
      doMatch(seasonId);
    }
    ```

  - `pub/sub` 做 `WebSocket` 跨副本转发（见消息框架）

- 一个反直觉的决定：**推送限流没有用 `Redis`**，而是把每次发送落库（`Mongo` 文档），滚动窗口内计数——代价是性能，换来的是运营后台可以直接审计限流决策（见“推送”）

#### Redis 在这里更像“加速器和协调器”，不是第二个主数据库

`RedisKeyProvider` 如果解析不到 discrepancy 会直接抛出异常，说明 key 隔离是必需配置而不是“最好加一下”。Spring Cache 的序列化器也启用了受限多态类型，只允许项目包、基础 JDK、时间和 BSON 类型，避免把任意类型名交给反序列化器。用户仓库目前只缓存 `users-v2` 的部分查询，写方法统一 `@CacheEvict(allEntries = true)`，是偏保守的正确性优先策略；批量 `MongoTemplate` 直写则必须主动调用 `evictUsersCache()`，否则缓存会留下旧用户摘要。

排行榜没有统一使用 Redis。竞技场榜有独立缓存服务和多个 key 变体，但活动榜的真实数据、动态排序和每日数据本来就在 MongoDB，读取时还可能合成机器人并按相同排序重排；把它同步到 ZSET 会新增“Mongo 写成功但 ZSET 更新失败”“运营改排序后重建”等一致性问题。周榜等固定指标才适合 `ZSET`：分数模型稳定、更新频繁、榜单可按周期过期。

源码里真正使用 Redis 锁的代表是联盟赛季匹配：`SET NX EX 5 minutes` 后再次检查是否已经建组，锁只负责减少并发建组，不承担最终正确性；最终仍靠已存在的竞争组检查、数据库状态和快照。释放锁时应使用 compare-and-delete Lua，当前这段代码的 `setIfAbsent` 体现的是获取和 TTL，面试时要主动说出安全释放、续租和 fencing token 的缺口。

### 消息队列：`RabbitMQ`

- 核心选型是 `rabbitmq_delayed_message_exchange` 插件：一个 `x-delayed-message` 类型的交换机充当**分布式延迟任务调度器**，机器人行动、赛季阶段推进、延迟通知全部经由它武装（arm）未来的执行：

  ```java
  @Bean
  CustomExchange delayExchange() {
    return new CustomExchange("delay-exchange", "x-delayed-message", true, false,
        Map.of("x-delayed-type", "direct")); // 路由键 = 队列名，延迟毫秒数放 header
  }

  // 30 分钟后执行一次机器人攻击
  rabbit.convertAndSend("delay-exchange", queueName, event, m -> {
    m.getMessageProperties().setDelay((int) Duration.ofMinutes(30).toMillis());
  });
  ```

- 为什么用 `broker` 而不是进程内调度器（`@Scheduled`/`Quartz`）：
  - 延迟执行状态在 `broker` 里，**跨进程重启不丢**；进程内调度器重启即失忆
  - 队列的多消费者语义天然把工作分摊到副本
  - 持久化 + 死信：奖励关键链路（结算）配 `DLX/DLQ`，重试退避耗尽后消息**停在死信队列等待人工重放**而不是静默丢失：

    ```java
    return QueueBuilder.durable(settlementQueue)
        .deadLetterExchange(settlementQueue + ".dlx") // 重试耗尽 → 进死信
        .deadLetterQueue(settlementQueue + ".dlq")   // 运营手动重放
        .build();
    ```

  - `publisher confirm`：发布方阻塞等待 `broker` 确认延迟消息送达

- 多产品共用一个 `broker`，隔离只靠队列命名：队列名**没有默认值，未配置直接启动失败**（`@PostConstruct` 校验），宁可 fail-fast 也不静默串台：

  ```java
  @PostConstruct
  void verify() {
    if (queueName == null || queueName.isBlank())
      throw new IllegalStateException(
          "queue name must be set to isolate this product/env on the shared broker");
  }
  ```

- 没有 `broker` 的进程（如本地开发）只对明确支持降级的路径使用同步实现：联盟结算可以内联执行，阶段消息生产者则是 noop，靠 jobs 的 tick 对账推进；不能笼统地说所有延迟任务都会自动变成内存调度

#### 以联盟赛季阶段为例，消息和状态机各自负责什么

阶段消息不是“请把 phaseIndex 加一”这么简单，而是携带 `seasonId`、`expectedPhaseIndex` 和 `expectedPhaseStartedAt`。消费者重新读取赛季后，只有阶段下标和开始时间都与消息快照一致，且当前时间已经达到边界，才允许推进；如果宕机跨过了多个阶段，`advance()` 会在一次消费中循环追赶，并根据计划边界计算新的 `phaseStartedAt`，而不是用当前时间重新锚定，从而避免长期漂移。

推进成功后，进入 `ROUND_SETTLEMENT` 或 `SETTLING` 只负责投递结算命令，重量级结算由 API 消费端执行；没有 RabbitMQ 时，`AllianceSeasonSchedulerConfig` 提供同步内联的 settlement scheduler，阶段 scheduler 则是 noop，靠 jobs 对账 tick 推进。这是降级，不是“所有消息功能自动保留”。

不同生产路径的可靠性也不完全相同：阶段切换和机器人延迟消息使用 `rabbitTemplate.invoke(... waitForConfirms(5s))`，明确检查 broker confirm；联盟结算目前以 `x-delay=0` 投递，没有共享的 confirm 等待。结算队列额外设置 DLX，API 容器生产环境最多重试 30 次，`BusinessException` 这类确定性业务拒绝不重试，其他异常指数退避后进入 DLQ。结算消息本身仍必须幂等，因为 confirm、消费 ack 和业务写入之间不可能天然形成 exactly-once。

### 可观测性：`PLG`

- 应用以 `JSON` 格式打日志到 `stdout`（自定义 `logback` layout 把 `MDC` 里的请求标识打平进 `JSON`），环境变量可切回人类可读的控制台模式——然后 `Promtail` 抓容器 `stdout` → `Loki` → `Grafana`：

  ```json
  {
    "ts": "2026-08-17T10:00:00Z",
    "level": "INFO",
    "logger": "...ActivationController",
    "id": "67f2...",
    "name": "u123",
    "ip": "203.0.113.7",
    "msg": "activated"
  }
  ```

- `Grafana` 的看板（`JVM`/`Spring`/`MongoDB`/日志/`WebSocket`）与告警规则全部 `as code` 随 `helm` 部署；`Loki` 用对象存储做 `chunk` 与索引后端，配置保留期
- 指标走 `actuator` 暴露 `/actuator/prometheus`，`Pod` 注解声明抓取；网格 `sidecar` 的访问日志同样输出到 `stdout`，入口流量与应用日志在 `Loki` 里统一：

  ```yaml
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/path: "/actuator/prometheus"
  ```

- 有意思的一笔：监控栈里挂了一个把 `Grafana` 暴露给 `AI agent` 的 `MCP` 服务（只读），让运维助手能自己查图查日志

#### 观测数据是怎样从代码走到看板的

应用侧没有把日志直接写文件，而是 `logback.xml` 同时准备普通控制台 appender 和 `STDOUT_JSON`；`LOG_APPENDER` 决定实际使用哪一个。`JsonLogLayout` 继承 JSON layout，只额外把当前 `MDC` 展平到日志对象。认证过滤器、激活流程和 WebSocket 拦截器会写入用户 id、昵称、设备、IP 等上下文，因此一次请求的日志可以按 id 串起来；但这仍然不是完整的分布式 trace，跨 RabbitMQ 或 Redis 的调用没有自动 trace parent。

指标则走 Spring Boot Actuator/Prometheus。`spring-app` chart 在 Deployment 上声明 `/actuator/prometheus` 的抓取注解，监控 chart 用 Prometheus relabel 规则发现这些 Pod；MongoDB、JVM、Pod 和 WebSocket 的 Grafana dashboard/alert rule 都以 Helm 模板提交。Grafana MCP 通过单独的 Viewer service account 和 `disableWrite` 参数只开放查询能力，运维 agent 可以查图和日志，但不能通过同一个 MCP 直接修改 dashboard 或执行写操作。

这套 PLG 的边界也要说清：Loki 适合按标签和日志内容排障，Prometheus 适合计数、延迟和资源告警；不能把高基数的 `userId`、订单号全部当 Prometheus label，否则时序数量会爆炸。真正要补齐的下一步是给跨 HTTP、队列和异步任务加 OpenTelemetry trace，以及为队列堆积、DLQ 数量、缓存命中率、Mongo 慢查询建立业务告警。

### 项目部署结构

- 项目是经典的`gradle`多模块应用，不涉及应用层面的分布式框架
- 项目使用`k8s`，依靠其强大的基础设施能力，实现容器的部署和管理，有时遇到分布式服务间调用仍使用`k8s`结合`istio`实现服务发现和网格服务治理
- 部署层面做了两层抽象：
  - 内部 `helm` `chart`：应用 `chart`（`Deployment` + `Service` + 网关路由 + `HPA` + `PDB` + `NetworkPolicy` + 证书/镜像/数据库密钥）与任务 `chart`（`CronJob` 包装器），一个 `release` = 一个产品的一个应用，所有产品差异都在 `values` 文件里
  - 多云兼容：存储类按云厂商出不同模板，同一份 `chart` 跑在异构集群上
- 副本策略上 `CPU` request == limit，减少 `CPU` 节流带来的长尾延迟，也让云成本可预测；但内存 request/limit 并不总是相等，因此不能把这套 chart 直接称为完整的 `Guaranteed QoS`：

  ```yaml
  resources:
    requests: { cpu: "2", memory: 2Gi }
    limits: { cpu: "2", memory: 4Gi } # CPU 不超卖 → 不被 cgroup 节流
  env:
    - name: JAVA_TOOL_OPTIONS
      value: "-XX:MaxRAMPercentage=75 -XX:ActiveProcessorCount=2"
  ```

#### 多模块不是按“技术名词”拆，而是按启动形态和依赖方向拆

根 `settings.gradle` 中可以看到 `superlight-api`、`superlight-jobs`、`superlight-ops`、`superlight-service`、`superlight-dao`、`superlight-alliance`、`superlight-arena`、`superlight-generic-activity` 等模块。DAO 提供 Mongo 模型和 repository，service 依赖 DAO，业务域模块通过 `api project(...)` 暴露自己需要的 DTO/服务，应用模块负责把它们装配成一个可启动的 Spring Boot 程序。基础设施驱动在业务模块中尽量是 `compileOnly`，避免把 Rabbit、Redis 或 Mongo 的自动配置强行带进只需要领域逻辑的类路径。

应用 chart 的 Deployment 同时设置 CPU/memory request 和 limit；jobs chart 则主要设置 limit，因为 Job 的资源语义和常驻 API 不完全相同。jobs 的 `main` 使用 `WebApplicationType.NONE`，`CommandLineRunner` 完成后主动关闭 Spring context，让 AMQP 连接等资源释放，进程自然退出。CronJob 模板固定 `concurrencyPolicy: Forbid`、`backoffLimit: 2`，并关闭 Istio sidecar，否则业务容器结束而 sidecar 仍存活时 Job 可能一直不完成。

部署脚本还做了两个实际的工程化处理：一个是 `helm upgrade --install --wait --atomic`，发布失败自动回滚；另一个是把 Mongo URI、镜像仓库和产品 values 作为参数注入，而不是把密钥写进 chart。应用 chart 中的 `preStop`、readiness/liveness probe、PDB 和滚动更新参数共同决定发布期间是否会丢连接；不能只看 Deployment 副本数就认为发布安全。

### `Spring` 生态

- `Spring Boot` + `Java 21`：`Security`（游戏接口 `JWT` + 运营后台 `OAuth2` 登录）、`Data MongoDB/Redis`、`AMQP`、`WebSocket`(`STOMP`)、`AI`（`OpenAI` 兼容端点 + `Milvus` 向量库）
- `API` 文档用 `Spring REST Docs` + `Asciidoctor`：**文档由测试生成**，天然不会和实现漂移；放弃了手写 `swagger` 注解的路线：

  ```java
  this.mockMvc.perform(get("/events"))
      .andExpect(status().isOk())
      .andDo(document("events-list",
          responseFields(fieldWithPath("[].event").description("event type"))));
  ```

- `WebSocket` 两个刻意从简的决定：
  - 只用原生 `WebSocket` 不用 `SockJS`——移动游戏客户端是原生连接，不需要浏览器降级
  - `broker` 用内存 `simple broker` 而不是外置 `STOMP broker relay`——托管云 `RabbitMQ` 只暴露 `AMQP` 不装 `STOMP` 插件；跨副本投递改用 `Redis pub/sub` 桥接（见消息框架）

#### 为什么模块使用 `compileOnly`，以及 REST Docs 的真实链路

例如 `superlight-arena` 的代码依赖 `MongoTemplate`、Redis、AMQP 和 `wasmtime`，但这些 Spring starter 可以作为 `compileOnly`，由最终的 API 或 jobs 应用决定是否装配；竞技场核心计算则放进 `superlight-arena-power`，保持为可脱离 Spring 的纯 Java 模块。这样可以在单元测试里直接喂输入测试数学逻辑，也不会让每个领域模块都隐式启动一套基础设施。

API 文档不是运行时扫描 Controller 注解生成的。测试通过 MockMvc 调接口，`document(...)` 产出 `build/generated-snippets`，再由 Asciidoctor 组装进静态文档并打进 bootJar。它的优点是状态码、字段和响应示例会跟测试一起失败；代价是测试必须覆盖并维护文档片段，新增接口如果没有相应测试就不会自动出现在文档里。

## 通用活动框架

目标是“一个引擎，运营配置出任意限时活动”，新活动**不写代码**，只加配置文档

- **模板/实例分离 + 快照**：`GenericActivity` 是模板（时间模式、排序字段、机器人规则、客户端配置），开跑时物化为 `GenericActivityInstance`，配置整体拷贝进实例——赛季中改模板不会改写已经开跑的实例，历史不被重写
- 时间模式两种：`CALENDAR`（全局统一时刻表）与 `PERSONAL`（每人进入时起算的个人计时）；支持**链式活动**（`nextActivityId`，可自环）与**循环实例**（`maxCycles` 可为无限），实例的有效开始/结束时间和大部分状态按 `now` 计算，但数据库仍保留一个标记为 deprecated 的 `status` 字段，任务会用它做兼容查询/启动迁移，不能简单说成“状态完全不落库”：

  ```java
  public Status getStatus() { // 主要按时间派生；开始前仍可能回退到旧 status
    var now = Instant.now();
    if (now.isBefore(instanceStartTime)) return status; // deprecated 字段可能为空
    if (now.isAfter(instanceEndTime)) return COMPLETED;
    return IN_PROGRESS;
  }
  ```

- **分组策略**让排行榜变成同龄人竞赛：全局一组 / 按加入日分组（同一天进场的用户共享一个榜，人人有登顶机会的经典设计）/ 按开始日分组；分组名由模板里的格式串渲染：

  ```java
  public interface GroupStrategy {
    String groupOf(User user, GenericActivity activity);
    // BY_JOIN_DATE → "{date}_{activityId}"：2026-08-17_42
  }
  ```

- 排行榜是**纯 `Mongo` 查询**而不是 `Redis zset`：`SortField` 抽象排序（支持多字段、越小越好），按 `(activityId, group, cycle)`（日榜再加时区感知的 `dateKey`）查询排序；一个活动可挂多个自定义榜单（`CustomLeaderboard`），日榜/终榜分别配置是否结算；`RankUnlockRule` 控制名次窗口的可见性：

  ```json
  {
    "fieldName": "data.mergeScore",
    "order": "DESC",
    "priority": 0,
    "botRule": {
      "botCount": 40,
      "updateCount": 120,
      "minValue": 5,
      "maxValue": 60,
      "throttle": 25,
      "coefficient": 1
    }
  }
  ```

- 参与进度 `UserProgress` 以 `(activityId, userId, instanceCycle)` 唯一索引兜底；用户数据是 `(activity, group, cycle)` 下的 `Map<String, Object>` 灵活 `blob`，日维度数据另存；点赞数这类敏感计数只走专用原子接口，**绝不接受客户端提交的数值**：

  ```java
  // 点赞：服务器端原子自增，客户端只能“请求点赞”不能“提交点赞数”
  template.updateFirst(
      query(where("_id").is(docId)),
      new Update().inc("data.likes", 1), GenericActivityUserData.class);
  ```

- **机器人规则内嵌在排序字段里**（`botCount/updateCount/min-max` 增量/初始区间/节流/系数），分配时把规则**快照**进 `BotActivityAssignment`（`botUserId+activityId+cycle+fieldName` 唯一），实例中途改规则不影响已分配的机器人
- 结算：按榜单发 `UserEvent`（复用用户事件框架做奖励分发）；日结算用 `dailySettledKeys` 集合（`leaderboardId:dateKey`）做实例级幂等并保留最近 7 天；自注入 `@Lazy` 代理用于某些手动结算入口绕过 Spring AOP 自调用限制，但大榜单的自动结算**刻意不包成一个长事务**：

  ```java
  @Autowired @Lazy private GenericActivitySettlementService self;

  void settleAll(List<Instance> due) {
    for (var instance : due)
      self.settle(instance); // 需要事务的入口经代理调用；大榜单仍按幂等步骤重跑
  }
  ```

#### 一次提交从接口到排行榜的具体路径

用户先调用 `startActivity`。服务按时间模式选出实例，调用用户分组服务确定 group，再以 `(activityId,userId,instanceCycle)` 做 upsert 创建进度。`userCycle` 不是实例轮次：它记录这个用户跨实例参与的次数，所以同一活动链中“全局第 3 轮”和“用户第 2 次参与”可以同时存在。

提交数据时服务先检查进度是否存在、是否已结算；如果已结算且下一轮进度已经被结算任务预创建，就把写入转到下一轮，否则拒绝或忽略。主数据集合按 `(activityId,userId,instanceCycle)` 定位，写入 `data.<key>`、group、timestamp；时间戳大于零时先用条件更新，只接受大于已有 timestamp 的写，竞争失败后再区分“文档不存在”还是“写入过期”。配置了日榜时，同一份请求再写一份带时区 `dateKey` 的日数据集合。

排行榜读取不是把 `Map` 全部加载进 Java 后盲排：查询先按 activity、group、cycle 和 Mongo sort 取 `limit * 2`，真实用户不足时才调用机器人数据生成器，最后在内存中合并、处理并列排名和解锁规则。排序字段来自运营配置，因此排序合法性、缺省值、字段类型和 Mongo 索引必须一起考虑。

自动结算的事实链是：读取最近 3 天过期实例 → 每组每个榜查询排名 → 为用户写 `ACTIVITY_REWARD` 事件 → 标记 progress settled → 标记 instance 完成并预生成未来实例。源码明确选择事件创建和 progress 标记幂等，而不是把全榜放进 Mongo 事务；但事件“先写、进度后写”的中途失败会留下可重复检查的状态，因此 `createActivityRankingEvent` 还会按用户、活动、cycle、leaderboard/dateKey 检查已有事件。

## 排行榜

- 两套实现各得其所：
  - **活动榜用 `Mongo`**：数据本来就在文档的 `data.*` 路径上，灵活排序字段、多榜单、日榜 `dateKey` 都是查询条件的事；写多读少且榜随实例结束作废，不值得维护第二份 `zset` 状态
  - **周榜这类全局长期榜用 `Redis zset`**：高频读写 + 天然过期
- 榜单读取时**动态拼接机器人**：真实用户不足 `limit` 时，按分段规则即时合成若干机器人行（只存在于读时，从不落库），与真实数据合并后内存重排——填充氛围的成本几乎为零：

  ```java
  var rows = repo.findTop(activityId, group, cycle, limit * 2);
  if (rows.size() < limit)
    rows.addAll(botGenerator.generateInBands(limit - rows.size())); // 读时合成，不落库
  rows.sort(comparator); // 合并重排后裁剪到 limit
  return rows.stream().limit(limit).map(this::toRankItem).toList();
  ```

- 竞技场榜前挂一层缓存服务（全局/分组/档位多个变体），显式的清除钩子保证结算后立刻失效：

  ```java
  @Cacheable("arena-leaderboard")
  List<RankItem> leaderboard(String seasonId, int tier, int group) { /* ... */ }

  @CacheEvict(cacheNames = "arena-leaderboard", allEntries = true)
  void onSeasonSettled(SeasonSettledEvent e) { /* 结算钩子立即失效 */ }
  ```

需要诚实说明当前源码状态：`ArenaLeaderboardCacheService` 已经定义了全局、分组、段位分组三类缓存 key 和 `@CacheEvict` 方法，但其 `buildLeaderboardItems` 仍是待完善的占位实现，不能在面试中把它包装成已经完成的用户榜单缓存。真正能讲清楚的是缓存边界和失效策略：key 必须包含 season/tier/group/limit，结算或赛季切换后至少按 season 失效；如果采用 `allEntries = true`，会简单但会把其他赛季的缓存一起打掉。还要审查 `clearSeasonLeaderboardCache` 使用的 `'*' + seasonId + '*'`：Spring Cache 的普通 key 不提供通配符匹配，这个 key 很可能清不掉真实缓存，应该改成显式维护 key、按 season 分 cache，或接受 all-entries。缓存的正确性不应影响 Mongo 事实数据，缓存服务返回空或过期时应能回源。

## 联盟系统

业务上是公会那一套（成员/申请/建设/团购/科技树），值得记的不是玩法而是承载玩法的几个通用模式：

- **数值全部配置驱动**：任务、里程碑、商店、科技的数值都在运营可编辑的 `ObjectConfig` 里，代码只有引擎没有数值，新玩法调参不发版；唯一需要在服务端设防的是**跨联盟的重放**——建设任务上报用 `Redis` 冷却 `key` 防止换联盟后对同一任务重复上报，同联盟上报豁免：

  ```java
  var key = keyProvider.withParts("cooldown", userId, taskId); // 12h TTL
  var prev = redis.opsForValue().get(key);
  if (prev != null && !prev.equals(allianceId)) return; // 别的联盟已记过这题
  redis.opsForValue().set(key, allianceId, Duration.ofHours(12));
  ```

- **赛季是显式状态机**：`NOT_STARTED → 部署期 → 备战期 → 战斗期 → 回合结算 → 赛季结算 → ENDED`，阶段计划是**有序阶段列表**（类型 + 时长）；推进消息带 `expectedPhaseIndex/expectedStartedAt`，消费者会拒绝过期/重复消息，并在一次调用中追赶跨越的边界；调度器进程另有对账 `tick` 兜底重挂丢失的延迟消息。需要注意：当前实现是“读取后做幂等守卫再 save”，并不是下面伪代码所示的真正单条 Mongo `findAndModify` CAS；如果多个消费者同时读到同一版本，严格的一次推进仍应补充版本字段或条件更新：

  ```java
  // advance(seasonId, expectedPhaseIndex, expectedPhaseStartedAt)
  template.updateFirst(
      query(where("_id").is(seasonId)
          .and("phaseIndex").is(expectedPhaseIndex) // CAS：过期 tick 全部失配
          .and("phaseStartedAt").is(expectedStartedAt)),
      new Update().inc("phaseIndex").set("phaseStartedAt", now),
      AllianceSeason.class);
  ```

- **匹配**：幂等 + `Redis` 锁 + double-check；同赛区联盟按总战力降序切块进竞争组；人数不足的组**合成机器人联盟**补位，名字等级**快照进赛季组**——机器人联盟赛后即删除，快照保历史
- **地图节点战**：挑战锁带过期时间与回滚字段（锁前占领者）；驻守得分按 `lastHoldingScoreSettledAt` 幂等地发；战斗结果客户端上报、服务端用确定性引擎校验：

  ```json
  {
    "challengerUserId": "u9",
    "lockEndsAt": "2026-08-17T12:00:00Z",
    "preLockOccupierUserId": "u3"
  }
  // releaseExpiredLock() 过期释放时回滚到“锁前占领者”
  ```

#### 联盟战斗里 Redis 和 Mongo 各自做什么

匹配时先检查赛季是否已有竞争组，再用 Redis `SET NX` 把“同一赛季建组”短时间串行化；拿到锁后做 double-check，按联盟总战力排序切成固定规模竞争组，不足时补机器人联盟，并把成员列表、机器人名字、阵营等快照到 `AllianceSeasonGroup`。锁过期或消息重复时，数据库中的“已经有组”才是最终幂等依据。

挑战一个节点时，服务校验当前赛季、轮次、HQ 解锁条件、不能挑战己方和节点未被锁定，然后把节点文档写成 `LOCKED`，同时保存 `preLockOccupierUserId/AllianceId/OccupiedAt` 和 `lockEndsAt = now + 180s`。Redis 这里只记录联盟级短冷却，防止建设/战斗相关的重复上报；节点锁本身在 Mongo 文档里，因为挑战者、被挑战者、轮次和回滚状态必须一起被读取。

战斗在客户端完成后上报 `roundIndex`、攻守分数和攻方阵容。服务端只接受“仍为 LOCKED 且 challengerUserId 等于当前用户”的文档，胜利时结算旧占领方持有积分、更新占领者、防守阵容和战报；失败时恢复锁定前状态。每分钟 tick 还会以字段级原子更新推进 `lastHoldingScoreSettledAt`，避免整文档 `save` 把并发中的 LOCKED 状态覆盖掉。锁超时释放同样是条件更新，只有文档仍为超时 LOCKED 才广播 `LOCK_TIMEOUT`。

阶段推进要谨慎描述：当前 `AllianceSeasonLifecycleService` 会把消息中的期望阶段与数据库对象比较，再调用 `save`，并非真正原子 `findAndModify`。如果要严格证明两个 API 副本不可能同时推进，应该增加 Mongo `@Version` 或把期望阶段放进条件更新，并在更新成功后才发送后续消息。现有的对账 tick、消息重复 no-op 和业务状态幂等仍能提供较好的恢复能力，但不能把“参数像 CAS”说成“数据库已经 CAS”。

## 竞技场

- 一套实体走所有玩法：每用户每赛季一条 `ArenaPlayer`（`tier`、小组、分数、场次、阵容战力、是否机器人、升降级结果）
- 赛季按 `tier` 独立时钟，惰性创建（查无 `ACTIVE` 即开新季）
- **机器人密度是一等参数**：小组先填满真人，再垫机器人到下限（下限与低一档的小组规模联动）——榜面上永远热闹：

  ```java
  var floor = playersPerGroup(lowerTier(tier)); // 低一档的小组规模
  var botsNeeded = max(0, floor - realPlayers.size());
  ```

- 匹配规则与积分规则全部运营可编辑（`ObjectConfig`，每次请求重读，改完即生效无需重启）：分数区间加权抽取对手；胜负积分按百分位分档：

  ```json
  [
    { "tierPercent": 0.2, "winPoint": 12, "failPoint": -8 },
    { "tierPercent": 1.0, "winPoint": 8, "failPoint": -12 }
  ]
  ```

- 结算：每组头部比例晋级、尾部比例降级；排名奖励走 `UserEvent`
- 机器人攻击的节奏见“游戏内机器人体系”

#### 竞技场的“匹配”与“发起战斗”不是同一个概念

玩家加入某个 tier 的当前 active season 时，服务会清理其余 active season 的残留成员，再用唯一索引兜底并发插入。真实玩家加入后会释放低分机器人，按段位人数上限、低一档规模、最小/最大机器人数量补位；大乱斗段位则把该 tier 可用机器人全部拉入，不跨 tier 借用。这个过程解决的是分组和人口密度，不是选择一次对手。

点击匹配时，服务先复用 requester 当前的 ACTIVE 候选快照；快照中的玩家 id 会重新从 Mongo 读取最新分数，玩家被删除才刷新候选。没有候选时按 rank 对应的运营规则，基于当前分数百分比筛选，再从备用池随机补足；某些产品还会强制保证候选列表里有机器人。刷新候选会把旧记录标记 `INVALIDATED`，而不是覆盖历史。

真正发起战斗时，`ArenaCombatService` 在事务内确认双方在赛季、确认防守者属于当前候选列表、确认挑战者没有 pending 比赛，然后插入 `PENDING` match。完成战斗时只处理仍为 PENDING 的比赛，写入胜负和双方分数变化，再更新玩家统计；重复回调会直接返回，不重复加分。这里的一个风险是分数变化来自请求参数，生产接口必须确保这些值来自权威战斗计算或服务端校验，不能只依赖客户端上报。

## 战斗校验与 `WASM` 模块

- 问题：客户端（`Unity`）本地预测战斗结果，服务端需要**权威校验**，两边逻辑必须逐位一致
- 三层方案：
  - 纯 `Java` 移植客户端战斗数学（无依赖的独立模块），技能条件用正则解析一套迷你 `DSL`：

    ```java
    // 表驱动的技能条件表达式 → 正则逐条解析
    // Count(AllyLineup, AnchorId == 3) >= 2 → 队伍中该角色数 >= 2
    // Any(AllyLineup, HP% < 50)            → 任一队友血量过半以下
    ```

  - **`WASM`**：把战斗逻辑编译成 `WebAssembly`，服务端用 `wasmtime-java` 执行；如果客户端也加载同一份二进制，就能显著减少预测与校验的漂移，策划还可以通过模块版本切换减少整包发版。但当前仓库同时保留纯 Java 引擎和 WASM 适配器，不能直接宣称所有客户端都执行了同一个二进制：

    ```java
    // 模块导出：init(initialization.json) / calculate_power / calculate_battle
    // JSON-in/JSON-out，走 wasm 线性内存
    var result = wasm.call("calculate_battle", battleJson);
    ```

  - `WASM` 模块作为文档存储（`key+version` 唯一、`enabled` 开关、二进制放对象存储），运营上传后可先 **`dry-run`** 再启用；`JMH` 基准对比 `wasm` 与 `Java` 引擎的性能：

    ```json
    {
      "key": "ARENA_POWER_MODULE",
      "version": 3,
      "enabled": true,
      "wasmFilepath": "wasm/arena-power/v3.wasm"
    }
    ```

- 引擎单例 + 版本化热替换：`WasmModule` 用 `(key, version)` 唯一，服务每次从 Mongo 找 enabled 的最高版本；`AbstractWasmMemoryService` 发现版本/路径变化后重新加载模块和内存上下文，旧上下文关闭。这里的“热替换”仍需要关注并发请求与 `volatile` context 的生命周期，不能把它等同于完整的无停机版本治理。

#### Java 引擎、WASM 适配器和输入输出协议

仓库其实保留了两条实现路线。`superlight-arena-power` 的 `ArenaBattleEngine` 是无 Spring/Mongo 依赖的纯 Java 引擎，加载 `Anchor.json`、`LiveScene.json`、技能表，并用确定性的 PCG seed 计算回合、技能触发和最终热值；这条路线便于单元测试、回放和 JMH 基准。`superlight-arena` 的 `WasmArenaPowerService` 则实现同一个 `ArenaPowerService` 接口，把请求序列化成 JSON bytes，写进 WASM 导出的线性内存，调用 `calculate_power` 或 `calculate_battle`，再从返回的 64 位地址/长度打包值读回 JSON。

WASM context 启动时要求模块导出 `memory`、`malloc`、`free`，并调用 `init(initialization.json)` 加载初始化资源；每次调用都会分配输入内存、执行、读取输出并释放输入/输出，异常路径也释放输入。模块文件由 `StorageService` 提供，启用版本来自 `wasm_modules` 集合，因此运营可以把文件上传、存储、数据库开关和执行验证拆成步骤。测试里既有脱离 Mongo 配置直接加载本地 WASM 的 dry run，也有模拟 `WasmModuleRepo + StorageService` 验证“查最新启用版本 → 加载 → 计算”的路径。

“客户端和服务端完全一致”需要谨慎表达：如果客户端确实使用同一 WASM 二进制，确定性输入和 seed 可以显著减少漂移；但当前仓库本身也有 Java port 和 WASM adapter，任何一条路线都必须用固定战斗样例、版本号和回放数据验证，不能因为用了 WASM 就自动获得防作弊能力。WASM 解决的是执行一致性/隔离和热更新，不解决客户端伪造请求、模块内容审计或结果落库幂等。

## 支付系统

- 设计核心是**统一内购交易管线**：`App Store` 服务端通知、网页聚合支付、平台小游戏渠道，全部归一到同一个交易模型，发放/核销逻辑与渠道无关：

  ```java
  // 三种渠道都归一到同一交易：id = 渠道交易号，发放逻辑只认它
  IAPTransaction.builder()
      .id(channelTradeNo).productId(productId)
      .redeemed(false).status(PURCHASED).build();
  ```

- 网页聚合支付网关的典型集成：
  - 下单：本地生成 `outTradeNo`，参数按 `key` 的 `ASCII` 排序、跳过空值与 `sign`、嵌套对象按内部排序渲染、拼上密钥后 `SHA-256` 大写十六进制——签名算法在代码里逐条注释成文档：

    ```java
    // 1) 非 null 参数按 key ASCII 排序  2) 嵌套对象渲染成 "{k1=v1,k2=v2}"（内部也排序）
    // 3) "k1=v1&k2=v2" + "&key=<signKey>" → SHA-256 → 十六进制大写
    var sign = sha256Hex(joinSorted(params) + "&key=" + signKey).toUpperCase();
    ```

  - **回调幂等用唯一键插入**（见索引实践）：`DuplicateKeyException` 即已处理过，直接回 `success`；然后验签、映射订单、创建交易；`ProcessingStatus`（`IN_PROGRESS/INVALID_SIGN/USER_NOT_FOUND/PROCESSED`…）既是状态机也是审计日志

- `App Store` 集成（独立无依赖库，不引 `Spring`）：
  - `App Store Connect API` 同步商品目录与**所有地区的价格**：`ES256 JWT` 鉴权（`PKCS#8 .p8` 私钥签发，短 `TTL` 缓存），`JSON:API` 的 `included` 资源解析 + 游标分页：

    ```text
    # ES256：.p8 私钥签发的 JWT，20 分钟有效，缓存复用
    claims: { iss: issuerId, kid: keyId, exp: now + 20m }
    GET /v1/inAppPurchases?limit=200&cursor=...  # included 里解析各地价格点
    ```

  - 遗留收据的 `ASN.1/PKCS#7` 解析（`BouncyCastle` 验签 + 字段类型码 + `bundleId` 校验）

- 优惠券独立成券码 + 状态检查 + 幂等核销接口（返回奖励）

#### 支付链路里最重要的是“交易事实”和“发放动作”解耦

`IAPTransaction` 以渠道交易号为 id，保存用户、商品、消费类型、redeemed/status 和订阅过期时间；网页支付成功回调只负责把渠道交易转换成这个统一交易，客户端进入商店购买时再通过 `shopBuy` 核销交易并调用 `increaseShopPurchaseItem`。后者用 `ShopUserPurchase.transactionId` 做 upsert，只有首次 upsert 得到 `upsertedId` 才增加 `UserOperationStats.expense`、发布消费事件和写购买记录；重复核销返回 0，不重复累计消费。

EezPay 的下单阶段先生成 `eez + UUID` 的 `outTradeNo`，把请求参数落在 `EezPayOrderRecord`，按 `TreeMap` 排序、递归序列化嵌套对象、附加 sign key 后 SHA-256 大写签名，再调用网关。回调阶段保存完整 payload、更新订单状态，再把成功支付转换成 `IAPTransaction`。订单 id 与 outTradeNo 的索引设计、金额/币种/商品的服务端比对、回调重试和发货失败补偿，决定它是否真正安全。

这里必须直面当前实现的一个问题：`EezPayCallbackController` 先 `insert(IN_PROGRESS)`，再验签；同一个 `outTradeNo` 再次到达时直接返回 `success`。如果攻击者能在合法回调之前抢占这个 id，后续合法回调会被当成重复，从而造成拒绝服务；另外回调记录和 IAP 交易写入之间也不是一个跨系统事务。更稳妥的改法是先验签，再用唯一键占用；或者允许 `IN_PROGRESS/INVALID_SIGN` 重新进入状态机，并严格比对订单、金额、币种、商户和交易号。唯一索引只能解决并发去重，不能替代验签和状态机。

App Store Connect 客户端被单独放在无 Spring 依赖的 `appstore-java` 模块：它用 `.p8` 私钥签发 20 分钟 ES256 JWT，虚拟线程并发拉取 IAP/订阅商品价格，沿 JSON:API `links.next` 分页并解析 `included` 资源；手工价格优先于自动均衡价格。这样把 Apple API 的协议解析与业务发放解耦，也便于对账号迁移期间的 fallback credentials 做多凭据尝试。

## 用户注册与账号体系

- **设备优先的匿名激活**：`User` 以 `(deviceId, pkgName)` 为锚，公开激活接口 `upsert` 后签发 `JWT`；平台 `openId`（小程序/小游戏渠道）作为身份补充；`Telegram` 小程序入口校验 `initData` 后用 `telegramId` 合成设备号
- **第三方绑定是一个 root/peer 图，不是一个字段**：
  - `Apple/Facebook/Google` 等身份存映射表（每用户每类型至多一条，数据库唯一索引保证；并发竞态靠捕获 `DuplicateKeyException` 幂等吸收）
  - 不变量：**链深恒为 1**——绑定一个“未被认领”的身份则成为 `root`；绑定一个已被认领的身份只允许“完全处女”账号（任何类型都没绑过）以**非 `root` peer** 身份加入：

    ```java
    if (mappingRepo.existsByIdentity(type, identifier)) {
      if (!user.mappings().isEmpty())
        throw new BusinessException(ALREADY_CLAIMED); // 防止出现混合状态
      mapping = Mapping.of(user, type, identifier, /*root=*/false); // peer 直指 root
    } else {
      mapping = Mapping.of(user, type, identifier, /*root=*/true);
    }
    ```

  - `resolveActivatedUser()` 把任意 `peer` 的登录重定向到 `root`，所有共享身份的设备收敛到同一个 `User.id`——这就是“异地登录互踢”的实现；所谓**账号迁移/数据转移不搬数据**，只是把根文档重新指向：

    ```java
    User resolveActivatedUser(User user) {
      return user.mappings().stream()
          .filter(m -> !m.isRoot()).findAny()
          .map(Mapping::rootUser) // peer → root；链深 1 保证一步到位
          .orElse(user);
    }
    ```

  - 解绑是最微妙的部分：还有 `peer` 的 `root` 解绑时把 `deviceId` 改名为 `<deviceId>_orphan_<时间戳>`（映射行保留，`peer` 仍可解析），最后一个 `peer` 离开才删行；提供 `wouldLoseIdentityDataIfMemberUnbinds()` 预测“数据是否会永久搁浅”给客户端弹警告：

    ```java
    // root 解绑但仍有 peer：数据搁浅而不是删除，映射行保留让 peer 仍能解析
    renameDevice(user, deviceId + "_orphan_" + now.toEpochMilli());
    ```

  - 旧版 1:1 字段（`user.appleId` 等）由镜像服务同步，纯粹为客户端兼容

- 请求体校验切面兜底：`body` 里声称的 `userId` 必须与 `JWT` 主体一致（激活类接口可显式豁免）：

  ```java
  @Around("@annotation(org.springframework.web.bind.annotation.PostMapping)")
  Object check(RequestBody body, Authentication auth) {
    if (body instanceof UserIdClaiming c && !c.userId().equals(auth.getName()))
      throw new ForbiddenException(); // 不能替别人写数据
    return proceed();
  }
  ```

#### 激活、认证和绑定是三层不同的问题

激活不是普通的“登录查用户”。`ActivationController` 的 v2 接口接收加密请求，服务端根据 `openId`、`deviceId + pkgName`、平台 referer 或 Telegram `initData` 找到或创建用户，再把邀请人、审核/强更配置、IP 和客户端版本一起处理；保存用户后签发 JWT。JWT 使用 HS512，subject 是用户 id，过滤器每次请求除了验签还会重新加载用户并检查封禁状态，最后把 id、昵称、设备和 IP 写入 MDC。

第三方绑定则由 `ThirdPartyLinkService` 维护一组映射文档。`(userId,type)` 唯一索引保证一个用户每种身份只有一行，`(type,identifier)` 的 partial unique index 保证一个外部身份只有一个 root。绑定已被认领的身份时，只允许没有任何映射的“处女账号”作为 peer 加入；这条限制看起来严格，但它避免了 A 账号绑定 Apple、B 账号绑定 Google 后再互相拼接，形成需要递归合并的复杂图。

解绑时服务按物理设备的 `deviceId/pkgName` 定位成员，而不是只按当前已解析出的 root；root 仍有 peer 时会把设备改名为 orphan 并保留映射，最后一个 peer 离开才释放根映射。这个实现把“身份还能登录”和“这台设备原有的未绑定数据是否还会被看见”分开，`wouldLoseIdentityDataIfMemberUnbinds` 会在客户端操作前给出风险提示。

请求体的 `userId` 校验是第二道边界。JWT 解决“请求来自哪个已认证主体”，Advice 解决“body 是否试图替另一个主体写数据”；激活、回调等特殊接口必须显式列入豁免。运营后台则走 Feishu OAuth2 和方法级权限，不能把游戏 JWT 直接当成后台管理员身份。

## 客户端数据模型与同步

游戏真相在端上，服务端是“薄服务端 + 抽取 + 校验”：

- 存储模型：`ClientCustomData`，每 `(userId, key)` 一条文档，值是**不透明的 `JSON` 串**；`key` 如 `UserProxy`（经验/等级/道具）、`MergeProxy`（关卡进度/各玩法等级）、`FightingData`（阵容）等十几种
- 同步协议：
  - **时间戳单调的 last-writer-wins**：`upsert` 带时间戳守卫，过期写入静默丢弃；时间戳为零表示管理端强制覆盖；版本化接口从明文 → 加密 → 批量 `kvs[]` + **同一请求内原子完成一笔内购核销**（存档与购买不会脱钩）演化，另有 `RFC 6902 JSON-Patch` 端点在同样时间戳守卫下打补丁：

    ```java
    // 单调时间戳守卫：断网重连后的迟到写入不覆盖新数据
    var query = query(where("userId").is(uid).and("key").is(key))
        .addCriteria(or(where("timestamp").exists(false),
                        lt("timestamp", incoming.timestamp())));
    template.upsert(query, Update.fromDocument(doc), ClientCustomData.class);
    ```

    ```json
    {
      "kvs": [{ "key": "UserProxy", "value": "...", "timestamp": 1755400000 }],
      "iap": { "transactionId": "..." }
    }
    ```

  - 写入后异步“抽取”：解析不透明 `blob`，把服务端需要的少数字段镜像到 `User` 文档（等级、关卡、`VIP` 过期时间…）并发布领域事件——其余永远保持不透明：

    ```java
    @Async
    void extract(User user, String userProxyJson) {
      var proxy = parse(userProxyJson);
      template.updateFirst(/* ... */, new Update()
          .set("grade", proxy.grade())
          .set("episode", proxy.legendData().episode()), User.class);
      eventPublisher.publishEvent(new UserGradeChangedEvent(user.getId()));
    }
    ```

- `UserProxy` 里的道具是 `Map`（如 `"1"` 体力、`"2"` 钻石、`"3"` 金币），并带一套**互相咬合的客户端校验方程**（若干字段之间的等式关系），服务端可以复算校验
- 反作弊在数据入口做：每次同步异步跑欺诈检测（道具数量阈值 + 校验方程复算），超限自动封禁（保留原等级便于解封恢复）+ 白名单豁免 + 告警到 `IM`（附上该用户消费额，供运营判断是否大 `R` 误伤）：

  ```java
  for (var item : proxy.items().entrySet()) {
    var threshold = thresholds.of(item.getKey()); // "2" → 钻石上限
    if (item.getValue() > threshold)
      fraudHandler.handle(user, item); // 自动封禁 + 白名单豁免 + IM 告警
  }
  ```

#### 同步接口为什么要做“写存档 + 抽取镜像”两条路径

`ClientCustomDataService.update` 先以 `(userId,key)` upsert 原始 JSON，再异步调用代理方法执行 `extractInterestingFields`。这里的“代理”很重要：方法本身带 `@Async`，同一个 bean 内直接 `this.extract...()` 会绕过 Spring 代理，最终仍然在请求线程执行；源码用 `@Lazy` 注入自身来触发异步拦截。抽取失败不能回滚客户端 blob，因为两者已经是不同事实边界，应该记录告警并允许后续重试。

镜像只保存服务端确实需要查询的字段。例如 `UserProxy` 提取等级、episode、VIP 等并发布等级变化事件；`MergeProxy` 更新 matchStage、PVP/coop 等字段后显式清理 `users-v2` 缓存；道具 proxy 进入阈值和校验方程的风控流程。运营或其他后端查询用户时读镜像，客户端下次同步仍以原始 blob 为输入，避免把整个客户端 schema 复制进服务端模型。

时间戳接口的语义是“每个 key 的 LWW”，不是整个用户文档的版本。带 timestamp 的请求用 `updateFirst` 条件限制 `timestamp < incoming`；零时间戳是管理端明确要求的 unconditional upsert。JSON Patch 先读取当前值、确认 incoming timestamp 没过期、应用 RFC 6902 patch，再用旧 timestamp 条件 `findAndModify`；如果竞争失败返回 null，让客户端重新拉取而不是盲目覆盖。

这套协议仍然信任客户端时钟，因此不能用于货币、购买、奖励等高价值事实。真正需要服务端权威的数据应使用服务端递增版本、ETag/CAS 或事件账本；加密和时间戳只能降低传输篡改与离线乱序，不能证明客户端没有作弊。

## 用户事件框架

- `UserEvent` 是**面向用户的收件箱**：事件枚举（被赞/活动奖励/补偿/邀请返利/排名奖励/联盟战报…）+ `acked` 标志 + 一族**类型化的附件 `record`**（不同奖励结构内嵌同一个集合，见数据库一节）；部分场景用**部分唯一索引**幂等（如 `(userId, event, seasonId)` 的战报）：

  ```java
  UserEvent.builder()
      .userId(uid).event(Event.ACTIVITY_REWARD).acked(false)
      .attachment(ActivityRewardsAttachment.of(items)) // 类型化附件内嵌
      .build();
  ```

- `ack` 走 `CAS` 式更新，成功后发布 `UserEventAckedEvent`——监听器再把对应的站内通知推过 `WebSocket`，**收件箱与通知流保持一致**；被赞事件在读取时惰性填充点赞者列表：

  ```java
  // ack：仅当未 ack 时更新，天然吸收重复 ack
  template.updateFirst(
      query(where("_id").is(id).and("acked").is(false)),
      new Update().set("acked", true), UserEvent.class);
  eventPublisher.publishEvent(new UserEventAckedEvent(id));
  ```

- 活动结算、排名奖励、补偿发放全部复用这个框架：发奖 = 落一条带附件的事件，客户端 `ack` 后领取
- 运营后台可分页检索用户事件（客服排查的入口）

#### UserEvent 为什么像收件箱，而不是普通领域事件

它至少有三个生命周期：服务端创建未确认事件、客户端读取并展示、客户端 ack 后变成已确认。`acked=false` 的查询是用户收件箱；ack 使用 `_id + acked=false` 的条件更新，重复 ack 匹配不到任何文档，因此不会重复触发后续动作。被赞事件的点赞者详情按读取时补齐，避免每次点赞都把一个大数组写回事件文档。

活动结算和补偿并不直接改客户端货币，而是写带附件的 `ACTIVITY_REWARD`/`COMPENSATION` 事件，由客户端领取或 ack。部分联盟战报、竞技场赛季报告和活动奖励使用 partial unique index 或按业务字段查询去重。这里的 UserEvent 更接近“可查询的用户 inbox/outbox”，但它没有自动和所有外部副作用组成事务：WebSocket 通知、APNs 发送或 IM 告警失败时仍需依靠监听器重试、状态字段和运营补发。

因此它同时承担产品和运维价值：客户端拿到的是统一收件箱协议，客服可以按用户分页查历史，结算任务可以安全重跑。代价是集合会增长，事件附件需要版本兼容，查询必须按 userId/createdDate/acked 等访问模式建立索引，并定期清理已经没有业务价值的历史。

## 消息框架

### `WebSocket`

- `STOMP` over 原生 `WebSocket`，端点 `/ws`；用途：游戏内聊天（按 `{module}/{itemId}` 分话题 + 历史回放）、站内通知（每用户队列）、联盟战斗的实时转发
- **跨副本投递用 `Redis pub/sub` 桥接**：内存 `simple broker` 只懂本进程的订阅，发布方先把消息发到带 discrepancy 的 `Redis` 频道，每个副本的订阅器收到后再扇出到本地 `broker`（广播到 `/topic`，单播到 `/user/queue`）——用现成组件拼出一个“穷人的外置 `broker`”：

  ```java
  // 发送端：不直接投本地 broker，先发 Redis 频道
  redis.convertAndSend(keyProvider.withParts("chat", destination), payload);

  // 每个副本的订阅器：收到后投给本地 broker（本副本持有的订阅者）
  public void onMessage(String destination, String payload) {
    messagingTemplate.convertAndSend(destination, payload); // /topic 或 /user/queue
  }
  ```

- 鉴权在 `STOMP` 层不在 `HTTP` 层：`HTTP` 对 `/ws` 全放行，`CONNECT` 帧上解析 `JWT` 设置 `Principal`，`ChannelInterceptor` 链对 `CONNECT/SUBSCRIBE/SEND` 强制认证；流量按目的地打点进 `Micrometer`：

  ```java
  // CONNECT 帧的 native header 里带 JWT
  public Message<?> preSend(Message<?> m, StompCommand cmd) {
    if (cmd == StompCommand.CONNECT)
      return withPrincipal(m, jwt.parse(header(m, "Authorization")));
    return enforceAuthenticated(m, cmd); // SUBSCRIBE/SEND 必须已认证
  }
  ```

#### WebSocket 的连接、消息和历史分别落在哪里

配置使用原生 `/ws`，不启用 SockJS；客户端是移动游戏，不需要浏览器长轮询降级。HTTP 层对 `/ws/**` 放行只是因为 JWT 在 STOMP `CONNECT` native header 里，`WebSocketAuthChannelInterceptor` 解析 token 并设置 `Principal`，随后 Spring Security 的 inbound authorization 对 CONNECT、SUBSCRIBE 和应用消息做认证检查，DISCONNECT/HEARTBEAT 等控制帧按规则放行。

实时消息本身走 Redis Pub/Sub：`WebSocketRedisPublisher` 按环境和 destination 生成 channel，`ChatRedisSubscriber` 收到后如果没有 targetIds 就发 `/topic`，有 targetIds 就逐个发 `/user/queue`。Redis Pub/Sub 不保存消费位点，所以副本短暂断线会丢即时消息；这适合战斗刷新、聊天提示，不适合支付结果。

聊天历史另有 Redis ZSET，key 类似 `chat:{chatId}:history`，member 是序列化消息，score 是创建时间；读取最近消息时再批量查用户元数据，历史超过约 3 天会被裁剪。于是系统形成“Pub/Sub 负责低延迟广播，ZSET 负责短期回放，UserEvent 负责重要通知”的三层模型。源码还把 Redis listener executor 固定为单线程 FIFO，因为战斗 relay 消息乱序会导致客户端状态跳跃；极端积压时 AbortPolicy 可能丢消息，客户端必须能通过 REST 重新同步。

### 推送（`APNs`）

- 单一注入点 `PushNotifier`，当前唯一实现走 `Pushy`；按用户持有的设备令牌推送（不信任 `user.os` 字段——客户端更新不可靠）
- **限流落库**：每次发送先同步写入 `SENDING` 日志再计数（收窄并发窗口），滚动窗口内超阈值记为 `RATE_LIMITED`（同样落库，可审计）；阈值来自运营可编辑配置，短缓存：

  ```java
  var log = repo.save(PushLog.of(uid, SENDING)); // 先落库，收窄并发窗口
  var dispatched = repo.countByUserIdAndStatusNotAndCreatedAtGreaterThanEqual(
      uid, RATE_LIMITED, now.minus(window)); // 非 RATE_LIMITED 计数
  if (dispatched >= rateLimit.max()) {
    repo.updateStatus(log, RATE_LIMITED); // 丢弃但留痕，运营可审计
    return;
  }
  ```

- 通知模板按场景 `groupKey` + 语言唯一，按用户偏好语言解析、英文兜底；“何时发/发给谁”是 `core` 里定义的多态 `SPI`（具体实现放领域模块，`core` 不依赖它们）：

  ```java
  public interface NotificationWhen { boolean shouldSend(User user, Instant now); }
  public interface NotificationWho  { List<String> recipients(Context ctx); }
  // 具体子类放领域模块，经 Jackson type 判别反序列化 —— core 保持零依赖
  ```

- 延迟发送复用 `RabbitMQ` 延迟消息；无 `broker` 时退化为即时调度
- `IM` 机器人消息是另一通道：按时刻表（每天/每周的时分槽）由每分钟一跑的 `CronJob` 以一分钟回看窗口 + 每用户历史去重发送（幂等），间隔几十毫秒限速（对端接口有全局限速）

推送和实时 WebSocket 也有不同的失败语义。APNs 发送可以有失败、过期 token、模板渲染失败和限流，业务更关心“这次发送为什么没发”；因此 Mongo 发送日志保留 `SENDING`、`RATE_LIMITED` 等状态，运营能查到用户为什么没有收到。IM 定时任务则不依赖进程内状态：每次运行回看一个时间窗口，并用用户/模板/时间槽历史判断是否已经发过，任务重跑不会重复轰炸用户。

### 运营 `IM SDK`

- 一个很薄的 `IM` 开放平台封装：发消息/回复 + `tenant access token` 缓存 + 统一响应壳（`code != 0` 抛错）：

  ```java
  public <T> T call(Api<T> api) {
    var resp = post(api.path(), api.body(), tenantToken());
    if (resp.code() != 0) throw new IMException(resp.code(), resp.msg());
    return resp.data();
  }
  ```

- 两个集成方向：
  - **出站**：反作弊告警、用户反馈分流等系统消息发到运营群/个人
  - **入站**：运营后台用 `IM` 的 `OAuth2` 做登录（`SSO`），`tenant_key` 匹配则映射全套内部角色，否则普通用户——运营后台不再自建账号体系：

    ```java
    public Collection<GrantedAuthority> mapAuthorities(OAuth2User user) {
      return tenantKeyMatches(user)
          ? List.of(ROLE_SEAMAN, ROLE_INTERNAL, ROLE_ADMIN)
          : List.of(ROLE_USER);
    }
    ```

这个 SDK 的价值不是把 HTTP POST 包一层，而是统一三类契约：token 获取和缓存、平台错误码到异常的映射、业务 payload 的序列化。入站 OAuth2 则把身份认证与角色映射分开：Feishu 负责“是谁”，本地配置和 tenant key 决定“能做什么”，Controller 上再叠加 URL/方法权限。这样客服、版本管理员和超级管理员可以共享登录入口，但不会共享全部数据权限。

## 任务系统

- 一个 `Spring Boot` 应用装着几十个任务类，每个是 `@Component implements CommandLineRunner`，用任务专属 `@Profile` 门控；一次部署通过激活一个 `profile` **只跑一个任务**：

  ```java
  @Component
  @Profile("arena-daily-ranking-reward & !staging & !all-in-one")
  @RequiredArgsConstructor
  public class ArenaDailyRankingRewardJob implements CommandLineRunner {
    public void run(String... args) { /* 幂等是自己的义务 */ }
  }
  ```

- **没有进程内调度器，没有分布式锁**：调度完全交给 `K8s CronJob`，`concurrencyPolicy: Forbid` 就是全部的互斥故事——上一轮没跑完，这一轮不启动；`backoffLimit: 2` 快速失败防污染数据：

  ```yaml
  kind: CronJob
  spec:
    schedule: "7 1 * * *"
    concurrencyPolicy: Forbid # 全部的互斥：上一轮没结束，本轮不启动
    jobTemplate:
      spec:
        backoffLimit: 2 # 快速失败，避免反复重试污染数据
        template:
          metadata:
            annotations:
              sidecar.istio.io/inject: "false" # sidecar 会卡住 Job 完成
  ```

- 幂等因此成为每个任务自己的设计义务（回看窗口 + 去重、唯一索引、只进不退的推进）
- 任务型 `Pod` 显式关闭网格 `sidecar` 注入（业务容器退出后 `sidecar` 不退出会导致 `Job` 永不 `Completed`）
- 典型任务：赛季轮转与结算发奖、各类机器人更新（见下）、`RFM` 更新、热更版本状态、内容模块处理、数据保留清理

#### 为什么把“业务任务”和“调度器”拆开

`superlight-jobs` 并不是一个常驻 scheduler。`JobsApplication` 是非 Web 应用，启动后只装配满足 profile 的 `CommandLineRunner`，执行一次就关闭 context。比如 `AllianceSeasonSchedulerJob` 只调用一次 `allianceSeasonService.tick()`；`GenericActivitySettlementJob` 先把到点实例标记为进行中，再结算最近过期实例和昨日的日榜；`ArenaBotAttackScheduleJob` 会扫描所有 active season 的真人玩家并重发 `UserJoinArenaEvent`，用来补偿功能上线前或之前漏掉的事件。

这样做把时间触发交给 Kubernetes，把业务执行交给 Java。`CronJob` 的 `Forbid` 只能防止同一个 CronJob 的常规重叠，不能防住手工启动、多个 release 或 API 请求，因此每个任务仍需自己的幂等：回看窗口、状态条件更新、唯一索引、重复事件检查和可重跑的补偿逻辑都不能省。

以通用活动结算为例，任务不是只查“昨天刚过期”的一条记录，而是查最近三天的窗口；实例结算成功后写 `lastSettledAt`，日榜用 `leaderboardId:dateKey` 记录已结算 key 并裁剪七天前的 key。窗口扩大是为了吸收 Pod 重启和临时失败，幂等检查则保证扩大窗口不会重复发奖。

#### `spring-jobs` 镜像里到底有哪些任务

这里需要先澄清一个很容易在面试中说错的概念：`spring-jobs`（仓库中的镜像名是 `superlight-jobs`）不是一个“每十分钟把所有任务都跑一遍”的大程序，而是**同一个镜像、多个 CronJob、每个 CronJob 激活一个 profile**。`JobsApplication` 使用 `WebApplicationType.NONE`，Spring 容器启动后只装配当前 profile 对应的 `CommandLineRunner`，任务执行完就关闭上下文。部署 values 里例如可以把同一镜像分别配置成 `generic-activity-bot-updating-job`、`generic-activity-settlement-job` 或 `alliance-season-scheduler`，它们是不同的 CronJob 和不同的进程实例。

因此我在面试中会明确区分“我实际做过”“我参与过维护”“我读过代码、可以讲清楚但不是我主导”。这比把镜像里的几十个类全部说成自己写的更可信。

另外，Go 写的“视频第一帧缩略图”是单独的 `video-thumbnail-generator` 镜像和 CronJob，不属于这个 `spring-jobs` 镜像；面试时可以把它作为独立的跨语言批处理项目讲，不要和下面的 Java Job 混成一个任务。

| 个人边界               | 任务                                                                                                                       | 可以讲到的程度                                                                                                                                                               |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 深度参与/主导          | `AllianceCombatBotUpdatingJob`                                                                                             | 联盟战斗机器人按联盟分组、随机化、每轮限额、落 `AllianceCombatBotLog`、再交给异步战斗链路执行；相关代码和部署由联盟机器人功能一起演进                                        |
| 深度参与/主导          | `GenericActivityBotUpdatingJob`                                                                                            | 通用活动机器人分配、规则快照、`throttle`、随机增量、边界裁剪、日榜初始化、更新日志和历史清理；我会说明自己重点做的是规则和任务链路改造，不把整个活动引擎包装成一个人从零完成 |
| 深度参与/主导          | `UserConversationCategoryUpdatingJob`                                                                                      | 批量生成反馈会话分类和优先级，发现最高优先级反馈后生成飞书卡片并通知运营；它和运营后台、AI 分类服务是一条完整链路                                                            |
| 深度参与/主导          | `TwitterCouponDispatchJob`                                                                                                 | 拉取目标推文的 mention/转发状态，筛选同时满足条件的用户，随机分配有效优惠券，记录发放历史并通过私信发送；通过 Redis 保存 mention 游标，避免每次从头扫描                      |
| 参与维护/修复          | `ArenaSeasonRollover`、`ArenaSeasonReassigner`                                                                             | 理解赛季结束、最后一天奖励补发、升降段、失效匹配候选、真实玩家重新入场和机器人不迁移等边界；可以讲修复过的最后一天奖励问题，但不说自己从零写完整竞技场结算                   |
| 可以基于系统设计讲清楚 | `GenericActivitySettlementJob`、`AllianceSeasonSchedulerJob`、`ArenaDailyRankingRewardJob`、`ArenaBotAttackScheduleJob` 等 | 说明它们的输入、状态变化、幂等键、消息/数据库事实来源和失败恢复，但明确不是自己的主要代码贡献                                                                                |

##### 我实际做过或深度参与的较大 Job

**联盟战斗机器人更新任务。** 这个任务不是简单地“遍历机器人然后攻击”。生产环境的 `alliance-combat-bot-updating-job` 以 CronJob 运行，典型配置是每 30 分钟触发一次；任务启动后先读取 `ALLIANCE_COMBAT_BOT` 配置，`dailyAttackCountMax` 或 `attackBotCount` 不合法时直接停用，避免错误配置造成机器人集中攻击。

任务随后从用户集合找出 `bot=true` 的用户，读取当前联盟地图，查询仍处于 `IN_PROGRESS` 的机器人赛季上下文，再按联盟分组。每个联盟内先随机打乱机器人顺序，避免每次 tick 都是同一批机器人攻击，然后用 `attackBotCount` 限制本轮每个联盟最多发起多少次攻击。真正执行前先写一条 `AllianceCombatBotLog`，状态为 `SCHEDULED`，再交给 `AllianceBotService.processInProgressBot`。也就是说，Job 负责“扫描、筛选、限额、留痕和触发”，节点选择、阵容准备、延迟消息、战斗结果处理属于联盟服务和 RabbitMQ 链路。

这个拆分有几个好处：任务本身只需要处理有限数量的调度动作，不把战斗计算塞进 CronJob；每次攻击有日志，运营可以看到机器人为什么攻击、是否执行、结果如何；机器人攻击还可以继续沿用延迟消息和消费者重试。代价是状态跨越了 Job、Mongo 日志、RabbitMQ 消息和 API 消费者，必须把 `SCHEDULED`、执行中、完成/失败以及重复触发都定义清楚。因为联盟机器人直接影响玩家体验，审计和可恢复性比把代码写成一个大循环更重要。

**通用活动机器人更新任务。** 这是通用活动里比较大的批处理，不是一个“小定时加分”。任务分两个阶段：第一阶段扫描运行中的 `GenericActivityInstance`，确保机器人已经按照活动排序字段生成 `BotActivityAssignment`，并把当时的机器人规则保存下来；第二阶段读取这些 assignment，检查用户进度是否存在、活动是否已结算、实例是否过期，再根据 `lastUpdatedAt`、`updateCount` 和 `throttle` 判断当前是否应该更新。

真正更新时，任务读取机器人当前活动数据，按规则随机生成增量，处理 `minValue/maxValue` 的反转输入和实例绝对上下界，避免机器人分数越界；配置了日榜时，还要先保证当天的日榜数据已经初始化。写入活动数据后保存 assignment 的 `lastUpdatedAt`，同时记录 `BotUpdateLog`，最后按保留周期清理旧的机器人活动数据、进度和分配记录。

我在这块重点参与的是机器人规则和任务链路的改造，例如增加 `throttle`，把“更新次数”从固定等间隔变成带人类抖动的时间窗口；同时要考虑 `coefficient`、越小越好、初始值范围、实例上下界和日榜初始化。这里最值得讲的不是随机数，而是为什么必须持久化 assignment 和更新时间：如果只在内存里记录本轮更新，CronJob 重启后机器人会重新加分；如果不保存规则快照，运营中途改配置会让同一个活动里的机器人行为前后不一致。

**用户反馈分类与高优先级飞书通知任务。** `UserConversationCategoryUpdatingJob` 是运营流程中的批任务，生产环境可以按天运行。它调用 `AIFeedbackService.batchGenerateConversationCategory(true)`，后者通过 Mongo 聚合用户会话、过滤欢迎消息和已读状态，再让模型输出分类、优先级和摘要。Job 不负责重新实现 AI 分类，而是负责把批量分类结果转成运营动作：如果结果里出现最高优先级反馈，就生成飞书卡片，把摘要汇总后通知配置的群或用户。

这里的关键是区分“AI 结果”和“通知副作用”。分类结果需要 upsert，任务重复运行不能无限生成分类；通知失败不能把所有用户分类回滚，只能记录错误并通过后续任务或人工补发恢复。模型输出还可能不是合法 JSON，所以分类服务先解析和校验，Job 只消费结构化结果。这个任务体现了 AI 落地不是只调用一次模型，还要把批处理、人工运营、通知失败和重跑语义接起来。

**Twitter 优惠券发放任务。** 这个任务把外部平台状态、业务资格和优惠券库存串在了一起。任务先从 `KeyValue` 读取当前活动推文，使用 Redis 保存的 `since_id` 增量拉取 mention，再查询推文转发用户；只有同时评论/mention 和转发，且仍是目标活动的用户，才进入待发放集合。之后从 Mongo 聚合当前有效、未被发放过的优惠券码，随机取码给用户发送私信，并写 `CouponDispatchHistory`、更新 `UserTweetInteraction.sended`。

它的优点是活动配置可以放在运营配置里，任务重跑时可以依靠 mention 游标、用户互动状态和优惠券发放历史缩小范围；缺点是 Twitter API、Redis、Mongo、私信发送不在同一个事务里，发送成功但历史落库失败时仍可能重复或需要人工核对。所以面试时我会主动说：它是一个**外部 API + 本地状态机 + 幂等记录**的任务，不会声称依靠某个事务保证 exactly-once。

##### 同一个 `spring-jobs` 镜像中，虽然不是我主导但可以讲的较大任务

**通用活动结算与数据清理。** `GenericActivitySettlementJob` 先把到开始时间的实例从 `SCHEDULED` 标记为进行中，然后调用结算服务处理过期实例和日榜。结算服务为用户创建 `UserEvent` 奖励、标记进度和结算 key，任务本身可以通过回看窗口重复执行。`GenericActivityDailyDataCleanupJob` 则清理超过保留期的日数据和用户活动数据。面试时可以重点讲为什么结算和清理拆成两个 Job：结算是业务事实，清理是存储保留策略；清理失败不能影响奖励发放，结算失败也不能靠删除数据“修复”。

**竞技场赛季轮转和每日奖励。** `ArenaSeasonScheduler` 生产环境按段位检查当前赛季是否仍在本周，过期后依次执行结束赛季、补结算最后一天日榜奖励、处理升降段、失效旧匹配候选、标记赛季状态、创建下一赛季，并把真实玩家重新分配到新段位。`ArenaSeasonStagingJob` 只替换“是否过期”的判断，使用配置的分钟数做短周期演练；这说明业务流程和环境时间策略被拆开了。

`ArenaDailyRankingRewardJob` 则按“昨天结束时刻”结算每日排名奖励。它和赛季轮转之间有一个很容易漏掉的边界：赛季最后一天结束后，原来的每日任务只处理 `ACTIVE` 赛季，赛季一旦被轮转成 `ENDED/REWARDED`，最后一天可能永远没有奖励。因此轮转流程需要补处理最后一天，并依靠 `(season, day)` 之类的业务键幂等。这个问题我参与过修复，可以在面试中讲清楚，但要说明自己主要是维护和修复，不把整个竞技场赛季系统说成独立完成。

`ArenaBotAttackScheduleJob` 的职责又不同：它扫描所有 active season 的真实玩家，重新发布 `UserJoinArenaEvent`，让机器人攻击监听器重新武装缺失的延迟攻击。它更像一个自愈任务而不是业务结算任务，价值在于补偿历史漏事件；真正的幂等由 pending attack log 或状态检查保证。

**联盟赛季阶段推进。** `AllianceSeasonSchedulerJob` 一次只调用 `allianceSeasonService.tick()`，阶段计划、边界时间、过期消息、RabbitMQ 延迟消息和 WebSocket 通知由联盟服务协作完成。这个 Job 看起来很短，但不能只说“每分钟改一次状态”：它是一个对账入口，负责发现阶段已经到期但延迟消息没有成功推进的情况；状态机仍以 Mongo 赛季文档为事实，消息只是触发器。面试时需要把它和 `AllianceCombatBotUpdatingJob` 分开：前者推进赛季生命周期，后者产生机器人战斗动作。

**内容模块处理任务。** `ContentModuleProcessJob` 扫描存储目录中的 zip 包，逐个读取内容模块文件，按文件内容计算 MD5 作为文件名，生成缩略图，把原文件和缩略图写入存储，再按 package/module 汇总 Normal/Special 图片并写入 `ContentModule`，成功后删除 zip。它适合用来回答“为什么不在上传接口同步处理”：解压、图片处理和对象存储都可能耗时或失败，独立 Job 可以隔离在线请求；但也要补充 zip 路径穿越、压缩包大小、重复处理和删除时机等安全问题。

**用户分层和活动增长任务。** `UserRFMUpdater` 按最近活跃用户计算 `r/f/m`，其中 `r` 是窗口内最大商品价格，`f` 是购买次数，`m` 是累计消费，再根据运营阈值更新 `rLevel`，并用 `UserRFMState` 做降级防抖。`UserActivityTokenDailyRewardJob` 读取活动分组的前 3 名并写 `UserEvent` 奖励；配套的 bot plan/update 任务会为机器人生成每日 token 增长计划，再按小时或周期推进，模拟真实活动曲线。这些任务适合讲批处理扫描、配置驱动、奖励幂等和机器人数据与真实用户数据的隔离，但不应说成自己主导了全部用户分层系统。

**热更、应用版本和消息发送任务。** `HotUpdateVersionStatusUpdater` 处理未完成的资源压缩包，解压到存储目录并周期性更新 `processedFile/finished`；`AppInformationUpdater` 调用 Google Play/App Store 等平台服务比较版本和灰度比例，更新数据库后异步通知飞书；`TelegramBotMessageSenderJob` 每分钟运行，用一分钟回看窗口容忍 CronJob 延迟，按照日/周时刻筛选用户，再依靠发送历史去重，并以约 50ms 间隔控制外部接口速率。这些任务共同说明：外部 API 批任务最重要的不是“调用成功一次”，而是游标、回看窗口、速率限制、失败重试和本地审计。

**一次性修复任务也要和长期任务分开。** `ArenaPlayerSettledBackfillJob`、`ArenaMitigateTierJob`、`ArenaPlayerDedupJob` 这类任务用于历史数据回填、脏赛季迁移、重复记录清理和唯一索引修复，通常只在发布窗口单独启用 profile，不应当伪装成日常业务调度。它们很适合面试时回答“线上数据错了如何修”：先 fail-fast 检查目标状态，按唯一键和时间选择保留记录，分阶段更新，记录修改数量，重跑要么 no-op 要么结果一致，完成后再建立约束索引。

##### `spring-jobs` 任务如何运维

每个任务的发布配置至少要回答五件事：Cron 表达式、产品/环境 profile、数据库和外部服务连接、资源限制、失败后的重跑方式。例如通用活动机器人可以每分钟运行，结算可以每 5 分钟运行，联盟机器人可以每 30 分钟运行，反馈分类可以每天固定时刻运行；这些频率来自不同业务的延迟容忍度，不是把所有 Job 统一成一个频率。

运维上我会关注：Job 是否按时创建、Pod 是否启动成功、单次耗时是否接近调度周期、Mongo 游标和连接池是否耗尽、RabbitMQ 是否出现堆积/DLQ、任务处理数量和错误数是否异常、是否因为 `sidecar` 没退出导致 Job 卡在 `Running`。任务失败时优先查看结构化日志里的产品、profile、业务 id、处理数量和异常阶段；可以安全重跑的任务通过 CronJob 重试或手工创建同 profile 的 Job，不可盲目重跑支付和发奖任务。

`concurrencyPolicy: Forbid` 只阻止同一个 CronJob 的常规重叠，不能代替数据库幂等。比如活动结算要用业务结算 key，竞技场赛季要用 `settled/claimed` 条件更新，联盟机器人要靠攻击日志和状态机，反馈分类要用 upsert，外部优惠券要记录已发放历史。任务的真正可靠性来自“可重跑 + 可观测 + 可补偿”，而不是来自 Kubernetes 恰好只启动了一个 Pod。

## 运营后台

- 定位是“运营也是一个产品”：一套后台服务所有游戏（产品上下文经请求头路由）
- 能管的东西：`KeyValue/ObjectConfig` 配置、用户/封禁/白名单、大 `R` 消息（带阅读/投票/领取互动统计）、相册与 `AI` 陪聊内容、广告位与统计、优惠券、奖励目录、通用活动、联盟/竞技场运营视图、商品目录同步、通知模板、热更版本、客服快捷回复、站点代理白名单、`WASM`/内容模块上传
- 技术上有两个值得记的设计：
  - **通用 `Mongo` 表格查询框架**：一套 `filter/sort/pagination` 的声明式规格（`field:op:value` 过滤串）驱动所有后台表格和 `MCP` 工具——一张新表接入运营查询几乎零代码：

    ```yaml
    filter: ["userId:eq:67f2...", "event:eq:ACTIVITY_REWARD"]
    sortBy: createdDate
    order: DESC
    # TableSpec → Mongo Query/Sort/Pageable，运营表格与 MCP 工具共用
    ```

  - **角色模型**：`IM SSO` + 按角色的 `URL` 规则（客服只看用户/消息，版本管理员只看热更…）+ 方法级安全；`API`/`MCP` 访问另发**个人访问令牌**（可识别前缀的 `bearer`）

- 前端 `Vue 3 + Vuetify + Pinia`，`STOMP` 订阅实时数据

#### 运营表格框架的抽象边界

后台表格不是每个 Controller 手写一套查询，而是由表格定义提供可过滤字段、排序字段、投影和权限，再把 `field:operator:value` 解析成 Mongo `Criteria`，把分页转换成 `Pageable`。MCP 工具复用同一套 TableSpec，所以“后台能查到什么”和“agent 能查询什么”来自同一个字段白名单；如果让 MCP 直接接受任意 Mongo JSON，等于把数据库查询权限暴露给模型，既难审计也容易绕过字段级权限。

产品路由在 ops 也只对 `all-in-one` profile 生效，`ProductContext` 会同时影响 Mongo、Redis、Rabbit、APNs 和 Storage；普通部署仍然是一个 release 对一个产品。运营配置的 `ObjectConfig`/`KeyValue` 与应用静态 `AppConfiguration` 也要区分：前者适合可热改的数值和文案，后者适合连接串、密钥、队列名等不应由普通运营在线修改的基础设施配置。

## 安全设计

- **传输加密是注解驱动的 `AOP`**：`@EncryptedRequest/@EncryptedResponse`（及对应的明文豁免注解）配合 `RequestBodyAdvice/ResponseBodyAdvice` 做 `AES-CBC` + `RSA` 的信封加解密，`ThreadLocal` 标记贯穿日志脱敏；整个开关按 `profile` 可关（本地开发）：

  ```java
  @PostMapping("/sync/v5")
  @EncryptedRequest @EncryptedResponse // body 密文进来、密文出去
  Response sync(@RequestBody SyncRequest body) { /* ... */ }

  // DecryptRequestBodyAdvice: RSA 解 AES 密钥 → AES-CBC 解 body → 反序列化
  ```

- `JWT`（`HS512`，`subject` = `userId`）+ 封禁检查过滤器：封禁带等级与过期时间；还支持按包体的**破解版本封禁**（特定旧版本号登录即拒）
- `IP` 解析（`CDN` 头优先）进 `MDC`，激活时落到用户文档——地理与风控的基础：

  ```java
  // CF-Connecting-IP → X-Forwarded-For 首段 → remoteAddr
  MDC.put("ip", clientIp(request)); // JsonLogLayout 会打平进每条 JSON 日志
  ```

- **审核期配置伪装**：按包体的 `REVIEW_VERSION`/`REVIEW_LEVEL` 键（全局回退），审核中的构建拿到一套“合规配置”，过审后自动切回正式配置；白名单用户豁免；另有强制升级版本键
- **站点代理白名单**：服务端代理只转发到运营登记的前缀匹配主机，剥逐跳头；配合每日每用户使用上限，承载广告/任务墙类流量；站点行为脚本是混淆下发的 `Lua`：

  ```java
  public ResponseEntity<byte[]> proxy(URI target) {
    if (!isAllowedHostBoundary(target.getHost(), allowedHosts)
        || resolvesToPrivateOrMetadata(target.getHost()))
      return forbidden(); // 只转发到运营登记的主机前缀
    return forward(target, stripHopByHopHeaders());
  }
  ```

#### 加密 AOP 解决什么，不能解决什么

`DecryptRequestBodyAdvice` 只对带注解且 profile 允许的请求生效：读取 `{s,k,iv}`，用 RSA 私钥解出 AES key/iv，再用 AES/CBC/PKCS5 解 body；RSA 解密过的 key/iv 会放进一个带 1 小时 TTL、最多 20 万条的 Guava LoadingCache，降低重复握手成本。响应 Advice 按同一套注解和 `JsonView` 加密输出，`ThreadLocal` 标记用于避免日志打印明文。

这层设计把加解密从 Controller 中拿出来，也能逐 endpoint 兼容旧明文协议；但 AES-CBC 本身没有 AEAD 的完整性标签，不能仅凭“加密了”证明数据没有被篡改。生产上应依赖外层 HTTPS，并考虑 AES-GCM/独立 MAC、密钥轮换、重放 nonce 和请求时效；RSA/AES 信封也不能阻止被控制的客户端构造合法格式请求。

安全链路的顺序同样重要：先从可信 JWT 得到 userId，再做 body userId 校验，再做业务权限和数据范围校验；只在最后写 Mongo。上面的 `isAllowedHostBoundary` 是安全实现应有的边界判断，不是简单的 `startsWith`：当前代理代码的前缀匹配仍需要补齐域名边界、DNS 解析、重定向复核和私网/云元数据地址拒绝，否则 `allowed.example.evil.com`、重定向和内网解析都可能绕过。

## 缓存、云与存储

- 用户读走 `@Cacheable`（`UserRepo` 所有写方法 `@CacheEvict allEntries`，`MongoTemplate` 直写后显式调用逃逸口清缓存）——**缓存失效必须覆盖所有写路径**，直写模板绕过仓库的地方最容易漏：

  ```java
  @Cacheable("users-v2")
  Optional<User> findById(String id);

  @CacheEvict(cacheNames = "users-v2", allEntries = true)
  <S extends User> S save(S user); // 所有写方法统一清空

  // 逃逸口：MongoTemplate 直写（批量/聚合更新）后必须手动调
  userRepo.evictUsersCache();
  ```

- **存储抽象刻意只到 `POSIX`**：`StorageService` 接口只有文件系统实现；`K8s` 里把各云厂商的对象存储（`OSS/COS/GCS/US3`）经 `CSI` 挂成 `PVC`——应用**不链接任何云厂商 `SDK`**，换云是运维的事不是代码的事：

  ```java
  public interface StorageService {
    String store(String key, Resource resource); // 只有 POSIX 语义
    Resource load(String key);
    void deleteAll(String prefix);
  }
  ```

需要把“应用抽象”和“部署实现”分开：源码中的 `StorageType` 目前只有 `LOCAL`，`FileSystemStorageService` 只理解本地 POSIX 路径，上传文件名会用内容 MD5 生成，`load/resolve` 会规范化路径并拒绝跳出根目录。多产品模式下按 ProductContext 懒加载每个产品的文件系统服务，避免启动时为所有目录同时初始化。

云厂商差异在 Helm 层体现：`spring-app`/`spring-job` chart 根据 values 把 OSS、COS、GCS 或 US3 的 bucket 通过 PVC/CSI 挂载到容器，应用仍按文件系统读写。因此换云不需要引入 SDK，但会把一致性、挂载延迟、目录列表性能和 CDN 刷新责任交给 CSI/存储运维；如果将来需要真正的对象存储语义（分片上传、预签名 URL、跨区域复制），这个 POSIX 抽象就不够了，应该新增显式的 object storage port，而不是继续伪装成文件系统。

## `CI/CD`

- 主流水线：按模块矩阵跑测试（`CI` 上测试重试两次）→ 格式化门禁（`Google Java Format`）→ 矩阵构建/推送镜像（`bootBuildImage`，`git SHA` + `latest` 双标签，多镜像仓库）→ 按集群矩阵 `helm upgrade --install`（`values` 组合 + 环境变量替换；与其他 `release` 操作撞锁时**最多重试十次并自动 `helm rollback`**）→ 生成发布说明并发 `IM` 卡片（记录上次部署 `SHA`，给出 `diff` 链接）
- 巧思几处：
  - 提交信息带 `[skip build]` 时复用 `latest` 镜像（`pullPolicy: Always`）跳过构建——改一行配置不用等三十分钟构建
  - 分支到环境的映射：主干进生产命名空间，其余进 `staging`
  - 路径过滤：只改了前端站点或 `chart` 不触发后端构建
  - 应用列表（哪些产品要出镜像）从内部表格下载生成——**配置表格化**
- 依赖治理在构建层：`Gradle` 严格约束钉住易漂移的传递依赖（如 `gRPC` 全家桶统一版本）；快照依赖在 `CI` 上走自建镜像（公共快照仓库从 `runner` 访问不可靠）：

  ```groovy
  dependencies {
    // 传递依赖漂移控制：全家桶钉同一版本
    constraints {
      implementation("io.grpc:grpc-core:1.81.0")
      implementation("io.grpc:grpc-stub:1.81.0")
    }
  }
  ```

## `AI` 助手与 `Agent` 框架

### `Agent` 框架（`spring-agent`）

自研的基于 `Spring AI` 的工具使用型 `agent` 运行时，发布到 `Maven Central`，模块切分本身就是设计：

- `core` 是**无持久化后端**的运行时；`persistence-jpa/mongodb/redis` 各自实现 `core` 的仓库契约 + 提供 `Spring AI` 的聊天记忆仓库；`tools-shell-kubernetes/docker` 提供每用户一次性 `shell` 沙箱；`integration-feishu` 把 `IM` 变成 `agent` 的交互面；`app`（可部署服务器）与 `cli`（本地命令行）按属性在运行时选择后端组合
- 几个值得抄的决定：
  - **一套领域模型服务所有后端**：`core` 里的 `record` 同时标 `@Entity + @Document + @RedisHash`，持久化 `API` 在 `core` 是 `compileOnly`——反射时缺失的注解类型直接被丢弃，于是换后端不用换模型：

    ```java
    @Entity @Document @RedisHash // 一套模型，三种后端
    public record ScheduledTask(/* ... */) {}
    ```

  - **`classpath` 不决定行为**：加一个持久化/沙箱模块到类路径什么都不会激活，只有属性才会；解析器的主要用途是在报错里告诉你“缺哪个构件”。两个开关都是 `@Conditional` 且在 `AOT` 期求值——`native` 镜像里它们是**构建期决定**
  - 类路径隔离由**构建强制**：自写 `Gradle` 检查任务在 `Mongo` 驱动/`Jedis`/`Hibernate` 漏进 `core` 运行时类路径时直接失败——模块切分要守的不变量在 `Java` 源码里没有任何东西会报错，所以放进 `CI`：

    ```groovy
    // checkRuntimeClasspathIsolation：这些构件出现在 core 的 runtimeClasspath 就构建失败
    forbidden = ['org.mongodb:mongodb-driver-sync',
                 'redis.clients:jedis', 'org.hibernate.*']
    ```

  - **提问打断回合**：异步交互面（`IM`）上 `agent` 需要向用户提问时，持久化一个 `PendingQuestion`、抛出专用异常终止本轮，答案之后作为同一会话的新请求到达；每个会话同时最多一个未答问题（仓库层强制）；同步面（`CLI`）实现标记接口原地等答案续跑：

    ```java
    pendingQuestions.save(Question.of(conversationId, text));
    throw new QuestionNotAnsweredException(); // 本轮终止
    // 答案之后带着同一 conversationId 作为新请求进来
    ```

  - **每用户 `Pod` 沙箱**：`Bash` 工具在一个按用户哈希命名的 `K8s Job` 里执行（用 `Job` 而不是裸 `Pod`，是为了能探测到已终止的残留并在 `409` 时重建）；`PVC` 持久化用户家目录、空闲超时 + 硬期限、凭据走 `Secret` 挂载、输出截断
  - 用户可注册远程 `MCP` 服务器，但只允许 `streamable-HTTP`（不启 `stdio`——那会在宿主机上拉起进程），带 `SSRF` 防护（仅 `https`、拒绝私网/环回/云元数据地址）与运营白名单：

    ```java
    if (!"https".equals(url.getProtocol())
        || resolvesToPrivateOrMetadata(url)) // 169.254.169.254 等云元数据
      throw new IllegalArgumentException("blocked");
    ```

  - `agent` 可以**自我调度**：定时任务存库，重启后重载，到点经同一个入口发起新一轮——`agent` 用一个工具给自己定闹钟
  - 工具按场景组装（每用户的文件系统工具、待办工具、技能工具、活跃 `MCP` 工具），身份经类型化的工具上下文传递而不是字符串；横切关注点（超长响应拦截、卡片流式更新）放在 `ToolCallInterceptor` 链上
  - 一个踩坑记录在代码里：`ChatClient` 的工具调用 `advisor` 与记忆 `advisor` 叠加时，必须显式重新开启会话历史——没有哪个聊天记忆仓库会保存工具消息，否则工具循环在迭代间丢掉自己的工作消息、重复调工具不收敛

#### 一次 Agent 请求究竟经过了什么

`AgentRequest` 是框架和接入层之间唯一的入口对象，里面带 `userId`、`conversationId`、场景、提示词变量、类型化 `toolContext` 和 listeners。Feishu、CLI 或运维机器人只负责把外部消息翻译成这个 record，然后调用 `SpringAgent.fire()`；接入层不直接操作 `AgentToolsProvider`、MCP client 或 Reactor，这让“换聊天入口”不会复制一套 agent 循环。

`fire()` 是 fire-and-forget：先触发开始监听器，再按当前用户和场景组装工具，读取可访问且启用的 MCP 配置；用户自己的 MCP server 为每轮新建 HTTP client，多个 server 可并发连接，连接或 `listTools` 失败的 server 被记录并跳过。随后把模型调用放到 bounded elastic 调度器，监听器接收 token、工具调用和最终状态；`doFinally` 无论成功、失败还是取消都会关闭这轮创建的 MCP client。全局配置的 MCP 连接属于 Spring context 生命周期，不应在每轮重复关闭。

```text
外部消息
  → AgentRequest(user/conversation/scenario/context)
  → 组装本地工具 + 用户 MCP + 全局 ToolCallbackProvider
  → ChatClient（记忆 advisor + tool calling advisor）
  → 模型/工具循环
  → listener 流式输出、完成或失败
  → finally 释放本轮资源
```

持久化后端通过 `app.persistence.type` 选择 JPA（默认 SQLite）、MongoDB 或 Redis，而且同一个选择同时决定 agent 自己的仓库和 Spring AI chat memory。Redis 方案不是普通 key-value 就够了：聊天记录使用 RedisJSON/RediSearch，通常要求 Redis Stack 并关闭淘汰；metadata 中的 `messageType` 需要保持可检索，否则非字符串 metadata 会破坏索引。这个设计的收益是运行时只维护一套仓库契约，代价是每种后端都必须通过同一组语义测试：任务状态更新、问题状态、已处理消息去重和局部更新都要一致。

提问是一个真正的跨请求状态机。异步的 Feishu handler 先以唯一的 conversation/pending 状态保存 `PendingQuestion`，再展示卡片并结束本轮；用户提交答案后，handler 先原子地把问题改成 answered、关闭旧卡片，再用同一个 conversationId 创建新的 `AgentRequest`。CLI 实现 `SynchronousQuestionHandler` 时才可以在同一轮等待。定时任务也不依赖进程内内存：任务记录落库，应用启动加载 active 任务，到点重新走 `fire()`，所以重启不会让 agent 的闹钟消失。

### 运维机器人服务

一个独立的 `Spring Boot` 小服务，两半几乎互不相干：

- **`Webhook → IM 卡片桥**（无 `LLM`）：代码托管平台与监控告警的 `webhook` 进来，`HMAC`验签（防伪造）→ 按事件名分发到处理器`bean`（`PR`/评论/流水线/告警各一个，逐个 `try/catch`，一个坏载荷不影响其他投递）→ 渲染成 `IM` 交互卡片发到配置的群/人：

  ```java
  // 1) HMAC-SHA256 验签：密钥对原始 body 的签名比对（常量时间）
  var expected = hmacSha256(secret, rawBody);
  if (!MessageDigest.isEqual(expected, signature(header)))
    return ResponseEntity.status(401).build();
  // 2) 事件名 → 处理器 bean；逐个 try/catch，坏载荷不影响别的投递
  handlers.of(gitHubEvent).forEach(h -> safely(() -> h.handle(payload)));
  ```

- **`AI` 运维助手**，几乎全部由 `spring-agent` 组装，应用本身只贡献：
  - 模型配置：`OpenAI` 兼容端点，聊天/嵌入/视觉/语音转写各是独立模型；两个值得记的 `workaround`——快照版依赖会把 `"strict": null` 序列化出去被对端当严格校验，强制关掉；嵌入接口不管 `token` 数只卡**行数**，默认按 `token` 批量的策略要换成固定行数的批：

    ```java
    // 嵌入端点按“行数”而不是 token 批量 → 固定 10 行一批
    new FixedSizeBatchingStrategy(10);
    ```

  - 一个业务工具：收到表格链接后正则抽取 `token`，把翻译任务异步扔进线程池立即返回“已提交”——**长任务立即确认、结果后置通知**，不让模型干等：

    ```java
    @AgentTool(description = "翻译指定表格，完成后通知")
    String translateSheet(String link) {
      var token = extractByRegex(link);
      translatePool.submit(() -> translateService.translateAsync(token));
      return "任务已提交，完成后会通知你"; // 模型立即拿到结果继续对话
    }
    ```

  - 游戏文案翻译服务：系统提示词里写死了硬规则——保留 `<color=#...>` 这类标记、`{0}` 占位符原样保留、术语对照参考块保证一致性
  - 大段中文系统提示词描述家规：先读用户记忆、回答带引用的先查上下文、多步任务用待办工具、涉时间必调时间工具、不可逆操作先问、“长结果放进新表格，回复只给链接”

- 架构方向是它最有意思的地方：**应用是薄的，框架拥有 `agent` 循环**——工具发现走向量检索（工具描述做嵌入，按请求取 `top-5`，只有相关的工具会被提供给模型），交互面、沙箱、记忆、`MCP` 全部来自自动配置

## 知识库系统

知识库不是简单地“把文档切片后丢进向量数据库”，我参与的是一套从资源管理到解析、切片、向量化和问答检索的系统。它大致分成几个边界：

- 空间和权限：用户先在 Space 中拥有资源，再把资源加入 KnowledgeBase；知识库内部有 Catalog、CatalogMember 和 CatalogResource，目录负责组织资源，成员关系负责表达资源是否属于某个目录。
- 资源和解析：资源可以来自云文档、文本或其他可解析来源；解析任务不应该阻塞上传请求，而是写入处理状态，由独立 job 分页扫描并推进解析。解析结果拆成 ParsedDocument chunk，保存来源资源、顺序、文本和处理状态。
- 检索：文本切成带上下文的 chunk 后调用 embedding model，向量写入 Milvus；问答时先按空间/知识库权限过滤，再做向量召回，必要时结合关键词和文档元数据，把检索结果放进模型上下文。
- 一致性和重跑：同一资源重复提交不能无限产生重复向量；解析失败要能重试，资源被删除或替换时要清理过期来源，不能只删 UI 上的一条记录。大批量解析用 Mongo 分页，不把所有文档读进内存，也不依赖进程内 cron 状态。

这里有两个很有代表性的重构。第一，把自定义 AI provider 配置迁移到 Spring AI 的统一属性和模型接口，减少一套自研抽象与 Spring AI 的适配层；第二，把旧的 Doc、DocCatalog、ParsedDocumentSegment 关系收敛成 CatalogResource、ParsedDocument，避免同一资源在多个表和链接模型中重复表达。前者的优点是跟上框架生态、减少维护面，缺点是受框架生命周期和配置约束；后者的优点是查询和删除链路更直，缺点是需要迁移旧数据和重新确认领域边界。因为知识库的核心复杂度在资源生命周期和权限，而不是自定义模型名称，所以收敛模型的优点大于短期迁移成本。

面试时我不会把 Milvus 说成“用了就能搜索”。向量检索的质量取决于切片粒度、重叠窗口、embedding 模型、元数据过滤、top-k、重排和提示词长度；切得太大召回不精确，切得太小丢上下文，模型更换还会造成向量空间不兼容。生产上需要记录 embedding 模型版本、维度和来源版本，资源更新时重新向量化，删除和重试要有可观测状态。

## 游戏内机器人体系

（与上面的 `AI` 助手完全是两回事：游戏机器人是 `User` 集合里 `bot=true` 的普通用户文档，由任务与服务驱动，目标是**让游戏看起来有人气、给玩家对抗感**）

- **供给**：启动时的 `CommandLineRunner` 批量幂等初始化——从一个 `JSON` 名单/每行一个昵称的 `CSV` 建 `User`，跳过已有数量；给机器人**随机采样真实玩家的阵容快照**（按档位分层的资源文件，水塘抽样）——机器人穿的是匿名的真人阵容；昵称重置从唯一名字池洗牌分配，池尽则跳过绝不复用：

  ```java
  // 水塘抽样：从每档资源文件里均匀随机选一个真人阵容给机器人穿上
  FightingData sample(List<FightingData> pool) {
    FightingData chosen = pool.get(0);
    for (int i = 1; i < pool.size(); i++)
      if (random.nextInt(i + 1) == 0) chosen = pool.get(i); // 1/i 概率替换
    return chosen;
  }
  ```

- **竞技场机器人**是设计密度最高的部分：
  - 恒等不变量：**机器人只当攻击者不当受害者**；攻击节奏走延迟消息（几十分钟到几小时的随机间隔，像人）；攻击阵容按期望胜负组装（`winRate` 只是提示，真实战斗计算器说了算）；机器人若攻击阵容强于自己防守阵容，就把攻击阵容存为防守——“像真人一样摆最强队”
  - **双车道（two-lane）**：基础规则之上叠加**定时 override**（如“结算前最后一小时、只对前 10 名生效”）；`resolveLanes()` 产出 `normal + shadow` 双车道——`dry-run` 的 override **不影响真实攻击**，而是在并行的 `shadow` 车道全流程演练（数据落库、分数不动）；抑制型 override 同时杀掉两车道：

    ```java
    record Lanes(BotLane normal, BotLane shadow);

    // override 为 dry-run：真实车道照旧，shadow 车道全流程演练但不动分
    var lanes = resolveLanes(baseConfig, activeOverride(player));
    schedule(lanes.normal()); // 真实攻击
    if (lanes.shadow() != null) schedule(lanes.shadow()); // 演练：落日志、不动分
    ```

  - **`dry-run` 默认开启**——安全默认值：全链路照跑但分数永不结算，运营确认后再放开
  - 每次武装在 `(playerUserId, seasonId, shadow, status=SCHEDULED)` 上有唯一部分索引保证幂等（见索引实践）；被更紧的 `override` 取代时删除重挂；“先落日志再发消息，发送失败删日志”的顺序；每次攻击全量记审计日志（双方阵容、分数变化、中文跳过原因如“分差过大”）

- **联盟战斗机器人**：强度锚定是关键——目标战力取**同组同人数真人防守阵容的中位数**（回退到样本池的配置分位），再在 `[目标×下界%, 目标×上界%]` 区间采样具体阵容；选节点按“空/占用 × 总部/核心/普通”加权抽取，先在服务端预校验解锁规则避免无谓重试；每个 `tick` 把上下文洗牌（避免同一批机器人每轮都动手）、限制派出数量；被真人攻击后有延迟反击：

  ```java
  // 强度锚定：模仿同组真人而不是固定难度
  var target = median(realDefenses(group, headcount));
  var lineup = sampleWithin(pool, target * lowerPct, target * upperPct);
  ```

- **活动机器人**（通用活动框架内）：步调 = `实例时长 / 更新次数`，再被 `throttle` 随机提前——**均匀推进 + 人类抖动，绝不等距节拍器**；增量 = `随机[min,max] × 系数`（系数支持 -1 越小越好、>1 离散步进），夹在实例绝对界内；跨任务运行用稳定的机器人排序防漂移；`VIP` 机器人加权优先选出：

  ```java
  var interval = instanceDuration.dividedBy(rule.updateCount()); // 均匀推进
  var jitter = randomMinutes(0, rule.throttle()); // 人类抖动
  var delta = random(rule.minValue(), rule.maxValue())
      * rule.coefficient(); // -1 = 越小越好；5 = 五个一档
  ```

- **资料/榜单机器人**：每日给机器人用户的等级/分数字段加随机增量，让全局榜看起来活着
- 一切机器人动作都有运营可查的审计表（`ops` 里有专门的机器人标签页和日志过滤器）

## 用户分层与运营

- `RFM` 简化版：任务定期把 `r`（最大单笔）/`f`（笔数）/`m`（累计消费）写到用户上，`levelR` 三档由阈值配置；带**防抖规则**——窗口内最多降一档，观察期满才允许硬降：

  ```java
  // 降档防抖：一个窗口内最多降一档，避免单次未消费就大幅降级
  var next = min(previousLevel - 1, levelByThresholds(r, f, m));
  ```

- **冷启动分层看广告价值**：新用户注册时按其地区/设备的广告 `eCPM` 直接落入六个阈值档——一分钱没花也先估出付费潜力
- `IAP` 阶梯按 30 天消费分档，运营手动标志（核心玩家、剧情付费玩家“放慢其资源获取”）覆盖算法分层；最高档自动进**大 `R` 名单**，配套定向消息系统（阅读/投票/领取互动统计、消息级客户端数据快照、导出）

#### RFM、IAP 等级和大 R 运营不是同一套标签

仓库里至少有三种容易被混为一谈的概念。`UserRFMUpdater` 是一个 profile 受控的批任务：生产环境按 `app.rfm.calculationThresholdDays` 计算，非生产环境把窗口缩短到 30 分钟；它只扫描最近 14 天有更新的用户，减少全表扫描。`r` 是窗口内购买商品的最高价，`f` 是窗口内购买次数，`m` 是 `UserOperationStats.expense` 的累计消费金额，不是三个都用同一个时间窗口。

```text
UserRFMUpdater
  → 最近活跃用户
  → 查询窗口内 ShopUserPurchase + IAPItem
  → r=max(商品价), f=购买次数, m=累计 expense
  → 回写 User.r/f/m
  → 用运营 KeyValue 的 medium/high threshold 计算 rLevel 1/2/3
  → UserRFMState 记录上次等级与更新时间
```

`rLevel` 的降级有防抖：如果目标等级比当前低超过一级，且上次状态更新还在观察窗口内，先只把 `User.levelR` 降一级；观察窗口过去后才直接落到目标级。这样既能及时识别升级，也避免一次退款或短期不消费让运营权益瞬间跳水。初始化接口 `/rfm/init` 则使用另一组 `eCPM` 阈值给冷启动用户估计等级，某些产品还会保留用户已有等级；它不是把广告估值伪装成真实消费。

`UserService.userIAPLevel()` 是另一套购买等级：它看累计消费、近 30 天消费和特定商品规则；`isCorePlayer`/`BigRUser` 又是运营可以手动维护的名单，消息系统能按这个名单发送大 R 消息，并记录阅读、投票、领取以及当时的客户端数据快照。把这些维度拆开很重要：RFM 适合批量计算和自动降级，IAP level 适合商城权益，Big R 名单适合人工运营；如果把它们压成一个 `level`，客服无法解释“为什么这个用户有这个权益”。

# 面试官视角：可能会问的问题与参考回答

下面的问题不是把组件名称再背一遍，而是把“业务约束 → 方案选择 → 没选什么 → 代价与边界”讲完整。实际面试时还要把自己的贡献边界说清楚：亲手实现的、参与设计或排查的、只读过代码了解的，不能把整个系统都包装成一个实习生独立完成。

## 你先介绍一下项目，以及你具体负责了什么？

**候选人：** 这是一个面向多个游戏包体复用的游戏后端：同一份 `Spring Boot` 代码，通过产品配置隔离数据库、队列、缓存和对象存储；业务覆盖用户、活动、排行榜、联盟、竞技场、支付、推送和运营后台。它的核心目标不是“把中间件堆齐”，而是在客户端变化快、运营需要频繁调参、多个副本并行运行的条件下，保持数据隔离、幂等和可运营。

关于个人贡献，我会按事实回答：“我直接负责的是……，参与评审和联调的是……，其余模块我通过接口、数据模型和线上排障了解。”如果被问到没有亲自实现的部分，就说明输入输出、关键不变量和取舍，不假装能现场写出全部细节。对于实习经历，可信的边界比把项目说得无所不包更重要。

如果具体展开，我直接负责或主导过这些方向：

- 运营平台：和策划、客服以及客户端同学沟通反馈处理需求，开发反馈的 AI 分类和优先级、AI 助手、MCP 工具接入，以及“按反馈时刻保存客户端数据快照”。其中最容易被低估的是最后一项：原始需求是“任意时候都能查到某个用户的客户端数据”，但客户端只会持续覆盖当前数据，并不知道后端需要按哪个业务时刻回溯。因此我把需求转成了消息创建时的事实快照：按 userId 读取当时的 ClientCustomData，写入 messageId、key、data 的快照集合，运营后台再按消息查看。这个工作让我真正理解了“客户端不知道后端怎么实现，后端必须把自然语言需求翻译成数据模型和时序不变量”。
- 通用活动框架：参与活动模板、实例快照、用户进度、动态排序字段、日榜索引、机器人进度和结算链路的开发，重点是让运营配置有边界，而不是让客户端或运营输入任意 Mongo 查询。
- 游戏机器人和 WASM：开发竞技场、联盟和通用活动中的机器人行为，处理延迟攻击、阵容采样、dry-run、攻击日志和幂等；同时参与 WASM 模块的服务端加载、版本管理、运营上传和 dry-run，让客户端战斗代码可以在服务端复用。这里既有业务规则，也有线性内存、模块版本和释放生命周期等底层问题。
- 飞书机器人、AI 助手和知识库：开发飞书消息发送、卡片/回复、运营登录或告警入口，以及 AI 助手的反馈问答和 MCP 工具。知识库侧参与资源、目录、解析、切片、embedding 和向量检索的系统演进，处理过从自研 AI 配置迁移到 Spring AI、从内存任务迁移到 Mongo 分页任务、合并文档模型等问题。
- 通用 WebSocket 和赛季框架：开发聊天、通知、联盟战斗等通用 WebSocket 能力，处理 STOMP 鉴权、Redis Pub/Sub 跨副本、聊天历史、消息顺序和 REST 重同步；参与通用赛季/联盟赛季的阶段推进、MQ 延迟消息、保护期超时通知、机器人部署和结算。
- 工程和运维：开发许多一次性或周期性 Job，也做过一个独立 Go 小任务：从视频截取第一帧生成缩略图，由 CronJob、Docker 和 GitHub Actions 部署。参与 Spring Boot 3.5 到 4.0、Gradle、依赖目录和 JSON Patch 等系统升级，并使用 OpenRewrite 批量迁移依赖和 API。

这些工作可以在仓库的已关闭 PR 中核对，例如运营 AI/MCP 和快照相关的 #1380、#1590、#1593、#1619、#1625，活动索引 #1568/#1569，WASM #1538/#1547/#1551/#1554/#1556，WebSocket #770/#1078，视频缩略图 #1218，OpenRewrite 升级 #1422。PR 只能证明改动范围，不能替代我对设计和代码的解释；面试时我会说明哪些是我主导、哪些是联调或维护，而不会把整个仓库的所有模块都说成自己独立完成。

## 你在运营平台上做过最有代表性的需求是什么？

**面试官：** 不要只告诉我“做了 AI 和后台页面”。请选一个需求，从原始描述、澄清问题、数据设计、接口、异步流程和验收方式完整讲一遍。

**候选人：** 最能代表我思考方式的是“按反馈时刻保存客户端数据”。原始描述是“运营以后任意时候都能查到某个用户的客户端数据”。如果直接把当前 ClientCustomData 查询出来展示，用户下一次同步后，运营看到的就不是反馈发生时的状态，需求实际上没有被满足。

我先把问题拆成三个时间点：反馈发生时间、运营创建/处理消息的时间、运营查询时间；再和策划、客服确认他们需要的是哪个时间点。最终把业务事实定义为“运营消息创建时的用户数据快照”。创建消息成功后，异步按 userId 读取所有客户端数据，写入 messageId、userId、key 和 data；运营后台按消息查询快照，而不是回源当前数据。异步化是因为一条用户可能有多个数据 key，不能让发消息接口被大文档读取和写入拖慢。

这里的优点是查询语义稳定、历史可审计、不会被后续客户端覆盖；缺点是快照会增加存储，消息创建后到异步任务执行之间仍存在失败窗口。为了解决失败，我让快照服务捕获异常并记录日志，后台数据模型保留 messageId 作为关联键，后续可以补偿；如果需求要求严格“消息提交瞬间”的一致性，则需要在同一事务边界内写消息和快照，或者把客户端数据版本号一起保存。当前快照是运营辅助证据，不是支付账本，所以异步方案的优点大于缺点。

**面试官追问：** 你为什么不保存一个用户当前数据的引用，而要复制完整数据？

**候选人：** 引用只能指向会变化的当前文档，不能回到历史时刻；如果保存客户端数据文档的版本号，也要求原始数据本身具备可回放版本。当前客户端数据是按 key 覆盖的 blob，没有服务端版本链，所以复制快照是最直接、最可靠的方案。缺点是存储成本和隐私暴露面更大，因此后台权限、保留周期和脱敏范围也必须一起设计。

## AI 反馈分类是怎么做的？为什么不先做关键词规则？

**面试官：** AI 分类听起来很简单。你具体处理了什么数据，如何保证输出可用，为什么不用规则或传统分类模型？

**候选人：** 反馈分类不是直接把一条文本扔给模型。运营后台先按用户聚合对话，区分客服和用户发言，过滤欢迎消息和不需要处理的消息；对未读会通过 Mongo aggregation lookup 关联运营已读状态，只把未处理会话送去分类。模型输出 category、priority、summary，再解析 JSON，写入 ConversationCategory 或 Feedback 文档，后台按分类、优先级、未读和大 R 条件筛选。

直接关键词规则的优点是稳定、便宜、可解释；缺点是同义表达、上下文和“问题已经解决”很难覆盖。通用分类模型的优点是上线快、能处理自然语言和图片占位等变化；缺点是输出不稳定、可能幻觉、成本和延迟不可控。因此我没有把模型输出当成最终事实：提示词限制标签和优先级范围，解析失败返回空结果，客服仍能人工修改或继续处理。当前反馈量和标签变化速度让 AI 的优点大于纯规则，但支付、封禁等高风险决策不能只依赖模型。

**面试官追问：** 你怎么判断分类质量？只看“模型返回了 JSON”够吗？

**候选人：** 不够。JSON 只说明格式正确，不说明分类正确。我会抽样建立人工标注集，比较 category 的准确率、priority 的排序一致性、已解决问题的误判率，并按语言、游戏产品、反馈长度分桶。线上还要看客服改标签比例、模型解析失败率、重复分类率和高优先级反馈的漏报率。模型升级前用固定样本回归，不能直接用线上感觉判断。

## AI 助手为什么要接 MCP？为什么不把所有查询写成几个固定接口？

**面试官：** 你做了运营 AI 助手。请解释 MCP 在这里解决了什么，以及它带来的安全风险。

**候选人：** 运营问题不是固定的：可能要查用户、活动、联盟战报、反馈、Grafana 或消息记录。如果每种问题都写一个固定聊天接口，短期明确但工具数量会快速膨胀，模型也无法组合查询。MCP/ToolCallback 把后端能力描述成受控工具，Agent 根据场景选择工具，工具内部仍使用运营表格的字段白名单和权限。

固定接口的优点是类型、安全边界和审计都很清晰；MCP 的优点是工具可组合、可以复用现有服务、接入新数据源成本低。MCP 的缺点是工具描述、模型调用、权限和失败重试更复杂，模型还可能被提示注入诱导去调用不该调用的工具。因此我把查询工具设计成只读优先，使用个人访问令牌和方法级权限，工具回调统一记录 before/after，写操作必须显式开放并区分环境。对当前运营助手，组合能力的优点大于复杂度；支付和封禁等高风险操作仍应保留人工确认。

## 你在运营平台与策划、客户端是怎么沟通需求的？

**面试官：** 如果策划说“任何时候都能查到数据”，客户端说“我们只保存当前存档”，你如何推进？

**候选人：** 我不会直接在两个团队之间转述原话，而是把需求转成可以验收的事件和数据问题：查询对象是谁、时间点是什么、数据是否允许被修改、保留多久、谁能看、客户端是否需要感知。然后分别确认客户端能提供什么、后端能观察什么、运营真正需要什么。

我会给出至少两个方案：保存当前数据引用，或者按业务事件保存快照；把两者的优点、缺点、存储成本、实现时间和历史准确性写出来。和策划确认“查到的应该是消息发送时还是客服处理时”，和客户端确认 key 的稳定性、数据大小和敏感字段，再用一个具体用户流程做演示。最终把结论写到接口示例、数据模型和验收用例里，而不是只靠口头共识。

## 通用活动框架中你真正负责的难点是什么？

**面试官：** “配置驱动活动”很容易变成一句空话。请具体说一个你解决的工程问题。

**候选人：** 难点是运营可以配置动态排序字段，但 Mongo 查询性能依赖索引，而注解索引在编译时不知道 data.score 或 data.mergeScore。我的处理思路是启动时读取活动配置，为每个活动创建带 activityId partial filter 的动态复合索引，同时把 group/userId、cycle、排序字段和 dateKey 组合起来；活动实例又会快照配置，避免赛季进行中改模板导致历史语义变化。

另一个边界是结算。大榜单不能包在一个长 Mongo 事务里，所以我把奖励事件、进度 settled 和日榜 settled key 设计成可重复检查的步骤。这样做的优点是活动数量和榜单规模增加时可以分页、重跑和局部恢复；缺点是要面对“事件已写但进度未标记”的中间状态，必须有唯一业务键和补偿任务。因为活动的核心是规则可运营和结算可恢复，动态索引加短步骤幂等的优点大于强行全局事务。

## 游戏机器人和 WASM 这块你做了什么？为什么不是只写一个随机攻击脚本？

**面试官：** 请把机器人行为、战斗计算、运营控制和失败恢复分开讲。

**候选人：** 机器人首先是普通 User 数据和业务状态，不是一个独立的“假用户服务”。竞技场机器人要按段位和组规模补位，攻击动作走延迟消息；联盟机器人要根据同组真人防守阵容的战力分布选目标，攻击前服务端过滤不可挑战节点；通用活动机器人要根据排序字段的 min/max、updateCount、throttle 和 coefficient 推进。

我还参与了 WASM 复用客户端代码：服务端按 key 和 version 查找 enabled 模块，从 StorageService 加载，初始化线性内存，写入 JSON 输入，调用 calculate_power 或 calculate_battle，再读取并释放返回内存。运营侧可以上传、dry-run、启用和回滚版本；机器人攻击默认 dry-run 时完整跑链路但不改变真实分数，便于验证规则。

随机脚本的优点是实现快；缺点是行为太假、无法解释强度、没有幂等和审计。规则驱动机器人加 WASM 的优点是可调参、可回放、与战斗逻辑一致；缺点是状态、消息、版本和资源生命周期复杂。因为机器人直接影响玩家体验和活动公平性，后者的优点大于简单随机。

## 飞书机器人具体做了什么？它和游戏内 WebSocket 是什么关系？

**面试官：** 飞书机器人只是调用 SDK 发消息吗？

**候选人：** 我做的是一层可复用的 Feishu 集成：封装 tenant access token 获取与缓存、统一响应错误码、发送消息和回复消息；在上层接入运营告警、反馈处理、GitHub/GitLab/监控事件卡片，以及 Agent 的问答入口。卡片消息要考虑用户点击后的回调、消息 id、幂等和失败重试，不能只把 JSON POST 出去。

飞书是运营协作入口，WebSocket 是游戏客户端实时连接，两者都能承载通知但生命周期和可靠性不同。飞书消息要可审计、可重试、可定位操作者；游戏 WebSocket 更重视低延迟和在线连接，重要事件仍然落 UserEvent。把两者共用一个“消息发送接口”会掩盖可靠性差异，所以我会共用底层序列化和错误处理，不共用业务语义。

## 通用 WebSocket 是怎么设计的？为什么需要 Redis 桥和历史回放？

**面试官：** 一条聊天消息从客户端发送到另一台 Pod 上的客户端，完整链路是什么？

**候选人：** 客户端通过原生 WebSocket 连接 /ws，STOMP CONNECT 带 JWT；inbound interceptor 解析 token 并设置 Principal，之后校验 SUBSCRIBE 和 SEND 的权限。业务消息先保存必要的聊天历史，再发布到带产品/环境隔离前缀的 Redis channel；每个 API Pod 的 subscriber 收到后，转给本 Pod simple broker 中的本地订阅者。

Redis Pub/Sub 只负责跨 Pod 即时广播，不保存消费位点；聊天历史另用 Redis ZSET 保存有限时间，重要通知落 UserEvent，客户端重连后通过 REST 补齐。listener executor 固定单线程是为了保证战斗 relay 的 FIFO，极端积压时仍可能丢即时消息，所以客户端必须具备重同步能力。

直接使用外部 STOMP broker 的优点是 broker 原生管理订阅，缺点是托管 RabbitMQ 没有相应插件且会扩大客户端凭据暴露面；simple broker 加 Redis 的优点是接入简单、鉴权仍在应用，缺点是顺序和丢失语义需要自己承担。当前方案的优点大于缺点。

## 通用赛季框架中，MQ 和业务状态分别承担什么职责？

**面试官：** 如果延迟消息丢了，赛季会不会永远卡住？

**候选人：** MQ 只负责把“到某个边界后尝试推进”的命令送到消费者，赛季文档才是当前阶段和开始时间的事实。消息带 expectedPhaseIndex 和 expectedPhaseStartedAt，消费者重新读取后做状态守卫；重复消息 no-op，停机错过多个边界时按计划追赶。jobs 定期 tick 负责对账，发现状态已经到边界但没有下一条消息时重新武装。

因此消息丢失不能让业务永远依赖它，但当前读后 save 仍不是严格原子 CAS，两个消费者并发推进的边界需要进一步补条件更新或版本号。这个回答比“RabbitMQ 保证不丢所以没问题”更准确：Rabbit 的优点是持久化和重试，数据库状态机和对账才负责最终恢复。

## 知识库开发中，资源解析和向量检索最难的地方是什么？

**面试官：** 为什么不把每个文档直接存进 Mongo，然后全文搜索？

**候选人：** 结构化管理和语义检索是两个问题。Mongo 适合保存空间、知识库、目录、资源、解析状态和来源版本；向量数据库适合按 embedding 相似度召回语义相关片段。资源上传后先落元数据，再由 job 分页解析、切片和向量化，失败可以重试；问答时按权限过滤资源，再召回片段。

全文搜索的优点是解释性和精确关键词匹配好，缺点是同义表达和自然语言问题效果有限；向量检索的优点是语义召回，缺点是模型版本、切片策略、维度和召回误差都需要治理。因此我不会用向量库替代业务元数据，也不会只依赖向量 top-k；混合检索和来源版本记录的优点大于单一方案。

## Go 视频缩略图任务为什么单独做成 CronJob？

**面试官：** Java 服务里也能截视频第一帧，为什么用 Go 和独立容器？

**候选人：** 这个任务是文件处理型批任务，输入输出清晰：扫描待处理视频，解码第一帧，生成缩略图，写回 CDN URL 和处理状态。它不需要和 API 共享请求线程，也不应该因为某个大视频拖慢用户接口。独立 Go 容器可以使用更合适的媒体库和资源限制，CronJob 可以按产品和环境独立调度。

放进 Java API 的优点是部署单一、代码复用；缺点是 CPU、内存和异常文件会影响在线流量。独立 Go Job 的优点是故障隔离和资源可控，缺点是多一个镜像、workflow 和运行时。因为视频解码是高资源、低实时性的工作，独立任务优点大于多服务运维成本；任务仍要按视频 id 幂等、限制文件大小并记录失败原因。

## 你参与 OpenRewrite 的系统升级时，真正的工作是什么？

**面试官：** OpenRewrite 是自动改代码，那是不是运行一下命令就结束了？

**候选人：** 自动重写只能处理已知语法和 API 迁移，不能替代依赖图、运行时行为和测试。升级 Spring Boot 3.5 到 4.0 时，我还需要统一 Gradle 版本、依赖目录、Java 21 配置，检查 Jakarta 包和 JSON Patch 等 API 变化，处理 Mongo、WebSocket、加密 Advice、测试和 REST Docs 的编译/行为差异。

我的流程是先在分支上跑 rewrite，查看 diff 和依赖报告；再按模块编译，运行单元测试和 Testcontainers 集成测试；最后构建镜像并在 staging 验证探针、消息、WebSocket 和关键接口。自动化的优点是批量迁移一致、减少手工遗漏；缺点是可能生成能编译但语义错误的代码。因为仓库模块多、升级范围大，OpenRewrite 的优点大于纯手工修改，但必须用测试和发布验证兜底。

## 你具体做过哪些较大的定时任务？哪些虽然不是你做的，但你可以讲？

**面试官：** 我看到 `spring-jobs` 镜像里有很多 Job。请你不要泛泛地说“我做过定时任务”，而是按个人贡献边界讲清楚：哪些是你真正做过的，哪些只是看过代码？

**候选人：** 首先我会澄清部署模型：`spring-jobs` 镜像不是一个进程里同时执行几十个任务，而是同一镜像通过不同的 Spring profile 被部署成多个 Kubernetes CronJob。`JobsApplication` 是非 Web 应用，当前 profile 对应的 `CommandLineRunner` 执行结束后进程退出。

我实际深度参与、面试时可以详细展开的较大任务主要有四类。

第一类是**联盟战斗机器人更新任务** `AllianceCombatBotUpdatingJob`。它不是直接在 Job 里完成一场战斗，而是每次运行先加载机器人配置，检查每日攻击上限和本轮每个联盟的攻击上限，再查找机器人用户、联盟地图和处于 `IN_PROGRESS` 的赛季上下文。任务按联盟分组，随机打乱一个联盟内的机器人，避免每次都是同一批机器人先攻击；对每个联盟达到 `attackBotCount` 后就把剩余机器人留到下一轮。每次动作先写 `AllianceCombatBotLog` 的 `SCHEDULED` 记录，再调用服务触发节点选择、阵容准备和 RabbitMQ 延迟战斗链路。

我会特别说明我的工作边界：我参与了机器人联盟创建、更新、攻击调度、日志和运维配置等一整条链路的开发与修复，相关 PR 包括机器人联盟能力、攻击 CronJob 修复和攻击日志改造；但不会说“Job 一行代码完成了所有战斗”。真正的战斗规则、节点合法性、阵容和消费端执行在联盟服务和 API/MQ 模块中。这个任务适合讲批处理扫描、每联盟限流、状态日志、异步消息和失败恢复。

第二类是**通用活动机器人更新任务** `GenericActivityBotUpdatingJob`。它先扫描运行中的活动实例，确保机器人按排序字段生成 assignment，再处理 assignment：检查活动进度、结算状态、实例结束时间，根据 `updateCount`、`lastUpdatedAt` 和 `throttle` 判断是否到更新时间，生成随机增量并按实例上下界裁剪，写回活动数据和 `BotUpdateLog`。配置日榜时还要初始化当天的机器人数据，最后清理旧轮次的活动数据、进度和 assignment。

我参与的重点是机器人规则和任务链路改造，例如增加 `throttle`，让机器人不是严格按固定间隔跳分，而是在平均节奏上有随机抖动；同时处理 `coefficient`、初始值、绝对上下界、越小越好和日榜初始化。这类任务不能只在内存里记“上次加分时间”，因为 CronJob 重启后会失忆，所以更新时间和规则快照要落 Mongo。这里我会说“深度参与了任务改造”，不会说自己从零写完通用活动引擎。

第三类是**反馈分类与高优先级通知任务** `UserConversationCategoryUpdatingJob`。任务调用 `AIFeedbackService` 批量处理未读用户会话，服务侧通过 Mongo 聚合用户和客服消息，调用模型生成 category、priority、summary 并 upsert 分类结果；Job 只负责把最高优先级结果聚合成飞书卡片，通知配置的运营群或用户。分类结果和飞书通知是两个事实边界：分类可以重跑，通知失败需要日志和补偿，不能因为飞书失败就回滚已经保存的分类。

第四类是**Twitter 优惠券发放任务** `TwitterCouponDispatchJob`。任务根据运营配置找到目标推文，用 Redis 中的 `since_id` 增量读取 mention，查询转发状态，筛选同时满足评论/mention 和转发条件的用户；然后从有效且没有发放历史的优惠券中取码，通过私信发送，并记录 `CouponDispatchHistory`、更新 `UserTweetInteraction`。它本质上是外部 API 状态、Mongo 业务状态和优惠券库存的组合，不是一个简单的 HTTP 定时调用。它的难点是外部发送成功、本地状态更新失败时如何核对和补偿。

另外，竞技场赛季轮转的最后一天奖励问题我参与过修复：每日奖励任务只处理仍是 `ACTIVE` 的赛季，但赛季轮转可能先把它变成 `ENDED/REWARDED`，导致最后一天的日榜奖励漏发。修复后轮转流程补结算最后一天，并用赛季和日期维度的幂等语义避免重复发奖。这里我会把自己定位成“参与维护和修复这条任务链路”，不会把 `ArenaSeasonRollover` 整个说成是我独立完成的。

**面试官追问：** 那些不是你主导的 Job，你能讲哪些？

**候选人：** 我可以按业务链路讲，而不是只背类名。

- 通用活动方面，我能讲 `GenericActivitySettlementJob`：先启动到点实例，再结算过期活动和日榜；奖励以 `UserEvent` 作为事实，结算 key 和重复检查保证可重跑；`GenericActivityDailyDataCleanupJob` 负责按保留期清理数据，和奖励任务分开，避免清理策略影响业务事实。
- 竞技场方面，我能讲生产 `ArenaSeasonScheduler`、staging 的短周期版本、`ArenaDailyRankingRewardJob` 和 `ArenaBotAttackScheduleJob`。赛季轮转会处理结束、最后一天奖励、升降段、旧候选失效、创建新赛季和真人重新入场；机器人调度任务则是重新发布 `UserJoinArenaEvent`，用来补偿之前漏掉的机器人攻击武装，不等于直接执行战斗。
- 联盟方面，我能讲 `AllianceSeasonSchedulerJob`：它只调用 `tick`，但真正推进的是赛季状态机；延迟消息是触发器，Mongo 赛季文档是事实，定时 tick 是对账和重挂入口。还可以讲联盟战斗建议通知、团购每日初始化等任务的触发条件和幂等边界。
- 数据和运营方面，我能讲 `UserRFMUpdater` 如何扫描最近活跃用户并计算 `r/f/m`，为什么降级需要防抖；`ContentModuleProcessJob` 如何扫描 zip、按 MD5 存文件、生成缩略图和更新内容模块；`HotUpdateVersionStatusUpdater` 如何增量解压并更新 `processedFile/finished`；`AppInformationUpdater` 如何比较商店版本和灰度比例后通知飞书。
- 消息和活动增长方面，我能讲 Telegram 每分钟任务的一分钟回看窗口、用户发送历史去重和 50ms 发送间隔，也能讲活动 token 的每日奖励和机器人计划任务如何用 `UserEvent`、日榜数据和计划状态组合起来。

这些任务我会明确说“代码不是我主导，但我理解输入、状态、输出、失败路径和运维方式”。面试官真正想判断的是我能不能接手和排查，而不是我是否把仓库里的每个类都写过。

**面试官追问：** 为什么不把这些任务放进一个常驻 Spring 进程，用 `@Scheduled` 统一调度？

**候选人：** 常驻进程的优点是启动成本低、任务之间可以共享内存和连接池；缺点是一个任务卡住会占用同一进程资源，多个副本会重复执行，任务 profile、资源限制和发布回滚也不容易隔离。当前任务有不同的频率和资源特征：联盟机器人需要 RabbitMQ，内容模块处理需要文件和图片处理，反馈分类需要飞书和 AI，RFM 是批量数据库扫描，把它们放在一起会扩大故障半径。

Kubernetes CronJob + 一个 profile 一个 Job 的优点是资源、日志、失败重试、发布和手工重跑都能按任务隔离；缺点是每次启动 Spring 上下文有成本，也需要维护很多 release 和 values。因为这些任务大多是分钟级或更低频的批处理，隔离和可运维性大于启动成本，所以选择 CronJob。`concurrencyPolicy: Forbid` 只能减少同一个任务的常规重叠，不能替代业务幂等。

**面试官追问：** 如果 Job 运行到一半宕机，重新跑会不会重复发奖、重复发券或重复攻击？

**候选人：** 这要按任务类型设计，而不是给所有任务加同一种锁。活动结算用活动/榜单/用户/日期等业务键和 `UserEvent` 检查，赛季结算用 `settled/claimed` 条件更新；机器人调度使用日志状态、pending 检查和每轮限额；反馈分类用 upsert，通知失败单独记录；Twitter 则依靠 mention 游标、互动状态和 `CouponDispatchHistory`，但外部私信成功后本地落库失败仍需要人工对账或补偿。

因此答案不是“CronJob 保证只跑一次”，而是“任务按至少一次执行设计，数据库事实和幂等键保证重复执行不会扩大业务结果；对于外部副作用，再加发送记录、回查接口和人工补偿”。这也是我在实际开发中对定时任务最重要的认识。

## 为什么选 MongoDB？为什么不选 MySQL 或 PostgreSQL？

**候选人：** 选择 MongoDB 的主要原因不是简单地说“它更快”，而是业务数据有几个特点：客户端模型经常变化；活动、奖励附件和运营配置存在嵌套结构；很多读写天然以一个用户或一个活动实例文档为边界；活动排序字段还可能由运营配置决定。文档模型可以减少为了适应客户端变化而频繁拆表，也能用单文档原子更新和唯一索引解决大量并发问题。

不选关系型数据库，是因为本项目最常见的访问模式并不是复杂多表关联，且 `UserEvent`、客户端 `blob`、活动 `data.*` 这类结构需要灵活演进。但这不代表 MongoDB 在所有场景都更好：支付账务、强约束库存、复杂报表和大量多表分析更适合关系型数据库。MongoDB 仍然需要索引设计、数据兼容和迁移，只是迁移压力从“每次加字段都改表”变成了“改变读写契约时管理版本和兼容窗口”。

## 这里说的“分布式事务”到底是什么？为什么不用 Seata？

**候选人：** 文章里账号绑定、数据迁移、活动结算使用的是 MongoDB 的多文档事务。严格说，如果参与者只有同一个 MongoDB 集群，这首先是**数据库多文档事务**，不能把它包装成已经解决了跨服务分布式事务。它需要副本集，事务范围也要尽量小，并处理瞬时错误、超时和重试。

本项目没有把一次操作同时写 MongoDB、Redis、RabbitMQ 和外部支付平台，因此没有引入 Seata 这类全局事务协调器。跨边界的动作采用“数据库状态机/幂等键 + 消息重试 + 补偿”处理：例如先记录支付回调，再验签和推进状态；发奖以 `UserEvent` 作为可重试的事实记录。若将来拆成订单、库存、奖励多个服务，我会优先考虑事务消息或 `outbox + inbox`、Saga 和补偿，而不是期待一个分布式锁或全局事务解决所有问题。

## 为什么要用 RabbitMQ 延迟消息，而不是 `@Scheduled` 或 Quartz？为什么不选 Kafka？

**候选人：** 机器人行动、赛季阶段推进和延迟通知是“未来某个时间执行一次的命令”，需要在进程重启后仍然存在，并能由多个副本消费。RabbitMQ 的持久化队列、消费者确认、重试和死信队列适合这个模型；`@Scheduled` 的状态在进程内，Quartz 虽然可以持久化，但会额外引入调度集群和任务管理复杂度。

Kafka 更适合高吞吐事件流、顺序追加、回放和多下游消费。这里的消息大多是任务命令，不需要长期保留完整日志，也不需要让多个消费者独立回放，所以没有为了“以后可能做数据分析”而选 Kafka。粗粒度的每日批处理则交给 `K8s CronJob`，不同问题使用不同调度工具。

代价是依赖 RabbitMQ 延迟消息插件，云环境不一定支持；延迟消息也不是 exactly-once。生产实现必须把消费设计成至少一次：消息落地后再 `ack`，失败进入重试或 `DLQ`，业务用唯一索引、状态机或幂等键去重；发送确认成功和业务记录写入之间仍可能出现缝隙，重要任务应增加 outbox 或对账重挂机制。

## 某个地方的延迟任务是怎么实现的？

**候选人：** 以赛季阶段推进为例，阶段记录里保存当前阶段、开始时间和阶段计划。创建或推进时发送带 `seasonId`、`expectedPhaseIndex`、`expectedPhaseStartedAt` 的延迟消息。消费者重新读取赛季，检查消息快照是否仍对应当前阶段、阶段边界是否已到；如果进程停机跨过了多个阶段，就按计划边界循环追赶，而不是把下一阶段的开始时间重置为“现在”。推进完成后保存赛季，再武装下一条阶段消息。

```java
var season = seasonRepo.findById(seasonId).orElseThrow();
if (!season.matches(expectedPhaseIndex, expectedStartedAt)
    || clock.instant().isBefore(season.nextBoundary()))
  return; // 重复、过期或提前的消息都是 no-op

while (season.canAdvance(clock.instant())) {
  season.advanceOnce(); // 按阶段计划边界推进，可能一次追赶多个阶段
}
seasonRepo.save(season);
armNextPhase(season); // 保存后再发下一条消息
```

这里要把“当前实现”和“理想的原子实现”分开：仓库当前是**读取后做期望阶段/开始时间守卫，再修改对象并 `save`**，并不是上面这种单条 Mongo CAS。两个消费者仍可能同时读到同一个版本；重复消息通常会因后续状态检查变成 no-op，但严格证明“只推进一次”还需要版本字段或条件更新。例如可以把关键写入改成：

```java
var updated = template.updateFirst(
    query(where("_id").is(seasonId)
        .and("phaseIndex").is(expectedPhaseIndex)
        .and("phaseStartedAt").is(expectedStartedAt)),
    new Update().inc("phaseIndex", 1).set("phaseStartedAt", nextStartedAt),
    AllianceSeason.class);
if (updated.getModifiedCount() == 1) armNextPhase(seasonId);
```

因此当前方案的可靠性来自“消息至少一次 + 状态守卫 + jobs tick 对账 + 业务幂等”的组合，而不是来自延迟队列的 exactly-once。若继续演进，还要处理“数据库已推进但发送下一条消息失败”的窗口，通常会加 outbox 或可重建的对账任务。

## 为什么 Redis、缓存和分布式锁用得不多？是不是技术能力不够？

**候选人：** 恰恰是因为先确定了事实来源和一致性边界。Redis 在项目里承担了有限而明确的职责：用户和排行榜读缓存、周榜 `ZSET`、计数和发号、赛季匹配锁、联盟上报冷却、WebSocket 跨副本 Pub/Sub。它没有被拿来替代所有 MongoDB 数据，也没有把每一个写操作都包一层锁。

活动榜数据已经在 MongoDB 的 `data.*` 路径中，排序字段灵活且活动结束后价值下降，直接查询 MongoDB 比维护一份 Redis `ZSET` 状态更简单；推送限流需要运营审计，所以选择落 MongoDB 文档而不是只在 Redis 里 `INCR`。缓存也只放读多写少、允许短暂陈旧的数据；用户进度、支付状态和奖励事实不能因为缓存失效或过期而成为唯一来源。

分布式锁只用于“多个副本可能同时做同一轮匹配，且业务上确实需要短时间互斥”的场景。唯一索引、单文档原子更新和 CAS 通常更容易证明正确。中间件越多，故障模式、监控和运维成本也越多；没有可量化的读压力、跨节点协调需求或持久化异步任务，就不应该为了简历把它加进来。

## Redis 分布式锁具体怎么保证不会误解锁？

**候选人：** 获取锁使用随机 `lockValue`、`SET key value NX EX`；拿到锁后必须 double-check 数据状态。释放时不能无条件 `DEL`，而要用 Lua 原子判断 value 仍是自己的，再删除；业务执行时间可能超过 TTL 时，要么设置合理的上限和失败恢复，要么续租，并让业务带 fencing token 防止旧持有者在锁过期后继续写。

因此文章里的简化代码表达的是“获取 + double-check”核心，完整实现还应包含安全释放、超时、异常路径和指标。锁不是正确性的唯一来源：匹配结果还要靠唯一键和状态机幂等。对于赛季阶段推进，CAS 比锁更合适；对于 `K8s CronJob`，调度层的 `concurrencyPolicy: Forbid` 负责常规互斥，但任务仍必须幂等，因为手工重跑、多个集群或调度故障都可能打破这个假设。

## 缓存如何保证一致性？为什么不把所有用户数据都缓存？

**候选人：** `UserRepo` 的读路径用 `@Cacheable`，写路径统一 `@CacheEvict`；绕过仓库使用 `MongoTemplate` 批量更新时，必须显式调用清理入口。缓存 key 带版本和产品/环境前缀，设置短 `TTL`，并把 MongoDB 作为最终事实来源。

缓存策略要先回答“允许多旧”和“谁负责失效”。高频读取、低频修改的用户摘要和榜单适合缓存；客户端存档、支付、奖励和风控状态写入频繁且不能读到旧值，就不适合简单缓存。还要考虑缓存击穿、穿透、雪崩、热点 key 和 `allEntries` 清空带来的放大效应。规模上升后可以使用本地 `Caffeine` + Redis 两级缓存、按用户或版本失效、请求合并和预热，但前提是先用指标证明瓶颈在哪里。

## 断网同步为什么用客户端时间戳？它安全吗？

**候选人：** 这个时间戳解决的是离线场景的“迟到写覆盖新写”问题，不是安全认证。服务端以 `(userId, key)` 为条件做原子 upsert，只接受不存在旧时间戳或 `incoming.timestamp` 更大的写入；断网期间产生的旧数据回来后会被静默丢弃。管理端时间戳为零是明确的强制覆盖通道。

客户端可以篡改时间戳，设备时钟也可能漂移，所以它不能单独防作弊。对于货币、购买和奖励，仍然需要服务端事实记录、原子增量和幂等交易；如果业务允许更强的一致性，应把客户端时间戳升级为服务端发放的单调版本号或 `ETag/CAS`，对相同版本的冲突返回明确结果，而不是只依赖 LWW。

## 支付回调为什么用唯一索引？实现中有什么容易被追问的坑？

**候选人：** 回调的 `outTradeNo` 或渠道交易号建唯一索引，直接插入比“先查再插”少一个竞态窗口；重复回调撞 `DuplicateKeyException` 后，可以根据已保存状态安全应答。交易状态机还要区分 `IN_PROGRESS`、验签失败、用户不存在、已发放等状态。

这里有一个不能回避的细节：如果接口在验签前就把任意请求以“已处理”状态占住，理论上恶意请求可能抢先污染幂等键，使后续合法回调被误判为重复。因此更稳妥的顺序是先做基础格式和验签，再以唯一键写入；或者把记录标为 `PENDING`，重复请求根据状态重试/继续处理，只有确认合法且完成发放后才返回最终成功。唯一索引解决并发，不等于解决验签、状态机和重试语义。

## WebSocket 为什么用 Redis Pub/Sub 桥接？它的缺点是什么？

**候选人：** 托管 RabbitMQ 只提供 AMQP，客户端使用原生 WebSocket，不需要引入 STOMP broker relay；Spring 的 simple broker 保留在应用内，每个副本收到 Redis 频道消息后再投递给本地订阅者。这样接入成本小，聊天和实时战斗转发可以水平扩展。

Redis Pub/Sub 是即时转发，不保存消息；副本断线期间的消息会丢，不能用它承载支付结果或奖励事实。所以站内通知先写入可查询的 `UserEvent`，重新连接时可以回放；聊天若允许丢失可以接受，否则应改为 Redis Streams、RabbitMQ 持久队列或专用消息系统，并定义消息顺序、重放和消费位点。这个取舍要按消息价值回答，而不是笼统地说“Redis 更快”。

## “客户端是真相，服务端很薄”会不会带来安全问题？

**候选人：** 会，所以“薄”不能等于“信任客户端”。客户端负责离线体验和大量状态计算，服务端只抽取必要字段并校验方程、阈值和版本；但支付、奖励、点赞计数、封禁和交易状态仍由服务端掌握，客户端只能请求动作，不能直接提交最终计数。加密传输也只解决窃听和篡改传输，不能证明客户端本身可信。

这套方案适合已有客户端数据模型或低风险字段，不适合把高价值资产完全交给客户端。若作弊损失上升，演进方向是服务端权威资源账本、服务器签发的快照/版本号、关键操作事件化以及回放校验，而不是继续增加一层加密或再加一个缓存。

## 这么多模块会不会是“造火箭”？你会怎样判断是否过度设计？

**候选人：** 有这个风险。WASM、AI Agent、MCP、延迟消息、动态索引、双车道机器人等并非所有游戏都需要。判断标准应是业务约束和可验证收益：多产品复用是否真的降低发版成本，活动是否真的需要动态排序，延迟任务是否必须跨重启，运营是否真的需要在线配置和审计；再用 QPS、P95、队列堆积、缓存命中率、故障恢复时间和发布频率验证结果。

如果规模很小，我会删掉跨副本 Pub/Sub、复杂热替换或动态索引，先用 MongoDB 原子更新、唯一索引和 `K8s CronJob` 交付。即使保留复杂组件，也要把它们放在边界清晰的模块里，提供降级路径、监控和回滚，而不是让核心业务到处依赖它们。

## 如果产品、运营和研发对“新活动不写代码”理解不同，你怎么沟通？

**候选人：** 我不会直接承诺“任意活动都能配置”。先拿一个具体活动把规则拆成状态、时间、参与条件、排序字段、结算、补偿和异常处理，确认哪些是引擎已经支持的，哪些需要新增能力；再给运营提供配置校验、预览、dry-run、审计和回滚。模板与实例快照的边界也要提前说明：模板改动影响下一实例，不能悄悄改写正在进行的赛季。

技术方案沟通时，我会同时讲成功路径和失败路径：消息重复怎么办、配置解析失败回退什么、结算重跑会不会重复发奖、数据库不可用时是否允许继续请求。会议后把不变量、接口样例和未决问题写下来，用测试或演练验证，而不是只在群里说“应该没问题”。

## 如果流量突然增长十倍，你先看什么？会不会马上加中间件？

**候选人：** 先看证据：接口 QPS、P95/P99、Mongo 慢查询和连接池、索引命中、Redis 内存和热点 key、RabbitMQ 队列深度与消费延迟、WebSocket 连接数、Pod CPU/内存和 GC。然后区分读热点、写冲突、后台任务堆积还是外部依赖变慢。

对应措施可能是补索引和分页、拆分高写集合、批量处理、限制并发、调整消费者数量、隔离任务 Pod、缓存读多写少数据或对热点请求做合并。只有当现有模型确实不适合时，才引入新的存储或消息系统；“流量变大所以换 Kafka/上分布式事务”不是容量分析。

## 这篇文章里没有使用哪些中间件？你知道它们适合什么地方吗？

**候选人：** 知道用途不等于应该在当前项目使用。可以这样回答：

| 中间件或方案                   | 我会考虑使用的场景                                                  | 当前没有使用的原因                                                                         |
| ------------------------------ | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| `Kafka/Pulsar`                 | 用户行为、埋点、CDC、审计事件和数据分析，需要高吞吐、分区顺序和回放 | 当前核心消息是延迟命令，不需要长期回放；RabbitMQ 更直接                                    |
| `Redis Streams`                | 需要持久化、消费组、重放和消费位点的通知或轻量任务                  | WebSocket 转发允许即时丢失；持久通知已经落 `UserEvent`                                     |
| `Elasticsearch/OpenSearch`     | 运营后台全文检索、复杂日志检索、用户反馈搜索                        | MongoDB 已覆盖结构化查询；没有足够搜索规模就不增加双写和一致性成本                         |
| `ClickHouse/Doris`             | 埋点、广告、付费和 RFM 的大规模聚合报表                             | RFM 与运营查询目前是业务任务和 MongoDB 查询，尚未达到分析型数据库的必要规模                |
| `Temporal/PowerJob/XXL-JOB`    | 长流程、补偿、人工审批、可视化重试和跨服务工作流                    | 现有延迟任务由 RabbitMQ、批任务由 `K8s CronJob` 承担；流程复杂度还没高到需要工作流引擎     |
| Saga/outbox/事务消息           | 订单、库存、奖励拆成多个服务，要求跨服务最终一致                    | 当前主要数据在一个 MongoDB 边界内，没有跨服务全局事务                                      |
| `Nacos/Apollo`                 | 多服务共享配置、灰度发布、配置变更推送和权限审计                    | 当前配置已经进入 `ObjectConfig`，且产品隔离由配置文件和 profile 完成；再加配置中心收益有限 |
| `OpenTelemetry + Jaeger/Tempo` | 多服务调用链、跨队列 trace、定位长尾延迟                            | 现有日志、指标和 Grafana 先满足基础观测；跨服务变复杂后应补分布式 tracing                  |
| `Caffeine` 两级缓存            | 本地热点、低延迟读和减少 Redis 网络访问                             | 需要处理本地副本间失效广播；当前短 TTL 的 Redis/应用缓存已够用                             |
| `Bloom Filter`                 | 用户、昵称、优惠券等大量不存在查询，防止缓存穿透和数据库压力        | 数据规模和查询压力未证明需要；错误率、重建和删除语义也要先设计                             |
| `Kong/APISIX` 或独立 WAF       | 统一入口鉴权、限流、灰度、恶意流量拦截                              | K8s、Istio 和应用内的角色/业务限流已经覆盖当前边界；它们不是重复安装越多越好               |

另外，服务发现、负载均衡和部分治理能力已经由 `K8s Service + Istio` 提供，因此没有再引入一套应用层注册中心。对象存储、CDN、密钥管理、WAF 等也可以在规模和合规要求上升后补上，但要先明确数据类型、故障责任和运维归属。

## 最后，如果让你重新做一次，你会优先改什么？

**候选人：** 我会先补三类可证明正确性的东西：第一，所有关键异步链路都明确“数据库事实、消息状态、重试、死信和对账”的关系，重要任务考虑 `outbox`；第二，分布式锁补齐安全释放、续租和 fencing token，能用唯一索引或 CAS 的地方继续去锁；第三，为支付、奖励、用户存档和配置变更补齐状态机、审计、回滚和故障演练。

然后再根据真实指标决定是否引入 Kafka、搜索引擎、分析型数据库或工作流引擎。我的判断原则是：先把业务不变量和失败语义讲清楚，再选择中间件；先让系统可观测、可重试、可回滚，再追求更高吞吐。

## 深入追问：数据库、索引与一致性

### MongoDB 的“单文档原子性”到底覆盖什么？

**面试官：** 你反复提到单文档原子更新。请不要只说“MongoDB 单文档操作是原子的”，具体解释它保证了什么，又没有保证什么。

**候选人：** 单文档的一次写操作，例如 `updateOne`、`findOneAndUpdate` 或带条件的 upsert，可以把一个文档内的多个字段更新作为一个不可分割的数据库操作提交。并发请求不会看到一个请求只更新了半个文档的中间状态；如果我用 `$inc`、`$set`、`$addToSet` 这类更新表达式，也不会先把整个文档读到应用内再保存，从而避免覆盖掉别的请求刚写入的字段。

但它不等于“业务操作天然安全”。第一，它只覆盖这次数据库操作涉及的文档；同时修改用户文档、订单文档和奖励事件仍然是多文档问题。第二，它不保证客户端请求是可信的；一个原子地把钻石加一百的请求仍然可能是作弊。第三，它不保证消息、缓存和外部支付平台同步成功；MongoDB 提交成功后，Redis 失效或 WebSocket 推送仍可能失败。

在本项目中，点赞使用 `$inc`，联盟节点超时释放使用带状态和时间条件的更新，活动进度使用唯一索引和 upsert。这些场景的优点是操作短、冲突边界清楚、吞吐高；缺点是业务必须被建模成单文档状态转移，复杂跨文档流程不能硬塞进去。因为用户存档、节点状态和计数的大多数不变量都可以收敛到一个文档，所以单文档原子操作的优点大于缺点；只有账号绑定、迁移等确实跨文档的流程才进入事务或补偿流程。

### MongoDB 多文档事务的底层语义是什么？为什么需要副本集？

**面试官：** 你说项目用了 MongoDB 事务。它和 MySQL 的事务完全一样吗？为什么单节点 MongoDB 还要配置副本集？

**候选人：** 不能说完全一样。MongoDB 多文档事务需要在支持事务的部署模式上运行，副本集是最常见的基础条件；本地环境也要以单节点副本集启动，否则驱动会直接认为当前部署不支持事务。事务里多个写入对外表现为一次提交或回滚，但它仍然受到事务生命周期、锁竞争、网络重试和读写关注级别的约束。

从实现角度看，事务会在会话上绑定事务状态，数据库为事务提供一致的读视图，并在提交时协调涉及的文档。事务期间不能无限制地扫描和处理大榜单，因为锁、快照和 oplog 保留都有成本；超过 transactionLifetimeLimitSeconds 或遇到瞬时错误时，应用还要判断是否可以安全重试。读写关注决定“提交成功”意味着只写到主节点、写入多数副本，还是具备更强的持久性保证。

它的优点是能把同一个 MongoDB 边界内的账号映射、迁移记录等多文档变更做成原子提交，开发者容易理解；缺点是吞吐和延迟都比单文档更新差，长事务还会放大资源占用。因此项目没有把整个活动榜结算包进一个超长事务，而是把奖励事件和已结算标记设计成可重试的幂等步骤。这个取舍的关键不是“事务高级所以都用”，而是事务边界必须小到能在故障和重试下解释清楚。

### 什么是 MongoDB 索引的 ESR 原则？这个项目如何用它设计索引？

**面试官：** 你提到复合索引。请从查询执行角度解释 Equality、Sort、Range，以及为什么字段顺序不能随便写。

**候选人：** ESR 是设计复合索引时的一个实用启发：等值条件通常放前面，排序字段尽量在能被索引直接提供排序的位置，范围条件通常放后面。比如活动查询先按 activityId、group、cycle 做等值过滤，再按配置的 data.score 排序，索引就应该优先覆盖前缀等值字段和排序字段；如果把一个低选择性的范围条件放在前面，后续排序可能无法充分利用索引。

但 ESR 不是机械公式。我要看真实查询、字段基数、排序方向、数据分布和 explain 结果。索引会占内存和磁盘，也会让每次写入维护更多 B-Tree；索引太多时，查询优化器未必总能选到理想索引。项目里的通用活动索引之所以按活动动态创建，是因为排序字段由运营配置决定，并且用 partial filter 把索引限制在对应 activityId，减少全集合索引膨胀。

选择动态索引的优点是查询可以直接走索引排序，避免把大批文档拉到 Java 内存里排序；缺点是活动配置变化会产生旧索引，当前启动器只 create-if-missing，不自动删除，长期会造成写放大和运维债务。因为活动排序是读多、字段相对稳定，而且排行榜延迟比索引管理成本更敏感，所以当前阶段优点大于缺点；如果活动数量和配置变更频率继续上升，我会增加索引生命周期管理和上线前的 explain 验证。

### 唯一索引为什么可以当作并发控制？它和“先查再插”差在哪里？

**面试官：** 你说重复报名、重复支付回调、重复发奖用唯一索引。请解释竞态窗口。

**候选人：** “先查再插”实际是两个操作：请求 A 查不到，请求 B 也查不到，然后 A、B 都插入。即使两个请求之间只有几微秒，也存在竞态。唯一索引把判断和写入交给数据库，两个并发插入中只有一个成功，另一个得到 DuplicateKeyException。应用把异常映射成“已存在/幂等命中”，就不用依赖应用线程之间的锁。

它的优点是约束由最终事实来源维护，进程重启和多副本都有效；缺点是索引字段必须准确表达业务不变量，异常处理也要区分“正常重复”和“数据损坏”。例如通用活动进度按 activityId、userId、instanceCycle 唯一，而用户数据按 activityId、userId、cycle 唯一，点赞接口如果误用 instanceCycle 查询，就会把唯一索引和业务语义错开。

所以唯一索引不是万能幂等。支付还要验签和比对金额，奖励还要区分事件是否已领取，状态机还要防止旧状态覆盖新状态。它解决的是并发下“只能有一份事实”，不解决这份事实是否可信、是否应该进入下一状态。

### 为什么“查出对象后修改再 save”容易丢更新？如何用 CAS 修复？

**面试官：** 联盟阶段推进当前就是读后 save。请你现场解释一个丢更新例子，再给出改进方案。

**候选人：** 假设两个 API 副本都读到 phaseIndex=3。副本 A 计算要进入 4，副本 B 也计算要进入 4；A 先保存，B 随后保存。结果虽然看起来还是 4，但 A 可能已经计算并写入了下一阶段的附加字段，B 的旧对象会把这些字段覆盖。更严重的是，如果两边依据不同时间判断，可能各自发送后续结算消息。

更可靠的做法是把期望版本放进写条件：

```java
var result = template.updateFirst(
    query(where("_id").is(seasonId)
        .and("phaseIndex").is(expectedIndex)
        .and("phaseStartedAt").is(expectedStartedAt)),
    new Update().set("phaseIndex", nextIndex)
        .set("phaseStartedAt", nextStartedAt),
    AllianceSeason.class);
if (result.getModifiedCount() == 1) {
  // 只有真正抢到这次状态转移的请求才继续发送后续消息
}
```

或者给文档加 @Version，让 Spring Data 在保存时用版本号做乐观锁。CAS 的优点是不需要额外 Redis 锁，冲突时失败快速、事实写入和条件绑定在数据库；缺点是冲突高时会反复重试，复杂的“推进一个阶段并创建多个副作用”仍然需要 outbox 或补偿。当前代码的期望阶段检查和 jobs 对账已经能吸收重复消息，但从严格并发证明看，还应该补原子条件更新。

### 活动结算为什么不使用一个大事务？幂等结算如何避免重复发奖？

**面试官：** 不用大事务会不会出现“奖励发了但进度没标记”的脏状态？

**候选人：** 会出现中间状态，所以不能假设流程一次成功。当前设计接受这个中间状态，把“写奖励事件”和“标记进度已结算”拆成可重跑步骤。结算前按活动、用户、周期、榜单和日期检查是否已有相同奖励事件；创建事件成功后，即使进度标记暂时失败，下一次任务也会看到事件已存在而不会再发一份。日榜还用 leaderboardId:dateKey 作为结算 key，并保留一段时间吸收任务重跑。

大事务的优点是成功或失败看起来更整齐；缺点是排行榜可能很大，事务时间接近或超过 MongoDB 生命周期限制，失败时回滚成本也高。拆分结算的优点是每个用户的小步骤短、单个失败可以重试、任务可以按窗口恢复；缺点是必须设计事件唯一性、补偿和运营查询。因为结算的核心不变量是“同一榜单同一用户同一周期最多一份奖励事实”，而不是“所有用户在同一瞬间提交”，所以短步骤幂等的优点大于大事务。

### CAP、BASE 和“最终一致”在这个项目中分别对应什么？

**面试官：** 不要只背 CAP 三个字母。请结合本项目说清楚你放弃了什么一致性、保留了什么一致性。

**候选人：** CAP 讨论的是发生网络分区时，分布式系统不能同时保证强一致和可用性。它不是说一个系统平时只能选两个字母，也不是数据库产品的固定标签。项目中 MongoDB 作为事实来源，支付、奖励和用户身份更偏向一致性；Redis Pub/Sub 和缓存更偏向可用和低延迟，允许短暂丢消息或读到旧值，但不能让它们取代支付事实。

BASE 可以帮助描述异步业务：基本可用、软状态、最终一致。例如 UserEvent 写入后，WebSocket 或 APNs 可能稍后才到；活动结算先产生事件，再由客户端读取和确认。这里的最终一致不是“最后一定会好”，而是必须有重试、对账、死信、人工补发和监控，才能把暂时不一致收敛。

选择这种分层的优点是关键资产不依赖缓存和广播，普通通知又不必被强一致拖慢；缺点是开发者必须在每条链路上标注事实来源、允许的延迟和补偿方式。因为不同数据的价值不同，统一要求所有链路强一致反而会增加复杂度和故障半径，所以分层一致性的优点大于缺点。

## 深入追问：Redis、缓存与分布式锁

### Redis 为什么快？“Redis 是单线程”这句话到底对不对？

**面试官：** 这是一个典型八股。请从事件循环、内存数据结构和并发模型解释，不要只说“因为在内存里”。

**候选人：** Redis 的核心命令执行路径通常由一个事件循环串行处理，避免了多个线程同时修改同一份数据结构时的大量锁开销；数据主要在内存中，避免了每次命令都触发磁盘随机读；网络层使用 I/O 多路复用，让一个线程能管理许多连接。像 ZSET、哈希表、跳表等数据结构针对不同访问模式做了专门优化。

“单线程”是不完整的说法。现代 Redis 可能使用 I/O 线程处理读写，持久化、异步删除等工作也可能由后台线程或子进程承担。命令执行逻辑仍然要求串行；一个大 key、KEYS、超大范围 ZRANGE 或同步删除大对象仍会阻塞命令执行线程，导致所有请求的 P99 一起变差。

它的优点是低延迟、原子命令和丰富数据结构；缺点是内存成本高、单线程执行长命令会形成全局阻塞、主从和持久化也有额外故障语义。项目把 Redis 用在短 TTL 缓存、ZSET 周榜、锁和 Pub/Sub，是因为这些访问模式能利用它的优点；用户存档、支付和奖励事实不放 Redis，是因为持久性、审计和恢复能力的缺点会超过收益。

### Redis 的 ZSET 为什么适合周榜？它内部大致如何工作？

**面试官：** 你为什么不用普通 Set 或 Mongo 排序？请讲清楚有序集合的复杂度和边界。

**候选人：** ZSET 同时维护 member 到 score 的映射和按 score 排序的结构，常见实现可以理解为哈希表加跳表。哈希表便于按 member 更新分数，跳表便于按分数范围和排名读取。添加、更新通常是对数级别，读取 top N 也可以只遍历需要的范围。

周榜的分数模型稳定，写入后马上要读 top N，而且 key 可以在周期结束时整体过期。Redis ZSET 的优点是读写模型直接、无需每次扫描 Mongo 文档、支持按周期自然隔离；缺点是它是另一份状态，Mongo 写成功而 ZSET 更新失败时会不一致，还要处理同分排序、重建和内存峰值。

活动榜没有统一用 ZSET，是因为活动排序字段可能由运营配置，数据还可能临时合成机器人。此时 Mongo 的真实数据和动态索引更容易回源，维护 ZSET 反而增加双写和重建复杂度。周榜的固定指标满足“高频更新、固定排序、周期淘汰”，所以优点大于缺点。

### 缓存穿透、击穿、雪崩分别是什么？这个项目如何应对？

**面试官：** 请不要把三个词混在一起，并说明你不会为了背概念就把所有方案都加上。

**候选人：** 缓存穿透是请求大量不存在的 key，缓存和数据库都查不到，压力直接落到数据库；缓存击穿通常指一个热点 key 过期瞬间，大量请求同时回源；缓存雪崩是大量 key 在相近时间过期或缓存整体故障，导致数据库被集中打穿。

应对穿透可以用参数校验、负缓存或 Bloom Filter；应对击穿可以用互斥重建、请求合并、逻辑过期；应对雪崩可以使用 TTL 加随机抖动、分批预热、限流和多级缓存。每种方案都有成本：负缓存会暂时保存“不存在”，Bloom Filter 有误判且删除困难，分布式互斥会增加等待，逻辑过期会接受旧数据，多级缓存还要解决本地副本失效。

本项目先采用 UserRepo 的统一缓存清理、版本化 key 和 TTL，把 Mongo 作为事实来源；没有在每个查询上强行加 Bloom Filter 或分布式重建锁。优点是实现简单、失效路径可审计；缺点是 allEntries 清理可能造成短时回源，热点增长后还需要请求合并和本地缓存。因为当前已知读热点规模没有证明必须引入更复杂方案，所以简单策略的优点大于缺点；上线后用命中率、回源 QPS 和 Mongo P99 决定是否升级。

### 缓存一致性有哪些常见方案？为什么本项目主要用 Cache-Aside？

**面试官：** 请比较 Cache-Aside、Read/Write Through、Write Behind 和发布订阅失效。

**候选人：** Cache-Aside 是应用先查缓存，未命中查数据库并回填；写入时先写数据库，再删除缓存。它的优点是应用明确知道哪些数据可缓存，数据库仍是事实来源，失败时容易回源；缺点是读写之间存在短暂旧值，删除失败还会留下脏缓存。

Read Through 把回源逻辑放进缓存层，业务代码更薄，但要依赖缓存产品或框架提供一致的加载语义；Write Through 同时写缓存和数据库，读取简单但写路径更重；Write Behind 先写缓存后异步落库，吞吐高却会把缓存变成暂时事实，掉电和顺序问题都更难。发布订阅可以广播失效，适合多副本本地缓存，但 Pub/Sub 本身不保存失效事件，订阅者断线就可能漏掉。

UserRepo 采用 Cacheable + CacheEvict，批量 MongoTemplate 直写后暴露显式清理入口，本质是偏保守的 Cache-Aside。优点是支付、奖励、用户存档不会因缓存写丢失而变成错误事实；缺点是缓存清理粒度粗，读延迟和回源压力需要监控。因为这些用户数据允许缓存加速但不允许缓存主导正确性，所以 Cache-Aside 的优点大于缺点。

### Redis 分布式锁为什么一定要带 value？TTL 到期后旧客户端会发生什么？

**面试官：** 假设客户端 A 获得锁后卡顿，锁过期，客户端 B 获得锁；这时 A 恢复并执行删除，会发生什么？

**候选人：** 如果 A 直接执行 DEL key，它会把 B 的锁删除，导致 B 以为自己仍然持有锁。这就是 value 的意义：A 获取锁时生成随机 token，释放时用 Lua 脚本原子判断当前 value 是否仍等于自己的 token，只有相等才删除。

但安全释放还不够。A 的业务操作可能已经超过 TTL，锁虽然没了，A 仍可能向数据库写入旧结果。续租只能降低概率，不能从根本上证明旧客户端不会继续写；更强的做法是给每次成功获取锁分配递增 fencing token，数据库写入时要求 token 不小于文档中的最新 token，旧客户端即使恢复也会被拒绝。

项目里的联盟匹配锁主要用于减少同一赛季的并发建组，并由数据库状态和唯一约束兜底，因此它不是唯一正确性来源。锁的优点是把高概率重复计算挡在前面，缺点是 Redis 故障、时钟/TTL、客户端暂停都会带来复杂边界。因为匹配本身可以通过状态检查重试，短 TTL 锁的优点大于缺点；如果锁直接保护支付扣款或资产转移，我不会只依赖这种实现。

### Redlock 能不能彻底解决分布式锁问题？

**面试官：** 你知道 Redlock 吗？为什么不直接说“上 Redlock 就安全了”？

**候选人：** Redlock 试图在多个独立 Redis 节点上取得多数锁，并用时间窗口判断锁是否仍然有效。它比单实例锁增加了故障容忍思路，但它仍然依赖网络延迟、进程暂停、时钟和客户端对“租约有效期”的判断，不能自动把一个外部系统变成线性一致的锁服务。

如果真正需要强互斥，我会先问业务是否能改成数据库 CAS、唯一索引或幂等状态机；如果不能，再评估 ZooKeeper/etcd 这类带一致性协议的协调服务，并在资源写入侧加入 fencing token。Redlock 的优点是部署在 Redis 生态里、对部分节点故障有容忍；缺点是解释成本和误用风险都高。当前联盟匹配只需要短时减少重复工作，并非资产一致性的唯一屏障，所以不引入 Redlock；这是因为简单方案的优点大于其缺点，而不是不了解 Redlock。

## 深入追问：RabbitMQ、消息语义与分布式事务

### RabbitMQ 的 exchange、queue、binding 分别是什么？为什么不能直接把消息发给消费者？

**面试官：** 请从消息进入 broker 到消费者收到的路径讲一遍。

**候选人：** 生产者把消息发布到 exchange，exchange 根据 routing key 和 binding 规则把消息路由到一个或多个 queue，消费者再从 queue 拉取或被推送消息。消费者实际消费的是 queue，不是 exchange。这个间接层让同一条消息可以按不同 binding 分发，也让生产者不需要知道具体消费者实例。

项目使用 durable 的 exchange、queue、binding，并把产品、环境和用途编码到队列名中。延迟交换机插件通过消息的 x-delay header 让消息在未来才进入目标队列；队列上的 DLX/DLQ 则负责处理拒绝或重试耗尽的消息。队列名没有默认值，启用 RabbitMQ 却没配置隔离队列时 fail-fast，是为了避免多个产品共享 broker 时误消费。

这个方案的优点是路由、持久化、消费者竞争和死信语义由 broker 提供；缺点是队列声明、插件、命名和权限都成为运维依赖。因为项目确实有跨重启的延迟命令和多副本消费需求，RabbitMQ 的优点大于额外运维成本。

### Publisher Confirm、Consumer Ack 和业务成功有什么区别？

**面试官：** 如果 publisher confirm 成功了，是否说明业务一定成功？如果消费者 ack 了，是否说明奖励一定发出？

**候选人：** 不是。Publisher Confirm 只说明 broker 接受了发布并按当前配置确认，不等于消费者已经消费，更不等于业务写库成功。Consumer Ack 只表示消费者告诉 broker 可以从队列中确认这条消息；如果消费者在 ack 前写库失败，消息可以重投；如果业务写库成功后进程在 ack 前崩溃，消息也可能重投，所以业务必须幂等。

反过来，如果先 ack 再写库，业务进程崩溃会导致消息丢失。项目的 listener 配置让失败消息不重新入队或进入重试/DLQ，结算服务还用奖励事件和状态检查吸收重复。阶段/机器人生产路径明确等待 confirm，但结算的 x-delay=0 投递没有统一等待 confirm，这个差异应该在面试中主动说出，而不是把所有生产者都描述成同样可靠。

Publisher Confirm 的优点是能发现 broker 层的发布失败，Consumer Ack 的优点是把消费完成边界交给业务；缺点是两者之间仍有多个故障窗口。因为它们分别解决生产可靠性和消费确认，组合“幂等业务 + 重试 + DLQ”比幻想 exactly-once 更现实，所以这种方案的优点大于缺点。

### 延迟消息为什么不是定时任务的 exactly-once？

**面试官：** 延迟消息到点后可能重复、提前或延迟吗？你如何设计业务才能接受这种不确定性？

**候选人：** 延迟消息表示“尽量在某个时间后投递”，不表示只投递一次、精确在某一毫秒执行。broker 重启、消费者积压、网络重连和重试都可能导致实际执行时间偏移；同一消息在业务成功写库但 ack 丢失时也可能再次到达。

因此消息体必须带业务上下文和期望版本，例如联盟阶段消息带 seasonId、expectedPhaseIndex、expectedPhaseStartedAt。消费者重新读取数据库，检查当前状态是否仍匹配；重复消息变成 no-op，过期消息不覆盖新状态；执行成功后还要有对账 tick 检查“状态已到边界但没有下一条消息”的情况。

RabbitMQ 延迟消息的优点是可以持久化未来命令并跨副本消费；缺点是时间精度、插件依赖和至少一次语义。因为业务状态机可以把重复和延迟变成可接受的输入，RabbitMQ 仍然适合当前需求；如果是毫秒级定时、海量时间轮或复杂工作流，我会重新评估专门调度系统。

### RabbitMQ 和 Kafka 应该怎么选？“高吞吐所以 Kafka”为什么不够？

**面试官：** 请给出一套不依赖品牌偏好的选择思路。

**候选人：** 我先看消息本质。Kafka 更像持久化的分区日志：强调顺序追加、吞吐、消费者组位点、长期保留和多下游回放；RabbitMQ 更像路由和任务 broker：强调队列、确认、路由、重试、死信和工作分发。两者都能传消息，但优化目标不同。

当前机器人的攻击、阶段推进和结算主要是“未来执行一次的命令”，需要延迟投递、消费确认和 DLQ，不需要长期回放所有历史消息。RabbitMQ 的优点是模型直观、任务语义匹配；缺点是延迟插件和 broker 运维依赖。Kafka 的优点是未来做埋点、行为事件、CDC 和分析回放时吞吐和保留能力强；缺点是把单次任务、延迟和重试做得自然需要额外设计。

因此不是 RabbitMQ 永远优于 Kafka，而是当前消息的工作负载让 RabbitMQ 的优点大于缺点。若未来要做用户行为数据平台，我会让 Kafka 承担事件流，而不是为了分析需求把当前所有命令队列迁过去。

### 为什么不使用 Seata 或两阶段提交解决 Mongo、Redis、RabbitMQ 的一致性？

**面试官：** 如果 Mongo 写成功、Rabbit 发消息失败，Seata 能不能解决？

**候选人：** 首先要区分参与者是否支持同一种分布式事务协议。Mongo、Redis、RabbitMQ、外部支付平台的事务能力和提交模型不同，强行做全局两阶段提交会引入协调者、锁持有、超时和恢复日志；外部支付平台通常也不会参加我的全局事务。

两阶段提交的优点是抽象上接近“全部成功或全部回滚”，对少数受控参与者很有吸引力；缺点是协调者故障会让参与者长期等待，网络分区下可用性差，事务时间长还会锁住资源。Seata 更适合有明确支持边界的关系型微服务事务，并不是跨任何组件的魔法开关。

当前项目优先使用同一 Mongo 边界内的短事务，跨边界用数据库事实、幂等键、消息重试、DLQ 和对账。若 Mongo 写成功后需要发 Rabbit，我会考虑 transactional outbox：在同一个 Mongo 事务里写业务状态和 outbox 记录，再由发布器可靠投递并标记已发布；消费者用 inbox/唯一键幂等。outbox 的优点是把“业务提交”和“待发送事实”绑定，缺点是增加集合、发布器和清理任务。因为当前跨边界一致性比引入全局协调器更适合最终一致，outbox/Saga 的优点大于 2PC。

### outbox、inbox 和 Saga 各解决什么问题？

**面试官：** 请不要把它们都说成“最终一致性方案”。

**候选人：** Outbox 解决的是本地数据库提交和消息发布之间的双写问题：业务事务里同时写状态和 outbox，发布器重试 outbox。Inbox 解决的是消费者重复收到消息时的幂等：先用消息 id 或业务键记录已处理，再执行业务，或让唯一索引直接承担去重。Saga 解决的是跨多个服务的长流程：每一步提交本地事务，失败时执行补偿动作，而不是保持一个全局长事务。

例如订单、库存、奖励拆成三个服务时，订单服务写订单和 outbox；库存服务收到消息后用 inbox/唯一订单号扣库存；奖励服务在库存成功后发放，失败时由 Saga 状态和补偿重试推进。补偿不是数据库回滚，它可能需要退款、释放库存或人工介入。

Outbox/inbox 的优点是依赖普通数据库和消息系统、故障窗口可观测；缺点是数据会暂时重复、发布器和清理都要运维。Saga 的优点是长流程可拆分、不会长时间占住全局锁；缺点是补偿逻辑复杂、业务需要接受中间状态。当前项目还没有拆成多个订单/库存服务，因此没有提前引入全套；如果服务边界出现，优点才会大于缺点。

## 深入追问：Spring、Java、网络与业务安全

### Spring Bean 从定义到可用经历了什么生命周期？

**面试官：** 你用了很多 Spring 注解。请解释 BeanDefinition、实例化、依赖注入、初始化和代理的大致顺序。

**候选人：** Spring 先扫描配置类、组件和自动配置，形成 BeanDefinition；之后根据依赖关系实例化对象，执行构造器注入、字段或 setter 注入，再调用各种 aware 接口和 BeanPostProcessor。初始化阶段会执行 PostConstruct、InitializingBean 或自定义 init 方法；如果某个后置处理器创建了代理，最终放进容器的可能不是原始对象，而是代理对象。

这解释了几个项目里的现象。RabbitMQ 队列名在 PostConstruct 阶段校验，所以缺少隔离配置会快速失败；ConditionalOnProperty 和 profile 在装配阶段决定某个生产者或监听器是否存在；Lazy 自注入则让服务拿到一个可以经过 Spring AOP 的代理，而不是在初始化时直接拿到自己。

Spring 的优点是把基础设施装配、条件开关和横切逻辑统一起来；缺点是运行时代理和自动配置容易让真实依赖隐藏在 classpath 中。项目用 compileOnly、模块隔离和条件配置约束依赖方向，优点大于“所有东西都放进一个 starter 自动装配”的方便。

### 为什么同一个类里直接调用方法会让事务或异步注解失效？

**面试官：** 请从代理机制解释，不要只说“Spring 有 bug”。

**候选人：** Spring AOP 通常在 Bean 外面包一个代理。外部调用代理方法时，代理可以在进入目标方法前开启事务、切换线程或执行拦截器；但同一个对象内部用 this.method() 调用，直接走目标对象自身，不经过代理，所以事务、异步、缓存和自定义切面都可能不生效。

客户端同步抽取字段的方法带 Async，如果 update 方法内部直接调用抽取方法，抽取仍会在请求线程执行。源码通过 Lazy 注入自身代理来调用，才能真正进入异步拦截。活动结算也使用 self proxy 进入需要事务的入口，避免同类自调用绕过事务。

解决方式有三种：把方法拆到另一个 Bean；注入自身代理；或者显式使用编程式事务/任务执行器。拆 Bean 的优点是依赖方向清晰、测试容易，缺点是类数量增加；自注入改动小，缺点是容易产生循环依赖和隐藏耦合；编程式控制最明确，缺点是模板代码多。项目选择自代理是局部修复，优点大于缺点；如果异步边界继续增多，我会优先拆分服务。

### 异步方法和事务方法之间为什么不能默认共享事务？

**面试官：** 假设一个事务方法内部调用异步方法，异步方法能不能直接使用同一个事务？

**候选人：** 通常不能。Spring 事务管理器把数据库连接、事务状态等资源绑定到当前线程；Async 会把任务提交到另一个线程，那个线程默认没有原线程的事务资源。即使把一些上下文复制过去，也不能安全地共享同一个 Mongo session。

因此异步方法应该有自己的事务边界，或者由主线程先提交事实，再让异步任务读取已提交的数据。项目的客户端 blob 写入和镜像抽取就是两个事实边界：原始数据先落库，异步抽取失败不能回滚已经成功保存的 blob，而应该记录告警并重试。

事务的优点是把短小的数据库状态变更绑定起来；异步的优点是削峰和降低接口延迟。把两者强行当成一件事的缺点是失败语义不清、事务资源泄漏和用户以为成功但后台未完成。因为同步提交事实、异步处理派生数据更容易恢复，所以这种拆分的优点大于跨线程共享事务。

### Spring 事务传播和隔离级别如何选择？

**面试官：** 如果一个结算方法调用另一个事务方法，你会使用 REQUIRED、REQUIRES_NEW 还是 NESTED？

**候选人：** 默认的 REQUIRED 表示有事务就加入，没有就新建，适合一组必须共同提交的短操作；REQUIRES_NEW 会挂起外层事务，单独提交，适合审计日志或失败记录必须独立保存的场景，但它会增加连接和提交次数；NESTED 依赖底层 savepoint，不能把它当成跨数据库事务。

项目中的账号绑定和迁移适合一个短事务，让映射和相关文档一起提交；大榜单结算不适合把全部用户包在一个事务里，而是按用户和事件做可重试步骤。对于 Mongo，隔离语义还要结合事务快照、读写关注和部署模式理解，不能直接把 MySQL 的每个隔离级别名称套上去。

选择事务传播的原则是先问“哪些写必须同时成功”，再问“失败后是否可补偿”。事务的优点是局部原子，缺点是锁和生命周期成本；REQUIRES_NEW 能保存故障记录，缺点是外层回滚时它不会跟着回滚。因为不同数据的生命周期不同，统一使用一个传播级别反而危险。

### compileOnly 为什么能帮助模块化？它有什么风险？

**面试官：** 领域模块编译时依赖 Mongo、Redis、WASM，运行时却由 API 或 jobs 提供。为什么不全部使用 implementation？

**候选人：** compileOnly 允许源码编译和测试使用类型，但不会把依赖强行带到该模块的运行时 classpath。这样 arena-power 可以保持纯 Java，arena 的 Spring、Mongo、Redis、wasmtime 依赖由最终应用决定；API 和 jobs 可以选择不同启动形态，领域模块不会隐式启动不需要的自动配置。

它的优点是依赖方向清晰、可复用性好、减少错误自动配置；缺点是如果最终应用忘记提供依赖，问题会在启动或运行时才暴露。项目用 Gradle 的 runtime classpath 检查禁止 Mongo driver、Jedis、Hibernate 漏进 core，并在 CI 中守住这个不变量。若没有这层检查，compileOnly 只是开发者的约定，长期容易被传递依赖破坏。

### Java 21 的虚拟线程和线程池有什么区别？项目为什么在 App Store 客户端使用它？

**面试官：** 虚拟线程是不是可以无限创建？它适合 CPU 密集型任务吗？

**候选人：** 虚拟线程是由 JVM 调度、挂载到少量 carrier platform threads 上执行的轻量线程。遇到支持的阻塞 I/O 时，虚拟线程可以挂起，让 carrier 去运行其他任务，因此适合大量并发、每个任务会等待网络响应的工作。

它不是无限资源，也不是 CPU 加速器。每个任务仍然消耗内存，外部服务连接数、限流和响应时间仍然是瓶颈；CPU 密集型任务最终仍受 CPU 核心数限制。某些 native 调用或特定 synchronized 临界区中可能发生 pinning，让 carrier 被占住，所以要监控。

App Store Connect 需要并发拉取多个商品、分页和地区价格，虚拟线程能让阻塞式 HTTP 调用保持直观代码，同时提高 I/O 并发。优点是迁移成本低、线程栈开销小；缺点是会掩盖下游限流和连接池配置问题。因为这里是 I/O 密集且任务边界清晰，优点大于缺点；战斗计算和批量 Mongo CPU 工作不会因为换虚拟线程就自动变快。

### JVM 内存、GC 和接口变慢如何联系起来？

**面试官：** 如果 P99 从 100ms 突然升到 3s，你如何判断是 GC、线程池还是数据库？

**候选人：** JVM 内存可以粗分为堆、线程栈、元空间和 native/off-heap。对象分配在堆上，短命对象通常在年轻代回收，存活对象晋升；G1 这类收集器按 region 管理堆并尽量控制停顿，但并不承诺任何业务请求都没有 Stop-The-World。线程池队列、堆外缓冲、连接池和 native memory 也可能耗尽，不是只看 heap used。

排查时我会对齐同一时间窗口：GC pause/count、allocation rate、heap occupancy、线程数和线程池队列、Mongo/Redis/Rabbit 的等待、接口分位延迟。若 GC pause 与延迟尖峰重合且 old region 持续增长，可能是对象留存或堆不足；若线程池队列增长但 GC 正常，可能是下游阻塞；若只有 Mongo 查询延迟升高，应看慢查询、连接池和索引。

直接把堆调大有时能延后 Full GC，但也可能让一次回收停更久；盲目加线程会把下游和上下文切换压垮。正确做法是先定位分配和阻塞来源，再调整对象生命周期、分页、批量大小、连接池和资源限制。性能优化的优点必须由指标证明，否则“加内存/加线程”只是把问题推迟。

## 深入追问：HTTP、认证、加密与实时通信

### JWT 和 Session 有什么本质区别？为什么项目选择 JWT？

**面试官：** JWT 是不是无状态？Token 被盗后怎么撤销？

**候选人：** Session 把身份状态保存在服务端，客户端只带 session id；JWT 把 claims 和签名放进客户端 token，服务端验证签名即可得到 subject。JWT 减少了中心 session 存储和跨服务共享 session 的需要，但并不等于整个认证系统无状态：项目每次请求仍会重新加载用户并检查封禁状态，所以封禁可以即时生效。

JWT 的优点是 API 和 WebSocket 连接都容易携带，水平扩展不必共享 session；缺点是 token 一旦签发，在过期前可能持续有效，撤销和密钥轮换需要额外策略，而且不能把敏感数据随便塞进 payload。项目用 HS512，密钥由服务端共享，适合同一受控服务签发和验证；如果多个互不信任的服务只需要验证，我会考虑 RS256/ES256，让验证方只持有公钥。

被盗 token 的处理包括短过期时间、refresh token 撤销、用户封禁检查、密钥轮换、设备绑定和风险重认证。JWT 解决的是签名后的身份声明，不解决设备被控制、请求重放或业务授权；所以 body userId 校验和方法级权限仍然必要。

### RSA、AES-CBC 和 AES-GCM 如何选择？当前加密设计有什么问题？

**面试官：** 既然请求已经 RSA + AES 加密，为什么还说需要 HTTPS？

**候选人：** 非对称加密适合安全传输小的会话密钥，AES 适合高效处理大 body，所以信封加密通常是 RSA 解 AES key/iv，再用 AES 处理正文。当前实现使用 AES-CBC/PKCS5，能够提供机密性，但 CBC 本身不提供认证标签；如果没有独立 MAC，攻击者可能篡改密文并利用错误响应或 padding oracle 类问题。

AES-GCM 是 AEAD，同时提供机密性和完整性，通常更适合作为新协议；它的代价是 nonce 管理必须严格，nonce 重用会破坏安全性。HTTPS 仍然必要，因为它保护传输层的握手、路径、headers、重放环境和整体通信，也能避免中间人替换请求元数据。应用层加密可以满足旧客户端协议或字段级保护，但不能替代 TLS。

当前代码把 RSA 解开的 key/iv 放进带 TTL 和容量上限的缓存，降低重复握手成本；这有性能优点，但缓存内容是高敏感密钥，必须避免日志泄漏并设置合理淘汰。若重新设计，我会选择 AES-GCM、随机 nonce、请求时效/序列号和密钥轮换；保留 HTTPS 和服务端授权。新方案安全性更强，缺点是协议迁移和客户端兼容成本，但在高价值接口上优点大于缺点。

### WebSocket、STOMP 和 Redis Pub/Sub 各自解决什么问题？

**面试官：** 为什么不直接让客户端连 RabbitMQ？为什么 HTTP 层对 /ws 放行却仍然安全？

**候选人：** WebSocket 是客户端和服务端之间的长连接传输，STOMP 在其上定义 CONNECT、SUBSCRIBE、SEND 等消息语义；Spring simple broker 负责本进程内的订阅和目的地路由。Redis Pub/Sub 不是客户端协议，而是副本之间的桥：发布副本把消息发到 Redis，所有副本的 subscriber 再把消息投给自己持有的连接。

托管 RabbitMQ 只提供 AMQP，没有 STOMP broker relay，直接让移动客户端接入 broker 会暴露 broker 凭据、权限和协议细节，也很难把游戏业务鉴权放在正确边界。因此保留应用内 simple broker，再用 Redis 桥接。优点是客户端协议和业务权限仍由应用控制，缺点是 Pub/Sub 不持久化，断线副本会丢即时消息。

HTTP 的 /ws 放行不代表匿名。连接升级后的 JWT 在 STOMP CONNECT 帧 header 中解析，inbound interceptor 对 CONNECT、SUBSCRIBE、SEND 设置 Principal 并执行授权。实时战斗可以丢一条刷新消息，因为客户端可以 REST 重同步；支付和奖励不走 Pub/Sub，而走 UserEvent。不同消息价值使用不同可靠性，是这个设计成立的前提。

### SSRF 是什么？为什么 URL 前缀白名单不够？

**面试官：** 运营代理和用户可注册 MCP server 都接收 URL。你会如何防止访问内网？

**候选人：** SSRF 是服务端替用户发起请求，攻击者把目标指向回环地址、私网、云元数据地址或内部服务，从而绕过网络边界。字符串层面的 allowed.example 检查不够，因为可能有 allowed.example.evil.com、DNS 解析到私网、重定向到内网、IPv6 或整数 IP 表示等情况。

防护至少包括协议 allowlist、精确 host 或域名边界判断、解析所有 A/AAAA 记录并拒绝 loopback、private、link-local、metadata 地址、限制端口、关闭或复核重定向、连接和响应大小超时，并在出网网络层再做一层 egress policy。MCP 还只允许 streamable HTTP，不启用 stdio，避免用户配置在宿主机拉起任意进程。

当前 MCP URL 注册时会尝试连接并 listTools，运行时仍需要重新验证解析结果，且 DNS 检查和实际连接之间存在 TOCTOU 窗口。应用白名单和网络隔离的优点是降低攻击面，缺点是合法内网服务接入需要显式配置和运维配合。因为 Agent 工具拥有更强的外部操作能力，安全限制的优点大于接入便利。

## 深入追问：核心业务链路与安全边界

### 客户端数据是主要事实时，服务端到底校验什么？

**面试官：** 如果 UserProxy 是不透明 JSON，服务端如何防止玩家把钻石改成一百万？

**候选人：** 服务端不试图复制客户端全部 schema，而是在入口抽取少数有业务价值的字段：等级、关卡、VIP、阵容、道具数量等。对于道具，会按运营配置检查阈值；对于客户端提供的校验方程，服务端复算字段关系；异常时保存风控上下文、封禁或告警，并让白名单用户可以人工豁免。

这只能降低风险，不能把客户端变成可信执行环境。客户端可以伪造时间戳、篡改加密前数据、绕过本地逻辑或重放合法请求。货币、购买、奖励和消费统计因此不能只相信 blob，而要由服务端交易、UserEvent、IAPTransaction 和原子计数承担事实。若作弊成本和资产价值继续上升，演进方向是服务端权威账本、服务端版本号、关键操作事件化和战斗回放，而不是再加一层缓存。

### 客户端时间戳 LWW 的并发原理是什么？为什么它不能防作弊？

**面试官：** 两台设备的时钟不一致怎么办？时间戳相同怎么办？

**候选人：** 当前协议按每个 userId/key 保存一个 timestamp，服务端只接受 incoming timestamp 大于旧值的写，旧数据静默丢弃；timestamp 为零是管理端的强制覆盖通道。它解决的是离线重连时“旧写晚到覆盖新写”，本质上是客户端参与的 LWW。

它的优点是协议简单、无需服务端保存每次离线操作、单文档条件更新可以高效执行；缺点是客户端时钟不可信，两个设备可能时间倒退或相同，真正冲突的先后无法由客户端时间证明。相同 timestamp 需要明确拒绝、按设备 tie-breaker，或改用服务端递增版本。

所以这个时间戳只处理一致性排序，不处理授权。高价值操作应使用服务端签发的版本、ETag、CAS、交易 id 和服务端增量；加密只能保护传输，不能证明客户端没有改值。

### 支付回调的正确处理顺序是什么？当前实现的风险在哪里？

**面试官：** 你之前说唯一索引能幂等。请把验签、占位、订单校验和发货顺序完整说一遍。

**候选人：** 推荐顺序是：读取原始 body 和签名 header；先做格式校验和渠道签名验证；根据 outTradeNo 查订单，核对商户号、商品、金额、币种和用户；再用渠道交易号的唯一键插入或推进回调状态；最后在数据库事务或幂等发货动作中创建统一 IAPTransaction/ShopUserPurchase，并返回渠道要求的成功响应。

当前 EezPay 实现的风险是先插入 IN_PROGRESS 回调记录，之后才验签；同一 outTradeNo 再来会直接返回 success。攻击者如果能在合法回调之前抢占交易号，可能让合法回调被误判为重复，造成拒绝服务。更稳妥的改法是先验签再占唯一键，或允许 INVALID_SIGN/IN_PROGRESS 状态按合法请求重新进入状态机，并保存审计。

唯一索引的优点是并发下不需要先查再插；缺点是错误记录也可能占用 key，且支付平台重试会把状态机边界暴露出来。因为支付安全优先级高于少一次数据库写入，验签优先和可恢复状态的优点大于当前“先占位再快速返回”的便利。

### WASM 为什么能改善一致性？为什么仍然不能直接等于防作弊？

**面试官：** 如果服务端执行 WASM，客户端把战斗结果改掉是不是仍然可能作弊？

**候选人：** WASM 的主要价值是把同一套计算逻辑以版本化模块运行，服务端可以使用 wasmtime 执行 power 和 battle；如果客户端也使用同一模块，并且输入、初始资源和随机 seed 一致，预测与服务端校验的漂移会减少。模块通过 key、version 和 enabled 管理，可以先 dry-run，再启用。

但服务端仍然必须相信自己的输入和执行结果，而不是客户端传来的最终分数。客户端可以伪造阵容、seed、回合结果或请求身份；WASM 也可能有资源加载、版本切换、内存释放、恶意模块和并发热替换问题。服务端应记录规则版本、输入摘要、seed、输出和战斗日志，必要时用固定样例和回放验证。

WASM 的优点是执行隔离、热更新和跨语言复用；缺点是调试困难、线性内存协议复杂、模块审计和生命周期管理成本高。项目还保留纯 Java 引擎用于测试和基准，正是为了有可解释的参考实现。因为战斗规则更新和一致性收益确实存在，优点大于缺点，但它不是一键防作弊。

### 竞技场的匹配和战斗完成为什么必须分成两个状态机？

**面试官：** 如果匹配服务已经找到了对手，为什么不能直接更新双方分数？

**候选人：** 匹配只产生候选快照，解决“对谁展示或允许挑战”；发起战斗还要确认双方仍在同一赛季、挑战者没有 pending match、被挑战者仍在候选列表。发起时插入 PENDING match，完成时只接受仍为 PENDING 的记录，重复回调直接 no-op。

分开有两个优点：候选列表可以缓存和刷新，战斗状态可以独立处理超时、重复回调和玩家退出；缺点是多了快照失效、状态查询和异常路径。分数变化必须来自权威战斗计算或服务端校验，不能因为请求带了 scoreDelta 就直接写入。

如果把匹配和战斗合成一个接口，代码短，但长战斗、客户端断线和重复结果会把一个请求变成不可恢复的长事务。当前两阶段状态机的优点大于复杂度。

### 通用活动为什么要模板和实例两层？为什么不直接实时读取模板？

**面试官：** 运营修改活动排序字段后，正在进行的活动应该立即变化吗？

**候选人：** 模板是未来活动的配置来源，实例是在开跑时把规则和客户端配置快照下来。正在进行的实例使用快照，运营修改模板只影响未来实例；否则同一赛季前后用户看到的排序规则、结算规则可能不一致，历史奖励也难以解释。

快照的优点是历史可重放、结算可审计、配置变更不会悄悄改变进行中的事实；缺点是运营修错配置后不能简单地“改模板立刻修复”，需要显式生成新实例、补丁或迁移。这个取舍适合活动和赛季这类有明确生命周期的业务，优点大于实时配置便利。

### 动态 Mongo 查询为什么不能让客户端直接传字段名？

**面试官：** 活动排序字段由运营配置，客户端如果也能传 data.score，会发生什么？

**候选人：** 客户端传任意字段会带来查询越权、索引失效和潜在的查询资源消耗。运营配置也不能无限信任，因为动态字段可能指向高基数数组、未建立索引的路径或不允许暴露的内部字段。

正确做法是把排序字段限制在活动模板的 allowlist，解析成服务端的 SortField，再验证类型、方向、缺省值和对应索引。运营后台和 MCP TableSpec 也使用字段白名单，不能让模型直接提交任意 Mongo JSON。

动态能力的优点是新活动和后台表格少写代码；缺点是运行期行为更难静态检查，需要配置校验、dry-run、explain、审计和回滚。因为活动规则确实变化频繁，受控的动态配置优点大于把所有查询写死；开放任意字段则不是同一个方案。

## 深入追问：Kubernetes、运维与可观测性

### Deployment、Job 和 CronJob 分别解决什么问题？

**面试官：** 为什么 API 用 Deployment，结算任务不用常驻服务里的定时线程？

**候选人：** Deployment 管理长期运行的无状态副本，负责滚动更新、Service 接入和副本自愈；Job 表示一次性执行直到成功或失败；CronJob 是按时间创建 Job 的控制器。项目把 API、ops 和 bot 作为常驻应用，把 superlight-jobs 做成 WebApplicationType.NONE 的一次性进程，执行 CommandLineRunner 后关闭上下文。

这样拆分的优点是调度责任交给 Kubernetes，任务失败、重试和完成状态可以由 Job 观察，API 不会被批处理线程抢占资源；缺点是每次任务启动都要装配 Spring 上下文，任务频繁时启动成本较高。因为结算、赛季 tick 和 RFM 都是周期性批处理，不需要常驻内存状态，所以 CronJob 的优点大于常驻 scheduler。

### readiness、liveness 和 startup probe 有什么区别？

**面试官：** 一个 Pod 进程还活着，但 Mongo 没连上，三种 probe 应该分别怎么处理？

**候选人：** Liveness 判断进程是否需要重启，不能把所有下游故障都当成 liveness 失败，否则 Mongo 短暂抖动会让所有 Pod 一起重启。Readiness 判断 Pod 是否应该接收流量；Mongo 连接池或关键依赖不可用时可以暂时 not ready，让 Service 摘除它。Startup probe 用于启动慢的应用，在启动完成前暂时抑制 liveness/readiness 的误判。

项目 Deployment 有健康探针和优雅退出配置。发布时先让新 Pod readiness 通过，再按 rolling update 缩旧 Pod；preStop 给长连接和正在处理的请求留出时间。probe 的优点是把流量切换交给平台，缺点是探针设计错误会导致重启风暴或流量黑洞。因为 API 的正确性依赖“只把可服务实例放进负载均衡”，readiness 的优点大于只看进程存活。

### requests、limits、HPA 和 PDB 的关系是什么？

**面试官：** CPU request 等于 limit 就一定不会有长尾吗？PDB 能保证发布不丢连接吗？

**候选人：** request 是调度器预留和资源计算的基准，limit 是 cgroup 层面的上限；CPU limit 可能造成 throttling，内存超过 limit 可能 OOMKill。HPA 通常根据 CPU、内存或自定义指标调整副本数，但没有合理 request，利用率百分比就没有稳定含义。PDB 约束自愿驱逐时同时减少的副本数，不是发布成功保证，也不能防住节点崩溃。

chart 中 API 的 CPU request/limit 相等，主要为了降低 CPU 节流和长尾；内存 request/limit 不总相等，所以不能称为完整 Guaranteed QoS。PDB、滚动更新、readiness 和连接排空要一起看。它们的优点是分别保护调度、扩缩容和可用性；缺点是资源成本会增加、配置之间可能互相冲突。因为游戏 API 更怕延迟尖峰和发布时全量失效，当前配置的优点大于成本，但仍要通过 HPA 和 GC 指标校准。

### Helm upgrade --wait --atomic 是什么？回滚能解决什么、不能解决什么？

**面试官：** 发布失败时 Helm 自动回滚了，数据库迁移也会回滚吗？

**候选人：** wait 会等待资源达到就绪条件，atomic 会在升级失败时尝试回滚 release。它能恢复 Deployment、Service、ConfigMap 等 Kubernetes 资源版本，降低坏镜像、探针失败和模板错误造成的影响。

它不能撤销已经写入 Mongo 的数据、外部支付操作、已经发出的消息或不可逆的 schema 迁移。因此发布必须兼容旧代码和新代码的读写窗口，数据迁移采用 expand/contract：先增加兼容字段和读逻辑，再切换写入，确认所有 Pod 更新后才删除旧结构。Helm 回滚的优点是平台层恢复快，缺点是容易给人“整个系统回到了过去”的错觉。因为应用发布失败是高频故障，atomic 值得使用，但它必须配合业务补偿和向后兼容。

### CronJob 的 Forbid 能不能当分布式锁？

**面试官：** 如果上一轮任务还没结束，Forbid 阻止了下一轮，这是不是就足够了？

**候选人：** 不足够。Forbid 只约束同一个 CronJob 控制器创建的常规任务，不能覆盖手工创建 Job、多个 release、跨集群部署、控制器故障或 API 请求直接触发。任务还可能被重试，Pod 也可能在状态同步前重复执行。

因此任务自身需要幂等：结算回看最近三天并检查已结算 key，奖励使用唯一业务键，状态推进用条件更新，RFM 批任务对每个用户独立捕获异常。Forbid 的优点是低成本减少常规重叠，缺点是不能提供业务级互斥。因为它适合做第一层调度保护，但不是事实正确性的唯一来源，所以必须和数据库幂等一起使用。

### 线上故障你会按照什么顺序处理？

**面试官：** 如果玩家反馈“支付成功但没到账”，你怎么排查，不要直接重启服务。

**候选人：** 第一阶段先确认影响范围和时间窗口：是单用户、单渠道、单产品，还是全量；同时冻结可能扩大损失的自动重试或发奖动作。第二阶段按交易号串起日志、回调记录、订单状态、IAPTransaction、ShopUserPurchase 和 UserEvent，确认每个事实是否存在以及最后状态。

第三阶段看依赖：回调 HTTP 错误率、签名失败率、Rabbit 队列和 DLQ、Mongo 慢查询、连接池、Pod 重启和外部渠道状态。若是已验签但发货失败，优先用幂等补偿重放；若是验签前被恶意占位，则修复状态机并清理错误记录。所有人工操作都要记录操作者、原状态、新状态和依据，不能直接在 Mongo shell 随意改字段。

处理顺序的优点是先止损、再定位、后补偿，避免把偶发故障扩大成重复发奖；缺点是需要完整的审计和查询工具。因为支付问题的损失和合规风险高于恢复速度，证据链优先的优点大于“先重启看看”的便利。

### PLG 日志、Prometheus 指标和 Trace 各自看什么？

**面试官：** 为什么有 Grafana 和 JSON 日志还要 OpenTelemetry？

**候选人：** 日志回答某个请求或业务事件发生了什么，适合按用户、订单、异常堆栈检索；Prometheus 指标回答一段时间内有多少请求、延迟、错误和资源使用，适合告警和趋势；Trace 把一次请求跨 HTTP、Mongo、Redis、Rabbit 和异步任务的调用关系串起来，适合定位长尾和跨服务瓶颈。

项目的 JsonLogLayout 把 MDC 打进 stdout，Promtail/Loki/Grafana 负责日志检索；Actuator 暴露 Prometheus 指标，chart 通过注解发现 Pod；Grafana MCP 使用只读账号给 Agent 查询。现有日志和指标能覆盖基础排障，但跨消息的 trace parent 还不完整，所以未来需要 OpenTelemetry。

日志不能把高基数 userId、订单号都当 Prometheus label，否则时序数量爆炸；指标也不适合保存详细业务 payload。三者的优点是互补，缺点是采集、存储和上下文传播成本。当前阶段先用 PLG 交付基础可观测性，再补跨异步链路 tracing，优点大于一次性重做全套监控。

### 如何设计一个可操作的告警，而不是只做漂亮的 Dashboard？

**面试官：** 队列堆积、缓存命中率下降、Mongo 慢查询分别应该告警什么？

**候选人：** 告警要对应动作。Rabbit 队列要看消息年龄、堆积量、消费速率和 DLQ 增长；单看数量可能误报，因为业务高峰时积压会自然增加。缓存要看命中率、回源 QPS、热点 key 和 Redis 内存，而不是只看 Redis up/down。Mongo 要看慢查询、P95/P99、连接池等待、锁/事务冲突和索引计划。

告警内容要包含产品、环境、队列或集合、当前值、持续时间、runbook 链接和负责人。分级上，支付 DLQ 和奖励重复风险是高优先级；普通聊天 Pub/Sub 丢消息可能只触发低级别指标。优点是告警能直接指导处理，缺点是需要持续调阈值和维护 runbook。因为没有行动路径的告警只会制造疲劳，所以可操作性优先。

## 深入追问：CI/CD、测试与质量

### CI 为什么要按模块矩阵测试？只跑一次全量测试不行吗？

**面试官：** 多模块项目为什么不直接执行一条全量 Gradle test？

**候选人：** 模块矩阵让变更更容易定位：API、联盟、竞技场、活动和 jobs 可以有自己的测试任务；涉及 Mongo、Redis、Rabbit 的集成测试用 Testcontainers，格式化和构建作为独立门禁。全量测试仍然可以作为最终验证，但矩阵能减少失败反馈时间，也能让不同模块的资源需求隔离。

它的优点是反馈快、失败定位清楚、可以并行；缺点是矩阵配置复杂，模块依赖和共享测试资源可能导致重复耗时。项目还要求 Java 21、Google Java Format 和 REST Docs 片段一起通过，避免“代码能编译但接口文档过期”。如果矩阵拆得过细导致隐藏集成问题，我会保留定期的全量集成流水线。

### 为什么 REST Docs 用测试生成，而不是手写 Swagger 注解？

**面试官：** 测试和文档绑定有什么代价？

**候选人：** MockMvc 测试实际调用 Controller，并通过 document 生成请求、响应和字段 snippets，再由 Asciidoctor 组装进 bootJar。实现改变状态码或字段时，测试/文档断言会失败，文档不容易悄悄漂移。

优点是契约来自可执行测试，示例更接近真实响应；缺点是每个接口都需要维护测试和字段描述，测试覆盖不足时文档仍然不完整，不能自动表达所有业务语义。手写 OpenAPI 的优点是可以先设计契约和生成客户端，缺点是容易与实现分叉。当前 API 规模和测试基础适合 REST Docs，所以优点大于缺点。

### 你会怎么测试分布式故障，而不是只测成功路径？

**面试官：** 具体说说支付、消息和缓存的测试用例。

**候选人：** 支付要测试重复回调、验签失败、金额不一致、用户不存在、回调记录已存在、发货成功但响应丢失，以及发货重试不会重复增加消费。消息要测试 publisher confirm 失败、消费者在写库前后崩溃、重试耗尽进 DLQ、重复消息和乱序消息。缓存要测试命中、失效绕过、MongoTemplate 直写后的显式清理、Redis 不可用时回源或降级。

状态机测试应覆盖每个非法转移和边界时间，联盟锁超时、节点锁释放和活动结算要用固定时钟。WASM 用固定输入、版本和 seed 做回放，Java 引擎和模块结果对照。集成测试的优点是发现真实驱动和容器差异，缺点是慢且环境更脆；单元测试无法替代这些故障语义，所以关键链路必须承担这部分成本。
