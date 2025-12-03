---
title: 如何用Markdown编写笔记
date: 2023-10-3
categories: [tools, notes]
tags: [language, markdown, note, beginner]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
<a id="HOME"></a>

# `Markdown`

## `MD`软件

- `Typora`：`Typora`有“**所见即所得**”的特点，即你所看见的就是文件导出后的样子，付费
- `Obsidian`：`Obsidian`的插件市场丰富，且支持笔记之间的链接功能，个人版免费
- `VS Code`+`Markdown All in One`插件：左边编辑、右边预览，支持所见即所得，该插件提供了最基本的渲染以及快捷键配置，其它插件可查看[`Markdown`插件](./1_VS%20Code.md)

## 常用记号

- `'\'`：**转义记号**，类似`C`语言里的`'\n'`等，它可以将内嵌的命令转义

- `'#'`：**标题记号**，它可以将一行文字快捷地变为一级标题

   格式为：# title，井号越多，标题等级越低

   具有该功能的快捷键是`ctrl+$d`(`$d`表示标题等级)

- `'*'`或`'_'`：**斜体记号**，两个`*`组成一队，可以将中间的文本变为*斜体*

   例如\*abcd\*的效果为：*abcd*

   具有该功能的快捷键是`ctrl+I`

- `'**'`或`'__'`：**加粗记号**，两组`**`组成一对，可以将中间的文本**加粗**，且可以和斜体记号一起使用

   例如\*\*\*abcd\*\*\*的效果为：***abcd***

   具有该功能的快捷键是`ctrl+B`

- '\`'(英文反引号)：**行内代码记号**，两个\`组成一对，可以将中间的文本变为`代码字体`，注意代码块内的记号会失效，例如使用'\\'将其转义时有时会产生混乱，**`行内代码可以加粗`**

   例如\`#include\`的效果是：`#include`

   具有该功能的快捷键是`shift+ctrl+`\`

- `'```'`：**代码块记号**，三个反引号两组组成一对，可以在其中输入代码语言与相应代码等，例如：

   ```c
   int main() {
       return 0;
   }
   ```

- `'$$'`：**块级数学公式记号**，两组`$$`组成一对，可以在其中用`LaTeX`语法写入数学公式。使用时只需键入'$$'后按下回车即可。例如：

    $$
    a^{\varphi(x)}\%\ n=1\ (a与n互素)
    $$

    具体该如何写数学公式将在[这里](6_LaTeX.md)讲述

- `'$'`：**行内数学公式记号**符，两个`$`组成一对，可以在其中用`LaTeX`语法写入数学公式。

    例如：$\begin{align}\int_0^{\Large\frac{\pi}{2}}f(\sin x)dx=\int_0^{\Large\frac{\pi}{2}}f(\cos x)dx\end{align}$

- `'[]()'`：**链接记号**，在'[]'内输入文本，在'()'内输入链接，这个链接可以是**网址**、**本地文件**或**文章内锚点**，文件支持使用相对路径或绝对路径，建议使用相对路径

    例如：[Markdown](https://markdown.com.cn/basic-syntax/)，[aFile](./0_Markdown.md)，[aFlag](#HOME)

    `'crtl+左键'`点击可跳转至该链接

    若链接中存在空格，建议使用`%20`代替

    `()`最后允许包含用`""`或`()`或`''`包围的提示，鼠标悬停时会指出预设的文字
- `'[][]'`：引用表示的链接，由第二个`[]`作为中介传递链接，例如：

    ```markdown
    [文字][1]

    [1]: https://markdown.com.cn/basic-syntax/ (悬停提示)
    ```

- `'![]()'`：**图像记号**，与链接记号类似，但只能输入图片文件的路径，且会解析图片
  
  例如：![](markdown/image_example.png (a))

  图像与链接一样，支持引用表示与悬停提示

- `'[toc]'`：**目录记号**，可以用它生成自带锚点的目录，例如开头所示的索引
- `'---'`：**分割线记号**，可以用它生成一条直线，例如：

  ---

## 内嵌`HTML`

### 表格

`Typora`支持内嵌`HTML`，常见用法为写表格，例如：

<table border="2">
    <tr>
        <td>title</td>
        <td>body</td>
    </tr>
    <tr>
        <td>exp1</td>
        <td>111</td>
    </tr>
    <tr>
        <td>exp2</td>
        <td rowspan="4" style="display:table-cell; vertical-align:middle">11</td>
    </tr>
    <tr>
        <td>exp3</td>
    </tr>
</table>

如图，使用`html`语法，写出一个四行两列的、边界值为2的、内容`'11'`被设置为占据四行且居中的表格

### 图片

通过`<img />`标签可以定义并放缩图片：

<img src="markdown/image_example.png" style="zoom:33%;" />

图片可以缩放，即在前方的`html`标签内加上`style="zoom:num%"`，`num`为缩放比例

### 链接

`<a id="111"></a>`：这条元素定义了一个`id`为`'111'`的锚点，在文章内部可以通过`[点击](#111)`链接跳转到这个锚点，例如本文在目录处设定了一个`'HOME'`锚点，您可以[点击我](#HOME)来跳转

`<a href="https://www.baidu.com">Baidu</a>`：这条元素定义了一个链接为百度域名，显示内容为`'Baidu'`的链接，<a href="https://www.baidu.com">Baidu</a>

### 强制分页

`<div style="page-break-after: always;"></div>`

在导出时，这条元素将强制新建一页

### 透明字体及右对齐

`<p style="color: rgba(400, 0, 400, 0.2)" align="right">ishiooo</p>`

`style`属性中，前三个数字为`rgb`值，第四个数字为透明度

这条元素定义了一个透明度为0.2，内容为`'ishiooo'`，右对齐的段落：

<p style="color: rgba(400, 0, 400, 0.2)" align="right">ishiooo</p>
