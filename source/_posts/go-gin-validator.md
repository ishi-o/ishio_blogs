---
title: 'Gin: 数据校验'
categories:
  - Programming
  - Go
  - 3P Framework
  - Gin
tags:
  - Go
  - validation
  - webapp
mathjax: false
date: 2025-12-27T22:31:56.000Z
---
<!-- placeholder -->
<!-- more -->

# [`validator`](https://github.com/go-playground/validator)

## 标签

### 语法介绍

- `gin`框架依赖于`validator`，`validator`是基于结构体标签和反射进行数据校验的框架
- `validator`提供的结构体标签是`validate`：

  ```go
  type A struct {
  	Pub string `validate:"endwith=abc"`
  }
  ```

- `validator`提供的标签在`baked_in.go`中
- 标签的不同规则通过`,`相与，通过`|`相或，标签的参数通过`=`传递，`oneof`标签的不同参数通过空格` `分隔，`-`表示跳过验证，特殊字符(`-`、`,`、`=`、`|`)需要`unicode`转义

  ```go
  type A struct {
  	Pub    string `validate:"requried,min=3,max=8"`
  	Enum   string `validate:"required,oneof=admin rdonly user"`
  	Abc    string `validate:"required,eq=abc0x3d"`
  	Ignore int    `validate:"-"`
  }
  ```

  `=`为`0x3d`，`,`为`0x2c`，`|`为`0x7c`，`-`为`0x2d`

- 与和或不支持使用括号，只能从左到右校验

### 通用校验

- `eq/lt/gt/ne/lte/gte=xxx`：等于/小于/大于/不等于/小于等于/大于等于校验
- `required`：强制要求
  - 数值/布尔值：允许为零值
  - 结构体/字符串：禁止为空结构体/空串
  - 不允许为`nil`
- `min/max`：限制数值范围
- `omitempty`：可选字段，即可以是`nil`或零值，如果不为`nil`则要经过后续的校验
- `omitnil`：可选字段，可以是`nil`，但如果是零值则要经过后续校验
- `oneof=xxx1 xxx2`：限制可选值范围，可用于枚举
- `len=xxx`：限制长度
- `required_if=Field xxx`：`Field`为`xxx`时强制要求
- `unique`：不允许重复

### 常用格式校验

- `email`：是否为邮箱格式
- `uuid/uuid4`：`uuid`/`uuid4`校验
- `json`：`json`格式校验
- `datetime=DateTimeFormat`：日期时间验证
- `base64`：`base64`格式校验

### 常用字符串校验

- `alphanum/alpha/numeric`：是否只包含数字字母/字母/数字
- `contains=xxx`：是否包含`xxx`子串
- `containsany=xxx`：是否包含`xxx`字符集中的任何字符
- `excludes=xxx`：是否不包含`xxx`子串
- `excludesall=xxx`：是否不包含`xxx`字符集中的任何字符
- `startswith=xxx/startsnotwith=xxx`：是否以/不以指定文本开始
- `endswith=xxx/endsnotwith=xxx`：是否以/不以指定文本结束
- `uppercase/lowercase`：是否全大写/小写

### 常用跨字段校验

- `validator`比起`java`的`bean validation`，还支持跨字段验证
- `eqfield/nefield=OtherField`：本字段和指定字段相等/不相等
- `gtfield/gtefield/ltfield/ltefield=OtherField`：大于/大于等于/小于//小于等于指定字段
- `csfield`系列可以跨结构体校验，但是不常用

### 常用网络字段校验

- `ip/ipv4/ipv6`：通用`ip`/`ipv4`/`ipv6`地址格式校验
- `url/uri`：`url`/`uri`格式校验
- `hostname`：主机名校验
- `fqdn`：完全限定域名校验

## 验证`API`

### 创建验证器

- 使用`validator.New() *validator.Validate`创建验证器
- 通常使验证器为单例模式

### 结构体校验

- `*Validate.Struct(any) error`：验证传入的结构体的所有**公开字段**，即首字母大写的字段，默认进行递归校验
- 应该传入结构体指针，否则返回`InvalidValidationError`
- 如果验证出错，将返回`ValidationErrors`，被封装成`error`
- 提取校验错误：

  ```go
  err := validate.Struct(a)
  for _, err := range err.(validator.ValidationErrors) {
  }
  ```

  其中`err`即`validator.FieldError`

- `FieldError`提供若干方法：<!--TODO: validator.FieldError-->

### 其它数据结构的校验

- 虽然通常用于结构体，但也可以用于其它数据结构体的校验
- `map`对象：使用一个和`map`对象结构相同的`map`对象，其中每个键对应的值是标签数据

  通过`*Validate.ValidateMap(data, rules) map[string]any`校验，返回错误`map`，每个键对应一个`FieldError`

- 切片：通过`*Validate.Var(any, tag string)`校验，从左到右维度下降(切片本身到切片的每个元素)，通过`dive`标签划分不同的维度

  多维切片道理类似，通过`dive`标签划分不同维度

- 一般变量：同样通过`Var()`进行验证

## 自定义标签

### 自定义别名

- `*Validate.RegisterAlias(alias, tag string)`：将一个`tag`命名为`alias`，则验证时会直接展开成`tag`

### 自定义验证函数

- `RegisterValidation(tag string, fn Func, callValidationEvenIfNull ...bool) error`：通过注册函数注册
- `Func`是一个函数，签名为`func(fl FieldLevel) bool`，其中`FieldLevel`接口是一系列返回`reflect.Value`的快捷函数，最常用的是`Filed()`和`Param()`，表示获取当前字段的值和标签参数
- `fn`返回`true`表示校验成功

## 与`Gin`的集成

- 在`gin`中，标签使用`binding`而不是`validate`
- 在`gin`中，不需要显式创建验证器和显式调用验证方法，所有校验错误会在解析数据、绑定数据失败时返回`error`

