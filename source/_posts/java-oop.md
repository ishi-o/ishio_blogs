---
title: "Java: 面向对象编程"
date: 2025-03-06
categories: [Programming, Java, Java SE]
tags: [Java, OOP]
---
<!-- placeholder -->
<!-- more -->
## `Java`面向对象编程

### 数据类型

- `java`的数据类型分为两种：基础数据类型和对象类型
- `java`允许自动提升，不允许自动`down cast`，需要强制类型转换
- 基础数据类型只有八种，且全小写，分别为`byte、short、int、long、float、double、boolean、char`，它们直接存储在栈中
  - 不带后缀的整数均为`int`型字面量
  - 不带后缀的浮点数均为`double`型字面量
  - `boolean`类型字面量只有`true`和`false`
  - `char`类型字面量均为`Unicode`编码，一个字符占两个字节
  - 其它类型的字面量：`0x????、????h、0b????、?.??f`
- 对象类型是类类型，它们真正存在于堆中，由`new`关键字创建
  所有对象类型的变量**存储在栈中的值都是指针**，称为**引用**，指向在堆中的对象
- `java`得益于有`GC`，所以没有`delete`也没有析构函数，此乃一胜
- `null`为空指针
- 基本数据类型的默认值为零，且不能将`null`赋给基本数据类型

### 运算符

- 赋值运算符`=`：所有的赋值均为**传值**，是将**复制栈中的值**给另一个变量，给方法传参也是赋值
- 比较运算符：所有的比较均为比较栈中的值，所以针对引用变量时，没有意义
- `obj instanceof Type`：只用于引用类型，如果`obj`是`Type`或`Type`派生类的对象时，返回`true`
- 按位运算符：其中移位运算`>>>`为无符号右移，`>>`为有符号右移，`<<`在移动位数大于总位数时会将移动位数自动对总位数取模
- 逻辑运算符：`||`、`&&`
- 算术运算符：针对基础数据类型中的整型类型，最小单位是`int`，例如两个`byte`运算时会把`byte`提升到`int`然后返回`int`
- 成员访问运算符`.`：针对引用类型变量，对比`cpp`就是`->`

### `Object`

- `Object`是所有类的基类，所有类若没有显式地继承哪个类，就会隐式地继承`Object`
- `Object`对象提供的方法一般没有意义，重要在于提供了一套**规范的方法名**，重点在于**重写**
  - `public boolean equals(Object o)`：判等，注意参数为`Object`类型
  - `public int hashCode()`：获取哈希值
  - `public String toString()`：将对象转化为字符串，若实现该方法，则可以通过字符串的`+`语法糖自动转换
  - `protected Object clone() throws CloneNotSupportedException`：创建并返回本对象的副本(浅拷贝副本)，会抛出**受检异常**`CloneNotSupportedException`，防止你没有重写但又在某个地方调用了该方法
    重写它一般需要声明`implements Cloneable`(一个空接口)并`public`地重写
    但是`clone()`是很没用的，一般使用一些集成框架提供的深拷贝或浅拷贝的克隆方法
  - `Class<?> getClass()`：获取引用指向的**对象所属的类**，是堆中正在运行的那个对象的类，而不是引用的类
    详见`java`反射机制

### `this、super`

- `this`引用：指向现在在`JVM`中**正在运行的那个对象**，无论`this`在哪个类中，其本质是一个动态绑定的引用
  例如以下例子：

  ```java
  class A {
    public void a() {}
    void b() {
        a();
    }
  }
  class B extends A {
    public void a() {
        b();
    }
  }
  ```

  若声明`A a = new B()`，并调用`a.b()`，即执行`A`类的`b()`方法，即执行`this.a()`，即执行`B`类中重写的方法(因为`this`指向一个`B`类的实例)，所以这是一个无限循环的调用
- `super`关键字：`super`和`this`不同，其本质只是一个关键字，**静态绑定**所在类的父类的一个对象
  同样是上面的例子：

  ```java
  class BaseA {
    public void a() {}
  }
  class A extends BaseA {
    public void a() {}
    void b() {
        a();
        super.a();
    }
  }
  class B extends A {
    public void a() {
        b();
    }
  }
  ```

  同样，调用`b()`后会调用`this.a()`，执行`B`类的`a()`
  而`super.a()`静态绑定了所在类`A`的父类`BaseA`的`a()`方法，和`this`指向的对象隔了两代，所以区别还是很大的
- 但总而言之，`this`和`super`都不应该用来访问类的静态成员
- `this(参数)`和`super(参数)`还可以引用构造方法，同上`this`动态绑定当前运行的对象，`super`静态绑定所属类的父类对象

### 类的构造器

- 构造器：
  - 若没有自定义构造器，编译器提供一个没有参数的构造器
  - 若有自定义构造器，编译器不再提供没有参数的构造器
  - 所有构造器的第一条语句，如果不是通过`this`或`super`引用的构造方法，则会隐式地添加`super()`，即父类的无参构造器
  所以当父类没有无参构造器且子类没有构造器时，编译器会检查出错误
- `java`创建一个对象分为四步(初始化的顺序)：
  - `new`：分配内存
  - 默认赋值：基础类型赋为零，引用类型赋为`null`
  - 显式初始化：若在声明属性时用`=`赋值了，那么会将其放在`this`或`super`引用的构造器之后，构造方法代码之前
  - 执行用户的代码

### 注解(`Annotation`)

- 注解是一种元数据，编译器不会跳过注解，而是根据注解**进行检查**或其它操作
- 注解以`@`开头，例如重写方法要用`@Override`注解，下面介绍一下`java`标准库的注解
- 针对代码的注解：
  - `@Override`：表明方法被重写
  - `@Deprecated`：表明方法已过时，检查到使用该方法时，会警告
  - `@SuppressWarnings(value={"key"})`：标注部分内容，令编译器忽视被标注内容发出的警告
- 针对注解的注解(元注解)：
  - `@Retention`：标识该如何存储注解
  - `@Documented`：标识注解应该包含在用户文档中
  - `@Inherited`：标识注解继承于哪个注解
- 自定义注解模板：

  ```java
  @Documented // 可选, 表明该注解可以存在于Javadoc中
  @Target({ElementType.TYPE, ...})    // 指定注解的类型, ElementType是一个枚举, 用于限制该注解作用的范围
  public @interface MyAnnocation {}   // 自定义注解需要实现Annocation接口, @interface表明该注解实现了它
  ```

- 自定义注解的作用主要体现在反射机制上

### 接口

- `Java 8`后支持在接口内存在有实现的方法或类，具体规则在本节不讨论
- 接口同样支持多态，可以通过接口类型的引用访问实现了该接口的类对象实例
- 接口的作用：
  - 空接口：接口支持多重实现，空接口可以作为一个标记，然后通过泛型编程就能筛选掉那些没有这个标记的类
    例如`java.io.Serializable`，可序列化空接口
  - 接口可以将程序划分为多个模块，模块解耦实现多人分工
  - 用接口定义公共`API`更为清晰
- 标准库常用接口：
  - `Comparable<T>`：包含`public int compareTo(T o)`
  - `Runnable`：包含`void run()`，用于多线程编程
  - 集合框架的接口
