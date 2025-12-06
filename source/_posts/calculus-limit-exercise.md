---
title: "微积分: 极限练习题"
date: 2023-11-05
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 极限

## 证明数列极限存在

应当利用单调有界收敛原理

---

**eg:**

$\begin{align}&已知x_1=1，x_{n+1}=\frac{1}{1+x_n}，n=1,2,3,...，证明\lim_{n\rightarrow\infty}x_n存在\\&证:\\&\because x_{n+1}-x_n=-\frac{x_n^2+x_n-1}{1+x_n}>0，\therefore x_n单调递减\\&又x_n>0恒成立，\therefore x_n有界\\&根据有界收敛原理，\lim_{n\rightarrow\infty}x_n存在，令A=\frac{1}{1+A}，得极限=\frac{1+\sqrt5}{2}\end{align}$

## 证明极限不存在

判断极限存在时，一定要注意趋近值的两边

***

**eg1:**

$1. 当x\rightarrow1时，函数\Large\frac{x^2-1}{x-1}e^{\frac{1}{x-1}}\normalsize的极限$[](#1 "答案")<a id="~1"></a>



A. 等于2 B. 等于0 C. 为$\infty$ D. 不存在但不为$\infty$

***

**eg2:**

$\begin{align}&当x\rightarrow x_0时，判断左右极限\\&当x\rightarrow\infty时，取相近的两无穷大数判断\\&证明\lim_{x\rightarrow+\infty}x\sin x不存在\\&证:取x_1=n\pi，x_2=n\pi+\frac{\pi}{2}\\&易得\lim_{x\rightarrow+\infty}f(x_1)=0，\lim_{x\rightarrow+\infty}f(x_1)=+\infty\\&故不存在\end{align}$

---

**eg3: 换元**法将复杂问题变简单

$\begin{align}&\lim_{x\rightarrow1}(1-x)^2e^{\Large\frac{1}{x-1}}\\&解:\lim_{x\rightarrow1^-}f(x)=0\\&\lim_{x\rightarrow1^+}f(x)\xlongequal{\Large t=\frac{1}{x-1}}\lim_{t\rightarrow+\infty}\frac{e^t}{t^2}=+\infty\\&故原极限不存在\end{align}$

## 判断数列/函数的极限

数列是特殊的，需要注意它是一个个点组成的，在判断有界无界，收敛发散时与函数不同

***

**eg1:**

​ 2. 设数列 $\begin{align}x_n与y_n满足\lim_{n\rightarrow\infty}x_ny_n=0\end{align}，则正确的是$[](#2 "答案")<a id="~2"></a>

​ A. 若$x_n$发散，则$y_n$必发散  B. 若$x_n$无界，则$y_n$必有界

​ C. 若$x_n$有界，则$y_n$必为无穷小 D. 若$\begin{align}\frac{1}{x_n}\end{align}$为无穷小，则$y_n$必为无穷小

---

**eg2:**

3. 关于数列{$x_n$}，下列说法不正确的是[](#3 "答案")<a id="~3"></a>

   A. $\begin{align}若\lim_{x\rightarrow\infty}x_n=a,则\lim_{x\rightarrow\infty}x_{2n}=\lim_{x\rightarrow\infty}x_{2n-1}=a\end{align}$

   B. $\begin{align}若\lim_{x\rightarrow\infty}x_{2n}=\lim_{x\rightarrow\infty}x_{2n-1}=a,则\lim_{x\rightarrow\infty}x_n=a\end{align}$

   C. $\begin{align}若\lim_{x\rightarrow\infty}x_n=a,则\lim_{x\rightarrow\infty}x_{3n}=\lim_{x\rightarrow\infty}x_{3n-1}=a\end{align}$

   D. $\begin{align}若\lim_{x\rightarrow\infty}x_{3n}=\lim_{x\rightarrow\infty}x_{3n-1}=a,则\lim_{x\rightarrow\infty}x_n=a\end{align}$

---

函数中，注意$\infty$是**有符号**的，在判断极限是否存在时需注意两边极限，例如

$\begin{align}\lim_{x\rightarrow\infty}\arctan x，\lim_{x\rightarrow1}\arctan\frac{1}{x-1}都不存在\end{align}$



## 分子或分母有理化

应用于去根号

### 平方差

---

**eg:**

$\begin{align}&\lim_{x\rightarrow-\infty}x(\sqrt{x^2+100}+x)\\&解:原式=\lim_{x\rightarrow-\infty}\frac{100x}{\sqrt{x^2+100}-x}=\lim_{x\rightarrow-\infty}\frac{100}{-\sqrt{1+\Large\frac{100}{x^2}}\normalsize-1}=-50\end{align}$

需要注意的是，由于**x趋于负无穷**，从根号里提出一个x时需要**变号**!

### 三角函数内去根号

根据三角函数的周期性，可以让根号通过平方差进行更换

---

**eg:**

$\begin{align}&求\lim_{x\rightarrow0}\frac{1-\cos x\sqrt{\cos{2x}}\sqrt[3]{\cos{3x}}}{x^2}\\&解:原式=\lim_{x\rightarrow0}\frac{1-\cos x+\cos x(1-\sqrt{\cos{2x}}\sqrt[3]{\cos{3x}})}{x^2}\\&=\lim_{x\rightarrow0}\frac{1-\cos x+\cos x(1-\sqrt{\cos{2x}}+\sqrt{\cos{2x}}(1-\sqrt[3]{\cos{3x}}))}{x^2}\\&=\frac{1}{2}+1+\frac{2}{3}=3\end{align}$

---

[更多例子](./三角函数去根号[工数上].pdf)(由于typora导出时会强制替换为绝对路径，又没有服务器供存放附件，所以link失效)

## 倒代换

对于**趋于无穷的x**，建议先做一次**倒代换**，这能使问题更清晰，有时甚至可以直接解出题目

当然，从根号提出时一定注意x的**正负**!

还有一个好处是，对于$\sin x$这类x**趋于无穷**时是一个有界量，x**趋于0**时sin x~x的易混淆的极限值不容易被坑

还有类似$e^x，\cos x，x^\alpha$等等

---

**eg1:**

4. 设$\begin{align}f(x)=\frac{\tan x}{|x|}\arctan\frac{1}{x}，则\end{align}$[](#4 "答案")<a id="~4"></a>

   A. x=0是振荡间断点 B. x=0是无穷间断点 C. x=0是可去间断点 D. x=0是跳跃间断点

---

**eg2:**



$\begin{align}&\lim_{x\rightarrow+\infty}x^3\Bigg(\sqrt[3]{\frac{x^3+x}{x^6+x^3+1}}-\sin{\frac{1}{x}}\Bigg)\\&解:原式=\lim_{x\rightarrow0}\frac{1}{x^3}\Bigg(\sqrt[3]{\frac{x^5+x^3}{x^6+x^3+1}}-\sin x\Bigg)=\lim_{x\rightarrow0}\frac{\Bigg(x\Big(\Large\sqrt[3]{\frac{x^2+1}{x^6+x^3+1}}\normalsize-1\Big)+x-\sin x\Bigg)}{x^3}\\&=\lim_{x\rightarrow0}\frac{-x^6-x^3+x^2}{3x^2(x^6+x^3+1)}+\frac{1}{6}=\frac{1}{2}\end{align}$

## 多项式/多因式极限

### 夹逼定理

在求多项式子和的极限时，可以**放缩**求极限或者化为定积分。在这里只讨论前者

例如$\begin{align}\lim_{n\rightarrow\infty}\sqrt[n]{f(n)}=1\end{align}$

---

**eg1:**

$\begin{align}&\lim_{n\rightarrow+\infty}(\frac{n}{n^2+\pi}+\frac{n}{n^2+2\pi}+...+\frac{n}{n^2+n\pi})\\&解:原式\le\lim_{n\rightarrow+\infty}(\frac{n^2}{n^2+\pi})=1\\&原式\ge\lim_{n\rightarrow+\infty}(\frac{n^2}{n^2+n\pi})=1\\&故原式=1\end{align}$

---

**eg2:** 利用**回归法**证明不等式后夹逼

$\begin{align}&证明不等式\frac{1}{2n}<\frac{1}{2}·\frac{3}{4}·...·\frac{2n-1}{2n}<\frac{1}{\sqrt{2n+1}}，并求\lim_{n\rightarrow\infty}\frac{1}{2}·\frac{3}{4}·...·\frac{2n-1}{2n}\\&证:1).\ 中间式子=\frac{3}{2}·\frac{5}{4}·...·\frac{2n-1}{2n-2}·\frac{1}{2n}>\frac{1}{2n}\\&2).\ 两边平方，得\frac{1·3}{2^2}·\frac{3·5}{4^2}·...·\frac{(2n-1)(2n+1)}{(2n)^2}·\frac{1}{2n+1}<\frac{1}{2n+1}\\&\therefore\lim_{n\rightarrow\infty}\frac{1}{2}·\frac{3}{4}·...·\frac{2n-1}{2n}=0\end{align}$

### 合并后求极限

观察式子，可以**加减或乘除**式子可以将原式合并(回归法)，或根据等比/等差数列**求和**

---

**eg1:**

$\begin{align}&\lim_{n\rightarrow\infty}(\frac{1}{n^2+1}+\frac{2}{n^2+2}+...+\frac{n}{n^2+n})\\&解:原式\le\lim_{n\rightarrow\infty}\frac{1+2+...+n}{n^2}=\lim_{n\rightarrow\infty}\frac{n(n+1)}{2n^2}=\frac{1}{2}\end{align}$



$\begin{align}&原式\ge\lim_{n\rightarrow\infty}\frac{1+2+...+n}{n^2+n}=\lim_{n\rightarrow\infty}\frac{n(n+1)}{2(n^2+n)}=\frac{1}{2}\end{align}$

---

**eg2:**

$\begin{align}&\lim_{n\rightarrow+\infty}(1+x)(1+x^2)...(1+x^{2^n})，(|x|\le1)\\&解:原式=\lim_{n\rightarrow+\infty}\frac{(1-x)(1+x)(1+x^2)...(1+x^{2^n})}{1-x}=\frac{1-x^{2^{n+1}}}{1-x}=\frac{1}{1-x}\end{align}$

---

**eg3:**

$\begin{align}&\lim_{n\rightarrow\infty}(1+\frac{1}{n^2})·(1+\frac{2}{n^2})·...·(1+\frac{n}{n^2})\\&解:原式=e^{\Large\lim_{n\rightarrow\infty}\ln(1+\frac{1}{n^2})+\ln(1+\frac{2}{n^2})+...+\ln(1+\frac{n}{n^2})}\\&=e^{\Large\lim_{n\rightarrow\infty}\frac{n(n+1)}{2n^2}}=e^{\frac{1}{2}}\end{align}$

## 求同底数或同幂数项的极限

**并非**一定是把后面项提出，提出一项的目的是让括号内式子**变为"1+0"或"$1^+-1$"**的形式

---

**eg:**

$\begin{align}&求f(x)=\lim_{n\rightarrow\infty}\frac{\ln{(e^n+x^n)}}{n}\\&解:1).当x\lt e时，原式=\lim_{n\rightarrow\infty}\frac{\ln{e^n}+\ln{(1+\Large(\frac{x}{e})^n)}}{n}\normalsize=1\\&2).当x=e时，原式=1\\&3).当x\gt e时，原式=\lim_{n\rightarrow\infty}\frac{\ln{x^n}+\ln{(1+\Large(\frac{e}{x})^n)}}{n}\normalsize=\ln x\end{align}$

## 极限中参数的确定

先确定**带变量的值**，再确定其他参数，有时需要通过**洛必达法则**确定参数

---

**eg:**

$\begin{align}&\lim_{x\rightarrow+\infty}(\sqrt[3]{1-x^6}-ax^2-b)=0，求a和b的值\\&解:原式=\lim_{x\rightarrow+\infty}\Big[x^2(\sqrt[3]{\frac{1}{x^6}-1}-a)-b\Big]，得-1-a=0，即a=-1\\&\therefore b=\lim_{x\rightarrow+\infty}(\sqrt[3]{1-x^6}-ax^2)=\lim_{x\rightarrow+\infty}x^2\frac{1}{3x^6}=0\end{align}$

---



## 关于计算极限时需要注意的细节

1. **有限个**无穷小的和或积是无穷小，**无限个**项则不成立

2. 无穷小与**有界量**的乘积是无穷小

3. 计算极限时不能随意进行**代入**或**等价无穷小替换**

   3.3. 原因：极限是整体变化的，但是有时候这样替换是正确的，这是因为我们把**四则运算**的过程简化了

   这一问题的更详细的说明是：直接代入或等价无穷小替换是**有精度的**，分母**越小**，越会对替换带来的变化起反应，导致极限运算错误

   **判断方法**：进行**四则运算拆开**时，需要**两项因式极限都存在**才可以进行替换

   简单例子:

   $\begin{align}&\lim_{x\rightarrow0}\frac{x-\sin x}{x^3}=\lim_{x\rightarrow0}\frac{1}{x^2}-\lim_{x\rightarrow0}\frac{\sin x}{x^3}=\infty-\infty，不能替换\\&\lim_{x\rightarrow0}\frac{x+\sin x}{x}=\lim_{x\rightarrow0}1+\lim_{x\rightarrow0}\frac{\sin x}{x}=1+1，可以替换\end{align}$

   ### 经典例子

   ---

   **eg1:**

   $\begin{align}&\lim_{x\rightarrow0}\frac{x\cos x-\sin x}{x^3}\\&解:原式=\lim_{x\rightarrow0}\frac{x\cos x-x+x-\sin x}{x^3}=-\frac{1}{3}\end{align}$

   ---

   **eg2:** 这是一个隐含四则运算的极限，不能直接套用$\lim_{x\rightarrow\infty}{(1+\large\frac{1}{x})^x}=e^x$，最后用到洛必达法则

   $\begin{align}&\lim_{x\rightarrow+\infty}\frac{(1+\large\frac{1}{x})^x}{e^x}\\&解:原式=\lim_{x\rightarrow+\infty}\frac{e^{x\ln(1+\Large\frac{1}{x})}}{e^x}=\lim_{x\rightarrow+\infty}e^{x^2\Big(\ln(1+\Large\frac{1}{x}\normalsize)-\Large\frac{1}{x}\normalsize\Big)}=e^{\lim_{x\rightarrow0}\large\frac{\Large\ln(1+x)-x}{x^2}}=e^{-\frac{1}{2}}\end{align}$

   ---

   **eg3:**

   $\begin{align}&\lim_{x\rightarrow+\infty}\frac{e^x}{(1+\large\frac{1}{x})^{x^2}}\\&解:原式=e^{\Large\lim_{x\rightarrow+\infty}x-x^2\ln(1+\frac{1}{x})}=e^{\Large\lim_{x\rightarrow0}\LARGE\frac{x-\ln(1+x)}{x^2}}=e^\frac{1}{2}\end{align}$

   ---

4. 发现不能直接替换后，可以尝试**泰勒展开**或者**回归法(加一项减一项)**来解题

## 定理证明题

**介值定理**：在**闭区间**$[a,b]$上的**连续**函数任取一点，该点必定满足$m\le f(x)\le M$

引申出**零点定理**：如果存在两点**异号**，则必定存在一点$\in(a,b)$满足$f(x)=0$

**保号性定理**：如果有$\begin{align}\lim_{x\rightarrow x_0}f(x)>0\end{align}$，则在$x_0$某**去心邻域**内，$f(x)>0$成立

---

**eg1:**



$\begin{align}&设f(x)在[0,1]上连续，f(1)=0，\lim_{x\rightarrow\frac{1}{2}}\frac{f(x)-1}{\large(x-\frac{1}{2})^{\normalsize2}}=1，证明:\\&1). 存在\xi\in(\frac{1}{2},1)，使f(\xi)=\xi\ \ \ \ \ 2). f(x)在[0,1]上最大值大于1\\&解:1)由洛必达法则，f(\frac{1}{2})=1\\&令F(x)=f(x)-x，易得\begin{cases}F(1)=-1\\F(\large\frac{1}{2})=\frac{1}{2}\end{cases}\\&\therefore 连续函数F(x)存在一点\xi\in(\frac{1}{2},1)，使得F(\xi)=0\\&2). \because\lim_{x\rightarrow\frac{1}{2}}\frac{f(x)-1}{\large(x-\frac{1}{2})^{\normalsize2}}=1>0\\&由保号性可知，当x\in(\frac{1}{2}-\delta x,\frac{1}{2})\cup(\frac{1}{2},\frac{1}{2}+\delta x)时，\frac{f(x)-1}{\large(x-\frac{1}{2})^{\normalsize2}}>0\\&\therefore在这个去心邻域内f(x)>1\end{align}$

---

**eg2:**

设$a_{2m}<0$，证明：$x^{2m}+x^{2m-1}a_1+...+xa_{2m-1}+a_{2m}=0$至少有两个零点

$\begin{align}&证:左边=x^{2m}\Big(1+\frac{a_1}{x}+...+\frac{a_{2m-1}}{x^{2m-1}}+\frac{a_{2m}}{x^{2m}}\Big)\\&当x\rightarrow\infty时，原式=+\infty\\&又当x=0时，原式=a_{2m}<0\\&\therefore对(-\infty,0)和(0,+\infty)分别使用零点定理，得证\end{align}$

---

**eg3:**

$\begin{align}&设f(x)在[0,n](n\in N^*且n\ge2)上连续，f(0)=f(n)，证明:存在\xi,\xi+1\in[0,n],使f(\xi)=f(\xi+1)\\&证:令F(x)=f(x)-f(x+1)\\&则F(0)=f(0)-f(1),F(1)=f(1)-f(2),...,F(n-1)=f(n-1)-f(n)\\&合并得\sum_{i=0}^{n-1}F(i)=f(0)-f(n)=0\\&设F_{min}=m,F_{max}=M,可知nm\le\sum_{i=0}^{n-1}F(i)\le nM\\&即m\le\frac{1}{n}\sum_{i=0}^{n-1}F(i)\le M\\&由介值定理可知，存在\xi\in[0,n-1],使f(\xi)=\frac{1}{n}\sum_{i=0}^{n-1}F(i)=0\end{align}$

# 总结

1. 求单项极限时，应想到**回归法**凑无穷小、**换元法**或**平方差**解决式子**有理化**问题、提出**同幂或同底**项、**倒代换**、化为**e的对数**、**上下同除**等技巧



2. 求多项和的极限时，应想到**合并、夹逼、定积分**，求多项积的极限时，应想到化为**e的对数**，**拆开**各因式

3. 求三角函数的极限时，运用**和差化积**或**回归法**(利用周期性)

4. 从**根号**中提出x时，一定注意是否需要**变号**，这在判断两边极限(或极限是否存在)问题时很关键

5. 对待数列时，应注意数列的特殊性，即非连续而是一点一点的，且$n\in N^*$

6. 证明极限时，想到**单调有界收敛**原理

7. 证明式子时，应想到作**辅助函数**



<div style="page-break-after: always;"></div>

# 部分答案及解析

---

[1. D](#~1 "返回")<a id="1"></a>

​ 此题是经典的$x^\alpha，\alpha趋向正负无穷时是不一样的$

---

[2. D](#~2 "返回")<a id="2"></a>

​ 此处只探讨B和D。

​ B. 由于两者是数列，那么让前者为{0,1,0,1,...}，让后者为{1,0,1,0,...}，它们都无界，而乘积的极限为0

​ D. 原式$\begin{align}=\lim_{n\rightarrow+\infty}\frac{y_n}{\large\frac{1}{x_n}}=0\end{align}$

---

[3. D](#~3 "返回")<a id="3"></a>

---

[4. C](#~4 "返回")<a id="4"></a>

---


