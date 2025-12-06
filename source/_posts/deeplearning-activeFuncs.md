---
title: "机器学习: 激活函数"
date: 2025-07-09
categories: [ML/DL]
tags: [DL]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 激活函数

### 基本概念

- 激活函数是神经网络中十分重要的一部分，用于对线性叠加后的输出添加非线性因素，增大模型容量，实现对非线性问题的拟合
- 通常激活函数会**限制**输出的**值域**，控制均值
  如果一个结点的激活值为零(或接近零)，则它对下一层所有结点的贡献均为零(或很小)，形象地称该神经元死亡
- 激活函数应该满足：**非线性**、连续**可导**、部署在隐藏层和输出层中

### 常见激活函数

- `Sigmoid`(又称`Logistic`)：$\begin{align}\sigma(x)=\frac1{1+e^{-x}}、\sigma'(x)=\frac{e^{-x}}{(1+e^{-x})^2}=\sigma(x)(1-\sigma(x))\end{align}$
  - 定义域为$(-\infty,+\infty)$，值域为$(0,1)$，是一种二分类函数
  - 性质：严格单调递增、输出范围有界且狭小
  - 缺点：
    - 输入的绝对值较大时，输出达到饱和，对输入变化不敏感，容易丢失这部分的梯度信息
    - 输出恒大于零，意味着均值不为零，将导致后续神经元的输入也是非零均值的
- `Tanh`(双曲正切)：$\begin{align}\tanh(x)=\frac{e^x-e^{-x}}{e^x+e^{-x}}=2\sigma(x)-1\end{align}$
  - 定义域为$(-\infty,+\infty)$，值域为$(-1,1)$
  - 解决了`Sigmoid`函数非零均值的问题
- `ReLU`(`Rectified linear unit`，整流线性单元)：$f(x)=\max(0,x)、f'(x)=\begin{cases}1,&x>0\\0,&x\le0\end{cases}$
  - 是最常用的激活函数
  - 性质：计算简单、求导简单、正输入的输出不会饱和
  - 缺点：
    - 非零均值
    - `Dead ReLU`(神经元坏死)：负输入的输出为零，容易导致死一大片神经元
- `Leaky ReLU`：$\begin{align}&f(x)=\max(\alpha x,x),&\alpha\text{为十分小的固定正数}\end{align}$
  `PReLU`：$\begin{align}&f(x)=\max(\alpha x,x),&\alpha\text{为十分小的正数,可在学习过程中改变}\end{align}$
  - 负输入的输出将向下渗漏，不恒为零
  - 前者的参数是固定的，使函数近似线性，后者的$\alpha$则为超参数
- `ELU`：$f(x)=\begin{cases}x,&x>0\\\alpha(e^x-1),&x\le0\end{cases}$
- `Softmax`：
