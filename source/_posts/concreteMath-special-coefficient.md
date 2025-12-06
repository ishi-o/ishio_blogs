---
title: "具体数学: 特殊系数"
date: 2024-11-01
categories: [SE Courses, Concrete Math]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 二项式系数

- 定义：$\begin{align}(x+y)^n=\sum_{k=0}^n\left(\begin{matrix}n\\k\end{matrix}\right)x^ky^{n-k}\end{align}$

- 下指数一定为整数，当其为负数时，二项式系数为零，否则：

  - 当上指数为整数时，$\begin{align}\left(\begin{matrix}n\\k\end{matrix}\right)=\frac{n!}{k!(n-k)!}\end{align}$，具有组合意义
  - 上指数可推广为实数，此时$\begin{align}\left(\begin{matrix}n\\k\end{matrix}\right)=\frac{n^{\underline k}}{k!}\end{align}$，可视为$n$的$k$次多项式($(1+r)^k$泰勒展开的系数)

- 当上指数为**正整数**时：

  - $\begin{align}\sum_{k=0}^n\left(\begin{matrix}n\\k\end{matrix}\right)=2^n\end{align}$、$\begin{align}\sum_{k=0}^n(-1)^k\left(\begin{matrix}n\\k\end{matrix}\right)=0\end{align}$，均可通过定义证明
  - 下指数任意，$\begin{align}\left(\begin{matrix}n\\k\end{matrix}\right)=\left(\begin{matrix}n\\n-k\end{matrix}\right)\end{align}$

- 当上指数为**任意实数**时：

  - 下指数不为零时，$\begin{align}\left(\begin{matrix}n\\k\end{matrix}\right)=\frac nk\left(\begin{matrix}n-1\\k-1\end{matrix}\right)\end{align}$
  - $\begin{align}\left(\begin{matrix}n\\k\end{matrix}\right)=\left(\begin{matrix}n-1\\k\end{matrix}\right)+\left(\begin{matrix}n-1\\k-1\end{matrix}\right)\end{align}$

- 特殊值：

  - $\left(\begin{matrix}n\\0\end{matrix}\right)=1$
  - $\left(\begin{matrix}n\\1\end{matrix}\right)=n$
  - $\left(\begin{matrix}n\\n\end{matrix}\right)=1$

- 序列构造母函数：$<a_0,\cdots,a_n>$的母函数为$A(x)=a_0+a_1x+a_2x^2+\cdots+a_nx^n$

- 母函数乘积的系数：若$C(x)=A(x)B(x)$，则$\begin{align}c_n=\sum_{k=0}^na_kb_{n-k}\end{align}$

  - 证明范德蒙德卷积：由$(1+x)^{a+b}=(1+x)^a(1+x)^b$，设为$C(x)=A(x)B(x)$
    由$\begin{align}&C(x)=\sum_{k=0}^n\left(\begin{matrix}a+b\\k\end{matrix}\right)x^k\\&A(x)=\sum_{k=0}^n\left(\begin{matrix}a\\k\end{matrix}\right)x^k\\&B(x)=\sum_{k=0}^n\left(\begin{matrix}b\\k\end{matrix}\right)x^k\end{align}$

    得$\begin{align}c_n=\left(\begin{matrix}a+b\\n\end{matrix}\right)=\sum_{k=0}^n\left(\begin{matrix}a\\k\end{matrix}\right)\left(\begin{matrix}b\\n-k\end{matrix}\right)\end{align}$

- 常见母函数：

  - $\begin{align}(1+x)^n=\sum_{k=0}^n\left(\begin{matrix}n\\k\end{matrix}\right)x^k\end{align}$
  - $\begin{align}\frac1{1-x}=\sum_{k\ge0}x^k、\frac1{1+x}=\sum_{k\ge0}(-1)^kx^k\end{align}$
  - $\begin{align}\frac1{(1-x)^{n+1}}=\sum_{k\ge0}\left(\begin{matrix}n+k\\k\end{matrix}\right)x^k\end{align}$
  - $\begin{align}\frac{x^n}{(1-x)^{n+1}}=\sum_{k\ge0}\left(\begin{matrix}k\\n\end{matrix}\right)x^k\end{align}$，通过上式移项换元得到

## 第二类斯特林数

- 定义：$\begin{align}x^{n}=\sum_{k}\left\{\begin{matrix}n\\k\end{matrix}\right\}x^{\underline k}\end{align}$，组合意义为将$n$个元素分为$k$个无区别的非空集合
- 递推式：$\begin{align}\left\{\begin{matrix}n\\k\end{matrix}\right\}=k\left\{\begin{matrix}n-1\\k\end{matrix}\right\}+\left\{\begin{matrix}n-1\\k-1\end{matrix}\right\}\end{align}$
  组合意义：将第$n$个元素分到$k$个原有盒子的方案为第一项，将它单独作为一个集合的方案数为第二项
- 特殊值：
  - $\left\{\begin{matrix}n\\0\end{matrix}\right\}=\begin{cases}1,&n=0\\0,&n\ne0\end{cases}$
  - $\left\{\begin{matrix}n\\1\end{matrix}\right\}=1$
  - $\begin{align}\left\{\begin{matrix}n\\2\end{matrix}\right\}=\frac{\sum_\limits{k=1}^{n-1}\left(\begin{matrix}n\\k\end{matrix}\right)}2=2^{n-1}-1\end{align}$
  - $\left\{\begin{matrix}n\\n-1\end{matrix}\right\}=\left(\begin{matrix}n\\2\end{matrix}\right)=\frac{n(n-1)}2$
  - $\left\{\begin{matrix}n\\n\end{matrix}\right\}=1$

## 第一类斯特林数

- 定义：$x^{\overline n}=\sum_{k\ge0}\left[\begin{matrix}n\\k\end{matrix}\right]x^k$，组合意义为将$n$个元素分为$k$个**轮换**

- 递推式：$\left[\begin{matrix}n\\k\end{matrix}\right]=(n-1)\left[\begin{matrix}n-1\\k\end{matrix}\right]+\left[\begin{matrix}n-1\\k-1\end{matrix}\right]$

  组合意义为将第$n$个元素插入到已有划分的$n-1$个位置上(第一项)，或作为单独一个轮换(第二项)

- 特殊值：

  - $\left[\begin{matrix}n\\0\end{matrix}\right]=\begin{cases}0,&n\ne0\\1,&n=0\end{cases}$
  - $\left[\begin{matrix}n\\1\end{matrix}\right]=(n-1)!$
  - $\left[\begin{matrix}n\\n-1\end{matrix}\right]=\left(\begin{matrix}n\\2\end{matrix}\right)=\frac{n(n-1)}2$
  - $\left[\begin{matrix}n\\n\end{matrix}\right]=1$
  - 一般来说，第一类斯特林数大于第二类斯特林数，当$k=0,n-1,n$时两者相等
  - $\sum_{k=0}^n\left[\begin{matrix}n\\k\end{matrix}\right]=n!$

## 调和数

- 定义：$\begin{align}H_n=\sum_{k=1}^n\frac1{k}\end{align}$

- 和第一类斯特林数关系：$n!H_n=\left[\begin{matrix}n+1\\2\end{matrix}\right]$

  证：$\begin{align}&\left[\begin{matrix}n+1\\2\end{matrix}\right]=n\left[\begin{matrix}n\\2\end{matrix}\right]+(n-1)!\\&\Rightarrow\frac1{n!}\left[\begin{matrix}n+1\\2\end{matrix}\right]=\frac1{(n-1)!}\left[\begin{matrix}n\\2\end{matrix}\right]+\frac1n,由递推式\\&\Rightarrow\frac1{n!}\left[\begin{matrix}n+1\\2\end{matrix}\right]=H_n\end{align}$

$\begin{align}&当2^k\le n<2^{k+1}时,只需证明n=2^k时H_n>\frac{\lfloor\lg n\rfloor+1}2\\&且n=2^{k-1}-1时H_n\le\lfloor\lg n\rfloor+1即可\\&(1)当n=2^k时,右式=\frac{k+1}2\\&H_n=\sum_{i=0}^k\sum_{j=2^{i-1}}^{2^i}\frac1j>\sum_{i=0}^k\frac{2^{j-1}}{2^j}>\frac{k+1}2\\&(2)当n=2^{k+1}-1时,右式=k+1\\&H_n=\sum_{i=0}^k\sum_{j=i}^{2^{i+1}-1}\frac1j\le\sum_{i=0}^k\end{align}$
