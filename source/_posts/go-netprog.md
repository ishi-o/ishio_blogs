---
title: 'Go: 网络编程基础'
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - net
  - webapp
mathjax: false
date: 2025-12-25T11:39:18.000Z
---
<!-- placeholder -->
<!-- more -->

# `net`

## `net/http`

### 服务端

- `http.Server`结构体的常用配置项：
  - `Addr`：监听端口
  - `Handler`：请求处理器
  - `XXXTimeout`：一些超时控制
- `http.ListenAndServe(addr string, handler Handler) error`：启动一个服务器，如传入`(":8000", nil)`

  或调用`http.Server.ListenAndServe()`方法

- 处理器实现：`Handler`接口包含`ServeHTTP(ResponseWriter, *Request)`方法，可以针对请求进行不同的写入
- `http.Handle(pattern string, handler Handler)`：将处理器绑定到`pattern`路径上路由
- `http.HandleFunc(pattern string, handler func(ResponseWriter, *Request))`：可以直接传入一个匿名函数，内部通过适配器转换成一个`Handler`对象

### 反向代理

### 客户端

- `http.Client`结构体的可配置项：
  - `Transport RoundTripper`：连接池配置
  - `CheckRedirect func(req *Request, via []*Request) error`：重定向控制函数
  - `Jar CookieJar`：`cookie`配置
  - `Timeout time.Duration`：超时配置

### 独立请求

- `Get(url string) (resp *Response, err error)`：发起`get`请求
- `Post(url, contentType string, body io.Reader) (resp *Response, err error)`：发起`post`请求
- `Response`结构体的常用字段：
  - `StatusCode`：状态码
  - `Header`：响应头，是`map[string][]string`
  - `Body`：响应体，是`io.ReadCloser`，需要`Close()`，只能读一次
- `Response`的常用方法：
  - `Cookies() []*Cookie`：提取`Header`中的`Cookie`
- `http.StatusXXX`：状态码常量

