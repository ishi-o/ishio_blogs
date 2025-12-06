---
title: "数论: 扩展欧拉定理"
categories: [Algorithm, Math]
tags: [algorithm, math]
mathjax: true
date: 2025-04-20
---
<!-- placeholder -->
<!-- more -->
### 扩展欧拉定理

- 扩展欧拉定理通常**用于快速模幂中的降幂**，其内容为：$a^b\equiv\begin{cases}a^{b\ {\rm mod}\ \varphi(m)},&\gcd(a,m)=1\\a^{(b{\rm \ mod}\ \varphi(m))+\varphi(m)},&\gcd(a,m)\ne1,b\ge\varphi(m)\\a^b,&\gcd(a,m)\ne1,b<\varphi(m)\end{cases}\ \ \ \ ({\rm mod}\ m)$

  即，若底数和模数互素，可将幂$b$降为$\varphi(m)$
  否则，若指数比模数大，可将幂$b$降为$(b\mod\varphi(m))+\varphi(m)$
  很简单可以得到，如果**模数为质数**，则$\varphi(m)=m-1$，则$a^b\equiv a^{b{\rm\ mod}\ (m-1)}\ ({\rm mod}\ m)$
  实际上，当$\gcd(a,m)=1$时，第二项仍成立，这里的意思是优先考虑第一项、如果不互素才考虑第二项；其实可以把$\gcd()$限制去掉直接记第二项
  即$\begin{align}&a^b\equiv\begin{cases}a^{(b\ {\rm mod}\ \varphi(m))+\varphi(m)},&b\ge\varphi(m)\\a^b,&b<\varphi(m)\end{cases}\end{align}$

- 证明：

  - 第一项是容易得出的，由欧拉定理，若$\gcd(a,m)=1$，则$a^{\varphi(m)}\equiv1\ ({\rm mod}\ m)$
    那么$a^b\equiv a^{n\varphi(m)+(b\ {\rm mod}\ \varphi(m))}\equiv1^n·a^{b\ {\rm mod}\ \varphi(m)}\ \ ({\rm mod}\ m)$

  - 第二项：因为$\gcd(a,m)\ne1$，将$a$拆解为多个质因数的幂的乘积形式，即$\begin{align}a=\prod_{p_i为素数\wedge p_i|a} p_i^{r_i}\Rightarrow a^b=\prod_{p_i为素数\wedge p_i|a} p_i^{b·r_i}\end{align}$
    对每一个质因子$p_i$：

    - 如果$\gcd(p_i,m)=1$，则由欧拉定理得$p_i^{\varphi(m)}\equiv1\ ({\rm mod}\ m)\Rightarrow p_i^{b}\equiv p_i^{b\ {\rm mod}\ \varphi(m)}\ ({\rm mod}\ m)$

    - 否则，将$m$写成$p_i$因子和非$p_i$因子的两部分$m=s·p_i^k$，其中$k>0$
      由于$s$不是$p_i$的幂，即$\gcd(s,p_i)=\gcd(s,p_i^k)=1$，再次用欧拉定理和欧拉函数的性质得$p_i^{\varphi(s)}\equiv1\ ({\rm mod}\ s)、\varphi(m)=\varphi(s·p_i^k)=\varphi(s)\varphi(p_i^k)$

      因此$p_i^{\varphi(m)}\equiv p_i^{\varphi(s)\varphi(p_i^k)}\equiv1^{\varphi(p_i^k)}\ ({\rm mod}\ s)$
      两边同乘$p_i^k$，得$p_i^{\varphi(m)+k}\equiv p_i^k\ ({\rm mod}\ m)$
      因此$p_i^b\equiv p_i^{b-k+k}\equiv p_i^{b-k+k+\varphi(m)}\equiv p_i^{b+\varphi(m)}\ ({\rm mod}\ m)$，其中$b\ge\varphi(m)$
      于是可以对$b$进行变换，得$p_i^{b-\varphi(m)}\equiv p_i^{b}\ ({\rm mod}\ m)$，其中$b-\varphi(m)\ge\varphi(m)$，即$b\ge2\varphi(m)$
      即只有当$b\ge2\varphi(m)$时，可以降幂；于是幂的最小值范围为$[\varphi(m),2\varphi(m))$
      即降幂的最小值为$(b\ {\rm mod}\ \varphi(m))+\varphi(m)$

    结合两种情况(第一种情况可以变成第二种情况的形式)，对$a$的任意质因子$p_i$，有$p_i^b\equiv p_i^{(b\ {\rm mod}\ \varphi(m))+\varphi(m)}\ ({\rm mod}\ m)$，即$p_i^{br_i}\equiv p_i^{((b\ {\rm mod}\ \varphi(m))+\varphi(m))r_i}\ ({\rm mod}\ m)$
    即$a^b\equiv a^{(b\ {\rm mod}\ \varphi(m))+\varphi(m)}\ ({\rm mod}\ m)$，原式得证

  - 第三项：既然$b<\varphi(m)$了，自然不能降幂了
