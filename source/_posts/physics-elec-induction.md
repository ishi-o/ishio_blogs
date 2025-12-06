---
title: "大学物理: 电磁感应"
date: 2024-10-14
categories: [SE Courses, Physics]
tags: []
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 电磁感应

### 电动势

- 电磁感应现象的本质是产生电动势来阻止磁通量的变化，若形成闭合回路，则表现出感应电流

- 电动势定义：

  - 把单位正电荷从负极板移至正极板，**非静电力**所做的功，是标量
  - 非静电性场的场强沿从负极板到正极板的直线路径的线积分，即$\begin{align}\mathscr E=\int_-^+\boldsymbol E_k\boldsymbol ·\mathrm d\boldsymbol l\end{align}$
  - 和电势差的意义完全不一样，电动势始终**指向电势升高**的方向
  - 展开后可得，$\begin{align}\mathscr E=-\frac{\mathrm d\left(\iint\boldsymbol {B·}\mathrm d\boldsymbol S\right)}{\mathrm dt}\end{align}$

- 法拉第电磁感应定律：$\begin{align}\mathscr E=-\frac{\mathrm d\phi_m}{\mathrm dt}\end{align}$

  - 感应电动势的方向和感应电流的流向相同，且总是阻碍引起感应电流的原因

    判断时，可先判断感应电流磁场的方向，再得到感应电流的方向

  - 多匝线圈的磁通量称为全磁通，若每匝磁通相同，也称磁通链数

  - 其符号表示**参考方向和实际方向的区别**，实际方向以楞次定律判断后，若求出负值，说明参考方向和实际方向相反；以电磁感应定律求出的电动势的参考方向和原磁场方向呈右螺旋关系

- 动生电动势：$\begin{align}\mathscr E=\int_a^b(\boldsymbol{v\times B})\boldsymbol ·\mathrm d\boldsymbol l\end{align}$，由洛伦兹力充当非静电力，由$\boldsymbol{v\times B}$充当非静电性场

  - 动生电动势公式在参考方向和实际方向的区别体现在线积分的方向，若线积分方向与实际方向相反，则求出的电动势为负值
  - 可由$\boldsymbol{v\times B}$求出电动势方向

- 感生电动势：$\begin{align}\mathscr E=\oint_L\boldsymbol{E_k·}\mathrm d\boldsymbol l=-\iint_S\frac{\part\boldsymbol B}{\part t}\boldsymbol ·\mathrm d\boldsymbol S\end{align}$，其中闭合回路方向和磁场呈右螺旋关系

  - 感生电场由变化的磁场产生，是无源有旋场，因参考方向取上述方向，故根据电磁感应定律在前面有负号

### 自感和互感和磁能

- 磁通量出发，将所有与线圈本身有关的物理量合并成**自感系数**$\begin{align}L=\frac{\phi_m}{I}\end{align}$

- 由法拉第电磁感应定律，自感电动势$\begin{align}\mathscr E=-L\frac{\mathrm dI}{\mathrm dt}\end{align}$

- 自感系数的一种求法：假设通有电流$I$，求出其磁通量；例如长直螺绕管的自感系数$\begin{align}L=\mu n^2V\end{align}$

- 自感系数的三层含义：

  - 线圈自感效应大小的量度
  - 线圈电磁惯性大小的量度
  - 线圈储存磁能本领的量度

- 互感系数$M$比自感系数多出一种影响因素，即有效占据面积，当两线圈所围平面同轴平行时，磁通量最大

  两线圈组成的系统中，互感系数相等

  计算$1$对$2$的互感系数时，需使用$1$的$n$，$2$的$I$，面积取**有效占据面积**

- 线圈的磁能：$\begin{align}W_m=\frac{LI^2}{2}\end{align}$

- 磁场任意点的磁能密度：$\begin{align}w_m=\frac{BH}{2}=\frac{\mu H^2}{2}=\frac{B^2}{2\mu}\end{align}$

### 麦克斯韦方程组

- 位移电流：(关于电流的连续性)变化的电场在周围激发的磁场等效于电流，其定量描述为：

  全电流安培环路定理：$\begin{align}\oint_L\boldsymbol{H·}\mathrm d\boldsymbol l=I_c+\iint\limits_S\frac{\part\boldsymbol D}{\part t}\boldsymbol{·}\mathrm d\boldsymbol S\end{align}$，其中$I_c$为传导电流

  - 由平行板电容器引出，电位移通量与极板上积聚的电荷相同，且电位移通量对时间的偏导在数值上等于传导电流、方向上与其一致

- 磁通连续定理：$\begin{align}\int\kern{-8pt}\int_\limits S\kern{-23mu}\bigcirc\boldsymbol{B·}\mathrm d\boldsymbol S\equiv0\end{align}$

- 电场的高斯定理：$\begin{align}\int\kern{-8pt}\int_\limits S\kern{-23mu}\bigcirc\boldsymbol{D·}\mathrm d\boldsymbol S=\sum q_{自由电荷}=\iiint\limits_V\rho\ \mathrm dV\end{align}$

- 电场的环路定理：$\begin{align}\oint_L\boldsymbol{D·}\mathrm d\boldsymbol l=-\iint\limits_S\frac{\part\boldsymbol H}{\part t}\boldsymbol ·\mathrm d\boldsymbol S\end{align}$

### 电磁波

- 波动方程：由麦克斯韦方程组导出，$\begin{cases}\left(\grad^2-\varepsilon\mu\large\frac{\part^2}{\part t^2}\right)\boldsymbol E=0\\\left(\grad^2-\varepsilon\mu\large\frac{\part^2}{\part t^2}\right)\boldsymbol H=0\end{cases}$的解即波动的定义，其中哈密顿算子为矢量三个二阶偏导之和
- 若以电磁波方向为$x$轴正向，可以解得$\begin{cases}\boldsymbol E=\boldsymbol jE_0\cos(\omega t\pm\omega{\large\frac{x}{c}}+\varphi)\\\boldsymbol H=\boldsymbol kH_0\cos(\omega t\pm\omega{\large\frac{x}{c}}+\varphi)\end{cases}$，其中$\begin{align}\pm\omega\frac{x/y/z}{c/u}\end{align}$为传播项，分子定义传播方向，分母定义波速(真空/介质中)，负号为正向传播，正号反之
- 电磁波的性质：
  - 是横波，传播方向由传播项体现，且满足$\boldsymbol{e_E\times e_B}=\boldsymbol{e_u}$
  - 是偏振波，且$\boldsymbol E、\boldsymbol H$同相，即$\cos()$内部完全一致
  - 满足$\sqrt\varepsilon E=\sqrt\mu H$，波速$\begin{align}u=\sqrt{\frac1{\varepsilon\mu}}\end{align}$
- 在$LC$振荡电路中，由$\begin{align}u=\frac qC=\mathscr E=-L\frac{\mathrm dI}{\mathrm dt}=-L\frac{\mathrm d^2q}{\mathrm dt^2}\end{align}$，得出简谐运动方程$\begin{align}\frac{\mathrm d^2q}{\mathrm dt^2}+\frac1{LC}q=0\end{align}$
  - 当二阶常系数线性齐次微分方程中，若“常系数”为只与自身因素有关的正的常数，则称其为简谐运动
  - 振荡角频率$\begin{align}\omega=\sqrt\frac1{LC}\end{align}$
- 电磁辐射能：电能和磁能之和
- 能流密度/辐射强度/坡印廷矢量：$\boldsymbol S=(\omega_c+\omega_m)\boldsymbol u$
