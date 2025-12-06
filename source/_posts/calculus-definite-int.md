---
title: "微积分: 定积分"
date: 2023-11-28
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 定积分

## 定义

$\begin{align}\int_a^bf(x)dx=\lim_{\lambda\rightarrow0}\sum_{i=1}^nf(\xi_i)\Delta x_i\end{align}$

## 可积的条件

1) f(x)在[a, b]上连续
2) f(x)在[a, b]上单调
3) f(x)在[a, b]上有界，且只有有限个间断点

## 一些特殊性质

### 单调性(反之不成立)

$\begin{align}若f(x)与g(x)在[a,b]上均可积，且f(x)\le g(x)，则\int_a^bf(x)dx\le\int_a^bg(x)dx\end{align}$

### $\textcolor{red}{最大最小值}$

$若M和m分别是可积函数f(x)的最大值和最小值，则\\m(b-a)\le\int_a^bf(x)dx\le M(b-a)$

### $\textcolor{red}{区间可加性}$

$\begin{align}&可以分为若干个分割，即\\&\int_a^bf(x)dx=\int_a^cf(x)dx+\int_c^bf(x)dx\\&\int_a^bf(x)dx=-\int_b^af(x)dx\end{align}$

### 乘法可积

$若f(x), g(x)在[a,b]上可积,则f(x)g(x)在[a,b]上也可积$

### $\textcolor{red}{积分中值定理}$

$\begin{align}&若f(x)在[a,b]上连续,则存在\xi\in[a,b],使\\&\int_a^bf(x)dx=f(\xi)(b-a)\end{align}$

## 牛顿莱布尼茨公式

$\begin{align}&求定积分时，可以直接求f(x)不带C的原函数F(x)\\
&\int_a^bf(x)dx=F(x)(b-a)\end{align}$

## 变上限函数

### 定义(f不带C的一个特殊原函数)

$\begin{align}\phi(x)=\int_a^xf(t)dt\end{align}$

### 性质

#### 连续且有界

$如果f可积，则\phi(x)连续且有界$

#### 导函数

$\phi'(x)=f(x)$

任何连续函数都具有原函数

## [题型]计算定积分

#### $\begin{align}\textcolor{red}{1.\int_a^bf(x)dx=\int_a^bf(a+b-x)dx}\end{align}$

即令t=a+b-x, 此方法可以证明它本身和f(sin x)与f(cos x)的关系

$\begin{align}利用此方法还可以计算：\int_a^bf(x)dx=\frac{1}{2}\int_a^b(f(x)+f(a+b-x))dx\end{align}$

#### 2.利用牛顿莱布尼茨公式

#### $\textcolor{red}{3.利用奇偶性}$

$\begin{align}\int_{-a}^af(x)dx=\int_0^a{(f(x)+f(-x))dx}\end{align}$

当f(x)是奇函数时,$\begin{align}\int_{-a}^af(x)dx=0\end{align}$

当f(x)是偶函数时,$\begin{align}\int_{-a}^af(x)=2\int_0^af(x)dx\end{align}$

当f(x)中有一部分是奇或偶函数时也可尝试这个公式

#### 4.利用周期性

$\begin{align}&若f(x)以T为周期，则\\&\int_a^{a+T}f(x)dx=\int_0^Tf(x)dx\\&\int_a^{a+nT}f(x)dx=n\int_0^Tf(x)dx\end{align}$

#### 5.关于三角函数的定积分

$\begin{align}1).\int_0^{\frac{\pi}{2}}(\cos x)dx=\int_0^{\frac{\pi}{2}}(\sin x)dx\end{align}$

​ 1)可以使$\begin{align}t=\frac{\pi}{2}-x证明\end{align}$

$\begin{align}2).\int_0^\pi xf(\sin x)dx=\frac{\pi}{2}\int_0^\pi f(\sin x)dx=\pi\int_0^\frac{\pi}{2}f(\sin x)dx\end{align}$

$\begin{align}&3).\int_0^\frac{\pi}{2}f(\sin^nx)dx\\
&当n为偶数时,\int_0^\frac{\pi}{2}f(sin^nx)dx=\frac{n-1}{n}\frac{n-3}{n-2}...\frac{1}{2}·\frac{\pi}{2}\\&当n为奇数时,\int_0^\frac{\pi}{2}f(sin^nx)dx=\frac{n-1}{n}\frac{n-3}{n-2}...\frac{1}{3}·1\end{align}$

## [题型]计算极限

### 求数列多项和的极限

1. 先考虑夹逼定理

2. 若无法夹逼，可以提出$\begin{align}\frac{1}{n}\end{align}$后，化为一个0到1的定积分

例：

$\begin{align}&求\lim_{n\rightarrow\infty}(\frac{n}{n^2+1^2}+\frac{n}{n^2+2^2}+\frac{n}{n^2+3^2}+...+\frac{n}{n^2+n^2})\\&解：\\&原式=\frac{1}{n}\sum_{i=1}^\infty n·\frac{n}{n^2+i^2}(n\rightarrow\infty)=\int_0^1\frac{1}{1+x^2}dx=\arctan 1=\frac{\pi}{4}\end{align}$

### 关于变限函数导数的计算

通过换元，可以得到$\begin{align}\int_{v(x)}^{u(x)}f(t)dt=\int_0^{u(x)}f(t)dt-\int_0^{v(x)}f(t)dt=f\Big(u(x)\Big)u'(x)-f\Big(v(x)\Big)v'(x)\end{align}$

## 求面积

求面积时一定注意结果是**正数**

### 单个函数

最基础的，求一个连续图形的面积和x轴包围的面积，即求$\begin{align}\int_a^by\ dx\end{align}$

如果要求和y轴包围的面积，反函数连续，即求$\begin{align}\int_a^bx\ dy\end{align}$

### 多条曲线

如果求多条曲线包围的面积，先观察它或反函数在这个区间内是否连续，取微分时，被积函数是连续的可以方便计算，同时也要判断反函数是否好求

---

**eg1**：

$\begin{align}&在第一象限内，求曲线\rm y=-x^2+1上的一点，使该点处的切线与所给曲线及两坐标轴围成的图形面积为\\&最小，并求此最小面积\\&解:先求切线，设切点为(a,-a^2+1)\\&则切线为\rm y=-2ax+a^2+1\\&得到与x、y轴的交点分别为(\frac{a^2+1}{2a},\ 0)、(0,\ a^2+1)\\&则S=\int_0^{\Large\frac{a^2+1}{2a}}(-2ax+a^2+1)dx-\int_0^1(-x^2+1)dx=\frac{a^3}{4}+\frac{a}{2}+\frac{1}{4a}-\frac{2}{3}\\&\therefore S'=\frac{3a^2}{4}+\frac{1}{2}-\frac{1}{4a^2}(a\in[0,1])，令S'=0，得a^2=\frac{1}{3}，即a=\frac{1}{\sqrt3}\\&易得该点为最小值S_{min}=\frac{2}{9}(2\sqrt3-3)\end{align}$

---

**eg2**：

$\begin{align}&已知抛物线\rm y=px^2+qx(其中p<0,q>0)在第一象限内与直线x+y=5相切，且抛物线与x轴围成\\&的平面图形的面积为S\\&(1)问:p和q为何值时，S达到最大值?\\&(2)求出此最大值\\&令y=0得x=-\frac{q}{p}，故S=\int_0^{\large-\frac{q}{p}}(px^2+qx)dx=\frac{q^3}{6p^2}\\&又由相切，得\begin{cases}2px_0+q=-1\\px_0^2+qx_0=-x_0+5\end{cases}，切点为(x_0,y_0)\\&\therefore S=\frac{200q^3}{3(q+1)^4}，令S'=0，得q=3，易得S_{max}=\frac{225}{32}\end{align}$

---

**eg3**：

$\begin{align}&设F(x)=\begin{cases}e^{2x},&x\le0\\e^{-2x},&x>0\end{cases}，S表示夹在x轴与曲线y=F(x)之间的面积，对任何t>0，S_1(t)表示矩形\\&-t\le x\le t,0\le y\le F(t)的面积，求\\&(1)S(t)=S-S_1(t)的表达式\\&(2)S(t)的最小值\\&解:S(t)=2\int_0^{+\infty}e^{-2x}dx-2t·e^{-2t}=1-2t·e^{-2t}\\&\therefore S'(t)=(4t-2)e^{-2t}\\&令S'(t)=0，得t=\frac{1}{2},易得S(t)_{min}=1-\frac{1}{e}\end{align}$

### 极坐标下面积

在利用定积分前，需要知道怎么将一般方程转换成极坐标方程：令$\rm x=a\cos\theta,\ y=a\sin\theta$即可

---

**eg1**：

求双纽线$\rm (x^2+y^2)^2=x^2-y^2$的极坐标方程

$\begin{align}&解:令x=r\cos\theta,y=r\sin\theta,得r^4=r^2(\cos^2\theta-\sin^2\theta),即r=\sqrt{\cos2\theta}=\rho(\theta)\end{align}$

接下来求它围成的面积，对$\theta$取微分，将面积分割为许多个扇形，而$\begin{align}\mathbf{S_{扇形}=\frac{1}{2}r^2\theta}\end{align}$

那么，积分上下限怎么求呢? $\theta$是从原点出发而言的，双纽线是对称图形，这意味着只需要求第一象限部分的面积，那么积分下限就是0，而**$\mathbf r$是大于0的**，因此**令$\mathbf{r=0}$**，得到$\begin{align}\theta=\frac{\pi}{4}\end{align}$

因此$\begin{align}\rm S=4\int_0^{\large\frac{\pi}{4}}\frac{1}{2}r^2d\theta\end{align}$

---

**eg2**：求两种极坐标下曲线围成的面积，注意一般这种曲线具有**对称性**，所以是两倍的$\begin{align}[0,\pi]区间或[-\frac{\pi}{2},\frac{\pi}{2}]\end{align}$

求心脏线$\begin{align}\rho=a(1+\cos\varphi)\end{align}$与圆$\rho=a$所围成的各部分的面积(a>0)

$\begin{align}&解:\\&(1)求共有面积\\&[简单来说,谁的r小就取谁的表达式,因为是''共有面积'']\\&\therefore当\cos\theta\le0时，取心脏线\rho_1，否则取\rho_2\\&\therefore S=\frac{1}{2}·2\Bigg(\int_0^{\large\frac{\pi}{2}}a^2d\theta+\int_{\large\frac{\pi}{2}}^\pi\Big (a(1+\cos\varphi)\Big)^2d\theta\Bigg)\\&(2)求心脏线独有面积\\&[当心脏线大于圆时计算面积]\\&S=2·\frac{1}{2}\int_0^{\large\frac{\pi}{2}}\Big(a(1+\cos\theta)\Big)^2-a^2d\theta\\&(3)求圆独有面积:与(2)同理，不多赘述\end{align}$

---

**eg3**：简易的画图可以帮助更快地做题，我们将右式在$[0,2\pi](或[-\pi,\pi])$上展开，大于0的范围合起来就是图

$\begin{align}&\rm求曲线r=\sqrt2\sin\theta及r^2=\cos2\theta所围成的公共部分的面积\\&解:[分析:只有r大于0时才算进图里，对r_1来说,\theta\in{\small[0,\pi]},对r_2来说,\theta\in{\small[-\frac{\pi}{4},\frac{\pi}{4}]\And[\frac{5\pi}{4},\frac{7\pi}{4}]}]\\&令r_1=r_2，得\theta=\frac{\pi}{6}或\frac{5\pi}{6},当\frac{\pi}{6}<\theta<\frac{\pi}{4}时，r_1>r_2\\&\therefore S=\frac12·2(\int_0^{\large\frac{\pi}{6}}2\sin^2\theta\ d\theta+\int_{\large\frac{\pi}{6}}^{\large\frac{\pi}{4}}\cos2\theta \ d\theta)=\frac\pi6+\frac{1-\sqrt3}2\end{align}$

### 旋转体表面积

## 求体积

### 柱壳法

当所求旋转体是中空时，可以用柱壳法求体积，以每层薄薄的"柱壳"为$dt$，对'$周长·高$'积分

**eg**：

$\begin{align}&\rm求曲线y=x^2-2x,y=0,x=1,x=3所围成的平面图形绕y轴旋转一周的旋转体的体积V\\&解:先画图\end{align}$

<img src="旋转体.png" alt="旋转体" style="zoom:33%;" />

$\begin{align}&\rm因为[1,2]范围是负数,所以V=\int_2^12\pi xy\ dx+\int_2^32\pi xy\ dx=9\pi\end{align}$

其中，$\rm 2\pi x$为每个柱壳的周长，$\rm y$为每个柱壳的高。注意，当旋转轴不是单纯的x轴或y轴时，求柱壳体积需额外注意，例如本例，如果绕$x=-1$旋转，则$\rm V_{柱壳}=2\pi(x+1)y$

### 截面法

显然，不是所有的情况都适用柱壳法，大多数情况下使用的是截面法，特别是多条复杂曲线所围成的图形

**eg**：

$\begin{align}&\rm设直线y=\frac{\sqrt2}2x和y=x^2围成的面积为S_1,它们与x=1围成的面积为S_2,求这两个平面图形绕x轴旋转\\&而成旋转体的总体积\rm V\\&解:画图\end{align}$

<img src="定积分_旋转体2.png" alt="定积分_旋转体2" style="zoom:33%;" />

$\begin{align}&\rm V=\int_0^{\large\frac{\sqrt{2}}2}\pi(\frac{x^2}2-x^4)dx+\int_{\large\frac{\sqrt{2}}2}^1\pi(x^4-\frac{x^2}2)dx=\frac{(1+\sqrt2)\pi}{30}\end{align}$
