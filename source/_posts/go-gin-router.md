---
title: 'Gin: 路由'
categories:
  - Programming
  - Go
  - 3P Framework
  - Gin
tags:
  - Go
  - net
  - webapp
mathjax: false
date: 2025-12-27T14:58:39.000Z
---
<!-- placeholder -->
<!-- more -->

# [`Gin`](https://github.com/gin-gonic/gin)路由

## `httprouter`

### 特点

- `Gin`的路由基于`julienschmidt/httprouter`，是一个轻量级的路由组建（总共三个文件一千余行代码）
- `httprouter`只提供路由功能，标准库中的`HandleFunc()`依赖`ServeMux`来路由，而`ServeMux`针对前缀匹配采用遍历的方式匹配，效率较差
- `httprouter`提供：请求方法和路径分离，路由算法优化（基于基数树）因此一次路由极快，路径模糊匹配，路径参数捕获
- 那么`httprouter.Router`就可以作为一个前端控制器直接传递给`http.ListenAndServe(path, handler)`了

### `API`

- `httprouter.New()`：获取路由器对象
- `*httprouter.Router`提供一系列方法绑定路由（仅仅处理路由）
  - `Handle(method, path string, handle Handle)`：将方法、路径绑定到处理器，其中`handle`为`httprouter.Handle`
  - `GET/POST/PUT/DELETE/PATCH/OPTIONS/HEAD(path string, handle Handle)`：`Handle()`的快捷方法
  - `Handler()/HandlerFunc()`：`net/http`的适配器
- 路径参数捕获：路径中使用`:`可以捕获，例如`/users/:uid`，将不包含路径分隔符的路径参数捕获并赋值给`uid`
  - 使用`httprouter.Handle`：`Handle`方法的签名就是`func(http.ResponseWriter, *http.Request, Params)`，直接使用`Params`参数来访问
  - 使用`http`的处理器`API`：则适配器会将参数挂到`*http.Request.Context()`中，通过`httprouter.ParamsFromContext(Context)`获取`Params`对象
- 还可以使用`*`捕获包含路径分隔符的路径参数，因此它们只能放在最后，例如`/*userinfo`
- 当模糊匹配和精确匹配发生冲突时，按精确匹配大于`:`大于`*`的顺序进行匹配
- `Params`提供`ByName(string)`获取键对应的值

## `Gin`的参数解析

- `gin.Default(...OptionFunc) *Engine`：获取一个`*gin.Engine`，默认配置即可，可以通过选项模式修改配置
- `Gin`支持路由参数，和`httprouter`的使用方法类似
  - `*gin.Engine`对象提供和`*httprouter.Router`类似的方法：`Handle()/GET()/...`
  - 路径匹配规则类似：精确匹配、`:`匹配、`*`匹配
  - 区别在于处理器函数的类型是`HandlerFunc`：`func(*gin.Context)`

    ```go
    func(http.ResponseWriter, *http.Request, Params)

    func(*gin.Context)
    ```

    将参数替换成了上下文

  - 因此获取参数也变成了：`*gin.Context.Param(string)`

- `Gin`还支持捕获`URL`参数：
  - `*gin.Context.Query(string)`：获取`GET`参数的值，若无返回空
  - `*gin.Context.DefaultQuery(string, string)`：允许自定义默认值
- `Gin`还支持捕获请求体参数
  - `*gin.Context.PostForm(string)`：可以解析表单参数`application/x-www-form-urlencoded`与`multipart/form-data`
  - `*gin.Context.ShouldBind(any)`：自动选择解析器，并赋值给传入的参数，因此传入的参数应该是指针，不会进行错误处理
  - `*gin.Context.Bind(any)`：是对`ShouldBind()`的封装，但如果参数解析错误则会自动返回`400`错误
  - `*gin.Context.MustBindWith(any, binding.Binding)`：指定解析器，和`Bind()`一样会自动返回`400`错误
  - `BindJSON()/BindQuery()/BindYAML()/BindXML()`等：`MustBindWith()`的快捷函数
- 对于`JSON`、`XML`、`ProtoBuf`等数据，`ShouldBind()`只能调用一次，需要使用`ShouldBindBodyWith()`方法

## 分组路由

- `Gin`提供`*gin.Engine.Group(relativePath string, handlers ...HandlerFunc) *RouterGroup`方法分组
- `relativePath`的捕获语法与`httprouter`一致，相当于分组所表示的前缀

  可以表示相对路径（即开头不带`/`），则相对于`Engine`等价于相对于根路径，相对于`RouterGroup`等价于相对于路由组的路径

- `handlers`会在子路由处理器执行之前执行，它们不是`relativePath`对应的处理器，如果需要则应该在分组内绑定一个`""`路由
- `*RouterGroup`的方法和`*Engine`注册路由类似，此外它本身也可以注册新的路由组
- 为了规范，在注册路由组下的路由时，通常用`{}`括住：

  ```go
  g := e.Group("v1")
  {
   g.GET("/xxx", func(c *gin.Context) {
   })
  }
  ```

## 错误处理器

- 针对`404`路由响应（没有对应的路由），可以自定义处理器：`*gin.Engine.NoRoute(HandlerFunc)`
- 针对`405`路由响应（有对应的路由但是没有匹配的请求方法），可以使用`*gin.Engine.NoMethod(HandlerFunc)`
- `gin`本身没有十分完整的捕获错误、统一处理，不像`spring-mvc`的`@ControllerAdvice`，但是可以通过中间件实现
- 但是可以使用中间件，`gin`的中间件引入非常灵活，扩展性很高，完全可以实现类似统一错误处理器的效果

