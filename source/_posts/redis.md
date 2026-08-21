---
title: "Redis: 数据结构、场景实践与高可用"
date: 2026-06-14T00:00:00.000Z
categories: [Tools & Utilities, 3P Tools, Redis]
tags: [redis, spring boot, cache, distributed lock]
mathjax: false
---

<!-- placeholder -->
<!-- more -->

# `Redis`

## 基本用法

### 简介

- **`Redis`**(REmote DIctionary Server)是一个基于内存的 `key-value` 存储，官方定位是数据库、缓存与消息中间件三合一
- 三个让它快的根本原因，面试时按顺序讲：
  - **纯内存**操作，瓶颈在内存带宽与网络 `RTT`，而不是磁盘 `I/O`
  - **单线程命令执行**(`6.0` 之后网络 `I/O` 多线程，命令仍串行)，免去了锁与上下文切换，也保证了单个命令的原子性
  - 针对每种用途**特化的底层数据结构**(SDS、跳表、listpack 等)，而不是像通用 `HashMap` 那样一套结构包打天下
- 单线程的代价是**任何慢命令都会阻塞全部请求**:`KEYS *`、大 `key` 的 `DEL`、`HGETALL` 一个十万元素的 `hash` 都属于线上事故命令，用 `SCAN` 系列渐进式替代

### Spring Boot 接入

- 依赖是 `spring-boot-starter-data-redis`，底层客户端默认为 `Lettuce`(基于 `Netty` 的同步/异步连接，连接天然线程安全)；`Jedis` 是直连阻塞式实现，需要连接池，新项目没有理由再选它

  ```xml
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
  </dependency>
  ```

- 连接配置，单机与哨兵二选一：

  ```yaml
  spring:
    data:
      redis:
        # 单机
        host: 127.0.0.1
        port: 6379
        password: ${REDIS_PASSWORD}
        lettuce:
          pool:
            max-active: 16
            max-idle: 8
        # 哨兵(启用后 host/port 失效)
        # sentinel:
        #   master: mymaster
        #   nodes: 10.0.0.1:26379,10.0.0.2:26379,10.0.0.3:26379
  ```

- `RedisTemplate` 默认用 `JDK` 序列化，存进去的 `key` 是带类型前缀的二进制，`redis-cli` 里根本没法读，必须显式配置序列化器：

  ```java
  @Configuration
  public class RedisConfig {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory cf) {
      RedisTemplate<String, Object> template = new RedisTemplate<>();
      template.setConnectionFactory(cf);

      var jack = new Jackson2JsonRedisSerializer<>(Object.class);
      var om = new ObjectMapper()
          .registerModule(new JavaTimeModule())
          .activateDefaultTyping( // 记录类型信息, 反序列化才能还原具体类
              LaissezFaireSubTypeValidator.instance,
              ObjectMapper.DefaultTyping.NON_FINAL, JsonTypeInfo.As.PROPERTY);
      jack.setObjectMapper(om);

      template.setKeySerializer(new StringRedisSerializer());
      template.setHashKeySerializer(new StringRedisSerializer());
      template.setValueSerializer(jack);
      template.setHashValueSerializer(jack);
      return template;
    }
  }
  ```

- 日常更推荐 `StringRedisTemplate` + 手动 `JSON`:`value` 就是一段可读 `JSON`，排查问题时 `redis-cli` 直接能看；类型信息丢失的问题由业务侧固定 `DTO` 解决

  ```java
  @RequiredArgsConstructor
  @Service
  public class UserCacheService {
    private final StringRedisTemplate redis;
    private final ObjectMapper om;

    private static final Duration TTL = Duration.ofMinutes(30);

    public User getUser(long id) throws Exception {
      String key = "user:%d".formatted(id);
      String json = redis.opsForValue().get(key);
      if (json != null) {
        return om.readValue(json, User.class);
      }
      User user = userRepository.find(id);          // 回源 DB
      redis.opsForValue().set(key, om.writeValueAsString(user), TTL);
      return user;
    }
  }
  ```

- 也可以直接用 `Spring Cache` 抽象，注解驱动，底层由 `RedisCacheManager` 承接；缺点是注解表达不了"缓存穿透打特殊值"这类精细逻辑，复杂场景还是手写

  ```java
  @Cacheable(cacheNames = "user", key = "#id")
  public User find(long id) { return userRepository.find(id); }

  @CacheEvict(cacheNames = "user", key = "#id")
  public void update(User u) { userRepository.save(u); }
  ```

## 数据结构与底层实现

- 这是 `Redis` 面试的核心章节。总纲一句话：**外层的 `key` 永远是字符串，`value` 才有类型**；每个 `key-value` 在全局 `dict` 里是一个 `entry`,`value` 是一个 `redisObject`,其 `encoding` 字段记录底层实现，同一类型会随数据规模**自动换挡**

  ```text
  redisDb.dict: "user:1" -> redisObject(type=ZSET, encoding=skiplist)
                              type:    对外类型(String/List/Hash/Set/ZSet/Stream)
                              encoding: 实际编码(listpack / skiplist / hashtable ...)
                              ptr:     指向底层结构
  ```

### SDS: 字符串

- 字符串的底层是 **SDS**(Simple Dynamic String)，而不是C原生字符串。结构为 `len + alloc + flags + buf[]`:

  ```c
  struct sdshdr {
    uint32_t len;    // 已使用长度, O(1) 取长度, 不用像 C 那样遍历找 \0
    uint32_t alloc;  // 分配总长度
    unsigned char flags;
    char buf[];      // 内容, 末尾仍带 \0, 兼容部分 C 函数
  };
  ```

- 相对C字符串的四个改进，每个都对应一个工程问题：
  - **O(1) 长度**:`C` 取长度要 `strlen` 遍历，`SDS` 直接读 `len`
  - **杜绝缓冲区溢出**：拼接前检查 `alloc - len` 是否够用，不够先扩容，`strcat` 那种踩内存不存在
  - **空间预分配与惰性释放**：扩容时若 `len < 1MB` 则多分配一倍，否则多加 `1MB`；缩短时不立刻回收，留作下次增长——减少反复 `realloc`
  - **二进制安全**：以 `len` 而非 `\0` 判定结尾，可以存图片、序列化对象等任意字节
- 编码换挡：小整数用 `int`，短字符串用 `embstr`(redisObject 与 SDS 一次分配、内存连续，适合小对象)，超过 44 字节升级为 `raw`(两次分配)

### Dict: 渐进式 rehash

- 全局 `key` 空间、`hash` 类型、`set` 的底层都是 **dict**——拉链法哈希表，`hash` 冲突用头插链表，扩容翻倍并重新散列
- 问题在于 `Redis` 是单线程：一亿个 `key` 一次性 `rehash` 会卡死服务。解决方案是**渐进式 rehash**:
  - dict 同时持有 `ht[0]`(旧表)与 `ht[1]`(新表)，`rehashidx` 游标从 0 开始
  - 每次**增删改查**顺带把 `ht[0]` 上 `rehashidx` 桶里的所有 `entry` 搬到 `ht[1]`，再推进 `rehashidx`；同时后台定时任务也会搬一小批
  - rehash 期间**读请求查两张表**，写请求只写 `ht[1]`(保证 `ht[0]` 只减不增)
- 面试追问点：渐进式 rehash 期间一个 `key` 可能既在旧表又在迁移中间态，所以 `KEYS`、`BGSAVE` 这类全量遍历都要感知双表;`SCAN` 的反向二进制迭代(cursor 按"低位+1"递进)正是为了在 rehash 中不漏 key(可能重复，需要业务幂等)

### Listpack: 小数据量的紧凑编码

- `hash`、`zset`、`list` 在元素少且小时，底层是**紧凑的连续内存块**:`7.0` 之前是 `ziplist`(`entry` 里记录前一个节点长度，极端级联更新是著名缺陷)，`7.0` 起改为 **listpack**——每个 `entry` 只记自己的长度，从根本上消灭了级联更新
- 以 `hash` 为例，`value` 是 `[k1][v1][k2][v2]...` 顺序排列，查找是 O(n) 线性扫描，但 n 很小时**内存连续带来的缓存友好性完胜哈希表**，且没有指针开销

  ```text
  hash-max-listpack-entries: 128   # 超过 128 个 entry 换 hashtable
  hash-max-listpack-value:   64    # 任一 member 超过 64 字节也换
  ```

- 实践含义：**把 `hash`/`zset` 当小对象容器用时，要留意换挡阈值**。一个 200 字段的 `hash` 已经是 hashtable,`HGETALL` 的代价完全不同；反过来，存"一个实体的几十个字段"用 `hash` 比序列化成 `String` 更省，还能按字段读写

### Quicklist: List 的实现

- `3.2` 之后 `list` 底层统一为 **quicklist**：一个双向链表，但每个节点是一个 listpack(早期可配置 ziplist)，相当于"分段的紧凑链表"
- 兼得了两头：链表的 O(1) 头尾操作(`LPUSH`/`RPOP` 队列语义)，和 listpack 的紧凑存储；单个节点过大时中间压缩(`LZF`)
- 典型用途是**消息队列雏形**(`LPUSH` + `BRPOP` 阻塞弹出)和最新动态列表(`LPUSH` + `LTRIM 0 999` 固定长度)

### Skiplist: ZSet 的实现

- `zset` 在小数据量时是 listpack，超过阈值后是 **skiplist + dict 双结构**:

  ```java
  typedef struct zset {
    dict *dict;       // member -> score, O(1) 查分数(ZSCORE)
    zskiplist *zsl;   // 按 score 排序的跳表, O(logN) 范围查询(ZRANGE)
  } zset;
  ```

- 跳表本质是**多层有序链表**：每个节点插入时以 1/4 概率(每层晋升概率)决定是否在上一层出现，平均 O(logN) 的查找就像二分一样逐层缩小范围。查询 `ZRANGEBYSCORE` 时先在跳表里定位起点，再沿底层链表顺序收集
- 面试高频对比——**为什么用跳表不用红黑树**:
  - 实现简单得多，没有旋转与变色的多种情况
  - 范围查询天然友好：定位起点后沿链表走即可；红黑树需要中序遍历，还要维护栈
  - 内存可调：平均每节点约 1.33 个指针(1/(1-p) - 1)，比树的三指针省
  - 代价是**最坏 O(N)**(概率保证平均性能，不保证平衡)，以及没有树那样成熟的并发改造先例——但 `Redis` 单线程执行命令，这点无所谓
- `dict` 与跳表**共享 member 与 score 的引用**，不存两份，`ZSCORE` 与 `ZRANGE` 各走各的索引

### Set 与 Bitmap

- `set` 的编码换挡：全是整数且少量时用 **intset**(有序整数数组，二分查找)，否则换 hashtable(value 为 null 的 dict)
- 集合运算 `SINTER`/`SUNION`/`SDIFF` 是原生命令，做共同好友、标签交集很方便；注意大集合间 `SINTERSTORE` 会阻塞，复杂度 O(N*M)
- **Bitmap** 不是新类型，而是字符串上的位操作(`SETBIT`/`GETBIT`/`BITCOUNT`)，一亿用户的日活签到只占 ~12MB;`BITOP AND` 可以做多日留存交集。活跃统计若允许 0.81% 误差，**HyperLogLog**(`PFADD`/`PFCOUNT`)每个 `key` 固定 12KB,是亿级 `UV` 统计的标准答案

## 业务场景实践

### 缓存：三件套异常

缓存是 `Redis` 第一用途，面试必问的是三个失效模式，逐个给出解法：

- **缓存穿透**(查不存在的 `key`,每次都打到 `DB`):
  - 缓存空值:`null` 也写入，短 `TTL`(如 60s);实现简单，缺点是大量不同 `key` 时占用内存
  - 布隆过滤器(`RedisBloom` 模块或 `Redisson` 内置 `RBloomFilter`):布隆过滤器说"不存在"就一定不存在。代价是误判率、以及标准布隆过滤器**无法删除元素**(计数布隆可以)
- **缓存击穿**(单个热 `key` 过期瞬间，海量并发同时回源):
  - 互斥回源：只放一个线程去查 `DB`，其余等待。用 `SET key value NX EX` 抢锁，或本地 `Caffeine` 做 `loader` 级别的合并
  - 逻辑过期：`value` 里带上过期时间字段，物理上不设 `TTL`;发现逻辑过期后异步刷新，期间返回旧值——用短暂的一致性换取可用性
- **缓存雪崩**(大量 `key` 同一时刻集体过期，或缓存实例整体宕机):
  - `TTL` 加随机抖动(`TTL + random(0, 300s)`),打散过期时间
  - 多级缓存(本地 `Caffeine` + `Redis`)挡住流量尖峰
  - 宕机场景靠高可用部署(哨兵/集群)与限流降级兜底

一致性方面，标准答案是 **Cache Aside**：读——先查缓存， miss 回源并回填；写——**先更新 `DB`，再删缓存**。面试要点：

- 为什么是"删"而不是"改"缓存：并发写时后写者的旧值可能覆盖新值；删除让下一次读自然回填最新值
- 为什么先 `DB` 后缓存：反过来会出现"`DB` 还没提交，缓存已删，另一个读请求回填旧值，此后一直脏读"的窗口
- 该方案仍有极小概率的脏数据窗口(读请求在删缓存后、回填前被写请求插队)，强一致要么上分布式锁串行化，要么接受"`TTL` 兜底最终一致"——绝大多数业务选后者

### ZSet: 排行榜

- `zset` 是排行榜的天选结构:`ZADD` 更新分数、`ZREVRANK` 查名次、`ZREVRANGE` 取TopN、`ZINCRBY` 原子加分，全部 O(logN) 或更好，天然原子性免掉了"读-改-写"竞态

  ```java
  @RequiredArgsConstructor
  @Service
  public class LeaderboardService {
    private final StringRedisTemplate redis;
    private final ObjectMapper om;

    // 榜单 key 按赛季分区, 便于整榜过期与历史留存
    private String key(String season) { return "leaderboard:%s".formatted(season); }

    public void addScore(String season, long userId, double delta) {
      redis.opsForZSet().incrementScore(key(season), String.valueOf(userId), delta);
    }

    /** 榜单页: [from, to] 闭区间名次, 附带每个 entry 的用户信息 */
    public List<RankEntry> page(String season, long from, long to) {
      var raw = redis.opsForZSet()
          .reverseRangeWithScores(key(season), from, to);   // ZREVRANGE
      if (raw == null || raw.isEmpty()) return List.of();

      var ids = raw.stream().map(t -> Long.parseLong(t.getValue())).toList();
      Map<Long, User> users = userRepository.findAllById(ids).stream()
          .collect(Collectors.toMap(User::getId, u -> u));  // 批量回源, 避免 N+1

      List<RankEntry> result = new ArrayList<>(raw.size());
      long rank = from + 1;
      for (var t : raw) {
        var id = Long.parseLong(t.getValue());
        var u = users.get(id);
        if (u != null) {                                     // 用户可能已注销, 跳过
          result.add(new RankEntry(rank, u.getId(), u.getName(), t.getScore()));
        }
        rank++;
      }
      return result;
    }

    /** 自己的名次: ZREVRANK, O(logN); 榜上没有则提示距离上榜还差多少分 */
    public long myRank(String season, long userId) {
      Long rank = redis.opsForZSet().reverseRank(key(season), String.valueOf(userId));
      if (rank == null) throw new NotFoundException("not on board");
      return rank + 1;   // ZREVRANK 从 0 计
    }
  }
  ```

- 两个生产细节：
  - **分数并列时要稳定排序**：把次要排序维度编进分数，如 `score * 1e10 + (MAX_TS - 达成时间戳)`，先比分数、再比达成先后；不要试图用 `+inf` 之类技巧，浮点精度先崩
  - **大榜与全榜导出不要走 `ZRANGE 0 -1`**:百万成员一次拉完会同时打爆 `Redis`、网络与 `JVM` 内存；分页渐进导出，或者干脆由离线任务从 `DB` 算

### Pub/Sub: 跨节点 WebSocket 消息路由

- 场景:`WebSocket` 服务水平扩展成多实例后，用户 `A` 连在节点 1、用户 `B` 连在节点 2,`A` 给 `B` 发消息，应用层怎么把消息送到节点 2?——各实例订阅同一个 `Redis` 频道，**任一节点投递的消息会被所有实例收到**，各自检查"目标用户在不在我这"，在就推下线：

  ```text
  node-1 ──publish──> Redis channel "chat:room:42" ──broadcast──> node-1 / node-2 / node-3
                                                              各自检查本地 session 表, 命中才推
  ```

  ```java
  @Component
  @RequiredArgsConstructor
  public class RedisWsBroker implements MessageListener {

    private final StringRedisTemplate redis;
    private final SimpUserRegistry userRegistry;     // 本实例的 WebSocket 用户表
    private final SimpMessagingTemplate ws;

    /** 任一节点调用: 消息发到频道, 由持有目标 session 的节点实际推送 */
    public void route(String room, WsPayload payload) throws Exception {
      redis.convertAndSend("ws:room:" + room, payload.json());
    }

    @Override
    public void onMessage(Message message, byte[] pattern) {
      var payload = WsPayload.parse(new String(message.getBody(), UTF_8));
      // 每个节点都收到, 但只有真正持有该用户 connection 的节点会推送
      if (userRegistry.getUser(payload.toUser()) != null) {
        ws.convertAndSendToUser(payload.toUser(), "/queue/messages", payload);
      }
    }

    @PostConstruct
    public void subscribe() {
      var container = new RedisMessageListenerContainer();
      // ... 配置连接工厂后:
      container.addMessageListener(this, new ChannelTopic("ws:room:*"));
    }
  }
  ```

- `pub/sub` 的**无界投递语义**要牢记：没有消费者就不投(不存盘)，慢消费者会被断开。所以它适合**在线推送**这类"人不在就算了"的场景；离线消息、重放、消费确认请用 `Stream`(见下文)

### 限流与全局 ID

- 限流用**固定窗口计数器**最简单(`INCR` + `EXPIRE NX`)，临界突刺问题用**滑动窗口日志**(ZSet 存时间戳，`ZREMRANGEBYSCORE` 清窗口外)解决：

  ```java
  /** 滑动窗口限流: window 内最多 limit 次 */
  public boolean tryAcquire(String key, int limit, Duration window) {
    long now = System.currentTimeMillis();
    String k = "ratelimit:" + key;
    redis.execute(SessionCallback.<Void>of(s -> {
      s.multi();
      s.opsForZSet().removeRangeByScore(k, 0, now - window.toMillis()); // 清理窗口外
      s.opsForZSet().add(k, UUID.randomUUID().toString(), now);         // 记录本次
      s.opsForZSet().zCard(k);
      s.expire(k, window);                                              // 兜底过期, 防泄漏
      var res = s.exec();
      Long count = (Long) res.get(2);
      if (count != null && count > limit) {                             // 超限: 撤销刚写入的记录
        redis.opsForZSet().removeRangeByScore(k, now, now);
        return null;  // rejected
      }
      return null;
    }));
    return true;
  }
  ```

  生产上通常直接用 `Redisson` 的 `RRateLimiter`(令牌桶， Lua 原子实现)，或者网关层(`Envoy`/`Sentinel`)做第一道

- 全局 ID 常见解法对比:`INCR` 单点原子自增(简单，但分库分表场景离散性差、有时间回溯风险)、**号段模式**(`INCRBY step` 一次取一批，本地发号， `DB` 与 `Redis` 都适用)、雪花算法(纯本地，依赖时钟)。`Redis` 方案的核心优势是原子性不依赖任何 `DB`

## 功能特性

### 过期与内存淘汰

- 过期删除是**惰性 + 定期**的组合：访问时检查(`lazy`)，加后台每秒若干次随机抽样清理(`active`)，所以 `TTL` 到了不保证立刻消失
- 删大 `key` 用 `UNLINK`(异步回收内存)代替 `DEL`(同步阻塞);`SCAN` 找出目标后分批 `UNLINK`
- 内存满时按 `maxmemory-policy` 淘汰，常用的四种：`noeviction`(拒绝写，默认)、`allkeys-lru`/`allkeys-lfu`(全量键按最近/最不常用淘汰，**缓存场景的标准选择**)、`volatile-ttl`(只淘汰带 `TTL` 的)。`LFU`(`4.0`+)以对数计数器记录访问频率，比 `LRU` 更抗"偶发批量扫描污染缓存"

### 事务、Pipeline 与 Lua

- 三者解决三个不同的问题，经常被混为一谈：
  - **`pipeline`**：批量攒命令一次发送，节省 N 次 `RTT`。**不保证原子性**，中间可能插入其他客户端的命令
  - **`MULTI/EXEC` 事务**：命令入队后顺序执行，**保证不被插队**(隔离性)，但**没有回滚**——某条命令运行时出错，前面的照常生效。这个反直觉的设计官方解释是：错误只应来自语法或类型误用，属于开发期 `bug`，回滚救不了生产
  - **`Lua` 脚本**：整段脚本作为单命令原子执行，**既有隔离性又能做条件逻辑**，是"读-判-写"类需求(限流、锁、库存扣减)的正解。`EVAL` 每次传脚本体有网络开销，应使用 `SCRIPT LOAD` + `EVALSHA`

  ```java
  // 例: 库存扣减, 原子完成 "查库存 -> 判足够 -> 扣减", 并发下不会超卖
  private static final String DEDUCT = """
      local stock = redis.call('GET', KEYS[1])
      if not stock then return -1 end
      stock = tonumber(stock)
      local n = tonumber(ARGV[1])
      if stock < n then return 0 end
      redis.call('DECRBY', KEYS[1], n)
      return 1
      """;

  public boolean deduct(String sku, int n) {
    Long r = redis.execute(RedisScript.of(DEDUCT, Long.class),
        List.of("stock:" + sku), String.valueOf(n));
    return r != null && r == 1;
  }
  ```

- `Lua` 脚本内不要写慢逻辑：脚本执行期间整个 `Redis` 都被阻塞，`redis.busy-script` 超时后其他客户端只能排队

### Pub/Sub 与 Stream

- `pub/sub`(上文 `WebSocket` 场景)语义是**fire-and-forget**:`SUBSCRIBE`/`PUBLISH`，不持久化、不确认、无消费组，断线即丢
- **`Stream`**(`5.0`+)是 `Redis` 版的 `Kafka`:消息以 `id`(毫秒时间戳-序号)追加进日志结构，支持
  - **消费组**(`XGROUP`):组内消费者负载均衡，各自维护 `PEL`(pending list)记录已取出未确认的消息
  - **确认机制**(`XACK`):处理完才确认，崩溃恢复后 `XPENDING`/`XCLAIM` 可以把积压转移给别的消费者重做
  - **阻塞读**(`XREAD BLOCK`):不空转

  ```java
  // 生产者: 追加一条订单事件
  redis.opsForStream().add("stream:order",
      Map.of("orderId", "1024", "event", "PAID"));

  // 消费组: 组内多消费者均分, 手动 ack
  redis.opsForStream().createGroup("stream:order", "order-group");
  var messages = redis.opsForStream().read(
      Consumer.from("order-group", "consumer-1"),
      StreamReadOptions.empty().count(10),
      StreamOffset.create("stream:order", ReadOffset.lastConsumed()));
  // 处理成功后
  redis.opsForStream().acknowledge("stream:order", "order-group", messageId);
  ```

- `Stream` 的定位是**轻量消息队列**：消息可靠性、积压重放在中型流量下完全够用，省去一套 `Kafka`/`RabbitMQ` 的运维；但受限于单机内存与 `XTRIM`/`MAXLEN` 的粗粒度保留策略，不适合大数据管道或严格顺序多订阅场景。`5.0` 之前的 `PUB/SUB` 与 `Stream` 不是替代关系，一个管"在线广播"，一个管"可靠消费"

### 持久化: RDB 与 AOF

- **RDB**(Redis Database):某时刻全量内存快照，二进制紧凑。`SAVE` 阻塞主线程，生产用 `BGSAVE`——`fork` 子进程写临时文件，借助操作系统**写时复制**(COW)主进程照常服务，子进程看到的是 fork 瞬间的冻结视图
  - 优点：文件小、恢复快(直接载入)、适合异地容灾备份
  - 缺点：**两次快照之间的数据在宕机时丢失**；`fork` 本身在内存大、写入密集时可能卡顿(COW 页复制拖慢父进程)
- **AOF**(Append Only File):每条写命令追加到日志，重启时重放恢复。三个关键配置：

  ```text
  appendonly yes
  appendfsync everysec    # always(每条都 fsync, 最多丢 1 条) / everysec(默认, 最多丢 1s) / no(交给 OS)
  auto-aof-rewrite-percentage 100   # 比上次重写后体积翻倍时触发 BGREWRITEAOF
  ```

  - AOF 重写:`INCR x` 一万次后日志膨胀，`BGREWRITEAOF` 同样 `fork` 子进程，把当前内存状态反推为最小命令集写出；期间新写命令同时写旧 AOF 与重写缓冲，保证不丢
- **混合持久化**(`4.0`+，`aof-use-rdb-preamble`):重写后的 AOF 前半段是 RDB 格式全量快照，后半段是增量命令——恢复速度接近 RDB，丢失窗口接近 AOF，生产默认选择
- 版本演化注意:`7.0` 起 AOF 从单文件改为 `manifest` + 多分片(`base` + `incr`)，老的直接改写 `appendonly.aof` 的运维脚本要跟着改
- 缓存 vs 数据库的选型话术：纯缓存可以全关持久化(`save ""` + `appendonly no`)换取极限性能；当作"真正的存储"就必须 AOF(everysec 起步)，并接受 `fork` 延迟调优(`vm.overcommit_memory=1`、关闭大页 THP)

### 高可用: 复制、哨兵与集群

- **主从复制**：从库 `PSYNC` 全量同步时，主库 `BGSAVE` 生成 RDB 发给从库，期间新写命令攒在缓冲区，RDB 载入后补发；此后走命令流增量传播，断线重连凭 `offset` 尝试部分重同步，退回全量。读写分离可以扩读，但**主库挂了需要人工切换**
- **哨兵 Sentinel**：三个以上奇数哨兵组成集群做三件事——监控(`PING`)、自动故障转移(主观下线 → 多数派客观下线 → 选新主 → 改客户端配置)、通知。客户端连哨兵而不是直连主库，`Lettuce` 原生支持(上文配置里的 `sentinel.master`)
- **Cluster**:数据分片方案，`16384` 个 `slot` 按 `CRC16(key) % 16384` 分布到多主，每主挂从；节点间 `gossip` 通信，客户端直连任一节点，`MOVED`/`ASK` 重定向到正确 `slot`
  - 意义：突破单机内存与写吞吐上限
  - 代价：**多 `key` 命令要求所有 key 在同一 `slot`**，跨 `slot` 事务与 `Lua` 直接不可用，业务要用 `{hash tag}`(如 `user:{1000}:profile` 与 `user:{1000}:orders` 强制同 `slot`)；运维复杂度显著上升
- 选型：容量单机可扛、只求高可用 → 哨兵；容量或写入需要横向扩展 → Cluster。缓存场景常见的一句实话：**先把单机 `Redis` 的内存用满(垂直扩容)，再考虑 Cluster**，多数业务的 `QPS` 远没到需要分片

## 分布式锁: Redlock 与 Redisson

### 从 SET NX 到看门狗

- 锁的原子获取:`SET lock_key <unique_value> NX PX 30000`，一个命令同时完成"不存在才设置 + 过期兜底"。两个不可省的细节：
  - **value 必须是持有者唯一标识**(UUID + 线程ID):释放时校验"是自己的锁才删"，否则会删掉别人的锁——`A` 超时后锁被 `B` 获取，`A` 缓过来的 `DEL` 会把 `B` 的锁释放
  - **删除必须用 `Lua`**(先 `GET` 比对再 `DEL` 两步要原子)，过期时间 `PX` 是防死锁兜底，不是精确的租期

  ```java
  private static final String UNLOCK = """
      if redis.call('GET', KEYS[1]) == ARGV[1] then
        return redis.call('DEL', KEYS[1])
      else
        return 0
      end
      """;

  public boolean tryLock(String key, String holder, Duration lease) {
    return Boolean.TRUE.equals(redis.opsForValue()
        .setIfAbsent(key, holder, lease));   // SET NX PX
  }

  public void unlock(String key, String holder) {
    redis.execute(RedisScript.of(UNLOCK, Long.class), List.of(key), holder);
  }
  ```

- 手写锁的痛点在**续期**：业务执行超过 `lease` 后锁自动失效，两个持有者并存。`Redisson` 的答案是**看门狗**：不指定 `leaseTime` 时默认锁 30s，后台定时任务每 10s(`lockWatchdogTimeout / 3`)检查持有者还活着就把过期时间重置回 30s；进程崩溃则无人续期，30s 后自然释放

  ```xml
  <dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.27.2</version>
  </dependency>
  ```

  ```java
  @RequiredArgsConstructor
  @Service
  public class SettlementService {
    private final RedissonClient redisson;

    public void settle(long activityId) {
      RLock lock = redisson.getLock("lock:settle:" + activityId);
      try {
        // 不传 leaseTime → 激活看门狗自动续期; waitTime 控制排队上限
        if (!lock.tryLock(3, TimeUnit.SECONDS)) {
          return;  // 别的实例在结算, 幂等跳过
        }
        doSettle(activityId);
      } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
      } finally {
        if (lock.isHeldByCurrentThread()) lock.unlock();
      }
    }
  }
  ```

- `Redisson` 锁默认是**可重入**的(计数器存 `hash` 结构:`field` = 持有者标识, `value` = 重入次数)，还提供公平锁、读写锁、`Semaphore`、`CountDownLatch` 等一整套 `JUC` 同步器的分布式版本

### Redlock 与它的争议

- **主从架构下的锁失效**:`A` 在主库加锁成功，锁还没同步到从库时主库宕机，从库晋升为新主——`B` 在新主上加锁同样成功，两个客户端并存。单实例 `Redis` 锁的可靠性等价于"加锁后主库不在这几百毫秒内宕机"，对多数业务够用，对强正确性场景不够
- **Redlock 算法**：向 N 个(通常 5 个)**完全独立**(无主从复制)的 `Redis` 实例依次加锁，多数派(≥3)成功且总耗时小于锁有效期才算持有；失败则向全部实例释放。它不依赖任何单个实例的数据复制，因此不受上述主从切换问题影响(`Redisson` 对应 `RedissonRedLock`，新版已标记 `@Deprecated`，推荐 `RLock` + `MultiLock`)
- **Martin Kleppmann 与 antirez 的著名论战**(`2016`),面试加分项，两边的核心论点:
  - Kleppmann 反对：① 锁依赖**进程停顿与时钟**——GC pause 或时钟跳变会让客户端在锁已过期后才继续执行关键区，Redlock 没有类似 fencing token 的单调令牌来让存储侧拒绝过期持有者；② 5 个独立实例的运维本身引入新故障模式
  - antirez 回应：时钟跳变可通过运维(NTP 渐进同步)控制；且 fencing token 方案在存储侧不支持校验时同样失效
- **工程结论**，按正确性需求分层：
  - **效率目的**(避免重复干活，偶尔重复无碍)：单实例 `Redis` 锁或 `Redisson RLock`，够了
  - **正确性目的**(重复执行会造成资损，如扣款、转账)：不要用任何 `Redis` 锁，用 `ZooKeeper`/`etcd`(会话租约 + 单调 `zxid`/`revision` 天然可当 fencing token)，或者**让资源侧自身幂等/带版本号**——数据库唯一索引、乐观锁(`WHERE version = ?`)往往比分布式锁更简单也更可靠
  - Redlock 介于两者之间：比单实例强，但依赖时钟假设，争议未决；新项目选它不如直接上 `ZooKeeper`
- 最后一个常见误区：**分布式锁不能替代事务**。锁只保证"同一时刻一个执行者"，不保证执行中途崩溃后的状态一致；关键链路永远是"锁(或队列)串行化 + 资源侧幂等 + 补偿任务"三件套
