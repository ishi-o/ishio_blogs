---
title: "Go: 模板"
categories: [Programming, Go, Spec & Stdlib]
tags: [Go, template, rendering]
mathjax: false
date: 2026-08-18T00:00:00.000Z
---

<!-- placeholder -->
<!-- more -->

# `Go`的模板

## 基本概念

- `Go`通过`text/template`标准库提供数据驱动的模板，用于生成文本输出，配置文件、邮件正文等都可以使用它
- 模板由普通文本和`action`组成，普通文本会原样复制到输出，`action`使用`{{`和`}}`包围：

  ```gotemplate
  Hello, {{.Name}}!
  ```

- `Execute()`执行模板时会把传入的数据绑定到`.`，`.`称为`dot`，模板执行过程中可以随着`with`、`range`等语句切换
- `text/template`默认不会对输出进行转义，模板作者必须是可信的；生成`HTML`时应该使用接口相同但带上下文自动转义的`html/template`
- 模板解析成功后可以并发执行，但多个执行过程如果共用同一个`io.Writer`，输出可能互相交错

## 执行模板

- 最基本的流程是创建模板、解析模板文本、传入数据执行：

  ```go
  package main

  import (
  	"os"
  	"text/template"
  )

  type Inventory struct {
  	Material string
  	Count    uint
  }

  func main() {
  	data := Inventory{Material: "wool", Count: 17}
  	tmpl, err := template.New("inventory").Parse(
  		"{{.Count}} items are made of {{.Material}}",
  	)
  	if err != nil {
  		panic(err)
  	}

  	if err := tmpl.Execute(os.Stdout, data); err != nil {
  		panic(err)
  	}
  }
  ```

- `template.New(name)`创建模板对象，`name`用于标识模板本身，不是模板文件名
- `Parse(text)`只负责解析模板文本，语法错误会在这里返回
- `Execute(writer, data)`执行模板并写入`writer`，执行阶段的字段访问、函数调用、方法调用错误也会通过返回值报告
- `template.Must(tmpl, err)`在`err`不为`nil`时直接`panic`，适合模板文本固定且程序启动时就应该失败的场景，不适合解析用户输入

## 数据访问与`dot`

- 初始的`.`就是传递给`Execute()`的`data`，可以访问结构体字段、`map`键和无参数方法：

  ```gotemplate
  {{.Title}}
  {{.User.Name}}
  {{.Config.timeout}}
  {{.DisplayName}}
  ```

- 结构体字段必须是公有字段；`map`的键名可以是小写字母开头的标识符
- 方法必须是无参数方法，并且返回一个值，或者返回`(value, error)`；第二种形式返回非空错误时，模板执行会立即结束
- 访问链可以混合字段、键和方法，例如`.User.Profile.Name`
- 初始的根数据也可以通过`$`访问。进入`range`或`with`后`.`可能已经改变，但`$`仍然指向本次执行的根数据：

  ```gotemplate
  {{range .Items}}
  	{{$.Title}}: {{.Name}}
  {{end}}
  ```

## `action`与管道

### 普通表达式

- `{{.}}`输出当前`dot`的默认文本表示，`{{.Name}}`输出字段值
- 可以使用布尔值、字符串、数字、`nil`、变量、字段、键、方法和函数作为参数：

  ```gotemplate
  {{"hello"}}
  {{42}}
  {{.Name}}
  ```

- 注释写在`action`内部，不会出现在输出中：

  ```gotemplate
  {{/* 这是一段模板注释 */}}
  ```

### 空白裁剪

- `action`外部的普通文本默认全部保留，包括换行和缩进；可以在分隔符中加入`-`来裁剪相邻的空白
- `{{-`会裁剪`action`前面普通文本末尾的空格、制表符、回车和换行：

  ```gotemplate
  A
    {{- "B"}}
  C
  ```

  上面的模板会输出`AB\nC`，左侧的换行和缩进都会被裁剪

- `-}}`会裁剪`action`后面普通文本开头的空白：

  ```gotemplate
  A{{"B" -}}
    C
  ```

  上面的模板会输出`ABC`，右侧的换行、制表符和缩进都会被裁剪

- 两侧同时使用可以去掉模板控制语句制造的空行：

  ```gotemplate
  A
    {{- "B" -}}
  C
  ```

  输出为`ABC`

- `-`旁边的空白是语法的一部分：`{{-3}}`表示输出负数`-3`，不是左裁剪；`{{- 3}}`才表示左裁剪后输出`3`
- 空白裁剪只作用于`action`两侧原本存在的普通文本，不会修改函数或字段本身产生的字符串内容

### 管道

- 管道使用`|`连接多个表达式，前一个表达式的结果会作为后一个函数的最后一个参数：

  ```gotemplate
  {{.Name | printf "Hello, %s!"}}
  ```

  上面的写法等价于调用`printf("Hello, %s!", .Name)`

- 管道适合将格式化逻辑串起来，例如：

  ```gotemplate
  {{.Title | printf "%q"}}
  ```

- `text/template`内置了常用函数：
  - `and`、`or`、`not`：逻辑运算
  - `eq`、`ne`、`lt`、`le`、`gt`、`ge`：比较值
  - `len`：获取数组、切片、`map`、字符串等值的长度
  - `index`：按索引或键访问数组、切片、字符串、`map`
  - `slice`：对数组、切片或字符串进行切片
  - `print`、`printf`、`println`：格式化输出
  - `call`：调用一个函数值

### 空值

- 在`if`、`with`和`range`中，以下值会被当作空值：`false`、`0`、`nil`指针或接口，以及长度为`0`的数组、切片、`map`和字符串
- 空值的判断不需要显式调用函数：

  ```gotemplate
  {{if .User}}
  	欢迎回来，{{.User.Name}}
  {{else}}
  	请先登录
  {{end}}
  ```

## 条件与循环

### `if`

- `if`只在管道结果非空时渲染内容，支持`else`以及直接连接的`else if`：

  ```gotemplate
  {{if eq .Status "success"}}
  	成功
  {{else if eq .Status "pending"}}
  	处理中
  {{else}}
  	失败
  {{end}}
  ```

- `if`不会改变`.`，条件块内仍然可以访问外层数据

### `range`

- `range`用于遍历数组、切片、`map`、通道等值：

  ```gotemplate
  {{range .Items}}
  	- {{.Name}}
  {{else}}
  	暂无数据
  {{end}}
  ```

- 循环体中的`.`会切换为当前元素，因此需要访问根数据时使用`$`
- 可以声明索引和元素变量。对数组、切片和字符串来说，第一个变量是索引；对`map`来说是键：

  ```gotemplate
  {{range $index, $item := .Items}}
  	{{$index}}: {{$item.Name}}
  {{end}}
  ```

- `map`的遍历顺序不应依赖；当键是有确定顺序的基本类型时，模板会按键的排序顺序访问

### `with`

- `with`适合在数据非空时切换`.`，可以减少重复的字段链：

  ```gotemplate
  {{with .User.Profile}}
  	姓名：{{.Name}}
  	邮箱：{{.Email}}
  {{else}}
  	没有个人资料
  {{end}}
  ```

- `with`块内的`.`是`.User.Profile`，但`$`仍然是根数据

## 模板变量

- 模板变量以`$`开头，可以保存管道结果：

  ```gotemplate
  {{$title := .Title}}
  标题：{{$title}}
  ```

- 变量可以在后续通过`=`重新赋值，但变量的作用域只到当前控制结构的`end`，或当前模板的结尾
- `range`可以同时声明变量，因此通常用来获取索引和元素；`range`的`else`分支中，这些变量不会被设置为新的循环值

## 自定义函数

- 通过`template.FuncMap`可以把`Go`函数暴露给模板：

  ```go
  package main

  import (
  	"strings"
  	"text/template"
  )

  var funcs = template.FuncMap{
  	"upper": strings.ToUpper,
  	"join":  strings.Join,
  }

  var tmpl = template.Must(
  	template.New("page").
  		Funcs(funcs).
  		Parse(`{{upper .Title}}: {{join .Tags ", "}}`),
  )
  ```

- 函数必须在`Parse()`之前通过`Funcs()`注册，因为解析阶段就需要确认函数名存在
- 函数可以返回一个值，也可以返回`(value, error)`；如果第二个返回值是非空错误，模板执行会停止并返回这个错误
- 暴露给模板的函数应该保持简单、确定且容易测试，避免在模板函数中修改状态或访问外部资源
- 模板适合展示和格式化，数据查询、权限判断和复杂计算应该在执行前完成，避免把业务逻辑堆积到模板函数中

## `Sprig`

- [`Masterminds/sprig`](https://github.com/Masterminds/sprig)是一个专门为`Go`模板提供函数的库，弥补了标准库只提供少量基础函数的不足
- `Sprig`不改变`text/template`的解析和执行模型，本质上仍然是通过`Funcs()`向模板注册一组函数
- 引入依赖：`go get github.com/Masterminds/sprig/v3`

### 注册函数

- 使用`Sprig`时，先把函数表注册到模板，再进行`Parse()`：

  ```go
  import (
  	"text/template"

  	sprig "github.com/Masterminds/sprig/v3"
  )

  var tmpl = template.Must(
  	template.New("page").
  		Funcs(sprig.TxtFuncMap()).
  		Parse(`{{.Name | upper}}: {{.Tags | join ", "}}`),
  )
  ```

- `TxtFuncMap()`适用于`text/template`，`HtmlFuncMap()`适用于`html/template`
- `Funcs()`必须在`Parse()`之前调用，因为解析阶段就需要确认模板中的函数名存在
- `Sprig`函数可以和标准库内置函数、自己的`FuncMap`混合使用；同名函数后注册的定义会覆盖先注册的定义，因此应该避免命名冲突

### 函数分类

- 字符串：`upper`、`lower`、`trim`、`trimPrefix`、`trimSuffix`、`replace`、`contains`、`quote`、`cat`
- 列表：`list`、`first`、`last`、`rest`、`append`、`prepend`、`concat`、`uniq`、`without`、`has`
- 字典：`dict`、`get`、`set`、`hasKey`、`pluck`、`merge`、`mergeOverwrite`、`dig`
- 数学与逻辑：`add`、`sub`、`mul`、`div`、`max`、`min`、`default`、`coalesce`、`ternary`
- 编码与摘要：`toJson`、`fromJson`、`b64enc`、`b64dec`、`sha256sum`
- 时间和格式化：`now`、`date`、`dateModify`、`ago`

  上面只是常用函数的分类，具体函数的参数类型仍然要以当前引入的`Sprig`版本为准；模板函数通常要求参数类型可赋值，不能像动态语言一样任意转换

### 列表与字典

- `list`可以创建模板列表，`dict`可以创建字符串键的字典，再结合`join`、`get`和`hasKey`生成配置或消息：

  ```gotemplate
  {{$ports := list 80 443}}
  ports: {{join "," $ports}}

  {{$labels := dict "app" .Name "tier" "web"}}
  {{if hasKey $labels "app"}}
  app: {{get $labels "app"}}
  {{end}}
  ```

- 管道可以把多个函数组合起来，例如：

  ```gotemplate
  image: {{.Tag | default "latest" | quote}}
  tags: {{.Tags | uniq | join ", "}}
  ```

- `default`、`coalesce`等函数沿用模板的“空值”判断，空字符串、`0`、`false`、空列表和`nil`都可能触发默认分支，不能把“未配置”和“显式配置为零值”混为一谈
- `set`、`merge`等字典函数会修改或合并字典；如果同一个字典在多个模板片段之间复用，需要留意它们共享同一份数据带来的副作用

### 可复现性与安全边界

- `Sprig`中有些函数依赖当前时间、随机数或环境变量，这些函数会降低输出的可复现性；配置和构建模板应该谨慎使用
- `Sprig`本身不负责`HTML`上下文转义；生成`HTML`时仍然应该使用`html/template`和合适的函数表，不能因为引入`Sprig`就把不可信数据标记为安全内容
- 不要把完整的函数表无条件暴露给不可信的模板作者；模板源代码本身需要可信，涉及环境、文件或随机性的函数也应按场景限制

## 命名模板

- 可以通过`define`声明可复用的命名模板，通过`template`执行它：

  ```gotemplate
  {{define "item" -}}
  - {{.}}
  {{- end}}

  {{define "list" -}}
  {{range .Items}}{{template "item" .}}
  {{end -}}
  {{- end}}
  ```

- `{{template "item" .}}`会把当前`.`传给`item`；如果省略数据参数，命名模板收到的是`nil`
- 解析后通过`ExecuteTemplate(writer, name, data)`可以执行指定名称的模板：

  ```go
  var output bytes.Buffer
  if err := tmpl.ExecuteTemplate(&output, "list", data); err != nil {
  	return err
  }
  ```

- 多个文件可以通过`ParseFiles()`、`ParseGlob()`或`ParseFS()`解析，适合把页面布局、片段和具体页面拆分到不同文件

## 从`embed.FS`加载模板

- `ParseFS()`可以直接从`io/fs`读取模板，配合`embed`可以把模板文件编译进二进制：

  ```go
  import (
  	"embed"
  	"text/template"
  )

  //go:embed templates/*.tmpl
  var templateFS embed.FS

  var tmpl = template.Must(
  	template.ParseFS(templateFS, "templates/*.tmpl"),
  )
  ```

- 模板文件中的命名模板可以通过`ExecuteTemplate()`选择执行；直接解析的文件模板通常使用文件的基础名称作为模板名
- 固定模板可以在包初始化时解析并复用，不要在每次请求到来时重复读取和解析文件

## 处理缺失键与执行错误

- 当`.`是`map`时，访问不存在的键默认不会在解析阶段报错，执行结果可能是`<no value>`
- 可以通过`Option("missingkey=error")`让缺失键直接使执行失败：

  ```go
  tmpl, err := template.New("config").
  	Option("missingkey=error").
  	Parse(`name={{.Name}}`)
  if err != nil {
  	return err
  }
  ```

- `Execute()`可能已经写入一部分结果后才遇到错误，因此需要对重要输出先写入`bytes.Buffer`，确认执行成功后再写入最终的`io.Writer`：

  ```go
  var output bytes.Buffer
  if err := tmpl.Execute(&output, data); err != nil {
  	return err
  }

  _, err := writer.Write(output.Bytes())
  return err
  ```

- 模板解析错误和执行错误都应该显式处理；只检查`Parse()`而忽略`Execute()`的返回值，会把运行时数据错误伪装成成功响应

## `html/template`

- `html/template`和`text/template`的模板语法基本一致，但会根据输出位置对数据进行上下文转义：

  ```go
  package main

  import (
  	"bytes"
  	"html/template"
  	"net/http"
  )

  var page = template.Must(template.New("page").Parse(`
  {{define "page"}}
  <!doctype html>
  <html>
  	<head><title>{{.Title}}</title></head>
  	<body><h1>{{.Title}}</h1></body>
  </html>
  {{end}}
  `))

  func handler(w http.ResponseWriter, r *http.Request, data any) {
  	var output bytes.Buffer
  	if err := page.ExecuteTemplate(&output, "page", data); err != nil {
  		http.Error(w, "render failed", http.StatusInternalServerError)
  		return
  	}

  	w.Header().Set("Content-Type", "text/html; charset=utf-8")
  	_, _ = w.Write(output.Bytes())
  }
  ```

- 同一个值放在普通文本、`HTML`属性、`URL`、`CSS`或`JavaScript`上下文中时，`html/template`会采用不同的转义策略
- `text/template`不会自动转义，不能用来直接生成包含不可信数据的`HTML`
- `template.HTML`等安全类型可以让开发者声明一段内容已经安全，但这会绕过对应的自动转义，只能用于已经验证过的可信内容
- 模板源代码本身仍然必须可信：自动转义保护的是执行数据，不是恶意模板作者

## 常见用法与项目

- 云原生工具通常复用`Go template`的语法模型，再注入自己的数据上下文和函数集合；因此同样的`{{if}}`、`{{range}}`在不同项目中可用的对象并不完全相同
- `Helm`：[`helm/helm`](https://github.com/helm/helm)在`Chart`的`templates/`目录中渲染`Kubernetes`清单，向模板提供`.Values`、`.Release`、`.Chart`等对象，并额外提供`include`、`required`、`tpl`等函数；大量基础辅助函数来自`Sprig`

  `include`、`required`、`tpl`是`Helm`自己的扩展，`nindent`、`toYaml`等则来自`Helm`或其依赖；这些函数都不能直接复制到只有标准库的`Go`程序中

- `kubectl`：`Kubernetes`的[`kubectl`](https://github.com/kubernetes/kubernetes)支持`-o go-template`和`-o go-template-file`，可以直接从资源对象中提取字段：

  ```sh
  kubectl get pods -o go-template='{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}'
  ```

- 监控告警：[`Prometheus Alertmanager`](https://github.com/prometheus/alertmanager)和[`Grafana`](https://github.com/grafana/grafana)都使用`Go template`语法生成告警标题、正文和通知消息，但`.Alerts`、`.Labels`以及查询函数等上下文由各自项目定义
- 配置生成：[`hashicorp/consul-template`](https://github.com/hashicorp/consul-template)使用模板从`Consul`、`Vault`等外部数据源生成配置文件或触发命令，适合需要随外部配置变化自动渲染的场景

这些项目的共同点是复用模板的语法模型，差异则主要来自数据上下文和函数集合；阅读云原生模板时，首先要确认它是标准库模板、`Sprig`扩展，还是`Helm`、`Grafana`等项目的进一步扩展方言
