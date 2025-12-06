---
title: "线性代数: 矩阵"
date: 2024-05-11
categories: [SE Courses, Linear Algebra]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 矩阵

## 概念

形如$\begin{align}\left[\begin{matrix}a_{11}&a_{12}&...&a_{1\rm n}\\a_{21}&a_{22}&...&a_{2\rm n}\\\vdots&\vdots&&\vdots\\a_{\rm m1}&a_{\rm m2}&...&a_{\rm mn}\end{matrix}\right]\end{align}$的矩阵称为$m\times n$矩阵，记为$A_{m\times n}$

- 行矩阵/列矩阵：只有一行/一列的矩阵
- $n$阶矩阵：$A_{n\times n}$
- 三角形矩阵($0$一般省略不写)：三角形矩阵一定是方阵
  - 对角矩阵：$\begin{align}\left[\begin{matrix}a_{11}&0&...&0\\0&a_{22}&...&0\\\vdots&\vdots&&\vdots\\0&0&...&a_{\rm nn}\end{matrix}\right]\end{align}$，记为$\rm diag(a_{11},a_{22},\cdots,a_{nn})$
  - 上三角矩阵：$\begin{align}\left[\begin{matrix}a_{11}&a_{12}&...&a_{1\rm n}\\0&a_{22}&...&a_{2\rm n}\\\vdots&\vdots&&\vdots\\0&0&...&a_{\rm nn}\end{matrix}\right]\end{align}$
  - 下三角矩阵：$\begin{align}\left[\begin{matrix}a_{11}&0&...&0\\a_{21}&a_{22}&...&0\\\vdots&\vdots&&\vdots\\a_{\rm m1}&a_{\rm m2}&...&a_{\rm nn}\end{matrix}\right]\end{align}$
- 单位矩阵：$\rm diag(1,1,\cdots,1)$，记为$E$或$I$
- 同型矩阵：行、列数相同的两矩阵

## 运算

### 基本运算

- 加/减法运算：$A+B$即把各对应的元素相加/减，其中$A、B$必须是同型矩阵
- 数乘运算：$k\bf A$即每个元素都乘上$k$
- 乘法运算：$A_{\rm m\times k}B_{\rm k\times m}=C_{\rm m\times n}$，$\begin{align}c_{ij}=\sum_{a=1}^ka_{\rm ia}b_{\rm aj}\end{align}$

### 运算的性质

加减法与数乘运算的性质不赘述

- 一般$AB\ne BA$，若$AB=BA$，则$A、B$为同阶方阵
- $A\ne O$时，若$AB=AC$，一般$B\ne C$
- 若$AB=O$，一般$A\ne O$或$B\ne O$；若$A\ne O$且$B\ne O$，$AB$可能等于$O$
- $A(BC)=(AB)C$
- $k(AB)=(kA)B=A(kB)$
- $A(B+C)=AB+AC$；$(A+B)C=AC+BC$
- $AE_1=E_2A=A$，若$A$为方阵，则$E_1=E_2$
- 两个同阶且相同类型的三角形矩阵相乘还是对应类型的三角形矩阵，且新矩阵对角元等于两矩阵对角元相乘

## 线性方程组的矩阵形式

形如$\begin{align}\begin{cases}a_{11}x_1+a_{12}x_2+\cdots+a_{1\rm n}x_n=b_1,\\a_{21}x_1+a_{22}x_2+\cdots+a_{2\rm n}x_n=b_2,\\\ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \ \cdots\cdots\cdots\cdots\\a_{\rm m1}x_1+a_{\rm m2}x_2+\cdots+a_{\rm mn}x_n=b_m\end{cases}\end{align}$的$m$条$n$元一次方程组称为$m\times n$型方程组的一般形式

令$\begin{align}A=\left[\begin{matrix}a_{11}&a_{12}&...&a_{1\rm n}\\a_{21}&a_{22}&...&a_{2\rm n}\\\vdots&\vdots&&\vdots\\a_{\rm m1}&a_{\rm m2}&...&a_{\rm mn}\end{matrix}\right],x=\left[\begin{matrix}x_1\\x_2\\\vdots\\x_n\end{matrix}\right],b=\left[\begin{matrix}b_1\\b_2\\\vdots\\b_n\end{matrix}\right]\end{align}$

则$Ax=b$，这是它的矩阵形式，其中$A$称为系数矩阵，$b$称为常数向量

增广矩阵：$[A,b]=\left[\begin{matrix}a_{11}&a_{12}&...&a_{1\rm n}&b_1\\a_{21}&a_{22}&...&a_{2\rm n}&b_2\\\vdots&\vdots&&\vdots&\vdots\\a_{\rm m1}&a_{\rm m2}&...&a_{\rm mn}&b_m\end{matrix}\right]$

当$b=O$时，称为齐次线性方程组

## 转置

### 定义

$A_{n\times m}$称为$A_{m\times n}$的转置矩阵，记为$A^{\rm T}$

### 转置的性质

- $(A+B)^{\rm T}=A^{\rm T}+B^{\rm T}$
- $(AB)^{\rm T}=B^{\rm T}A^{\rm T}$
- 对称矩阵/反称矩阵：设$A$为方阵，若$A^{\rm T}=A$，则为对称矩阵；若$A^{\rm T}=-A$，则为反称矩阵
- 正交矩阵：若$A^{\rm T}A=AA^{\rm T}=E$，称$A$为正交矩阵
- 幂零矩阵：若存在正整数$m$，使$A^m=O$，则称$A$为幂零矩阵
- 幂等矩阵：若$A^2=A$，称$A$为幂等矩阵
- 对合矩阵：若$A^2=E$，称$A$为对合矩阵

## 分块矩阵

分块后矩阵的运算与一般矩阵类似，若$A=\left[\begin{matrix}A_{11}&\cdots&A_{1n}\\\vdots&&\vdots\\A_{m1}&\cdots&A_{mn}\end{matrix}\right]$，则$A^{\rm T}=\left[\begin{matrix}A_{11}^{\rm T}&\cdots&A^{\rm T}_{1n}\\\vdots&&\vdots\\A^{\rm T}_{m1}&\cdots&A^{\rm T}_{mn}\end{matrix}\right]$

### 常用分块方法

- 按行或按列分为$m$个$n$阶行向量或$n$个$m$阶列向量
- 分为$2\times2$的分块矩阵，其中应利用$E$或$O$或三角形矩阵

## 矩阵的初等变换

对一个线性方程组进行操作：

- 调换两条方程的位置
- 方程两边同乘非零常数
- 某行乘以一非零常数后加到另一行

不改变该方程组的解，这些操作称为线性方程组的初等变换

对应到增广矩阵上，这些操作为：

- 对调行变换/对调列变换：对调两行/两列，记为$r_i\leftrightarrow r_j$或$c_i\leftrightarrow c_j$
- 倍乘行变换/倍乘列变换：第$i$行/列乘上一个非零常数，记为$r_i\times k$或$r_i\times k$
- 倍加行变换/倍加列变换：第$i$行/列乘上一个非零常数后加到第$j$行/列上，记为$r_j+kr_i$或$c_j+kc_i$

称为矩阵的初等变换，有限次变换后可逆，称原矩阵和得到的矩阵等价或相抵

### 初等矩阵

由$E$进行一次初等变换后的矩阵称为初等矩阵$(k\ne 0)$：

- 对调矩阵：$E_{i,j}$，即第$i$行/列与第$j$行/列对调，行或列对调得到的矩阵相同
- 倍乘矩阵：$E_i(k)$，即第$i$行/列元素乘上$k$，行或列倍乘得到的矩阵相同
- 倍加矩阵：$E_{i,j}(k)$，即第$j$行/列乘上$k$后加到第$i$行/列上，$E_r=E_c^{\rm T}$

#### 初等矩阵的性质

- $E_{i,j}^{\rm T}=E_{i,j}、E_i^{\rm T}(k)=E_i(k)、E^{\rm T}_{i,j}(k)=E_{j,i}(k)$
- $E_{i,j}E_{j,i}=E_i(k)E_i(k^{-1})=E_{i,j}(k)E_{i,j}(-k)=E，k\ne0$

### 等价标准形

$F=\left[\begin{matrix}E_s&O\\O&O\end{matrix}\right]$称为$A$的等价标准形，$A$通过有限次初等变换可以变成$F$

$F$有特例：$\left[\begin{matrix}E_s&O\end{matrix}\right]、\left[\begin{matrix}E_s\\O\end{matrix}\right]、\left[\begin{matrix}E_s\end{matrix}\right]$

一个矩阵的等价标准形是唯一的

## 矩阵的行列式

- 当一个矩阵是方阵时，才有行列式，记为$|A|$
- $|A^{\rm T}|=|A|$
- $|\lambda A|=\lambda^n|A|$
- $|AB|=|A||B|$
- $|AB|=|BA|$
