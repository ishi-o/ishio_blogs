---
title: "Java: 函数式编程"
date: 2025-05-04
categories: [Programming, Java, Java SE]
tags: [Java, FP]
---
<!-- placeholder -->
<!-- more -->
## 函数式编程

### 类的初始化代码

- 在一个类中由`{}`框住的代码块称为初始化代码，在**构造方法之前**执行

  ```java
  class ClassName {
    {
      // 实例创建后构造方法执行前运行
    }
  
    static {
      // 类加载后执行一次
    }
  }
  ```

### 匿名类

- 语法：

  ```java
  new Parent() {
    // 类体
  }
  ```

- 其中`Parent`必须是存在的类或接口，这个匿名类相当于继承这个`Parent`类或`Parent`接口
- 若为实现一个接口，则类体必须实现这个接口的方法
- 运行到匿名类声明的地方时，会创建一个匿名实例，由于匿名类没有构造方法(因为没有名字)，所以使用初始化代码块`{}`来初始化

### `lambda`表达式

- 语法：

  ```java
  (参数列表) -> { 函数体 }
  // 小括号与大括号可省略
  ```

- 从`Java 8`开始，可以用`lambda`表达式作为单方法接口的一个实例，表达式的函数体作为该接口的这个方法的实现
  单方法接口指由`@FunctionalInterface`注解的含**一个抽象实例方法**的接口，可以包含`default`或`static`方法
  被该注解修饰的接口称为**函数式接口**，`java.util.function`提供了一系列基础的泛型函数式接口
- 方法引用：除了`lambda`表达式，还可以用`类名::方法名`表示一个方法的引用，这个方法引用只能作为参数传递，不能赋值给一个变量，前提是形参是一个单方法接口且方法签名与提供的方法引用的签名一致
  用`类名::new`表示该类的构造方法引用
  若传递的方法为实例方法，则其第一个参数的类型被视为该实例对应的类型
- 内置的函数式接口：
  - `Function<T, R>`：提供的方法只有一个类型为`T`的参数，且返回类型为`R`
  - `Consumer<T>`：提供的方法只有一个类型为`T`的参数，且没有返回值
  - `Predicate<T>`：相当于`Function<T, Boolean>`
  - `BiFunction, BiConsumer, BiPredicate`：相比于上述函数式接口，允许接受两个参数
  - `Supplier<T>`：提供的方法不接受任何参数，且返回类型为`T`
  - `Comparator<T>`：自定义比较器，须实现`compare()`方法
  - `Runnable`：自定义线程，须实现`run()`方法
  - `Callable, FileFilter, PathMatcher, InvocationHandler, PropertyChangeListener, ActionListener`等
  - 由于`java`的泛型无法使用基本数据类型，因此为了效率，还提供了一系列类似`TypeToTypeFunction, TypeConsumer`的针对基本数据类型的函数式接口

### 流式`API`

- 流式`API`指`java.util.stream`，和`java.io`提供的输入输出流完全不同，文件`io`用于处理输入和输出以及序列化、反序列化，而`Stream<T>`是顺序输出的一系列同类型对象
- `Stream`和`List`等序列集合的区别是，前者既可以表示有限个集合，也可以通过存储函数映射关系来实现“存储”序列对象，通过实时计算的方式“存储”无限个元素
  当然，无限个元素很多时候只需要有限个，所有`Stream`提供的功能都可以用集合框架实现，但流式`API`可以使一系列集合操作变得紧凑而清晰
- `Stream`对象创建方法：
  - `Stream.of(T...)`：提供一个有限序列
  - `Stream.generate(Supplier<T>)`：提供一个`Supplier`的实例，调用聚合函数时通过该方法实时计算元素
  - 通过`Collection`实例的`stream()`方法获取
  - 通过流对象的转换方法获取
- 常用转换方法：可以链式调用以获取新的流式对象的实例方法
  需要注意，**调用转换方法不会有任何实际计算开销**
  - `limit(long)`：流虽然能获取无限个元素，但所有情况下都只需要有限个元素，且处理时也只需要有限个元素，因此可以使用`limit()`截取前若干个元素的`Stream`对象
  - `skip(long)`：同上，流可以跳过若干个元素
  - `map(Function<T, R>)`：将流的所有元素通过该方法转换为`R`类型的元素
  - `forEach(Consumer<? super T>)`：使流中的每个元素经过该函数，要求流是有限集合
  - `filter(Predicate<? super T>)`：筛选流中的元素，将不符合条件的元素去除
  - `sorted()`和`sorted(Comparator<T>)`：排序
  - `distinct()`：去重
  - `concat()`：拼接两个流对象
  - `flatMap(Function<T, Stream<R>>)`：将所有元素经过该方法转化为流对象，再进行`concat()`使所有流对象转换为单个流对象，通常用于元素为集合类型的流的转换
  - `parallel()`：获取一个并行的流对象，不需要编写多线程代码
  - `count()`：获取流对象的元素数量
  - `max/min(Comparator<T>)`：获取最大值与最小值
  - `allMatch(Predicate<T>)`与`anyMatch(Predicate<T>)`：判断是否所有元素/存在元素满足条件
- 常用聚合方法：将一个流对象的所有元素聚合为一个统计量的实例方法，会实时计算需要的元素
  - `reduce(BinaryOperator<T>)`：提供一个二元运算函数，使用该函数对所有元素进行从左到右的累加，返回`Optional<T>`
  - `toList()`：聚合为一个`List<T>`对象
  - `toArray()`：不传递任何参数则只能返回`Object[]`数组，所以一般需要传递`T[]::new`返回`T[]`数组
  - `collect(Collector<? super T, A, R>)`：通过`Collector`方法进行聚合
  - `collect(Supplier<R>, BiConsumer<R, ? super T>, BiConsumer<R, R>)`：通过第一个参数获取一个初始的`R`类型的值，然后通过第二个参数将元素聚合到这个初始值
    第三个参数用于并行流的并行计算后，将两个`R`类型的值合并
- 通过上述方法，可以用简洁的代码对集合中的所有元素进行处理，其中`collect()`方法可以提供`Collector`，其工具类为`Collectors`，包含一系列返回`Collector`实例的静态方法，分别有三类：
  - 恒等变换：
  - 分组分区：
  - 统计意义：
- `Collector<T, A, R>`三个泛型参数的实际意义是：元素类型、中间结果类型、收集器返回类型
  源码实现为`CollectorImpl<T, A, R>`，需要提供四个参数：
  - `Supplier<A>`：用于获取一个初始的中间容器
  - `BiConsumer<A, T>`：用于聚合所有元素到中间容器中
  - `BinaryOperator<A>`：用于将两个中间容器合并为一个中间容器，用于并行流
  - `Function<A, R>`：将中间容器转化为`R`类型的结果
  - `Set<Characteristics>`：`Characteristics`枚举的集合
    该枚举包含三项：
    `CONCURRENT`：表示收集器可以并发执行合并操作而不影响结果
    `IDENTITY_FINISH`：表示中间容器有标识功能，`A`可以直接强制转换为`R`类型
    `UNORDERED`：表示收集器获取两个元素进行合并时，没有固定的获取顺序
