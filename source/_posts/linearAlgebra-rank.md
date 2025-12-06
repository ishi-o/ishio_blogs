---
title: "线性代数: 矩阵的秩"
date: 2024-05-31
categories: [SE Courses, Linear Algebra]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 矩阵的秩

## 矩阵判别

### 秩的性质

- $r(A_{m\times n})\le\min(m,n)$，换句话说，如果确定$r(A)\ge m$或$n$，则$r(A)=m$或$n$
- 通过将矩阵化为行阶梯型或行最简型求具体矩阵的秩，如果求线性方程组系数阵的秩，应该只进行初等行变换；有时也通过定义求解

### 秩与其它运算

- $r(lA)=\begin{cases}r(A)&l\ne0\\0&l=0\end{cases}$
- $r(A^{-1})=r(A)=n、r(A)\begin{cases}<n&|A|=0\\=n&|A|\ne0\end{cases}$
- $P、Q$可逆$\Leftrightarrow r(PAQ)=r(A)$
- $r(A\pm B)\le r(A)+r(B)$，通常使用右$\rightarrow$左限定范围
- $\begin{align}r(A)+r(B)-n\le r(A_{m\times n}B_{n\times m})\le \min\Big(r(A),r(B)\Big)\end{align}$，乘积秩不增加
- $r(A^*)=\begin{cases}n&r(A)=n\\1&r(A)=n-1\\0&r(A)<n-1\end{cases}$
- $r(AA^T)=r(A^TA)=r(A)=r(A^T)$
  - $A^TAx=0$与$Ax=0$同解$\Rightarrow k-r(A^TA)=k-r(A)\Rightarrow r(A^TA)=r(A)$
  - $AA^Tx=0$与$A^Tx=0$同解$\Rightarrow k-r(AA^T)=k-r(A^T)\Rightarrow r(AA^T)=r(A^T)=r(A)$
- 分块矩阵：
  - $r\left(\begin{matrix}A&C\\O&B\end{matrix}\right)\ge r(A)+r(B)$，若$C=O$，则$=r(A)+r(B)$
  - 增广矩阵$\max(r(A),r(B))\le r(A,B)\le r(A)+r(B)$
  - $r(A,B)=r(B,A)$

## 向量组

### 线性相关

- 定义：若存在一组**不全为$0$**的数$k_1,k_2,\cdots,k_n$，使$0$能够被向量组$\alpha_1,\alpha_2,\cdots,\alpha_n$线性表示，则称该向量组**线性相关**
- 性质：
  - 线性无关组**删去向量仍无关**、线性相关组**添加向量仍相关**
  - 线性无关组**添加分量仍无关**、线性相关组**删去分量仍相关**
  - 向量组$A(\alpha_1,\alpha_2,\cdots,\alpha_n)$线性无关，$B(A,\beta)$线性相关$\Leftrightarrow\beta$可由$A$**唯一线性表示**
  - 一组向量线性相关$\Leftrightarrow$不满秩$\Leftrightarrow$极大无关组个数小于$n$$\Leftrightarrow$至少有一个向量能被其余向量表示
  - 一组向量线性无关$\Leftrightarrow$定义式中系数$k$全为且只能为$0$，故可用待定系数法求解线性无关问题
- 极大无关组
  - 向量组中任意一个向量都能被极大无关组表示
  - 等价矩阵的极大无关组

### 秩与等价

- 秩的判别：
  - 定义：存在$r$个向量线性无关，任意$r+1$个向量线性相关
  - 极大线性无关组不唯一
  - 化为矩阵判别更高效
- 若两个向量组能**互相线性表示**，称为**等价**，等价性质：
  - 等价矩阵对应的向量组也等价，且**极大无关组的序号**一致，**线性相关性**也一致
  - 子向量组可由父向量组线性表示
  - **向量组与其极大无关组**等价、**向量组的两个极大无关组**等价
  - $A$可由$B$线性表示$\Leftrightarrow r(B)=r(B,A)\le r(A)+r(B)\Leftrightarrow r(A)\le r(B)$$\Leftrightarrow$存在$K$，使$A=BK$，其中设$\begin{cases}\alpha_1=B\vec k_1\\\cdots\\\alpha_n=B\vec k_n\end{cases}$，则$K=\Big(k_1,k_2,\cdots,k_n\Big)$，$k_i$为列向量
  - 若$A$可由$B$线性表示且两向量组的**向量个数相同**，则证明$A\equiv B$就是证明$K$可逆
  - $A、B$等价$\Leftrightarrow r(A)=r(A,B)=r(B)$
  - 反推，如果$A$不可由$B$线性表示$\Leftrightarrow r(B)<r(B,A)\Leftrightarrow r(B)<r(A)$

## 线性方程组中的应用

### 齐次线性方程组

- 设$n$为**未知量的个数**
- 齐次线性方程组一定有解，即$r(A)\equiv r(A,\beta)$，$\begin{cases}r(A)=n&有唯一零解\\r(A)<n&有无穷解\end{cases}$
- 若有无穷解：
  - 基础解系为解向量组的极大无关组，且个数为$n-r(A)$
  - 求基础解系的过程就是**找出$n-r(A)$个线性无关的解组**
  - 通解为$\begin{align}\sum_{i=1}^{n-r(A)}k_i x_i\end{align}$
  - 在齐次方程组中，找另一个基础解系的过程就是找**和已知基础解系等价**的向量组

### 非齐次线性方程组

- $\begin{cases}r(A)\ne r(A,\beta)&无解\\r(A)=r(A,\beta)=n&有唯一解\\r(A)=r(A,\beta)<n&有无穷解\end{cases}$
- 若有无穷解：
  - 基础解系仍是解向量组的极大无关组，且个数为$n-r(A)+1$
  - 对应**齐次线性方程组的基础解系**，再加上**本身的一个特解**，不是基础解系，但构成它的通解
  - 由于$\beta\ne0$，方程组的**两个不同解**一定**线性无关**
  - 通解为$\begin{align}X+\sum_{i=1}^{n-r(A)}k_i x_i\end{align}$，其中$x_i$为齐次线性方程组的解
- 无论$Ax=\beta$是否有解，$A^TAx=A^T\beta$一定有解
  - $r(A^TA)\le r(A^TA,A^T\beta)=r(A^T(A,\beta))\le r(A^T)=r(A^TA)$

### 求解过程

- 对一个非齐次线性方程组，有$Ax=b$，先找$Ax=0$的通解，再找$Ax=b$的一个特解
- 将$A$化为**行最简型**，找出自由变量(非阶梯处，个数一定为$n-r(A)$个)，**依次令其中一个为$1$，同时其余自由变量为$0$**，解出$n-r(A)$个解向量，即为基础解系
- 回到$Ax=b$，**令所有自由变量为$0$**，解出的解向量即为特解
- **齐次**线性方程组**解的线性组合**仍是该方程组的解
- **一个非齐次**线性方程组的解和**多个**对应齐次线性方程组的**解的线性组合**仍是该**非齐次**线性方程组的解
- **非齐次**线性方程组的**解的线性组合**仍为该方程组的解$\Leftrightarrow$**系数和为$1$**
- **非齐次**线性方程组的**解的差**是对应**齐次**线性方程组的解

## 条件推理复习

$\begin{align}&1.由线性方程组Ax=b解的情况推出秩(共有n个未知量):\\&1).无解:r(A)\ne r(A,b)\\&2).有唯一解:r(A)=r(A,b)=n\\&3).有无穷解:r(A)=r(A,b)<n;\ 其中基础解系个数为\ n-r(A)\\\\&2.三个空间平面方程的关系:\\&1).重合\Rightarrow有无穷解且可化为1条方程\Rightarrow r(A)=r(A,b)=1\\&2).交于一条直线\Rightarrow有无穷解且只能化为2条方程\Rightarrow r(A)=r(A,b)=2\\&3).交于一点\Rightarrow有唯一解\Rightarrow r(A)=r(A,b)=3\\&4).平行\Rightarrow无解\Rightarrow r(A)\ne r(A,b)\\\\&3.给出齐次线性方程组的一个基础解系A,同时给出可由A表出的B,判断B是否也是一个基础解系\\&-\ B可由A表出\Rightarrow B也是解\Rightarrow当B与A等价时,也为基础解系\Rightarrow写出B=AK,K可逆时,结论成立\end{align}$
