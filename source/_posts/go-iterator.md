---
title: go-iterator
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - data structure
mathjax: false
date: 2025-12-25T11:37:27.000Z
---
<!-- placeholder -->
<!-- more -->

# 迭代器

## `range`

### 特定数据结构的迭代

- 在一开始，`range`只能用于数组、切片、字符串、`map`、可读`chan`的迭代
- 数组/切片：

  ```go
  for idx, val := range slice {
  }
  ```

- `map`：

  ```go
  for key, val := range mapp {
  }
  ```

- 字符串：

  ```go
  for idx, bt := range str {
  }
  ```

- 可读通道

  ```go
  for elem := range channel {
  }
  ```

### 整型迭代

- `Go 1.22+`引入了整型迭代语法糖

- ```go
  for i := range n {
  }
  // 等价于
  for i = 0; i < n; i++ {
  }
  ```

- 同时引入了不含索引变量的版本：

  ```go
  for range n {
  }
  ```

## 迭代器函数

### 推送式迭代器

- 在`Go 1.22`，引入了`Range over func`，即迭代器函数，迭代器函数会降低一定的可读性
- 推送式迭代器函数的形式：

  ```go
  func Func(...any) func(yield func(T) bool) {
  	// 环境声明...
  	var a T
  	// 返回闭包
  	// 若干次调用 yield(a) ... a 即推送给 临时变量 v 的值
  	return func(yield func(T) bool) {
  		// for 中的 break / return 将使 yield() 返回 false
  		if !yield(a) {
  			return // return 退出闭包函数, 外部 for 结束
  		}
  		// 迭代 a
  		a = DoSth(a)
  	}
  }

  func A() {
  	// 使用
  	for v := range Func(n) {
  		// body
  	}
  }
  ```

- 迭代器函数返回一个闭包函数，闭包函数返回空，接收一个回调函数`yield`

  回调函数`yield`由运行时自动注入，是外部开发者定义的`for`块，即：

  ```go
  for v := range Func(n) {
  	// body
  }
  // 等价于
  iterator := Func(n) // 获取提供了指定环境的闭包函数
  // 传递 for 块作为 yield 传递给闭包函数
  // 其中 v 在闭包中由开发者主动推送
  iterator(func(v T) bool {
  	// body
  	return true
  })
  ```

- `for`体默认在结尾返回`true`，在遇到`break`、`return`等时，体现在`yield()`里则是返回`false`
- 在一切正常时(`for`体没有`break`和`return`)，`for`执行多少次完全取决于闭包函数内`yield()`被调用了多少次

  简单的理解就是：`range`后跟自定义的迭代器函数，迭代器函数返回一个闭包函数

  闭包函数提供`v`的序列，通过`yield()`一轮一轮地推送

  `yield()`是`for`内的代码块，在结尾自动添加`return true`以及在特定关键字后添加`return false`

- `yield()`最多支持两个参数，允许只接收一个参数但没有意义
- `Go 1.23`引入了`iter.Seq[T any]`与`iter.Seq2[T, S any]`，它们简化了迭代器函数的返回值，推送式迭代器和原生的`for range`的性能相差不大

### 拉取式迭代器

- 推送式迭代器的迭代逻辑全部位于闭包函数内，而很多时候迭代器需要由`for`块主动调用，这种迭代器称为拉取式迭代器
- `Go 1.23`引入了`iter.Pull[V any](seq Seq[V]) (next func() (V, bool), stop func())`和`iter.Pull2(...)...`，用于将推送式迭代器转化为拉取式迭代器
- 这个实现原理是：推送式作为生产者会不断调用`yield`，而`Pull`在中间添加了一个协程，用于伪装一个`yield`(并不是`for`体)，这个协程提供的`yield`收到推送式迭代器提供的参数后转发给`next()`，并阻塞式地等待返回值，从而实现阻塞式地迭代

  那么在调用`next()`时，推送式迭代器调用的`yield`就是协程提供的`yield`

- 但是这种实现方式并不常用，因为先实现一个推送式迭代器再启用一个协程带来更大的复杂度和性能损耗，但和使用闭包并没有很大的区别
- `stop()`则是向生产者协程主动发送停止信号，让它停止阻塞式地等待消费者的信号

  ```go
  import "iter"

  func A() {
  	next, stop := iter.Pull[int](seq)
  	defer stop()

  	for range n {
  		v := next()
  	}
  }
  ```

## 标准库的迭代器

### `slices`

- `All[Slice ~[]E, E any](s Slice) iter.Seq2[int, E]`：将切片转换成切片迭代器，内部实际上就是`for range s`，一般用于数据流处理
- `Values[Slice ~[]E, E any](s Slice) iter.Seq[E]`：转换成不返回索引的切片迭代器
- `Chunk[Slice ~[]E, E any](s Slice, n int) iter.Seq[Slice]`：返回若干子切片，切片最多含有`n`个元素
- `Collect[E any](iter.Seq[E]) []E`：将切片迭代器收集成切片

### `maps`

- `Keys()`：返回键的迭代器
- `Values()`：返回值的迭代器
- `All()`：返回键值对的迭代器，和直接使用`for range`类似，一般用于数据流处理
- `Collect()`：将映射表迭代器收集成映射表

## 流式处理

- `Go`采用返回闭包并封装成标准库函数的形式实现迭代器函数，而不是声明一个流对象并以流对象作为接收者和返回值来实现(像`Java`那样)
- 因此`Go`的标准库并不提供链式调用的流式处理，因此调用链长度较长时可读性会下降
- 因此需要自己实现流式调用形式的，即自己定义一个流对象并封装一系列它的方法，在内部调用`slices`和`maps`提供的方法
- `Go`目前还不支持匿名函数的简写，需要完整写完函数定义和函数体

