---
title: "Java EE: JPA"
date: 2025-07-09
categories: [Programming, Java, Java EE]
tags: [ORM, Java, JPA]
---
<!-- placeholder -->
<!-- more -->
# `Jakarta Persistence API`

## 介绍

- `JPA`是一套`API`规范，从`EJB`中分离出来，是持久层的流行`API`
- 单纯引入`JPA`是没用的，其著名实现有`Hibernate`等，
- `Spring Data JPA`不是`JPA`的实现，而是基于`JPA`的一套`Repository`接口
- `JPA`定义了一套`ORM`框架，使`POJO`得以和关系联系起来
- 接口依赖：

  ```xml
  <dependency>
      <groupId>jakarta.persistence</groupId>
      <artifactId>jakarta.persistence-api</artifactId>
      <version>3.2.0</version>
  </dependency>
  ```

- `hibernate`实现依赖：

  ```xml
  <dependency>
      <groupId>org.hibernate</groupId>
      <artifactId>hibernate-core</artifactId>
      <version>6.1.4.Final</version>
  </dependency>
  ```

## 核心组件

### `Persistence`

- `Persistence`用于获取`EntityManagerFactory`
- 配置：

### `EntityManager`与`EntityManagerFactory`

- `EntityManagerFactory`全局单例且线程安全，用于获取`EntityManager`
- `EntityManager`接口管理对象的持久化操作，定义了四种实体状态：
  - `Transient`：新建状态
  - `Managed`：托管状态
  - `Detached`：分离状态
  - `Removed`：移除状态
- `EntityManager`实例包含以下方法：
  - `persist(T)`：准备保存实体，会生成实体的主键，但可能直到`flush`才保存
  - `find(Class<T>, Object pk)`：立即加载获取指定主键(第二个参数)的对象
  - `getReference(Class<T>, Object pk)`：获取一个实体代理，只有在访问时才查询
  - `merge(T)`：合并分离出去的实体，返回托管实体(传进去的实体仍是分离状态)
  - `remove(T)`：删除实体

### `EntityTransaction`

- 用于事务管理

## 基于注解的`ORM`配置

### 单表注解

- `@Entity`注解一个`POJO`类是实体类
- `@Table`注解一个实体类所对应的关系，若不使用，则`@Entity`注解类的默认表名为类名的全大写

  在该注解内定义约束等类似`SQL`中在属性外的表内定义约束
  - `name`：指定表名
  - `schema`：指定模式
  - `catalog`：指定表所属的目录
  - `uniqueConstraints`：指定表的唯一性约束
  - `indexes`：指定表的索引
  - `check`：指定表的检查约束
- `@UniqueConstraint`注解表示一个唯一性约束，在`@Table`的`uniqueConstraints`中起作用
  - `name`：约束名
  - `columns`：受约束的属性组
- `@Index`注解表示一个索引，在`@Table`的`indexes`中起作用
  - `name`：索引名
  - `columnList`：属性组，是一个字符串，不同属性由`,`连接
- `@CheckConstraint`注解表示一个检查约束，在`@Table`的`check`中起作用
  - `name`：约束名
  - `constraint`：一个本地`SQL`约束
- `@Id`注解作用于属性，标明一个主键
- `@GeneratedValue`：作用于被`@Id`注解的属性，标明如何初始化主键值，`strategy`属性有以下选择
  - `GenerationType.AUTO`(默认)策略：根据具体的数据库，自适应策略
  - `GenerationType.IDENTITY`(常用)策略：依赖数据库的`AUTO_INCREMENT`(或其他)自增
  - `GenerationType.SEQUENCE`(可能高性能)策略：依赖数据库的序列，需要配置`@GeneratedValue`的`generator`属性
  - `GenerationType.UUID`(特殊场景)策略：依赖`UUID`生成器

  定义`@GeneratedValue`后，允许使用不带主键参数的构造方法
- `@Transient`注解作用于属性，标明这个属性永远不会被持久化
- `@Column`注解作用于属性，是数据库的列映射
  - `name`：指定属性映射的列名，默认为属性名
  - `unique`：布尔值，标识是否唯一
  - `nullable`：布尔值，标识是否允许为`null`
  - `columnDefinition`：指定本地`SQL`为该列的定义，而不是使用`JPA`生成
  - `length`：指定长度
  - `check`：同`@Table`的`check`
- `@Version`：表示该对象应使用乐观锁来查询/更新，若在事务中发现冲突则会抛出`OptimisticLockException`

### 属性组主键注解

- 大部分约束，例如唯一性约束、检查约束、非空约束，由之前介绍的注解可以覆盖大部分场景(包括属性组的约束)

  但依靠上述注解无法使一个属性组作为主键
- `@Embeddable`注解标明一个类是可嵌入的

  该类的所有属性不需要`@Column`注解，且该类不需要`@Table`以及`@Entity`注解，应该作为一个普通的`POJO`，只是多了一个`@Embeddable`

  为了方便作为主键，需要**覆写`equals()`与`hashCode()`方法，且实现`Serializable`空接口**
- `@EmbeddedId`注解标明一个属性是主键，且类型是一个`@Embeddable`类，可用于属性组为主键的场景

  默认不支持`AUTO`、`IDENTITY`的策略生成嵌入主键，可以使用`SEQUENCE`策略
- `@Embedded`注解标明一个属性的类型是一个`@Embeddable`类

  `JPA`转化为`DDL`语句时，会检测所有的`Embedded`属性，将该类的每个属性就像一般属性那样写成`SQL`
- 默认情况下，`@Embeddable`类中的所有属性对应的列名就是属性名，就像没有添加`@Column`注解的一般属性那样
  
  在作为实体类的属性时，建议用`@AttributeOverride`注解清晰地决定映射：

  ```java
  @Entity
  @Table(name = "embed_demo", schema = "demo")
  public class EmbedEntity {
      @Embeddable
      public static class EmbedId implements Serializable {
          private Long uid;
          private String username;
          // 重写 equals() hashCode() ...
      }

      @EmbeddedId
      @AttributeOverrides({
          @AttributeOverride(name = "uid", column = @Column(name = "uid" /** 其它约束 */
          )),
          @AttributeOverride(name = "username", column = @Column(name = "username"))
      })
      private EmbedId id;

      @Embedded
      @AttributeOverrides({
          @AttributeOverride(name = "uid", column = @Column(name = "parent_uid" /** 其它约束 */
          )),
          @AttributeOverride(name = "username", column = @Column(name = "parent_username"))
      })
      private EmbedId pid;
  }
  ```

### 多表关联注解

- `@JoinColumns`注解包含多个`@JoinColumn`，表示一个属性有多个外键依赖
- `@JoinColumn`注解表示一个属性是外键，这样的属性其类型应是一个`@Entity`类(或是一个列表包含`@Entity`类)而不应该是引用的属性的类型，`JPA`会将这个实体类视为外表，在其中查找外键
  - `name`：该属性在本表的列名
  - `referencedColumnName`：该属性所引用的其它表的列名
  - `foreignKey`：具体的外键细节
  - `unique`：一般来说，外键是和其它表的主键相关联的，因此不需要设置`unique`属性，只需要知道遇到其它情况时记得设置`unique=true`即可
- `@ForeignKey`注解在`@JoinColumn`的`foreignKey`中起作用
  - `name`：外键约束名
  - `foreignKeyDefinition`：使用本地`DDL`定义外键
- `@JoinTable`是用于生成中间表的注解，通常配合`@ManyToMany`一起使用，它包括`@Table`的所有属性，同时多出若干属性用于拆解多对多关系
  - `joinColumns`：指向己方实体的外键列，由若干`@JoinColumn`组成`@JoinColumn`里的`name`定义中间表的列名，`referencedColumnName`指向己方表的列名
  - `inverseJoinColumns`：指向对方实体的外键列，类似`joinColumns`，只不过`referencedColumnName`指向对方表的列名
  - `foreignKey`与`inverseForeignKey`：类似`@JoinColumn`的`foreignKey`
- `@MapsId`、`@PrimaryKeyJoinColumns`、`@PrimaryKeyJoinColumn`：

  有时，一个实体的外键不仅是被引用表的主键，而且是引用表的主键(例如为了满足范式要求而拆表)，即它们共享主键

  `@MapsId`支持使用共享主键，`value`属性默认是被引用表的主键列名，使用`@MapsId`后，本实体类的`@Id`属性自动关联到被引用表的主键上，因此不需要`@GeneratedValue`注解也能使用不含主键参数的构造方法

  我们知道`@JoinColumn`等注解是会体现在`JPA`生成的`DDL`中的，但在共享主键的场景下，`@Id`属性就是外键，此时如果希望通过对象引用导航到主实体的实例上，就可以使用`@PrimaryKeyJoinColumn`和`@PrimaryKeyJoinColumns`注解，它们和一般的`@JoinColumn`注解的唯一区别是：`DDL`不会带有多余的外键列
- 数据库的映射基数有四种：一对一、一对多、多对一、多对多，其中只有多对多需要使用中间表，其它在由`ER`图转化为关系模式时都可变作“多”那方的外键引用“一”那方的主键
  
  在`JPA`中，有点反直觉的是，外键是整个被引用实体的引用而不是这个实体的主键类型

  特别是在双向关联的场景下，如果仅靠`@JoinColumn`或`@JoinTable`注解，`JPA`将只知道有关联，但不知道返回值的数量，无法使`JPA`知道对方是多方还是一方，己方是多方还是一方，那么对象导航(绑定)就无法进展下去(无法知道返回的是`List<T>`还是单纯作为一个实体类来解析)
  
  综上，联系类型的注解是很有必要的，且多对多`@ManyToMany`需要配合`@JoinTable`，而`@ManyToOne`、`@OneToOne`需要配合`@JoinColumn`，`@OneToMany`通常不和任何列映射注解配合使用
- `@OneToOne`注解：表示一对一注解

  两个`@OneToOne`可配对实现双向关联，其中一方为`@JoinColumn+@OneToOne(optional=...)`、另一方为`@OneToOne(mappedBy=...)`
  - `cascade`：设置级联的操作
  - `fetch`：设置级联操作的时机
  - `optional`：设置参与度约束，默认为`true`，即允许为`null`

    在数据库中，设置参与度约束是通过将外键设置为`NOT NULL`限制的
  - `orphanRemoval`：设置关系断开的级联操作，即在主控方断开关联后后是否级联地删除关系另一方的相应记录
  - `mappedBy`：通常用于双向关联的场景，主控方若想要维护一个反向的对象导航，不需要(也不应该)像外键持有方那样使用`@JoinColumn`，而是使用`mappedBy`属性

    在这种场景下，反向导航不会被数据库存储，即这个属性不是一个`@Column`或`@JoinColumn`属性，而是一个仅由`JPA`根据`mappedBy`获取的反向引用

    `mappedBy`属性的值是对方实体的属性名，而不是对方表的列名
- `@OneToOne`注解包含所有映射基数注解的所有属性，因为外键方可以是任意一方，所以有`optional`属性，又因为被引用的那方可以是任意一方，所以有`mappedBy`属性

  在一对多或多对一的映射基数上，多方一般是持有外键的那方，所以`@OneToMany`只有`mappedBy`(因为一方不持有外键)、`@ManyToOne`只有`optional`(因为多方持有外键，外键可选)

  此外，`@OneToMany`注解属性的类型应该是一个集合，例如`List`

  `@OneToMany(mappedBy=...)`和`@JoinColumn+@ManyToOne(optional=...)`配对实现双向关联
- `@ManyToMany`是一个特殊的映射基数，它只有`mappedBy`属性，因为在数据库中，多对多的映射基数需要**借助中间表**，这个中间表持有两个外键，从而将`@ManyToMany`拆成两个`@OneToMany`
  - 单向关联：如果不使用`@JoinTable`也会加上一个默认的`@JoinTable`，所以建议显式定义`@JoinTable`便于管理
  - 双向关联：像`@OneToOne`那样，需要一方使用`mappedBy`属性，避免生成两个中间表

    即`@JoinTable+@ManyToMany`和`@ManyToManay(mappedBy=...)`配对实现多对多的双向关联
- `CascadeType`：作用于映射基数注解里的`cascade`属性，`JPA`在实体中的一大优点是实体含有整个的引用而不只是其它实体的主键，因此也可以实现对实体进行持久化操作的同时对其含有的外键进行相同的操作
  - `PERSIST`：主体保存时，同时保存其关联的实体
  - `MERGE`：主体更新时，同时更新其关联的实体
  - `REMOVE`：主体删除时，同时删除其关联的实体

    `orphanRemoval`在关系断开时就会级联删除(包括将关联属性替换、设`null`)，而`REMOVE`策略只是在主体删除时才进行级联删除
  - `REFRESH`：主体刷新时，同时获取其关联的实体
  - `DETACH`：实例被分离出持久层管理时，同时将其关联的实例也分离出去
  - `ALL`：包括以上所有策略
- `FetchType`：作用于映射基数注解里的`fetch`属性，表示加载的策略
  - `LAZY`：懒加载，只在真正访问时加载

    `LAZY`是一对多、多对多注解的默认值，因为是`List`通常需要很长的时间加载，许多时候是没必要的
  - `EAGER`：急加载，查询到主实体后立刻加载该引用

    `EAGER`是一对一、多对一注解的默认值，因为仅加载一个关联实体的实例并不花多久

### 继承注解

### 例子

- 自关联的例子：

  ```java
  @Entity
  @Table("employee")
  public class Employee {
      @Id
      @GeneratedValue(strategy = GenerationType.IDENTITY)
      @Column(name = "eid")
      private Integer eid;

      @JoinColumn(name = "manager", referencedColumnName = "eid", foreignKey = @ForeignKey(name = "fk_EmpManager_EmpEid"))
      @ManyToOne(optional = true)
      private Employee manager;

      @OneToMany(mappedBy = "manager")
      private List<Employee> emps;
  }
  ```

- 共享主键例子：

  ```java
  @Entity
  @Table("user")
  public class User {
    @Id
    @Column(name = "uid")
    private Long uid;

    @Column(name = "username", nullable = false, length = 50)
    private String username;

    @OneToOne(mappedBy = "user")
    private UserDetail userDetail;
  }

  @Entity
  @Table("user_detail")
  public class UserDetail {
      @Id
      @Column(name = "uid")
      private Long uid;

      @PrimaryKeyJoinColumn(referencedColumnName = "uid")
      @MapsId
      @OneToOne(optional = false)
      private User user;

      // 其它信息
      @Column(name = "password", check = @CheckConstraint("password ~ ^[0-9a-zA-Z]{8,20}$"))
      private String password;
  }
  ```

## `JPQL`

### 参数化查询

- 为了防止`SQL`注入，参数化查询提供了类型检查，避免了`SQL`语句的拼接
- 命名参数：使用方法中形参的名字引用值，如`:paramName`表示取`paramName`这个形参的值
- 位置参数：使用`1-idx`的位置来索引形参，如`?1`表示取第一个参数的值

### 查询细节

- `JPQL`(`Java Persistence Query Language`)，是`JPA`规范提供的查询语言，它和`SQL`的语法类似，但是**操作对象是实体类**，而不是关系

  也因此，`JPQL`是编译时检查的，类型检查能防止很多查询错误
- 由于有映射基数的注解，在`FROM`子句使用`JOIN`时，可以不用`ON`关键字，除非需要复杂的查询条件
- 推荐使用面向对象的思想编写`JPQL`，即要多表查询时，使用`A JOIN A.B`而不是`A JOIN B`，用`.`运算符使用对象导航
- 对于列表类型，允许使用`[0-idx]`访问其中的实体实例
- 一对多的关系(列表类型)的懒加载会导致`N+1`查询问题，如果大部分场景需要懒加载，但在特定场景需要获取多方的所有实体时，可以使用`JOIN FETCH`，能优化为两次查询
- 多态查询：
- 在不使用`Spring Data JPA`的场景下，查询结果无法自动转化成返回类型的对象，需要使用`NEW ObjClass(...)`

### 面向对象的便捷函数

- `TYPE(obj)`：获取其类型
- `TREAT(obj AS Type)`：向下转型
- `SIZE(collection)`：获取集合的大小，是`COUNT()`加子查询的便捷版
- `obj MEMBER OF collection`：类似`IN`
- `collection IS EMPTY`：等价于`SIZE(collection) = 0`

## `Criteria API`

## `JPA`生命周期
