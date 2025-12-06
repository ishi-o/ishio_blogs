---
title: "微积分: 关于极限"
date: 2023-10-30
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 极限

## [注意]关于极限

1. $\begin{align}x\rightarrow 0时,\cos \frac{1}{x},\sin \frac{1}{x}都为有界量,和x相乘等于无穷小\end{align}$
2. $\begin{align}合理利用换元,如求\lim_{x\rightarrow 0}(\frac{x}{x+1})^x可令t=x+1\end{align}$
3. 合理利用夹逼定理，如求多项式极限时，有时也会用到定积分
4. 求式子中参数a、b时，若一边为0，可猜测一边也为0(因为这样才有不为0的极限存在)，用洛必达法则求出参数

## 求反函数

虽然并不属于极限的范畴，但是是有可能考到的题型

总的来说就是让newY=oldX，newX=f(x)

newX的范围就是f(x)的值域

---

eg:

$\begin{align}&f(x)=\begin{cases}1-2x^2,&x\lt-1\\x^3,&-1\le x\le2\\12x-16,&x\gt2\end{cases}\\&求f^{-1}(x)\\&解:1).当x\lt-1时，f^{-1}(x)=\sqrt\frac{1-x}{2}，x\lt-1\\&2).当-1\le x\le2时，f^{-1}(x)=\sqrt[3]{x}，-1\le x\lt8\\&3).当x\gt2时，f^{-1}(x)=\frac{16+x}{12}，x>8\end{align}$

---

## [题型]利用单调有界收敛定理证明函数极限存在

<font color="red">**def：单调（增减）且有（上下）界的数列一定收敛**</font>

1. 放缩法证明其有（上/下）界

2. 观察法证明其单调性

3. 合理利用假设法

   ---

$\begin{align}&eg:\\
&设c>0,a_1=\sqrt c,且a_{n+1}=\sqrt{c+a_n}.证明数列{a_n}收敛,并求其极限值\\&证明:\\&\because a_2=\sqrt{c+a_1}>\sqrt c=a_1.\\&假设a_n>a_n-1,则有a_{n+1}=\sqrt{c+a_n}>\sqrt{c+a_{n-1}}=a_n\\&\therefore a_n单调递增\\&又\because a_2<\sqrt c+1.假设a_n<\sqrt c+1\\&则有a_{n+1}=\sqrt{c+a_n}<\sqrt{c+\sqrt c+1}<\sqrt c+1\\&\therefore a_n有上确界\\&\therefore \lim_{n\rightarrow\infty}a_n=\sqrt c+1\end{align}$

---

## 求极限

### [题型]同底数异指数或同指数异底数(常考确定某个常量的值)

提出后一项求极限

eg:

---

$\begin{align}
&1):同指数异底数\\&
已知极限\lim_{n\rightarrow\infty}\frac{n^\alpha}{(n+1)^\beta-n^\beta}=2017,求\alpha,\beta\\&
解：\\&
\because 原式=\lim_{n\rightarrow\infty}\frac{n^\alpha}{n^\beta((\frac{n+1}{n})^\beta-1)}\\&
=\lim_{n\rightarrow\infty}\frac{n^\alpha}{n^\beta(e^{\beta \ln(\frac{n+1}{n})}-1)}\\&
=\lim_{n\rightarrow\infty}\frac{n^\alpha}{n^\beta(\beta\ln(\frac{n+1}{n}))}\\&
=\lim_{n\rightarrow\infty}\frac{n^\alpha}{n^\beta\frac{1}{n}\beta}=\lim_{n\rightarrow\infty}\frac{n^\alpha}{n^{\beta-1}\beta}=2017\\&
\therefore 易得\alpha=\beta-1,\beta=\frac{1}{2017}\\&
\therefore \alpha=-\frac{2016}{2017}\end{align}$

---

$\begin{align}&2):同底数异指数\\&
\lim_{x\rightarrow 0}\frac{e^x-e^{\sqrt{3}{x}}}{x^k}=c,c为非零常数,k为常数,求c,k\\&
解：\\&
\because 原式=\lim_{x\rightarrow 0}\frac{e^{\sqrt{3}{x}}(e^{x-\sqrt{3}{x}}-1)}{x^k}\\&
=\lim_{x\rightarrow 0}\frac{e^0(x-\sqrt{3}{x})}{x^k}=c\\&
\therefore k=\frac{1}{3},c=-1\end{align}
$

---

### [题型]上下同除

利用除以$x^3$和四则运算求极限

---

$\begin{align}&eg:\\&
计算\lim_{x\rightarrow 0}\frac{\tan x-x}{(\sqrt{1+x}-1)\ln(x^2+1)+x^4·\cos \frac{1}{x}}\\&
解：原式=\lim_{x\rightarrow 0}\frac{\frac{\tan x-x}{x^3}}{\frac{(\sqrt{1+x}-1)\ln(x^2+1)+x^4·\cos \frac{1}{x}}{x^3}}=\frac{2}{3}\end{align}
$

---

## [题型]间断点判断

1. 找出间断点

   1.1. 寻找间断点时，这个点可能是**未定义**的，也可能是**两边极限不等**的

   1.2. 具体例子为：前者$\begin{cases}\large\frac{1}{x}\normalsize,&x=0\\\ln x,&x=0\end{cases}$，后者$\begin{cases}x^\infty,&x=1\\ e^{\Large\frac{1}{x-1}},&x=1\end{cases}$

2. 求极限

   ---

   $\begin{align}&eg:\\&
   求函数f(x)=(1+x)^{\frac{x}{\tan(x-\frac{\pi}{4})}}在区间(0,2\pi)内的间断点,并判断其类型\\&
   解：\\&
   令\tan(x-\frac{\pi}{4})=0,得x=\frac{\pi}{4}或x=\frac{5\pi}{4}\\&
   令x-\frac{\pi}{4}=\frac{\pi}{2}+k\pi,得x=\frac{3\pi}{4}或x=\frac{7\pi}{4}\\&
   x\rightarrow\frac{\pi}{4}^+时,\lim_{x\rightarrow\frac{\pi}{4}^+}f(x)不存在(趋于正无穷大)\\&
   x\rightarrow\frac{\pi}{4}^-时,\lim_{x\rightarrow\frac{\pi}{4}^-}f(x)=0,故x=\frac{\pi}{4}是第二类间断点,x\rightarrow\frac{5\pi}{4}与其类似\\&
   x\rightarrow\frac{3\pi}{4}时,\lim_{x\rightarrow\frac{3\pi}{4}}f(x)=\lim_{x\rightarrow\frac{7\pi}{4}}f(x)=1,故均为第一类间断点,由于两边极限相等,故为可去间断点\end{align}$

---

## [题型]复合函数的极限

拆出f(x),用α(x)代替

---

$\begin{align}&eg:\\&
已知\lim_{x\rightarrow 0}\frac{\sqrt{1+f(x)\sin 2x}-1}{\tan 3x}=2,求\lim_{x\rightarrow 0}f(x)\\&
解：\\&
令\alpha(x)=\frac{\sqrt{1+f(x)\sin 2x}-1}{\tan 3x},则\lim_{x\rightarrow 0}\alpha(x)=2\\&
\therefore f(x)=\frac{(\alpha(x)\tan 3x+1)^2-1}{\sin 2x}\\&
\therefore \lim_{x\rightarrow 0}f(x)=\lim_{x\rightarrow 0}\frac{2\alpha(x)\tan 3x}{\sin 2x}=6\end{align}$

---

# 导数与微分

## 基本公式

$\begin{align}&(\tan x)'=\frac{1}{\cos^2x}=sec^2x\\&
(\sec x)'=\sec x\tan x,\ (\csc x)'=-\csc x\cot x\\&
(arctan x)'=\frac{1}{1+x^2},\ (arccot x)'=-\frac{1}{1+x^2}\\&
(\arcsin x)'=\frac{1}{\sqrt{1-x^2}},\ (\arccos x)'=-\frac{1}{\sqrt{1-x^2}}\end{align}$

## [题型]是否可导问题和求导数问题

1. 先判断在该点是否连续，不连续则必不可导

2. 后用定义求左右导数，若都存在且相等才可导

3. 在其他部分正常求导即可
   $$
   \begin{align}\Large&是否连续与是否可导不一样\\\\&连续:\lim_{x\rightarrow x_0}f(x)=f(x_0)\\&可导:f'(x_0)=\lim_{x\rightarrow x_0^+}\frac{f(x)-f(x_0)}{x-x_0}=\lim_{x\rightarrow x_0^-}\frac{f(x)-f(x_0)}{x-x_0}\end{align}
   $$

4.

## [题型]反函数的求导

$\begin{align}&f'(x) = \frac{1}{\varphi'(y)}\\&
即\frac{dy}{dx} = \frac{1}{\frac{dx}{dy}}\end{align}$

## [注意]函数的微分

$\begin{align}&\frac{函数的微分}{自变量的微分}=函数的导数=f'(x)\\\\&
所以\textcolor{red}{dy = f'(x)·dx}\\&
函数的增量\Delta y = f(x + \Delta x) - f(x)\\&当\Delta x\rightarrow 0时,\textcolor{red}{\Delta y = f'(x)·\Delta x + o(\Delta x) = dy + o(\Delta x)}\end{align}$

## [题型]高阶导数和微分

### 技巧

1. 归纳为递推公式

2. sin或cos进行降次(合并为幂次方或二倍角公式或积化和差公式)

   2.1 积化和差公式

$\begin{align}&\ \ \ \ \ \ \ 令\alpha=\frac{a+b}{2},\beta=\frac{a-b}{2}\\&
\ \ \ \ \ \ \ 例如\sin\alpha\cos\beta可以化为\frac{1}{2}(\sin(\alpha+\beta)+\sin(\alpha-\beta))\end{align}$

3. 取对数法：多项乘法由于展开后更好求，一般可以取对数

### 莱布尼茨公式(前面系数与二次项系数类似)

$\begin{align}&(f\small(x)·g(x)\big)^{(n)} = C^0_nf·g^{(n)} + C^1_nf'·g^{(n-1)} + … + C^0_nf{(n)}·g\\\\&可以分解假分式后利用莱布尼茨公式展开真分式\end{align}$

### [题型]隐函数的求导

两边同时对x求导再整理即可

### [题型]参数方程求导

$\begin{align}&y' = \frac{dy}{dx} = \frac{\frac{dy}{dt}}{\frac{dx}{dt}}\\&
y'' = \frac{d}{dx}(y') = \frac{d}{dt}(y')·\frac{1}{\frac{dx}{dt}}\ 即先对t求导,再乘上x对t求导的倒数\end{align}$

### [题型]相关变化率

1. 先找所给量间的关系
2. 两边函数的导数如果都是和某个变量相关的不同函数，则两边同时求导

### [题型]求曲率

$$
\begin{align}&K=\frac{\Delta\Theta}{\Delta s}=\frac{y''}{(1+(y')^2)^\frac{3}{2}}\\&
\Delta s=\sqrt{1+(y')^2}dx=\sqrt{dy^2+dx^2}\\&
R=\frac{1}{K}\end{align}
$$

### [注意]泰勒公式

$$
常用拉格朗日公式：\\\sin x=x-\frac{x^3}{3!}+\frac{x^5}{5!}-……\\
\cos x=1-\frac{x^2}{2}+\frac{x^4}{4!}-……\\
\ln x=x-\frac{x^2}{2}+\frac{x^3}{3}-……\\
e^x=1+x+\frac{x^2}{2}+\frac{x^3}{3!}+……\\
(1+x)^\alpha=1+\alpha x+\frac{\alpha(\alpha-1)x^2}{2!}+……\\
$$
