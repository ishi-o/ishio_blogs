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

一个多游戏复用的社交游戏后端：单个 `Spring Boot` 代码库，通过产品级 `profile` 配置同时服务多个游戏包体（一个构建产物 + N 份 `application-{product}.yaml`），按产品隔离数据库连接、队列与缓存前缀

```yaml
# 一份 per-product 配置 = 一个游戏的全部连接信息（AppConfiguration record 承接）
product:
  mongodb-uri: mongodb://...
  redis-uri: redis://...
  rabbitmq: { host: ..., queues: { settlement: "game-a:prod:settlement" } }
  storage: { type: oss, bucket: ... }
```

## 技术选型

### 数据库

- 项目采用`Mongodb`数据库，与传统关系型数据库相比，它的优点在于：
  - 它拥有灵活的 `schema`，或者说它没有 `schema`，针对游戏后端这类业务以及客户端需求时常变化，特别是需要支持多个游戏的后端而言，需要非常灵活的 `schema`
    - 新增字段无需执行 `ALTER TABLE`，也就是不需要维护`schema`，同时也不需要数据迁移
    - 向后兼容性好，旧数据缺少新字段不影响读取
  - 拥有灵活的模式带来的好处不仅是灵活添加或删除字段，也意味着原本需要存储于多个关系表的数据可以内嵌存储在同一个 `MongoDB` 的集合里，减少许多关联查询的同时，不会消耗更多的存储空间；比如说我们有一个 `UserEvent`，一些用户事件可能会附带一些奖励，而这些奖励的结构不尽相同，但由于类型也并不算多，所以采用了不同种奖励存储在同一个集合的做法
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

### 索引实践

- **唯一索引当并发守卫用**：对“只应发生一次”的写入（活动参与、回调处理、奖励发放），与其在代码里先查后插（查和插之间存在竞态窗口），不如直接建 `(activityId, userId, cycle)` 这类唯一复合索引——并发提交的重复文档在数据库层被拒绝，应用捕获 `DuplicateKeyException` 当幂等命中处理；查重逻辑和竞态窗口一起消失：

  ```java
  try {
    callbackRepo.insert(CallbackRecord.of(outTradeNo, rawBody)); // outTradeNo 唯一
  } catch (DuplicateKeyException e) {
    return "success"; // 已处理过 → 幂等命中，直接应答
  }
  // ... 验签、映射订单、创建交易
  ```

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

  - 每个活动一小片索引，而不是全体活动共用一个大复合索引；活动下线后这片索引可以独立删除
  - 索引名带活动 `id` 前缀，按名字 create-if-missing，启动器天然幂等
  - 建索引的 `CommandLineRunner` 挂 `@Profile("!all-in-one")`：正常部署在启动时创建，本地一体式开发跳过、交给 `auto-index-creation` 兜底
  - 代价是索引只增不删：活动中途改排序字段会生成新索引，旧索引留在那成了写放大——接受它，因为活动实例本身有快照语义，排序字段的变更频率很低

### 缓存：`Redis`

- `key` 统一经 `RedisKeyProvider` 加“环境/产品”前缀拼接，多产品、多环境**共用一套 `Redis`** 而不互相污染：

  ```java
  // {产品}:{环境}:{业务}:{参数}，一处定义处处使用
  keyProvider.withParts("lock", "season-matching", seasonId);
  // => "game-a:prod:lock:season-matching:123"
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

- 没有 `broker` 的进程（如本地开发）通过 `@ConditionalOnProperty` 自动降级为同步/内存调度，发布侧 `bean` 和监听器是同一套代码

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

### 项目部署结构

- 项目是经典的`gradle`多模块应用，不涉及应用层面的分布式框架
- 项目使用`k8s`，依靠其强大的基础设施能力，实现容器的部署和管理，有时遇到分布式服务间调用仍使用`k8s`结合`istio`实现服务发现和网格服务治理
- 部署层面做了两层抽象：
  - 内部 `helm` `chart`：应用 `chart`（`Deployment` + `Service` + 网关路由 + `HPA` + `PDB` + `NetworkPolicy` + 证书/镜像/数据库密钥）与任务 `chart`（`CronJob` 包装器），一个 `release` = 一个产品的一个应用，所有产品差异都在 `values` 文件里
  - 多云兼容：存储类按云厂商出不同模板，同一份 `chart` 跑在异构集群上
- 副本策略上 `CPU` request == limit（近似 `Guaranteed QoS`），避免 `CPU` 节流带来的长尾延迟，也让云成本可预测：

  ```yaml
  resources:
    requests: { cpu: "2", memory: 2Gi }
    limits: { cpu: "2", memory: 4Gi } # CPU 不超卖 → 不被 cgroup 节流
  env:
    - name: JAVA_TOOL_OPTIONS
      value: "-XX:MaxRAMPercentage=75 -XX:ActiveProcessorCount=2"
  ```

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

## 通用活动框架

目标是“一个引擎，运营配置出任意限时活动”，新活动**不写代码**，只加配置文档

- **模板/实例分离 + 快照**：`GenericActivity` 是模板（时间模式、排序字段、机器人规则、客户端配置），开跑时物化为 `GenericActivityInstance`，配置整体拷贝进实例——赛季中改模板不会改写已经开跑的实例，历史不被重写
- 时间模式两种：`CALENDAR`（全局统一时刻表）与 `PERSONAL`（每人进入时起算的个人计时）；支持**链式活动**（`nextActivityId`，可自环）与**循环实例**（`maxCycles` 可为无限），周期与状态全部**读时派生**（`now` 与起止时间比较），落库的状态字段已废弃——派生的东西不存储就不会和事实打架：

  ```java
  public Status getStatus() { // 不落库，读时派生
    var now = Instant.now();
    if (now.isBefore(instanceStartTime)) return SCHEDULED;
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
- 结算：按榜单发 `UserEvent`（复用用户事件框架做奖励分发）；日结算用 `dailySettledKeys` 集合（`leaderboardId:dateKey`）做幂等；每实例独立事务靠 `@Autowired @Lazy` 自注入代理绕过 `Spring AOP` 自调用失效：

  ```java
  @Autowired @Lazy private GenericActivitySettlementService self;

  void settleAll(List<Instance> due) {
    for (var instance : due)
      self.settle(instance); // 经代理调用 → @Transactional 生效；直接 this.settle() 不生效
  }
  ```

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

## 联盟系统

业务上是公会那一套（成员/申请/建设/团购/科技树），值得记的不是玩法而是承载玩法的几个通用模式：

- **数值全部配置驱动**：任务、里程碑、商店、科技的数值都在运营可编辑的 `ObjectConfig` 里，代码只有引擎没有数值，新玩法调参不发版；唯一需要在服务端设防的是**跨联盟的重放**——建设任务上报用 `Redis` 冷却 `key` 防止换联盟后对同一任务重复上报，同联盟上报豁免：

  ```java
  var key = keyProvider.withParts("cooldown", userId, taskId); // 12h TTL
  var prev = redis.opsForValue().get(key);
  if (prev != null && !prev.equals(allianceId)) return; // 别的联盟已记过这题
  redis.opsForValue().set(key, allianceId, Duration.ofHours(12));
  ```

- **赛季是显式状态机**：`NOT_STARTED → 部署期 → 备战期 → 战斗期 → 回合结算 → 赛季结算 → ENDED`，阶段计划是**有序阶段列表**（类型 + 时长）；推进由延迟消息武装，回调带 `expectedPhaseIndex/expectedStartedAt` 的 `CAS` 式参数——并发的 `tick` 只有一个能成功；调度器进程另有对账 `tick` 兜底重挂丢失的延迟消息：

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

## 战斗校验与 `WASM` 模块

- 问题：客户端（`Unity`）本地预测战斗结果，服务端需要**权威校验**，两边逻辑必须逐位一致
- 三层方案：
  - 纯 `Java` 移植客户端战斗数学（无依赖的独立模块），技能条件用正则解析一套迷你 `DSL`：

    ```java
    // 表驱动的技能条件表达式 → 正则逐条解析
    // Count(AllyLineup, AnchorId == 3) >= 2 → 队伍中该角色数 >= 2
    // Any(AllyLineup, HP% < 50)            → 任一队友血量过半以下
    ```

  - **`WASM`**：把同一份战斗逻辑编译成 `WebAssembly`，服务端用 `wasmtime-java` 跑**同一个二进制**——客户端预测与服务端校验结果天然一致，策划可以**不发版更新战斗模块**：

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

- 引擎单例 + 版本化热替换：`LoadedModule` 按版本加载，新版本 `enable` 后替换旧引用，旧模块等存量请求结束后释放

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

## 消息框架

### `WebSocket`

- `STOMP` over 原生 `WebSocket`，端点 `/ws`；用途：游戏内聊天（按 `{module}/{itemId}` 分话题 + 历史回放）、站内通知（每用户队列）、联盟战斗的实时转发
- **跨副本投递用 `Redis pub/sub` 桥接**：内存 `simple broker` 只懂本进程的订阅，发布方先把消息发到 `Redis` 频道（按环境/产品前缀隔离），每个副本的订阅器收到后再扇出到本地 `broker`（广播到 `/topic`，单播到 `/user/queue`）——用现成组件拼出一个“穷人的外置 `broker`”：

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
    if (allowedHosts.stream().noneMatch(h -> target.getHost().startsWith(h)))
      return forbidden(); // 只转发到运营登记的主机前缀
    return forward(target, stripHopByHopHeaders());
  }
  ```

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
