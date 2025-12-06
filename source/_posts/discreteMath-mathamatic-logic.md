---
title: "离散数学: 数理逻辑"
date: 2024-04-19
categories: [SE Courses, Discrete Math]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 数理逻辑

## 命题逻辑

### 基本概念

- 命题：只有真、假两种状态的陈述语句，在命题逻辑里，原子命题不提供任何除真假以外的信息，这也导致命题逻辑的局限性
- 逻辑等价($p\equiv q$)：表示**$p$等价于$q$**，即有相同的真值表
- 否定($\neg p$)：表示**$p$的否定**
- 析取($p\vee q$)：表示**$p$或者$q$**
- 合取($p\wedge q$)：表示**$p$并且$q$**
- 异或($p\oplus q$)：表示**$p$异或$q$**
- 条件($p\rightarrow q$)：表示**若$p$，则$q$**
  - 从定义上看，只有当$p=T\wedge q=F$时违反了语句，故其它情况均**人为定义为$T$**
- 双条件($p\leftrightarrow q$)：表示**$p$当且仅当$q$**，与$p\odot q$等价
  - 若$p\leftrightarrow q$为**永真式**，称$p\equiv q$或$p\Leftrightarrow q$
- 对条件语句$p\rightarrow q$：
  - **反命题**：$\neg p\rightarrow\neg q$，即对输入取反，真值表上下颠倒
  - **逆命题**：$q\rightarrow p$，真值表上下颠倒
  - **逆否命题**：$\neg q\rightarrow \neg p$，和原命题**等价**
- 优先级：$\neg>\wedge>\vee>\rightarrow>\leftrightarrow$

### 与其它逻辑系统的对应

| 命题逻辑          | 逻辑代数 | 位运算符   | 集合论 |  自然语言描述  |
| :---------------: | :------: | :-------: | :----------------------: | :---------------: |
| $\neg p$          | $\bar p$ | $\rm NOT$ |                      | 否定/取反 |
| $\vee$            | $+$      | $\rm OR$  | $\cup$ | 析取/或/逻辑和/并 |
| $\wedge$          | $·$      | $\rm AND$ | $\cap$ | 合取/与/逻辑积/交 |
|$p \rightarrow q$ |          |           | | $p$蕴涵$q$ |
| $p\Rightarrow q$ |  |  || $p$永真蕴涵$q$/$p$是$q$的充分条件 |
| $p\leftrightarrow q$ | $\odot$ | $\rm XNOR$ |  | $p$当且仅当$q$/$p$同或$q$ |
| $p\Leftrightarrow q$ |  |  | | $p$是$q$的充要条件 |
|  | $|$ | $\rm NAND$ |  | 与非 |
|                   | $\downarrow$ | $\rm NOR$ |  | 或非 |

### 基本等价公式

- 满足交换律、结合律、分配律
- **德$\cdot$摩根律**：
  - $\neg(p\wedge q)\equiv \neg p\vee\neg q$
  - $\neg(p \vee q)\equiv\neg p\wedge\neg q$
- 支配律：$p\vee T\equiv T、p\wedge F\equiv F$
- 自等律：$p\vee p\equiv p、p\wedge p\equiv p$
- 恒等律：$p\vee F\equiv p、p\wedge T\equiv p$
- 否定律：$p\vee \neg p\equiv T、p\wedge \neg p\equiv F$
- **吸收律**：
  - $p\vee(p\wedge q)\equiv p$
  - 由分配律，$p\wedge(p\vee q)\equiv p$，也即上式的对偶
- **消解律**：
  - $(p\wedge q)\vee (\neg p\wedge r)\equiv q\wedge r$
  - 取对偶，$(p\vee q)\wedge(p\vee r)\equiv q\vee r$

### 条件命题的等价

- $p\rightarrow q\equiv \neg q\rightarrow\neg p\equiv \neg p\vee q$
- $(p\rightarrow q)\wedge(p\rightarrow r)\equiv p\rightarrow(q\wedge r)$
- $(p\rightarrow r)\wedge(q\rightarrow r)\equiv(p\vee q)\rightarrow r$
- $(p\rightarrow q)\vee(p\rightarrow r)\equiv p\rightarrow(q\vee r)$
- $(p\rightarrow r)\vee(q\rightarrow r)\equiv(p\wedge q)\rightarrow r$

## 逻辑函数

### 对偶规则

- 恒等式通常成对出现，称其为对偶，以$F^d$表示$F$的对偶函数，对一个恒等式两边取对偶不影响恒等性
- $F^d\not\equiv F(一般)、(F^d)^d\equiv F$
- 对偶函数由原函数**交换逻辑与、逻辑或，交换$0$和$1$得到**

### 反演规则

- 一个函数的反函数可以由原函数**交换逻辑与、逻辑或，交换$0$和$1$，同时原变量变反变量、反变量变原变量**而得，其中反变量特指单个变量上的非号，跨两个及以上变量的非号不能动
- 同样可借$\bar F$来求$F$

## 谓词逻辑

### 命题逻辑的局限

命题逻辑的最小单元是命题，所以命题逻辑无法表示那些不真不假的语句，因为它们没有指定变量的值

因此也无法从若干简单命题中得到新命题，例如亚里士多德三段论：

- 三个原子命题：$p:$所有人都要死、$q:$苏格拉底是人、$r:$所以苏格拉底要死
- 根据逻辑推理，由$p$且$q$一定可以得到$r$，即要证明$(p\wedge q)\Rightarrow r$
- 尽管$p\wedge q=T$，在命题逻辑的系统里，仍然可以把$F$**指派**给$r$
- 换句话说就是，给命题赋予真值的同时，也**掩盖掉了具体的信息**，你只知道$p$是对的，$q$是对的，然后无法得知任何其它信息，也就无法得知$r$的真值
- 如果能变成假言三段论的形式，是可以通过命题逻辑表示的，但原子命题已经是最小的命题了

谓词逻辑解决这样的事情：提取出命题中的信息，并提供一套符号用于产生新的、适用更广的命题

### 谓词与量词

通过谓词可以从这些语句中生成命题，谓词逻辑允许对命题进行更细的划分

- 个体：命题所描述的对象，约定用小写字母表示
- 谓词：命题赋予主体的属性或动作，约定用大写字母$P、Q、R$表示
- 命题的谓词形式：由**谓词**和若干**个体常元**组合成$P(a_1,a_2,\cdots,a_n)$的形式
  - 称$P(x_1,x_2,\cdots,x_n)$为一个$n$元谓词，其中$x$为个体变元，称它能够表示的所有个体的集合为它的个体域
- 量词：通过量词来限定个体域在谓词中的不确定性，例如***所有、至少***等
  - 全称量词$(\forall x)$：表达**所有**
  - 存在量词$(\exist x)$：表达**存在**
  - 存在唯一量词$(\exist\ !\ x)$：表达**存在唯一**
  - 后接谓词$P(x)$表示该量词的辖域

### 谓词逻辑公式

- 德$·$摩根律：
  - $\neg(\exist x)P(x)\equiv (\forall x)\neg P(x)$
  - $\neg (\forall x)P(x)\equiv(\exist x)\neg P(x)$
- 量词的辖域扩张：
  - $(\forall x)P(x)\vee Q\equiv (\forall x)(P(x)\vee Q)$，其中$Q$**不含变元**
  - 同理，将析取换为合取、将全称量词换位存在量词也成立
  - $(\forall x)(P(x)\rightarrow Q)\equiv(\exist x)P(x)\rightarrow Q$，将$\forall$和$\exist$互换同理
    - 证明：左式$=(\forall x)(\neg P(x)\vee Q)=(\forall x)\neg P(x)\vee Q=\neg(\exist x)P(x)\vee Q=$右式
  - $(\forall x)(Q\rightarrow P(x))\equiv Q\rightarrow(\forall x)P(x)$，将$\forall$换成$\exist$同理
- 量词的分配律：
  - $\forall x$对$\wedge$、$\exist x$对$\vee$满足分配律
  - $\forall x$对$\vee$、$\exist x$对$\wedge$**不满足**分配律，只满足$(\forall x)P(x)\vee (\forall x)Q(x)\Rightarrow(\forall x)(P(x)\vee Q(x))\\(\exist x)(P(x)\wedge Q(x))\Rightarrow(\exist x)P(x)\wedge (\exist x)Q(x)$
- 永真蕴涵式：$(\exist x)P(x)\rightarrow(\forall x)Q(x)\Rightarrow(\forall x)(P(x)\rightarrow Q(x))\Rightarrow (\forall x)P(x)\rightarrow(\forall x)Q(x)$

### 嵌套量词

- 嵌套量词的顺序：
  - $(\forall x)(\forall y)\equiv (\forall y)(\forall x)\\(\exist x)(\exist y)\equiv (\exist y)(\exist x)$
  - $(\forall x)(\forall y)\Rightarrow(\exist x)(\forall y)\Rightarrow(\forall y)(\exist x)\Rightarrow(\exist x)(\exist y)\\(\forall x)(\forall y)\Rightarrow(\exist y)(\forall x)\Rightarrow(\forall x)(\exist y)\Rightarrow(\exist x)(\exist y)$
- 嵌套量词的否定：
  - 层层套用德$·$摩根律，结果为$\forall$和$\exist$互换，非号在最里层，例如：
  - $\neg(\forall x)(\exist y)P(x,y)\equiv(\exist x)(\forall y)\neg P(x,y)$

## 推理规则

### 命题逻辑规则

- $P$：引入一个前件
- $T$：引用公式$S$
- $CP$：$P\wedge Q\rightarrow R\equiv P\rightarrow(Q\rightarrow R)$
- 假言推理：表示为永真式$(p\wedge(p\rightarrow q))\rightarrow q$
- 假言三段论：由一个大条件和一个小条件可得出，表示为永真式$((p\rightarrow q)\wedge(q\rightarrow r))\rightarrow (p\rightarrow r)$
- 利用永真式化简命题，其实就是在使用规则$T$

### 谓词逻辑规则

- 谓词逻辑是命题逻辑的推广，同样可使用命题逻辑的推理规则
- 全称特指$(\rm US)$：$(\forall x)P(x)\Rightarrow P(y)$
- 存在特指$(\rm ES)$：$(\exist x)P(x)\Rightarrow P(a)$
- 存在推广$(\rm EG)$：$P(y)\Rightarrow(\exist x)P(x)$，$y$属于$x$的个体域
- 全称推广$(\rm UG)$：$P(y)\Rightarrow(\forall x)P(x)$，$x$属于$y$的个体域
