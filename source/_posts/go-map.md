---
title: 'Go: map'
categories:
  - Programming
  - Go
  - Spec & Stdlib
tags:
  - Go
  - hash
  - data structure
mathjax: false
date: 2025-12-24T18:13:26.000Z
---
<!-- placeholder -->
<!-- more -->

# 映射表

- 在`Go`中， `map`作为关键字而不是标准库提供，并且十分简洁，底层使用哈希的方式实现
- 除了通过`map[keytype]valuetype{...}`的方式创建，还可通过`make(Type, cap)`分配容量并创建
- 访问`map`元素与访问数组类似，`obj[key]`，不同的是返回两个值`val, exist`，如果不存在则返回值类型对应的零值
- 新增或覆盖：`obj[key] = value`
- 删除键值对：使用内置函数`delete(map, key)`
- 映射表是一个`iterable`，可以使用`for range`遍历
- 同切片一样可以使用`clear()`清空表(`Go 1.21+`)
- 和其它内置的非`sync`类型一样，都是非并发安全的
- go`没有内置的`set`类型，因为`map[key]struct{}`完全可以作为`set`来使用，一个空结构体不会占用任何内存
- `maps`：`maps`是标准库中的映射表工具库，但内部函数较少，许多和`slices`库和[`strings`](/source/_posts/go-strings.md#strings)库的函数类似，也提供了[迭代器函数](/source/_posts/go-iterator.md#maps)

