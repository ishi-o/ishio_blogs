---
title: "Java: JDBC编程"
date: 2025-06-02
categories: [Programming, Java, Java SE]
tags: [Java, JDBC]
---
<!-- placeholder -->
<!-- more -->
# `Java`数据库编程

## 传统`JDBC`编程

### `Driver`和`DriverManager`和`Connection`

- `JDBC`由各种数据库支持，提供各种包下的`Driver`类(例如`com.mysql.cj.jdbc.Driver`)，它们实现了`java.sql.Driver`接口
- `Driver`接口是驱动类，所有实现了这个接口的驱动类会在静态加载时将其本身注册到`DriverManager`类中(用一个写时拷贝的`CopyOnWriteArrayList`管理)
- `DriverManager`接口是驱动的管理器，用于创建一个连接
- `Conection`是用于管理一个数据库连接的

- ```java
  Class.forName("数据库驱动主类名");
  Connectoin conn = DriverManager.getConnection("连接url");
  ```

### `PreparedStatement`

- `PreparedStatement`是预编译的语句

## `JPA`

## `Spring JDBC`

## `Spring Data JPA`

## `Spring Boot`与`MyBatis`

## `Spring Data JDBC`
