---
title: "Python: 使用argparse开发CLI"
date: 2025-01-20
categories: [Programming, Python, Spec & Stdlib]
tags: [CLI]
---
<!-- placeholder -->
<!-- more -->
### 前置命令行工具知识

- `argparse`是用于开发命令行工具接口的库，通过内建的解析器来解析自定义的命令行参数
  相比于`optparse`(`py2.7`后被`argparse`替代，并不再维护)，接口功能更强，但灵活性略低
  关于`getopt`，它是基于`C`的`getopt()`函数族(通常可以在`Linux`看到)实现的一套很底层的接口，比较难用(对比`optparse`接口功能更加单一，需要手动实现复杂功能)

- 对于一个开发完毕的命令行工具脚本(假设名为`a.py`)，其固有(库会自动添加)的命令是`-h`或`--help`

  ```sh
  python a.py -h
  ```

  将输出开发者自定义的命令参数及其帮助字符串
  常使用命令行工具的开发者应该已经知道，含`'-'`和`'--'`前缀的参数为可选项，其中前者为命令缩写、后者为命令全称；上述`-h`的全称为`--help`
  称不含此类前缀的参数为**位置参数/占位参数**，即用户必须传递的参数
  此类可选项为**可选参数**，即用户可以根据情况传递的参数(并附上选项缩写或全称)

- 必须进行约定，以区分开发者向接口传递的参数和用户输入的参数，我们将后者始终称作用户输入(或使用位置参数和可选参数这两个名词)，其余的参数均为接口的参数、或明确说明是“开发者传递的”
  用户输入可选参数，需要指出选项缩写或全称，再跟实际要传递的参数
  用户输入位置参数，不需要上述操作，例如：

  ```sh
  $ python a.py 1  # 位置参数不需要选项名
  $ python a.py -o 1 # 可选参数需要以选项名-o为该项参数的起点, 后续1为实际要传递的参数
  # 假设有若干个位置参数(a1, a2, ..., an), 那么它们必须按部就班地顺序排列
  $ python a.py 1 2 ... n
  # 可选参数可以在任意位置插入其中
  ```

  可选参数可以随意插入其中并以选项名作为起点，那么解析器如何区分它们是连续的位置参数、还是掺杂着可选参数的位置参数呢？
  实际上，**可选参数后面的实际要传递的参数，默认只能传递一个**
  后面可以看见，可以设置让可选参数后可跟多个实际参数，此时可以用显式分隔符`--`解决上述问题：

  ```sh
  python a.py -o 1 2 3  # [1, 2, 3] -> o
  python a.py -o 1 2 -- 3 # [1, 2] -> o、3 -> 位置参数

### 最常用`API`

- `argparse`库通过`ArgumentParser`类(参数解析器)实现其功能，创建一个解析器对象：

  ```python
  p = argparse.ArgumentParser()
  ```

  - 构造方法的默认参数基本不用修改(几乎都是对帮助信息的设置)，故可以为空；如果想添加对该命令行工具概括性的描述，可以传递关键字参数`prog:str、usage:str、description:str`
    该描述将在用户传递`--help`时在`positional/optional argument`前将程序名称、程序使用说明、功能描述打印在屏幕上
  - 一些其次的参数：
    `add_help:bool=true`：是否自动添加帮助命令
    `parents:Sequence[ArgumentParser]`：从其它解析器对象继承参数(如要使用，其它解析器对象的`add_help`最好设置为`false`)
    `prefix_chars:str='-'`：如其名

- **`add_augument()`**：添加用户需要传递的命令行参数，包含一个可变参数及若干关键字参数
  解析器将用户输入的参数**存储在一个命名空间**，以**用户输入的参数作为值**、
  以开发者传入的**选项名作为键**，**传入的选项名应该符合命名规范(除前缀符)**

  - `help:str`：对于开发者来说是必填的，将显示在帮助文档中对应参数后，表示该项参数的功能

  - **`*name_or_flags:str`**：
    开发者传递单个字符串时，可以是位置参数或可选参数；**传递多个字符串时，只能全部是可选参数**
    若为可选参数，如果全为缩写(即前缀为`-`)，则将第一个缩写视为键
    如果**存在全称(即前缀为`--`)，则将第一个全称视为键**
    对一个正常的开发者来说，只会设置一个缩写与一个全称，所以不用了解更多的机制
    一个简单的例子：

    ```python
    # 设置时没有设置每个参数的数量, 默认为接收1个
    p.add_argument('position_name1')# 添加position_name1作为位置参数
    p.add_argument('p_name2')  # 添加p_name2作为位置参数, 必须在上述参数后传递
    p.add_argument('-a', '--aaa') # 添加全称aaa作为可选参数
    args = p.parse_args()
    
    # 用户视角:
    > python a.py 1 -a 2 3
    # 传递后, args.position_name1== '1'、args.p_name2 == '3'、args.aaa == '2'
    # args.a不存在, 因为提供了全称aaa
    ```

  - **`type`**：上述例子中，传递后的属性均为字符串，因为解析器会**默认将输入视为字符串**
    可以用`type=otherType`让解析器进行类型转换，其中`otherType`应是一个**类型转换的函数指针**
    其**默认可以选择`int`、`float`**并在参数**不合法时抛出`argparse.ArgumentTypeError`**
    开发者**可以自定义**类型转换用于处理传入的字符串，记得在参数不合法时手动`raise argparse.ArgumentTypeError`
    也可以把该参数看作一种对用户输入的类型限制

  - **`choices:list`**：用于限定输入的值(必须是列表内的值)，注意**列表元素的类型应与`type`一致**

  - **`dest`**：用于限定输入值存储于指定属性(属性名必须符合规范)；默认为本条`argument`的名称
    这样就**可以随意命名选项名**了，可以不符合命名规范

    ```python
    p.add_argument('-a') # 用户输入->args.a
    p.add_argument('-a', dest='b') # 用户输入->args.b
    p.add_argument('-a--abc', dest='b') # 用户输入->args.b

  - **`default`**：设置`default`参数后，用户可以选择**不输入该项(包括选项名)**并以`default`的值作为用户输入

  - `const`：设置`const`参数后，用户可以选择**不输入该项的实际参数**并以`const`的值作为用户输入
    要使用`const`，必须同时**设置`nargs='?'`**、或者**设置`action`为常量相关行为**

  - **`nargs`**：指定用户输入的数量

    - 非负整数`N`：指定用户必须输入`N`个值

    - `'?'`：用户可以输入零/一个值

    - `'*'`：用户可以输入零/多个值，存储为列表，用户可以用显式分隔符`--`分割不同参数项

    - `'+'`：用户可以输入一/多个值，存储为列表

    设置为收集为列表的选项时，自动设置`action='extend'`

  - **`action:str='store'`**：用于解析器对用户输入**最初的行为**
    **视`action`的值，其它参数可能不允许设置**
    一些常见的`action`：

    - **`store`**：默认值，存储用户输入

    - `store_true/store_false`：将本`argument`**视作布尔值**，仅在本条参数为可选参数时有效
      用户只需要指出这个可选项的名称(缩写或全称)，**后面不允许接任何参数**
      当`action`设置为`store_true`时，用户指出这个可选项则本条参数设置为`true`
      `store_false`同理

      ```python
      p.add_argument('-c', action='store_true')
      
      # 用户视角:
      > python a.py -c 1  # 出错, -c后不允许跟参数(除非有其它位置参数在后面)
      > python a.py -c  # 正常, 且args.c == True
      > python a.py   # 正常, 且args.c == False
      ```

      因其特性，**`nargs、type、choices`不允许设置**

    - **`append`**：允许**多次使用本选项**(而不是一个选项名后接若干个参数)，并将所有参数全部收集成为列表类型

    - `store_const`：行为类似于`store_true`，用户不允许提供实际参数，并使用`const`参数的值作为属性值；仍不允许设置`nargs、type、choices`
      `append_const`：在`store_const`的基础上增加了`append`的特性，其它参数的限制同上

    - 也可以继承`Action`类自定义行为，有兴趣可以了解[Python 命令行之旅_博客园](https://www.cnblogs.com/xueweihan/p/11415703.html)

- `parse_args()`：返回一个命名空间，其中存储用户输入的各个参数值

### 其它`API`(了解即可)

- `parse_known_args()`：在`parse_args()`的基础上，收集用户多余的输入(不报错)；返回二元组，前者为`parse_args()`的返回值、后者为收集到的多余输入(列表)

- `add_argument_group()`：返回一个参数分组，用法和解析器对象一致，方便将用户输入分给多个解析器对象管理，适用于维护大型命令行工具
  参数分组只能添加参数/添加参数分组，其本身及其子分组收集到的参数均上交给解析器，并由`parse_args()`展现给开发者

- `add_mutually_exclusive_group()`：在`add_argument_group()`的基础上，限制用户一次命令只能输入组内的其中一个参数

- `add_subparsers()`：返回子解析器，并允许开发者设置子命令，当主解析器遇到子命令时，将解析任务交给子解析器
  `add_parsers(name:str)`：子解析器可以调用该方法返回一个子命令参数解析器，必须指定子命令的名称`name`，其它参数和`ArgumentParser()`构造方法一致
  拥有多个子解析器是没有意义的，一个子解析器足矣，正确用法是**通过这个子解析器调用多次`add_parsers()`返回多个参数解析器对象**，这些对象的`API`和之前学过的常用`API`一致
  可以将子解析器看作一个中间件

  ```python
  p = argparse.ArgumentParser()
  subp = p.add_subparsers()
  subp1 = subp.add_parsers(name='init')
  subp1.add_argument('-a')
  
  # 用户视角
  > python a.py init 1 # 1 -> subp1.parser_args().a init为子命令1

- `error()`：默认的错误处理方法

  可以将解析器的`error`函数指针替换为自定义错误处理函数
