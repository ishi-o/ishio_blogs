---
title: "Spring framework: Validation API"
date: 2025-07-14
categories: [Programming, Java, Spring framework]
tags: [Spring, Java, validation]
---
<!-- placeholder -->
<!-- more -->
# `Spring Validation`

## `org.springframework.validation`

### 介绍

- `Spring Validation`是原生的`API`，不依赖任何外部框架或规范

### `org.springframework.validation.Validator`接口

- 这里的`Validator`不是`jakarta.validation.Validator`
- 仅包含两个方法：
  - `boolean supports(Class<?>)`：检验该验证器是否支持指定的类
  - `void validate(T obj, Errors)`：检验`obj`，将错误信息交给`Errors`引用
  对比`jakarta.validation.Validator`的`validate()`方法，它将错误信息交给返回值`Set<ConstraintViolation<T>>`

### `Errors`接口

- `Errors`接口包含一系列方法用于获取错误信息
- `reject(String errorCode)`：注册一个全局的错误，`errorCode`若要分层，风格可使用`.`符号分隔不同层

  `reject(String errorCode, String message)`：使用自定义错误信息
- `rejectValue(String field, String errorCode, String message)`：对特定字段`field`注册一个错误
- `hasErrors()`：判断是否有错误

  `hasGlobalErrors()`：判断是否有全局错误

  `hasFieldErrors(String Field)`：判断是否有特定字段错误
- `List<ObjectError> getGlobalErrors()`：获取所有全局错误

  `FieldError getFieldError(String field)`：获取指定的字段错误

  `List<FieldError> getFieldErrors()`：获取所有字段错误

### `BindingResult`接口

- `BindingResult`继承了`Errors`接口
- `Object getTarget()`：获取发生错误的目标对象

## 集成`Bean Validation API`

- `Spring`使用适配器模式将`Spring Validation`与`Bean Validation API`集成起来
  
  `LocalValidatorFactoryBean`是一个`org.springframework.validation.Validator`，内部使用一个`jakarta.validation.Validator`的实现(一般是`Hibernate`)从而支持注解形式的验证`API`，它同时也是一个`FactoryBean`，因此可以通过`IoC`容器管理
- `@Validated`是一个`Spring`额外提供的注解，比起`@Valid`，它提供分组功能，分组使用方法和`Bean Validation API`里的`groups`属性类似
- 和`Spring MVC`的联动：在方法参数中使用`@Valid`或`@Validated`或其它的注解约束，由容器会自动进行校验，并将抛出的异常赋给`BindingResult`对象自动注入

  也可定义`@ExceptionHandler`方法处理
