---
title: "离散数学: 代数系统"
date: 2024-08-07
categories: [SE Courses, Discrete Math]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 代数系统

## 定义

- $S$上的$n$元运算是一个特殊的函数$f:S^n\rightarrow S$，具有**封闭性**
- 代数系统：由非空集合$S$和$S$上的运算组成的系统，记作$<S,f_1,f_2,\cdots,f_n>$，简称代数
- 代数的基数等于$S$的基数(因为运算具有封闭性)
- 同类型的代数：要求一一对应的各个运算的元数相同，和$S$无关
- 子代数：给定代数$<S,f_1,\cdots,f_n>$，若非空集合$T\subseteq S$，且$T$和$f_1,\cdots,f_n$能够组成代数(即$f$在$T$上封闭)，则$<T,f_1,\cdots,f_n>\,\subseteq\,<S,f_1,\cdots,f_n>$

## 代数运算

### 代数运算的性质

设在$S$上的二元运算$\circ$满足：

- **结合律**：$\forall x\forall y\forall z(x,y,z\in S\rightarrow (x\circ y)\circ z=x\circ(y\circ z))$
- **交换律**：$\forall x\forall y(x,y\in S\rightarrow x\circ y=y\circ x)$
  - 若同时满足结合律和交换律，则表达式可任意调换次序
- **分配律**：设$\circ$对$S$上的另一个运算$\times$满足：
  - 左分配律：$\forall x\forall y\forall z(x,y,z\in S\rightarrow x\circ(y\times z)=(x\circ y)\times (x\circ z))$
  - 右分配律：$\forall x\forall y\forall z(x,y,z\in S\rightarrow (x\times y)\circ z=(x\circ z)\times (y\circ z))$
  - 注意运算的对应关系，$\circ$对$\times$满足，则左式$\circ$在括号外
  - 同时满足左右分配律，才能称$\circ$对$\times$满足分配律
  - 若$\circ$满足交换律，且对$\times$满足左或右分配律，则可证明它对$\times$满足分配律
- **吸收律**：类似于分配律的左右定义方式，$\circ$对$\times$满足**左吸收律**$\Rightarrow\forall x\forall y(x,y\in S\rightarrow x\circ(x\times y)=x)$
- **幺元**(单位元)：有左右幺元之分，若$e$是关于$\circ$的左幺元，则$\forall x(x\in S\rightarrow e\circ x=x)$
- **零元**：有左右零元之分，若$\O$是关于$\circ$的左零元，则$\forall x(x\in S\rightarrow \O\circ x=\O)$
  - 如果一个运算的左右幺元/零元存在，则左右分别相等，且这个幺元/零元是**唯一**的
  - 如果一个运算同时存在幺元和零元，则它们不相等
- **等幂律**：$\forall x(x\in S\rightarrow x\circ x=x)$，并定义$x$为$\circ$的**等幂元**
  - 若满足，则所有元素都为等幂元
  - 一个运算的幺元和零元一定是等幂元
- **逆元**：若$e$为$\circ$的幺元，有左右逆元之分，且有两种对应关系
  - $x$是关于$\circ$的左逆元：$\exist y(y\in S\wedge x\circ y=e)$
  - 称存在的这个$y$为$x$的右逆元，即$x\circ y=e$
  - 如果$x$和$y$互逆($x$为$y$的左右逆元)，则$x$和$y$都是关于$\circ$的逆元
  - 一个运算的幺元一定是逆元
- **可约律**：有左右可约律，若$\O$为$\circ$的零元，则$\circ$是左可约的$\Rightarrow\forall x\forall y\forall z((x,y,z\in S\wedge x\ne\O\wedge x\circ y=x\circ z)\rightarrow y=z)$
  - 定义$x$为$\circ$的**左可约元**
  - 如果一个运算是可约的，那么所有非零元的元素都为可约元
  - 存在一个可约元，不代表一个运算是可约的
  - ！如果$\circ$满足结合律，同时$x\ne\O$且$x$是$\circ$的逆元，则$x$是$\circ$的可约元
  - 一个运算的幺元一定是可约元


### 性质在运算表上的体现

将二元运算的左元、右元分别作为行、列，将运算结果作为表中数据，生成的运算表(方阵)中：

- 封闭性：表中数据一定都属于$S$，否则该运算不是$S$上的运算
- 可交换：该方阵是对称的
- 等幂：主对角线上元素和行头元素相等
- 左/右零元：左/右零元所在行/列元素和该元素相等
- 左/右幺元：左/右幺元所在行/列元素和对应的列/行元素相等
- 左/右逆元：左/右逆元所在行/列元素中必定有一个幺元
- ！两元素互为逆元：两元素所在行和列的元素都是幺元

以上冒号左右两边的描述是等价的

## 常见代数运算的性质

<table align="center" border="2">
    <tbody align="center">
        <tr>
            <th>集合</th>
            <th>运算</th>
            <th>幺元</th>
            <th>零元</th>
            <th>等幂元</th>
            <th>逆元</th>
            <th>可约元</th>
        </tr>
        <tr>
            <td rowspan="2" style="display:table-cell; vertical-align:middle">R</td>
            <td>加法</td>
            <td>0</td>
            <td>无</td>
            <td>0</td>
            <td>相反数</td>
            <td>任意元素</td>
        </tr>
        <tr>
            <td style="display:table-cell; vertical-align:middle">乘法</td>
            <td style="display:table-cell; vertical-align:middle">1</td>
            <td style="display:table-cell; vertical-align:middle">0</td>
            <td style="display:table-cell; vertical-align:middle">1和0</td>
            <td>除0外的<br>数的倒数</td>
            <td>除0外的<br>任意元素</td>
        </tr>
        <tr>
            <td rowspan="2" style="display:table-cell; vertical-align:middle">P(S)</td>
            <td>并集</td>
            <td>空集</td>
            <td>S</td>
            <td>任意元素</td>
            <td>空集</td>
            <td>空集</td>
        </tr>
        <tr>
            <td>交集</td>
            <td>S</td>
            <td>空集</td>
            <td>任意元素</td>
            <td>S</td>
            <td>S</td>
        </tr>
        <tr>
            <td rowspan="2" style="display:table-cell; vertical-align:middle">命题</td>
            <td>析取</td>
            <td>F</td>
            <td>T</td>
            <td>任意元素</td>
            <td>F</td>
            <td>F</td>
        </tr>
        <tr>
            <td>合取</td>
            <td>T</td>
            <td>F</td>
            <td>任意元素</td>
            <td>T</td>
            <td>T</td>
        </tr>
        <tr>
            <td>集合上的<br>所有关系<br>构成集合</td>
            <td style="display:table-cell; vertical-align:middle">合成</td>
            <td style="display:table-cell; vertical-align:middle">恒等关系</td>
            <td style="display:table-cell; vertical-align:middle">空关系</td>
            <td style="display:table-cell; vertical-align:middle">恒等关系</td>
            <td style="display:table-cell; vertical-align:middle">关系矩阵可逆<br>的所有关系</td>
            <td style="display:table-cell; vertical-align:middle">关系矩阵可逆<br>的所有关系</td>
        </tr>
        <tr>
            <td>f:S->S<br>所有双射函数<br>构成的集合</td>
            <td style="display:table-cell; vertical-align:middle">合成</td>
            <td style="display:table-cell; vertical-align:middle">恒等函数</td>
            <td style="display:table-cell; vertical-align:middle">无</td>
            <td style="display:table-cell; vertical-align:middle">恒等函数</td>
            <td style="display:table-cell; vertical-align:middle">所有元素</td>
            <td style="display:table-cell; vertical-align:middle">所有元素</td>
        </tr>
    </tbody>
</table>

最后两条：

- 一个集合上的所有双射函数都有其对应的反函数，且反函数也是该集合上的双射函数
- 一个集合上的双射函数和其反函数合成的结果是恒等函数
- 一个集合上的双射函数构成集合，是所有关系构成集合的子集

## 同类型的代数

以下关系，要求$<S,\circ>$和$<T,\times>$是同类型的

### 同态关系

- 记号：记$<S,\circ>\simeq<T,\times>$表示前者同态于后者
- 定义：上式$\Rightarrow\exist f(f\in T^S\wedge\forall x\forall y(x,y\in S\rightarrow f(x\circ y)=f(x)\times f(y)))$，其中称$f:S\rightarrow T$为$<S,\circ>$到$<T,\times>$的同态映射
  - 即先通过运算映射再通过$f$映射等于先通过$f$映射再通过运算映射
  - $f$不唯一
  - 在代数有多个二元运算时，要求$f$是同一个
  - 由$f$的值域$R(f)\subseteq T$，易得$<R(f),\times>\subseteq <T,\times>$

### 同态映射的分类

根据$f$的性质，对该映射进行分类：

- $f$为满射$\Rightarrow$从前者到后者的满同态映射
- $f$为单射$\Rightarrow$从前者到后者的单一同态映射
- $f$为双射$\Rightarrow$从前者到后者的同构映射

### 满同态映射的运算继承性质

只要$f$是满射的(包括同构关系)，就有以下性质：

- 两代数的对应运算的结合、交换、分配、等幂律等性质是一致的
- 前代数中运算的幺元、零元、逆元经$f$后，等于后代数中对应运算的幺元、零元、逆元

### 同构关系

- 记号：$<S,\circ>\cong<T,\times>\Rightarrow$前者同构于后者
- 同构意味着，每个$f(x)$有其唯一对应的原像，因此两个代数可以**互相映射**，即同时存在同构关系$<T,\times>\cong_{f^{-1}}<S,\circ>$；而不是像同态一样，只能单向地映射系统
- 代数间的同构关系是等价关系
  - 可通过同构关系，将所有代数形成的集合按其分类
  - 同态、同构关系指的不是$f$这个函数关系，而是由两代数作为第一、二元素的二元关系
- 称从$<S,\circ>$到自身的同态、同构关系为自同态、自同构关系

## 代数间的运算

### 同余关系

- 定义：若$E$是$S$上的等价关系，且满足$\forall x_1\forall y_1\forall x_2\forall y_2((x_1,y_1,x_2,y_2\in S\wedge x_1Ey_1\wedge x_2Ey_2)\rightarrow(x_1\circ x_2)E(y_1\circ y_2))$，则称$E$是$<S,\circ>$上的同余关系
  - 称由$E$划分的等价类为同余类
  - $E$是**特殊的等价关系**，能保证元素运算后，仍能满足该关系
  - $E$的这种代换需要满足代数中的**所有运算**
  - 同余关系和同态、同构不是一种概念
- 另一种表达：$[x_1]_E=[y_1]_E\wedge[x_2]_E=[y_2]_E\rightarrow[x_1\circ x_2]_E=[y_1\circ y_2]_E$

### 商代数

- 若$f$是$<S,\circ>$到$<T,\times>$的同态映射，则$S$上的关系$E_f$(满足$xE_fy:f(x)=f(y)$)一定是同余关系

  - 证明：任取$x_1,y_1,x_2,y_2\in S$，若有$x_1E_fy_1\wedge x_2E_fy_2$，则$f(x_1)=f(y_1)\wedge f(x_2)=f(y_2)$；根据同态映射的定义，有$\begin{cases}f(x_1)\times f(x_2)=f(x_1\circ x_2)\\f(y_1)\times f(y_2)=f(y_1\circ y_2)\end{cases}$；三式联立得$f(x_1\circ x_2)=f(y_1\circ y_2)$，即$(x_1\circ x_2)E_f(y_1\circ y_2)$，得证
  - 称$E_f$为由$f$诱导的同余关系

- 商代数：给定$<S,\circ>$及其上的一个同余关系$E$：

  - 以$E$导出的商集$S/E$为集合，构造满足同态定义的运算$\times$：$\forall x\forall y\in S\rightarrow [x\circ y]_E=[x]_E\times [y]_E$，则称$<S/E,\times>$为它的商代数
  - 商代数是人为构造的，满足$<S,\circ>\simeq<S/E,\times>$，$g_E(x)=[x]_E$
  - 易得$g_E$为**满同态映射**，因此商代数的运算继承了本代数运算的性质
  - 该映射被称为**正则映射**、**自然同态**

- 设$<S,\circ>\simeq<T,\times>$，且$f$为**满同态映射**，则$<T,\times>\cong<S/E_f,*>$(后者为$<S,\circ>$的商代数)

  - 证明：构造函数$\begin{cases}h:T\rightarrow S/E_f\\h(y)=[x]_E\end{cases}$，其中$y=f(x)$

  - 证其为同态映射

    $h(y_1\times y_2)=h(f(x_1\circ x_2))=[x_1\circ x_2]_E$，由**商集定义**，上式$=[x_1]_E*[x_2]_E$

    $\Rightarrow\forall y_1\forall y_2(y_1,y_2\in T\rightarrow [x_1]_E*[x_2]_E=h(y_1\times y_2))$

    $\Rightarrow <T,\times>\simeq<S/E_f,*>$，$h$为其同态映射

  - 证其为满射：$f、g_{E_f}$为满射$\Rightarrow h$为满射

  - 证其为单射：若$h(y_1)=h(y_2)$

    $\Rightarrow [x_1]_E=[x_2]_E\Rightarrow x_1E_fx_2\Rightarrow f(x_1)=f(x_2)\Rightarrow y_1=y_2$

    得证


### 积代数

设有同类型的代数$<S,\circ>$和$<T,\times>$，则定义运算$\otimes$：$\forall x_1\forall x_2\forall y_1\forall y_2(x_1,x_2\in S\wedge y_1,y_2\in T\rightarrow <x_1\circ x_2,y_1\times y_2>=<x_1,y_1>\otimes<x_2,y_2>)$

称$<S\times T,\otimes>$为上述两个代数的**积代数**；上述两个代数为该积代数的**因子代数**

- 上述为两因子代数形成的积代数，多因子代数形成的积代数可递归定义
- 积代数和因子代数是同类型的
- 积代数的运算包含了所有因子代数的运算及其笛卡尔积顺序的信息

