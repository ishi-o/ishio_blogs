---
title: "数据结构: 前缀和"
categories: [Algorithm, Data Structure]
tags: [algorithm, data structure, array]
mathjax: false
date: 2025-03-19
---
<!-- placeholder -->
<!-- more -->
### 一维前缀和

- 对一个原数组$a_{1\sim n}$，令$\begin{align}s_i=\sum_{k=1}^ia_k\end{align}$，即称$s$为$a$的前缀和数组

- 不难发现有以下性质：
  - ${\rm len}(a)={\rm len}(s)$
  - 任意以$a_1$为起始的子区间$[a_1,a_i]$，其元素和为$s_i$

- 对于满足如下要求的运算：

  - 元素间运算**满足结合律**
  - 所有元素具有逆元，即**满足消去律**(可由运算结果及其中一个操作数得知另一个操作数)
  - 满足上述要求的区间信息包括加和、乘积、异或等

  令$s_0=0$，则有$sum(a_i,a_j)=s_j-s_{i-1}$

- 原地实现：

  ```c++
  int a[MAXN], t;
  for (int i = 1; i <= n; ++i) {
      cin >> t;
      a[i] += t;
      a[i + 1] = a[i];
  }
  ```

### 二维前缀和

- 类似一维前缀和，对一个$n$行$m$列的二维数组，令$\begin{align}s_{ij}=\sum_{i=1}^n\sum_{j=1}^ma_{ij}\end{align}$，即称$s$为$a$的前缀和数组

- 类似一维前缀和，结合容斥原理，对于特定的运算，任意一个子区间(左上角为$a_{ij}$，右下角为$a_{xy}$)的元素和为$s_{xy}+s_{(i-1)(j-1)}-s_{x(j-1)}-s_{(i-1)y}$，其中$s_{0·}=s_{·0}=0$

- 原地实现：

  ```c++
  int a[MAXN][MAXM], t;
  for (int i = 1; i <= n; ++i) {
      for (int j = 1; j <= m; ++j) {
          cin >> t;
          a[i][j] += t;
          a[i][j + 1] += a[i][j] - a[i - 1][j]; // 容斥
          a[i + 1][j] = a[i][j];
      }
  }
  ```

  也可以不依赖容斥，像扫地一样实现(虽然高维前缀和不实用，但这种方法能比较清晰地算出高维前缀和)：

  ```c++
  for (int i = 1; i <= n; ++i) {
      for (int j = 1; j <= m; ++j) {
          cin >> t;
          a[i][j] += t;
          a[i][j + 1] = a[i][j];
      }
  }
  for (int i = 1; i <= n; ++i) {
      for (int j = 1; j <= m; ++j) {
          a[i + 1][j] += a[i][j];
      }
  }

