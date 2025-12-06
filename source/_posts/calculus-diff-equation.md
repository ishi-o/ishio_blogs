---
title: "微积分: 常微分方程"
date: 2024-04-20
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 微分方程

## 概述

称$\begin{align}f(x)+f_0(x)g_0(y)+...+f_n(x)g_n(y^{(n)})=0\end{align}$为一个$n$阶常微分方程，有如下概念：

- 当所有$g(y^{(t)})=y^{(t)}$时，称其为**线性微分方程**
- 当所有$f_t(x)=C$时，称其为**常系数微分方程**
- 当$f(x)=0$时，称方程**齐次**
- 称含相互独立的、一共$n$个$C$的解为**通解**，由特定条件(称其为定解条件)求出$C$的值的解为特解，含有$C$的解为**隐式通解**(也称通积分)
- 称通解的图形为积分曲线族，特解的图形为积分曲线

## 初等积分法

### 分离变量法

称$\begin{align}\frac{dy}{dx}=h(x)g(y)\end{align}$为一阶可分离变量方程，求解过程：

- 分离变量：将$x、y$相关分别分为左右两部分，由于相除过程中$g(y)$不能等于$0$，所以需判断$g(y)=0$是否为它的特解
- 两边积分，得到隐式通解
- 如遇到$f(x、y)$，应先换元得到$\begin{align}\frac{du}{dx}=h(x)g(u)\end{align}$

---

**eg1:**

$\begin{align}&求y'=\sin^2{(x-y+1)}的通解\\&解:\\&令u=x-y+1.\ 得\frac{du}{dx}=1-y'\\&\therefore 1-\frac{du}{dx}=\sin^2u\\&\therefore\frac{du}{\cos^2u}=dx\\&\therefore \tan u=\tan(x-y+1)=x+C\end{align}$

---

**eg2:**

$\begin{align}&求解初值问题:\cos ydx+(1+e^{-x})\sin ydy=0,y(0)=\frac\pi4\\&解:\\&整理得\tan ydy=-\frac{dx}{1+e^{-x}}\\&积分得\ln|\cos y|=\ln(e^{-x}+1)+C_1\\&\therefore |\cos y|=(e^{-x}+1)e^{C_1}\\&\therefore C\cos y=e^{-x}+1,\ (C=\pm \frac{1}{e^{C_1}})\\&故该初值问题的解为2\sqrt2\cos y=e^{-x}+1\end{align}$

---

**eg3:**

$\begin{align}&已知\int_0^1f(\alpha x)d\alpha=\frac12f(x)+1,\ 且f'(x)存在,\ 求f(x).\\&解:\\&令\alpha x=u,\ 得\frac1x\int_0^xf(u)du=\frac12f(x)+1\\&\therefore \frac12xf(x)+x=\int_0^xf(u)du\\&\therefore \frac12\big(f(x)+2+xf'(x)\big)=f(x)\\&\therefore f'(x)=\frac{f(x)-2}x\\&\therefore f(x)=Cx+2\end{align}$

### 常数变易法

教材先讲的不完整的线性方程的求法，实际上就是[常系数线性方程](#线性方程)

一阶齐次线性方程也是一个一阶可分离变量方程，对于一阶非齐次线性方程，即$\begin{align}\frac{dy}{dx}+f(x)y=g(x)\end{align}$，由于所有**非齐次线性方程的通解等于与其对应的齐次线性方程通解加上它的一个特解**，所以首先也得求齐次通解，求解步骤为：

- 求对应一阶**齐次**线性方程的通解，得到$y=Ch(x)$
- 视$C$为$u(x)$，两边求导得到$\begin{align}\frac{dy}{dx}\end{align}$
- 把$y$和$\begin{align}\frac{dy}{dx}\end{align}$代入原方程，得到$u'(x)$后积分得到$u(x)$，注意不定积分后的$u(x)$含$C$，得到答案应检查是否含$C$

由推导得$u(x)$会被消掉，所以将二、三步合为一步：

- 令$u'(x)h(x)=g(x)$，积分求出$u(x)$，即$C$，得到齐次通解

只需要整理成$\begin{align}\frac{dy}{dx}\end{align}$系数为$1$的形式，就可以快速得到答案

---

**eg1:**

$\begin{align}&已知连续函数f(x)满足f(x)=\int_0^{3x}f(\frac t3)dt+e^{2x},\ 求f(x).\\&解:\\&\int_0^{3x}f(\frac t3)dt=3\int_0^xf(u)du\\&两边求导得,\ f'(x)=3f(x)+2e^{2x}\\&先求对应齐次方程通解,\ 当f'(x)=3f(x)时,\ \ln\Big|f(x)\Big|=3x+C_0\\&\therefore f(x)=C_1e^{3x},\ 令C_1=u(x)\\&\therefore u'(x)·e^{3x}=2e^{2x}\\&\therefore u(x)=-2e^{-x}+C,\ f(x)=(-2e^{-x}+C)e^{3x}\\&由于f(0)=1,\ \therefore f(x)=3e^{3x}-2e^{2x}\end{align}$

这道题是**隐含了特解**的

### 伯努利方程

称形如$\begin{align}\frac{dy}{dx}+f(x)y=g(x)y^n,\ (n\ne 0,\ 1)\end{align}$的微分方程为伯努利方程，求解步骤如下：

- 两边除以$y^n$，得$y^{-n}y'+f(x)y^{1-n}=g(x)$，因为$(1-n)y^{-n}y'=(y^{1-n})'$，故令$u(x)=y^{1-n}$
- $\begin{align}\frac{u'}{1-n}+f(x)u=g(x),\ u=y^{1-n}\end{align}$
- 视为一阶线性方程求解
- 将$u$替换为$y^{1-n}$

---

**eg1:**

$\begin{align}&求微分方程3(1+x^2)y'+2xy=2xy^4的通解.\\&解:\\&令u=y^{-3},\ 得-(1+x^2)u'+2xu=2x\\&\because (1+x^2)u'=2xu的通解为:\\&\ln u=\ln(1+x^2)+C_0\\&即u=C_1(1+x^2)\\&令C_1=v(x),\ 得v'(x)(1+x^2)=-2x(1+x^2)^{-1}\\&\therefore v(x)=\frac{1}{1+x^2}+C,\ y^{-3}=(C+\frac{1}{1+x^2})(1+x^2)\end{align}$

### 齐次微分方程

当$x、y$次数相吻合，即$\begin{align}y'=f(\frac yx)\end{align}$时：

- 令$\begin{align}u(x)=\frac yx\end{align}$，则$y'=(ux)'=u'x+u=f(u)$，即$u'x=g(u)$
- 通过分离变量法求解
- 将$u$替换为$\begin{align}\frac yx\end{align}$

### 化为齐次微分方程

对于形如$\begin{align}\frac{dy}{dx}=f\Big(\frac{a_1x+b_1y+c_1}{a_2x+b_2y+c_2}\Big)\end{align}$的方程：

- 若$c_1=c_2=0$，则显然可以上下同除$x$，构造齐次
- 否则，对$\begin{cases}a_1x+b_1y+c_1=0\\a_2x+b_2y+c_2=0\end{cases}$进行求解，如果有解，设$X=x-x_0,\ Y=y-y_0$，构造齐次
- 如果无解，即$\left|\begin{matrix}a_1&b_1\\a_2&b_2\end{matrix}\right|=0$时，令$\begin{align}\lambda=\frac{a_1}{a_2}=\frac{b_1}{b_2},\ u=a_2x+b_2y\end{align}$，构造可分离变量方程$\begin{align}y'=\Big(\frac{du}{dx}-a_2\Big)\frac1{b_2}=f\Big(\frac{ku+c_1}{u+c_2}\Big)\end{align}$

### 可降阶的高阶微分方程

假设关于$y(x)$的微分方程：

- 若$y^{(n)}=f(x)$，逐层积分即可
- 若$y''=f(x,y')$，即**含有$x$，但不含$y$**，可令$y'=u(x)、y''=u'(x)$，代入后求出$u(x)$，积分后为$y(x)$
- 若$y''=f(y,y')$，即**含有$y$，但不含$x$**，可令$\begin{align}y'=u(y)、y''=\frac{du}{dx}=u'y'=u'·u\end{align}$，代入后求出$\begin{align}u(y)\end{align}$，再通过$y'=u(y)$和**分离变量法**求解即可
- 若**既不含$x$，也不含$y$**，则上述两种方法均可，但令$y'=u(x)$更快捷

### 一些技巧

- 常规方法求解时，一定先将$\begin{align}\frac{dy}{dx}\end{align}$整理出来
- 将$x$视为$y$，$y$视为$x$，可能变成可以求解的一阶线性方程。当整理式子发现$y'=\frac{单项式}{多项式}$时，可以考虑化为$x'(y)$
- 记住一些常见导数，将整体化为$u(x)$：
  - $xy'+y=(xy)'$
  - $\begin{align}\frac{y-xy'}{y^2}=(\frac xy)'\end{align}$

---

**eg1:**

$\begin{align}&求微分方程(x-2xy-y^2)\frac{dy}{dx}+y^2=0的通解.\\&解:\\&\frac{dx}{dy}=\frac{y^2+2xy-x}{y^2}=1+x\frac{2y-1}{y^2}\\&若\frac{dx}{dy}=x\frac{2y-1}{y^2},\ 则\frac{dx}x=\frac{2y-1}{y^2}dy\\&\therefore \ln|x|=\ln y^2+\frac1y+C_0\\&\therefore x=C_1y^2e^{\Large\frac1y},\ 令C_1=u(y)\\&得u'(y)·y^2e^{\Large\frac1y}=1\\&\therefore u(y)=e^{\Large-\frac1y}+C\\&\therefore x=y^2+Cy^2e^{\Large \frac1y}\end{align}$

## 高阶线性微分方程

### 常系数齐次方程<a id="线性方程"></a>

对于齐次线性微分方程，它的特解会是$e^{\lambda x}$的形式，故对于$A_0y^{(n)}+A_1y^{(n-1)}+\dots+A_ny=0$，它的特征方程为：

- $A_0\lambda^n+A_1y^{n-1}+\dots+A_n=0$，求出$\lambda$即得到它的一个特解
- 得出$\lambda_n$为一重根：通解为$y=C_1e^{\lambda_1 x}+C_2e^{\lambda_2 x}+\dots+C_ne^{\lambda_n x}$
- 得出$\lambda_*$为二重根：通解为$y=(C_1+C_2x)e^{\lambda_* x}+...$
- 得出$\lambda_*$为$k$重根：通解为$y=(C_1+C_2x+C_3x^2+\dots+C_{k}x^{k-1})e^{\lambda_*x}$
- 根据欧拉公式$e^{ix}=\cos x+i\sin x$，得出$\lambda_*$为一对共轭复根$\alpha\pm\beta x$：通解为$y=e^{\alpha x}(C_1\cos\beta x+C_2\sin\beta x)$
- 得出$\lambda_*$为一对共轭复根各$k$重$\pm\alpha\pm\beta x$：通解为$y=e^{\alpha x}\big((C_1+C_2x+\dots+C_kx^{k-1})\cos\beta x+(C_1'+C_2'x+\dots+C_k'x^{k-1})\sin\beta x\big)$

### 常系数非齐次方程

对于一个右边为单项式的非齐次线性微分方程，需要先求出它对应齐次微分方程的通解，再根据特征根求出原方程的一个特解(假设$P_m(x)$为系数确定的一元$m$次线性函数，$Q_m(x)$为系数待定的一元$m$次线性函数)：

- 右式为$e^{\alpha x}P_m(x)$形式，如果$\alpha$是$k$重特征根，则特解形式为：$x^ke^{\alpha x}Q_m(x)$
- 右式为$e^{\alpha x}\big(P_{m_1}(x)\cos \beta x+P_{m_2}(x)\sin \beta x\big)$形式，如果$\alpha+\beta x$是$k$重特征根，则特解形式为：$x^ke^{\alpha x}\big(Q_m(x)\cos\beta x+Q_m'(x)\sin \beta x\big)$，其中$m=\max{(m_1,\ m_2)}$

求出形式后，代入原方程求解待定的系数即可

对于多项式，可以拆分成多个单项式方程的特解之和求解

### 欧拉方程

形如$A_1x^{n}y^{(n)}+A_2x^{n-1}y^{(n-1)}+\dots+A_nxy'+A_{n+1}y=f(x)$的方程称为欧拉方程

- 令$t=\ln x$，导数算子$\begin{align}D(y)=\frac{dy}{dt}\end{align}$，得到$\begin{align}&xy'=D(y)=y'(t)、x^2y''=D(D-1)(y)=y''(t)-y'(t)、\dots、\\&x^ny^{(n)}=D(D-1)(D-2)\dots(D-n+1)(y)\end{align}$
- 代入方程得到的是关于$y(t)$的微分方程

### 解结构降阶法

对解的结构的应用，类似于线性代数中线性方程组的解，以下所指非齐次解均指同一个非齐次线性方程的解、齐次解均指该非齐次线性方程对应的齐次线性方程的解：

- 齐次解+齐次解=齐次解
- 非齐次解+齐次解=非齐次解
- 非齐次解-非齐次解=齐次解
- 已知齐次通解$y$，则非齐次通解为$y+y^*$，其中$y^*$为非齐次特解
- 假设一个关于$y(x)$的**齐次线性**微分方程，且**已知一个特解**$y^*$，则$Cy^*$也是解，视$C$为$u(x)$，代入原方程解出$u(x)$即可得到通解，需注意解出的$u(x)$含$C$

降阶在于，代入原方程后$u(x)$会被消掉，只剩$u^{(n)}(x)$
