---
title: "Java EE: Tomcat"
date: 2025-07-08
categories: [Programming, Java, Java EE]
tags: [tomcat, Java, webapp]
---
<!-- placeholder -->
<!-- more -->
## `Tomcat`

### 依赖

```xml
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-core</artifactId>
    <version>11.0.10</version>
</dependency>

<!-- 若用到jsp技术 -->
<dependency>
    <groupId>org.apache.tomcat.embed</groupId>
    <artifactId>tomcat-embed-jasper</artifactId>
    <version>11.0.10</version>
</dependency>
```

### 结构介绍

- `Tomcat`是`Apache`开发的开源`Servlet`容器

- 其中，`WEB-INF`目录是受保护的，客户端无法访问这个目录下的资源，保存应用的信息

  这个目录下面还有`classes/`、`lib/`以及`web.xml`

  `classes/`存放`Java`程序的编译结果、`lib/`存放一系列第三方`jar`包

  其余应是一系列静态资源，例如`js/`、`css/`、`views/`等常见的命名

  使用`maven`管理项目的情况下，`lib/`不需要手动寻找、配置
- 传统部署：指`Tomcat`服务器包含开发者开发的`.war`包这样的`Web`应用，`Servlet`应用开发好后打包成`.war`包并复制到`Tomcat`服务器中，然后启动服务器

  传统部署中，`classes/`与`lib/`由`Tomcat`解包`.war`文件得到

- 内嵌`tomcat`：指`Tomcat API`作为`java`程序的一部分而存在，若使用`maven`的情况下调用`Tomcat`实例的`addWebapp()`方法，实例会从指定的路径中找到`WEB-INF/web.xml`文件并解析，然后生成`classes/`与`lib/`等在`target/`下

### `web.xml`映射

- 这是一种比较传统的映射方式，仅了解即可

- `web.xml`应位于`webapp/WEB-INF`目录下，其中`webapp`为自定义的网页应用的名字，`web.xml`里的所有的配置会**覆盖注解形式的映射**

- 声明为`webapp`：

  ```xml
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
        http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd" id="WebApp_ID" version="4.0">

  </web-app>
  ```

- 声明一个`Servlet`：

  ```xml
  <servlet>
      <servlet-name>Servlet_name</servlet-name>
      <servlet-class>类路径</servlet-class>
  </servlet>
  ```

- 将`Servlet`映射到某个路径上：

  ```xml
  <servlet-mapping>
      <servlet-name>Servlet_name</servlet-name>
      <url-pattern>/路径</url-pattern>
  </servlet-mapping>
  ```

- 初始化某个`Servlet`或某个`Filter`的参数(`ServletConfig`)：

  ```xml
  <init-param>
    <param-name>param_name</param-name>
    <param-value>xml文件、字符串等</param-value>
  </init-param>
  ```

- 设置`ServletContext`的配置：

  ```xml
  <context-param>
    <param-name>name</param-name>
    <param-value>value</param-value>
  </context-param>
  ```

- 设置欢迎文件：

  ```xml
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
  </welcome-file-list>
  ```

- 设置`session`的生效时间

  ```xml
  <session-config>
    <session-timeout>30</session-timeout>
  </session-config>
  ```

### 注解映射

- `@WebServlet`注解：在`Servlet 3.0`前，需要在`web.xml`中配置大量信息，在`WebServlet`注解出现后，所有的配置被分散到各个服务类上，最重要且必要的属性是`urlPatterns`，是`String`集合，表示这些`URI`均映射到这个`Servlet`类上

  在传统部署时，它们的解析是全自动的

### 内嵌`Tomcat`

- 内嵌部署`Tomcat`显得更加灵活，由开发者完全控制`Tomcat`的配置，但因此**注解映射、静态资源需要由开发者亲自解析、引入**，因此在不借用`Spring boot`框架的情况下，直接配置`web.xml`并调用`addWebapp()`或手动通过`Tomcat`对象注册服务以及配置映射是更常用的
- 可以理解为：通过`web.xml`配置的底层实现就是让`addWebapp()`解析该`xml`文件，并间接地在内部调用`addServlet()`、`addMapping()`等方法
- 由`new Tomcat()`直接构造一个`Tomcat`实例，调用`start()`启动该服务器(会抛出`LifecycleException`受检异常)，调用`getServer().await()`使该服务器不断地等待请求

- `Tomcat`实例方法：

  - `setHostname(String)`：设置服务器地址

  - `setPort(int)`：设置服务器端口

  - `setBaseDir(String)`：设置基准目录，表示`Tomcat`实例工作时，临时文件等存放的根目录，这里的临时文件不是`classes/`与`lib/`等，而是运行时生成的文件，例如`jsp`翻译后生成的文件

    推荐设置为`target/`下的某个目录

  - `getConnector()`：获取其绑定的连接器，这个方法在首次调用时会进行一系列的初始化操作，包括应用在**此前**设置的`baseDir`等参数

    这个方法应尽量晚些调用

  - **`Context addWebapp(String contextPath, String docBase)`**：添加`Web`应用并获取它的上下文实例

    `contextPath`指上下文的根所**映射的`URL`路径**，例如传入`""`则该上下文会映射到`"/"`

    `docBase`指要引入的**资源的本地路径**，提供的路径下必须含有`WEB-INF`目录及`web.xml`文件

    等价于将整个`Web`应用加载到服务器上，使用`addWebapp()`会自动生成并注册一个默认的`Servlet`服务，当客户端访问那些在用户自定义`Servlet`服务中找不到的服务时，会转向这个默认服务

    默认服务会试图在`docBase`下的静态资源中查询是否有对应的资源，若找到则返回，否则返回`404`响应

  - `Context addContext(String contextPath, String docBase)`：添加一个**空白的上下文**实例，参数意义同上

  - **`addServlet(String contextPath, String servletName, Servlet serv)`**：登记一个名为`servletName`的微服务，对应到路径为`contextPath`的上下文

    有静态方法的版本，其中`contextPath`应替换为`Context`对象

    同时需要使用`Context`实例的`addServletMappingDecoded(String pattern, String name)`将微服务映射到`URL`的某个路径上

    这两步等价于在`web.xml`中声明`Servlet`服务和映射

    该方法返回的`Wrapper`对象可链式调用`addMapping(String pattern)`，它等价于使用`Context`实例的`addServletMappingDecoded()`方法

- 上下文类`Context`抽象的是整个`Web`容器，和`ServletContext`不同，一般作用于容器启动前，而后者作用于`Web`应用运行中处理业务，`Context`常用的实例方法有：

  - `addWelcomeFile(String)`：设置当客户端访问根路径时，返回的资源文件
  
    即将`String`指向的资源文件映射到`"/"`
  
  - **`addServletMappingDecoded(String pattern, String name)`**：将微服务映射到某个`URL`上
  
  - `addFilterDef(FilterDef)`：登记一个`Filter`，由`FilterDef`封装其信息
  
    `FilterDef`常用方法：`setFilter(Filter)`、`setFilterName(String)`、`addInitParameter(String, Object)`
  
    分别为登记`Filter`对象、登记`Filter`服务名、初始化`Filter`参数
  
  - `addFilterMap(FilterMap)`：登记从`Filter`到`URL`的映射，由`FilterMap`封装映射信息
  
    `FilterMap`常用方法：`addURLPatternDecoded(String)`、`addServletName(String)`、`setFilterName(String)`
  
    分别为登记其应用的`URL`、登记其应用的`Servlet`、设置其绑定的`Filter`服务名

### 例子

- 基本配置：

  ```java
  Tomcat tomcat = new Tomcat();

  tomcat.setHostname("localhost");
  tomcat.setPort(8080);
  tomcat.setBaseDir("/target/tomcat/");
  // 
  // ...其它配置...
  // 
  tomcat.getConnector();
  tomcat.init();
  tomcat.start();
  tomcat.getServer().await();
  ```

- `Servlet`等配置：

  ```java
  String contextPath = "/api";
  // String docBase = new File("/src/main/webapp").getAbsolutePath();
  String docBase = Path.of("/src/main/webapp").toAbsolutePath().toString();
  Context context = tomcat.addContext(contextPath, docBase);

  String servletName = "ServletDemo";
  tomcat.addServlet(contextPath, servletName, new ServletDemo()).addMapping("/*");
  // context.addServletMappingDecoded("/*", servletName);

  FilterDef filterDef = new FilterDef();
  filterDef.setFilter((req, resp, chain) -> {
  // 过滤 ...
  // 链式处理
      chain.doFilter(req, resp);
  });
  filterDef.setFilterName("FilterDemo");

  FilterMap filterMap = new FilterMap();
  filterMap.addURLPatternDecoded("/*");
  filterMap.addServletName(servletName);
  filterMap.setFilterName("FilterDemo");

  context.addFilterDef(filterDef);
  context.addFilterMap(filterMap);
  ```

- 上述配置与以下配置是近乎等价的，只是以下配置会有多出一个默认的`Servlet`服务，用于检索静态资源

  ```java
  tomcat.addWebapp("/api", "src/main/webapp/");
  ```

  ```xml
  <!-- web.xml -->
  <web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
           xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
           id="WebApp_ID" version="4.0">

    <servlet>
      <servlet-name>ServletDemo</servlet-name>
      <servlet-class>demo.servlet.ServletDemo</servlet-class>
    </servlet>

    <servlet-mapping>
      <servlet-name>ServletDemo</servlet-name>
      <url-pattern>/*</url-pattern>
    </servlet-mapping>

    <filter>
      <filter-name>FilterDemo</filter-name>
      <filter-class>无法用lambda表达式声明的接口实现</filter-class>
    </filter>

    <filter-mapping>
      <filter-name>FilterDemo</filter-name>
      <!-- 针对/*这个URL路径设置Filter -->
      <url-pattern>/*</url-pattern>
      <!-- 针对ServletDemo这个服务单独设置Filter -->
      <servlet-name>ServletDemo</servlet-name>
    </filter-mapping>

  </web-app>
  ```
