---
title: "微积分: 级数"
date: 2024-04-14
categories: [SE Courses, Calculus]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 无穷级数

## 常数项级数

### 定义

无穷多个数相加，记为$\begin{align}\sum_{n=1}^\infty u_n\end{align}$，称为常数项无穷级数，又称常数项级数或级数

### 部分和数列

令$\begin{align}S_n=\sum_{k=1}^nu_k(n=1,2,...)\end{align}$，我们说，当$n\rightarrow\infty$时，$\begin{align}S_n\approx\sum_{n=1}^\infty u_n\end{align}$

### 敛散性

当$\begin{align}\lim_{n\rightarrow\infty}S_n=S\end{align}$，且$S$为一个有限值时，称$\begin{align}\sum_{n=1}^\infty u_n\end{align}$收敛，且称$S$为它的和，否则称它发散

对于敛散性问题，我们将它转换成**求$S_n$通项**，再判断极限是否存在的问题

### 常见级数

- **调和级数**：$\begin{align}\sum_{n=1}^\infty \frac1n\end{align}$发散
  - $\begin{align}&证明:u_n=\frac1n=\int_n^{n+1}\frac1ndx>\int_n^{n+1}\frac1xdx\\&\therefore S_n>\int_1^{n+1}\frac1xdx=\ln x\Bigg|_1^{+\infty}=+\infty\\&\therefore\sum_{n=1}^\infty \frac1n发散 \end{align}$
- p级数(超调和级数)：$\begin{align}\sum_{n=1}^\infty\frac1{n^p}\end{align}$当$p\le1$时发散，当$p>1$时收敛
  - $\begin{align}&证明:u_n=\frac1{n^p}=\int_n^{n+1}\frac1{n^p}dx>\int_n^{n+1}\frac1{x^p}dx\\&u_n=\frac1{n^p}=\int_{n-1}^{n}\frac1{n^p}dx<\int_{n-1}^{n}\frac1{x^p}dx\\&\therefore S_n>\int_1^{n+1}\frac1{x^p}dx，S_n<\int_0^n\frac1{x^p}dx\\&1)当p=1时，为调和级数，已证明为发散\\&2)当p<1时，S_n>\frac1{1-p}x^{1-p}\Bigg|_1^{+\infty}，易得p<1时发散\\&3)当p>1时，S_n<\frac1{1-p}x^{1-p}\Bigg|_1^{+\infty}，易得p>1时收敛\end{align}$
- $\begin{align}\sum_{n=2}^\infty\frac1{n^p(\ln x)^q}\end{align}$当$p<1$时发散；当$p>1$时收敛；当$p=1$时，$\begin{cases}发散,&q\le1\\收敛,&q>1\end{cases}$

### 级数的性质

- 一个级数乘以一个实数不改变其敛散性
- 两收敛级数相加减得到的级数仍收敛，且$S=S_1\pm S_2$
- 改变级数的**有限个**项，级数的敛散性不变
- 若用括号将级数(S1)的部分项合并为一项，将得到的级数称为S2，若S1收敛则S2仍收敛，若S2发散则S1必发散
- $\begin{align}\sum_{n=1}^\infty u_n收敛\Rightarrow\lim_{n\rightarrow\infty}u_n=0，反之不成立(例如调和级数);\ \lim_{n\rightarrow\infty}u_n\ne0\Rightarrow\sum_{n=1}^\infty u_n发散\end{align}$

### 正项级数的比较判别法

#### 一般形式

对于正项级数，$\{S_n\}$本身是单调递增的，根据单调有界收敛原理，若$\begin{align}\lim_{n\rightarrow\infty}S_n\end{align}$存在，则该级数收敛

正项级数的比较判别法与无穷区间上反常积分的比较判别法类似：
$$
\begin{align}若存在N\in N^+,当n\ge N时0<u_n\le v_n,若\sum_{n=1}^\infty u_n发散,则\sum_{n=1}^\infty v_n发散;若\sum_{n=1}^\infty v_n收敛,则\sum_{n=1}^\infty u_n收敛\end{align}
$$
在用比较判别法判断时，应强调一般项的正负和用于比较的标准级数

标准级数一般为p级数、等比级数等

#### 极限形式

$$
\begin{align}&\large若\lim_{n\rightarrow\infty}\frac{u_n}{v_n}=l(u_n、v_n>0):\\&\large1)若0<l<+\infty，则\sum_{n=1}^\infty u_n和\sum_{n=1}^\infty v_n的敛散性相同(此时k_1v_n<u_n<k_2v_n)\\&\large2)若l=0，则说明u_n<v_n，对应比较判别法\\&\large3)若l=+\infty，则说明v_n<u_n\end{align}
$$

### 正项级数的比值判别法

比较判别法需要借助一个别的标准级数，而比值判别法由级数本身来确定它的敛散性
$$
\begin{align}&\large若\lim_{n\rightarrow\infty}\frac{u_{n+1}}{u_n}=\rho(u_n>0):\\&\large1)当\rho<1时，级数收敛\\&\large2)当\rho>1或\rho=+\infty时，级数发散\end{align}
$$

### 正项级数的根植判别法

也称柯西判别法，当一般项的外部带n次方时可以使用
$$
\begin{align}&\large若\lim_{n\rightarrow\infty}\sqrt[n]{u_n}=\rho(u_n>0):\\&\large1)当\rho<1时，级数收敛\\&\large2)当\rho>1或\rho=+\infty时，级数发散\end{align}
$$

### 正项级数的积分判别法

上述的调和级数和p级数敛散性问题借用反常积分转换为了反常函数判别法的问题，对于正项级数，如果它的一般项**单调递减**，则它与$\begin{align}\int_1^{+\infty}u(x)dx\end{align}$的敛散性相同

### 交错级数的莱布尼茨判别法

交错级数，即各项正负相间的级数，表现为$\begin{align}\sum_{n=1}^\infty(-1)^{n-1}u_n\end{align}$

若$u_n$单调递减且$\begin{align}\lim_{n\rightarrow\infty}u_n=0\end{align}$，则交错级数收敛

### 绝对收敛与条件收敛

对任意项级数，由其每一项的绝对值组成的新级数称为**绝对值级数**，如果它收敛，则原级数也收敛，这称为原级数**绝对收敛**。因此对于一个交错级数，既可以用莱布尼茨判别法，也可以用正项级数的判别法

但是，当绝对值级数发散时，不能断定原级数发散，而只能说明它不是绝对收敛。如果原级数收敛而绝对值级数发散，则称原级数**条件收敛**，例如$\begin{align}\sum_{n=1}^\infty(-1)^{n-1}\frac1n\end{align}$

因为交错级数的部分和数列是小于它的绝对值级数的部分和数列的，这就很像比较判别法：大的收敛则小的也收敛；而大的发散，小的不一定发散

但是，由比值判别法或根值判别法得出绝对值级数发散的原级数也发散，这很好理解：由比值判别法得到$u_n$单调递增，不符合莱布尼茨判别法的第一条件；由根值判别法得到$\begin{align}\lim_{n\rightarrow\infty}u_n=+\infty\end{align}$，不符合莱布尼茨判别法的第二条件

->插曲：一个很有意思的事，欧拉证明欧拉公式时用到了泰勒级数的知识，被称为形式推导，没有考虑到$e^{i\pi}$展开后条件收敛的情况，这时调节项的位置有可能改变它的敛散性，这样证明是不严谨的

**黎曼重排定理**：一个条件收敛的级数存在不同的排序方法，使其收敛于任意数(包括$+\infty$)

证明：假设级数$\begin{align}\sum_{n=1}^\infty u_n\end{align}$**条件收敛**，由于$\begin{align}\sum_{n=1}^\infty |u_n|\end{align}$发散，故必有**发散**的两个子级数$\begin{align}\sum_{n=1}^\infty a_n、\sum_{n=1}^\infty b_n\end{align}$，其中$a_n<0,b_n>0$，由原级数条件收敛的性质，虽然它们发散，但$\begin{align}\lim_{n\rightarrow\infty}a_n=\lim_{n\rightarrow\infty}b_n=0\end{align}$

设想一个值S，它可以是有限值或无穷，从$b_n$中取有限个项直至重排数列的和大于S，再从$a_n$中取有限个项直至重排数列的和小于S，重复此操作直到$a_n与b_n$剩下无穷个趋于0的项，这样，重排后的数列和将无限接近S

一个简单、不严谨的例子：级数$(1-1)+(1-1)+...$收敛于0，取一个1放在开头，变成：$1+(1-1)+(1-1)+...$，则它收敛于1。这只是便于理解，但它是错误的，不能随意去掉收敛级数的括号或给发散级数添加无限个括号

## 函数项级数

### 定义

所谓函数项级数，就是每一项都由一个函数组成的级数

如果在$x=x_0$处形成的数项级数收敛，则称它为这个函数项级数的收敛点，否则称为发散点

所有收敛点的集合称为这个幂级数的收敛域

### 幂级数

幂级数是最广泛的一类函数项级数，$\begin{align}\sum_{n=0}^\infty a_nx^n\end{align}$称为$x$的幂级数

#### 阿贝尔定理

若一个幂级数在$x=x_0$处收敛，则它在$(-|x_0|,\ |x_0|)$上绝对收敛；如果在此处发散，则它在$(-\infty,\ -|x_0|)\cup(|x_0|,\ +\infty)$上发散

幂级数只有三种情况：存在一个$R$

- 如果$0<R<+\infty$，则幂级数在$(-R,R)$上绝对收敛，在$(-\infty,-R)\cup(R,+\infty)$上发散
- 如果$R=0$，则幂级数处处发散
- 如果$R=+\infty$，则幂级数处处绝对收敛

所以，收敛域一般是形如$(-R,R)$的区间，因此在$x=\pm R$处需要额外判断敛散性

#### 收敛半径计算法

$R$的求法类似比值判别法，对于幂级数$\begin{align}\sum_{n=0}^\infty a_nx^n\end{align}$的绝对值级数，假设有：

$\begin{align}\lim_{n\rightarrow\infty}\Bigg|\frac{a_{n+1}x^{n+1}}{a_nx^n}\Bigg|=\rho|x|,即\lim_{n\rightarrow\infty}\Bigg|\frac{a_{n+1}}{a_n}\Bigg|=\rho\end{align}$

则当$\rho|x|<1$，即$\begin{align}|x|<\frac1\rho\end{align}$时，级数收敛，反之发散

因此：当$\rho\ne0$时，$\begin{align}R=\frac1\rho\end{align}$；当$\rho=0$时，$R=+\infty$；当$\rho=+\infty$时，$R=0$

所以，我们求$\begin{align}\lim_{n\rightarrow\infty}\Bigg|\frac{a_n}{a_{n+1}}\Bigg|\end{align}$，对应的结果便是$R$

这个方法只适用于缺有限项或不缺项的幂级数，而类似$\begin{align}\sum_{n=0}^\infty a_nx^{2n}\end{align}$这样缺无穷多项的级数应直接用比值判别法

**推论**：形似$\begin{align}\sum_{n=0}^\infty a_nx^{kn}\end{align}$的收敛半径为$\sqrt[k]R$

#### 性质

- 线性运算：两个幂级数相加减，收敛半径为$R=\min(R_1,R_2)$
- 乘法运算：两个幂级数相乘，收敛半径也为$R$
- 若$\begin{align}\sum_{n=0}^\infty a_nx^n\end{align}$收敛半径为$R$，则$\begin{align}\sum_{n=1}^\infty na_nx^{n-1}、\sum_{n=2}^\infty n(n-1)a_nx^{n-2}\end{align}$和$\begin{align}\sum_{n=0}^\infty \frac{1}{n+1}a_nx^{n+1}\end{align}$的收敛半径也为$R$
- 假设幂级数的和函数为$S(x)$，收敛半径为$R$，则$S(x)$在$(-R,R)$上**连续且可导且可积**，且：
  - $\begin{align}S'(x)=\sum_{n=1}^\infty na_nx^{n-1}\end{align}$，根据第三点，$S'(x)$对应的幂级数收敛半径也为$R$
  - $\begin{align}\int_0^xS(x)dx=\sum_{n=0}^\infty \frac{1}{n+1}a_nx^{n+1}\end{align}$，根据第三点，它对应的幂级数收敛半径也为$R$
  - 求导或积分后对应的幂级数在$x=R$上的敛散性可能变化

#### 泰勒级数

幂级数形式简单，有许多性质，而且可以把无法用初等函数表示的函数写成幂级数(例如一些函数的原函数)

把已知函数化为幂级数：

由$f(x)在x=x_0$处的泰勒展开$\begin{align}f(x)=f(x_0)+f'(x_0)(x-x_0)+\frac{f''(x_0)}{2!}(x-x_0)^2+...\end{align}$，得$f(x)$在$x=x_0$处的泰勒级数为$\begin{align}f(x)=\sum_{n=0}^\infty \frac{f^{(n)}(x_0)}{n!}{(x-x_0)}^n\end{align}$，充要条件为$f(x)$在$x=x_0$处有任意阶导数且$\begin{align}\lim_{n\rightarrow\infty}(拉格朗日型余项)\end{align}$趋于0

一般地，将$f(x)$展开为泰勒级数的步骤为：

- 对$f(x)$高阶求导，得到幂级数
- 求幂级数的收敛域
- 证明$\begin{align}\lim_{n\rightarrow\infty}\frac{f^{(n+1)}(\xi)}{(n+1)!}(x-x_0)^{n+1}=0\end{align}$

更好的方法为：记住常见的泰勒级数，对它们或**求导**，或**积分**，或**换元**来求所求函数的幂级数，例如要求$g(x)=f'(x)=\cos x$的级数，只需对$f(x)=\sin x$的泰勒级数逐项求导即可

常见的麦克劳林级数：

![](.\calculus-series\麦克劳林级数.png)

### 傅里叶级数

$\begin{align}f(x)=\frac{a_0}2+\sum_{n=1}^\infty(a_n\cos nx+b_n\sin nx)\end{align}$

#### 狄利克雷收敛定理

如果$T_{f(x)}=2\pi$且$f(x)$在$(-\pi,\pi]$上满足：

- 连续或有有限个第一类间断点
- 有有限个单调区间

则$f(x)$可展开为处处收敛的傅里叶级数且$\begin{align}&S(x)=\frac{a_0}2+\sum_{n=1}^\infty(a_n\cos nx+b_n\sin nx)=\begin{cases}f(x),&在x处连续\\\ \large\frac{\lim_{x\rightarrow x^-}f(x)+\lim_{x\rightarrow x^+}f(x)}2,&x为间断点\end{cases}\\&a_0=\frac1\pi\int_{-\pi}^\pi f(x)dx,\ a_n=\frac1\pi\int_{-\pi}^\pi f(x)\cos nxdx,\ b_n=\frac1\pi\int_{-\pi}^\pi f(x)\sin nxdx\end{align}$

#### $\bf T=2l$(一般周期)函数

$\begin{align}&f(x)\sim\frac{a_0}2+\sum_{n=1}^\infty(a_n\cos \frac\pi lnx+b_n\sin \frac\pi lnx)\\&a_0=\frac1l\int_{-l}^l f(x)dx,\ a_n=\frac1l\int_{-l}^l f(x)\cos \frac \pi lnxdx,\ b_n=\frac1l\int_{-l}^l f(x)\sin \frac \pi lnxdx\end{align}$

**延拓**：给一个只在有限区间上有定义的函数添加定义(如新定义它的周期)，从而拓广函数的定义域

在$[0,l]$上有定义的函数的傅里叶展开：在$[-l,0)$上添加定义即可，有两种：

- 奇延拓：使其展成正弦级数，$\begin{align}f(x)\sim \sum_{n=1}^\infty b_n\sin \frac\pi lnx\end{align}$
- 偶延拓：使其展成余弦级数，$\begin{align}f(x)\sim\frac{a_0}2+\sum_{n=1}^\infty a_n\cos \frac\pi lnx\end{align}$
