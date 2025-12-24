---
title: 'Go: 基本语法'
categories:
  - Programming
  - Go
  - Stdlib
tags:
  - Go
  - beginner
mathjax: false
date: 2025-12-23T22:48:43.000Z
---
<!-- placeholder -->
<!-- more -->

# `Go`基本语法

## 包管理

- `package`关键字表示当前文件属于哪个包，主函数文件想要当作入口文件，必须声明`package main`
- `import`是导入其它包的关键字
- `import`可以通过`()`对导入进行分组，也可以重命名：

  ```go
  package main

  import (
  	"fmt"
  	strs "strings"
  )

  func main() {
  	fmt.Print("Hello world!")
  	strs.Split("a,b,c", ",")
  }
  ```

- `import`起别名时可以用`_`表示匿名引入，只加载其中的`init`函数

  ```go
  import _ "mysql-driver-go"
  ```

- 内部包：`Go`约定，在命名为`internal`包内的函数无法被外部包调用

  可以导入内部包的文件，只有该内部包直接父目录的所有子文件

## 数据类型

### 基本数据类型

- 整型：
  - `uint8/16/32/64`：无符号整k型
  - `int8/16/32/64`
  - `uint`和`int`：至少`32`位的整型
  - `byte`：等价于`uint8`
  - `uintptr`：保证足够大以至于能存储任何指针
- 浮点型：`float32/64`
- 复数：`complex64/128`，本质为两个`float32/64`的封装
- 字符型：
  - `byte/uint8`：可存储`ASCII`码
  - `rune`：等价于`int32`，可存储`utf8`字符，用`rune`来在语义上区分整型和字符，也可表示一个`Unicode`
  - `string`：字符序列，可视为一个只读`[]byte`，它同样可以表示`utf8`(编码为`byte`序列后)字符串

    `string`是一个不可变类型

- 布尔型：`bool`

### 衍生类型

- 任意类型：`any = interface{}`
- 可比较类型：`comparable = interface{ comparable }`

  `comparable`应该作为函数参数的类型以限制形参，而不是用来定义一个变量

  所有基本数据类型、以及由可比较类型组成的数组、以及由可比较类型组成的`struct`都是`comparable`的

- 数组类型：`[n]Type`，其中`n`必须是编译时可确定的

  数组也是一种值类型

- 切片类型：`[]Type`，是可变长的序列
- 映射类型：`map[keyType]valueType`
- 指针类型：`*Type`
- 结构体：`struct {}`自定义类类型
- 接口：`interface {}`自定义接口类型
- 方法：`func(T...) R`自定义方法
- 通道类型：`Go`擅于多线程场景，通道是`Go`提供的线程间通信手段
  - `chan Type`：双向通道，`chan`表明它是一个通道，`Type`是通道传输的数据的类型
  - `chan<- Type`：只写通道
  - `<-chan Type`：只读通道
- 切片、映射、通道三大类型都是引用类型

### 字面量

- `true`和`false`
- 整型可使用`_`分割加强可读性
- `iota`：是预定义的字面量，**只在`const`块下生效**，表示**自增的自然数序列**，即`0-idx`

  `const`组的一个特性是：后续没有显式赋值的常量会延用最近使用的表达式，因此只需要第一项指定为`iota`，后续可默认生成

  ```go
  const (
  	a = iota     // 0
  	b            // 延用 iota, 为1
  	c = iota * 2 // 2 * 2
  	d            // 延用 iota * 2, 为3 * 2
  	e = 1
  	f = iota // 5
  )
  ```

  实际上，`iota`并不算很好的命名，目前只有`Cpp`与`APL`以及`Go`在使用

  一种场景是：

  ```go
  const (
  	a = 1 << iota
  	b // 1 << 1
  	c // 1 << 2
  )
  ```

- **`nil`**：空值，它不是类型，而是用于初始化指针类型、切片类型、通道类型、接口类型、映射类型的零值
- 数组/切片字面量：一维数组如下，高维数组类似

  ```go
  a := [5]int32{1, 2, 3, 4, 5}
  // 可以通过 idx:value 局部赋值
  // 0-idx
  b := [5]int32{1: 2, 4: 5}
  ```

- 结构体字面量：结构体字面量有若干种写法

  ```go
  // 一个类型的字面量是紧跟该类型后的大括号
  var a AStruct = AStruct{
  	1, // 如果不带字段名, 则按顺序赋值, 且必须全部手动赋值
  	2, // 所有属性值需要紧跟 ','
  }
  // 可以指定字段名, 此时顺序无影响
  // 此时会将所有未手动赋值的属性赋为零值
  var b BStruct = BStruct{
  	a: 1,
  }
  // 匿名结构体习惯使用 := 赋值
  a := struct {
  	a int32
  	b int32
  }{
  	1,
  	2,
  }
  // 允许混合声明, 指定字段必须放在最后
  // 但这是不好的代码习惯
  b := struct {
  	a int32
  	b int32
  }{
  	1,
  	a: 2,
  }
  ```

## 声明语句

### 变量声明

- `Go`是静态强类型语言，虽然可以使用`var`与`const`，但是最终需要编译时确定类型，即指定类型或指定初始值必须含一
- 根据字面量动态推断类型：

  ```go
  a := 1
  const b = 1 // const变量只能进行一次初始化
  c := 1      // 等价于 var c = 1
  ```

  短变量赋值语句只能在函数作用域内使用

- 指定数据类型：`Go`是类型后置的，如果使用`var`声明变量不附带初始值，则必须指定类型

  ```go
  var a int

  const b int = 1
  ```

  类型后置能带来一系列优点，首先是可以抛弃`void`而不影响观感，最重要的是方法签名和**函数类型易于阅读**

- 变量分组：通过`()`对变量进行分组能提高可读性

  ```go
  var (
  	a int32
  	b float32
  )

  const (
  	a = 1
  	b
  )
  ```

- 多返回值：`Go`支持多返回值以及函数多返回值

  ```go
  a, b := 1, 2
  a, b := b, a // swap
  a, b = func() (int32, int32) {
  	return 1, 2
  }()
  ```

### 类型声明

- 类型声明(创建新类型)使用`type TypeName OriginTypeName`，基于其它类型创建新类型(两者类型的头信息不同)

  所有在之前数据类型小节讲述的类型以及自定义的类型都可以作为`OriginTypeName`

- 与`var`和`const`类似，`type`关键字也可以使用`type ()`分组声明

  同一个`type`分组的类型声明，遵循先后顺序，且在前语句声明的类型可以在后语句立刻使用，需要注意循环引用

  `Go`的引用类型不像`Java`的引用类型，其更相似的是指针类型，使用指针类型可以实现循环引用

- 示例-数组：`type IntArr5 [5]int32`
- 示例-切片：`type IntSlice []int32`
- 示例-结构体：结构体是一系列属性的集合

  ```go
  type StructName struct {
  	a int64
  	b int32
  }
  ```

- 示例-接口：接口是一系列方法的集合

  ```go
  type InterfaceName interface {
  	A() int
  	B(a int32)
  }
  ```

- 示例-通道：

  ```go
  type (
  	DEIntChan chan int32
  	RIntChan  <-chan int32
  	WIntChan  chan<- int32
  )
  ```

- 示例-函数：

  ```go
  type RetIntFunc func() int32
  ```

### 类型别名

- 类型别名使用`type TypeName = OriginTypeName`，使用`=`后，这个类型是类型别名，两者等价

  别名，意味着`OriginTypeName`必须已经声明，而不能是匿名的（如匿名结构体、匿名接口），但函数类型是允许的

- 类型别名可以和类型声明放进同一个`type`分组里

  ```go
  type (
  	RetIntFunc = func() int32 // ok
  	StrcutA    = struct {     // error
  		a int32
  	}
  	StructB struct {
  		a int32
  	}
  	StructC = StructB
  )
  ```

- 同一个`type`分组里，类型别名的位置没有任何影响，即使其引用的原类型在同一分组内的后面定义

  ```go
  type (
  	A = B

  	B int32
  )
  ```

## 运算与表达式

### 运算

- 大多数运算符和其它语言类似
- 不含前缀`++`或前缀`--`，且后缀`++`和后缀`--`都不具有返回值，即它们变成了原子操作
- 不支持三元表达式
- 条件表达式不需要小括号括住，且条件表达式必须返回布尔值，其它类型不会自动隐式地转换为布尔值

### 条件控制

- `if`表达式与其它语言类似
- `switch`表达式则有较大不同：

  ```go
  // switch 后可跟简单的计算语句
  // 特殊的是这些语句需要以 ; 结尾
  switch t := nbr; {
  // case 语句后面跟条件表达式
  case t == 1:
  	stmt
  // 和其它语言不同, 默认情况下,
  // 每个条件块执行结束结束即退出(默认break)
  // 如果想继续执行下一分支则应使用 fallthrough
  case t == 2:
  	stmt
  	fallthrough
  default:
  	default_stmt
  }
  ```

- 虽然支持`label`和`goto`语句，但是尽量不用`goto`，倒是可以结合`break`和`continue`使用标签

### 循环控制

- 没有`while`表达式，全部使用`for`：

  ```go
  for init_stmt; expr; post_stmt {
  	for_stmt
  }
  for expr {
  	while_stmt
  }
  for { // 不带表达式默认为 true
  	while_true_stmt
  }
  ```

- `for-range`：

  ```go
  for idx, val := range iterable {
  	stmt
  }
  ```

- 不支持省略`if`、`for`语句的大括号
- 不支持左大括号换行
- 大部分风格可以由格式化工具如`goimport`和`gofumpt`决定，不需要过多注意

