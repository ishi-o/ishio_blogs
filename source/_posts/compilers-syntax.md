---
title: "编译原理: 语法分析"
date: 2025-12-03
categories: [SE Courses, Compilers]
tags: [compiler]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 语法分析

## 文法分类

- `0`型文法：又称短语文法
- `1`型文法：又称上下文有关文法
- `2`型文法：又称上下文无关文法
- `3`型文法：又称正则表达式

## 上下文无关文法

- 结构：$(V_T,V_N,S,P)$，其中$V_T$是终结符集合，$V_N$是非终结符集合，$S$是开始符号，$P$是产生式集合
- 上下文无关文法允许产生式右部出现所有在产生式左部出现的非终结符
- 推导：
  - 最左推导：每次展开都展开最左边的非终结符
  - 最右推导(规范推导)：每次展开都展开最右边的非终结符
- 分析树：推导的树型表示
- 二义性：存在多种推导的可能结果，例如产生式集合$E\rightarrow E+E|E\times E$
  - 消除二义性：通过引入中间非终结符解决：

    $$
    \begin{align}
    &E\rightarrow E+T
    \\&T\rightarrow T\times T
    \end{align}
    $$  

  - 一般，优先级高的运算，其产生式应该离根更远
- 消除左递归：在递归下降分析程序里，由于程序无法看到后续的记号，因此如果存在左递归会存在一个函数递归调用自身而不继续解析的问题
  - 消除直接左递归：$A\rightarrow A\alpha|\beta$

    $$
    \begin{align}
    &A\rightarrow \beta A'
    \\&A'\rightarrow \alpha A'|\epsilon
    \end{align}
    $$  

  - 消除间接左递归：$\begin{cases}A\rightarrow B\alpha|\beta\\B\rightarrow A\gamma|\epsilon\end{cases}$

    实际上可以先转化为直接左递归：  

    $$
    \begin{cases}A\rightarrow B\alpha|\beta\\B\rightarrow B\alpha\gamma|\beta\gamma|\epsilon\end{cases}
    $$  

- 提取公共左因子：存在公共左因子会使程序不知道应该选择哪个，如果定死顺序则很可能出现错误分析

  提取公共左因子非常简单，只需要添加中间的非终结符即可：

  $$
  \begin{align}
  &A\rightarrow \alpha A|\alpha B
  \\\\&A\rightarrow \alpha C
  \\&C\rightarrow A|B
  \end{align}
  $$

## 分析法

### 自上而下分析`LL(1)`

- `LL(1)`要求产生式集合没有二义性、没有左递归、没有公共左因子
- 求`first`集合：
  - 终结符的`first`集合只有一个元素：它本身
  - 非终结符的`first`集合：
    - 先找到所有形如$S\rightarrow a\alpha\dots$的产生式，其右部的开始符号是终结符
    - 再找到所有形如$S\rightarrow A\alpha\dots$的产生式，其右部的开始符号是非终结符，将$first(A)-\epsilon$加入$first(S)$
    - 如果在第二步，$\epsilon\in first(A)$，那么使$first(\alpha)-\epsilon$加入$first(S)$，继续本步骤
    - 如果在第三步，遍历到最后的符号且最后的符号的`first`集包含$\epsilon$，那么将$\epsilon$加入$first(S)$
- 求`follow`集合：`follow`集合不包含$\epsilon$
  - 将`$`加入开始符号的`follow`集合里
  - 对所有$A\rightarrow \alpha B\beta$，将$first(\beta)-\epsilon$加入$follow(B)$
  - 对所有$A\rightarrow \alpha B$或$A\rightarrow\alpha B\beta$且$\epsilon\in first(\beta)$，则将$follow(A)$加入$follow(B)$
- 非递归的下降分析：要求构建分析表，则程序分析时不递归而是查找分析表找到对应的产生式
  - 行名为(栈顶的)非终结符，列名为输入符号，表格项为产生式
  - 对每个$A\rightarrow\alpha$，对$\forall a\in first(\alpha)$且$a\ne\epsilon$，使$M(A,a)$为这个产生式
  - 对每个$A\rightarrow\alpha$，若$\epsilon\in first(\alpha)$，则对$\forall a\in follow(A)$使$M(A,a)$为这个产生式
  - 若运行时遇到表格项为空，说明程序分析出错
  - 若遇到一个表格项包含多个产生式，说明不是`LL(1)`文法
- 错误恢复：
  - 词法错误
  - 语法错误
  - 语义错误
  - 逻辑错误
- 同步选择集合：

### 自下而上分析

- 句柄：和某个产生式右部一致的串
- 移进归约分析：
  - 移进：将下一个符号入栈
  - 归约：将栈顶的整个句柄用对应的产生式归约，用产生式的左部替换这个句柄
  - 接受：到达目标串的结尾且分析表未出错
  - 错误：分析表到达了空表项
- 移进归约冲突：栈顶的句柄可以继续移进，也可以归约的时候
- 归约归约冲突：栈顶可以有多种归约可能情况
- `LR(0)`(`Left-to-right, Rightmost derivation, 0 lookahead`)：非常容易产生移进归约冲突和归约归约冲突
  - 项目：一个产生式的右部的符号中间添加`·`符号
  - 项目集规范族：
    - 引入增广文法（添加产生式$E'\rightarrow E$指向开始符号）
    - 对扩展文法新增的产生式求$E'\rightarrow ·E$闭包，即只要`·`后是非终结符则要把这个非终结符为左部的所有产生式加入闭包，递归进行此步骤
    - 对上述闭包可以移进的所有符号，进行移进得到新的项目集
    - 重复第二步和第三步直到构建出`DFA`
  - 从`DFA`构建`LR(0)`分析表：
    - 对`Ix->Iy`出边为终结符`a`的情况：`ACTION[Ix, a]=sy`
    - 对`Ix->Iy`出边为非终结符`A`的情况：`GOTO[Ix, A]=y`
    - 对`Ix`能够归约的项目(产生式编号为`y`，产生式左部为`A`)：对所有终结符`a`，使`ACTION[Ix, a]=ry`
    - 对归约项目为增广文法新增的产生式，使表项为`acc`
    - 使其余所有表项为`error`
  - 使用`LR(0)`分析表对输入进行语法分析：
    - 初始状态栈只有一个元素：`I0`
    - 设当前状态为`Ix`，输入符号为`a`，查询`ACTION[Ix, a]`：
      - 若为`sy`，则状态变为`Iy`，输入后移，`Iy`入栈
      - 若为`ry`，第`y`个产生式为$A\rightarrow\alpha$，则从状态栈出栈，设出栈后栈顶状态为`Iz`则状态变为`GOTO[Iz, A]`
      - 若为`error`，则进入错误恢复程序
      - 若为`acc`，表示接受输入，语法分析成功
- `SLR`分析表(`SLR(1)`)：与`LR(0)`对比，引入了`follow`集合，对`Ix`能够归约的项目(产生式编号为`y`，产生式左部为`A`)：对所有`a`属于`follow(A)`，使`ACTION[x, a]=ry`
- `LR(1)`分析表：
  - `SLR(1)`分析仍然会出现移进归约冲突
  - `LR(1)`分析对项目进行修改：使其带搜索符，例如$A\rightarrow ·\alpha, a$
  - 项目集规范族：
    - 增广文法产生的项目为$E'\rightarrow·\alpha, \$$
    - 同样对扩展文法新增的产生式求$A'\rightarrow ·B\alpha, a$闭包，即只要`·`后是非终结符则要把这个非终结符为左部的所有产生式加入闭包，递归进行此步骤，其中它们的搜索符是$first(\alpha a)$
  - 生成分析表的过程和`LR(0)`的区别是：只对搜索符对应的表项设置为可归约项
- `LALR`分析表：`LALR`在`LR(1)`的基础上，将略去搜索符后一致的项目族合并，因此和`SLR`拥有相似数量的项目族，但`LALR`可能出现归约归约冲突
