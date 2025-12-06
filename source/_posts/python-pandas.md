---
title: "Python: 使用Pandas处理数据"
date: 2025-01-18
categories: [Programming, Python, Data Science Utils]
tags: [pandas]
---
<!-- placeholder -->
<!-- more -->
### `pandas`介绍

- `pandas`名字来源于`panel datas`，即面板数据
- `pandas`主要用于数据分析及数据处理
- `pandas`最常用的两种数据结构是`Series`和`DataFrame`
  从形状上来看，`DataFrame`是二维数据结构的封装，`Series`是`DataFrame`中的一列，是一维数据结构的封装
- `pandas`提供的更重要的`API`包括数据清洗、数据预处理、数据操作、数据读取和导出

### `pandas.Series`

- 可以将`Series`看作更强的字典，提供了许多类似`numpy`数组的`API`
- 构造方法：`Series(data=None, index=None, dtype=None, name=None, copy=None)`
  - `data`：数据源，可以是列表、字典、数组
  
  - `index`：索引列表/数组

    当`data`提供的是列表或数组时，`index`默认为以`0`开头的递增索引序列，也可以提供**长度和`data`相同**的一维数据结构

    当`data`提供的是字典时，`index`若为`None`则默认`index`为该字典的键，`Series`中的值为该字典的值，若`index`不为`None`则起筛选作用，以`index`的值为索引值，若某索引值存在于`data`中则取`data`的值，否则为`numpy.nan`

    本质上是自动转换为一个`AxesData`对象(用于存储索引信息的数据对象)，可以通过自定义`index`实现类似字典的索引
  - `name`：该`Series`对象的名字，可以在`DataFrame`等结构中索引
- `Series`实例属性：
  - `index`：`Index`对象，表示索引数据

  - `values`：`numpy`数组，表示其存储的数据

  - `dtype`：存储数据的数据类型，可以在构造方法中指定，否则自动推断

  - `shape`：`Series`对象的形状，`Series`永远将`data`视为一维结构，即使传递一个二维数组
- `Series`实例方法：
  - `describe()`：返回`Series`对象，包含原对象的各种统计数据，包括`'count', 'mean', 'std', 'min', 'max'`

    也可以通过调用原对象的`count()`、`mean()`方法直接获取单个统计数据，其中`sum(), cov(), corr()`等是`describe()`没有涉及到的
  - `unique()`：返回去重后的`numpy`数组

  - `sort_index()`和`sort_values()`：返回根据索引或根据值排序后的`Series`对象

  - `dropna()`：返回删除了所有`nan`后的`Series`对象

  - `fillna(v)`：返回将所有`nan`用`v`填充后的`Series`对象

  - `to_list()`和`to_frame()`：转化为原生列表和`DataFrame`对象

  - `loc`和`iloc`：`pandas`支持切片语法，同时提供了另一种索引方法`loc(line of code)`，通过`loc[label]`或`iloc[index]`可以更清晰地表示索引，它们和原生索引的最明显**区别是`:`切片符是双闭切片**

    无论是原生切片符还是`loc`切片，`Series`对象都同样支持条件过滤，而且它实现了一般的运算符重载，均为逐元素运算，同`numpy`数组一样在运算时会自动对齐

  - `cumsum(), cumprod(), cummax(), cummin()`：返回前缀求和/求积/取最大/取最小处理后的`Series`对象
- 构造方法示例：

  ```python
  import pandas as pd

  s1 = pd.Series([1, 2])                # 0:1, 1:2
  s2 = pd.Series([1, 2], index=[2, 1])  # 2:1, 1:2
  s3 = pd.Series([[1, 2], [3, 4]])      # shape: (2,)   0:list[1,2], 1:list[3,4]
  s4 = pd.Series({1:2, 'a':3})          # 1:2, 'a':3
  s5 = pd.Series({1:2, 'a':3}, index=[1, 2])    # 1:2, 2:None
  ```

### `pandas.DataFrame`

- `Series`可以看作更强的字典，而`DataFrame`则可以看作值全为`Series`对象的字典，因此它是二维数据结构

  与`Series`不同的是，`DataFrame`有行索引和列索引
- 构造方法：`DataFrame(data=None, index=None, columns=None, dtype=None, copy=False)`
  - `data`：可以是列表、数组、字典，`DataFrame`永远将`data`视为二维数据结构来解析

    当`data`提供的是字典，则使用字典的键作为`columns`(列索引)，用字典的值作为列，要求每一列结构相同，索引统一为`index`参数(行索引)

    此外，`DataFrame`将`Series`看作一列，将二维数组的第一维看作行、第二维看作列

  - `index`和`columns`参数分别为行索引和列索引
- `DataFrame`实例方法：
  - 大多数方法和`Series`的实例方法一致，此外详细介绍`loc`系列索引方法

    `loc[x][y]`：按照先列再行的顺序索引，和原生的切片符不同，本质是先通过列索引取得`Series`再用`Series.loc`

    `loc[x, y]`：取`x`行`y`列，和原生的切片符一致

    `loc[[x1, x2, ...], [y1, y2, ...]]`：选取第`x1, x2, ...`行、第`y1, y2, ...`列的元素并返回`DataFrame`，其中这些值是离散的，不能用`:`

    `loc[[x1, x2, ...]]`：相当于在上述方法的基础上，默认选择全部列，只选择`x1, x2, ...`行的元素
  - 类型转换方法：`to_csv(), to_excel(), to_json(), to_sql()`

  - `transpose()`或`T`：返回转置后的`DataFrame`

### 读取数据

- 读取`csv`文件：`pandas.read_csv(filepath_or_buffer, sep, head, names, dtype, index_col)`

- 读取`excel`文件：`pandas.read_excel(io, sheet_name, header, names, index_col, usecols, dtype, engine, ...)`

- 读取数据库文件：`pandas.read_sql(query, conn_obj)`

- 读取网页文件：`pandas.read_html(url)`
