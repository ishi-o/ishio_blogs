---
title: "微积分: 反常积分"
date: 2023-12-03
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 反常积分

即广义积分，分为**无穷区间**上与**无界函数**上两种

## 计算

### 无穷区间上

$\begin{align}&\int_a^{+\infty}f(x)dx=\lim_{b\rightarrow+\infty}\int_a^bf(x)dx\end{align}$

拆为两项，其中需要关注$\begin{align}\lim_{x\rightarrow\infty}f(x)\end{align}$一项，如果它**存在**，则积分**收敛**，否则发散

需要熟悉不定积分的求法，利用**牛顿莱布尼茨**公式进行求解，不同之处在于它需要计算一项或两项的极限值

---

**eg:**

$\begin{align}&计算\int_0^{+\infty}\frac{xe^{-x}}{(1+e^{-x})^2}dx\\&解:原式\xlongequal{t=-x}\int_0^{+\infty}\frac{te^t}{(1+e^t)^2}dt=(-\frac{t}{1+e^t})\Bigg|_0^{+\infty}+\int_0^{+\infty}\frac{1}{e^t(1+e^t)}de^t\\&=(\ln\frac{e^t}{(1+e^t)}-\frac{t}{1+e^t})\Bigg|_0^{+\infty}=\ln2\end{align}$

## 证明广义积分等式

一般需要作代换，可以考虑**倒代换**、三角代换等

---

**eg:**

$\begin{align}&证明\int_0^{+\infty}\frac{dx}{1+x^4}=\int_0^{+\infty}\frac{x^2}{1+x^4}dx，并求值\\&证:左边\xlongequal{t=\Large \frac{1}{x}}\int_{+\infty}^0-\frac{dx}{x^2(1+\frac{1}{x^4})}=\int_0^{+\infty}\frac{x^2}{1+x^4}dx=右边\\&\therefore原式=\frac{1}{2}\int_0^{+\infty}\frac{1+x^2}{1+x^4}dx=\frac{1}{2}\int_0^{+\infty}\frac{1+\large\frac{1}{x^2}}{x^2+\large\frac{1}{x^2}}dx=\frac{1}{2}\int_0^{+\infty}\frac{d({x-\large\frac{1}{x}})}{(x-{\large\frac{1}{x}})^2+2}=\\&\frac{\sqrt2}{4}\arctan{\bigg[\frac{\sqrt2}{2}(x-\frac{1}{x})\bigg]}\Bigg|_0^{+\infty}=\frac{\sqrt2\pi}{4}\end{align}$
