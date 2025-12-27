---
title: 'Go: 并发编程'
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - concurrent
mathjax: false
date: 2025-12-25T11:37:57.000Z
---
<!-- placeholder -->
<!-- more -->

# 并发

## 协程

### 特点

- `Go`的并发支持是天生的，其中协程在`Go`中是十分简洁而高效的
- 其他语言如`Rust`、`C#`、`Kotlin`、`Lua`都支持协程，协程是用户级线程，上下文开销比`OS`级线程还要小，这种轻量级的线程却能完成十分复杂的任务

### 启动协程

- 启动一个`coroutine`十分简单，只需要使用`go`关键字后跟一个函数调用即可：

  ```go
  import "fmt"

  func A() {
  	go func() {
  		fmt.Print("Hello world!")
  	}()
  }
  ```

- `go`协程允许函数调用带返回值，但不允许后跟有返回值的内置函数，如`len()/make()`等
- 就像`java`的线程需要`join()`一样，单独声明`goroutine`是无效的，因为启动协程需要时间，而主协程在这之前已经退出了，因此需要通信来协调父子协程和同级协程

### 管道

- 传统协程或线程通过共享内存来通信，而`Go`的声明是：[通过通信来共享内存](https://go.dev/blog/codelab-share)

  传统线程模型中，多个线程通过锁来防止对同一份共享内存的操作发生冲突

  `Go`的管道模型中，多个协程通过管道来通信，而管道本身不需要开发者通过锁手动管理

  这其实是一句俏皮话，在传统模型中通信是目的，共享内存与锁是实现方法，而`Go`将共享内存和加解锁的过程封装成了**管道原语**，用更容易理解的话说：使用通信(共享内存的封装)来**代替**共享内存这种手段

- `chan`类型用于在不同协程之间通信，可以通过`make()`创建：

  ```go
  make(chanType, bufsize)
  ```

  如之前所说，分为`chan/<-chan/chan<-`三种管道类型，`bufsize`是管道的缓冲大小

- 使用内置函数`close(chan<-)`关闭一个管道，也可以使用`defer`延迟

  **写协程一方**一定要记得(也应该由这一方)`close()`，否则读协程可能会阻塞地等待写协程写入数据，但写协程已经不再使用写管道或已经结束了

  管道关闭后，读协程可以继续读取管道，但如果**不再有元素，也会立刻返回**，不过返回的标志位是`false`

- 管道操作：通过`data, fin := <-chanObj`读取管道，通过`chanObj <- data`写入管道
- 管道阻塞：
  - 管道是一个阻塞队列，管道的操作是同步的，即内部封装决定了同一时刻只有一个协程能够读取或写入管道，且读和写互斥
  - 无缓冲管道：缓冲区长度为零，一个协程向管道写入后必须由另一个协程读取这个管道，写协程才能继续运行，否则会阻塞
  - 有缓冲管道：写协程在写入满管道会阻塞、读协程在读取空管道会阻塞，其它情况可以继续运行
  - `nil`管道：无论写入还是读取都会导致协程阻塞
- 管道`panic`：
  - 关闭一个`nil`管道
  - 向已关闭管道写入
  - 关闭已关闭的管道
- 管道传递的是值本身，因此针对大对象应该使用指针，实现类似共享内存的效果，针对引用类型的对象，本来就是传递指针似的

  编译器不会检查在写入后修改原变量的行为，如果写入指针或引用类型对象，则写协程不应该对原变量继续修改，否则会造成丢失修改

  一个协程最好只在写入一个变量之前修改变量，读取一个元素后所有权转移给读协程

- `len()`与`cap()`可以作用于管道查询缓冲区的元素个数和空间大小
- 单向管道通常用于函数签名和函数返回值的类型限制，双向管道最为常用，双向管道也可以隐式地转换成单向管道
- 可读管道可以使用`for range`语法糖读取

### 等待其它协程(`sync.WaitGroup`)

- `WaitGroup`用于等待一组协程
- `WaitGroup`的实现是计数器加信号量，它是一个结构体所以零值是可用的，不需要自己构造，直接声明就能使用下面的方法
- `Add(delta int)`：指定要等待的协程的个数，即初始化计数器
- `Done()`：调用该方法的协程执行完毕，即计数器减一
- `Wait()`：调用该方法的协程需要等待子协程执行完毕，即通过信号量等待计数器减为零

### 上下文(`context.Context`)

- `Context`是一个接口，有更强的协程控制能力
  - `Deadline() (deadline time.Time, ok bool)`：获取上下文应该取消的时间，若`ok`为`false`则没有`ddl`
  - `Done() <-chan struct{}`：获取一个只读通道，若上下文取消时这个只读通道就会被关闭，部分实现中不支持它的返回值是`nil`
  - `Err() error`：表示上下文取消的原因，上下文未关闭时返回`nil`
  - `Value(key any) any`：返回上下文中`key`对应的值，不存在则返回`nil`
- 上下文是不可变对象，因此自然是并发安全的
- `context`标准库提供了若干实现：`emptyCtx/cancelCtx/timerCtx/valueCtx`
  - `context.Background()`：获取一个`emptyCtx`
  - `context.TODO()`：获取另一个预先声明的`emptyCtx`
  - `emptyCtx`的方法均返回零值，通常作为一个无法被取消的最顶层的上下文，由于不同上下文实例的内存理应不同(方便调试)，`emptyCtx`的底层类型是`int`

    使用`Background()`作为默认的根上下文，从这个空上下文开始派生子上下文

    使用`TODO()`作为占位符，当需要派生子上下文，但是不知道应该放置什么类型的上下文时使用`TODO`

  - `context.WithValue(parent Context, key, value any) Context`：基于`parent`派生一个`valueCtx`，只额外实现了`Value()`方法，会向上查找键对应的值
  - `context.WithCancel(parent Context) (ctx Context, cancel CancelFunc)`：可取消的上下文，执行`cancel()`后，其下的所有上下文（因为基于自己）都会被取消

    实现很简单，在创建时向上查找可取消的上下文，如果不存在则自己创建一个协程等待取消信号，就能实现当父上下文取消时所有子上下文都递归取消

    执行`cancel()`后，所有子上下文被关闭，`<-Done()`通道发出关闭事件，可以通过`select`检测

  - `context.WithTimeout(parent Context, time.Duration) (Context, CancelFunc)`：与`cancelCtx`类似，但是额外地实现了`Deadline()`，即使不手动取消，超时后也会取消
  - `context.WithDeadline(Context, time.Time) (Context, CancelFunc)`：与上面的区别是指定的是具体的时刻而不是时间间隔
  - 可取消的上下文可以通过`context.WithXxxCause()`获取一个能传递`error`的`cancel()`函数

## `select`语句

- `select`语句用于`IO`多路复用
- `select`类似`switch`语句，包含`case`和`default`子句，但是`case`检测的是一个管道是否有返回值(不是是否关闭)：

  ```go
  select {
  case <-chA: // 可以配合 管道关闭
  	// doSth
  case _, ok := <-chB:
  	// doSth
  default: // doSth
  }
  ```

- 当任何一个`case`都不可用时，选择`default`，若没有自定义`default`则阻塞直到有一个`case`可用；若同时有多个`case`可用则随机选择一个`case`执行
- 之前说过管道在关闭后，即使没有元素也会立刻返回，因此`<-ctx.Done()`在上下文取消后，管道关闭会立刻返回，因此能被`case`选中

## 锁

### `sync.Mutex`

- 多个协程同时访问共享数据时，比起使用通道，使用锁也是一种控制方法
- `sync.Mutex`是不可重入锁，即同一个协程不能重复获得同一个锁对象，否则会`fatal`
- 实现了`sync.Locker`接口：
  - `Lock()`：加锁
  - `Unlock()`：解锁
- `TryLock()`：非阻塞式加锁，`sync.Mutex`额外实现了它

### `sync.RWMutex`

- `sync.RWMutex`是读写互斥锁，分离了读锁和写锁
- 像`sync.Mutex`那样的方法被分离成了加写锁，解写锁和非阻塞式加写锁
- 额外实现了`RLock()`，`RUnlock()`，`TryRLock()`表示对读锁的操作
- 和其它语言的读写分离一样，存在写锁时无法加读锁和写锁，存在读锁时无法加写锁但可以加读锁，实现上是内部封装了单个`sync.Mutex`

### `sync.Cond`

- `sync.Cond`条件锁，或者更准确的名称：条件变量，和`java`的`Condition`一样，建立在锁之上
- 条件变量允许一个锁拥有多个等待队列
- `sync.NewCond(l Locker) *Cond`：获取一个针对`l`的条件变量指针
- `Wait()`：短暂释放锁
- `Signal()`：唤醒一个在等待队列的进程
- `Broadcast()`：可以理解为`SignalAll`，唤醒全部在这个条件上等待的进程

<!--TODO: Add `sync` note-->

## `sync`的其它工具

### `sync.Pool`

### `sync.Once`

### `sync.Map`

## `atomic`

