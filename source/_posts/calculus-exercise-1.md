---
title: "微积分: 练习题"
date: 2023-12-11
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 极限

## 函数极限

$\begin{align}&1.\lim_{x\rightarrow 0}\frac{\tan x-\sin x}{x\ln(1+\sin^2x)}\\&解:原式=\lim_{x\rightarrow0}\frac{\tan x-\sin x}{x^3}=\frac12\end{align}$

---

$\begin{align}&2.\lim_{x\rightarrow0}\frac{(x-\sin x)e^{-x^2}}{\sqrt{1-x^3}-1}\\&解:原式=\lim_{x\rightarrow0}\frac{\frac16x^3}{-\frac12x^2}=-\frac13\end{align}$

---

$\begin{align}&3.\lim_{x\rightarrow0}\frac{\ln(e^{\sin x}+\sqrt[3]{1-\cos x})-\sin x}{\arctan(4\sqrt[3]{1-\cos x})}\\&解:原式=\lim_{x\rightarrow0}\frac{e^{\sin x}+\sqrt[3]{1-\cos x}-1-\sin x}{4\sqrt[3]{1-\cos x}}\\&=\lim_{x\rightarrow0}\frac{1+\sin x+\frac{\sin^2x}2-1-\sin x+o(x^2)}{4\sqrt[3]{1-\cos x}}+\frac14\\&=\frac14\end{align}$

---

$\begin{align}&4.\lim_{x\rightarrow0}\frac{1-\cos x\sqrt{\cos 2x}\sqrt[3]{\cos 3x}}{x^2}\\&解:原式=\lim_{x\rightarrow0}\frac{1-\cos x+\cos x(1-\sqrt{\cos 2x}+\sqrt{\cos 2x}(1-\sqrt[3]{\cos 3x}))}{x^2}\\&=\lim_{x\rightarrow0}\frac{\frac12x^2+\frac12(1-\cos 2x)+\frac13(1-\cos 3x)}{x^2}=\lim_{x\rightarrow0}\frac{\frac12x^2+x^2+\frac32x^2}{x^2}=3\end{align}$

---

$\begin{align}&5.\lim_{x\rightarrow\frac\pi2}\frac{(1-\sqrt{\sin x})(1-\sqrt[3]{\sin x})\cdots(1-\sqrt[n]{\sin x})}{(1-\sin x)^{n-1}}\\&=\lim_{x\rightarrow\frac\pi2}\frac{\frac1{n!}(1-\sin x)^{n-1}}{(1-\sin x)^{n-1}}=\frac1{n!}\end{align}$

---

$\begin{align}&6.\lim_{x\rightarrow0}\frac{\sqrt{\frac{1+x}{1-x}}\sqrt[4]{\frac{1+2x}{1-2x}}\sqrt[6]{\frac{1+3x}{1-3x}}\cdots\sqrt[2n]{\frac{1+nx}{1-nx}}-1}{3\pi\arctan x-(x^2+1)\arctan^3x},n为正整数\\&解:原式\xlongequal{\ln极限逆用}\lim_{x\rightarrow0}\frac{\ln(\sqrt{\frac{1+x}{1-x}}\sqrt[4]{\frac{1+2x}{1-2x}}\sqrt[6]{\frac{1+3x}{1-3x}}\cdots\sqrt[2n]{\frac{1+nx}{1-nx}})}{3\pi\arctan x-(x^2+1)\arctan^3x}\\&=\lim_{x\rightarrow0}\frac{\frac12(\frac2{1-x}-2)+\frac14(\frac{2}{1-2x}-2)+\cdots}{3\pi x-x^5-x^3}\\&=\lim_{x\rightarrow0}\frac{\frac{x}{1-x}+\frac x{1-2x}+\cdots}{3\pi x-x^5-x^3}=\lim_{x\rightarrow}\frac{nx}{3\pi x}=\frac n{3\pi}\end{align}$

---

$\begin{align}&7.\lim_{x\rightarrow\infty}\frac{(\int_0^xe^{u^2}\mathrm du)^2}{\int_0^xe^{2u^2}\mathrm du}\\&解:原式=\lim_{x\rightarrow\infty}\frac{2\int_0^xe^{u^2}\mathrm du}{e^{x^2}}=2\lim_{x\rightarrow\infty}\frac{e^{x^2}}{2xe^{x^2}}=0\end{align}$

---

$\begin{align}&8.\lim_{x\rightarrow0}\left(\frac{e^x+e^{2x}+\cdots+e^{nx}}n\right)^{\Large\frac ex}\\&解:原式=e^{\lim_{x\rightarrow0}{\Large\frac ex}\ln(1+\frac{e^x+e^{2x}+\cdots+e^{nx}}n-1)}\\&幂上极限为\lim_{x\rightarrow0}\frac ex\frac{e^x+e^{2x}+\cdots+e^{nx}-n}n\\&=\lim_{x\rightarrow0}\frac ex\frac{x+2x+\cdots+nx}n=\frac{(n+1)e}{2}\\&故原极限=e^{\Large\frac{(n+1)e}2}\end{align}$

---

$\begin{align}&9.若f(x)在点x=a处可导,且f(a)\ne0,则\lim_{n\rightarrow\infty}\left(\frac{f\left(a+\frac1n\right)}{f(a)}\right)^n\\&原式=e^{\lim_\limits{n\rightarrow\infty}n\ln(\frac{f\left(a+\frac1n\right)}{f(a)})}\\&幂上极限=\lim_{n\rightarrow\infty} n\frac{f\left(a+\frac1n\right)-f(a)}{f(a)}=\frac{f'(a)}{f(a)}\\&故原极限=e^{\Large\frac{f'(a)}{f(a)}}\end{align}$

---

$\begin{align}&10.\lim_{x\rightarrow\infty}e^{-x}\left(1+\frac1x\right)^{x^2}\\&解:原式=e^{\lim_\limits{x\rightarrow\infty}-x+x^2\ln(1+\frac1x)}=e^{\lim_\limits{x\rightarrow\infty}(-x+x^2(\frac1x-\frac1{2x^2}+o(x^2)))}=e^{-\frac12}\end{align}$

---

$\begin{align}&11.\lim_{x\rightarrow0}\frac{\sin^2x-x^2\cos^2x}{x^2\sin^2x}\\&解:原式=\lim_{x\rightarrow0}\frac{(\sin x-x+x(1-\cos x))(\sin x-x+x(1+\cos x))}{x^4}\\&=\lim_{x\rightarrow0}\frac{(-\frac16x^3+\frac12x^3)(-\frac16x^3+2x)}{x^4}=-\frac13+1=\frac23\end{align}$

---

$\begin{align}&12.若f(1)=0,f'(1)存在,则极限\lim_{x\rightarrow0}\frac{f(\sin ^2x+\cos x)\tan 3x}{(e^{x^2}-1)\sin x}\\&解:原式=\lim_{x\rightarrow0}\frac{f(1+\sin^2x+\cos x-1)-f(1)}{\sin^2x+\cos x-1}·\frac{\tan 3x(\sin^2x+\cos x-1)}{(e^{x^2}-1)\sin x}\\&=f'(1)·\lim_{x\rightarrow0}\frac{3x(x^2-\frac12x^2)}{x^3}=\frac32f'(1)\end{align}$

---

$\begin{align}&13.设f(x)有二阶连续导数,且f(0)=f'(0)=0,f''(0)=6,则\lim_{x\rightarrow0}\frac{f(\sin ^2x)}{x^4}\\&解:原式=\lim_{x\rightarrow0}\frac{\sin2x·f'(\sin^2x)}{4x^3}=\lim_{x\rightarrow0}\frac{f'(\sin^2x)-f'(0)}{2\sin^2x}=3\end{align}$

---

$\begin{align}&14.设f(x)在点x=1附近有定义,且在点x=1处可导,f(1)=0,f'(1)=2,求\lim_{x\rightarrow0}\frac{f(\sin x+\cos x)}{x^2+x\tan x}\\&解:原式=\lim_{x\rightarrow0}\end{align}$

---

$\begin{align}&12.\end{align}$

---

$\begin{align}&12.\end{align}$

---

$\begin{align}&12.\end{align}$

---

$\begin{align}&12.\end{align}$

---

$\begin{align}&12.\end{align}$

## 数列极限

# 一元积分学

## 不定积分

### 常见微分等式

- $\begin{align}\mathrm d\left(\ln(x+\sqrt{\pm k^2+x^2})\right)=\frac1{\sqrt{\pm k^2+x^2}}\end{align}$
- $\begin{align}\mathrm d\left(x\pm\frac kx\right)=1\mp\frac k{x^2}\end{align}$，由于$\begin{align}\left(x\pm\frac kx\right)^2=x^2+\frac{k^2}{x^2}\pm2k\end{align}$，遇到**分子分母分别为二次、四次多项式，且不包含奇数幂次**时，可考虑用分子凑出该微分，而分母通过**配方**化作该式的平方，实现降幂
- $\begin{align}\mathrm d(\sin x\pm\cos x)=\cos x\mp\sin x\end{align}$，在处理$\sin x\cos x$时很有效

### 常见计算技巧

- 更换积分对象：
  - 分部积分：$\begin{align}\int u\mathrm dv=uv-\int v\mathrm du\end{align}$，若分部后的积分容易算，采用分部积分
  - 例一：分母带平方或其它幂次方，可尝试用分母凑微分，实现分母降幂
- 三角代换或三角公式：
  - 包含$\begin{align}\sqrt{k^2+x^2}\mathrm dx\end{align}$的表达式，可令$x=k\tan t$，则$\mathrm dx=k\sec^2t\,\mathrm dt、\sqrt{k^2+x^2}=k\sec t$
    - 此外，通过直角三角形可知，$\begin{align}\sin t=\frac x{\sqrt{1+x^2}}、\cos t=\frac1{\sqrt{1+x^2}}\end{align}$
    - 例如$\begin{align}&\int\frac{\mathrm dx}{\sqrt{1+x^2}}\xlongequal{x=\tan t}\int\sec t\,\mathrm dt=\int\frac{\mathrm d(\sin t)}{1-\sin^2t}\\&=\ln(\sqrt{\frac{1+\sin t}{1-\sin t}})+C=\ln(x+\sqrt{1+x^2})+C\end{align}$
  - 包含$\sqrt{k^2-x^2}\mathrm dx$的表达式，可令$x=k\sin t$或$x=k\cos t$
  - 包含$\sqrt{x^2-k^2}\mathrm dx$的表达式，可令$x=k\sec t$，则$\begin{align}\mathrm dx=k\frac{\sin x}{\cos^2x}、\sqrt{x^2-k^2}=k\tan x\end{align}$
  - 通过倍角公式，当分母是关于三角函数的多项式时，将其变为单项式
  - 通过辅助角公式，将所有$\sin x\pm\cos x、\sin x\cos x$统一为一个含有特定角度的三角函数；或者取$\sin x\pm\cos x$化为**微分**$\mathrm d(\pm\sin x-\cos x)$，将剩余的$\sin x\cos x$化为$(\pm\sin x-\cos x)^2+k$的形式后，令$t=\pm\sin x-\cos x$
- 微分代换：分母根式借助**配方**和微分等式代换，但一般没有三角代换常用
  - $\begin{align}\mathrm d(\arcsin x)=\frac1{\sqrt{1-x^2}}\end{align}$
  - $\begin{align}\mathrm d\left(\ln(x+\sqrt{1+x^2})\right)=\frac1{\sqrt{1+x^2}}\end{align}$

- 区间再现：实际上是定积分的技巧，包括正弦、余弦函数的华里士公式；体现在不定积分上，实际上就是通过分部积分导出的含有重复待求积分的式子。当幂数不常见时，可通过该方法求出递推式z
- 欧拉变换：设被积函数为关于$x$的，包含$\sqrt{ax^2+bx+c}$根式的函数
  - 欧拉第一代换：若$a>0$，则可令整个根式$=t\pm\sqrt ax$
  - 欧拉第二代换：若$c>0$，则可令整个根式$=xt\pm\sqrt c$
  - 欧拉第三代换：若可十字相乘(有两不同实根)，可令整个根式$=t(x-x_1)$，其中$x_1$为其中一个实根

- 万能代换将三角函数换为有理式：令$\begin{align}t=\tan\frac x2\end{align}$，则$\begin{align}\sin x=\frac{2t}{1+t^2}、\cos t=\frac{1-t^2}{1+t^2}、\mathrm dx=\frac{2}{1+t^2}\mathrm dt\end{align}$
- 积分抵消：仍是通过分部积分$\begin{align}uv=\int u\mathrm dv+\int v\mathrm du\end{align}$，若两个不定积分都不好求，但刚好能凑出两个该形式的不定积分，则可合成一个$uv$；有时有两个不相关的不定积分，也可尝试分别分部积分，分出的部分可能是有关的

### 题集

典型分母降幂

$\begin{align}&1.\int\frac{e^{-\sin x}\sin 2x}{(1-\sin x)^2}\mathrm dx\\&解:原式=2\int e^{-\sin x}\sin x\ \mathrm d\left(\frac1{1-\sin x}\right)=2\left(\frac{e^{-\sin x}\sin x}{1-\sin x}-\int\frac{\mathrm d(e^{-\sin x}\sin x)}{1-\sin x}\right)\\&=2\left(\frac{e^{-\sin x}\sin x}{1-\sin x}-\int e^{-\sin x}\mathrm d(\sin x)\right)=\frac{2e^{-\sin x}}{1-\sin x}+C\end{align}$

---

$\begin{align}&2.\int\frac{x^2e^x}{(x+2)^2}\mathrm dx\\&解:原式=-\int x^2e^x\mathrm d\left(\frac1{x+2}\right)=-\left(\frac{x^2e^x}{x+2}-\int\frac{\mathrm d(x^2e^x)}{x+2}\right)=\frac{x-2}{x+2}e^x+C\end{align}$

---

$\begin{align}&3.\int\frac{(2x+1)e^{-2x}}{x^2}\mathrm dx\\&解:原式=-\int(2x+1)e^{-2x}\mathrm d\left(\frac1{x}\right)=-\left(\frac{(2x+1)e^{-2x}}x-\int\frac{\mathrm d((2x+1)e^{-2x})}x\right)=-\frac{e^{-2x}}x+C\end{align}$

---

$\begin{align}&4.\int\frac{x\ln(x+\sqrt{1+x^2})}{(1+x^2)^2}\mathrm dx\\&原式=-\frac12\int\ln(x+\sqrt{1+x^2})\mathrm d\left(\frac1{1+x^2}\right)\\&=-\frac12\left(\frac{\ln(x+\sqrt{1+x^2})}{1+x^2}-\int\frac{\mathrm d\left(\ln(x+\sqrt{1+x^2})\right)}{1+x^2}\right)\\&=-\frac12\left(\frac{\ln(x+\sqrt{1+x^2})}{1+x^2}-\int\frac{\mathrm dx}{(1+x^2)^{\frac32}}\right)\xlongequal{三角代换}\frac{x\sqrt{1+x^2}-\ln(x+\sqrt{1+x^2})}{2(1+x^2)}+C\end{align}$

---

$\begin{align}&5.\int\frac{\ln(x+\sqrt{1+x^2})}{(1+x^2)^\frac32}\mathrm dx\\&解:原式=\int\ln(x+\sqrt{1+x^2})\mathrm d\left(\frac x{(1+x^2)^\frac12}\right)=\frac{x\ln(x+\sqrt{1+x^2})}{(1+x^2)^\frac12}-\int\frac{x\mathrm dx}{1+x^2}\\&=\frac{x\ln(x+\sqrt{1+x^2})}{\sqrt{1+x^2}}-\frac12\ln(1+x^2)+C\end{align}$

---

积分抵消，最后一步可不写

$\begin{align}&6.\int\left(1+x-\frac1x\right)e^{x+\frac1x}\mathrm dx\\&解:原式=\int\left[1+x\left(1-\frac1{x^2}\right)\right]e^{x+\frac1x}\mathrm dx=\int e^{x+\frac1x}\mathrm dx+\int x\mathrm d(e^{x+\frac1x})\\&=I+xe^{x+\frac1x}-I=xe^{x+\frac1x}+C\end{align}$

---

$\begin{align}&7.\int\frac{x+\sin x}{1+\cos x}\mathrm dx\\&解:原式=\int\frac{x+\sin x}{2\cos^2{\large\frac x2}}\mathrm dx=\int\frac{x\,\mathrm dx}{2\cos^2{\large\frac x2}}+\int\tan\frac x2\mathrm dx\\&=\int x\,\mathrm d\left({\tan\frac x2}\right)+\int\tan\frac x2\mathrm dx\xlongequal{分部积分}x\tan\frac x2+C\end{align}$

---

$\begin{align}&8.\int e^{2x}(1+\tan x)^2\mathrm dx\\&解一:原式\xlongequal{x=\arctan t}\int e^{2\arctan t}\frac{(1+t)^2}{1+t^2}\mathrm dt=\int e^{2\arctan t}\mathrm dt+\int e^{2\arctan t}\frac{2t}{t^2+1}\mathrm dt\\&=\int e^{2\arctan t}\mathrm dt+\int t\mathrm d\left(e^{2\arctan t}\right)=te^{2\arctan t}+C=\tan x·e^{2x}+C\\&解二:原式=\int e^{2x}\frac{1+2\sin x\cos x}{\cos^2x}\mathrm dx=\int e^{2x}\mathrm d(\tan x)+2\int e^{2x}\tan x\mathrm dx\\&=\int e^{2x}\mathrm d(\tan x)+\int\tan x\,\mathrm d\left(e^{2x}\right)=\tan x·e^{2x}+C\end{align}$

---

$\begin{align}&9.\int e^{\sin x}\frac{x\cos^3x-\sin x}{\cos^2x}\mathrm dx=\int x\mathrm d\left(e^{\sin x}\right)-\int e^{\sin x}\frac{\sin x}{\cos^2x}\mathrm dx\\&\xlongequal{分母降幂}\int x\mathrm d\left(e^{\sin x}\right)-\int e^{\sin x}\mathrm d\left(\frac1{\cos x}\right)\\&=\int x\mathrm d\left(e^{\sin x}\right)-e^{\sin x}\sec x+\int\sec x\mathrm d\left(e^{\sin x}\right)\\&=\int x\mathrm d\left(e^{\sin x}\right)-e^{\sin x}\sec x+\int e^{\sin x}\mathrm dx=e^{\sin x}(x-\sec x)+C\end{align}$

---

$\begin{align}&10.\int x\arctan x\ln(1+x^2)\mathrm dx\\&解:原式=\frac12\int\arctan x\ln(1+x^2)\mathrm d\left(1+x^2\right)\\&=\frac12\left(\arctan x\ln(1+x^2)(1+x^2)-\int(1+x^2)\mathrm d\left(\arctan x\ln(1+x^2)\right)\right)\\&=\frac12\left(\arctan x\ln(1+x^2)(1+x^2)-\int\ln(1+x^2)\mathrm dx-\int2x\arctan x\mathrm dx\right)\\&=\frac12\left(\arctan x\ln(1+x^2)(1+x^2)-\left(x\ln(1+x^2)-\int\frac{2x^2}{1+x^2}\mathrm dx\right)-2\int x\arctan x\mathrm dx\right)\\&\int x\arctan x\mathrm dx=\frac12\int\arctan x\mathrm d\left(x^2+1\right)=\frac{\arctan x·(1+x^2)-x}2+C\\&\int\frac{2x^2}{1+x^2}\mathrm dx=2(x-\arctan x)+C\\&综上,原式=\frac{\arctan{x[(x^2+1)\ln(1+x^2)-(x^2+3)]-x\ln(1+x^2)+3x}}2+C\end{align}$

---

$\begin{align}&11.\int\frac{x^2+1}{x^4+1}\mathrm dx\\&解:原式=\int\frac{1+\frac1{x^2}}{x^2+\frac1{x^2}}\mathrm dx=\int\frac{\mathrm d\left(x-\frac1x\right)}{(x-\frac1x)^2+2}=\frac1{\sqrt2}{\arctan\frac{x-\frac1x}{\sqrt2}}+C\end{align}$

---

$\begin{align}&12.\int\frac{x^2\pm1}{1+kx^2+x^4}\mathrm dx\\&解:原式=\int\frac{1\pm\frac1{x^2}}{\frac1{x^2}+k+x^2}\mathrm dx=\int\frac{\mathrm d(x\mp\frac1x)}{(x\mp\frac1x)^2+k\pm2}\\&\xlongequal{t=(x\mp\frac1x)\large\frac1{\sqrt{k\pm2}}}\frac1{\sqrt{k\pm2}}\int \frac{\mathrm dt}{t^2+1}=\frac1{\sqrt{k\pm2}}\arctan\frac{x\mp\frac1x}{\sqrt{k\pm2}}+C\end{align}$

---

$\begin{align}&13.\int\frac1{\sin^3x+\cos^3x}\mathrm dx,\frac14<x<\frac12\\&解:原式\xlongequal{立方和公式}\int\frac1{(\sin x+\cos x)(1-\sin x\cos x)}\mathrm dx\\&=\int\frac{\sin x+\cos x}{(1+2\sin x\cos x)(1-\sin x\cos x)}\mathrm dx\\&=\int\frac{\mathrm d(\sin x-\cos x)}{(2-(\sin x-\cos x)^2)(1-\frac{1-(\sin x-\cos x)^2}2)}\\&\xlongequal{t=\sin x-\cos x}2\int\frac1{(2-t^2)(1+t^2)}\mathrm dt=\frac23\arctan t+\frac1{3\sqrt2}\ln(\frac{\sqrt2+t}{\sqrt2-t})+C\\&=\frac23\arctan(\sin x-\cos x)+\frac1{3\sqrt2}\ln(\frac{\sqrt2+\sin x-\cos x}{\sqrt2+\cos x-\sin x})+C\end{align}$

---

$\begin{align}&14.\int\frac{\mathrm dx}{x\sqrt{x^2-2x-3}}\\&解:原式=\int\frac{\mathrm dx}{x^2\sqrt{1-\frac2x-\frac3{x^2}}}=-\frac1{\sqrt3}\int\frac{\mathrm d\left(\frac{\sqrt3}x+\frac1{\sqrt3}\right)}{\sqrt{-(\frac{\sqrt3}x+\frac1{\sqrt3})^2+\frac43}}\\&=-\frac1{\sqrt3}\int\frac{\mathrm d(\frac{\sqrt3}2(\frac{\sqrt3}x+\frac1{\sqrt3}))}{\sqrt{1-\frac34(\frac{\sqrt3}x+\frac1{\sqrt3})^2}}=-\frac{\arcsin(\frac{\sqrt3}2(\frac{\sqrt3}x+\frac1{\sqrt3}))}{\sqrt3}+C\end{align}$

---

$\begin{align}&15.\int\sqrt\frac{1-x}{1+x}\mathrm dx\\&解:原式\xlongequal{t=\sqrt{\frac{1-x}{1+x}}}-4\int\frac{t^2}{(1+t^2)^2}\mathrm dt=2\int t\mathrm d\left(\frac1{1+t^2}\right)\\&=2\left(\frac t{1+t^2}-\int\frac{\mathrm dt}{1+t^2}\right)=2\left(\frac t{1+t^2}-\arctan t\right)+C\\&=\sqrt{(1+x)(1-x)}-2\arctan \sqrt{\frac{1-x}{1+x}}+C\end{align}$

---

$\begin{align}&16.I(n)=\int\frac{1}{(a^2+x^2)^n}\mathrm dx,\ n为正整数\\&解:原式=\frac x{(a^2+x^2)^n}+2n\int\frac{x^2}{(a^2+x^2)^{n+1}}\mathrm dx\\&=\frac x{(a^2+x^2)^n}+2n\int\frac{1}{(a^2+x^2)^n}\mathrm dx-2na^2\int\frac{1}{(a^2+x^2)^{n+1}}\mathrm dx\\&故有递推式:2na^2·I(n+1)=(2n-1)·I(n)+\frac x{(a^2+x^2)^n},\ 当n=1时,I(1)=\arctan x+C\\&偏移后:I(n)=\frac{(2n-3)·I(n-1)+\Large \frac x{(a^2+x^2)^{n-1}}}{2(n-1)a^2}\\&特别地,\ 当n=2时,\ I(2)=\frac{\arctan x+\Large \frac x{a^2+x^2}}{2a^2}+C\end{align}$

$\begin{align}&由题15、16,形如\frac{ax^2+bx+c}{(k^2+x^2)^2}的被积函数可拆分成三部分计算\end{align}$

---

$\begin{align}&17.I=\int\sqrt{k\pm x^2}\mathrm dx,\ k\pm x^2\ge0,k>0\\&解:I=x\sqrt{k\pm x^2}-\int x\frac{\pm x}{\sqrt{k\pm x^2}}\mathrm dx=x\sqrt{k\pm x^2}-I+k\int\frac{\mathrm dx}{\sqrt{k\pm x^2}}\\&故I=\frac12\left(x\sqrt{k+ x^2}+k\ln(x+\sqrt{k+ x^2})\right)+C&k+x^2时\\&或I=\frac12\left(x\sqrt{k- x^2}+k\arcsin(\frac x{\sqrt k})\right)+C&k-x^2时\end{align}$

---

# 中值定理

## 复习

### 常用定理定义

- 零点定理：若$f(x)$在$[a,b]$内连续，且$f(a)f(b)<0$，则至少存在一点$x_0\in(a,b)$，使$f(x_0)=0$；零点定理可视为介值定理的特例
- 唯一零点定理：在**零点定理成立**的条件下，若$f(x)$可导且$f'(x)\ne0$，则有且仅有一点使$f(x_0)=0$
- 介值定理：设$f(x)$在$(a,b)$内连续，且最大值为$M$，最小值为$m$，若有$m\le A\le M$，则至少存在一点$x_0\in(a,b)$，使$f(x_0)=A$
- 罗尔中值定理：若$f(x)$在**闭区间**$[a,b]$内**函数连续**、**开区间**$(a,b)$内**函数可导**，且$f(a)=f(b)$，则在**开区间**$(a,b)$内存在一点$\xi$，使$f'(\xi)=0$；$\rm Rolle$中值定理是$\rm Largrange$中值定理的特例
- 拉格朗日中值定理：若$f(x)$在**闭区间**$[a,b]$内**函数连续**、**开区间**$(a,b)$内**函数可导**，则在**开区间**$(a,b)$内存在一点$\xi$，使$\begin{align}f'(\xi)=\frac{f(a)-f(b)}{a-b}\end{align}$；$\rm Largrange$中值定理是$\rm Taylor$中值定理的特例
- 泰勒中值定理：若$f(x)$在$x_0$的$(a,b)$邻域内**有$n+1$阶导**，则存在介于$x$和$x_0$之间的中值$\xi$($x\in (a,b)$)，使其泰勒展开后拉格朗日型余项为$\begin{align}R_n=\frac{f^{(n+1)}(\xi)}{(n+1)!}(x-x_0)^{n+1}\end{align}$
- 积分中值定理：若有$\begin{align}\int_a^bf(x)dx=0\end{align}$，则存在一点$x_0\in(a,b)$，使$f(x_0)=0$

### 构造辅助函数(`->`[微分方程](#微分方程))

通过中值定理证明给定方程$F(x)=0$的题型中，寻找辅助函数的过程相当于解出微分方程后移项，常见构造如下(可不记，相当于初等微分方程)：

- 构造的$G(x)$可任意加常数$C$，该**常数根据所给条件**，最终必须凑出$G(a)=G(b)$，一般是$0$
- 形如$f'(x)+P(x)f(x)=0$的待证方程，通常构造$\begin{align}G(x)=f(x)e^{\int P(x)dx}+C\end{align}$
- 形如$f'(x)+P(x)f(x)=Q(x)$的待证方程，通常构造$\begin{align}G(x)=f(x)e^{\int P(x)dx}-\int e^{\int P(x)dx}Q(x)dx+C\end{align}$
- 形如$f''(x)+pf'(x)+qf(x)=Q(x)$的待证方程，通常将$F(x)=f'(x)-Af(x)$作为整体降阶，相当于**解差分方程**，即$\lambda^2+p\lambda+q=0$的解分别作为$A、P$，得到等价的降阶方程$F'(x)+PF(x)=Q(x)$；类似的三对角行列式、递推数列也是差分方程

以上是记忆型的知识点，事实上只要**假设方程$F(x)=0$恒成立，解出$f(x)$的通解**，再**通过已知条件**设定一个特解，将**所有项移到一边**，就能得到辅助函数：

- 与关于$f(x)$微分方程的联系：如此构造出的$\begin{align}G(x)=f(x)e^{\int P(x)dx}-\int e^{\int P(x)dx}Q(x)dx=C\end{align}$，移项后求出的$f(x)$就是熟悉的公式$\begin{align}f(x)=e^{-\int P(x)dx}\left(\int e^{\int P(x)dx}Q(x)dx+C\right)\end{align}$；所以解出$f(x)$，辅助函数也就得到了
- 和微分方程的区别：微分方程恒成立，而中值定理往往是只有单中值或双中值满足给定方程，相当于求特解；反过来说，如果能保证构造出的$G(x)$恒等于常数，那么其中的$f(x)$就是通解了

## 题集

简单构造：

$\begin{align}&1.设函数f(x)在[0,1]上连续,在(0,1)内可微,且\end{align}$
$$
f(0)=f(1)=0,f\left(\frac12\right)=1
$$
$\begin{align}&证明:(1)存在\xi\in\left(\frac12,1\right)使得f(\xi)=\xi\\&(2)存在\eta\in(0,\xi)使得f'(\eta)=f(\eta)-\eta+1\\&证:(1)令F(x)=f(x)-x,则F\left(\frac12\right)=\frac12,F(1)=-1,\\&由零点定理,必然存在一点\xi\in\left(\frac12,1\right)使F(\xi)=0,证毕\\&[2](草稿纸过程:容易解得f(x)=e^{-x}(xe^{-x}+C),即辅助函数G(x)=f(x)e^x-xe^{-x})\\&令G(x)=f(x)e^x-xe^{-x},则G(0)=0,G(\xi)=0,由罗尔中值定理,存在\eta\in(0,\xi)使G'(\eta)=0,证毕\end{align}$

---

双中值：

$\begin{align}&2.设f(x)在[0,1]上连续,f(x)在(0,1)上可导,且f(0)=0,f(1)=1\\&证明:(1)存在x_0\in(0,1),使f(x_0)=2-3x_0\\&(2)存在\xi,\eta \in(0,1),且\xi\ne\eta,使[1+f'(\xi)][1+f'(\eta)]=4\\&证:(1)令F(x)=f(x)+3x-2,则F(0)=-2,F(1)=2,\\&根据零点定理,存在x_0\in(0,1)使F(x_0)=0,证毕\\&[2](双中值问题自然想法是拉中定理分区间)\\&在(0,x_0)区间上,拉中有\frac{2-3x_0-f(0)}{x_0}=f'(\xi),\xi\in(0,x_0)\\&在(x_0,1)区间上,同理有\frac{1-(2-3x_0)}{1-x_0}=f'(\eta),\eta\in(x_0,1)\\&代入得证,证毕\end{align}$

如果无视第一问给的条件，则要逆向推导：

$\begin{align}&(2)设在(0,1)上有一点x_1,使得拉中有:\\&\frac{f(x_1)-f(0)}{x_1}=f'(\xi),\frac{f(1)-f(x_1)}{1-x_1}=f'(\eta),则满足\xi\ne\eta\\&要使结论成立,则解出f(x_1)=2-3x_1成立,由(1)得总存在x_0使该式成立\\&则令x_1=x_0,故总存在...证毕\end{align}$

---

遇定积分可先通过积分中值定理得出一个零点，后续几问都相当于求解微分方程

$\begin{align}&3.设函数f(x)在[a,b]上连续,在(a,b)内二阶可导,且\end{align}$
$$
f(a)=f(b)=0,\int_a^bf(x)dx=0
$$
$\begin{align}&(1)证明存在互不相同的点x_1,x_2\in(a,b),使得f'(x_i)=f(x_i),i=1,2;\\&(2)证明存在\xi\in(a,b),\xi\ne x_i,i=1,2,使得f''(\xi)=f(\xi);\\&(3)证明存在\eta\in(a,b),\eta\ne x_i,i=1,2,使得f''(\eta)-3f'(\eta)+2f(\eta)=0\\&证:[1](积分中值定理)令F(x)=\int_a^x f(x)dx,则F(a)=0,F(b)=0,故存在一点x_0使得F'(x_0)=f(x_0)=0\\&令G(x)=f(x)e^{-x},则G(a)=G(x_0)=G(b)=0,则存在两点x_1\ne x_2,使f'(x_i)=f(x_i),i=1,2\\&其中x_1\in(a,x_0),x_2\in(x_0,b),证毕\\&[2](\lambda^2-1=0\Rightarrow f''(x)-f'(x)+f'(x)-f(x)=0)\\&令H(x)=e^{x}(f'(x)-f(x)),则H(x_1)=H(x_2)=0,故存在\xi\in(x_1,x_2)使H'(\xi)=0,证毕\\&[3](\lambda^2-3\lambda+2=0\Rightarrow f''(x)-f'(x)+2(f'(x)-f(x))=0)\\&令K(x)=e^{2x}(f'(x)-f(x)),...,证毕\end{align}$

---

给出若干低阶导，求高阶导；或给出若干定积分，求本身或其导，都有可能考察泰勒中值定理

通过**待定系数法**求出辅助函数的常量值，运用多次罗尔定理也可得到结论(又是通过微分方程，但是最高有三阶导)

$\begin{align}&4.设函数f(x)在闭区间[-1,1]上具有连续的三阶导数,且f(-1)=0,f(1)=1,f'(0)=0\\&求证:在开区间(-1,1)内至少存在一点\xi,使f'''(\xi)=3\\&证法一:在x=0点处泰勒展开,f(x)=f(0)+f'(0)x+\frac{f''(0)x^2}2+\frac{f'''(\xi)x^3}6,\xi介于x和0之间\\&则\begin{cases}f(-1)=0=f(0)-f'(0)+\frac{f''(0)}2-\frac{f'''(\xi_1)}6\\f(1)=1=f(0)+f'(0)+\frac{f''(0)}2+\frac{f'''(\xi_2)}6\end{cases}\\&得f(1)-f(-1)=1=2\left(f'(0)+\frac{f'''(\xi_1)+f'''(\xi_2)}6\right)\\&即\frac{f'''(\xi_1)+f'''(\xi_2)}2=3,由于\xi_1,\xi_2\in(-1,1),设在该区间内f'''(x)最小值为m,最大值为M\\&则m\le f'''(\xi_1),f'''(\xi_2)\le M,故m\le\frac{f'''(\xi_1)+f'''(\xi_2)}2\le M\\&由介值定理,必存在一点\xi\in(-1,1),使f'''(\xi)=\frac{f'''(\xi_1)+f'''(\xi_2)}2=3,证毕\end{align}$

$\begin{align}&证法二:[草稿纸过程]\\&由结论出发,令F''(x)=f''(x)-3x+A\\&再积分令F'(x)=f'(x)-\frac32x^2+Ax+B\\&再积分F(x)=f(x)-\frac{x^3}2+\frac A2x^2+Bx+C,\\&为了使其满足罗尔定理,令F(-1)=F(1)=F(0)=0&\exist两点使F'(x_1)=F'(x_2)=0\\&令F'(0)=F'(x_1)=F'(x_2)=0&\exist两点使F(x_3)=F(x_4)=0\\&求出A、B、C(即使包含未知的函数值也可,因为连续)\\&得到的F(x)就是辅助函数,通过多次罗尔定理得证\end{align}$

---

含变限积分的函数不需要继续积分，可借助积分中值定理和分部积分，因为只需要额外的一点

$\begin{align}&5.设函数f(x)在[0,1]上具有连续导数,且\int_0^1f(x)dx=\frac52,\int_0^1xf(x)dx=\frac32\\&证明:存在\xi\in(0,1),使f'(\xi)=3\\&证:[草稿纸过程]\\&同样为了使用多次罗尔定理,令F'(x)=f(x)-3x+A\\&F(x)=\int_0^xf(t)dt-\frac32x^2+Ax+B\\&令F(0)=F(1)=0解出A,B后,此时还需要找到一点\\&可以使用积分中值定理,只需证\int_0^1F(x)dx=0,则存在一点使F(x_0)=0,且x_0\in(0,1)\\&由分部积分,原定积分=xF(x)\bigg|_0^1-\int_0^1xF'(x)dx=F(1)-\int_0^1x(f(x)-3x+A)\\&确实等于0,则可通过三次罗尔定理得证\end{align}$

---

$\begin{align}&6.设函数f(x)在(-\infty,+\infty)上具有二阶导数,并且\end{align}$
$$
f''(x)>0,\lim_{x\rightarrow+\infty}f'(x)=\alpha>0,\lim_{x\rightarrow-\infty}f'(x)=\beta<0
$$
$\begin{align}&且存在一点x_0,使得f(x_0)<0,证明:方程f(x)=0在(-\infty,+\infty)恰有两个实根\\&先证至少有两个实根:\\&证法一\cdot广义洛必达法则:若有g(x_0)\rightarrow\infty,且\lim_{x\rightarrow x_0}\frac{f'(x)}{g'(x)}=A,\\&则{\bf\color{red}无论f(x_0)趋于何值},\lim_{x\rightarrow x_0}\frac{f(x)}{g(x)}=\lim_{x\rightarrow x_0}\frac{f'(x)}{g'(x)}=A\\&根据该定理,未知f(x_0)是否趋于无穷时,也可运用洛必达\\&则\lim_{x\rightarrow+\infty}\frac{f(x)}{x}=\lim_{x\rightarrow+\infty}f'(x)=\alpha>0,\lim_{x\rightarrow-\infty}\frac{f(x)}x=\lim_{x\rightarrow x_0}f'(x)=\beta<0\\&故\lim_{x\rightarrow+\infty}f(x)=+\infty,\lim_{x\rightarrow-\infty}f(x)=+\infty,\\&即\forall N_1>0,\exist x_1>N_1使f(x_1)>0\\&\forall N_2<0,\exist x_2<N_2使f(x_2)>0,由存在x_0使f(x_0)<0,由零点定理得证至少有两个实根\\&证法二·泰勒展开\\&在任意一点x_1处展开,f(x)=f(x_1)+f'(x_1)(x-x_1)+\frac{f''(\xi)(x-x_1)^2}2,\xi介于x和x_1间\\&由f''(x)>0,得f(x)>f(x_1)+f'(x_1)(x-x_1)\\&根据已给条件,\begin{cases}{\exist充分大N_1>0,当x_1>N_1时,f'(x_1)>0}\\\exist充分大N_2<0,当x_1<N_2时,f'(x_1)<0\end{cases}\\&因此\lim_{x\rightarrow+\infty}f(x)>\lim_{x\rightarrow+\infty}(f(x_1)+f'(x_1)(x-x_1))>0,[当x_1>N_1时,该极限为(常数+正数*正无穷)]\\&\lim_{x\rightarrow-\infty}f(x)>\lim_{x\rightarrow-\infty}(f(x_1)+f'(x_1)(x-x_1))>0,[当x_1<N_2时,该极限为(常数+负数*负无穷)]\\&通过和证法一相同的极限语言,零点定理得证\\&再证只有两个实根:\\&反证法,若有三个及以上实根,三次罗尔定理后应存在一点\eta使f''(\eta)=0,然而f''(x)>0,矛盾\\&故f(x)=0恰有两个实根,证毕\end{align}$

---

$\begin{align}&7.设函数f(x)在区间[0,1]上连续且\int_0^1f(x)dx\ne0,证明:在区间[0,1]上存在三个不同的点x_1,x_2,x_3,使得\end{align}$
$$
\frac\pi8\int_0^1f(x)dx=\left[\frac1{1+x_1^2}\int_0^{x_1}f(t)dt+f(x_1)\arctan x_1\right]x_3\\\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ =\left[\frac1{1+x_2^2}\int_0^{x_2}f(t)dt+f(x_2)\arctan x_2\right](1-x_3)
$$
$\begin{align}&证:[草稿纸过程]\\&观察两式,联想到分部积分,式子即F(x)=\arctan x\int_0^xf(t)dt的导数\\&观察x_3、1-x_3,x_3\in(0,1),然后x_1、x_2是双中值,尝试拉中\\&证:[作答]\\&令F(x)=\arctan x\int_0^xf(t)dt,则F(0)=0,F(1)=\frac\pi4\int_0^1f(x)dx\\&由\int_0^1f(x)dx\ne0,得0<\left|\frac\pi8\int_0^1f(t)dt\right|<\left|\frac\pi4\int_0^1f(t)dt\right|\\&又F(x)在[0,1]上连续,由介值定理必存在一点x_3\in(0,1)使F(x_3)=\frac\pi8\int_0^1f(t)dt\\&在[0,x_3]上拉中,\frac{\frac\pi8\int_0^1f(t)dt}{x_3}=F'(x_1)=\frac1{1+x_1^2}\int_0^{x_1}f(t)dt+f(x_1)\arctan x_1\\&在[x_3,1]上拉中,\frac{\frac\pi8\int_0^1f(t)dt}{1-x_3}=F'(x_2)=\frac1{1+x_2^2}\int_0^{x_2}f(t)dt+f(x_2)\arctan x_2\\&证毕.\end{align}$

---

$\begin{align}&8.设函数f(x)在[0,1]上有二阶导数,且有正常数A,B,使得\end{align}$
$$
|f(x)|\le A,|f''(x)|\le B
$$
$\begin{align}&证明:对于任意x\in[0,1],有|f'(x)|\le 2A+\frac B2\\&证:在[0,1]中任意点展开,f(x)=f(x_0)+f'(x_0)(x-x_0)+\frac{f''(\xi)(x-x_0)^2}2,\xi介于x和x_0间\\&则\begin{cases}f(0)=f(x_0)-f'(x_0)x_0+\frac{f''(\xi_1)x_0^2}2\\f(1)=f(x_0)+f'(x_0)(1-x_0)+\frac{f''(\xi_2)(1-x_0)^2}2\end{cases}\\&f(1)-f(0)=f'(x_0)+\frac{f''(\xi_2)(1-x_0)^2-f''(\xi_1)x_0^2}2\\&则有f(x_0)=f(1)-f(0)+\frac{f''(\xi_1)x_0^2-f''(\xi_2)(1-x_0)^2}2\\&|f'(x_0)|\le|f(1)|+|f(0)|+\left|\frac{f''(\xi_1)x_0^2}2\right|+\left|\frac{f''(\xi_2)(1-x_0)^2}2\right|\\&\le2A+(x_0^2+(1-x_0)^2)\frac B2\le2A+\frac B2,证毕\end{align}$

---

$\begin{align}&9.设函数f(x)在[-2,2]上二阶可导,且|f(x)|<1,又f^2(0)+[f'(0)]^2=4,\\&试证在(-2,2)内至少存在一点\xi,使f(\xi)+f''(\xi)=0\\&[给出的等式条件求导后恰好包含待证式]\\&证:令F(x)=f^2(x)+[f'(x)]^2,则F(0)=4,一个点不足以使用罗尔定理,因此要分区间\\&然而提供的是不等式,故推出新的不等式后需通过介值定理(或直接用费马引理)\\&[-2,0]上拉中,存在\xi_1\in(-2,0)使f'(\xi_1)=\frac{f(0)-f(-2)}2,|f'(\xi_1)|\le\frac{|f(0)|}2+\frac{|f(-2)|}2<1\\&[0,2]上拉中,存在\xi_2\in(0,2)使f'(\xi_2)=\frac{f(2)-f(0)}2,同理|f'(\xi_2)|<1\\&因此F(\xi_1)<2,F(\xi_2)<2,而F(0)=4\\&故由局部极大值定理和费马引理,\\&存在\xi\in(\xi_1,\xi_2),也即\xi\in(-2,2)使F'(\xi)=2f(\xi)f'(\xi)+2f'(\xi)f''(\xi)=0,且它是一个极大值\\&又F(\xi)=f^2(\xi)+[f'(\xi)]^2>2,(极大值的性质)得f'(\xi)\ne0\\&结论得证\end{align}$

---

通常双中值问题中，用于划分区间的中值$x_0$是给出的(一般是关于$x_0$的等式，例如题$2.(1)$)，而在一些未给出的问题中，需要通过待定系数法求解，即**假设存在可将大区间划分成两个子区间的中值**，**使**得出的**双中值满足待证结论**，再求出它所满足的式子

$\begin{align}&10.设函数f(x)在区间[0,1]上连续,且I=\int_0^1f(x)dx\ne0.证明:在(0,1)内存在不同的两点x_1,x_2使得\\&\frac1{f(x_1)}+\frac1{f(x_2)}=\frac2I\\&证:令F(x)=\int_0^xf(t)dt,则F(0)=0,F(1)=I,,记F(x_0)=A,x_0\in(0,1)\\&在[0,x_0]上拉中,存在x_1\in(0,x_0)使\frac{A}{x_0}=f(x_1)\\&在[x_0,1]上拉中,存在x_2\in(x_0,1)使\frac{I-A}{1-x_0}=f(x_2)\\&则\frac1{f(x_1)}+\frac1{f(x_2)}=\frac{x_0I-x_0A+A-x_0A}{(I-A)A}\\&令其=\frac 2I,解得I^2x_0-(2x_0+1)IA+2A^2=(x_0I-A)(I-2A)=0,\\&转化为证明存在一点x_0,使x_0I-F(x_0)=0或I-2F(x_0)=0\\&即找到一点x_0,使F(x_0)=\frac I2,显然由介值定理可得该点是存在的\\&找出x_0的过程为草稿纸过程,实际作答应直接说明存在x_0使F(x_0)=\frac I2再顺着逻辑证明\end{align}$

---

$\begin{align}&11.设f(x)在[0,+\infty)上可微,f(0)=0,且存在常数A>0,使得\\&|f'(x)|\le A|f(x)|在[0,+\infty)上成立,试证明:在(0,+\infty)上有f(x)\equiv0\\&分析:在[0,x]上拉中,则存在一点x_0\in(0,x)使f(x)=xf'(x_0)\\&则|f(x)|=|xf'(x_0)|\le A|f(x_0)||x|,由|f(x)|\ge0,要证f(x)\equiv0,即证|f(x)|\le0\\&证:取\ x\in\left(0,\frac1{2A}\right],记|f(x_1)|为\max\left(|f(x)|,x\in\left(0,\frac1{2A}\right]\right),\\&在[0,x_1]上拉中,存在一点\xi_1\in(0,x),使\ |f(x_1)|\le A|f(\xi_1)||x_1|\le\frac{|f(\xi_1)|}2\le\frac{|f(x_1)|}2\\&故在此区间内,|f(x)|的最大值为0,即0\le|f(x)|\le0\\&取x\in\left(\frac1{2A},\frac2{2A}\right],拉中后仍能得出在此区间内|f(x)|的取值只能为0\\&递推区间至[0,+\infty)仍成立,证毕\end{align}$

---

$\begin{align}&12.设函数f(x)在[0,1]上连续,且\int_0^1f(x)dx=0,\int_0^1xf(x)dx=1,试证:\\&(1)\ \exist x_0\in[0,1]使得\Big|f(x_0)\Big|>4;(2)\ \exist x_1\in[0,1]使得\Big|f(x_1)\Big|=4\\&\end{align}$

---

# 微分方程<a id="微分方程"></a>

## 复习

- 一阶非齐次线性微分方程：含公式
  - 关于$y(x)$的$y'(x)+P(x)y(x)=Q(x)$：$\begin{align}y=e^{-\int P(x)\mathrm dx}\left(\int e^{\int P(x)\mathrm dx}Q(x)\mathrm dx+C\right)\end{align}$
- 二/高阶线性微分方程：解差分方程，当作两个一阶非齐次线性微分方程解；含公式，公式原理即前者所述
  - 以下公式**以特征方程的解$\lambda$为基础**，实质上就是差分方程
  - 二阶的推导：对$y''+py'+qy=f(x)$，有$(y'-\lambda_1 y)'-\lambda_2(y'-\lambda_1y)=f(x)$，故有$\begin{align}&y'-\lambda_1y=e^{\lambda_2x}\left(\int e^{-\lambda_2x}f(x)\mathrm dx+C_1\right)\\&y=e^{\lambda_1 x}\left(\int \left(e^{(\lambda_2-\lambda_1)x}·\left(\int e^{-\lambda_2 x}f(x)\mathrm dx+C_1\right)\right)\mathrm dx+C_2\right)\end{align}$
  - 由数学归纳法有以下结论：$\begin{align}&当f(x)=0,\lambda_1\ne\lambda_2时,y=\frac{C_1'e^{\lambda_1x}+C_2'e^{\lambda_2 x}}{\lambda_2-\lambda_1}=C_1''e^{\lambda_1 x}+C_2''e^{\lambda_2x},(单根)\\&当f(x)=0,\lambda_1=\lambda_2时,y=e^{\lambda_1x}\left(C_1x+C_2\right),(二重根,多重根类似)\\&\\&当f(x)=0,\lambda为共轭复数单根\alpha\pm\beta\ i时,根据欧拉公式,y=e^{\alpha x}(C_1\cos\beta x+C_2\sin\beta x)\\&当f(x)=0,\lambda为共轭复数n重根时,\\&y=e^{\alpha x}((C_1+C_2x+\cdots+C_nx^n)\cos\beta x+(C_1'+C_2'x+\cdots+C_n'x^n)\sin\beta x)\\&\\&当f(x)\ne 0,且为(Ax^n+Bx^{n-1}+\cdots+C)e^{\alpha x}类型时,\\&y的一个特解为(A'x^n+B'x^{n-1}+\cdots+C')e^{\alpha x}x^k,\ 其中\alpha为k重特征根\\&当f(x)\ne0,且为((Ax^n+Bx^{n-1}+\cdots+C)\cos\beta x+(A'x^n+B'x^{n-1}+\cdots+C')\sin\beta x)e^{\alpha x}时,\\&y的一个特解为\\&((A''x^n+B''x^{n-1}+\cdots+C'')\cos\beta x+(A'''x^n+B'''x^{n-1}+\cdots+C''')\sin\beta x)e^{\alpha x}x^k,\\&其中\alpha+\beta\ i为k重特征根\\&当f(x)\ne0且为上述三种情况的线性组合时,y的一个特解是各项特解的线性组合\end{align}$
- 全微分方程：凑微分法或偏积分法

## 题集

$\begin{align}&1.求解微分方程:\begin{cases}{\large\frac{\mathrm dy}{\mathrm dx}}-xy=xe^{x^2}\\y(0)=1\end{cases}\\&解:y的通解为e^{\int x\mathrm dx}\left(\int e^{\int -x\mathrm dx} xe^{x^2}\mathrm dx+C\right)=e^{x^2}+Ce^{\frac{x^2}2}\\&代入y(0)=1,则C=0,\ y=e^{x^2}\end{align}$

---

$\begin{align}&2.求解微分方程:\begin{cases}{(x+1){\large\frac{\mathrm dy}{\mathrm dx}}}-xy=xe^{x^2}\\y(0)=0\end{cases}\\&解:y的通解为e^{^{\int\frac{x}{x+1}\mathrm dx}}\left(\int e^{\int-\frac{x}{x+1}\mathrm dx}·\frac{xe^{x^2}}{x+1}\mathrm dx+C\right)\\&=\frac{e^x}{x+1}\left(\frac{e^{2x^2-2x}}4+C\right),\ 代入y(0)=0,得C=-\frac14\\&故y=\frac{e^x}{x+1}\left(\frac{e^{2x^2-2x}}4-\frac14\right)\end{align}$

---

$\begin{align}&3.设实数a\ne0,求解微分方程\begin{cases}y''-ay'^2=0\\y(0)=0,y'(0)=-1\end{cases}\\&解:易解得y'=-\frac1{ax+C_1}\\&则y=-\ln(ax+C_1)+C_2\\&代入y(0)=0,y'(0)=-1,得C_1=1,C_2=0\\&故y=-\ln(ax+1)\end{align}$

---

$\begin{align}&4.求解微分方程:y''-(y')^3=0的通解\\&解:易得-\frac{(y')^{-2}}2=x+C\\&即y'=\sqrt{\frac1{C_1-2x}}\\&故y=\int y'\mathrm dx=-\sqrt{C_1-2x}+C_2\end{align}$

---

$\begin{align}&5.已知y_1=e^x和y_2=xe^x是齐次二阶常系数线性微分方程的解,则该微分方程是:\\&解:易猜测特征根为\ 1(双重根),则特征方程为(\lambda-1)^2=0\\&故微分方程为y''-2y'+y=0\end{align}$

---

$\begin{align}&6.已知y_1=xe^x+e^{2x},y_2=xe^x+e^{-x},y_3=xe^x+e^{2x}-e^{-x}\\&是某二阶常系数线性非齐次微分方程的三个解,求其微分方程\\&解:两特解之差为对应齐次方程的特解:y^*_1=e^{-x},y^*_2=e^{2x}\\&易得两根为-1,2,即对应齐次方程为y''-y'-2y=0\\&又有y_3+y_2-y_1=xe^x也是非齐次方程的一个特解,\\&代入得该非齐次方程为y''-y'-2y=e^x-2xe^x\end{align}$

---

$\begin{align}&7.已知可导函数f(x)满足f(x)\cos x+2\int_0^xf(t)\sin t\mathrm dt=x+1,则f(x)=\_\_\_\_\_.\\&解:两边求导:-f(x)\sin x+f'(x)\cos x+2f(x)\sin x=1\\&即f'(x)+\tan xf(x)-\sec x=0\\&解得f(x)=e^{-\int \tan x\mathrm dx}\left(\int e^{\int \tan x\mathrm dx}\sec x\mathrm dx+C\right)\\&=\sin x+C\cos x,\ 代入f(0)=1,得C=1,故f(x)=\sin x+\cos x\end{align}$

---

$\begin{align}&8.满足\frac{\mathrm du(t)}{\mathrm dt}=u(t)+\int_0^1u(t)\mathrm dt及u(0)=1的可微函数u(t)为\_\_\_\_.\\&解:设I=\int_0^1u(t)\mathrm dt,由u'(t)=u(t)+I,得\ln|u(t)+I|=t+C\\&即u(t)=Ce^t-I\\&两边从0到1积分,得I=C(e-1)-I\\&即I=\frac{C(e-1)}2\\&故u(t)=Ce^t-\frac{C(e-1)}2,代入u(0)=1,得C=\frac2{3-e}\\&故u(t)=\frac{2e^t+1-e}{3-e}\end{align}$

---

$\begin{align}&9.求方程(2x+y-4)\mathrm dx+(x+y-1)\mathrm dy=0的通解\\&解:全微分方程:\mathrm d(x^2-4x)+\mathrm d(\frac{y^2}2-y)+\mathrm d(xy)=0\\&故其通解为x^2-4x+\frac{y^2}2-y+xy+C\end{align}$

---

$\begin{align}&10.设函数f(x,y)具有一阶连续偏导数,满足\mathrm df(x,y)=ye^y\mathrm dx+x(1+y)e^y\mathrm dy及f(0,0)=0,则f(x,y)=\\&解:f(x,y)=ye^yx+Q(y)\\&f_y'(x,y)=(1+y)e^yx+Q'(y)=x(1+y)e^y\Rightarrow Q'(y)=0\Rightarrow Q(y)=C\\&故f(x,y)=xye^y+C,代入f(0,0)=0得C=0\\&故f(x,y)=xye^y\end{align}$

---

$\begin{align}&11.已知\mathrm du(x,y)=\frac{y\mathrm dx-x\mathrm dy}{3x^2-2xy+3y^2},则u(x,y)=\_\_\_\_.\\&解:u(x,y)=\int\frac y{3x^2-2xy+3y^2}\mathrm dx=y\int\frac{\mathrm dx}{(\sqrt 3x-\frac{y}{\sqrt 3})^2+\frac83y^2}\\&=\frac{\sqrt2}4\arctan(\frac{3x-y}{2\sqrt2y})+Q(y)\\&u_y'(x,y)=\frac{\sqrt2}4·\frac{\frac{-2\sqrt2y-2\sqrt2(3x-y)}{8y^2}}{1+\frac{9x^2-6xy+y^2}{8y^2}}+Q'(y)=\frac{-x}{3x^2-2xy+3y^2}\\&解得Q'(y)=0,即Q(y)=C\\&故u(x,y)=\frac{\sqrt2}4\arctan(\frac{3x-y}{2\sqrt2y})++C\end{align}$

---

$\begin{align}&12.设f(u,v)具有连续偏导数,满足f_u(u,v)+f_v(u,v)=uv,\\&求y(x)=e^{-2x}f(x,x)所满足的一阶微分方程,并求其通解\\&解:两边对x求导,得y'(x)=e^{-2x}(f_1'(x,x)+f'_2(x,x)-2f(x,x))=e^{-2x}x^2-2y(x)\\&故y=e^{-\int2\mathrm dx}\left(\int e^{\int 2\mathrm dx}e^{-2x}x^2\mathrm dx+C\right)=e^{-2x}\left(\frac{x^3}3+C\right)\end{align}$

---

$\begin{align}&13.设f(x)有连续导数,且f(1)=2.记z=f(e^xy^2),若\frac{\part z}{\part x}=z,f(x)在x>0的表达式为\_\_\_\_.\\&解:z_x=f'(e^xy^2)·y^2e^x=f(e^xy^2),解得\ln|f(e^xy^2)|=\ln|e^xy^2|+C_0\\&即f(e^xy^2)=Ce^xy^2,也即f(x)=Cx\\&代入f(1)=2,得C=2,即f(x)=2x\\&当x>0时,e^xy^2>y^2\ge0,则f(x)在x>0的表达式为f(x)=2x,x>0\end{align}$

---

$\begin{align}&14.设可微函数f(x,y)满足\frac{\part f}{\part x}=-f(x,y),f\left(0,\frac\pi2\right)=1,且\end{align}$
$$
\begin{align}\lim_{n\rightarrow \infty}\left(\frac{f\left(0,y+{\Large\frac1n}\right)}{f(0,y)}\right)^n=e^{\cot y}\end{align}
$$
$\begin{align}&则f(x,y)=\_\_\_\_.\\&解:由该极限可得\lim_{n\rightarrow\infty}e^{n\ln\large\frac{f\left(0,y+\frac1n\right)}{f(0,y)}}=e^{\cot y}\\&即\lim_{n\rightarrow\infty}\frac{(f(0,y+\frac1n)-f(0,y))n}{f(0,y)}=\cot y\\&即\frac{f_y'(0,y)}{f(0,y)}=\cot y,解得f(0,y)=C\sin y,代入f(0,\frac\pi2)=1,得C=1\\&即f(0,y)=\sin y\\&设f(x,y)=P(x)\sin y+Q(x),则P(0)=1,\ Q(0)=0,\\&f_x'(x,y)=-f(x,y)\Rightarrow-P(x)\sin y-Q(x)=P'(x)\sin y+Q'(x)\end{align}$

$\begin{align}&解得P(x)=e^{-x},Q(x)=0\\&即f(x,y)=e^{-x}\sin y\end{align}$

---

$\begin{align}&15.设函数f(x)的导数f'(x)在[0,1]上连续,f(0)=f(1)=0,且满足\end{align}$
$$
\int_0^1\left[f'(x)\right]^2\mathrm dx-8\int_0^1f(x)\mathrm dx+\frac43=0
$$
$\begin{align}&则f(x)=\_\_\_\_.\\&解:\int_0^1f(x)\mathrm dx=xf(x)\Bigg|_0^1-\int xf'(x)\mathrm dx\\&故该等式可化为\int_0^1[[f'(x)]^2+8 xf'(x)]\mathrm dx+\frac43=0\\&即\int_0^1[[f'(x)]^2+8xf'(x)+\frac43]\mathrm dx=0\\&若[f'(x)]^2+8xf'(x)+\frac43=[F(x)]^2\ge0,则由该积分可得F(x)=0\\&待定系数法设F(x)=f'(x)+4x+A,其中A为常数\\&则有A^2+8A\int_0^1x\mathrm dx+2A\int_0^1f'(x)\mathrm dx=\frac43-16\int_0^1 x^2\mathrm dx\\&即A^2+4A=-4,解得A=-2\\&得F(x)=f'(x)+4x-2=0\\&解得f(x)=-2x^2+2x+C,由f(0)=f(1)=0,得C=0\\&故f(x)=-2x^2+2x\end{align}$

---

$\begin{align}&16.设f(x)在[0,+\infty)上是有界连续函数,证明:方程y''+14y'+13y=f(x)的\\&每一个解在[0,+\infty)上都是有界函数\\&解:由y''+14y'+13y=f(x)\Rightarrow\lambda_1=-13,\lambda_2=-1\\&y'+13y=e^{-x}\left(\int e^xf(x)\mathrm dx+C_1\right),\\&解得y=e^{-13 x}\left(\int e^{12x}\left(\int e^xf(x)\mathrm dx+C_1\right)+C_2\right)\\&由f(x)在[0,+\infty)上是有界连续函数,故e^xf(x)可积,设m\le f(x)\le M\\&则me^x+C_1\le \int e^xf(x)\mathrm dx+C_1\le Me^x+C_1\\&则\frac m{13}e^{13x}+\frac{C_1}{12}e^{12x}+C_2\le \int e^{12x}\left(\int e^xf(x)\mathrm dx+C_1\right)+C_2\le \frac M{13}e^{13x}+\frac{C_1}{12}e^{12x}+C_2\\&即\frac m{13}+\frac{C_1}{12}e^{-x}+C_2e^{-13x}\le y\le \frac M{13}+\frac{C_1}{12}e^{-x}+C_2e^{-13x}\\&其中0<e^{-13x}<e^{-x}\le 1,(x\in[0,+\infty))\\&即对y的通解,它在[0,+\infty)上都是有界函数\end{align}$

---

$\begin{align}&13.\end{align}$

$\begin{align}&13.\end{align}$

$\begin{align}&13.\end{align}$

$\begin{align}&f(x,y)=(y+1)^2-(2-x)\ln x,求由曲线f=0绕y=-1旋转而成的几何体的体积\end{align}$
