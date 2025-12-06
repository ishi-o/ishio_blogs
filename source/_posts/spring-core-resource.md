---
title: "Spring framework: Resource API"
date: 2025-07-04
categories: [Programming, Java, Spring framework]
tags: [Spring, Java]
---
<!-- placeholder -->
<!-- more -->
# `Spring`资源管理

## `Resource`接口

- `Resource`类似`File`、`Path`、`URL`，是`Spring`提供的对一类资源的抽象
  `Resource`会增加应用与`Spring`框架的耦合，但实际上它只是一个对于`URL`等传统资源类的有力替代品和实用`API`，用其它资源`API`也是可以的
  `Resource`的好处是：方便，例如能够使用`classpath*`等路径前缀查找而不需要写完整的文件或`URL`的路径信息
- `Resource`接口包含的日常常用方法：
  - `boolean exists()`：判断资源是否存在
  - `boolean isReadable()`：判断资源是否可读
  - `InputStream getInputStream()`：获取该资源的输入流(继承`InputStreamSource`接口)
  - `ReadableByteChannel readableChannel()`：获取`nio`接口
  - `String getDescription()`：获取资源的全路径名，通常用于错误日志输出
- `Resource`的常见实现类：
  - `UrlResource`：是`java.net.URL`的包装
  - `ClassPathResource`：表示所有可以从`classpath`中获取的资源
  - `FileSystemResource`：是`java.io.File`以及`java.nio.file.Files`的包装
  - `ServletContextResource`：表示所有可以从`Web`应用中获取的资源
  - `ByteArrayResource`：表示一个`byte`数组资源

## `ResourceLoader`与`ResourcePatternResolver`接口

- 在平时，一般不会直接使用上述的实现类来显式地用`new`创建`Resource`实例，而是用`ResourceLoader`或`ResourcePatternResolver`帮助我们自动地选择实现类
- `ResourceLoader`包含两个方法，最常用的是`Resource getResource(String loc)`，用于加载`loc`指向的资源并自动选择合适的实现类，然后返回`Resource`实例
- `ResourcePatternResolver`继承`ResourceLoader`接口，此外还包含一个方法`Resource[] getResources(String locPattern)`，用于加载匹配`locPattern`模式串的所有资源，然后返回`Resource[]`数组
- 此前所说的`ApplicationContext`接口继承了`ResourcePatternResolver`接口，因此我们不需要额外地创建一个`ResourceLoader`或`ResourcePatternResolver`实例，而是使用`ctx.getResource(loc)`或`ctx.getResources(locPattern)`即可
- `String`转化为`Resource`格式：
  - `classpath:bean.xml`：从`classpath`(例如`target/classes`)中加载，使用`ClassPathResource`实现类
  - `file:///src/main/config.xml`：从文件系统中加载，使用`UrlResource`实现类
  - `https://www.baidu.com/`：从`URL`中加载，使用`UrlResource`实现类
  - `classpath*:WEB-INF/**/*.xml`：从`classpath`中查找，使用`Ant`风格的通配符，`**`表示零或多个目录、`*`表示零或多个字符(不能跨越目录)，会查找所有包含该路径的资源
  - 无前缀：视具体的`ApplicationContext`实现类不同，可能是`ClassPathResource`或`ServletContextResource`

## 例子

```java
public class Main {
    public static void main(String[] args) {
        ApplicationContext ctx = new ClassPathXmlApplicationContext("a.xml");
        Resource res = resourceLoader.getResource("https://www.baidu.com/");
        if (res.exists() && res.isReadable()) {
            try (InputStream is = res.getInputStream(); ReadableByteChannel bc = res.readableChannel();) {
            // use bio or nio
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

## `ResourceLoaderAware`接口(可选)

- 当一个`bean`需要在内部使用`ResourceLoader`或`ResourcePatternResolver`来方便地在类内部加载资源时，采用`@Autowired`才是更推荐的方式
- 如果需要隐式注入而非显式声明，可以使`bean`实现`ResourceLoaderAware`接口
- `ResourceLoaderAware`接口有一个方法`setResourceLoader(ResourceLoader rl)`，当这个`bean`被`ApplicationContext`实例纳入管理时，`ApplicationContext`会把自己当作参数传递给这个`set`方法
