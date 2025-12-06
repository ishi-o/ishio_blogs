---
title: "离散数学: 近世代数（群、环、域）"
date: 2024-09-27
categories: [SE Courses, Discrete Math]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 近世代数

## 群

### 半群定义

- 群和半群是特殊的代数，都只含一个非空集合和一个二元运算，因此元素运算时可省略运算符号
  给定代数$<S,\circ>$：
- 半群：若$\circ$可结合，称该代数为半群
- 可交换半群：在半群基础上，$\circ$可交换
- 独异点/含幺半群/拟半群：在半群基础上，$\circ$有幺元
- 子半群：半群的所有子代数都是半群，称为前者的子半群
- 循环半群：给定半群$M,\circ$，元素$a\in M$，由$a$自身进行$n$次运算得到的集合为$S:\{a^k|k\in Z\}$，称：
  - $a$生成了循环半群$<S,\circ>$，$a$为生成元
  - $<S,\circ>$为$<M,\circ>$的循环子半群
- 生成集：给定集合$G\subseteq S$，若所有$S$的元素可由$G$中元素及其相互运算后得到，则称$G$为生成集
- 积半群：两个半群的积代数为积半群

### 群的性质

- 群：若**独异点**中的每个元素**都有逆元**，称该独异点为群
- 基本性质：
  - 封闭性：群作为一个代数拥有的性质
  - 结合律：群作为一个半群拥有的性质
  - 存在单位元：群作为一个独异点拥有的性质
  - 所有元素作为逆元：群拥有的性质
- 幂运算规则：
  - $a^n=\begin{cases}e,&n=0\\aa^{n-1},&n>0\\(a^{-1})^{-n},&n<0\end{cases}$
  - $(ab)^{-1}=b^{-1}a^{-1}$，利用可约律证
- 每个非零元素都是可约元，因此方程有唯一解
- 元素的阶：
  - 使$a^r=e$的最小的正整数$r$，记为$|a|=r$
  - 若存在$a^k=e$，则必有$k\equiv0(\rm{mod}\ r)$
  - $|a^{-1}|=|a|$
- $\rm Abel$群：即可交换群，满足$(ab)^n=a^nb^n$；证明一个群为可交换群最简单的方法为证明$abab=a^2b^2$

### 子群

- 若群$G$的非空子集$H$关于$G$的运算构成群，称其为子群，根据集合关系有真子群之分，通过$\le$、$<$说明其关系
- 判定方法：
  - $\forall a\forall b(a,b\in H\rightarrow ab\in H\wedge a^{-1}\in H)$，即同时满足子代数、所有元素有逆元的性质
    - 只需证明含幺元，$aa^{-1}=e\in H$
  - $\forall a\forall b(a,b\in H\rightarrow ab^{-1}\in H)$
    - $\begin{align}(a,b\in H\rightarrow ab^{-1}\in H)\Rightarrow(a,a)\rightarrow e\in H\Rightarrow (e,a)\rightarrow a^{-1}\Rightarrow(a,b^{-1})\rightarrow ab\in H\end{align}$
  - $H$**有穷**，且$\forall a\forall b(a,b\in H\rightarrow ab\in H)$
    - $\begin{align}有穷\Rightarrow(\exist k)a^k=e\in H\wedge aa^{k-1}=e\in H\Rightarrow a^{-1}=a^{k-1}\in H\end{align}$
- 生成的子群：设$a\in G$，则由$a$生成的群$\{a^k|k\in Z\}$是$G$的子群，记作$<a>$
- 中心子群：称$\{a\in G\ |\ \forall x(x\in G\wedge ax=xa)\}$为$G$的中心
- 子群的并和交：若$H,K\le G$
  - $H\cap K\le G$
  - $H\cup K\Leftrightarrow (H\subseteq K)\vee (K\subseteq H)$
- 子群格：偏序集$<\{H\ |\ H\le G\},\subseteq>$为$G$的子群格
- 陪集：$H$为$G$的子群，左陪集定义为$aH=\{a\circ h\ |\ h\in H\}$，其中$a\in G$
  - 陪集相当于$H$在$G$中的扩充，陪集通常**不一定是$G$的子群**
  - 存在陪集$eH=H$
  - 所有陪集的阶相等，等于$|H|$
  - 若左陪集和右陪集总相等，称该子群为正规子群

- 左陪集关系：$C_H=\{<a,b>\ |\ a,b\in G\wedge b^{-1}a\in H\}$
  - 陪集关系是一种等价关系
  - $aH=[a]_{C_H}$，即$G$中任意元素在$H$上的陪集等于该元素所属的等价类
    - $\begin{align}&充分性:\forall h\in H,a、ah\in aH;令b=ah,即h=ba^{-1}\Rightarrow a,b\in G\wedge ba^{-1}\in H\\&必要性:a,b\in G\wedge ba^{-1}=h\in H\Rightarrow b=ah\in aH\end{align}$

  - 意义：在找出任意一个子群后，可通过陪集(扩充)来找到其余的所有陪集，而由陪集关系导出的等价类和陪集的对应关系，可得**$H$的不相同陪集个数等于总等价类个数**

  - 所有陪集的并集等于$G$，任意两个不相等陪集完全不相交

- 拉格朗日定理：**有限群**的阶一定能被任意子群$H$的阶整除，且商等于等价类的个数
  - $存在a是k阶元\Leftrightarrow k整除|G|$

### 循环群

- 若$G$中任意元素能由$a\in G$的任意次幂表示，则称$G=<a>$，$a$为生成元
- 找出循环群$G$的生成元：
  - 若$G$为无限循环群，则只有$a,a^{-1}$是生成元
  - 若$G$为$n$阶循环群，则$a^k$是生成元，其中$k$小于$n$且与$n$互素，因此共有$\varphi(n)$个生成元
- 循环群的子群仍是循环群，无限循环群的子群除平凡群外都是无限循环群
- $n$阶循环群$G=<a>$的子群：若$n$有$k$个正因子，则恰有$k$个子群，每个正因子$d$唯一对应一个$d$阶子群，且$\begin{align}a^{\large\frac{|G|}{d}}\end{align}$为该子群的生成元

## 环、域、格

### 环

- 具有两个二元运算的群，通常定义为$<R,+,\times>$，其中：
  - $<R,+>$构成$\rm Abel$群
  - $<R,\times>$构成乘法半群
  - $\times$对$+$满足分配律
  - 称**加法幺元**为**环的零元**
- 可交换环、含幺环、无零因子环：乘法半群满足交换律、含幺元、若两数乘积为零元则必有一项为零元
- 布尔环：乘法半群满足等幂律
- 整环：同时满足可交换环、含幺环、无零因子环的性质

### 域

- 域的定义：
  - $R$是整环的基础上，乘法群中任意元素有其逆元
  - $R$是环的基础上，乘法群是$\rm Able$群

### 格

- 偏序格：任意有两元素的子集都在整个集合中存在最小上界和最大下界的偏序集
- 代数格：代数$<L,\circ,\times>$，且$\circ,\times$满足交换律、结合律、等幂律、吸收律
  - 其中，定义每个元素的$\circ$运算为其最大下界，$\times$运算为其最小上界
- 偏序格和代数格等价
- 完全格：每个非空子集均有上下确界
- 有界格：有上下确界，完全格一定是有界格
- 分配格：两运算满足分配律
- 有补格(有余格)：对任意两元素的最小上界等于整个格的上确界，最大下界等于整个格的下确界

[离散数学（格与布尔代数） - gonghr - 博客园 (cnblogs.com)](https://www.cnblogs.com/gonghr/p/15474256.html#偏序格)
