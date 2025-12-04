---
title: "Java: 泛型编程"
date: 2025-03-18
categories: [Programming, Java, Java SE]
tags: [Java, generics]
---
<!-- placeholder -->
<!-- more -->
## `Java`泛型编程

### 泛型声明

- 泛型类似`cpp`的模板编程，用于重用代码
- **泛型类型必须为引用类型**，所以基础数据类型都提供了包装类
- 泛型类或接口：

  ```java
  public class ClassName <T> {}
  ```

- 泛型方法：

  ```java
  modifiers <T> returnType methodName(必须包含类型为T的参数) {}
  ```

### 泛型参数命名规范

- `E`：集合或数组元素的类型
- `T/U/S/P`：泛指所有类
- `K/V`：键值类
- `N`：`Number`类

### 通配符`?`

- 通配符`?`不是一种类型参数，也不是标记符，**不能在声明泛型参数时**使用，但可以在类型里面使用：

  ```java
  class ClassName <?> {}    // 错误
  class ClassName <T> {
    public static method() {
        ClassName<?> a;     // 正确, 相当于ClassName a;
    }
  }
  ```

- 仅使用`?`和省略尖括号没有区别，如果省略尖括号会有`RawTypes`警告
- 在较新版本中，使用`<>`可由编译器自动推断类型，例如：

  ```java
  Map<String> map = new HashMap<>();
  ```

### 泛型类或接口的继承

- 泛型类是一个整体，即时泛型参数之间有继承关系，泛型类或接口也没有继承关系：

  ```java
  class A {}
  class B extends A {}      // B是A的子类
  class ClassName <A> {}
  class ClassName <B> {}    // 但ClassName<B>不是ClassName<A>的子类
  ```

- 泛型类的继承规则：
  - 若子类不是泛型类，则父类必须确定泛型参数

    ```java
    class Father <T> {}
    class Son extends Father<String> {}   // Father<>必须指定参数
    ```

  - 若子类是泛型类，则父类的类型参数必须是子类所使用的参数，或者是具体的参数

    ```java
    class Father <T> {}
    class Son <T> extends Father<String> {}   // 具体的参数
    class Son <T, S> extends Father<T> {}     // Son提供的泛型参数
    ```

### 限定符`extends、super`

- 正因上述问题，`java`提供了`extends`和`super`表示了其上界和下界
- `extends TypeOrInterface`：虽然只提供了`extends`，但这个限定符后面可以跟类，也**可以跟接口**，表示这个类型参数必须继承或实现自`TypeOrInterface`类或接口
- `super TypeOrInterface`：表明这个类型参数必须是`TypeOrInterface`的父类或父接口
- 两者可以一起使用
