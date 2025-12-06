---
title: "机器学习: 全连接层"
date: 2025-07-15
categories: [ML/DL]
tags: [DL]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
### 全连接层

- 全连接层指本层的**每一个结点**都和上一层的**所有结点相连接**

- 全连接层适合处理**特征数量较少**的**全局特征数据**，要求各条样本之间**不含空间或时间上的相关性**

- 全连接层通常作为**任务头**，用于**生成输出**而不是中间计算，例如作为输出层

- 全连接层在不同深度学习框架中的命名有所不同，例如：

  - `Affine`层(仿射变换层)：根据其数学本质(线性变换与偏移)命名
  - `Linear`层(线性层)
  - `Dense`层(密集层)

- 具体结构：

  - 全连接层的输入和输出均为**特征展平的列向量**$\boldsymbol x$，本层输入的元素个数和上一层结点数相同、本层输出的元素个数和本层结点数相同

  - 权重为**二维矩阵**$W$，$W_{i,j}^{(l)}$表示第$l-1$层第$i$个结点到第$l$层的第$j$个结点的权重

    因此若第$k$层结点数为$c^{(k)}$，则$W^{(l)}$是一个$c^{(l-1)}$行$c^{(l)}$列的矩阵

  - 偏置则为列向量$\boldsymbol b$，它的元素个数与所在层的结点数相同

- 全连接层的`FP`(此后假设同一层的激活函数相同)：
  $$
  \begin{align}&x_{j}^{(l+1)}=g_j^{(l)}\left(\left(W_{:,j}^{(l)}\right)^T\boldsymbol x^{(l)}+\boldsymbol b^{(l)}\right)\\&\boldsymbol x^{(l+1)}=g^{(l)}\left(\left(W^{(l)}\right)^T\boldsymbol x^{(l)}+\boldsymbol b^{(l)}\right)\end{align}
  $$

- 全连接层的`BP`梯度计算：
  $$
  \begin{align}&\boldsymbol\delta^{(l)}=\begin{cases}{\Large\frac{\part L(\hat y)}{\part\hat y}\frac{\part g^{(l)}(Y^{(l)})}{\part Y^{(l)}}},&l为输出层\\W^{(l+1)}\boldsymbol\delta^{(l+1)}{\Large\frac{\part g^{(l)}(Y^{(l)})}{\part Y^{(l)}}},&l为隐藏层\end{cases}\\&\frac{\part L}{\part W^{(l)}}=\boldsymbol x^{(l)}(\boldsymbol\delta^{(l)})^T\\&\frac{\part L}{\part\boldsymbol b^{(l)}}=\boldsymbol\delta^{(l)}\end{align}
  $$
  
