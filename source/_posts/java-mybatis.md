---
title: "Java: Mybatis"
date: 2025-07-16
categories: [Programming, Java, Common Tools]
tags: [Java, ORM, quick start, Mybatis]
---
<!-- placeholder -->
<!-- more -->
# `MyBatis`

## `SqlSessionFactory`

- `SqlSessionFactory`通常是单例模式，它的实例用于获取`SqlSession`，并与其他事务控制

## `SqlSession`接口

- `SqlSession`包含所有执行`Sql`的方法，类似`JDBC`的`Connection`，但它是后者的进一步封装，它将一些繁琐的错误处理，结果处理，参数绑定简化了
