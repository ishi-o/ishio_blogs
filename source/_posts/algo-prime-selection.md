---
title: "数论: 质数筛法"
categories: [Algorithm, Math]
tags: [algorithm, math]
mathjax: true
date: 2025-04-22
---
<!-- placeholder -->
<!-- more -->
### 埃氏筛法

- 埃氏筛法，全名埃拉托斯特尼筛法
  若一个数是素数，那么必然不可能存在有除`1`和它自身的因子，也就是比它小的数(除了`1`)都无法整除它
  那么，从小到大遍历除`1`外的正整数，将所有素因子的倍数都标记上，如果遍历到一个数`i`，它没有被标记，那么它就是素数；初始时，`notprime[2]=false`(即`2`是素数)：

  ```c++
  bool notprime[MAXN];
  void getPrime() {
      for (int i = 2; i < MAXN; ++i) {
          if (!notprime[i]) { // 说明它是素数
              for (int j = 2; i * j <= MAXN; ++j) {
                  notprime[i * j] = true;
              }
          }
      }
  }
  ```

  可以简单地写出时间表达式：$\begin{align}\left\lfloor\frac{N}{2}\right\rfloor+\left\lfloor\frac{N}{3}\right\rfloor+\left\lfloor\frac{N}{5}\right\rfloor+\cdots+\left\lfloor\frac{N}{p}\right\rfloor\end{align}$，$p$为小于或等于`MAXN`的最大质数，证明过程是很难的，只需要知道埃氏筛法的时间复杂度为$O(n\log(\log(n)))$即可
  也就是说，在$10^6$数量级内，且需要快速判断一个大范围内的素数时，埃氏筛法是有效的

### 欧拉筛法
