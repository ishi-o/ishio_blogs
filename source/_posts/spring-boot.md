---
title: 'Spring Boot: 配置'
categories:
  - Programming
  - Java
  - Spring framework
tags:
  - Spring
  - Spring Boot
  - webapp
mathjax: false
date: 2025-10-09T16:43:42.000Z
---
<!-- placeholder -->
<!-- more -->

# `Spring Boot`配置

## 配置查找

- `Spring boot`按以下顺序注入配置：
  - 命令行传递的配置
  - 环境变量中定义的配置，格式为大写字母用`_`分隔
  - `jvm`的`-D`配置
  - 打包后`jar`包外部定义的配置文件
  - `jar`包同级目录下的配置文件
  - `src/main/resources/config`包内的配置文件
  - `src/main/resources`包内的配置文件
- 支持的配置文件：
  - `application.yml/yaml/properties`
  - `application-{profile}.yml/yaml/properties`：特定命名空间的配置文件，`profile`可完全自定义，通常通过注入环境变量来区分环境或分离配置
  - `application-test.yml/yaml/properties`：仅测试环境会加载
- `messages`国际化消息文件：
  - `i18n/messages.properties`：默认的消息文件，可以通过修改`spring.messages.basename`来修改路径
  - `i18n/messages_zh_CN.properties`：特定语言的消息文件，会优先搜索

## `server`配置

- 依赖于`spring-boot-starter-web`
- `port`：端口
- `servlet`：服务的配置
  - `context-path`：服务的上下文路径，如`/api/v1`
- `tomcat`：内嵌的`tomcat`配置，默认开启（即本身是一个`jar`包而不是`war`包）
  - `accept-count`：等待队列的长度
  - `threads.min-space`：最小的空闲线程数
  - `max-connections`：最大连接数
  - `max-threads`：最大线程数
  - `connection-timeout`：连接超时
- `compression.enabled`：响应压缩启用/禁用
- `ssl`：`https`配置
  - `enabled`：启用`https`

## `spring`配置

### `datasource`配置

- 依赖于`spring-boot-starter-jdbc`
- `type`：默认为`Hikari`连接池
- `url`
- `username`
- `password`
- `driver-class-name`：驱动类的全限定名
- `hikari`：连接池配置
  - `maximum-pool-size`：最大连接数
  - `minimum-idle`：最小空闲连接数
  - `connection-timeout`：连接超时
  - `idle-timeout`：空闲连接的超时
  - `max-lifetime`：连接的最大生存时间

### `redis`配置

- 依赖于`spring-boot-starter-data-redis`
- `host`：主机名
- `timeout`：连接超时
- `connect-timeout`：连接超时
- `port`
- `password`
- `database`：数据库编号
- `lettuce.pool`：连接池配置
  - `max-active`：最大激活连接数
  - `max-idle`：最大空闲连接数
  - `min-idle`

### `cache`配置

- 依赖于`spring-boot-starter-cache`以及具体实现
- `type`：采用的策略，可选`caffeine`、`redis`、`simple`、`none`
- `redis`：`redis`的缓存配置
  - `time-to-live`：缓存过期时间
  - `cache-null-values`：是否缓存`null`

### `logging`配置

- 依赖于`spring-boot-starter-logging`，默认使用`logback`
- `level`：
  - `root`：默认日志级别
  - `org.springframework.web`：指定包的日志级别，优先于`root`
  - `com.xxx`：同上
- `file.name`：日志文件路径
- `pattern`：日志编码（或者说输出格式）
  - `console`
  - `file`
- `logback.rollingpolicy`：
  - `max-file-size`：单个日志文件最大大小
  - `max-history`：日志保留的最长时间

### `jpa`配置

- 依赖于`spring-boot-starter-data-jpa`
- `hibernate`：经典实现`hibernate`的配置
  - `ddl-auto`：`update`每次启动时更新表，`create`每次启动时创建表，`none`表示什么也不干
- `show-sql`：表示是否将自动生成的`sql`显示出来

### `rabbitmq`配置

- 依赖于`spring-boot-starter-amqp`
- `host`
- `port`
- `username`
- `password`
- `listener`：消息中间件的监听者
  - `simple`：
    - `concurrency`：最小消费者数量
    - `max-concurrency`：最大消费者数量

### `rocketmq`配置

- 依赖于`rocketmq-spring-boot-starter`

### `eureka`服务发现

- 依赖于`spring-cloud-starter-netflix-eureka-client`

### 分布式配置中心

- 依赖于`spring-cloud-starter-config`

### `mybatis`配置

- 依赖于`mybatis-spring-boot-starter`

### `web.cors`配置

- `allowed-origins`
- `allowed-methods`
- `allowed-headers`
- `allow-credentials`

### `security`配置

- 依赖于`spring-boot-starter-security`
- `oauth2`：
  - `resourceserver.jwt`：

### 其它配置

- `application.name`：应用名（用于服务发现）
- `servlet.multipart`：针对文件内容的配置
  - `max-file-size`：最大文件大小
  - `max-request-size`：请求文件的最大大小
- `jackson`：`json`序列化配置

### 自定义配置导入

- 此前通过`@PropertySource`加载自定义配置文件
- `Spring Boot 2.4+`更支持通过`spring.config.import`数组来加载
- 通过`@ConfigurationProperties(prefix = "xxx")`来将加载后的配置绑定到一个具体的配置类里

