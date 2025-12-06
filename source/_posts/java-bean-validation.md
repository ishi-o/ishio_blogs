---
title: "Java EE: Bean Validation"
date: 2025-07-11
categories: [Programming, Java, Java EE]
tags: [validation, Java, security]
---
<!-- placeholder -->
<!-- more -->
# `Jakarta EE Validation API`

## 简介

- `Bean Validation`有多套标准，最早是`JSR 303`的`Bean Validation 1.0`，现今流行的是`Java EE 8`的`Bean Validation 2.0`，即`JSR 380`标准
- `Bean Validation`通过注解作用于属性字段上，在运行时进行验证

  需要说明的是，`JPA`规范的`@Id`、`@UniqueConstraint`等是在生成`SQL`时额外添加约束，这个约束是数据库层面上的，因此只有在保存、更新、删除时会检查值的有效性

  而`Bean Validation`作用于`Java`内存中，**给标识符赋值**的时候，例如`@NotNull`注解规定该标识符不能被赋予`null`，这种约束的检查是在`JVM`内部进行的，没有用到外部的软件

  添加`Bean Validation`，有助于在更早的时候发现问题，而对于没有问题的操作，`Validation`的开销并不大，对于`Validation`无法发现的问题，数据库的约束则是最后一层保障
- 接口依赖：

  ```xml
  <dependency>
      <groupId>jakarta.validation</groupId>
      <artifactId>jakarta.validation-api</artifactId>
      <version>3.0.2</version>
  </dependency>
  ```

- `hibernate`实现(包含接口依赖)：

  ```xml
  <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-validator</artifactId>
      <version>8.0.1.Final</version>
  </dependency>
  ```

- 可能使用到`el`表达式

## 验证注解

### 验证的通用属性

- `message`：验证失败时，将自动抛出`ConstraintViolationException`或`MethodArgumentNotValidException`异常，它们会包含该`message`

  `message`支持有限的`EL`表达式，默认会是`{jakarta.validation.constraints.ConstrantName.message}`这个`EL`表达式，是配置文件里设置的
- `groups`：指定`Class<?>[]`，表示这个验证注解所属的分组类(可有多组)

  在验证时，使用`validate(bean, Class...)`，只会检验`bean`中属于`Class`的验证注解的字段

  默认为`jakarta.validation.groups.Default`，`validate()`若不指定组则默认检验属于`Default`组的验证
- `payload`：指定`Class<? extends Payload>[]`，定义`payload`后，发生异常时可从异常中通过`getPayload()`获取`payload`信息

  通常用于定义一些元数据，例如便于分类地处理异常，例如按严重程度、按业务逻辑
- `@Xxx.List`子注解：指定`@Xxx`(同类型验证注解)数组，便于对同一类注解但不同的验证条件赋予不同的`message`

### 空值安全

- **只有以下注解会针对`null`，其它注解将始终视`null`为合法的**
- `@NotNull/@Null`：针对任意`Object`，不允许/必须为`null`
- `@NotBlank`：针对任意`CharSequence`，必须不为`null`且必须包含非空白字符，即不为空、不含`Unicode`空白字符(空格、制表符、换行符、回车符、换页符)
- `@NotEmpty`：针对任意`CharSequence`、`Collection`、`Map`、`Array`，必须不为`null`且不为空

### 数值检查

- `@AssertTrue/@AssertFalse`：针对`boolean`或`Boolean`，必须为`True/False`
- `@Min(value)/@Max(value)`：针对`BigDecimal`，`BigInteger`，`byte`、`short`、`int`、`long`以及它们的包装类，检查其最小值/最大值，`value`值是`long`

  `double`和`float`因为取整误差，不被支持设置`@Min`和`@Max`
- `@DecimalMin(value)/@DecimalMax(value)`：类似`@Min(value)/@Max(value)`，但是`value`是`String`，会被转化为`BigDecimal`表示

  此外，额外含有一个`inclusive`布尔属性，表示是否包含边界
- `@Negative/@NegativeOrZero/@Positive/@PositiveOrZero`：针对`BigDecimal`，`BigInteger`，`byte`、`short`、`int`、`long`、`float`、`double`以及它们的包装类，检查其是否为负/非正/正/非负数，因为只需要判零以及符号，所以支持`float`和`double`
- `@Digits(integer, fraction)`：针对`BigDecimal`，`BigInteger`，`byte`、`short`、`int`、`long`以及它们的包装类，以及任意的`CharSequence`，检查整数位数`<= integer`，小数位数`<= fraction`
- `@Size(min = 0, max = Integer.MAX_VALUE)`：针对任意`CharSequence`、`Collection`、`Map`、`Array`，其元素个数必须在`[min, max]`内，允许为`null`

### 特殊类型

- `@Pattern(regexp, flags = {})`：针对任意`CharSequence`，检查其是否符合`java.util.regex.Pattern`的检查

  `regexp`与`java.util.regex.Pattern`的规则一致，`flags`则是`@Flag[]`，表示检查模式，与`java.util.regex.Pattern`内部定义的标志位一致，以注解形式成为元数据的一部分
- `@Email(regexp = ".*", flags = {})`：提供一个默认的邮箱正则表达式规则，可以使用自定义的`regexp`覆盖原有表达式
- `@URL(protocal="", host="", port=-1, regexp=".*", flags={})`：针对任意`CharSequence`，检查其是否符合`java.net.URL`的构造方法的验证，可自定义协议名称、主机名称、端口号，以及可以覆盖`@Pattern`提供的`regexp`和`flags`来自定义匹配规则
- `@Future/@FutureOrPresent/@Past/@PastOrPresent`：针对任意`java.time`以及`java.time.chrono`的日期/时间类对象，检查是否是未来/未来或现在/过去/过去或现在的时间，`Now`这个对象基于`JVM`以及现在的时区来获取，具体到时间

  这里的`Present`的含义会因为针对的类型而变化，例如针对`Year`对象，`Present`视整一年为合法的“现在”，而不需要完全和`Now`一致
- **`@Valid`**：空注解，针对任意类型的对象，可以注解一个方法、一个方法的参数、一个类的属性，表示对返回值、参数值、属性值进行级联验证，即检查到这个对象时，会同时检查该对象的所有属性，但只会级联一层，不会深入到属性的属性(除非又有`@Valid`)，例如`@Valid List<@Valid ElementType>`，这样才会级联到`ElementType`的属性

## 验证器

- 所有注解在针对**方法参数以及方法**时，必须依赖`Spring`的`IoC`容器或其它`CDI`框架的实现，才是有用的(能自动校验)

  否则，在一般的`Hibernate`或其它实现中，必须依靠验证器，由开发者在代码中显式地校验
- `Validator`接口：
  - `validate(T obj, Class<?>... groups)`：验证`obj`的所有属性
  - `validateProperty(T obj, String pName, Class<?>...)`：验证其特定属性
  - `validateValue(T obj, String pName, Object value, Class<?>...)`：验证`value`是否符合其某个字段的约束
- `ConstraintViolation<T>`接口：所有上述验证方法的返回值是`Set<ConstraintViolation>`，表示出现冲突的约束信息
  - `getMessage()`：获取错误信息
  - `getRootBean()`：获取根对象
  - `getLeafBean()`：获取出错的对象
  - `getConstraintDescriptor()`：获取约束描述符
- `ConstraintDescriptor`：约束描述符
  - `getPayload()`：获取`payload`

## `Hibernate`实现下的扩展
