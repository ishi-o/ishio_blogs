---
title: "具体数学: 取整函数"
date: 2024-10-16
categories: [SE Courses, Concrete Math]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
- 对任意实数$x$，有$x-1<\lfloor x\rfloor\le x\le\lceil x\rceil<x+1$

  - 当$x$为整数时，$\lfloor x\rfloor=\lceil x\rceil$
  - 当$x$为小数时，$\lfloor x\rfloor=\lceil x\rceil-1$

- 两个取整函数关于原点对称：$\lceil-x\rceil=-\lfloor x\rfloor$，反之亦然

  - 回顾上阶幂和下阶幂的性质：$(-x)^{\underline m}=(-1)^mx^{\overline m}$

- 加减法中，整数项可以随意移出取整符号，但乘除法不行

- 若$f(x)$是任意**连续**的**单调上升**函数，且$f(x)\in Z\Rightarrow x\in Z$，则$\lfloor f(\lfloor x\rfloor)\rfloor=\lfloor f(x)\rfloor$(上取整亦然)

- 取整函数求区间内整数个数：

  - $[\alpha,\beta]$：$\lfloor\beta\rfloor-\lceil\alpha\rceil+1$
  - $[\alpha,\beta)$：$\lceil\beta\rceil-\lceil\alpha\rceil$，可通过上式分析
  - $(\alpha,\beta]$：$\lfloor\beta\rfloor-\lfloor\alpha\rfloor$，可通过上式分析
  - $(\alpha,\beta)$：$\lceil\beta\rceil-\lfloor\alpha\rfloor-1$，记住

- 若$\begin{align}\lfloor\sqrt[3]{n}\rfloor\backslash n,n\in N^+\end{align}$，则称$n$为赢者数；求**小于**正整数$N$的赢者数的个数：

  分为两段，一段是可被取完的，另一段是无法被取完的，其分割点为$k$，它满足$\sqrt[3]{k}=\lfloor\sqrt[3]{k}\rfloor=\lfloor\sqrt[3]{N}\rfloor$
  令$m=\sqrt[3]{k}$

  $\begin{align}&则有原式=\sum_{1\le i<m,n<(i+1)^3}[i\backslash n]+\sum_{i=m,n<N}[i\backslash n]=\sum_{i=1}^{m-1}\left\lceil\frac{(i+1)^3-i^3}{i}\right\rceil+\left\lceil\frac{N-m^3}{m}\right\rceil\\&=\sum_{i=1}^{m-1}\lceil3i+3+\frac1i\rceil+\lceil\frac{N}{m}\rceil-m^2=\sum_{i=1}^{m-1}(3i+4)+\left\lceil\frac Nm\right\rceil-m^2\\&=\frac12(7+3m+1)(m-1)+\left\lceil\frac Nm\right\rceil-m^2=\frac12m^2+\frac52m+\left\lceil\frac Nm\right\rceil-4\end{align}$

  若允许等于$N$，则由$\begin{align}\left\lceil\frac{N+1-m^3}{m}\right\rceil=\left\lceil\frac{N+1}{m}\right\rceil-m^2=\left\lfloor\frac{N}{m}\right\rfloor-m^2+1\end{align}$

- (续上问)将$\begin{align}\left\lceil\frac{n}{m}\right\rceil\end{align}$替换为下取整形式($n,m$为任意实数)：

  $\begin{align}&令n=mq+r,其中q=\left\lfloor\frac{n}{m}\right\rfloor,0\le r<m\\&则原式=\left\lceil\frac{mq+r}{m}\right\rceil=q+\left\lceil\frac{r}{m}\right\rceil,\\&而\left\lfloor\frac{n+m-1}{m}\right\rfloor=\left\lfloor\frac{(q+1)m+r-1}{m}\right\rfloor=q+1+\left\lfloor\frac{r-1}{m}\right\rfloor\\&无论r=0,r\ne0,两者均相等,因此原式=\left\lfloor\frac{n+m-1}{m}\right\rfloor=1+\left\lfloor\frac{n-1}{m}\right\rfloor\end{align}$

- 证明$spec(\sqrt2),spec(2+\sqrt2)$是对正整数集合的划分：

  $\begin{align}&先证它们中小于等于n的正整数恰好等于n:\\&N(\sqrt2,n)=\sum_i[1\le \left\lfloor i\sqrt2\right\rfloor\le n]=\sum_{i}[0<i<\frac{n+1}{\sqrt2}]=\left\lceil\frac{n+1}{\sqrt2}\right\rceil-1\\&2+\sqrt2的谱同理,因此N(\sqrt2,n)+N(2+\sqrt2,n)=\left\lceil\frac{n+1}{\sqrt2}\right\rceil+\left\lceil\frac{n+1}{2+\sqrt2}\right\rceil-2\\&=n+1+小数部分-2\\&由于分母为无理数,小数部分和必为1,因此原式=n\\&下证一个元素不可能同时出现在两个谱中:\\&假设有正整数\alpha,\beta,s,使\lfloor\alpha\sqrt2\rfloor=\lfloor\beta(2+\sqrt2)\rfloor=s\\&则有\alpha\sqrt2-1<s<\alpha\sqrt2,\beta(2+\sqrt2)-1<s<\beta(2+\sqrt2)\\&即\frac{s}{\sqrt2}<\alpha<\frac{s+1}{\sqrt2},\frac{s}{\sqrt2+2}<\beta<\frac{s+1}{\sqrt2+2}\\&两式相加,得s<\alpha+\beta<s+1,两个相邻整数间没有整数,因此不成立,原命题成立\end{align}$

- $n$人均匀分为$m$组问题：最大组含$\begin{align}\left\lceil\frac{n}{m}\right\rceil\end{align}$人，最小组含$\begin{align}\left\lfloor\frac{n}{m}\right\rfloor\end{align}$

  可证明：$\begin{align}n=\sum_{i=0}^{m-1}\left\lceil\frac{n-i}m\right\rceil=\sum_{i=0}^{m-1}\left\lfloor\frac{n+i}m\right\rfloor\end{align}$

  推广到任意实数$x$(通过分析$\begin{align}f(x)=\frac xm\end{align}$的性质可去除内部下取整符号)：$\begin{align}\lfloor mx\rfloor=\sum_{i=0}^{m-1}\left\lfloor x+\frac{i}m\right\rfloor\end{align}$

- 平方根取整求和：$\begin{align}\sum_{0\le k<n}\lfloor\sqrt k\rfloor\end{align}$

  $\begin{align}&解一:原式=\sum_{m,k\ge0}m[k<n][m\le\sqrt k<m+1]\\&=\sum_{m,k\ge0}m[k<n][m^2\le k<(m+1)^2]\\&=\sum_{m,k\ge0}m[m^2\le k<(m+1)^2\le n]+\sum_{m,k\ge0}m[m^2\le k<n<(m+1)^2]\\&\xlongequal[]{m=\lfloor\sqrt n\rfloor}\sum_{0\le i<m}i((i+1)^2-i^2)+m(n-m^2)\\&=mn-m^3+\sum_{i=1}^{m-1}(2i^2+i)=mn-m^3+\sum_{1}^{m}(2i^{\underline 2}+3i^{\underline 1})\delta i\\&=\frac23m^{\underline 3}+\frac32m^{\underline 2}+mn-m^3\end{align}$
