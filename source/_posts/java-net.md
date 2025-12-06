---
title: "Java: 网络编程基础"
date: 2025-05-10
categories: [Programming, Java, Java SE]
tags: [Java, socket]
---
<!-- placeholder -->
<!-- more -->
## 网络编程基础

### `TCP`客户端

- `Java`在客户端提供的`TCP`套接字的类为`Socket`
  `Socket`构造方法：`Socket(String, int)`，和提供的服务器`IP`和端口号绑定
- `Socket`常用实例方法：
  - 如果不用构造方法，也可使用`connect()`发起`TCP`连接
  - `getInputStream()`和`getOutputStream()`：获取对应的输入流和输出流对象
    其中`OutputStream`对象应该及时`flush()`
    通常使用`InputStreamReader`和`OutputStreamWriter`将字节流转化为字符流并传递指定的字符集处理

### `TCP`服务端

- `Java`在服务端提供的`TCP`套接字的类为`ServerSocket`
  `ServerSocket`构造方法：`ServerSocket(int)`，和提供的端口号绑定
- `ServerSocket`常用实例方法：
  - `accept()`：**阻塞**式地等待请求，收到请求后返回`Socket`对象，它和客户端的`Socket`对象相关联
    建议使用`try-with-resources`语法块构造`ServerSocket`对象避免调用`close()`
  - `getPort()`和`getInetAddress()`：返回服务器绑定的本地端口号、本地`IP`地址
    `getLocalSocketAddress()`：返回服务器绑定的`IP`地址与端口号组成的`SocketAddress`对象

### `UDP`客户端与服务端

- `Java`在客户端和服务端提供的`UDP`套接字的类为`DatagramSocket`
  `DatagramSocket`的构造方法同`ServerSocket`，但是注意，由于`TCP`和`UDP`是两种不同的协议，不同协议使用的端口号是互不干扰的，它们的相同点在于使用`16`位的端口号
- `DatagramPacket`：这个类对象作为数据报的抽象，是数据传输的中介
  - 构造方法：`DatagramPacket(byte[], int, InetAddress, int)`
    和一个`byte[]`数组绑定，后两个参数可选，和待发送的`IP`与端口绑定
  - `getData()、getOffset()、getLength()`：获取数据、偏移、长度
    通常用`new String(packet.getData(), packet.getOffset(), packet.getLength(), Charset)`解析收到的数据报
  - `setData(byte[])`：设置存储的数据
- `DatagramSocket`常用实例方法：
  - `receive(DatagramPacket)`：收取一个`UDP`数据报赋值给提供的引用
    接收后，这个`DatagramPacket`对象的目标`IP`和端口会被替换为发送方的`IP`与端口
  - `send(DatagramPacket)`：发送该数据报到提供的数据报对象指向的`IP`与端口
  - `connect(InetAddress, int)`：使当前的套接字对象绑定一个默认的目标`IP`及端口，它并不像`TCP`那样会发起连接，而只是在本地进行记录
    连接后，`send()`发送的`DatagramPacket`不需要有绑定的目标`IP`，且接收时会过滤掉这个`IP`及端口以外的数据报
    为了方便，通常在客户端使用`connect()`过滤掉不需要的包
    而服务端自然不需要也不应该设置默认的目标`IP`，由`receive()`自动设置即可
  - `disconnect()`：删除本地的默认连接

### `HTTP`客户端

- 直接使用`TCP`和`UDP`套接字灵活性很强，但应用层协议需要自己制定，在安全性和性能上没有设计好的协议简易而安全，例如要开发一个网页客户端，使用`HTTP`协议是最好不过的
  但是处理`HTTP`报文需要检查和解析响应行、响应头、响应体，十分麻烦
  早期`JDK`通过`HttpURLConnection`访问`HTTP`，但是不好用，`Java 11`后提供`java.net.http`简化开发
- `HttpClient`构造方法：建议通过静态方法创建
  - `HttpClient.newHttpClient()`：使用默认配置构建
  - `HttpClient.newBuilder().build()`：使用`HttpClient.Builder`构建器构建
    这种方法是最常用的，可以在`build()`前链式调用一系列方法自定义配置，例如：

    ```java
    HttpClient client = HttpClient.newBuilder()
         .version(Version.HTTP_2)     // HTTP版本
         .followRedirects(Redirect.NORMAL)   // 重定向策略
         .connectTimeout(Duration.ofSeconds(20)) // 超时时长
         .proxy(ProxySelector.of(new InetSocketAddress("proxy.example.com", 80))) // 正向代理
         .authenticator(Authenticator.getDefault()) // HTTP身份认证
         .build(); // 返回由该Builder构造的HttpClient对象
    ```

    其中`Redirect`与`Authenticator`是`HttpClient`的内部枚举，包含一系列重定向策略与身份认证策略
- 创建`HTTP`请求：一个`HttpRequest`对象代表一个请求报文，通过静态方法`HttpRequest.newBuilder()`创建构建器，这个构建器包含若干配置方法：
  - `uri(URI)`：设置目标网络资源
  - `header(String, String)`：设置某个请求头的键值
    也可以通过`headers()`一次性设置多个请求头
  - `GET()`与`POST(BodyPublisher)`：设置请求方式，默认是`GET()`，因此前者通常会被省略
    `BodyPublishers`是工厂类，用于创建`BodyPublisher`对象，常用方法有`ofString()`等
- 处理`HTTP`响应：一个`HttpResponse`对象代表一个响应报文，通过`HttpClient`对象的`send()`发送请求报文后阻塞式地(同步式地)获取**最终**响应报文
  - 该类是泛型类，泛型类型表示响应体被解析而成的类型，由`send()`时提供的`BodyHandler`参数决定解析类型，例如：
    `HttpResponse<String> resp = client.send(req, BodyHandlers.ofString())`
  - `statusCode()`获取响应码
  - `headers()`：获取只读的响应头集合`HttpHeaders`
  - `previousResponse()`：获取前一次响应，用于回溯重定向链
  - `version()、uri()、request()`：更多获取信息的方法

### 异步网络编程

### 了解`RMI`

- `RMI`(`Remote Method Invotation`，远程方法调用)是`java.rmi`提供的远程调用`API`，仅需了解即可
  远程调用十分常用，这使得客户端代码无需有接口的实现类，而只需要声明接口即可正常调用，实现类由服务端提供
- 远程方法调用的服务接口(即服务器端)需要继承`Remote`空接口且所有方法需要抛出受检异常`RemoteException`，创建实例后，通过`UnicastRemoteObject`转化为`RMI`服务，通过`Registry`注册到本地的端口上
  而客户端需要通过`Registry`的`getRegistry(host, port)`和`lookup(serv_name)`获取服务以及服务接口

- ```java
  // 服务端: 包含RemoteDemo接口, RemoteDemoImpl实现类
  // 在本地1999号端口上启动一个RMI注册表
  Registry registry = LocalRegistry.createRegistry(1999);
  // 将普通接口转化为RMI服务接口
  RemoteDemo serv = new RemoteDemoImpl();
  RemoteDemo serv_obj = UnicastRemoteObject.exportObject(serv, 0);
  // 将该RMI服务注册到注册表中
  registry.rebind("Serv_name", serv_obj);
  
  // 客户端: 只包含RemoteDemo接口
  // 获取目标IP及端口的注册表
  Registry registry = LocalRegistry.getRegistry("localhost", 1999);
  // 获取服务, 需要强制转换
  RemoteDemo remoteDemo = (RemoteDemo) registry.lookup("Serv_name");
  ```
