---
title: "具体数学: 递归"
date: 2024-09-21
categories: [SE Courses, Concrete Math]
tags: [math, recursion]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 递归问题分析

### 递归函数定义

- 递归被用于**定义数学函数**或**归纳证明**
- 递归的数学模型分为**基础**和**递归**两部分
- 通过递归证明命题被称为数学归纳法，以正确的最小命题为基础，分为**递归基础、递归假设、递归步骤**三部分

### 汉诺塔问题

- 问题描述：将$n$个从上到下大小依次递增的盘片将$A$柱借助$B$柱移到$C$柱，其中不允许大盘片在小盘片上，求解移动次数

- 问题分析：将$n$盘从$A$柱移到$C$柱，需先将$n-1$盘通过$C$柱移到$B$柱，再将第$n$盘片直接移到$C$柱，然后将$n-1$盘通过$A$柱移到$C$柱；若只剩一盘，则可直接移动

- 递归式：$\begin{cases}T(1)=1\\T(n)=2T(n-1)+1\end{cases}$，即移动两次$n-1$盘，一次直接移动

- 通项式：$T(n)=2^n-1$，可证明其是最优解

- 代码求解移动过程：

  ```c++
  void RecursionSolution(int n, int from, int use, int to) {
      if (n == 1) {
          // from -> to directly
          cout << from << "->" << to << endl;
          return;
      }
      RecursionSolution(n - 1, from, to, use);
      cout << from << "->" << to << endl;
      RecursionSolution(n - 1, use, from, to);
  }
  int HanoiTower(int n) {
      RecursionSolution(n, 0, 1, 2); // 输出移动过程
      return (1 << n) - 1; // 返回移动次数
  }

### 直线划分平面问题

- 问题描述：用$n$条直线划分一个平面，求出能划出的最多的部分数

- 问题分析：$S(0)=1,S(1)=2$；$n\ge 2$时，每次使其和剩下$n-1$条直线都有交点，则划分部分最多

- 递归式：$\begin{cases}S(0)=1\\S(n)=S(n-1)+n\end{cases}$

- 通项式：$\begin{align}S(n)=\frac{n(n+1)}2+1\end{align}$

- 证明其必定能做到上述操作：将$n-1$次划分后的交点用圆围起来，用直径$l$划分该圆，使交点平均地分布在圆的两部分(若交点数为奇数，则让$l$穿过一点后再平均分配)，则在$l$划分的两个部分中，这些直线在圆外是**不相交**(交点在圆内)且**不为同一直线**(同一直线被$l$划分在其两边)，平移$l$至圆外，则必定能交到$n-1$个交点

- 推广：平面角划分平面问题

  分析：使角始终处于上次划分后区域最大的部分，延长角的两条射线，则转换为$2n$条直线划分平面问题

  递归式：$\mathscr S(n)=S(2n)-2n$

  通项式：$\mathscr S(n)=2n^2-n+1$

- 圆划分平面问题：

  每增加一个圆，新增圆与其余所有圆都相交且有两个交点，新增部分数目与新增交点数目相同，即：

  $T_n=\begin{cases}2,&n=1\\T_{n-1}+2(n-1),&n>1\end{cases}$
  因此$\begin{align}T_n=T_1+2\sum_{i=1}^{n-1}i=2+n(n-1)\end{align}$

### 约瑟夫问题

- 问题描述：使编号依次递增(编号从$i$开始)的人(共有$n$人)按顺序围成一圈，每$m$次报数后踢出该人，求幸存者的编号

- 问题分析：要知道$f(n,m)$，就需要先知道$f(n-1,m)$，但后者从$i$开始，为了使前者能转换成后者，每次踢出后，需要找出其编号对应关系，若用$[k]$表示下一轮报数从$k$开始，则：$\begin{cases}f(n,m)踢出一人:i,i+1,\cdots,i+m-2,[i+m],\cdots,i+n\\f(n-1,m):[i],i+1,\cdots,i+n-1\end{cases}$

  当$i=0$时，可找到对应关系：$f(n,m)=f(n-1,m)\%\ n$

  使$[]$括起的编号一一对应后，可找到如下关系：$f(n,m)=(f(n-1,m)+m-i)\%\ n+i$

  其中$i$为偏移量，减去$i$即转换成编号从$0$开始的情况，最后加上$i$得到原问题从$i$开始的情况

  上述情况假设$m<n$，实际上$m\ge n$的情况下，踢出一人后报数从$i+(m\%\ n)$开始，因模运算的性质也满足上述递归式

- 递归式：$\begin{cases}f(1,m)=i\\f(n,m)=(f(n-1,m)+m-i)\%\ n+i\end{cases}$，从$i$开始编号

  若要求从$j(i\le j\le i+n)$开始报数，可先求从$i$开始报数，然后增加$j-i$(注意模$n$)

- 递归求解代码：

  ```c++
  int Solution(int n, int m, int i) { // 从i开始编号, 从i开始报数
      return n == 1 ? i : (Solution(n - 1, m, i) + m - i) % n + i;
  }
  int Josephus(int n, int m, int i, int j) { // 从i开始编号, 从j开始报数
      return (Solution(n, m, i) - i) % n + j - i;
  }

### $f(n,2)$及推广的通项式求解

上述过程实际只给出了递归式，实际上当$m=2$时，原问题是以下问题的一个特例：

- 给出递归式$\begin{cases}f(j)=\alpha_j,&0\lt j\lt m\\f(mn+j)=cf(n)+\beta_j,&0\le j\lt m,n>0\end{cases}$，求出$f(x),x\in N^+$

- 清单法(猜证法)：即给定$f(n)$可能的形式，通过给定特定的$f(n)=X(n)$(要求$X(n)$能求出唯一对应的$(\alpha_j,\beta_j)$值)，反过来推出$f(n)$中各系数的值

  容易得到，如果递推式中含$k$个变量，则至少需要找出$k$个上述函数

- 当$m=c=2$时，即所谓$f(n,2)$，可很容易地找到三个函数：
  
  猜测
  
  令$f(n)=1,解得\begin{cases}\alpha_1=1\\\beta_0=\beta_1=-1\end{cases}$，即$A-B-C=1$
  
  令$f(n)=n,解得\begin{cases}\alpha_1=\beta_1=1\\\beta_0=0\end{cases}$，即$A+C=n$
  
  令$f(n)=2^k(其中n=2^k+l),解得\begin{cases}\alpha_1=1\\\beta_0=\beta_1=0\end{cases}$，即$A=2^k$
  
  联立得$\begin{cases}A=2^k\\B=2^{k+1}-n-1=2^k-l-1\\C=n-2^k=l\end{cases}$，即$f(n)=2^k\alpha_1+(2^k-l-1)\beta_0+l\beta_1$
  
  - 当$\alpha_1=1、\beta_0=-1、\beta_1=1$时，正是$f(n,2)=2l+1$
  
- 很不幸，无法给出如此多的函数来使用清单法求出一般形式的通解，但能根据递推式给出如下关系：

  令$n=(n)_m=b_kb_{k-1}\cdots b_0$，每右移一位相当于取$nm+j$中的$n$，原本的末位则是$j$：

  $\begin{align}&f((n)_m)=cf((n)_m>>1)+\beta_{b_0}=c(cf((n)_m>>2)+\beta_{b_1})+\beta_{b_0}=\cdots\\&=c^kf(b_k)+c^{k-1}\beta_{b_{k-1}}+\cdots+c\beta_{b_1}+\beta_{b_0},其中b_k<m,故f(b_k)=\alpha_{b_k}\end{align}$

  因此对任何$n$，将其写成$m$进制，则$f(n)$等于$(\alpha_{b_k}\beta_{b_{k-1}}\cdots\beta_{b_0})_c$

  当限定为约瑟夫问题$f(n,2)$时，$f(n)=(n)_2循环左移一位$
