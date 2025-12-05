---
title: "VS Code: 较为轻量的IDE"
date: 2023-12-08
categories: [Tools & Utilities, editor, vscode]
tags: [beginner, vscode, plugins]
---
<!-- placeholder -->
<!-- more -->
# `VS Code`

## 基本功能介绍

- `VS Code`是微软开发的开源的代码编辑器，支持`Windows`、`MacOS`、`Linux`等操作系统
  其最显著的优点是**轻量**且**快速**，且得益于活跃的插件生态而**支持多种编程语言**
  高度灵活的自定义配置也是其吸引我的一点
- 左侧一栏为快捷的插件打开方式，包括资源管理器、项目管理器、扩展市场等
  - 资源管理器：与其他`IDE`类似，`VS Code`将一个项目当作一个工作区来管理，提供的快捷方式包括打开文件夹等
    其它更多有关文件管理的功能在顶栏的“文件”中
  - 项目管理器是图标为分叉的一项，它允许将项目绑定一个版本控制管理器，例如`Git`或`SVN`，同时也支持绑定`Github`或其它远程仓库托管网站的账号
  - 扩展市场：这是体现`VS Code`丰富生态的地方
- 顶上一个搜索栏可以最重要的作用是通过键入一个`>`，以快速搜索并调用插件的功能，之前所说的功能均可由搜索栏搜索得到
- 以上`UI`的布局均可随意拖动，以及可以在右上方调整布局
- 配置：`VS Code`将配置分为三级，默认配置、用户配置、工作区配置，默认配置存储于`defaultSettings.json`中，另外两者存储在同名文件`settings.json`中，所有均在**`首选项:打开XX配置(JSON)`**中，`VS Code`提供了`UI`界面帮助修改，但建议直接编辑`json`文件
  - 默认配置：`VS Code`本身及其插件会提供一系列的配置项，插件维护者需要为每一个配置项提供默认值，用户下载该插件后则会登记在默认配置中
    该文件是只读文件，无需在意
  - 用户配置：用户配置是**用户自定义的全局的配置**，存放在本机用户家目录下，是最常改的配置文件，在用户定义后，它会覆盖默认配置，否则使用默认配置
  - 工作区配置：存放在工作文件夹下自动生成的`./vscode/settings.json`，它会覆盖所有上级配置，未定义的配置项使用用户配置

## 基本的配置

- 编辑器的基本配置(仅包括影响较大的可设置为非默认值的配置)：
  - 区分内联建议与快速建议：快速建议指的是由字典服务提供的键入值后在光标右侧显示的提示，内联建议指由内置`AI`智能生成的代码补全
    不建议使用内联建议
  - 不在提交字符时接受建议(建议即快速提示)：

    ```json
    "editor.acceptSuggestionOnCommitCharacter": false
    ```

  - 不在按下`Enter`时接受建议：

    ```json
    "editor.acceptSuggestionOnEnter": "off"
    ```

  - 设置在保存代码时进行的动作：

    ```json
    "editor.codeActionsOnSave": {
        // 在显示保存时自动import所需包
        "source.organizeImports": "explicit",
        // 有其它source动作
    }
    ```

  - 设置打开文件时不检测`tabSize`与`insertSpaces`，即编辑器无法改变预设`tab`的空格数以及预设的是否用空格代替`tab`

    ```json
    "editor.detectIndentation": false
    ```

  - 设置保存文件时自动格式化(格式化程序需可用，通常分别设置格式化程序而在全局设置一次以下属性)：

    ```json
    "editor.formatOnSave": true
    ```

  - 设置键入`Tab`键时写入`\t`而不是空格：

    ```json
    "editor.insertSpaces": false
    ```

    此外，如果要设置一个`\t`的空格数以及一个缩进的空格数，可以设置`editor.tabSize`以及`editor.indentSize`
  - 设置接受快速建议时，替换右侧的代码：

    ```json
    "editor.suggest.insertMode": "replace"
    ```

  - 设置在宽度不足时换行显示：
  
    ```json
    "editor.wordWrap": "on"
    ```

- 通过设置`"[xxx]"`字典，可以设置专门适用于某种文件的配置，例如：

  ```json
  "[java]": {
    "editor.defaultFormatter": "redhat.java"
  }
  ```

  很常用的是对不同的文件配置不同的默认格式化程序

## 推荐插件

### 中文与颜色主题

- `Chinese (Simplified) (简体中文) Language Pack for Visual Studio Code`：简体中文插件
- 设置颜色主题：在左下齿轮图标中设置`主题>颜色主题`，或通过搜索栏搜索`>首选项: 颜色主题`，第一项可以搜索查找本机没有的主体并自动下载对应的插件
- 推荐的主题(浅色党)，安装对应的浅色主题插件后会有对应的暗色主题
  - `Default Light Modern`：`VS Code`的内置浅色主题
  - `Github Light`：类`Github`风格
  - `Monokai Pro Light`：偏灰色主题
  - `Falcon Light`：比较护眼的主题，有青、绿、灰、粉、黄色
  - `Trae Light`、`Pure White`、`Solarized Light`也是不错的主题
  - 其它主题可在<https://vscodethemes.com/>快速预览

### `Markdown`

- `Markdown PDF`：支持导出为`PDF`、`HTML`、图片、`json`等
- `markdownlint`：语法高亮与风格检测，同时提供更好用的格式化程序，通过以下配置来设置格式化程序、忽略部分警告：

    ```json
    "[markdown]": {
        "editor.defaultFormatter": "DavidAnson.vscode-markdownlint",
    },
    "markdownlint.config": {
        // 警告与错误提示
        "default": true,
        "MD033": false,
    }
    ```

- `Markdown Preview Enhanced`：提供更好的预览，通过以下配置来支持行内`LaTeX`公式渲染：

    ```json
    "markdown-preview-enhanced.mathRenderingOption": "MathJax"
    ```

- `Markdown Preview Mermaid Support`：提供`mermaid`代码块的渲染
- `GitHub Markdown Preview`：使用`Github`的预览风格

### `C/C++`

- `C/C++`：
- `C/C++ Runner`：支持运行`C/Cpp`代码，包含以下配置：

  ```json
  // 额外包含的路径
  "C_Cpp_Runner.includePaths": [
  ],
  // C/Cpp标准
  "C_Cpp_Runner.cStandard": "c17"
  "C_Cpp_Runner.cppStandard": "c++17"
  ```

- `clangd`
- `CMake`与`CMake Tools`

### `Java`

- `Extension Pack for Java`
- ?`Java Language Support`
- `Spring Boot Extension Pack`

### 远程连接

- `Remote Development`

### 数据库

- `Database Client`

### `Vim`

- `Vim`
- `settings.json`代替了`vimrc`配置：
  - 配置插入模式的快捷键，建议绑定`esc`

    ```json
    "vim.insertModeKeyBindings": [
        {
            "before": [
                "j",
                "j"
            ],
            "after": [
                "<esc>"
            ]
        }
    ]
    ```

    在插入模式下键入两次`j`等价于键入`esc`
  - 配置在正常模式下的快捷键(例如将保存文件设置为`<leader>+s`)：

    ```json
    "vim.normalModeKeyBindingsNonRecursive": [
        {
            "before": [
                "<leader>",
                "s"
            ],
            "commands": [
                ":w!"
            ]
        },
        {
            "before": [
                "<leader>",
                "q"
            ],
            "commands": [
                ":q!"
            ]
        }
    ]
    ```

  - 配置`leader`键为空格(默认为`\`)：

    ```json
    "vim.leader": "<space>",
    ```

### `LaTeX`

- `LaTeX`

### `Vue`

- `Vue (Official)`
- `Vetur`

### `Python`

- `Python Extension Pack`
- `Black Formatter`
- `Flake8`
- `Jupyter`

### `Rust`

### 其它插件

- `GitHub Pull Requests`：支持在`VS Code`中提`PR`与讨论`Issue`
