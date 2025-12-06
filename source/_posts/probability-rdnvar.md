---
title: "概率论与数理统计: 常见的随机变量汇总"
date: 2025-05-05
categories: [SE Courses, Probability Math]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 常见随机变量分布及其数学特征

### 离散型随机变量

- 二项分布$X\sim B(n,p)$：可看作$n$个独立同分布的$0-1$分布
  - $\begin{align}P(X=k)=\left(\begin{matrix}n\\k\end{matrix}\right)p^k(1-p)^{n-k}\end{align}$，$k\in [0,n]$
  - $EX=np$：$\begin{align}\sum_{k=0}^nkP(X=k)=nEX_i=np\end{align}$，前者用二项式定理或生成函数证明、后者为$n$个$0-1$分布叠加
    - 前者的二项式定理证明：
      $\begin{align}&原式=np\sum_{k=1}^n\frac{(n-1)!}{(k-1)!(n-k)!}p^{k-1}(1-p)^{n-k}=np\sum_{k=0}^{n-1}\left(\begin{matrix}n-1\\k\end{matrix}\right)p^k(1-p)^{n-k-1}=np\end{align}$
  - $DX=np(1-p)$：由于为$n$个独立的$0-1$分布，可以由$DX_i$叠加得出
    - 定义证法：
      $\begin{align}&EX^2=\sum_{k=0}^nk^2P(X=k)=\sum_{k=1}^nk\frac{n!}{(k-1)!(n-k)!}p^k(1-p)^{n-k}\\&=\sum_{k=0}^{n-1}\frac{(k+1)·n!}{k!(n-k-1)!}p^{k+1}(1-p)^{n-k-1}\\&=\sum_{k=1}^{n-1}k\frac{n!}{k!(n-k-1)!}p^{k+1}(1-p)^{n-k-1}+np\sum_{k=0}^{n-1}\frac{(n-1)!}{k!(n-k-1)!}p^{k}(1-p)^{n-k-1}\\&=n(n-1)\sum_{k=0}^{n-2}\frac{(n-2)!}{k!(n-k-2)!}p^{k+2}(1-p)^{n-k}+np\\&=n(n-1)p^2+np=n^2p^2-np^2+np\\&DX=EX^2-(EX)^2=np(1-p)\end{align}$
  - $\begin{align}EX^k=nEX^k_i=np^k\end{align}$
- 泊淞分布$X\sim P(\lambda)$
  - $\begin{align}P(X=k)=\frac{\lambda^k}{k!}e^{-\lambda}\end{align}$，$k\in N^+$
  - $\begin{align}EX=\lambda\end{align}$：经典级数
    - $\begin{align}\sum_{k=1}^{+\infty}k\frac{\lambda^k}{k!}e^{-\lambda}=\lambda e^{-\lambda}e^{\lambda}=\lambda\end{align}$
  - $DX=\lambda$
    - $\begin{align}&EX^2=\sum_{k=1}^{+\infty}k\frac{\lambda^k}{(k-1)!}e^{-\lambda}=\lambda e^{-\lambda}S(\lambda)\\&\int S\mathrm d\lambda=\int\sum_{k=1}^{+\infty}\frac{k\lambda^{k-1}}{(k-1)!}\mathrm d\lambda=\sum_{k=1}^{+\infty}\frac{\lambda^{k}}{(k-1)!}=\lambda e^{\lambda}\\&S=(\lambda+1) e^{\lambda}\Rightarrow EX^2=\lambda^2+\lambda\\&DX=EX^2-(EX)^2=\lambda\end{align}$
    - 或者：
      $\begin{align}&EX^2=\sum_{k=1}^{+\infty}(k-1+1)\frac{\lambda^{k}}{(k-1)!}e^{-\lambda}\\&=e^{-\lambda}\sum_{k=1}^{+\infty}(k-1)\frac{\lambda·\lambda^{k-1}}{(k-1)!}+\lambda=\lambda+e^{-\lambda}\lambda\sum_{k=0}^{+\infty}\frac{k\lambda^{k}}{k!}\\&=\lambda+\lambda^2\end{align}$
  - $\begin{align}&EX^n=\sum_{k=1}^{+\infty}k^n\frac{\lambda^k}{k!}e^{-\lambda}\end{align}$，可以将$k^n$替换为系数为第二类斯特林数的$k$的下阶幂的和
    可递归表示为第二类斯特林数系数结合$\lambda$的幂、或二项式系数结合低阶期望的形式：
    $\begin{align}EX^n=\sum_{k=0}^n\left[\begin{matrix}n\\k\end{matrix}\right]\lambda^k=\lambda\sum_{k=0}^{n-1}\left(\begin{matrix}n-1\\k\end{matrix}\right)EX^k\end{align}$

### 连续型随机变量

- 均匀分布$X\sim U(a,b)$：
  - $f(x)=\begin{cases}{\Large\frac1{b-a}},&a\le x\le b\\0,&\rm otherwise\end{cases}$
  - $F(x)=\begin{cases}0,&x<a\\{\Large\frac{x-a}{b-a}},&a\le x\le b\\1,&x>b\end{cases}$
  - $\begin{align}EX=\int_{a}^b\frac{x}{b-a}\mathrm dx=\frac{b+a}{2}\end{align}$
  - $\begin{align}EX^k=\int_{a}^b\frac{x^k}{b-a}\mathrm dx=\frac{b^{k+1}-a^{k+1}}{(k+1)(b-a)}\end{align}$
  - $\begin{align}DX=EX^2-(EX)^2=\frac{(b-a)(b^2+ab+a^2)}{3(b-a)}-\frac{(b+a)^2}4=\frac{(b-a)^2}{12}\end{align}$
- 指数分布$X\sim e(\lambda)$：
  - $f(x)=\begin{cases}\lambda e^{-\lambda x},&x>0\\0,&\rm otherwise\end{cases}$
  - $F(x)=\begin{cases}0,&x\le0\\1-e^{-\lambda x},&x>0\end{cases}$，可以记住$\begin{align}\int_x^{+\infty}\lambda e^{-\lambda x}\mathrm dx=e^{-\lambda x}\end{align}$
  - $\begin{align}EX=\int_0^{+\infty}\lambda xe^{-\lambda x}\mathrm dx=-\int_{0}^{+\infty}x\mathrm d\left(e^{-\lambda x}\right)=-xe^{-\lambda x}\Bigg|_{0}^{+\infty}+\int_0^{+\infty}e^{-\lambda x}\mathrm dx=\frac1\lambda\end{align}$
  - $\begin{align}&EX^k=\int_0^{+\infty}\lambda x^ke^{-\lambda x}\mathrm dx=-\int_{0}^{+\infty}x^k\mathrm d\left(e^{-\lambda x}\right)\\&=-x^ke^{-\lambda x}\Bigg|_0^{+\infty}+k\int_0^{+\infty}x^{k-1}e^{-\lambda x}\mathrm dx=\frac{k}{\lambda}EX^{k-1}\\&由EX=\frac1\lambda,得EX^k=\frac{k!}{\lambda^k}\end{align}$
  - $\begin{align}DX=EX^2-(EX)^2=\frac1{\lambda^2}\end{align}$
- 正态分布$X\sim N(\mu,\sigma^2)$：
  - $\begin{align}f(x)=\frac1{\sqrt{2\pi}\sigma}e^{\Large-\frac{(x-\mu)^{\large2}}{2\sigma^{\large2}}}\end{align}$，$x\in R$
  - $\begin{align}\int_{-\infty}^{+\infty}f(x)\mathrm dx=1\end{align}$，你可以由归一性使用结论，其证明过程如下(标准正态分布为例)：
    - $\begin{align}&令I=\int_{-\infty}^{+\infty}e^{\large-\frac{x^{\normalsize2}}{2}}\mathrm dx,则I^2=\int_{-\infty}^{+\infty}e^{\large-\frac{x^{\normalsize2}}2}\mathrm dx\int_{-\infty}^{+\infty}e^{\large-\frac{y^{\normalsize2}}2}\mathrm dy\\&=\int_{-\infty}^{+\infty}\int_{-\infty}^{+\infty}e^{-\frac{x^2+y^2}2}\mathrm dx\mathrm dy=\lim_{a\rightarrow+\infty}\int_{-a}^{a}\int_{-a}^{a}e^{-\frac{x^2+y^2}2}\mathrm dx\mathrm dy\end{align}$$\begin{align}&虽然被积区域不是圆,但可以用夹逼定理使其极坐标变换\\&D_1<D<D_2,其中D_1为内切圆、D_2为外接圆\\&在D_1上,\lim_{a\rightarrow+\infty}\int_{0}^{2\pi}\mathrm d\theta\int_{0}^{\large\frac{a}2}re^{\Large-\frac{r^{\normalsize2}}2}\mathrm dr=2\pi\\&在D_2上,\lim_{a\rightarrow+\infty}\int_{0}^{2\pi}\mathrm d\theta\int_{0}^{\large\frac{a}{\sqrt2}}re^{\Large-\frac{r^{\normalsize2}}2}\mathrm dr=2\pi\\&因此I=\sqrt{2\pi},\int_{-\infty}^{+\infty}f(x)\mathrm dx=\frac I{\sqrt{2\pi}}=1\end{align}$
  - $EX=\mu、EX^2=\sigma^2+\mu^2$
  - $\begin{align}&EX^k=\int_{-\infty}^{+\infty}\frac{x^k}{\sqrt{2\pi}}e^{\Large-\frac{x^2}{2}}\mathrm dx=-\int_{-\infty}^{+\infty}\frac{x^{k-1}}{\sqrt{2\pi}}\mathrm d\left(e^{\Large-\frac{x^2}2}\right)\\&=0+\int_{-\infty}^{+\infty}\frac{(k-1)x^{k-2}}{\sqrt{2\pi}}e^{\Large-\frac{x^2}2}\mathrm dx=(k-1)EX^{k-2}\end{align}$
  - 标准正态分布：$\begin{align}&EX^k=(k-1)!!,k是偶数\\&EX^k=0,k是奇数\end{align}$
  - $DX=\sigma^2$

### 随机变量函数

### 二维随机变量

- 二维正态分布

### 统计量

- 正态分布$N(\mu,\sigma^2)$的$n$个简单随机样本的样本均值：$\begin{align}\bar X=\frac{1}{n}\sum_{i=1}^nX_i\end{align}$
  - $E\bar X=\mu$
  - $\begin{align}D\bar X=\frac{\sigma^2}n\end{align}$
- 正态分布$N(\mu,\sigma^2)$的$n$个简单随机样本的样本方差：$\begin{align}S^2=\frac1{n-1}\sum_{i=1}^n(X_i-\bar X)^2\end{align}$
  - $ES^2=\mu^2+\sigma^2$
  - 由$\begin{align}U=\frac{(n-1)S^2}{\sigma^2}\sim\chi^2(n-1)\end{align}$，$\begin{align}&DS^2=\frac{\sigma^4}{(n-1)^2}DU=\frac{\sigma^4}{(n-1)^2}(n-1)DX^2\\&DX^2=EX^4-(EX^2)^2=3EX^2-(EX^2)^2=2\\&DS^2=\frac2{n-1}\end{align}$
- 标准正态分布：不再赘述，设总体$X\sim N(\mu,\sigma^2)$，以下为常见的构造
  - $\begin{align}\frac{\sqrt n(\bar X-\mu)}{\sigma}\sim N(0,1)\end{align}$
  - $\begin{align}\frac{\bar X_1-\bar X_2+(\mu_1-\mu_2)}{\sqrt{\frac{\sigma^2_1}{n_1}+\frac{\sigma^2_2}{n_2}}}\sim N(0,1)\end{align}$
- 卡方分布$U\sim\chi^2(n)$：设$X_1,\cdots,X_n$为服从标准正态分布且相互独立的随机变量
  - $\begin{align}U=\sum_{i=1}^nX_i^2\end{align}$
  - $EU=nEX^2=n$
  - $DU=nDX^2=n(EX^4-(EX^2)^2)=2n$
- $t$分布$U\sim t(n)$：设$X\sim N(0,1)、Y\sim \chi^2(n)$，且它们相互独立
  - $\begin{align}U=\frac{X}{\sqrt{\large\frac{Y}{n}}}\end{align}$
  - 假设$X_{1\sim n}$来自$N(0,1)$，则$\begin{align}\frac{\sqrt n(\bar X-\mu)}{S}\sim t(n-1)\end{align}$
- $F$分布$U\sim F(n,m)$：设$X\sim\chi^2(n)、Y\sim\chi^2(m)$，且它们相互独立
  - $\begin{align}U=\frac{\large\frac{X}{n}}{\large\frac{Y}{m}}\end{align}$
