---
title: go-error
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - exception
mathjax: false
date: 2025-12-25T11:37:12.000Z
---
<!-- placeholder -->
<!-- more -->

# 错误处理

## 级别分类

- `error`：正常的运行时错误，需要处理，若忽略程序也不会崩溃
- `panic`：恐慌，程序应该在处理完问题后立刻退出
- `fatal`：致命错误，程序无法处理并立刻退出

## 错误处理的设计

- `Go`没有`try-catch`语句，一切`error`(如果有)都作为返回值返回
- 因此处理变得清晰：

  ```go
  res, err := ThrowSomeError()
  if err != nil {
  	// process or:
  	return err
  }
  ```

- 但有若干缺点：
  - 堆栈丢失：标准库的错误信息不包含堆栈，需要引入非标准库
  - 代码冗余：有极多的`if err != nil`，不像`try-catch`那样有统一的结构化处理
- 但实际上也有一些优点：
  - 性能高：没有`try-catch`开销
  - 虽然错误信息不包含堆栈，但`error`本质就是返回值，可以一层一层向上调试找到
  - `error`作为值，更多时候作为返回值，能增强可读性，知道错误出自哪里
  - 编译时检查减轻处理负担：编译器或静态检查要求必须接收函数的返回值，当一个函数有可能抛出`error`时，编译器会提醒显示接收并处理(或`return`)

    而使用结构化处理的语言则总会遇到：当发生错误但没有显式处理时，直接使用`try-catch`块包围，带来性能损耗、难以定位

    它们更难受的问题在于，容易遗漏运行时错误而没有处理

- 与另一个将错误作为返回值的语言`rust`对比，`rust`提供了语法糖来处理错误，因此同样作为返回值却没有引起很大的讨论

## `error`

### 声明`error`

- `error`是预定义的类型

  ```go
  type error interface {
  	Error() string // 获取错误信息
  }
  ```

- `errors`是错误处理的工具包
- 创建错误：`errors.New(msg string)`

  也可以通过`fmt`创建：`fmt.Errorf(msg string, ...any)`传入格式化信息

  在没有链式调用的前提下，它们返回内置的`errorString`结构体

- `error`理应是一个常量(哨兵错误)，但在设计上却是通过`var`声明的，是一个变量

  为了可维护性，一般将常用的`error`当作全局变量预先声明

- 自定义`error`：`error`是一个接口，只要类型实现了`Error()`，就是一个`error`

### 链式错误

- 链式错误是`Go 1.13+`提供的用于获取链式调用信息的错误
- 通过`fmt.Errorf("%w", err)`可以包装一个错误并返回一个实现了`Unwrap()`的`error`，它返回的对象本质是内置的`wrapErrors`
- 因为`wrapErrors`不是预定义的，它是包私有的，所以只能通过`errors.Unwrap(error)`解包，如果传入的`error`没有实现`Unwrap()`则返回`nil`
- `errors.Is(err, target error) bool`：判断`err`的链式错误中是否包含`target`，即省去了自己写`for`的过程
- `errors.As(err error, target any) bool`：判断`err`的链式错误中是否包含匹配`target`的类型错误，即在`Is`的基础上传递的`target`变为指针的指针并赋值，将第一个匹配成功的错误赋值给`target`并返回`true`
  - `target`必须实现`error`
  - `target`必须是指针的指针

- ```go
  package main

  import (
  	"errors"
  	"fmt"
  )

  func main() {
  	// Is: 判断是否包含
  	tgt := Myerr{"msg"}
  	if errors.Is(err, tgt) {
  	}
  	// As: 判断是否有对应类型的错误并赋值
  	// 传入指针的指针，才可在函数内修改外部的错误变量
  	var tgt *Myerr
  	if errors.As(err, &tgt) {
  		fmt.Print(tgt.String())
  	}
  }
  ```

### 平级错误

- 通过`errors.Join(...error) error`创建一个封装了`[]error`的错误，它的返回值不是链式错误，而是`joinError`
- `joinError`实现了`Unwrap() []error`，`errors.Is()`与`errors.As()`都有单独处理它们的分支，所以可以像链式错误那样使用它们
- 但是`errors.Unwrap()`中不包含单独的判断，所以只会返回`nil`

### `github.com/pkg/errors`

- 这是另一个官方的`errors`包，但没有加入标准库
- 它会自动记录栈信息
- 但实际上，它的链式调用已被标准库替代，而堆栈记录已经被许多第三方依赖所吸收，早已处于归档状态

## `panic`

### 创建

- `panic(string)`内置函数用于自定义`panic`，但并不常用
- 仅有在程序真的不会继续执行的时候，才使用`panic()`，或者使用日志框架

  例如数据库连接失败时，使用`panic()`

- `panic`会进行善后，比如执行事先定义的`defer`语句

  `defer`语句按先进后出的顺序执行

- 关于`defer`：`defer`后跟函数的参数是会被预计算的

### 恢复

- `recover()`可用于恢复`panic`并不退出程序
- `recover()`必须在`defer`块中执行
- `panic`向上传递，即向调用者方向传递，因此如果在`defer`块内的闭包函数调用`recover()`是无效的

