---
title: "线性代数: 行列式"
date: 2024-05-14
categories: [SE Courses, Linear Algebra]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 行列式

## 结论

- $|A^T|=|A|$
- 一般矩阵一次对调变换后，$|A|$取反
- **一般矩阵或分块矩阵**一次倍加变换后，$|A|$不变
- $|kA|=k^n|A|$
- 分块矩阵一次对调变换后，$\left|\begin{matrix}A_{m\times j}&B_{m\times k}\\C_{n\times j}&D\end{matrix}\right|=(-1)^{m\times n}\left|\begin{matrix}C_{n\times j}&D\\A_{m\times j}&B_{m\times k}\end{matrix}\right|=(-1)^{j\times k}\left|\begin{matrix}B_{m\times k}&A_{m\times j}\\D&C_{n\times j}\end{matrix}\right|$
- $|A^{-1}|=|A|^{-1}$
- $若A、B都为方阵,\ 则|AB|=|A||B|$
- $若A可逆,\ 则A^*=|A|A^{-1}$
- $无论A是否可逆,\ |A^*|=\bigg||A|A^{-1}\bigg|=|A|^{n-1},(方便记忆,实际推导分情况)$
- $若A、C为方阵,\ 则\left|\begin{matrix}A&B\\O&C\end{matrix}\right|=|A||C|$
- $若A、B为同阶方阵,\ 则\left|\begin{matrix}A&B\\B&A\end{matrix}\right|=|A+B||A-B|$
- $|A|\ne0\Leftrightarrow A可逆\Leftrightarrow A为非奇异矩阵\Leftrightarrow 对应线性方程组有唯一解\Leftrightarrow r(A)=n$
- 上条的部分内容为**克莱姆法则**的一部分，若任一$n$个未知量、$n$条方程的线性方程组中，由系数组成的方阵的行列式**不为$0$**，则有**唯一解**，且$\begin{align}x=(\frac{D_1}D,\frac{D_2}D,\cdots,\frac{D_n}D)\end{align}$，其中$D_j$为将原行列式中的第$j$列元素替换成列向量$b$后形成的行列式
- 特别地，若$b=O$，即**齐次线性方程组**中，$\begin{cases}有唯一零解,&|A|\ne0\\一定含有非零解，也即无穷多个解,&|A|=0\end{cases}$
- 行列式某一行(列)元素与本身的代数余子式的乘积为该行列式的值，与**另一行**(列)**对应元素的代数余子式**的乘积为$0$

## 运算技巧

### 特殊行列式

**对角或上三角或下三角**：$\begin{align}|A|=\prod_{n\ge i\ge1}a_{ii}\end{align}$

**类对角或类上三角或类下三角**：$\begin{align}|A|=(-1)^{\frac{n(n-1)}2}\prod_{n\ge i\ge1}a_{ii}\end{align}$

**范德蒙德行列式**：

$\begin{align}\left|\begin{matrix}1&\dots&1\\x_1&\dots&x_n\\x_1^2&\dots&x_n^2\\\vdots&&\vdots\\x_1^{n-1}&\dots&x_n^{n-1}\end{matrix}\right|=\prod_{n\ge i>j\ge1}(x_i-x_j)\end{align}$

例：$\begin{align}\left|\begin{matrix}1&1&1&\dots&1\\2&2^2&2^3&\dots&2^n\\3&3^2&3^3&\dots&3^n\\\vdots&\vdots&\vdots&&\vdots\\n&n^2&n^3&\dots&n^n\end{matrix}\right|\end{align}$，虽然不是范德蒙德行列式，但可以转化为这种形式：每行均可以提出公因子，得原式$\begin{align}=n!\left|\begin{matrix}1&1&1&\dots&1\\1&2&2^2&\dots&2^{n-1}\\1&3&3^2&\dots&3^{n-1}\\\vdots&\vdots&\vdots&&\vdots\\1&n&n^2&\dots&n^{n-1}\end{matrix}\right|=n!(n-1)!\cdots2!\end{align}$

### 简单行列式消零

**一**：

大部分元素相同，而且每行、每列只有一个不同的元素，而且这些元素相同时，例如：

$\begin{align}\left|\begin{matrix}b&a&\dots&a\\a&b&\dots&a\\\vdots&\vdots&&\vdots\\a&a&\dots&b\end{matrix}\right|\end{align}$

将每一行(列)加到第一行(列)，提出公因子$b-(n-1)a$，再通过倍加变换消掉剩余的$a$，可得到大部分元素为$0$的行列式，特别地，在这个例子里，$|A|=\Big(b-(n-1)a\Big)(b-a)^{n-1}$

若观察到**每行(列)元素的和相同**，也可以采用这种方法

若观察发现**类似范德蒙德行列式**，却有某几行有区别，也可采用这种方法

---

**二**：**三对角行列式**

形如$\begin{align}D_n=\left|\begin{matrix}a&b&0&\dots&0\\c&a&b&\dots&0\\0&c&a&\dots&0\\\vdots&\vdots&\vdots&&\vdots\\0&0&0&\dots&a\end{matrix}\right|\end{align}$的三对角元分别相同的行列式，可采用**递推法**

在这个例子里，$D_n=aD_{n-1}-bcD_{n-2}$

设$D_n-xD_{n-1}=y(D_{n-1}-xD_{n-2})$，解得$\begin{align}\begin{cases}x=\large\frac{2bc}{a+\sqrt{a^2-4bc}}\\y=\large\frac{a+\sqrt{a^2-4bc}}2\end{cases}\end{align}$或$\begin{align}\begin{cases}x=\large\frac{2bc}{a-\sqrt{a^2-4bc}}\\y=\large\frac{a-\sqrt{a^2-4bc}}2\end{cases}\end{align}$

递归代入原方程后可得到两条关于$D_n$和$D_{n-1}$的式子，通过消元即可得到$D_n$

这种行列式很特殊，事实上三对角行列式都可以采用递推法

---

**三**：**递推变种**

$(1).$形如$\begin{align}D_n=\left|\begin{matrix}a&b&0&\dots&0\\0&a&b&\dots&0\\0&0&a&\dots&0\\\vdots&\vdots&\vdots&&\vdots\\c&c&c&\dots&a\end{matrix}\right|\end{align}$或$\begin{align}D_n=\left|\begin{matrix}a&0&0&\dots&b\\c&a&0&\dots&b\\0&c&a&\dots&b\\\vdots&\vdots&\vdots&&\vdots\\0&0&0&\dots&a\end{matrix}\right|\end{align}$的行列式也可用**递推法**

例如，在左式例子中，$D_n=aD_{n-1}+(-1)^{n+1}cb^{n-1}$

又例如，$\begin{align}D_n=\left|\begin{matrix}a&-1&0&\dots&0\\0&a&-1&\dots&0\\0&0&a&\dots&0\\\vdots&\vdots&\vdots&&\vdots\\x_1&x_2&x_3&\dots&a+x_n\end{matrix}\right|=aD_{n-1}+x_1\end{align}$

$\therefore D_n=a^n+a^{n-1}x_n+\dots+ax_2+x_1$

**若上式$a=b$**，还可以采用**逐列倍加法**，即把第$i$列的元素的$-1$倍加到第$i+1$列上

$(2).$形如$D_{2n}=\left|\begin{matrix}a&&&&&&&b\\&a&&&&&b\\&&\ddots&&&\dots\\&&&a&b\\&&&c&d\\&&\dots&&&d\\&c&&&&&\ddots\\c&&&&&&&d\end{matrix}\right|$的$2n$阶行列式也可用递推法

$D_{2n}=(ad-bc)D_{2n-2}、D_2=ad-bc\Rightarrow D_{2n}=(ad-bc)^n$

---

**四**：**升阶法**

当行列式每行(列)中有大部分元素相同，其中有部分元素不同时，可采用升阶法

例如$\begin{align}\left|\begin{matrix}a&x_2&x_3&\dots&x_n\\x_1&b&x_3&\dots&x_n\\x_1&x_2&c&\dots&x_n\\\vdots&\vdots&\vdots&&\vdots\\x_1&x_2&x_3&\dots&y\end{matrix}\right|\end{align}$，由于各行、各列和不相同，不能提出公因子

可以转化为$\begin{align}\left|\begin{matrix}1&x_1&x_2&x_3&\dots&x_n\\0&a&x_2&x_3&\dots&x_n\\0&x_1&b&x_3&\dots&x_n\\0&x_1&x_2&c&\dots&x_n\\\vdots&\vdots&\vdots&\vdots&&\vdots\\0&x_1&x_2&x_3&\dots&y\end{matrix}\right|\end{align}$，利用第一行消去其它行的大部分元素：

$\begin{align}\left|\begin{matrix}1&x_1&x_2&x_3&\dots&x_n\\-1&a-x_1&0&0&\dots&0\\-1&0&b-x_2&0&\dots&0\\-1&0&0&c-x_3&\dots&0\\\vdots&\vdots&\vdots&\vdots&&\vdots\\-1&0&0&0&\dots&y-x_n\end{matrix}\right|\end{align}$

再消去第一列的$-1$：$\begin{align}\left|\begin{matrix}{\large1+\frac{x_1}{a-x_1}+\dots+\frac{x_n}{y-x_n}}&x_1&x_2&x_3&\dots&x_n\\0&a-x_1&0&0&\dots&0\\0&0&b-x_2&0&\dots&0\\0&0&0&c-x_3&\dots&0\\\vdots&\vdots&\vdots&\vdots&&\vdots\\0&0&0&0&\dots&y-x_n\end{matrix}\right|\end{align}$

化为上三角行列式

---

**五**：**逐行(列)倍加法**

若大部分相邻两行(列)间的大部分一一对应的元素差距相似，可考虑该方法，例：

$\begin{align}D_n=\left|\begin{matrix}0&1&2&\cdots&n-2&n-1\\1&0&1&\cdots&n-3&n-2\\2&1&0&\cdots&n-4&n-3\\\vdots&\vdots&\vdots&&\vdots&\vdots\\n-2&n-3&n-4&\cdots&0&1\\n-1&n-2&n-3&\cdots&1&0\end{matrix}\right|\end{align}$

行和不相同，列和也不相同，但观察到第$i-1$行的**大部分**元素为第$i$行对应元素减$1$，则逐行向后倍加(第$i+1$行乘$-1$倍加到第$i$行，$i$从$1$开始)：

$\begin{align}&D_n=\left|\begin{matrix}-1&1&1&\cdots&1&1\\-1&-1&1&\cdots&1&1\\-1&-1&-1&\cdots&1&1\\\vdots&\vdots&\vdots&&\vdots&\vdots\\-1&-1&-1&\cdots&-1&1\\n-1&n-2&n-3&\cdots&1&0\end{matrix}\right|=\left|\begin{matrix}0&2&2&\cdots&2&1\\0&0&2&\cdots&2&1\\0&0&0&\cdots&2&1\\\vdots&\vdots&\vdots&&\vdots&\vdots\\0&0&0&\cdots&0&1\\n-1&n-2&n-3&\cdots&1&0\end{matrix}\right|\\&=(-1)^{1+n}(n-1)2^{n-2}\end{align}$
