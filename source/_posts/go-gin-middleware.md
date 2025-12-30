---
title: 'Gin: 中间件'
categories:
  - Programming
  - Go
  - 3P Framework
  - Gin
tags:
  - Go
  - middleware
  - webapp
mathjax: false
date: 2025-12-28T17:24:12.000Z
---
<!-- placeholder -->
<!-- more -->

# `Gin`的中间件

## 中间件的原理

- `Gin`的中间件提供的很少，但是提供了一个灵活的接口
- 所有中间件只要封装成`HandlerFunc`，就可以注册为中间件，每个路由对应的处理器也算是一个`HandlerFunc`，只不过是局部中间件，更多时候是在最后被调用的
- `*gin.Context.Next()`：这个方法是最核心的方法

  ```go
  func (c *Context) Next() {
  	c.index++
  	for c.index < int8(len(c.handlers)) {
  		c.handlers[c.index](c)
  		c.index++
  	}
  }
  ```

  `gin`会把所有中间件统一到`ctx`对象中，封装为`handlers`(`[]HandlerFunc`)和`index`，它们组成了处理器调用链

  `index`的初值为`-1`，由路由匹配成功后直接调用`Next()`，一上来就自增使其指向第一个处理器

  一上来就自增的原因是处理器可以在内部调用`Next()`，因此需要使其主动指向下一个处理器

  因此一个处理器要么返回后执行`for`内部的自增，要么在内部调用`Next()`自增，为了安全，假设每个处理器都调用一次`Next()`，那么所有处理器的数量不能超过`math.MaxInt8 >> 1 = 63`，这个条件虽然很难达到，但也在注册函数内被`assert`检查了

- 注册全局中间件：调用`*gin.Engine.Use(...HandlerFunc)`
- 注册局部中间件：调用路由组`*gin.RouterGroup(relativeName string, ...HandlerFunc)`
- 中间件执行顺序由左到右，由先加入到后加入

  此外全局中间件一定比局部中间件更先执行，所有中间件一定比定义路由时绑定的处理器先执行

## 错误处理中间件

- `*gin.Context.Error(error) *gin.Error`：在处理过程中添加自定义错误，返回`*gin.Error`
- `*gin.Error`可以继续链式地调用`SetType(gin.ErrorType)`和`SetMeta(data any)`
- 可以在处理器中使用`ctx.Errors`获取所有的错误

- ```go
  import (
  	"reflect"

  	"github.com/gin-gonic/gin"
  )

  var (
  	errorTypes    = make(map[reflect.Type]string)
  	errorHandlers = make(map[string]gin.HandlerFunc)
  )

  func RegisterErrorType(errorCategory string, errorType ...error) {
  	for _, errType := range errorType {
  		errorTypes[reflect.TypeOf(errType)] = errorCategory
  	}
  }

  func RegisterErrorHandler(errorCategory string, handler gin.HandlerFunc) {
  	errorHandlers[errorCategory] = handler
  }

  func ErrorHandlerMiddleware() gin.HandlerFunc {
  	return func(c *gin.Context) {
  		c.Next()
  		if len(c.Errors) == 0 {
  			return
  		}
  		err := c.Errors[0].Err
  		errType := reflect.TypeOf(err)

  		var handler gin.HandlerFunc
  		if t, exists := errorTypes[errType]; exists {
  			if h, exists := errorHandlers[t]; exists {
  				handler = h
  			}
  		}

  		if handler != nil {
  			handler(c)
  		} else {
  			c.JSON(500, gin.H{"error": "Internal Error"})
  		}
  		c.Abort()
  	}
  }
  ```

