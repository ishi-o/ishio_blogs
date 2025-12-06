---
title: "C: 基本语法"
date: 2023-10-09
categories: [Programming, C/Cpp]
tags: [beginner]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# C

## 预处理

### 介绍

在编译一个.c文件时，最开始的操作便是预处理，它会将包含的文件展开，把使用过宏的地方替换为宏定义的语句，删除注释等等...

可以用`gcc`生成一个预处理后的文件(后缀为.i)

```shell
# -E选项表示在预处理后停止，-C表示保留库函数里的注释，-o表示自定义输出文件的名称
gcc -E -C file.c -o file.i
```

### 宏

宏，如其名，它将一串命令序列转换为简短的语句，用短短几个单词即可以实现强大复杂的功能

预处理器对宏实际的操作是把参数作为**字符串**来替换并把这条替换后的语句**插入**程序控制流里

### 运算符

#### \\(宏延续)

它用于**隔开**两个语句，当宏太长时，可用`'\'`分隔为多行

#### #(字符串常量化)

它可以将宏参数**转换**为字符串，字符串的内容是宏参数的名称，如要在字符串中转换需要再加上一层双引号

#### ##(标记粘贴)

它可以将两边的标记合并成一个标记

#### defined(已定义)

它接受一个标识符，若这个标识符已定义，返回`true`，否则返回`false`

### 处理目标

#### #include

各种语言都有官方的函数库，这些库包含了许多优化度极高或十分方便的宏，想要使用它，我们需要声明

```c
#include <stdio.h>
```

上述语句包含了一个名为`stdio.h`的文件，如果使用`gcc`生成一个预处理后的文件(后缀为.i)，我们会发现这一文件条目很多，正是展开这一文件的效果

我们也可以用**双引号**包含，这时预处理器会**先从当前目录**而不是从库函数目录里寻找

```c
#include "myStd.h"
```

在我们可以编写自己的头文件时，需要用这种包含方式包含

#### #define

这是定义一个常量，或是一个简易函数等的方法

```c
#define COL 50
#define PRINT(n) printf(""#n"是%d", n)
#define PRINTMORE(...) printf(__VA_ARGS__)
#define GIVE_NEW(n) aNum##n
```

上述语句的第一条定义了一个名为`COL`，大小为`50`的常量

第二条语句定义了一个`PRINT`宏函数，传入`(int)n`，打印`"(n的名称)是(n的数值)"`

双引号括住#+宏参数名可以解析它为它的名称

第三条语句的宏参数为`"..."`，它可以**接受多个**参数，并把这些参数传给`__VA_ARGS__`

第四条语句被用于**批量创建变量**，##即黏合剂，将两边记号黏着

```c
int GIVE_NEW(3)= 5; // 创建一个名为"aNum3",值为5的int型变量
```

需要注意的是，当函数含参时，需要加上()括住，原因是传参时它将表达式参数当作字符串处理，加上括号才可以达到想要的结果

```c
#define POWT1(n) ((n)*(n))
#define POWT2(n) (n*n)
POWT1(5+4)==(5+4)*(5+4)==81;
POWT2(5+4)==5+4*5+4==29;
```

#### 使用场景

宏是内联函数，即在预处理阶段时会把相应语句直接**插入**到程序中

与一般函数不同，它不需要跳转，跳出，而只需要牺牲一定的空间；而一般函数会被保存为副本，每次调用都必须跳到入口进，从出口返回，这将**增大时间成本**

因此，在一个函数十分**简单**时，可以将它写成一个宏，牺牲空间，换取时间

#### todo:编写简单的头文件

接下来，将通过编写一个简单的头文件来帮助熟悉`#ifdef,#undef,inline`等语句和关键字

## 编译

### 介绍

`C`语言的编译过程即将高级语言转换为汇编语言的过程，它会将所有语句转换为汇编指令，并且会优化源代码

可以用`gcc`生成一个编译后的文件(后缀为.s)

```shell
gcc -S file.i -o file.s
```

可以发现生成后的文件已经是汇编代码了

### C语言语法

在这个阶段，开始涉及`C`语言的各种语法

#### 数据类型

`C`语言有许多关键字，其中不少是用于声明变量的

```c
     // 占用字节  16位  32位  64位
char ch = 'a';  // 字符型     1  1  1
short sn = 1;  // 短整型     2  2  2
int dn = 1;   // 整型     2   4   4
long ln = 1;  // 长整型     4  4  8 (尽量不使用,因为可能导致不兼容)
float fn = 1.1;  // 单精度浮点型  4    4    4
double lfn=1.1;  // 双精度浮点型  8    8    8
long long lln=1; // 长长整型        8     8
long double ldn=1; //        12    16
     // 所有指针类型  2     4    8
// unsigned:表示无符号，并不会改变位数
```

4位字节如`int`范围为`-2147483648~2147483647`，`unsigned int`范围为`0~4294967295`(20~43亿)($2^{32}$)

8位字节如`long long`范围为`-9223372036854775808~-9223372036854775807`

​      `unsigned long long`范围为`0~18446744073709551615`($2^{64}$)

除了基本的数据类型，还可以声明**结构体**来储存多个类型的值

```c
struct STRUCT {
    int a;
    char b;
    char* p;
    char arr[10];
};
```

给结构体变量的分配空间遵循**对齐**原则

1. 从第一个成员开始往下分配空间，默认从地址0开始存放
2. 每个成员的**偏移量**必须是**该成员**基本数据类型的整数倍
3. 总大小是**最大**成员基本数据类型的整数倍

例如，`char`占1个字节，那么它可以放在0,1,2,3,...地址上；`int`占4个字节，那么它可以放在0,4,8,12,...地址上

指针类型占8个字节，那么它可以放在0,8,16,...地址上；数组也同理，将它看作许多个基本类型成员的集合计算

#### 数据间的转换

`C`语言的`char`型比较特殊，它被转换为`int`数据存储，并用`ASCII`编码，可以和`int`类型数据相加减

<a href="https://c.biancheng.net/c/ascii/">ASCII码表</a>

它可以用"%d"转换符输出，但并非是一个整型数据!

一般数据类型的转换如图所示，在**赋值**运算中可以进行自动类型转换将**右值转换为左值**的类型，在**表达式**运算中可以**从低到高**进行自动类型转换，例如整数相除的结果会被去掉小数部分
$$
char\rightarrow int\rightarrow unsigned\rightarrow long\rightarrow double\leftarrow float
$$
指针类型十分严格，如果类型不同，则不能进行互相转换

`void*`无类型指针是特例，可以强制转换为任意类型的一级指针，它是**抽象**的，并不具有一般指针类型变量的加减或给其他变量赋值的操作，它只能是左值

#### 数据的前缀

```c
011  // 表示八进制
0x11 // 表示十六进制
0b11 // 表示二进制
'\072' // 表示转义为一个072对应的字符,实际占一个字符的空间
```

#### 数据的后缀

```c
// 浮点数的指数表达法,后面的数字必须是整数[1.0或表达式(5+4)也是错误的],e或E均可
// e的前面数字只能是(0,10)间的数据
1.1e2 // 表示1.1*pow(10,2)
-2.3e-4 // 表示2.3*pow(10,-4)
    
// 其它后缀
1u(或U)  // 表示是一个无符号数据,可以后加l或L表示无符号长整型
1.1f(或F) // 表示是一个float类型数据,若无后缀则表示double类型
2.2L(或l) // 表示是一个long double类型数据
```

#### 运算符

<table border="2">
    <tr>
        <td>运算符</td>
        <td>优先级</td>
        <td>运算顺序</td>
    </tr>
    <tr>
        <td>[]</td>
        <td rowspan="4" style="display:table-cell; vertical-align:middle">1</td>
        <td rowspan="4" style="display:table-cell; vertical-align:middle">从左到右</td>
    </tr>
    <tr>
     <td>()</td>
    </tr>
    <tr>
        <td>对象成员.</td>
    </tr>
    <tr>
        <td>指针成员-></td>
    </tr>
    <tr>
        <td>单目-</td>
        <td rowspan="8" style="display:table-cell; vertical-align:middle">2</td>
        <td rowspan="8" style="display:table-cell; vertical-align:middle">从右到左</td>
    </tr>
 <tr>
      <td>取反~</td>
 </tr>
 <tr>
      <td>自增++/自减--(包括前后缀)</td>
 </tr>
    <tr>
        <td>解引用*</td>
    </tr>
    <tr>
        <td>取地址&</td>
    </tr>
    <tr>
        <td>!</td>
    </tr>
    <tr>
        <td>强制类型转换(type)</td>
    </tr>
    <tr>
        <td>sizeof</td>
    </tr>
    <tr>
        <td>/</td>
        <td rowspan="3" style="display:table-cell; vertical-align:middle">3</td>
        <td rowspan="19" style="display:table-cell; vertical-align:middle">从左到右</td>
    </tr>
    <tr>
        <td>乘法*</td>
    </tr>
    <tr>
        <td>%</td>
    </tr>
    <tr>
        <td>+</td>
        <td rowspan="2" style="display:table-cell; vertical-align:middle">4</td>
    </tr>
    <tr>
        <td>-</td>
    </tr>
    <tr>
        <td><<</td>
  <td rowspan="2" style="display:table-cell; vertical-align:middle">5</td>
    </tr>
    <tr>
        <td>>></td>
    </tr>
    <tr>
        <td>></td>
  <td rowspan="4" style="display:table-cell; vertical-align:middle">6</td>
    </tr>
    <tr>
        <td>>=</td>
    </tr>
    <tr>
        <td><</td>
    </tr>
    <tr>
        <td><=</td>
    </tr>
    <tr>
        <td>==</td>
   <td rowspan="2" style="display:table-cell; vertical-align:middle">7</td>
    </tr>
    <tr>
        <td>!=</td>
    </tr>
    <tr>
        <td><=</td>
    </tr>
    <tr>
        <td>按位&</td>
  <td rowspan="1" style="display:table-cell; vertical-align:middle">8</td>
    </tr>
    <tr>
        <td>按位^</td>
  <td rowspan="1" style="display:table-cell; vertical-align:middle">9</td>
    </tr>
    <tr>
        <td>按位|</td>
        <td rowspan="1" style="display:table-cell; vertical-align:middle">10</td>
    </tr>
    <tr>
        <td>&&</td>
        <td rowspan="1" style="display:table-cell; vertical-align:middle">11</td>
    </tr>
    <tr>
        <td>||</td>
        <td rowspan="1" style="display:table-cell; vertical-align:middle">12</td>
    </tr>
    <tr>
        <td>?:</td>
        <td rowspan="1" style="display:table-cell; vertical-align:middle">13</td>
        <td rowspan="2" style="display:table-cell; vertical-align:middle">从右到左</td>
    </tr>
    <tr>
        <td>赋值符</td>
        <td rowspan="1" style="display:table-cell; vertical-align:middle">14</td>
    </tr>
</table>

基于优先级以及结合顺序，在这里讨论一些容易混淆的语句

```c
// ++与*优先级相等,而它们是从右到左结合的
int* p;
(*p)++;  // 括号优先级最高,语句意义为先取p指向的值,然后让p指向的值自增
*p++;  // 先取p指向的值,然后让p自增
++*p;  // 先让p指向的值自增,然后取p指向的值
++(*p);  // 与第三条语句等价
```

赋值运算符优先级极低，且赋值表达式的结果就是右值的结果

### 数组

数组将多个相同类型的数据连在一起，方便进行访问和管理，许多数据类型的结构都可以由数组实现

#### 二维数组

初始化二维数组时，它的**列**不能为空

```c
// 一层循环遍历二维数组
for (int i = 0; i < M * N; i++)
    printf("%d ", a[i / N][i % N]);
```

#### VLA(变长数组)

`VLA`是一个概念，有许多种方法可以实现它，其中肯定需要用到的是`<stdlib.h>`里的`malloc()或calloc()和free()`函数(在C++中可用`new和delete`关键字)

引入指针后，`VLA`的实现便很容易理解了

```c
#include<stdlib.h>
int n = 10;
int* pa = (int*)malloc(n * sizeof(int)); // 接受一个参数:元素个数*单个元素空间
int* pa0 = (int*)malloc(n, sizeof(int)); // 会进行初始化,效率比malloc低
free(pa); // 释放内存
```

### 函数

`C`语言是一门面向过程的编程语言，函数对于它十分重要，它能把程序划分为多块，分别实现不同的功能

#### 接口与返回值

```c
type function(paras){statements}
```

上述例子里，`paras`表示参数，它是进入这个函数的入口时由上一进程传入的数据，`type`规定了`function()`的返回值的类型，如果不显式声明则**默认**为`int`

#### 形参

在函数中改变形参值无法改变上一进程中传进来的值，如要改变，可以传指针进行修改

#### 递归

递归很适合用于处理需要**逆向**思考的问题，例如一包数据的倒序存储等；但递归也容易造成**栈溢出**，递归的层级越大，风险便越大

涉及递归函数时，首先需要找到**递归出口**，即**递归基础**，每进行一次递归，数据应越来越靠近这一出口

其次是**递归步骤**，需要找到每次递归的数据间的联系与规律

### 指针

`C`语言的指针强大，灵活

#### 指针基本类型

<table border="2">
    <tr>
        <td>数据类型</td>
        <td>名称</td>
        <td>意义</td>
    </tr>
    <tr>
        <td>type*</td>
        <td>一级指针</td>
        <td>指向一般数据类型的指针</td>
    </tr>
    <tr>
        <td>type**</td>
        <td>二级指针</td>
        <td>指向指针的指针</td>
    </tr>
    <tr>
        <td>type[*](n)</td>
        <td>数组指针(行指针)</td>
        <td>指向数组的指针</td>
    </tr>
</table>

#### 数组名的类型以及易混淆的声明

数组名的类型很好记，只要知道该数组装载着的**最高维数据的类型**，就知道数组名的类型是**指向这一类型的指针**

<table border="2">
    <tr>
        <td>声明方式</td>
        <td>数据类型</td>
        <td>意义</td>
    </tr>
    <tr>
        <td>type a[n]</td>
        <td>type*</td>
        <td>本身是一个一维数组，a是指向一般数据的常量指针</td>
    </tr>
    <tr>
        <td>type a[m][n]</td>
        <td>type[*](n)</td>
        <td>本身是一个二维数组，a是指向含n个元素的数组的常量指针</td>
    </tr>
    <tr>
        <td>type *a[n]</td>
        <td>type**</td>
        <td>本身是一个指针数组，a是指向指针的常量指针</td>
    </tr>
    <tr>
        <td>type[*a](n)t</td>
        <td>type[*](n)</td>
        <td>指向一维数组的指针，a[i]是未定义的</td>
    </tr>
</table>

#### 函数中的指针参数

##### 向函数传递指针

地址或数组名向函数传递时，都会**退化**为指针，而且建立一个形参指针变量

函数内的[形参]指针拥有一般指针变量的性质：例如可以进行加减(就算传入数组名，也会新建一个形参指针)

##### 由函数返回地址

返回形参或静态量的地址有意义

返回**局部**变量的地址无意义：局部变量离开函数时**被销毁**，故访问该地址没有意义

#### 关于赋值

指针类型限定很**严格**，只能用**相同类型**的值赋值给指针变量

```c
void func(int, int);

//几个赋值的例子：
int* p; 
int** pp;
int arr[n];
int arr2[m][n];
int *parr[n];
int(*pa)[n];

p = arr; // √
pp = parr;    // √
pa = arr2; // √
pa[?] = ??? // ×
void (*pfunc)(int, int); // 声明函数指针
(*pfunc)(a, b); // 调用函数指针
```

### 位运算

## 汇编

### 介绍

汇编即把汇编代码转换为机器语言(二进制机器码)的过程

可以用`gcc`生成一个汇编后的文件(后缀为.o)

```shell
gcc -c file.s -o file.o
```

### 汇编语言

#### 介绍

汇编语言(`assemble language,即asm`)**面向机器**，将许多长串的二进制码编写为一套**短小精炼**的命令语句，它帮助我们在寄存器、IO出口和存贮器间传递数据；将命令变为二进制码让机器听懂等。如果读者常使用IDA，则对汇编语言会有所了解

每一种处理器都有它们的一套汇编指令，它是软件开发者和机器直接进行交互的桥梁，对理解硬件之间的联系很有帮助

高级语言中提供的十六进制与八进制，正是因为汇编语言的数据会以2的幂来存储

#### 数据间的转换

二进制数据每进四个位对应的十六进制数将进一位，每进三个位对应的八进制数将进一位

$2^n$进制数拥有的这种特性让计算变得规律易懂

```assembly
1111-1011-1110-0101  ---> 0xF-B-E-5
0xBA93     ---> 1011-1010-1001-0011
```

#### 负数的表示:二进制补码

```assembly
-num == ~num + 1
# 即先取反,再加1
```

数据相减即让减数取反加1后进行加法运算，因为最后一位会因进位而丢失，所以可以正确表示两数相减

如果规定为无符号数据，则最后一位参加，最大值变为两倍(即$2^8$)，否则为$-2^7\sim2^7-1$

#### 数据寻址

处理器一次可以访问多个字节的内存，它先从内存**获取指令**，然后**解码**，最后**执行**；存储数据时，从低位开始压入栈，即会将低位字节存储于低地址上；当它取出数据时，会从高位开始取，这就像先入后出的栈一样

通过这种方式，处理器搭建了一座**内存和寄存器间**的桥梁

#### NASM

`NASM`全称`The Netwide Assembler`，可以在`Windows`和`Linux`下使用

我们在`Linux`环境下配置`NASM`，输入`sudo apt install nasm`即可

#### 汇编语法

汇编分为三部分:`data, bss, text`

```assembly
section.data
section.bss
section.text
 global _start ;entre
_start:
;explain
```

`data`: 初始化数据或常量

`bss`: 声明变量

`text`: 保存实际代码，必须以**`global _start`**开头，它规定了执行的开始位置

`;`: 注释，以英文`';'`开头

内存段分为数据段、代码段和堆栈段，数据段由`.data和.bss`表示、代码段由`.text`表示

#### 汇编语句

汇编有三种类型的语句，且汇编语言**不区分大小写**

1. 可执行指令
2. 汇编器指令(伪指令)
3. 宏指令

第一种语句告诉处理器要做什么，每一条语句汇编后都是一组二进制数，可**被CPU执行**；第二种语句不会变成机器语言，它们不可执行，但为汇编程序**提供汇编信息**(如数据和指令的区分，或数据的**字长**，或数据地址等)，**辅助**源程序的汇编；宏即一种文本替换机制，用于提高编程效率

每行只有一条语句，每条语句遵循下面的格式

```
[label]      mnemonic  [operands]   [;conment]
[名字]           指令助记符  [操作数]    [注释]
0~9,a~z,A~Z,_,?,@,$   实际可执行指令  数据或数据所在地址
方括号内是可选项
```

#### 数据项

**常量**：直接编码于指令中，不额外分配主存空间，也不存储在存贮器里

<table border="2">
    <tr>
        <td>进制</td>
        <td>后缀</td>
    </tr>
    <tr>
        <td>十进制</td>
        <td>d/D(可省略)</td>
    </tr>
    <tr>
        <td>十六进制</td>
        <td>h/H,以A-F开头时需在最高位加0</td>
    </tr>
    <tr>
        <td>二进制</td>
        <td>b/B</td>
    </tr>
</table>

**变量**：保存在可读可写的主存空间，声明一个变量的指令如下

```assembly
name DB n dup(2,dup(4)) ;dup为复制操作符,它会将括号内数据复制n遍,例如本例n=2: 2,4,4,2,4,4
name DW 20 dup(?)  ;?表示随机数,常用于预留存储空间
name DW 22h,11h   ;从左到右压入堆栈,即地址22h低于11h
;每个数据遵循小端存储,即高位数据存入高地址
```

定义变量的伪指令如下图

<table border="2">
    <tr>
        <td>助记符</td>
        <td>变量类型</td>
        <td>所占位</td>
    </tr>
    <tr>
        <td>byte(DB)</td>
        <td>字节</td>
        <td>8</td>
    </tr>
    <tr>
        <td>word(DW)</td>
        <td>字</td>
        <td>16</td>
    </tr>
    <tr>
        <td>dword(DD)</td>
        <td>双字</td>
        <td>32</td>
    </tr>
    <tr>
        <td>qword(DQ)</td>
        <td>四字</td>
        <td>64</td>
    </tr>
    <tr>
        <td>tword(DT)</td>
        <td>十字</td>
        <td>160</td>
    </tr>
</table>

**表达式**：

<table border="2">
    <tr>
        <td>运算符类型</td>
        <td>运算符</td>
    </tr>
    <tr>
        <td rowspan="5" style="display:table-cell; vertical-align:middle">算术运算符</td>
        <td>+</td>
    </tr>
    <tr>
        <td>-</td>
    </tr>
    <tr>
        <td>*</td>
    </tr>
    <tr>
        <td>/</td>
    </tr>
    <tr>
        <td>mod</td>
    </tr>
    <tr>
        <td rowspan="4" style="display:table-cell; vertical-align:middle">逻辑运算符(只能对数值进行的按位运算)</td>
        <td>and</td>
    </tr>
    <tr>
        <td>or</td>
    </tr>
    <tr>
        <td>not</td>
    </tr>
    <tr>
        <td>xor</td>
    </tr>
    <tr>
        <td rowspan="6" style="display:table-cell; vertical-align:middle">关系运算符(只能对数值的操作)成立返回0FFFFh(-1),否则返回0000h</td>
        <td>eq(=)</td>
    </tr>
    <tr>
        <td>le(<=)</td>
    </tr>
    <tr>
        <td>ge(>=)</td>
    </tr>
    <tr>
        <td>lt(<)</td>
    </tr>
    <tr>
        <td>gt(>)</td>
    </tr>
    <tr>
        <td>ne(!=)</td>
    </tr>
    <tr>
        <td rowspan="5" style="display:table-cell; vertical-align:middle">取值运算符</td>
        <td>seg(获取段地址)</td>
    </tr>
    <tr>
        <td>offset(获取段内偏移地址)</td>
    </tr>
    <tr>
        <td>type(获取类型属性,返回该类型的值)</td>
    </tr>
    <tr>
        <td>length(返回变量中元素个数)</td>
    </tr>
    <tr>
        <td>size(返回所占数据区的字节总数)</td>
    </tr>
</table>

## 链接

### 介绍

之前提到预处理将展开库函数，事实上，这个过程导入的只有原型，真正的实现在链接库里

而链接即把机器码和其它文件，库文件，启动文件链接起来的过程，将会生成一个可执行文件

可以用`gcc`生成一个链接后的文件，已经到最后步骤了，需要做的事情很少

```shell
gcc file.o -o file.out
```
