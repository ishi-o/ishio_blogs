---
title: "微积分: 不定积分"
date: 2023-11-12
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 不定积分

## 公式

### 基本公式

$\begin{align}
&\int kdx=kx+C,\ \int kx^\alpha dx=\frac{k}{\alpha+1}x^{\alpha+1}+C(\alpha\neq -1)\\&
\int \frac{dx}{x}=ln|x|+C, \int e^{kx}dx=\frac{1}{k}e^{kx}+C\\&
\int \sin xdx=-\cos x+C, \int \cos xdx=\sin x+C, \int \sec^2xdx=\tan x+C\\&
\int\csc^2xdx=\int\frac{dx}{\sin^2x}=-\cot x+C\\&
\int\sec x\tan xdx=\sec x+C, \int\csc x\cot xdx=\int\frac{\cos x}{\sin^2x}dx=-\csc x+C\\&
\int \frac{1}{1+x^2}dx=\arctan x+C,\int\frac{dx}{\sqrt{1-x^2}}=\arcsin x+C
\end{align}$

### 拓展公式(需要熟悉推导过程)

$\begin{align}&
\int\tan xdx=\int\frac{\sin xdx}{\cos x}=-\ln|\cos x|+C\\&
\int\cot xdx=\int\frac{\cos xdx}{\sin x}=ln|\sin x|+C\\&
\int\sec xdx=\int\frac{dx}{\cos x}=\int\frac{\cos xdx}{\cos^2x}=\int\frac{d(\sin x)}{1-\sin^2x}=\frac{1}{2}ln|\frac{1+\sin x}{1-\sin x}|+C\\&
\int\csc xdx=\int\frac{\sin xdx}{\sin^2x}=-\int\frac{d(\cos x)}{1-\cos^2x}=-\frac{1}{2}ln|\frac{1+\cos x}{1-\cos x}|+C\\\\\\&
对于\sin^nx与\cos^nx，n较小时\\&
若n为偶数:\\&
直接通过cos2x=1-2sin^2x=2cos^2x-1降次\\&
若n为奇数:\\&
将一个抽取出来凑微分，剩下的部分通过sin^2x+cos^2x=1进行替换\\\\&
若n较大:记公式(没必要)\int\sin^nxdx=-\frac{1}{n}sin^{n-1}xcosx+\int\frac{n-1}{n}sin^{n-2}xdx
\end{align}$

### 常凑微分

$\begin{align}&
(凑常数):\int f(ax+b)dx=\frac{1}{a}\int f(ax+b)d(ax+b)=\frac{1}{a}\int f(t)dt\Big\vert_{t=ax+b}\\&
(凑x^\alpha):x^{\alpha}dx=\frac{1}{\alpha+1}dx^{\alpha+1}\\&
(凑\frac{1}{x}):\frac{1}{x}dx=d(\ln x)\\&
(凑\sec^2 x):\sec^2 xdx=d(\tan x)\\&
(凑\csc^2 x):\csc^2 xdx=-d(\cot x)\\&(凑\sqrt x):\frac{dx}{\sqrt x}=2d(\sqrt x)\\&(凑\frac{dx}{\sqrt{二次项式}}):将分母凑配方，得到\sqrt{1-Ax^2}的形式\\&(凑1\pm\frac{1}{x^2}):(1\pm\frac{1}{x^2})dx=d(x\mp\frac{1}{x}),而(x\mp\frac{1}{x})^2=(x^2+\frac{1}{x^2})\pm 2\\&\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ 即通过倒代换凑出\end{align}$

### 分部积分

$\begin{align}\int f(x)g'(x)dx=\Big(f(x)·g(x)\Big)'-\int f'(x)·g(x)\end{align}$

## 求不定积分的技巧

### 分式

#### 假分式化为真分式(分子最高次幂大于等于分母最高次幂)

exp1:

$\begin{align}&求\int\frac{x^3}{x+3}dx\\&解:\\&原式=\int\frac{(x+3)(x^2-3x+9)-27}{x+3}dx=\frac{1}{3}x^3-\frac{3}{2}x^2+9x-27ln|x+3|+C\end{align}$

具体方法:

<img src="D:\Picture\Screenshots\截图[工数上]\1.jpg" style="zoom: 25%;" />

#### 真分式化为四种分式

$\begin{align}&1.\int\frac{A}{x-a}dx\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ Aln|x-a|\\&2.\int\frac{A}{(x-a)^n}dx\ \ \ \ \ \ \ \ \ \ \ A\frac{1}{1-n}(x-a)^{1-n}\\&3.\int\frac{Mx+N}{x^2+px+q}dx\ \ \ \ \ 将分母配方为(x-a)^2+b的形式即可\\&4.\int\frac{Mx+N}{(x^2+px+q)^n}dx\end{align}$

怎么拆开? 假设我们有:

$\begin{align}&1.\frac{分子}{(x+a)^p(x+b)^q},则应拆成:\\&\frac{A_1}{(x+a)^p}+\frac{A_2}{(x+a)^{p-1}}+\frac{A_3}{(x+a)^{p-2}}+...+\frac{B_1}{(x+a)^{q-1}}+\frac{B_2}{(x+a)^{q-2}}+...\\\\&2.\frac{分子}{(x^2+px+q)(其它分母)},则应拆成:\\&\frac{Ax+B}{x^2+px+q}+...\end{align}$

然后得到:

$\begin{align}分子=分式1分子+分式2分子+...\end{align}$
