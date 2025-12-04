---
title: "Java EE: Servlet API"
date: 2025-07-05
categories: [Programming, Java, Java EE]
tags: [Servlet, Java, webapp]
---
<!-- placeholder -->
<!-- more -->
## `javax.servlet`

### `Java EE`介绍

- 在`Java EE 8`及之前，这个企业版是由`Oracle`开发的闭源项目，包括一系列大型或中小型的商业软件包，但由于`Spring`等框架的流行，`Java EE`开始力不从心

  于是在之后，`Oracle`将整个`Java EE`移交给`Eclipse`开源基金会维护，并强制要求`Eclipse`不能继续使用`Java EE`的项目名以及`javax`的命名空间，因此在`Eclipse`接手这个项目后迫不得已将`javax`改为`jakarta`，并将大版本号提升至`5`，表示不再向下兼容

  总而言之，`javax`命名空间适用于`4.0`及之前的企业版项目，`jakarta`适用于`5.0`及之后的企业版

- `Servlet`源于`Server Applet`，即在服务端上运行的小程序

  在`SE`的网络编程中对于`Http`的处理只介绍了`HttpClient`，即客户端的编写，是因为服务端需要考虑的东西过多

  而`Servlet`项目致力于高效而安全地开发服务器

- `Servlet`本质可用于任意通信协议，但主要用于`HTTP`

  它内部封装了线程池，通过多线程的方式处理请求，因此比起此前的方案性能更高

- `Servlet`需要部署到专用的`Web`服务器上，称作`Servlet`容器，例如`Apache`的`Tomcat`项目、`Eclipse`的`Jetty`项目

  因为`Servlet`本质是一个`Web`应用，是`Servlet`容器向开发者暴露的规范接口，`Servlet`容器是更复杂的应用

### `servlet`依赖

- 接口依赖：

  ```xml
  <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>javax.servlet-api</artifactId>
      <version>4.0.1</version>
  </dependency>

  <dependency>
      <groupId>jakarta.servlet</groupId>
      <artifactId>jakarta.servlet-api</artifactId>
      <version>6.1.0</version>
  </dependency>
  ```

- `Servlet`接口的实现由`Web`容器提供，可以使用`provided`表示仅编译时需要

### `Servlet`接口与`HttpServlet`实现类

- `Servlet`有三个重要方法：`init()`、`service()`、`destroy()`，由`Servlet`容器调用进行生命周期管理

  `service()`含有两个参数`HttpServletRequest`与`HttpServletResponse`

- `HttpServlet`是最常用的实现类，有四个重要方法：`doGet()`、`doPost()`、`doPut()`、`doDelete()`

  它们也均含有两个参数：`HttpServletRequest`与`HttpServletResponse`

  重写它们来自定义处理逻辑

  一般不需要重写`service()`，由`HttpServlet`已经实现好了，他会自动解析请求方法并调用`doXxx()`

  除非需要使用`HttpServlet`处理其它通信协议

- `HttpServletRequest`接口表示一个`HTTP`请求报文，常用方法如下：

  - `setCharsetEncoding(String)`：设置解析的编码集
  - `String getParameter(String name)`：获取某个请求参数

    这里的参数已经消除了`GET`、`POST`等不同请求方法的差异，但对于`POST`方法而言，只会解析类型为`application/x-www-form-urlencoded`或`multipart/form-data`，其它类型如`application/json`需要通过输入流获取
  - `String getHeader(String name)`：获取某个请求头
  - `getRequestDispatcher(String location)`：获取一个请求分发器，指向内部的另一个`URI`，通常会继续调用`RequestDispatcher`接口的`forward(req, resp)`内部转发出去或`include(req, resp)`转发后回来
  - `getAttribute()`和`setAttribute()`：可以主动设置属性，方便请求处理链条中数据的传递

- `HttpServletResponse`接口表示一个`HTTP`响应报文，常用方法如下：

  - `setCharsetEncoding(String)`：设置解析的编码集
  - `setContentType(String type)`：设置响应体的类型
  - `PrintWriter getWriter()`：获取绑定的字符输出流
  - `ServletOutputStream getOutputStream()`：获取绑定的字节输出流
  - `setStatus(int)`：设置状态码
  - `setHeader(String name, String value)`：设置响应头
  - **`sendRedirect(String)`**：`302`临时重定向到另一个`URI`

- 一个服务器的所有`Servlet`实现类只会有一个实例，因此遇到多个请求时会用同一个实例来处理

  因此实现类的实例属性的所有访问需要保证线程安全

### `Session`与`Cookie`

- `Session`用于存储客户端的某种状态，能保证每个`Session`唯一，内部类似于一个`Map<String, Object>`
- `HttpServletRequest`对象的`getSession()`方法会(若不存在)创建并返回一个`HttpSession`对象，(若存在)直接返回`HttpSession`对象

  `getSession(false)`则不会创建，而只是返回已有的对象，若无则返回`null`
- `HttpSession`实例的常用方法为`setAttribute(String, Object)`和`Object getAttribute(Stirng)`以及`removeAttribute(String)`

  分别为设置、获取、删除其存储的对象数据
- `Session`通常是通过`Cookie`技术实现的，它会让客户端在发送请求时同时发送一个`Session`
  
  自定义`Cookie`需直接通过构造方法`new Cookie(String name, String value)`创建

  `Cookie`可设置最大时间`setMaxAge(seconds)`

  通过`HttpServletResponse`实例的`addCookie(Cookie)`方法传递给客户端

  通过`HttpServletRequest`实例的`getCookies()`获取`Cookie[]`数组

  通过`Cookie`实例的`getName()`和`getValue()`获取键值对

### `ServletContext`

- `ServletContext`是`Web`应用上下文的抽象，可以用它获取整个`Web`应用的信息，最常用的方法如下：
- 获取`Web`应用的属性：`Object getAttribute(String)`
- 设置`Web`应用的属性：`void setAttribute(String, Object)`
- 移除`Web`应用的属性：`void removeAttribute(String)`
- 记录日志：`log(String)`
- 使用上下文需注意，因为处于多线程环境，虽然`ServletContext`本身是线程安全的，但传入的属性值需要自行确保线程安全

### `ServletConfig`

- `ServletConfig`是某个`Servlet`服务的配置的抽象，可以用它获取某个`Servlet`服务独有的配置项，最常用的方法如下：
- 获取初始参数：`String getInitParameter()`，这个初始参数可以用`XML`或注解的形式设置，也可由内嵌`Tomcat`的`Wrapper`配置

### `JSP`技术

- `JSP`技术比较落后，了解即可
- 由于`Servlet`本质是内嵌`HTML`的`Java`微服务程序，大量的`HTML`文本由一行行的输出流打印比较麻烦，因此`JSP`技术出现

  `JSP`本质也是一个`Servlet`应用，由`Web`服务器在启动时自动将其编译成`Servlet`程序并在服务器上运行

  `JSP`作为静态资源可由`Web`容器自动部署，但也可以自定义部署`URL`路径
- `JSP`语法：
  - `Jsp`脚本：`<% Java代码 %>`，只执行代码，没有返回值
  - `Jsp`表达式：`<%= Expr %>`，返回表达式的值
  - `Jsp`声明：`<%! 成员变量、方法 %>`
  - `Jsp`指令：`<%@ 指令 %>`
  - `Jsp`动作：`<jsp:action />`
  - `Jsp`注释：`<%-- 注释 --%>`
  - `EL`表达式：`$ { expr }`
- `JSP`隐含对象：在`Jsp`脚本中不用声明就可使用的对象，但不能重复定义
  - `HttpServletRequest request`与`HttpServletResponse response`：因为`JSP`本身就是一个`Servlet`，自然有`request`和`response`参数
  - `HttpSession session`：等价于`request.getSession()`
  - `ServletConfig config`：`JSP`这个`Servlet`的配置
  - `JspWriter out`：不常用
  - `PageContext pageContext`：是整个页面上下文的抽象
  - `page`：等价于`this`，是这个`Jsp`翻译得到的`Servlet`程序实例，因为`this`是可省略的所以`page`不会被直接使用
  - `Exception exception`：用于异常处理
- `JSP`指令：
  - 声明页面的属性：`<%@ page import="java.util.*, java.net.*" %>`
  - 声明页面包含其它页面：`<%@ include other.jsp %>`
  - 声明`Tag`的来源以及前缀：`<%@ taglib uri="" prefix="" %>`
- `EL`表达式的隐含对象
  - `PageContext pageContext`：除它之外，以下所有**都是`Map`**
  - `param`：对应`request`的参数表
  - `paramValues`：对应`request`的参数表，只不过值是数组用于存储含多个值的参数
  - `header`：对应`request`的请求头表
  - `headerValues`：对应`request`的请求头表，只不过值是数组用于存储含多个值的请求头
  - `cookie`：对应`cookie`表
  - `initParam`：对应`ServletConfig`的初始参数表
  - `pageScope`：页面作用域，可通过它访问所有在页面内定义的标识符
  - `requestScope`：请求作用域，可通过它访问所有在`request`中定义的`attribute`
  - `sessionScope`：会话作用域，可通过它访问所有在`session`中定义的`attribute`
  - `applicationScope`：应用作用域
  - `EL`表达式可以不指定隐含对象地输出字段，此时会从小到大(`pageScope->requestScope->sessionScope->applicationScope`)地查找该字段
- `EL`表达式的关键字：`not、le、lt、ge、gt、eq、ne、div`等
- `Jsp`动作：
  - `<jsp:useBean id="a" class="com.Clazz" scope="page|request|session|application"/>`

    创建一个标识为`a`、类型为`com.Clazz`、所属作用域为所有作用域的对象
  - `<jsp:getProperty name="a" property="age"/>`

    获取标识为`a`的对象的`age`属性
  - `<jsp:setProperty name="a" property="age" value="value"/>`

    设置标识为`a`的对象的`age`属性为字面量`"value"`
  - `<jsp:setProperty name="a" property="age" param="age"/>`

    设置标识为`a`的对象的`age`属性为引用变量`age`的值

### `Filter`

- `Filter`接口作用于收到请求、初始化响应之后，`Servlet`服务处理请求之前，可用于身份校验、初始化配置等
- 实现`Filter`接口需要实现`doFilter(ServletRequest, ServletResponse, FilterChain)`方法，如果需要拦截则直接`return`，如果放行则需要调用`FilterChain`实例的`doFilter(ServletRequest, ServletResponse)`方法

### `Listener`

- `Listener`采用观察者模式，当某种实例创建、更改、销毁时，自定义的`Listener`组件会随之调用对应的方法
- `Listener`本身并不存在，只存在一系列的`XxxListener`接口
