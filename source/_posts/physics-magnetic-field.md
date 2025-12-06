---
title: "大学物理: 恒定磁场"
date: 2024-10-11
categories: [SE Courses, Physics]
tags: []
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 恒定磁场

### 电流的磁效应

- 真空磁导率：$\mu_0=4\pi\times 10^{-7}\rm\ T·m·A^{-1}$

- 磁感应强度：$1\rm T=10^4Gs$，定义运动正电荷所受磁力方向和其运动方向的叉乘为磁感应强度的方向

- 毕奥-萨伐尔定律：$\begin{align}\mathrm d\boldsymbol B=\frac{\mu_0}{4\pi}\frac{I\mathrm d\boldsymbol{l\times e_r}}{r^2}\end{align}$
  可配合磁场叠加原理求解场点的磁场，求解闭合回路电流产生的磁场时，需注意**电流分流**的影响
  
  - 载流直线产生磁场：$\begin{align}B=\frac{\mu_0I}{4\pi r}(\cos\alpha_1-\cos\alpha_2)\end{align}$，其中$r$为场点到直线的距离，$\alpha_1、\alpha_2$分别为沿电流方向的起点、终点与场点的**连线到载流直线**的**夹角**
  - 无限长载流直线：$\begin{align}B=\frac{\mu_0I}{2\pi r}\end{align}$
  - 圆弧电流在中心的磁场：$\begin{align}B=\frac{\theta}{2\pi}\frac{\mu_0I}{2r}\end{align}$，其中$\theta$为圆弧的圆心角
  
- 磁场的高斯定理：$\begin{align}\int\kern{-8pt}\int_\limits S\kern{-23mu}\bigcirc\boldsymbol{B·}\mathrm d\boldsymbol S\equiv0\end{align}$，通过闭合曲面的磁通量为零，揭示磁场是无源场

- 磁感应线的性质：无头无尾、不相交、环绕方向和$I$构成右手螺旋关系

- 安培环路定理：$\begin{align}\oint\boldsymbol{B·}\mathrm d\boldsymbol l=\mu_0\sum I\end{align}$，其中$I$为**穿过**闭合路径**内**的**闭合电流**
  可通过它求高度对称磁场的磁感应强度
  
  类比空间所有电荷对电场强度的贡献，空间任意一点的磁感应强度由空间中所有电流贡献
  
  - 均匀密绕长直载流螺线管：$\begin{align}\boldsymbol B=n\mu_0I\end{align}$，其中$n$为单位长度线圈匝数
  - 环形螺线管：$\begin{align}\boldsymbol B=\frac{\mu_0NI}{2\pi r}\end{align}$
  - 无限大载流导体薄板：$\begin{align}\boldsymbol B=\frac{n\mu_0I}2\end{align}$

### 磁场对实物的作用

- 洛伦兹力：$\boldsymbol F_m=q\boldsymbol{v\times B}$
- 载流子浓度：$\begin{align}n=\frac{N}{V}\end{align}$，**单位体积**内载流子的**数目**
- 霍尔效应与洛伦兹公式：$\boldsymbol F=q\boldsymbol E+q\boldsymbol{v\times B}$，体现在一小片半导体中时，产生的横向电场的霍尔电势差$\begin{align}V_H=R_H\frac{IB}{b}\end{align}$，其中$\begin{align}R_H=\frac1{ne}\end{align}$为霍尔系数，$b$为横向宽度，特别地称$\begin{align}K_H=\frac1{neb}\end{align}$为霍尔灵敏度
  - 由$\begin{align}I=\frac{\mathrm dQ}{\mathrm dt}=nqvS\end{align}$，即单位时间内通过横截面的电荷量(单位时间内，流向位移为$v$，则$vS$这个体积块内的电荷均可通过横截面，其数目为$nvS$，乘上$q$即为总电荷量)
  - 设半导体纵向(霍尔电场方向)宽度为$a$，横向(磁场方向)宽度为$b$，则平衡时有$\begin{align}I=nqvab、E=vB=\frac{V_H}{a}\end{align}$
    整理得$\begin{align}V_H=\frac{IB}{neb}\end{align}$
  - 由霍尔效应可判断半导体中载流子的正负性、制造速度选择器等
- 电流元(可视作极短直线的恒定电流)所受安培力：$\mathrm d\boldsymbol F=I\mathrm d\boldsymbol{l\times B}$

  - 电流元所受安培力是$N$个定向移动电荷所受**洛伦兹力的分力的合力**，其中$\begin{align}N=nS\mathrm dl\end{align}$($\mathrm d\boldsymbol l$和$\boldsymbol v$方向一致)，故
    $\begin{align}\mathrm d\boldsymbol F=N\boldsymbol F_m=qN(\boldsymbol{v\times B})=nqvS\mathrm dl(\boldsymbol{e_v\times B})=I\mathrm d\boldsymbol {l\times B}\end{align}$
  - 均匀磁场中，任意闭合线圈所受磁场**合力为零**，但作用线不在一条直线上时会产生磁力矩
- 磁矩：$\boldsymbol m=I\boldsymbol S=IS\boldsymbol e_s$，表示线圈电流产生磁场的能力，其中磁矩方向和电流方向符合右手螺旋定则
- 闭合带电线圈所**受外磁场**的**合**磁力矩：$\begin{align}\boldsymbol M=\boldsymbol{m\times B}\end{align}$，和线圈旋转方向构成右手螺旋关系
  - 公式中，$\boldsymbol B$为外磁场，而不是线圈产生的磁场
  - $\begin{align}\boldsymbol M=\oint _L\boldsymbol{r\times F}=I\oint_L(\boldsymbol {r\times}\mathrm d\boldsymbol{l\times B})=I\boldsymbol {S\times B}=\boldsymbol{m\times B}\end{align}$

### 磁力做功问题

- 首先明确，磁场是非保守力场，即磁力做功与路径有关，故一个位置相对另一个位置而言，没有固定的能量差，因此没有磁势的概念

- 洛伦兹力**对电子永不做功**，安培力实际上是洛伦兹力在垂直导线方向上的分力，因此**会对导线做功**

  实际上，**安培力做功等于感应电动势做功**，恒流情况下，$\begin{align}A=I\Delta\phi_m\end{align}$

- 刚体力学引理：力矩做功实际上就是力做功，通过数学推导得$\mathrm dA=\boldsymbol{F·}\mathrm d\boldsymbol r=M\mathrm d\theta$，方便刚体转动计算

  匀强磁场对闭合曲线的合力为零，但除去特定角度会产生磁力矩，而磁力矩会对线圈做功

  可以发现，在匀强磁场中，磁力对闭合线圈做功$\begin{align}A=\int_{\phi_1}^{\phi_2}I\mathrm d\phi_m=\int_{\phi_1}^{\phi_2}I\mathrm d(BS\cos\theta)=-\int_{\theta_1}^{\theta_2}IBS\sin\theta\mathrm d\theta=-\int_{\theta_1}^{\theta_2}M\mathrm d\theta\end{align}$

  即，该做功只与$\theta(\boldsymbol m,\boldsymbol B)$的**始末位置**有关，仿照电势能的概念，引入**磁矩的势能**，由上式可看出：

  磁矩势能的减少值即为磁力矩的做功，当**磁矩势能为零**时，**磁通量达到最大值**，线圈处于**稳定平衡**状态

  实际上，在一般磁场中也适用

  - 磁铁相互吸引的原理就是，磁力将磁矩势能转换为磁铁的动能，使其相互靠近

### 磁介质对磁场的作用

- 相对磁导率：$\mu_r$，根据其略小于一、略大于一、远大于一分为下面三类

- 磁介质的分类：顺磁质、抗磁质、铁磁质

- 磁化机理：

  - 分子可等效为具有固有磁矩的圆电流，根据$\begin{align}\sum\boldsymbol m\end{align}$(分子的固有磁矩)等于/不等于零，可区分抗磁质和顺磁质
  - 磁介质在常态没有磁性的原因是：无规律等概率的热运动使介质的所有分子的固有磁矩和为零
  - 外加磁场后，顺磁质的固有磁矩受磁场作用产生磁力矩，使所有分子电流趋向磁场方向，达到增强磁场的作用
  - 所有磁介质都有抗磁能力(磁通量增大时产生的反向感应电流会产生反向磁场)，只是顺磁质的顺磁作用远强于抗磁作用
  - 顺磁质、抗磁质的$\mu_r$为常数，而铁磁质的$\mu_r$不是线性的，受温度、磁化历史等影响
    - 磁饱和：当激励电流达到一定程度时，产生的附加磁场有饱和值
    - 剩磁现象：卸去磁化后的铁磁质的激励电流后，附加磁场有剩余，需加反向激励电流才能消磁
    - 矫顽力：使能消磁的附加的反向磁场强度
    - 磁滞现象：附加磁场从正饱和到负饱和的曲线和从负饱和到正饱和的曲线呈中心对称

- 根据铁磁质的磁滞回线形状，分为硬、软、矩磁材料：

  - 硬磁：矫顽力大，作永久磁铁
  - 软磁：矫顽力小，作变压器
  - 矩磁：剩磁和磁饱和近似相等，作磁盘等记忆元件

- 磁介质中的安培环路定理：$\begin{align}\oint_L\boldsymbol{H·}\mathrm d\boldsymbol l=\sum I_{传导电流}\end{align}$，其中$\boldsymbol H$为磁场强度且$\boldsymbol B=\mu_0\mu_r\boldsymbol H$

  - 推导前已知：磁化程度$\boldsymbol M$，且$\begin{align}\oint_L\boldsymbol{M·}\mathrm d\boldsymbol l=\sum I_{束缚}、\boldsymbol M=\chi_m\boldsymbol H\end{align}$

  - 推导：$\begin{align}&\oint_L\boldsymbol{B·}\mathrm d\boldsymbol l=\mu_0\sum\left(I_{传导}+I_{束缚}\right)=\mu_0\sum I_{传导}+\mu_0\oint_L\boldsymbol{M·}\mathrm d\boldsymbol l\\&\Rightarrow\oint_L\left(\frac{\boldsymbol B}{\mu_0}-\boldsymbol M\right)\mathrm d\boldsymbol l=\sum I_{传导}\end{align}$

    令$\begin{align}&\boldsymbol H=\frac{\boldsymbol B}{\mu_0}-\boldsymbol M\end{align}$，则$\boldsymbol B=\mu_0(1+\chi_m)\boldsymbol H$

    令$\mu_r=1+\chi_m$，得$\boldsymbol B=\mu_0\mu_r\boldsymbol H$

### 磁场的应用

- 质谱仪：$\begin{align}&\frac{mv^2}{2}=qU、\frac{mv^2}{r}=qvB\\&\Rightarrow v=\sqrt{\frac{2qU}{m}}、r=\frac1B\sqrt{\frac{2mU}{q}}\end{align}$
- 回旋加速器
- 磁聚焦
- 霍尔效应
- 速度选择器
- 磁流体发电机
