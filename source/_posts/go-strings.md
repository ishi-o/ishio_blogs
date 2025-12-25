---
title: 'Go: 字符串'
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - string
mathjax: false
date: 2025-12-24T18:13:16.000Z
---
<!-- placeholder -->
<!-- more -->

# 字符串

## 特性

- `string`是一个值类型，意味着可以是空串但不是`nil`
- `string`是一个不可变的`byte`数组，是不可变类型
- `string`对象本身可以被取地址，但它的元素不允许
- 并且由于本质是字节数组，通过数组访问符`[]`访问只能得到`byte`而不是`rune`并且也没有`char`类型
- `string`与`[]byte`和`[N]byte`可以相互转换，要修改`string`需要先转换为字节数组，为了内存安全，转换这一行为会分配新的内存给字节数组
- 原始字符串：用反引号包围

## 字符串操作

- 由于字符串本质是字节数组，许多数组/切片的操作用于`string`是很容易理解的
- 拼接：`Go`内置了`+`和`+=`字符串运算
- 字符串可以直接通过切片符获取子串，但本质是获取字节数组的切片
- `len()`：本质是求字节数组的长度，大部分情况字符是`ASCII`码因此只需要一个字节
- `copy()`：拷贝操作
- 按字符处理：先转换成`[]rune`，即`int32`序列再处理
- 按字符遍历：`range`的最小单位就是`int32`，因此：

  ```go
  for _, ch := range str {
  }
  ```

  这样的遍历是按`rune`遍历的

## `strings`

- `strings`是内置的字符串工具包，用于处理`utf8`字符串

### 创建与拷贝

- `Repeat(s string, cnt int) string`：按次数拷贝字符串
- `Clone(string) string`：拷贝字符串，若不为空则分配内存并返回新的字符串对象
- `Builder`结构体：用于通过消耗更小的内存来构建复杂字符串(相比使用`+`)
  - 实现了`io.Writer, io.ByteWriter, io.StringWriter`接口：通过`WriteXxx()`方法构建字符串，
  - `Grow(n int)`：预分配内存
  - `Reset()`：重置`Builder`
  - `String()`执行构建
  - `Builder`不能作为参数传递
- `Join([]string, rep string) string`：拼接字符串切片，分隔串为`rep`，比`+`性能更好

### 比较

- `Compare(a, b string) int`：按字符比较两个字符串，返回`1`，`0`，`-1`
- `EqualFold(s, t string) bool`：忽略大小写情况下是否相等

### 包含与匹配

- `Contains(s, substr string) bool`：判断前者是否包含`substr`子串
- `ContainsAny(s, chs string) bool`：判断前者的字符集是否包含`chs`字符集
- `Count(s, substr string) int`：前者包含多少次后者
- `HasPrefix(s, prefix string) bool`：判断前者是否有前缀`prefix`
- `HasSuffix(s, suffix string) bool`：判断前者是否有后缀`suffix`
- `Index(s, substr string) int`：返回子串第一次出现的位置
- `IndexAny(s, chs string) int`：返回字符集第一次出现的位置
- `IndexRune(s, chs rune) int`：返回指定`unicode`字符第一次出现的位置
- `IndexLastXxx`：上述三者都有`Last`版本

### 分割

- `Fields(s string) []string`：按空格分割字符串，类似`split`
- `FieldsFunc(s string, f func(rune) bool) []string`：按函数`f`的返回值决定是否分割

  ```go
  // 等价于`Fileds(s)`:
  FiledsFunc(s, func(ch rune) bool {
  	return ch == ' '
  })
  ```

- `Split(s, sep string) []string`：按子串`sep`分割
- `SplitN(s, sep string, n int) []string`：按子串`sep`分割，返回值最多`n`个元素
- `SplitAfter(s, sep string)`：与`Split`类似但是会保留分隔子串在每个元素的结尾
- `SplitAfterN(...)`

### 删除与替换

- `Cut(s, sep string) (before, after string, found bool)`：删除`s`中第一次出现的`sep`，`before`表示子串位置之前的串，`after`表示子串位置后的串
- `Map(mapping func(rune) rune, s string) string`：遍历替换`unicode`字符，若`mapping`返回`-1`则不进行替换而是直接删除该`unicode`字符
- `Replace(s, old, new string, n int) string`：将`s`中的`old`子串替换为`new`串，共能替换`n`次，`n`为负数时表示无限次
- `ReplaceAll(s, old, new string) string`：等价于`Replace(s, old, new, -1)`
- `ToUpper()`与`ToLower()`
- `Trim(s, cutset string) string`：删除两端只包含`cutset`字符集的子串
- `TrimLeft()`与`TrimRight()`
- `TrimPrefix(s, substr string) string`与`TrimSuffix(...)`：与上述四者不同，删除完全匹配`substr`子串的前缀或后缀
- `TrimSpace(s string)`
- `TrimFunc(s string, f func(rune) bool)`
- `*Replacer`：提前指定替换规则后复用，通过`NewReplacer(oldnew ...string) *Replacer`创建

  ```go
  import strings
  rep := strings.NewReplacer(old1, new1, old2, new2)
  str = rep.Replace(str)
  ```

