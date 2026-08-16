---
title: "实现一个最小可用的 LSP Server"
date: 2026-08-15
categories: [Tools & Utilities, Editor, Neovim]
tags: [lsp, neovim, typescript, codex]
---

<!-- placeholder -->
<!-- more -->

# 实现一个最小可用的 LSP Server

LSP（Language Server Protocol）名称中虽然带有 "Language"，但它本质上是一套基于
**JSON-RPC 2.0** 的补全/诊断/跳转协议，服务的对象不限于编程语言。这篇笔记总结
**一个最小可用的 LSP server 需要暴露哪些 endpoint**，以及 Neovim 侧如何用 Lua
接入。协议细节与实现语言无关。

文中会以 [nvim-codex-lsp](https://github.com/ishi-o/nvim-codex-lsp) 作为贯穿全文
的示例：它为 Codex CLI 的聊天输入框提供补全，在 Neovim 中编辑 Codex 的 prompt
缓冲区时，可以补全 `/slash` 命令、`$skill` 引用和 `@文件/插件` 提及。这只是
LSP 应用于非编程语言场景的一个例子，笔记本身的方法论是通用的。

## LSP server 需要暴露什么

一个只提供补全和 hover 的 server，只需处理 **4 组消息**：

### 生命周期：`initialize` / `initialized` / `shutdown`

`initialize` 是客户端连接后的第一个请求，server 在**响应中声明自己的能力
（capabilities）**，客户端只会在声明过的能力范围内发起请求。提供补全只需声明
以下几项：

```json
{
  "capabilities": {
    "textDocumentSync": 2, // Incremental 增量同步
    "completionProvider": {
      "triggerCharacters": ["/", "$", "@"], // 输入这些字符时客户端自动请求补全
      "resolveProvider": true // 支持补全项的懒加载详情
    },
    "hoverProvider": true
  }
}
```

`initialize` 请求中还包含 `rootUri`/`rootPath`（项目根目录），这是 server 做
文件补全、扫描项目级配置的锚点，应当妥善处理。

### 文档同步：`textDocument/didOpen` / `didChange` / `didClose`

这些是**通知（notification，无响应）**。客户端在打开/修改/关闭文件时把全文或增量
推送给 server，server 维护一份内存副本（`vscode-languageserver` 的 `TextDocuments`
已经封装了这一逻辑）。之后所有补全、hover 都基于这份副本计算——**server 不应直接
读取磁盘上已打开的文件**，而应以客户端的版本为准。

### 补全：`textDocument/completion` → `CompletionItem[]`

这是核心请求。客户端传入光标位置 `position`，server 返回补全列表。要点：

- **上下文自行计算**：取光标所在行、从行首到光标的文本，用正则判断当前所处的
  上下文（示例中 `/` 后返回命令、`$` 后返回 skill、`@` 后返回文件+插件）。协议
  对"何时返回什么"没有任何约束，完全由 server 决定。
- `label` 是菜单中显示的内容；实际插入的内容用 `insertText`（补齐光标后缺少的
  字符）或 `textEdit`（替换一个 range，适合需要改写已输入文本的场景，比如把
  `@xx` 替换成 `$skill-name`）。
- `data` 字段可以携带任意自定义数据（比如命令名），它会在后续的 `resolve` 请求中
  原样带回，用于关联"补全时"与"详情时"两个阶段。
- 较长的文档不应放在 completion 响应中——见下一条。

### 懒加载详情：`completionItem/resolve` + `textDocument/hover`

- `resolve`：客户端在用户**选中**某个补全项时才发送。适合把 markdown 文档、耗时
  查询推迟到这里，使 completion 响应保持轻量。
- `hover`：光标停在一个词上时，返回 `{ contents: { kind:
"markdown", value }, range }`。实现方式是以光标处的词进行查表。

以上就是最小实现所需的全部内容。诊断（publishDiagnostics）、格式化、跳转定义等
都是可选能力，不需要就不必声明。

## 传输层：stdio + Content-Length 帧

LSP 最常用的传输方式是 stdio 上的 JSON-RPC，消息格式是**先写 header 再写 body**：

```
Content-Length: 123\r\n
\r\n
{"jsonrpc":"2.0","id":1,"method":"initialize",...}
```

- 使用官方 SDK（Node 的 `vscode-languageserver`、Rust 的 `tower-lsp`、Python 的
  `pylsp` 生态）可以省去手写帧解析
- 把 server **bundle 成单文件**（esbuild 之类），用户只需要一个 node 运行时，
  无需 npm install。把产物 commit 进仓库，插件即可开箱即用。
- 另外 server 进程是长驻的，**一次性的发现工作**（示例中是扫描 skill 目录、
  `spawnSync` 调用 CLI 获取插件列表）应在 `initialize` 中完成一次并缓存，避免在每次
  补全请求中重复执行。

## Neovim connection

0.11+ 的 Neovim 只需要 `vim.lsp.config` + `vim.lsp.enable`：

```lua
vim.lsp.config("codex_lsp", {
  cmd = { "node", server_js_path, "--stdio" },  -- server 路径相对插件根目录解析
  filetypes = { "markdown.codex" },             -- 只在特定 filetype 上附加
  root_dir = function(_, on_dir) on_dir(vim.fn.getcwd()) end,
})
vim.lsp.enable("codex_lsp")
```

真正需要仔细处理的是**让目标缓冲区被正确识别**。示例场景中，Codex 通过
`$EDITOR` 打开的临时文件名是随机的 `.tmpXXXXXX.md`，因此使用 autocmd + 回调过滤：

```lua
vim.api.nvim_create_autocmd({ "BufReadPost", "BufNewFile" }, {
  pattern = "*.md",
  callback = function(ev)
    if not is_target_buffer(vim.api.nvim_buf_get_name(ev.buf)) then return end
    vim.bo[ev.buf].filetype = "markdown.codex"   -- 复合 filetype
  end,
})
```

`markdown.codex` 是**复合 filetype**：前者使 markdown 的语法高亮/treesitter 继续生
效，后者用于精确限定 LSP 的附加范围。两个配套细节：

- `vim.treesitter.language.register("markdown", "markdown.codex")`——把 markdown
  的 treesitter parser 注册到复合 filetype 上，否则高亮会失效；
- 若要为 `/命令`、`$skill` 添加自定义高亮，用 `vim.cmd([[syntax match ...]])`
  叠加一层正则匹配即可。

## 总结

| Endpoint                                  | 类型 | 作用                                             |
| ----------------------------------------- | ---- | ------------------------------------------------ |
| `initialize`                              | 请求 | 声明 capabilities，获取 rootPath                 |
| `textDocument/didOpen/didChange/didClose` | 通知 | 维护文档内存副本                                 |
| `textDocument/completion`                 | 请求 | 按 triggerCharacters 触发，返回 CompletionItem[] |
| `completionItem/resolve`                  | 请求 | 选中项的懒加载文档                               |
| `textDocument/hover`                      | 请求 | 光标处词的 markdown 说明                         |

一个"补全 + hover"级别的 LSP server，核心逻辑不过两三百行：声明能力、同步文档、
按行上下文过滤列表。剩余的工作量全部在**领域侧**——扫描哪些目录、调用哪个 CLI、
上下文正则如何编写。这也是 LSP 设计的成功之处：协议只负责管道，语义完全留给 server。
