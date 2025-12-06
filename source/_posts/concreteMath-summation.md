---
title: "具体数学: 和式"
date: 2024-10-02
categories: [SE Courses, Concrete Math]
tags: [math, summation]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 和式

### 表达方式

- 一般形式：$\begin{align}\sum_{\rho(k)}a_k\end{align}$，用于简略描述和式，方便推导和理解
- 定界形式：$\begin{align}\sum_{k=lower}^{upper}a_k\end{align}$，用于表示最终结果
- $\rm Iverson$约定：$\begin{align}\sum_{k}a_{k}·[\rho(k)]\end{align}$，即遍历$k$，若满足命题$\rho(k)$则加和，否则不加

### 求解和式

- 清单法：因和式能很简单地写成递推式$\begin{cases}S_0=0\\S_n=S_{n-1}+a_n\end{cases}$，因此可使用清单法

- 求和因子法：适用于求解形如$\begin{cases}S_0\\a_nS_n=b_nS_{n-1}+c_n\end{cases}$的递推式

  - $\begin{align}&两边乘s_n,得s_na_nS_n=s_nb_nS_{n-1}+s_nc_n\\&其中s_n满足s_nb_n=s_{n-1}a_{n-1}&(1)\\&因此T_n=s_na_nS_n=T_{n-1}+s_nc_n\Rightarrow T_n=s_0a_0S_0+\sum_{k=1}^ns_kc_k\\&其中s_0a_0只是为了满足(1)式假想的,故S_n=\frac1{s_na_n}\left(s_1b_1S_0+\sum_{k=1}^ns_kc_k\right)\\&因为求和式中只用到s(1\sim n),现根据(1)式求出s_n,不难得到s_n=\frac{a_{n-1}\cdots a_1}{b_n\cdots b_2}s_1\\&为方便求解,一般令s_1=1\\&观察式子可知a_n和b_n不允许有零项\end{align}$

  - 当$s_nc_n$是较好求的式子时，可用求和因子法

    事实上求和因子法大多数情况下无法求给定数列的和，只能解上述$a_n$和$b_n$满足特定性质的递推式

    因为给定数列$a_n$，写成和式的递推式$S_n=S_{n-1}+a_n$，会发现$s_n=s_1$，最终仍无法求$a_n$的和

- 扰动法：取出首项或尾项，构造出一个等式，并使两部分的和式都和$S_n$套上关系即可

  - 例如给定数列$a_n$，有$\begin{align}S_{n+1}=S_n+a_n=a_1+\sum_{k=2}^{n+1}a_k\end{align}$

    扰动法的目的是将$\begin{align}\sum_{k=2}^{n+1}a_k=\sum_{k=1}^na_{k+1}\end{align}$分解为$S_n$和一个相对好求的和式

  - 例求$\begin{align}\sum_{k=0}^nkx^k\end{align}$：

    $\begin{align}&解:S_{n+1}=S_n+(n+1)x^{n+1}=0+\sum_{k=1}^{n+1}kx^k\\&=xS_n+\sum_{k=0}^nx^{k+1}=xS_n+\frac{x^{n+1}-x}{x-1}\\&\Rightarrow S_n=\frac{(n+1)x^{n+1}(x-1)-x-x^{n+1}}{(x-1)^2}\end{align}$

- 例如快速排序的平均比较次数(具体数学上和数据结构里不同，这里以具体数学为例)：

  $C_n=\begin{cases}0,&n=0\\n+1+\frac2n\sum_\limits{i=0}^{n-1}C_i,&n>0\end{cases}$

  $\begin{align}&即nC_n=2\sum_{i=0}^{n-1}C_i+n^2+n\\&(n-1)C_{n-1}=2\sum_{i=0}^{n-2}C_i+(n-1)^2+n-1\\&得nC_n-(n-1)C_{n-1}=2C_{n-1}+2n\\&即nC_n=(n+1)C_{n-1}+2n\end{align}$

  $\begin{align}&求和因子为s_n=\frac{a_{n-1}\cdots a_1}{b_n\cdots b_2}s_1=\frac{(n-1)!}{(n+1)\cdots3}s_1=\frac{2}{(n+1)n},(令s_1=1)\\&s_nnC_n=s_n(n+1)C_{n-1}+2ns_n=s_{n-1}(n-1)C_{n-1}+\frac4{n+1}\\&因此s_nnC_n=s_1b_1C_0+4\sum_{i=1}^{n}\frac1{i+1}=4(H_{n+1}-1)\\&C_n=2(n+1)(H_{n+1}-1)=2(n+1)(H_n+\frac1{n+1}-1)=2(n+1)H_n-2n\end{align}$

### 多重和式

- $[1\le i<j\le n]+[1\le k<j\le n]=[1\le j,k\le n]-[1\le i=j\le n]$
- 切比雪夫单调不等式：若两序列是单调的：$\begin{align}\begin{cases}单调性相同,&\left(\sum_\limits{k=1}^na_k\right)\left(\sum_\limits{k=1}^nb_k\right)\le n\sum_\limits{k=1}^n a_kb_k\\单调性不同,&上式为大于等于\end{cases}\end{align}$
- 应用于$n^k$前$n$项和公式：
  - 摄动求和：应对$n^{k+1}$摄动
  - 清单求和：令$R_n=1,n,n^2,\cdots,n^{k+1}$
  - 积分替换：求积分和求和的误差，并通过其递推式求解误差
- **下阶幂定义**($x$为实数，$m$为整数)：$x^{\underline m}=\begin{cases}x(x-1)\cdots(x-m+1),&m>0\\1,&m=0\\\frac1{(x+1)\cdots(x-m)},&m<0\end{cases}$
- **下阶幂的差分**：$\Delta(x^{\underline m})=(x+1)^{\underline{m}}-x^{\underline m}=mx^{\underline{m-1}}$
- 定和分：$\begin{align}\sum_a^bg(x)\delta x=\sum_{k=a}^{b-1}g(k)=f(x)\Bigg|_a^b,其中g(x)=\Delta f(x)\end{align}$
- 下阶幂的定和分可配合第二类斯特林数的原始定义，求解$x^k$的和($k$为常数)
- 指数函数的差分：$\Delta(c^x)=c^{x+1}-c^x=(c-1)c^x$，因此**$2^x$的差分等于其自身**
- 和分求指数函数和：$\begin{align}\sum_{k=0}^nc^k=\sum_{0}^{n+1}c^x\delta x=\frac{c^{n+1}-1}{c-1}\end{align}$
