---
title: "Java: 异常处理"
date: 2025-04-04
categories: [Programming, Java, Java SE]
tags: [exception, Java]
---
<!-- placeholder -->
<!-- more -->
## `Java`异常处理

### 异常的分类

- 所有异常分为受检异常、非受检异常、错误
- 声明某方法可能会抛出某异常：`throws`关键字
  在某方法内抛出某异常`new`后跟一个异常类对象

  ```java
  void method() throws Exception {
    throw new Exception();
  }
  ```

- 受检异常：编译器会在编译期检查的异常，如果用户的某个方法**调用了会抛出受检异常的方法**，但既没有**处理**也没有**显式地抛出**，就会有红色波浪线提醒
  所有受检异常都是`Exception`的子类且不是`RuntimeException`的子类
  受检异常一般表示程序员无法控制的异常，它们必须包含显式处理，例如由文件系统发出的、动态引入库发出的等
- 非受检异常：运行时可能抛出的异常，不需要显式地用`throws`声明
  所有非受检异常都是`RuntimeException`的子类
  非受检异常一般表示程序员能够通过提高编程技巧而避免的异常
- 错误：这种异常是脱离程序的，可能是硬件出了问题，例如内存溢出

### 异常的实质

- ```mermaid
  graph TD
  Throwable --> Error
  Throwable --> Exception
  Exception --> RuntimeException
  Exception --- Checked(("检查性异常\nChecked Exception")) --> IOException
  RuntimeException --> A(...)
  Checked --> ParseException
  Checked --> ...
  ```

- 所有异常和错误类都实现了`Throwable`接口
- 自定义异常：很简单，只需要根据需求继承`Exception`或`RuntimeException`(通常是后者)并实现构造方法即可
  通常由业务层抛出，用户接口层捕获并处理
- 子类重写的方法抛出的**受检异常范围**不能比父类方法`throws`的**受检异常**范围大

### 异常处理

- `catch-try-finally`：`finally`可选，表示`try`后(即时出现异常)必须执行的代码块，例如关闭文件等
- `try-with-resource`：指在`try()`中定义需要`close()`的资源，此时不需要使用`finally`就可自动关闭`try`的资源
