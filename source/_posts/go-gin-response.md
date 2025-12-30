---
title: 'Gin: 响应处理'
categories:
  - Programming
  - Go
  - 3P Framework
  - Gin
tags:
  - Go
  - net
  - webapp
mathjax: false
date: 2025-12-28T14:03:18.000Z
---
<!-- placeholder -->
<!-- more -->

# `Gin`的响应

## 响应方法

- 以下方法都是`*gin.Context`的方法：
- `Status(code int)`：设置状态码
- `Header(key, value string)`：设置响应头
- 以下方法能够自动设置状态码和`Content-Type`响应头
- `String(code int, format string, values ...any)`：字符串响应
- `JSON(code int, obj any)`：JSON响应
- `XML(code int, obj any)`：XML响应
- `YAML(code int, obj any)`：YAML响应
- `ProtoBuf(code int, obj any)`：Protobuf响应
- `HTML(code int, name string, obj any)`：HTML模板渲染
- `File(filepath string)`：文件响应（显示文件）
- `FileFromFS(filepath string, fs http.FileSystem)`：文件响应（从`fs`中获取）
- `FileAttachment(filepath, filename string)`：文件响应（请求浏览器下载文件）
- `Data(code int, contentType string, data []byte)`：原始数据
- `Stream(step func(w io.Writer) bool)`：流式响应
- `Redirect(code int, location string)`：重定向
- `code`通常传递`http.StatusXxx`
- `obj`直接传递待发送的对象即可，针对模板文件它们是传入模板文件的数据，需要事先使用`*gin.Engine.LoadHTMLFiles()`或`LoadHTMLGlob()`

## 上传文件

- `FormFile(name string) (*multipart.FileHeader, error)`：传入文件`name`，获取
- `MultipartFrom() (*multipart.Form, error)`：获取完整的`multipart`表单
- `SaveUploadedFile(file *multipart.FileHeader, dst string, perm ...fs.FileMode) error`：保存文件到本地磁盘

