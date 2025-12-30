---
title: 'Go: 序列化与反序列化'
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - serializing
  - config
mathjax: false
date: 2025-12-30T08:50:23.000Z
---
<!-- placeholder -->
<!-- more -->

# `Go`的序列化

## 特性

- `Go`通过结构体标签来支持序列化和反序列化，内置的`encoding`标准库支持`xml/json`等配置文件的序列化和反序列化
- `gopls`本身支持`json`结构体标签的快速生成
- `Go`只会对一个结构体的公有字段进行序列化和反序列化

## `API`

- 通过`encoding`下的子包`encoding/xml`或`encoding/json`的`API`
- `Marshal(v any) ([]byte, error)`：将任意值序列化，返回字节数组
- `MarshalIndent(v any) ([]byte, error)`：类似`Marshal()`但是会提供缩进美化
- `Unmarshal(data []byte, v any) error`：反序列化，`v`应当是指针

## `xml-etree`

- [`beevik/etree`](https://github.com/beevik/etree)是用于解析`xml`的第三方库

## `yaml`

- 需要借助第三方库[`go-yaml/yaml`](https://github.com/go-yaml/yaml)
- 用法与`encoding API`一致，只不过`yaml`是严格缩进的，因此没有`MarshalIndent()`

## `protobuf`

- 需要借助第三方库[`github.com/golang/protobuf/proto`](https://github.com/golang/protobuf)

