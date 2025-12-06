---
title: "离散数学: 集合论"
date: 2024-04-22
categories: [SE Courses, Discrete Math]
tags: [math, set]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 集合论

## 约定

- 小写字母表示元素，大写字母表示集合
- 元素可指代变元、集合、元组等**任何离散结构**

## 集合基本概念

### 集合表示

离散结构大厦的地基！

- 集合是若干元素的**无序聚集**，且元素不重复
- 集合构造法：$V=\{x\ |\ P(x),Q(x),\cdots\}$，其中谓词$P,Q$表示$x$的性质
- 属于：$x\in V$表示一个**元素**属于一个**集合**，在构造法限定元素所属父集时可写在竖线左方
- 包含关系：
  - $V_1\subseteq V_2$表示集合**$V_1$包含于$V_2$**，或**$V_2$包含$V_1$**，称$V_1$是$V_2$的**子集**，与$V_2\supseteq V_1$等价
  - **逻辑表示**：$V_1\subseteq V_2\Leftrightarrow\forall x(x\in V_1\rightarrow x\in V_2)$
  - 所有集合都包含空集和它本身
  - 包含是严格区分离散结构的，变元即变元，集合即集合，例如$\{1\}\nsubseteq\{\{1\}\}$
- 相等：
  - 若$V_1\subseteq V_2$且$V_2\subseteq V_1$，称$V_1=V_2$
  - 逻辑表示：$V_1=V_2\Leftrightarrow\forall x(x\in V_1\leftrightarrow x\in V_2)$
- 基数：$|V|$表示集合的基数，即集合中元素的个数
- 常用集合约定记号：
  - $N$：所有自然数的集合
  - $Z$：所有整数的集合
  - $Z^+$：所有正整数的集合，和$\{x\in N\ |\ x\ne0\}$相等
  - $Q$：$\{p/q\ |\ p\in Z\wedge q\in Z\wedge q\ne0\}$，即所有有理数的集合
  - $R$：所有实数的集合
  - $R^+$：所有正实数的集合
  - $C$：所有复数的集合

### 集合分类及特殊集合

- 元素个数分类：有限集、无限集
- 空集：$\varnothing$，表示不含任何元素的集合
  - 空集不显式写在集合内，但若出现在集合内，证明是一个结构是集合的**元素**
  - 区分：空集单独作为集合写作$\varnothing$，作为其它集合的一个元素写作$\{...,\varnothing,...\}$
  - 逻辑表示：$\{x\ |\ P(x)\wedge\neg P(x)\}$
- **幂集**：$\mathcal P(V)$表示集合$V$的幂集，是$V$**所有子集的集合**
  - $|\mathcal P(V)|=2^{|V|}$，若一个集合每取一次幂集，基数就取一次$2$的幂
  - 所有集合一定包含空集和本身，而$\varnothing=\varnothing$，故$\mathcal P(\varnothing)=\{\varnothing\}$
  - 空集作为元素的边缘情况：$\mathcal P(\mathcal P(\varnothing))=\mathcal P(\{\varnothing\})=\{\varnothing,\{\varnothing\}\}$

### 集合运算

- 集合运算的结果也是集合
- 并集：$V_1\cup V_2=\{x\ |\ x\in V_1\vee x\in V_2\}$
- 交集：$V_1\cap V_2=\{x\ |\ x\in V_1\wedge x\in V_2\}$，$V_1\cap V_2=\varnothing\Leftrightarrow V_1和V_2不相交$
- 相对补集：$V_1-V_2=\{x\ |\ x\in V_1\wedge x\notin V_2\}$，称为$V_2$相对于$V_1$的补集，也称$V_1$和$V_2$的差集
- 对称差集：$V_1\oplus V_2=(V_1-V_2)\cup(V_2-V_1)$，即$V_1\cup V_2-V_1\cap V_2$
- 绝对补集：
  - 通常假定所有集合都包含于一个称为**全集**$U$的大集合，通常所说$V$的补集就是$V$相对于$U$的补集，记为$\sim V$或$\bar{V}$
  - $x\in\bar V\Leftrightarrow x\notin V$
- 集合的并集、交集运算满足布尔和、布尔积的各种恒等式，其中空集$\varnothing$对应$F$，全集$U$对应$T$

### 恒包含及补集恒等式

- $V_1\cap V_2\subseteq\ \begin{matrix}V_1\\V_2\end{matrix}\subseteq V_1\cup V_2$
- $V_1-V_2=V_1\cap\bar {V_2}=V_1-V_1\cap V_2\subseteq V_1$，相对补集满足吸收律，且最终包含于前集合
- $V_1\cap V_2-V_1\cap V_3=V_1\cap(V_2-V_3)$，即相对补集和交集满足分配律
- $V_1\cup V_2-V_1\cup V_3\subseteq V_1\cup(V_2-V_3)$
- $V_1\oplus V_1=\varnothing$，自身对称补集为空集
- $V_1\subseteq V_2\Leftrightarrow V_1-V_2=\varnothing$

## 二元关系

### 元组

- 单纯的集合结构聚集的元素是无序的，无法表示元素的聚集顺序，故通过元组来反映**次序**
- 元组表示：用圆括号或尖括号括起具有固定排列顺序的元素，称为一个元组
- 二元组：含有两个元素的元组，也称**序偶**，第一个元素称为第一元素
- 元组相等：两个元组的所有元素对应相等

### 笛卡尔积

笛卡尔积**从多个集合**中构造出能够**反映聚集顺序**的**元素全为元组**的**新集合**，不满足交换律
$$
V_1\times V_2\times\cdots\times V_n=\{(x_1,x_2,\cdots,x_n)\ |\ x_1\in V_1\wedge x_2\in V_2\wedge\cdots\wedge x_n\in V_n\}
$$
二元笛卡尔积即构造出以$V_1$元素为第一元素，以$V_2$元素为第二元素的所有序偶，并放在一个新集合里

边缘情况：若其中一个参数集合是空集，那么新集合为空集

不难得到，$\begin{align}|V_1\times V_2\times\cdots\times V_n|=\prod_{1\le i\le n}|V_i|\end{align}$

## 映射函数
