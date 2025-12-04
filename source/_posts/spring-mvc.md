---
title: "Spring framework: MVC"
date: 2025-07-12
categories: [Programming, Java, Spring framework]
tags: [Spring, Java, MVC, webapp]
---
<!-- placeholder -->
<!-- more -->
# `Spring Web MVC`

## `MVC`架构模式与前端控制器设计模式

- `MVC`(`Model-View-Controller`，模型-视图-控制器)是网页应用中最常见的架构
  - 模型用于存储数据与负责业务逻辑
  - 视图用于显示与渲染数据
  - 控制器是模型与视图的中介，用于接收并过滤来自视图的输入、格式化并输出来自模型的数据
- 在传统的应用中，视图渲染同时也在服务器端进行，即是一个前后端不分离的应用，此时控制器处理请求的返回值是一个视图，在`Spring`中由`ViewResolver`解析器保证开发者只需要返回`String`而不是具体的`View`
- 在前后端分离的现代`Web`应用中，前端交给一个前端框架开发，也许你已经听说过，此时后端应该提供`RESTful`的`API`，控制器返回的不再是`View`，而是数据
- 前端控制器设计模式旨在将`Controller`部分再次拆分，用一个单一的处理程序(前端控制器)来处理所有的请求，包括过滤、认证与授权、处理、记录日志等
  这个**前端控制器**可以根据请求的类型不同，通过一个调度器/**分发器**，将请求分发到下属的**不同的处理程序**
  不同的处理程序会有对应的响应数据，**视图**用于呈现这些响应数据
- 传统的`Servlet`开发有如下问题：
  - `Servlet`的单个`doXxx()`处理多个不同`URL`的请求，管理映射困难，需要用分支判断
    而如果对每一个`URL`都写一个单独的`Servlet`服务，虽然能解决上述问题，但相似业务逻辑的代码就会被分散在不同的文件中
  - 需要手动地通过分支判断决定重定向以及内部转发，视图和服务紧耦合，难以扩展
  - 需要通过`getParameter()`获取参数以及手动类型转换和校验
- `Spring Web MVC`是基于`Jakarta EE Servlet API`的`Web`框架，通常简称`Spring MVC`

## 配置相关

### `DispatcherServlet`的配置

- `Spring MVC`提供了大部分实现，以使开发者只需要专心于编写控制器或处理器程序，然而稍微了解其内部构造仍是有用的
- `DispatcherServlet`是一个前端控制器，它实现了`jakarta.servlet.http.HttpServlet`
  它的作用就是识别项目里被`@Controller`等注解的处理器以及内部的方法映射，将来自不同`URL`的请求分发给它们
- 这个前端控制器同时应该作为一个`Bean`被纳入`IoC`容器管理，而作为一个`Servlet`应用，应用启动的流程是：由`Servlet`容器创建并注册所有的`Servlet`服务类，然后启动，最后创建`ApplicationContext`实例，因此无法使用自动注入
- 为了解决这个问题，`DispatcherServlet`的构造方法有两种：默认构造方法会查询`ServletConfig`的`InitParameter`，使用`contextConfigLocation`配置；另一个构造方法要求一个`WebApplicationContext`参数，从而引导`Servlet`容器去创建`IoC`容器的上下文
  这两种方法对应两种配置：
   使用`web.xml`以及`IoC`容器所需的`XML`文件注册，部署在外部`Servlet`容器中
    `Servlet`容器在创建`DispatcherServlet`时，检测到`<init-param/>`标签，转而读取对应的`IoC`容器的`XML`文件，在创建`DispatcherServlet`之前创建`ClassPathXmlWebApplication`实例

    ```xml
    <!-- web.xml -->
    <servlet>
        <servlet-name>dispatcher</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>

        <init-param>
            <!-- 参数名需要记 -->
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/applicationContext.xml</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>dispatcher</servlet-name>
        <url-pattern>/ctx_path</url-pattern>
    </servlet-mapping>

    <!-- applicationContext.xml -->
    <!-- IoC容器的XML配置 -->
    <beans ...>
    </beans>
    ```

  - 使用`WebApplicationInitializer`接口，覆写`onStartup(ServletContext)`方法来编程式地创建`DispatcherServlet`，实际上等价于用`Java`代码替代`XML`配置

    ```java
    // 基于注解
    public class MyWebApplicationInitializer implements WebApplicationInitializer {
        @Override
        public void onStartup(ServletContext servletContext) {
            WebApplicationContext ctx = new AnnotationConfigWebApplicationContext(...);
        
            DispatcherServlet dispatcher = new DispatcherServlet(ctx);
            var registration = servletContext.addServlet("dispatcher", dispatcher);
            registration.addMapping("/api/*");
        }
    }
    ```

  - 如果使用基于`Java`的注解，更建议继承`AbstractAnnotationConfigDispatcherServletInitializer`：

    ```java
    public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

        @Override
        protected Class<?>[] getRootConfigClasses() {
            return new Class<?>[] { RootConfig.class }; // 返回null可能不是最佳实践
        }

        @Override
        protected Class<?>[] getServletConfigClasses() {
            return new Class<?>[] { MyWebConfig.class };
        }

        @Override
        protected String[] getServletMappings() {
            return new String[] { "/api/**" };
        }
        
    }
    ```

- `DispatcherServlet`默认有若干内置的对象，重要的接口如下：
  - `HandlerMapping`接口：将处理器对应的类映射到某个具体的`URL`上
    不需要调用或实现它，它的默认实现是`RequestMappingHandlerMapping`，它会自动解析`@RequestMapping`、`@GetMapping`等注解映射
  - `HandlerAdapter`接口：(适配器)将处理器的不同方法统一为一个方法，使前端控制器不需要关注其细节
    不需要调用或实现它，它的默认实现是`ResquestMappingHandlerAdapter`，它替代前端控制器来执行`HandlerMapping`找到的方法
  - `HandlerExceptionResolver`接口：将不同地方的各种异常映射到统一的异常处理器类上
    不需要调用或实现它，它有若干实现类用于处理不同类别的异常源
  - `ViewResolver`接口：负责解析后端处理器返回的字符串，变为`View`对象，如`JSP`或`Thymeleaf`模板
    不需要调用或实现它，它的默认实现是`InternalResourceViewResolver`
    因为现代`Web`应用一般使用前后端分离，所以`ViewResolver`也不常用了
  - `LocaleResolver`接口：负责`i18n`
    不需要调用或实现它，它有若干实现类
  - `ThemeResolver`接口：负责解析`UI`主题，即前端的工作
    前端的工作一般不用`Java`来做，所以一般不需要使用它
  - `MultipartResolver`接口：负责解析`multipart`类型的数据
    没有默认配置，需要自己配置，有现成的实现类

### 拦截链

- 就像`Jakarta EE`的`Filter`那样，`DispatcherServlet`在找到对应的处理器后，调用之前会先调用拦截器链
- 拦截器需要实现`HandlerInterceptor`接口，包含三个方法，它们会作为回调函数被观察者调用
  - `preHandle(HttpServletRequest req, HttpServletResponse resp, Object handler)`：调用处理器之前的拦截点，通常进行日志记录、身份验证
  - `postHandle(HttpServletRequest req, HttpServletResponse resp, Object handler, ModelAndView modelAndView)`：成功调用处理器之后，视图渲染之前的拦截点，通常用于添加视图的全局配置
  - `afterCompletion(HttpServletRequest req, HttpServletResponse resp, Object handler, Exception ex))`：完成整个请求后的回调，通常用于资源关闭、性能统计等，就像`finally`块那样，即使之前的处理链有异常发生，也会调用该方法
- 注册拦截器：
  - 基于`Java`的配置：

    ```java
    // 需实现WebMvcConfigurer接口(全部都有default实现, 因此可按需实现)
    @Configuration
    @EnableWebMvc
    public class WebMvcConfig implements WebMvcConfigurer {
        private MyHandlerInterceptor interceptor;

        public WebMvcConfig(MyHandlerInterceptor interceptor) {
            this.interceptor = interceptor;
        }

        @Override
        public void addInterceptors(InterceptorRegistry registry) {
            // 注册interceptor, 作用于/api/**, 忽略/api/noIntercept
            // 执行顺序为最高(0为最高优先级)
            // addInterceptor()返回InterceptorRegistration, 然后允许链式调用配置方法
            registry.addInterceptor(interceptor)
                    .addPathPatterns("/api/**")
                    .exludePathPatterns("/api/noIntercept")
                    .order(0);
        }
    }
    ```

  - 基于`XML`配置：待补
- `CORS`配置：`Spring MVC`的默认`CORS`策略是同源策略，即默认情况下，不允许被不同域访问(包括`localhost`的不同端口)，浏览器确认的具体做法是先发送预请求获取对方的`CORS`配置，然后限定只能发送或不能发送特定的请求
- `@CrossOrigin`类/方法级注解就是用于配置`CORS`的，针对一个类，它会作用于该类的所有方法
  - `origin`属性包括它允许的来源域
  - `originPatterns`属性包括它允许被获取的`URI`
  - `methods`属性包括它允许的请求方法
  - `allowedHeaders`和`exposedHeaders`：允许/不允许请求所包含的请求头
  - `maxAge`：秒为单位的`CORS`配置有效时间
- 除了注解配置，还可以基于`Java`配置，仍然是`WebMvcConfigurer`的方法：
  
  ```java
  @Configuration
  @EnableWebMvc
  public class WebMvcConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.allowedOrigins("localhost")
                .addMappings("/api/**")
                .allowedMethods("GET", "POST")
                .allowedHeaders(HttpHeaders.LOCATION)
                .exposedHeaders(HttpHeaders.ACCEPT)
                .maxAge(3600);
    }
  }
  ```

## 模型与视图

### `Model`接口

- `Model`类似`ServletRequest`的属性在`Servlet`应用中的作用，用于管理单个请求范围内的由后端程序设置的属性
- `Model`提供`addAttribute(String, Object)`和`getAttribute(String, Object)`和`containsAttribute(String)`三个常用的属性相关方法
  它不像`ServletRequest`那样提供删除属性的方法，因为`Model`的生命周期较短，设计者认为不需要删除属性
- `Model`实例通常不需要开发者自己创建，而是由`Spring`容器自动创建并注入为参数

### `ModelAndView`类

- `ModelAndView`类通常作为一个返回视图的方法的返回值，需要开发者自己创建，是`Model`结合返回值`String`的一个替代选择，它包揽了`Model`以及返回视图的功能，算是一种风格上的不同
  即使在前后端结合的应用中，它也不常用，因为它不止返回视图，还返回其中添加的属性
- 它提供`addObject(String, Object)`、`setViewName(String)`

## 注解式声明

### `Controller`声明

- `@ResponseBody`类/方法级注解，注解类时表示该类的所有方法的返回值不应经过`ViewResolver`，而是作为响应体返回；注解方法时只表示该特定方法的返回值不经过`ViewResolver`
- `@Controller`类级注解：标识该类是传统的控制器类，通常作为页面控制器而存在，虽然返回值允许多个类型，但期望返回`String`，表示视图类型，然后`ViewResolver`会解析它为视图
- `@RestController`类级注解：标识该类是现代的前后端分离的控制器类，通常作为数据接口，即`API`控制器而存在，期望返回`ResponseEntity<T>`或其它类型，表示数据
  `@RestController`本质是`@Controller`加上`@ResponseBody`注解，后者表示该类的方法返回的是数据本身而不是一个视图，不应经过`ViewResolver`解析
- `@ControllerAdvice`类级注解：表示该类中定义的`ExceptionHandler`方法会应用于全局的`Controller`类
- `@RestControllerAdvice`类级注解：等价于`@ControllerAdvice`加上`@ResponseBody`注解

### `URL`映射

- `@RequestMapping`：通常作为类级注解，`value`表示映射到的`URL`路径，`method`表示请求方法
- 针对方法，在`@RequestMapping`的基础上有若干的扩展注解，即`@GetMapping`、`@PostMapping`、`@PutMapping`等，它们规定了`@RequestMapping`的`method`属性，会继承来自类级`@RequestMapping`的映射路径，例如：

  ```java
  @Controller
  @RequestMapping("/api")
  public class ApiController {
    // 访问/api/obj时会重定向到此
    @GetMapping("/obj")
    public String getObj() {
        return "index";
    }
  }
  ```

- `@RequestMapping`除`value`(`URL`路径)以外，还有若干属性用于缩小一个方法对应的`URL`范围
  - `consumes`属性：指定该方法能处理的请求的数据类型，例如`consumes={"application/json"}`，它会自动解析`Content-Type`请求头
  - `produces`属性：指定该方法响应的数据类型，例如`produces={"application/json"}`，它会自动解析`Accept`请求头
  - `params`属性：限定该方法接受请求所带的参数，例如`params={"a", "!a", "a=b"}`，`"a"`表示检测请求是否带`a`参数，`!a`表示检测请求是否不带`a`参数，`a=b`表示检测请求里的`a`参数是否等于`b`
  - `headers`属性：限定该方法接受请求所带的请求头，格式和`params`类似
- 除了`*`以及`**`通配符以外，`Spring`提供`?`以及`{}`捕获功能
  - `?`匹配任意单个字符
  - `{variable}`：将该部分路径信息捕获，赋给`variable`模板参数，可以通过`@PathVariable`获取
  - `{variable:regex}`：仅将能成功匹配`regex`正则表达式的`URL`交给该方法处理，并将该部分路径赋给`variable`
    因为在`java`中，使用`regex`与`PCRE`不同，例如`\`需要`\\`转义
  - 类型转换由`Spring`提供，需要目标参数的类型能通过单个`String`对象构造

### 允许自动注入的方法参数

- 虽然允许直接注入`ServletRequest`、`ServletResponse`等`Servlet API`，但通常没有必要
- `InputStream`、`Reader`、`OutputStream`、`Writer`：从`Servlet API`获取的流对象
- `Model`：获取当前请求的整个模型实例
- `Spring`核心提供的类型转换十分强大，以下注解获取的数据只要能进行转换(包括`json`数据)，就能自动注入为各种非`String`类型的参数值
  甚至可以自定义`Converter`并注册，以实现自定义转换的功能
- **`@PathVariable`**：获取对应的从`URL`中捕获的参数
- `@MatrixVariable`：获取从`URL`最后由`;`分割的名值对
  例如`/api/a;name1=value1,value2;name2=value3`，不同名值对由`;`分割、同一名值对的多个值由`,`分割
- **`@RequestParam("paramName")`**：获取对应的请求参数，若为`POST`则只能获取表单或`multipart`参数
- **`@RequestHeader("headerName")`**：获取对应的请求头的值
- **`@CookieValue("cookieName")`**：获取对应的`Cookie`值
- **`@RequestBody`**：获取对应的请求体
- `@RequestPart`：获取`multipart`请求体的一个`part`
- **`@ModelAttribute("attr")`**：获取模型中，属性名为`attr`的值
- `@SessionAttribute("attr")`：获取当前会话的属性`attr`的值
- `@RequestAttribute("attr")`：获取当前请求中存的属性`attr`的值
- 其它方法参数，若不带任意注解且不是上述的任意类型，若为简单类型则解析为`@RequestParam`，否则解析为`@ModelAttribute`

### 返回值类型

- `ResponseEntity<T>`：完整的由开发者决定的响应体
  - 调用一个指定了状态码的静态方法，创建一个`ResponseEntity.BodyBuilder`或`ResponseEntity.HeadersBuilder`创建者：
    `status(HttpStatusCode)`：指定任意状态码，创建一个响应报文
    `ok()`：等价于`status(200)`，服务器成功返回网页
    `accept()`：等价于`status(202)`，服务器接受但尚未处理
    `basRequest()`：等价于`status(400)`
    `notFound()`：创建一个仅含响应头的响应报文`Builder`
    `internalServerError()`：等价于`status(500)`
  - `HeadersBuilder`接口包含以下方法：
    `build()`：创建`ResponseEntity`
    `header(String, String)`以及`headers(HttpHeaders)`：添加响应头
  - `BodyBuilder`继承`HeaderBuilder`接口，此外包含以下方法：
    `contentType(String)`：设置响应体数据类型
    `contentLength(long)`：设置响应体大小
    `body(T)`：设置响应体，会自动由`Converter`序列化
- `void`：方法参数必须包含输出流或`ServletResponse`，此时表示`Spring`认为请求在该方法内部完成了处理，返回值为`void`
- `@ModelAttribute`：它在修饰参数时表示获取，而在修饰方法时表示方法的返回值会添加到该请求的`Model`里而不是返回
- `String`、`View`、`ModelAndView`：返回字符串表示的视图、自己创建的视图、绑定了一些属性的视图

### 异常处理

- 由`@ExceptionHandler`注解修饰的方法用于处理来自`Controller`类的异常，它不需要`@RequestMapping`
- 其接受的方法参数在大多数情况应该是某一个异常类型
- 其方法的返回值类型规则与一般的`@RequestMapping`方法一致
- `@ExceptionHandler`允许设置一系列异常类的`class`属性，表示只会接受这些类型的异常
- 在`@Controller`类、`@RestController`类中定义的异常处理器只作用于其所在类的其它方法抛出的异常
- 在`@ControllerAdvice`类、`@RestControllerAdvice`类中定义的异常处理器作用于所有的`Controller`类

## 其它工具类

- `org.springframework.http.HttpStatus`：包含一系列状态码的枚举
- `org.springframework.http.MediaType`：包含一系列数据类型常量，包括可以用`parseMediaType(String)`将其中没有提供的数据类型显式转换为`MediaType`
- `org.springframework.http.HttpHeaders`：包含一系列请求头、响应头的常量以及设置方法
