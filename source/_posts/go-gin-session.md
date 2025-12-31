---
title: 'Gin: 会话管理'
categories:
  - Programming
  - Go
  - 3P Framework
  - Gin
tags:
  - Go
  - session
  - webapp
  - jwt
  - cookie
mathjax: false
date: 2025-12-30T15:20:05.000Z
---
<!-- placeholder -->
<!-- more -->

# `Gin`会话管理

## `session`

### 注册会话

- `Gin`本身不支持会话管理，需要第三方框架[`github.com/gin-contrib/sessions`](https://github.com/gin-contrib/sessions)
- `sessions.Store`表示一个存储引擎，`sesssion`的常用存储方式有三种：内存存储、`cookie`存储、`redis`存储（还支持`MongoDB`、`GORM`和`PostgreSQL`）

  这三种存储方式的实现在`sessions/memstore`、`sessions/cookie`、`sessions/redis`中，区别和`gin`没有关系，属于常识：

  内存存储和`redis`存储都是将会话信息存储在服务端然后使用`cookie`传输`sessionId`等会话的元数据，而`cookie`存储是将会话信息完全由`cookie`传输并存储在客户端浏览器上

- `cookie`引擎（通常附带密钥增加安全性）：

  ```go
  cookie.NewStore(
    []byte("authentication-key-32-bytes-long"),
    []byte("encryption-key-32-bytes-long")
  )
  ```

  通常附带两个密钥，第一个会被用于认证密钥（使客户端无法写会话内容），第二个会被用于加密密钥（使客户端无法读会话内容）

- 内存引擎（不支持集群）：`memstore.NewStore()`，也可以传递密钥，用于保护`cookie`的元数据
- `redis`存储（支持集群）：

  ```go
  func redis.NewStore(
   size int,
   network string,
   address string,
   username string,
   password string,
   keyPairs ...[]byte) (redis.Store, error)
  ```

  从上至下为连接池大小、传输协议、`redis`地址、`redis`用户名、`redis`密码、`keyPairs`与上述两种引擎一致，都是为了防止客户端修改和读取

- 通过`sessions.Sessions(name string, store sessions.Store) gin.HandlerFunc`获取`gin`的中间件，`name`就是传输给浏览器的`cookie`的名字
- `Store`包含以下可配置项：

  ```go
  import (
  	"net/http"

  	"github.com/gorilla/sessions"
  )

  func A() {
  	store.Options(sessions.Options{
  		Path:   "/", // cookie 生效的路径, 只有在请求响应该路径或子路径时才会发送
  		Domain: "",  // cookie 生效的域名, 空表示当前域名
  		// MaxAge=0 浏览器关闭即删除
  		// MaxAge<0 响应收到后即删除
  		// MaxAge>0 指定秒数后即删除
  		MaxAge:   0,
  		Secure:   false, // 是否仅允许 https
  		HttpOnly: true,  // 禁止 js 访问 cookie (与加密不同, 这是直接无法读取)
  		// 跨站请求是否发送 cookie, 防 CSRF
  		// 要么 NoneMode+secure=true 要么 LaxMode
  		SameSite: http.SameSiteLaxMode,
  	})
  }
  ```

### 会话`API`

- `sessions.Default(*gin.Context) sessions.Session`：获取和当前请求绑定的`session`实例
- `Session.Get(key any) any`：获取某个键对应的值
- `Session.Set(key, value any)`：设置某个键值对
- `Session.Delete(key any)`：删除某个键
- `Session.Save() error`：执行保存当前`session`的修改

## `JWT`

