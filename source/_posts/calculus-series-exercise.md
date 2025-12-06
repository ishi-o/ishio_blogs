---
title: "微积分: 级数练习题"
date: 2024-04-17
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 级数练习

## 求常数项级数敛散性

- 对所有收敛的常数项级数，$u_n$严格不为$0$
- 在此基础上，尝试对$|u_n|$进行**比值判别法**或**根值判别法**
  - 均为$\rho<1$收敛、$\rho>1$发散、$\rho=1$未定
  - 当出现**阶乘、双阶乘、多项式乘积、$n$次方**等时考虑比值判别法
  - 当出现**$n^\alpha$次方、多项式的幂次方**时考虑根值判别法
  - $\begin{align}\lim_{n\rightarrow\infty}\sqrt[n]n=1\end{align}、\sqrt[n]{1+n}=e$，两者均**单调递减**
  - 若为**交错级数**，通过这两种方法判断出来一定是**绝对收敛或发散**
- 在上述基础上，考虑与它相似的函数，根据比较判别法的极限形式进行判断(敛散性相同)
  - 常用比较有：**$p$级数、等比级数、三角$\rightarrow1$、积分$\rightarrow$可求积分、含参$\rightarrow$常数**
  - 放缩还可以：移项、拆项$...$
  - $\begin{align}\lim_{n\rightarrow\infty}\sqrt[A]{n^A+Bn^C}\sim n,(A,B,C为常数,A>C)\end{align}$
- 若为交错级数，考虑**莱布尼兹判别法**
  - $|u_n|$单调递减且趋于$0$
- 上述方案需要复习[**极限**](..\..\..\大一上\工数上\md[工数上]\PDF[工数上]\极限[工数上].pdf)，在此基础上，根据**直接求$S_n$极限**或**单调有界收敛原理**考虑$S_n$的敛散性
- 在上述基础上，考虑**柯西收敛原理**
  - 尝试让$|S_{n+p}-S_n|<f(n)$，且$f(n)$足够小，则$\epsilon=f(N)、n>N$时级数收敛
- 常见级数敛散性：
  - $p$级数：$p>1$收敛，否则发散
  - $\begin{align}u_n=\frac1{n^p(\ln n)^q}\end{align}$：$\begin{matrix}p<1,&发散\\p>1,&收敛\\p=1,q\le1,&发散\\p=1,q>1,&收敛\end{matrix}$

## 常数项级数的一般性结论

- 利用**收敛$\pm$收敛$=$收敛**、**收敛$\pm$发散$=$发散**、**发散$\pm$发散$=$未定**的性质
- 若一般项趋于$0$，则添加无限括号与去除无限括号后级数敛散性相同；若未强调一般项，则收敛级数不可随意去掉无限括号、发散级数不可随意添加无限括号
- 利用**基本不等式**进行放缩
- 根据第一条性质，在收敛级数的一般项上加上例如$\begin{align}\frac{1}{n}\end{align}$的发散级数一般项，从而使原级数发散

## 求幂级数和函数

求解$R$的方法：

- 若不缺项，$\begin{align}&R=\lim_{n\rightarrow\infty}\Bigg|\frac{u_n}{u_{n+1}}\Bigg|\end{align}$
- 若缺项，可尝试换元，使其不缺项
- 若无法换元，可用根值判别法，将$x$视为常数，再判断范围
- 得到**收敛区间**后，分别判断两个端点值，求出**收敛域**
- 若第一项或若干项为$0$，可进行移位以便看出和函数

对一个幂级数，首先要求其收敛域，才能求和函数

一般通过逐项定积分或求导，化为最简的$\begin{align}&\sum_{n=0}^\infty x^n\end{align}$的形式，利用它的$\begin{align}S(x)=\frac{1}{1-x}\end{align}$来求解；有时也会借助其它的麦克劳林级数判断

注意在抽取出$x^\alpha$后，$s(x)$定义域可能改变，需要分别讨论

## 求函数的幂级数展开

- 记住常见的麦克劳林级数：
  - $\begin{align}&\frac1{1-x}=\sum_{n=0}^\infty x^n,&x\in(-1,1)\\&\frac1{1+x}=\sum_{n=0}^\infty(-1)^nx^n,&x\in(-1,1)\end{align}$
  - $\begin{align}&e^x=\sum_{n=0}^\infty\frac{x^n}{n!},&x\in R\\&\ln (1+x)=\sum_{n=1}^\infty\frac{(-1)^{n+1}x^n}{n},&x\in(-1,1]\end{align}$
  - $\begin{align}&\sin x=\sum_{n=1}^\infty\frac{(-1)^{n+1}x^{2n-1}}{(2n-1)!},&x\in R\\&\cos x=\sum_{n=0}^\infty\frac{(-1)^nx^{2n}}{(2n)!},&x\in R\\&\arctan x=\int_0^x\frac1{1+x^2}dx+1,&x\in[-1,1],(在两点分别判断收敛)\end{align}$
  - $\begin{align}&(1+x)^\alpha=1+\sum_{n=1}^\infty\frac{\alpha(\alpha-1)\dots(\alpha-n+1)x^n}{n!},&R=1\\&\frac1{\sqrt{1-x^2}}=1+\sum_{n=1}^\infty(-1)^n\frac{(2n-1)!!·x^{2n}}{(2n)!!},&x\in(-1,1)\\&\frac1{\sqrt{1+x^2}}=1+\sum_{n=1}^\infty\frac{(2n-1)!!·x^{2n}}{(2n)!!},&x\in[-1,1]\end{align}$
- 判断展开后级数的收敛域：通过求$R$和收敛区间，以及端点处分别判断，以及原函数的**定义域**来得出；这里的$R$是对$x$而言的，应注意处理换元后的$R$

## 傅里叶级数

### 求傅里叶级数的收敛

- 根据**狄利克雷收敛定理**，一个周期函数可以展开成傅里叶级数形式，但该级数只有连续时才收敛于原函数
- 该级数$=\begin{cases}f(x_0),&x=x_0处连续\\\Large\frac{f(x_0^+)+f(x_0^-)}2\normalsize,&x=x_0处为间断点\end{cases}$
- 应根据这一点来确定傅里叶级数的收敛域

### 求函数的傅里叶级数展开

- 周期完整的函数
  - $\begin{align}f(x)\sim\frac{a_0}2+\sum_{n=1}^\infty(a_n\cos\frac{n\pi}lx+b_n\sin\frac{n\pi}lx)\end{align}$
  - $\begin{align}\begin{cases}\large a_n=\frac1l\int_{-l}^lf(x)\cos\frac{n\pi}lxdx\\\large b_n=\frac1l\int_{-l}^lf(x)\sin\frac{n\pi}lxdx\end{cases}\end{align}$
  - 收敛域为原函数的所有连续点的集合
  - 特别地，当函数为奇函数或偶函数时，只需要求$b_n$或$a_n$，且可将定积分的另一半提成$2$
- 周期不完整的函数
  - 一般只要求展成余弦或正弦级数，只需偶延拓或奇延拓
  - 注意展开后级数的**收敛域**

## 练习

---

$\begin{align}&1.证明\sum_{n=2}^\infty\frac1{n^p(\ln n)^q}的敛散性.\\&1).当q<0时,u_n>\frac1{n^p},若p\le1,则原级数发散\\&2).当p>1时,u_n\le\begin{cases}\frac1{n^{p+q}},&q\le0,&p>1-q时收敛\\\frac1{n^p},&q>0,&收敛\end{cases}\\&故令p'+1<p,由\lim_{n\rightarrow\infty}\frac1{n^{p'}(\ln n)^q}=0<1\\&故u_n=\frac1{n^{p-p'}}·\frac1{n^{p'}(\ln n)^q}<\frac1{n^{p-p'}},原级数收敛.\\&3).当p=1,q\ge0时,u_n非负连续递减,与对应反常积分敛散性相同.\end{align}$

---

$\begin{align}&2.\sum_{n=2}^\infty\frac1{(\ln n)^{\ln n}}\\&[由已知得该级数收敛,则放大.]\\&(\ln n)^{\ln n}>2^{\ln n},u_n<\frac1{2^{\ln n}},故收敛.\end{align}$

---

$\begin{align}&3.\sum_{n=1}^\infty\frac{n^{n-1}}{(2n^2+\ln n+2)^{\frac{n+1}2}}\\&[根值判别法.]\\&\lim_{n\rightarrow\infty}\sqrt[n]{u_n}=\lim_{n\rightarrow\infty}\frac n{\sqrt{2n^2+\ln n+2}}<\lim_{n\rightarrow\infty}\frac nn=1,故收敛.\end{align}$

---

> 交错级数优先考虑莱布尼兹判别法，但无法使用时，考虑$S_n$的敛散性
>
> 由于有奇偶性的分别，此时应先考虑$S_{2n}$，再由$u_n\rightarrow0$推广到$S_n$的敛散性

$\begin{align}&4.\sum_{n=2}^\infty\frac{(-1)^n}{[n+(-1)^n]^p},(p>0)\\&1).当p>1时,|u_n|\sim\frac1{n^p},原级数绝对收敛.\\&2).当p\le1时,|u_n|发散,而S_{2n}=(\frac1{3^p}-\frac1{2^p})+(\frac1{5^p}-\frac1{4^p})+\dots+(\frac1{(2n+1)^p}-\frac1{(2n)^p})\\&每项都小于0,故\{S_{2n}\}单调递减.\\&又S_{2n}=-\frac1{2^p}+(\frac1{3^p}-\frac1{4^p})+\dots+(\frac1{(2n-1)^p}-\frac1{(2n)^p})+\frac1{(2n+1)^p}>-\frac1{2^p}\\&故S_{2n}有下界,故\lim_{n\rightarrow\infty}S_{2n+1}=\lim_{n\rightarrow\infty}S_{2n}+\frac1{(2n+3)^p}=\lim_{n\rightarrow\infty}S_{2n}=S\\&故此时原级数条件收敛.\end{align}$

---

$\begin{align}&5.下列命题正确的是()\\&A.若\sum_{n=1}^\infty a_n收敛,则\sum_{n=1}^\infty\frac{(-1)^n}{\sqrt n}a_n收敛&a_n=\frac{(-1)^n}{\sqrt n}\\&B.若正项级数\sum_{n=1}^{\infty}a_n满足\frac{a_{n+1}}{a_n}<1(n=1,2,...),则\sum_{n=1}^\infty a_n收敛\\&C.若\lim_{n\rightarrow\infty}\frac{a_n}{b_n}=1,则级数\sum_{n=1}^\infty a_n与\sum_{n=1}^\infty b_n同敛散&交错级数\\&D.若a_n\le b_n\le c_n(n\ge1),且\sum_{n=1}^\infty a_n和\sum_{n=1}^\infty c_n都收敛,则\sum_{n=1}^\infty b_n收敛\end{align}$

- 交错级数**不可使用比较判别法的极限形式**判断
- 类似夹逼定理，在两收敛级数中间的级数也收敛

---

$\begin{align}&6.验证y(x)=\sum_{n=0}^\infty\frac{x^{3n}}{(3n)!}满足y''+y'+y=e^x,并求\sum_{n=0}^\infty\frac{x^{3n}}{(3n)!}的和函数\\&对y(x)逐项求导即可验证.\\&由\lambda^2+\lambda+1=0,解得\lambda_{1,2}=\frac{-1\pm\sqrt3i}2\\&故齐次通解为y_1(x)=e^{-\frac12}(C_1\cos\frac{\sqrt3}2x+C_2\sin\frac{\sqrt3}2x)\\&设特解形式为y^*(x)=Ae^x,则A=\frac13\\&由于y(x)在x=0处展开,故求此处的特解,将x=0代入\\&y(x)=y_1(x)+y^*(x)得y(0)=1=C_1+\frac13,y'(0)=0=-\frac{C_1}2+\frac{\sqrt3}2C_2+\frac13\\&\therefore y(x)=e^{-\frac12}(\frac23\cos\frac{\sqrt3}2x)+\frac13e^x\end{align}$

---

$\begin{align}&7.若a_n条件收敛,求幂级数\sum_{n=1}^\infty a_n(x-1)^n的收敛域.\\&解:\\&由a_n条件收敛,得幂级数R=1,收敛区间为(0,2)\\&当x=0时,原级数发散;当x=2时,原级数收敛\end{align}$

---

$\begin{align}&8.幂级数\sum_{n=0}^\infty\frac{2n+1}{n!}x^n,求收敛域、和函数.\\&解:\\&易得收敛域为(-\infty,+\infty)\\&设和函数为S(x),则S(x)=2s(x)-e^x,\\&s(x)=\sum_{n=0}^\infty\frac{(n+1)x^n}{n!},\\&\int_0^xs(x)dx=xe^x,s(x)=(x+1)e^x\\&故S(x)=(2x+1)e^x,x\in(-\infty,+\infty)\end{align}$

---

$\begin{align}&9.求\sum_{n=0}^\infty\frac{(n+1)^2}{n!}.\\&解:\\&设S(x)=\sum_{n=0}^\infty\frac{(n+1)^2}{n!}x^n,则\int_0^xS(x)-1=\sum_{n=0}^\infty\frac{n+1}{n!}x^{n+1}=(x^2+x)e^x\\&故S(x)=(x^2+3x+1)e^x,\therefore S(1)=5e\end{align}$
