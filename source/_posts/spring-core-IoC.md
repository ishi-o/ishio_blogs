---
title: "Spring framework: IoC容器"
date: 2025-07-03
categories: [Programming, Java, Spring framework]
tags: [Spring, Java, IoC]
---
<!-- placeholder -->
<!-- more -->
# `IoC`容器

## 简介

- `IoC`(`Inversion of Control`，控制反转)容器的原则是使原本有相互依赖的各个部分解耦，实现松耦合地运作
  在传统的组合中，上层的模块在内部定义某些对象的类型，控制权在上层的模块中，如果需要修改内部的业务逻辑则会违反开闭原则
  而`IoC`的思想则是使上层的模块将控制权“反转”给一个第三方，例如开发者自己或者一个`IoC`容器，由外部来控制并决定向上层模块提供什么样的实现类
- `IoC`容器的常见的实现方法是`DI`(`Dependency Injection`，依赖注入)，即上层模块所依赖的其它对象仅通过**构造方法**或**工厂方法**或**`setter`**方法暴露注入入口，容器在创建对象时控制注入对象的实现类
- 当我们用到`IoC`容器时，说明一个类的实例往往是全局单例的，只要明白这个点，就能理解为什么会发生注入冲突
  但同样可以实现非单例的实例化
- 例子：

  ```java
  class Person {
      private Eye e = new BlackEye(); // 在后续改动时违反开闭原则
  }
  
  class Person {
      private Eye e;
      
      public Person(Eye e) {
          this.e = e;
      }
  }
  Person = new Person(new BlackEye()); // 在外部注入
  ```

- 而在`Spring`框架中，这个“外部注入”不需要由开发者手动地注入，而是通过声明式注解由`Spring`内置的`IoC`容器**自动注入**
- `org.springframework.beans`与`org.springframework.context`是`IoC`容器的核心
- 依赖：

  ```xml
  <dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>7.0.0-M9</version>
  </dependency>
  ```

## 元数据配置

### 上下文

- `Spring`的`IoC`容器通过解析元数据来管理`Bean`

  元数据通常包括以下内容：
  - `Bean`的类型
  - `Bean`的`id`
  - `Bean`的依赖注入
  - `Bean`的作用域
  - `Bean`的生命周期行为(以回调形式存在)
  - `Bean`的懒加载、协作者等其它元数据
- `ApplicationContext`：应用上下文接口的实例表示一个`IoC`容器，它会自动创建、配置、组装所有的`Bean`
  - 继承`BeanFactory`接口：该接口提供一系列和`Bean`有关的方法
    `T getBean(String name, Class<T> clazz)`：获取它管理的类型为`T`、名字为`name`的`Bean`
  - 继承`MessageSource`接口：该接口提供国际化处理信息的能力
    `String getMessage(String msg, Object[] args, Locale locale)`：根据`locale`以及可选的选项`args`解析`msg`
- 创建一个`ApplicationContext`实例：
  创建`ApplicationContext`实例同时意味着许多配置，在**不依靠外部框架**(如`Spring Boot`和`Tomcat`)的情况下，`ApplicationContext`只能通过`new`创建
  `Spring`提供了两种构造`ApplicationContext`实例的方式，分别为基于`XML`的配置、基于`Java`的配置
   基于`XML`的配置：实现类为`ClassPathXmlApplicationContext`，构造方法需要的参数是`String...`，即一系列**`xml`配置文件的路径**
   基于`Java`的配置：实现类为`AnnotationConfigApplicationContext`，构造方法需要的参数是`Class<?>...`，即一系列**由`@Configuration`注解的配置类**
- 基于`XML`、基于注解、基于`Java`的关系
  - 三种方式都由`Spring`控制实例的注入
  - `XML`是最早的`Spring`提供的配置方式，它是外部的、支持非侵入式地修改元数据配置
  - `Spring 2.0`开始逐步地提供一系列注解来方便分散化、简洁化配置，提供了基于注解的配置，位于`org.springframework.stereotype`和`org.springframework.beans.factory.annotations`包中
    在基于注解的配置里，开发者对实例的创建和注入的控制方式和`XML`形式类似，只是停留在**声明`Bean`**上，目的是简化配置代码书写并提供编译时的类型检查
    例如`stereotype`包中的`@Service`、`@Repository`、`@Component`，它们的修饰表示**声明这个类属于某种`Bean`**，即配置**是什么**的元数据
    例如`beans.factory.annotations`包中的`@Autowired`和`@Qualifier`，它们修饰表示指示如何创建`Bean`
    这种方式的配置需要在`XML`或`Java`中启用组件扫描来纳入管理
  - `Spring 3.0`开始，提供`org.springframework.context.annotations`来替代整个`XML`形式的配置，并提供了新的`ApplicationContext`的实现类，例如`@Configuration`与`@Bean`
    尽管注入仍由`Spring`框架管理，但这种方式允许开发者十分灵活地决定如何创建一个实例，允许使用`Java`代码进行带有分支判断地创建实例
  - 第二种方式和另外两种方式并不是互斥的，第一种方式和第三种方式对应着两种不同的实现类，而第二种方式则是作为一种方便的声明式`API`存在，适用于结构简单的`Bean`的声明
    现代应用中常使用第二种与第三种方式混合的配置方案，但并不意味着`XML`配置完全无用，它的外部、非侵入性正是它的特点

### `Bean`的`XML`配置

- `<bean/>`的`id`、`name`属性或`<alias/>`：它们都是可选项，默认由`Spring`生成一个唯一的`id`
  - `id`是唯一标识，若自定义，则必须是唯一的，不允许使用特殊字符，通常使用驼峰命名法
  - `name`是别名集合(字符串表示)，不同别名用`,`或`;`隔开，允许重复、允许使用类似`/`的特殊字符
  - 若使用引用，则查找`id`是否含有匹配项、然后查找`name`是否含有匹配项、然后查找类型是否匹配，若有二义性则报错
  - `<alias name="" alias=""/>`：在定义`bean`标签以外的地方定义它的别名
- 选择`bean`的构造器：
  - `class`属性：用于指定该`bean`所使用的构造器的来源，并不一定是`class`类型的
  - 若不提供任何构造方法，则自动根据提供的参数匹配合适的构造方法
  - 使用静态工厂方法：指定`factory-method`属性，会调用指定`class`属性这个类内部的`factory-method`静态方法
  - 使用工厂类实例的工厂方法：不能指定`class`属性，而是指定`factory-bean`属性，它是另一个类型为某个工厂类的`bean`
  - `primary`布尔属性：标识该`<bean/>`对应的构造器为默认的创建方法
- 提供构造参数(依赖注入)来消除同名构造器的二义性：
  - 基于构造器的注入：`<constructor-arg/>`：作为`<bean/>`的子标签，表示传递给构造器的参数，为了消除二义性，它有若干属性，不必全部指定而只需要保证不存在歧义即可：
    `name`属性表示参数的名称
    `index`属性表示参数的索引(`0-idx`)
    `type`属性表示参数的类型
    `value`属性表示参数的字面值
    `ref`属性表示使用其它`bean`作为参数
  - 基于`setter`的注入：要求该`bean`对应的`class`含有指定属性的`setter`方法，则可使用`<property/>`传递用`setter`设置的参数
    其属性与`<constructor-arg/>`一致
  - 一般来说，`<constructor-arg/>`会严格匹配构造器，因此会更安全(不会出现`null`)
    `<property/>`则是在构造后通过调用`setter`方法进行配置的方案，可能导致某些属性没被注意而为`null`
    前者若想达到可选参数的效果，也可使构造器参数为`Optional<T>`或使用`@Nullable`注解
- 启用组件扫描：

  ```xml
  <beans>
    <context:component-scan base-package="包名" />
  </beans>
  ```

- 例子：

  ```xml
  <!-- primary标识首选 -->
  <bean id="userBean" class="demo.beans.UserBean" primary="true">
    <!-- 根据构造参数唯一确定构造器 -->
    <constructor-arg index="0" type="int" value="10"/>
    <constructor-arg index="1" name="otherBean" ref="otherBean"/>
    <constructor-arg index="2" type="java.lang.String" value="str"/>
    <!-- 后续的setter配置 -->
    <property name="optionalArg" value="1"/>
  </bean>

  <!-- 不带任何参数表示选择默认构造方法, 但这是在primary之后的选择 -->
  <bean id="userBean" class="demo.beans.UserBean">
  </bean>

  <!-- 在外部起别名 -->
  <alias name="userBean" alias="uB"/>
  ```

### `Bean`的`Java`配置

- 声明为具有某种功能的`bean`(类级修饰)：
  - `@Component(value="")`：声明为一般的`bean`，`value`可选，表示该`bean`的名称，默认为类名的驼峰命名
  - `@Service`：声明为服务类，`@Component`的一种
  - `@Repository`：声明为持久化类，`@Component`的一种
  - `@Controller`：声明为控制器类，`@Component`的一种
  - 需要用`@ComponentScan(basePackages="")`启动扫描
- 依赖注入：
  - `@Autowired(required=true)`：修饰方法或属性，表示自动注入，修饰方法时若修饰构造器或`setter`则对应构造器注入或`setter`注入、修饰一般方法则会使`bean`在初始化完成后调用该方法，(不推荐)修饰属性时表示直接注入
    默认强制要求注入，可使`required=false`或使用`Optional<T>`或使用`@Nullable`表示可选
    如果含有多个构造方法，基于注解的配置要求至少有一个`@Autowired`
  - `@Qualifier(value="")`：修饰类或`@Bean`方法时给该`bean`起限定符，修饰属性或形参时表示指定使用`value`对应的`bean`注入该属性
  - `@Value(value="")`：注入简单字面值，**可以修饰形参**
  - `@Primary`：标识某个类时表示当某个接口或某个抽象类存在多个实现类时，选择该类作为首选的实现
    标识某个`@Bean`方法时表示当存在多个返回同一类型的方法时首选该方法
- 基于`Java`的配置：
  - `@Configuration`：标识某个类为配置类`bean`，它也是一个`@Component`
  - `@Bean`：标识某个方法为创建某个类型`bean`的方法，配合`@Primary`决定首选方法
    其`name`属性表示这个`bean`的名字，它可以同`@Qualifier`的`value`属性一样被绑定然后用`@Qualifier`指定注入
    因为一般`name`的命名有一定风格(通常是类名的驼峰命名)，而如果需要用含特殊语义的命名，`@Qualifier`的`value`属性是一种选择
- 例子：

  ```java
  package demo.config;

  @Configuration
  @ComponentScan("demo.service")
  public class BeansConfiguration {

    // 默认用方法名作为@Bean的name
    @Bean
    public User user() {
        return new User();
    }

    @Bean
    @Primary
    public User user(int type) {
        return new User(type);
    }
  }
  
  package demo.service;

  @Service
  public class UserService {
    private final UserRepository userRepo;

    public UserService() {
    }
    
    @Autowired
    public UserService(UserRepository userRepo, @Value("0") int type) {
        this.userRepo = userRepo;
    }
  } 

  package demo;

  public class Main {
    public static void main(String[] args) {
        ApplicationContext ctx = new AnnotationConfigApplicationContext(demo.config.BeansConfiguration);
    }
  }
  ```
