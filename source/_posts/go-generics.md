---
title: 'Go: 泛型'
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - generics
mathjax: false
date: 2025-12-25T11:37:04.000Z
---
<!-- placeholder -->
<!-- more -->

# `Go`的泛型

## 通用接口

- `Go`的泛型在`1.18`引入，同时为了服务泛型，更改了`interface`的定义为“一系列类型的集合”
- 类型集用于约束或简化声明泛型参数`T`所能表示的类型
- 示例：

  ```go
  type SignedIntType interface {
  	int8 | int16 | int | int32 | int64
  }
  ```

- 并集：通过`|`获得两个类型集的并集

  ```go
  type IntType interface {
  	SignedIntType | UnsignedIntType
  }
  ```

- 交集：通过换行来分隔多个类型以获取交集

  ```go
  type EmptyType interface {
  	SignedIntType
  	UnsignedIntType
  }
  ```

- 空集：空集不是空接口，空集只能通过交集运算获得，而空接口`interface{}`或`any`可以表示所有类型
- 使用`type`声明的新类型：即使底层类型被包含在一个类型集中，只要这个新类型不在这个类型集中，就无法用作该泛型的参数

  ```go
  type A interface {
  	int8
  }

  // TinyInt is not a A !
  type TinyInt int8
  ```

- 底层类型声明：可以使用`~`解决上述问题

  ```go
  type A interface {
  	~int8
  }

  // TinyInt is a A
  type TinyInt int8
  ```

  `~int8`表示只要一个类型的底层类型是`int8`，那么这个类型就属于这个类型集

- 需要记住：方法集是一个类型(一般接口)，类型集不是一个类型(通用接口)
  - 类型集不能通过类型断言(因为不是一个类型)，但是泛型参数`T`可以(因为`T`是类型)
  - 类型集本身不能作为泛型实参，它本身不是一个类型而是一个类型集
  - 方法集可以作为泛型实参，它本身是一个接口类型
- 不兼容类型之间不能相并(不能使用`|`)，它们只能通过换行获取交集
  - `comparable`不能和其它类型相并
  - 方法集之间不能相并

## 声明泛型参数

- 泛型函数：

  ```go
  func Func[T any](a T, ...) (...) {
  }
  ```

- 新类型：

  ```go
  import "io"

  type (
  	NewType[T int | float64] []T
  	NewType[T int | float64] struct {
  		a T
  	}
  	NewType[T int | float64, S string | rune] struct {
  		k T
  		v S
  	}
  	NewType[T ATypeInterface] []T
  )

  func Write[W io.Writer](w W, bs []byte) (int, error) {
  	return w.Write(bs)
  }
  ```

- 当针对引用类型需要使用泛型时，推荐将`T`作为基本数据类型，然后使用`[]T`、`map[T]T1`、`chan T`
- 声明泛型参数时不能使用交集运算，可以临时使用并集运算
- 方法(有接收器的函数)不能定义泛型参数，但接收者可以携带泛型参数并供方法使用
- 匿名结构体和匿名函数不支持临时定义泛型参数，但可以使用外部定义的泛型参数

