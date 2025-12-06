---
title: "大学物理: 经典力学"
date: 2024-10-03
categories: [SE Courses, Physics]
tags: []
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 经典力学

### 质点运动的数学解释

- 位置矢量：质点在指定参考系下的坐标向量，用$\vec r$表示，简称**位矢**
- 运动方程：位置矢量关于时间的方程，用$\vec r=\vec r(t)$表示，也称运动函数
- 速度矢量：位矢对时间的**一阶导数**，用$\bar v(t)$表示，其方向与位移方向一致
- 加速度矢量：速度对时间的**一阶导数**，用$\bar a(t)$表示，其方向**总是指向轨道曲线的内侧**
- 若已知质点的加速度和初始状态，求运动方程相当于解微分方程的初值问题
- 加速度可分解为$\boldsymbol{a}=a_t\boldsymbol{\tau}+a_n\boldsymbol{n}$，其中：
  - $a_t$为切向加速度，$a_n$为法向加速度(向心加速度)，$\boldsymbol{\tau}$为速度的单位切向量，$\boldsymbol{n}$为速度的单位法向量
  - $\begin{align}a_t=\frac{\mathrm dv}{\mathrm dt}、a_n=\frac{v^2}{R}\end{align}$，其中$v$为速度的大小

### 牛顿定律

- 牛顿定律只对惯性系生效
- 牛顿第一定律(惯性定律)：任何不受力物体的物体保持静止或匀速直线运动状态
  - 力是物体运动状态改变的原因
  - 物体具有惯性，即保持其静止或匀速直线运动状态不变的属性
- 牛顿第二定律：
  - 动量$\boldsymbol{p}=m\boldsymbol{v}$
  - 力$\begin{align}\boldsymbol{F}=\frac{\mathrm d\boldsymbol{p}}{\mathrm dt}\end{align}$，当质量为关于$t$的常数时，$\boldsymbol{F}=m\boldsymbol{a}$
  - 力恒定时，质量越大、加速度越小、物体惯性越大

- 牛顿第三定律：相互作用力总是大小相等、方向相反、作用在一条直线上

### 参考系

- 参考系：可以是任意物体，以该物体作为参考系原点，即称其他物体是相对于该参考系的
- 惯性系：在惯性系中，只要物体不受力，它就满足牛一；即惯性系本身不具有加速度
- 绝对时空观：
  - 在不同参考系下(分别记为$S、S'$，原点为$O、O'$，质点为$P$)，满足$\boldsymbol{O'P}=\boldsymbol{r}'(t)、t=t'$
  - 伽利略变换：设$\begin{align}\boldsymbol{u}=\frac{\mathrm d\boldsymbol{OO'}(t)}{\mathrm dt}\end{align}$，则不同参考系下同一质点：
    - 位矢关系：$\boldsymbol r(t)=\boldsymbol r'(t)+\boldsymbol{OO'}(t)$，即$\boldsymbol{OP}=\boldsymbol{OO'}+\boldsymbol{O'P}$
    - 速度关系：$\boldsymbol{v}=\boldsymbol{v}'+\boldsymbol{u}$
    - 加速度关系：$\begin{align}\boldsymbol{a}=\boldsymbol{a}'+\frac{\mathrm d\boldsymbol{u}}{\mathrm dt}\end{align}$

  - 伽利略坐标变换：若两**惯性系**在相对做**匀速直线运动**，则$\boldsymbol{u}=\boldsymbol{C}$，有：$\begin{cases}x=x'+ut\\y=y'\\z=z'\\t=t'\end{cases}$

- 惯性力存在于非惯性系中，且$\boldsymbol{F}_i=-m\boldsymbol{a}$，其中$\boldsymbol{a}$为非惯性系的加速度

### 关于动量

- 动量是牛顿第二定律的另一种形式
- 动量定理：力是动量对时间的导数，冲量是力对时间的积分
  - 冲量$\begin{align}\boldsymbol{I}=\int_{t_1}^{t_2}\boldsymbol{F}(t)\mathrm dt=\int_{t_1}^{t_2}\mathrm d\boldsymbol{p}=\boldsymbol{p}_2-\boldsymbol{p}_1\end{align}$
- 动量守恒定律：若某质点系所受合外力为零，则$\sum_\limits i\boldsymbol{p}_i=\boldsymbol{C}$，例如质点系**静止或做匀速直线运动**
- 质心运动定理：
  - 质心坐标$\begin{align}\boldsymbol{r}_c=\frac{\int_\Omega\rho\ \boldsymbol{r}\mathrm d\Omega}{m}\end{align}$，其中$c$为质心这个点，$\boldsymbol{r}$为质点系中任意一点的位矢
  - 若该质点系满足动量守恒定律，则$\sum_\limits i\boldsymbol{F}_i=m\boldsymbol{a}_c$

### 关于角动量

- 设质点$P$相对于$O$的位矢为$\boldsymbol{r}$，且所受外力为$\boldsymbol{F}$，则定义$\boldsymbol{F}$对$O$的力矩为$\boldsymbol{M}=\boldsymbol{r\times F}$
- 力矩由力的大小、方向、作用点以及某固定点生成
- 角动量(动量矩)：与力矩的定义方式类似，$\boldsymbol{L}=\boldsymbol{r\times p}$，它包含动量对某固定点的角度信息
- 角动量定理：力矩是角动量对时间的导数
- 角动量守恒定律：若某质点系所受**合外力矩**为零，则$\sum_\limits i\boldsymbol{L}_i=\boldsymbol{C}$，例如质点系**做直线运动**或**做匀速圆周运动**

### 关于功和功率

- 力做功$\begin{align}A=\int_L\boldsymbol{F}\mathrm d\boldsymbol{r}=\int_L\boldsymbol{F·v}\ \mathrm dt\end{align}$，恒力做功$A=\boldsymbol{F·\Delta r}$
- 根据格林公式及斯托克斯公式，将做功与路径无关，即$\boldsymbol{F}$的雅可比行列式为$0$的力称为保守力
- 功率$\begin{align}P=\frac{\mathrm dA}{\mathrm dt}=\boldsymbol{F·v}\end{align}$
- 动能定理：$\begin{align}E_k=\frac12mv^2\end{align}$，$A=\Delta E_k$
- **保守内力**做功$A=-\Delta E_p$
- 机械能守恒定律：如果系统只有保守内力做功，则$E_M=E_k+E_p=C$

### 碰撞问题

- 完全非弹性碰撞：动量守恒，但有非保守内力做功，只满足$m_合\boldsymbol{v}_合=m_1\boldsymbol{v}_1+m_2\boldsymbol{v}_2$
- 完全弹性碰撞：动量守恒且机械能守恒
  - 矢量式：$m_1\boldsymbol{v}_{10}+m_2\boldsymbol{v}_{20}=m_1\boldsymbol{v}_1+m_2\boldsymbol{v}_2$
  - $\begin{align}\frac{m_1v_{10}^2}2+\frac{m_2v_{20}^2}2=\frac{m_1v_{1}^2}2+\frac{m_2v_{2}^2}2\end{align}$
  - 若$m_2$初速度为$0$，则$\begin{align}\boldsymbol{v}_1\boldsymbol{·v}_2=\frac{m_1-m_2}{2m_1}v_2^2\end{align}$，即$\begin{align}\cos<\boldsymbol{v}_1,\boldsymbol{v}_2>=\frac{m_1-m_2}{2m_1}\frac{v_2}{v_1}\end{align}$
  - 若为对心碰撞，则可总结出更一般的速度式，因为此时$\cos<\boldsymbol{v}_1,\boldsymbol{v}_2>=-1$，动量守恒式变成标量式：$\begin{align}\begin{cases}v_1=\frac{\large(m_1-m_2)v_{10}+2m_2 v_{20}}{\large m_1+m_2}\\v_2=\frac{\large2m_1v_{10}+(m_2-m_1)v_{20}}{\large m_1+m_2}\end{cases}\end{align}$(借助平方差公式推导)

### 刚体力学

- 角速度大小：$\begin{align}\omega=\frac{\mathrm d\theta}{\mathrm dt}=\frac1r\frac{\mathrm dl}{\mathrm dt}=\frac vr\end{align}$

- 角速度方向：$\boldsymbol{v}=\boldsymbol{\omega\times r}$

- 角加速度和加速度的关系：

  - $\begin{align}a_t=\frac{\mathrm dv}{\mathrm dt}=r\frac{\mathrm d\omega}{\mathrm dt}=r\alpha\end{align}$
  - $\begin{align}a_n=\frac{v^2}{r}=r\omega^2\end{align}$
  - $\begin{align}\boldsymbol{a}=\frac{\mathrm d(\boldsymbol{\omega\times r})}{\mathrm dt}=\boldsymbol{\alpha\times r}+\boldsymbol{\omega\times v}\end{align}$

- 刚体绕定轴转动时，轴向动量和位矢垂直，故轴向角动量的大小$L_i=|\boldsymbol{r\times p}_i|=mr_iv_i=mr_i^2\omega_i$

  定义刚体绕某轴旋转的**转动惯量**为$\begin{align}J=mr_i^2=\int_\Omega \rho\ r_i^2\mathrm d\Omega\end{align}$，则轴向角动量的大小为$L_i=J\omega$，轴向力矩大小为$M_i=J\alpha$

- 刚体定轴转动定律：刚体绕定轴的**角加速度**和**轴向力矩**成正比，类比加速度和力的关系$F=ma$

- 转动问题中，$\begin{align}A=\int_{\theta_1}^{\theta_2}M_i\mathrm d\theta\end{align}$，即外力做的功等于轴向力矩和角位移的积

- 转动动能：$\begin{align}E_k=\frac12J\omega^2\end{align}$
