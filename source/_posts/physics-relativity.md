---
title: "大学物理: 相对论"
date: 2024-10-19
categories: [SE Courses, Physics]
tags: []
mathjax: true
---
<!-- placeholder -->
<!-- more -->
## 狭义相对论

### 经典力学伽利略变换

- 在经典力学中，首先认为时间、质量、作用力是绝对的
- 在伽利略变换中，假定惯性系$S'$相对于另一个惯性系$S$以$u$速度匀速直线运动，且设**运动方向为$x$方向**、两惯性系在初始时刻原点在同一位置
- 则有伽利略(逆)变换：$\begin{cases}x=x'+ut'\\y=y'\\z=z'\\t=t'\end{cases}$
- 推广到牛顿运动定律，$\boldsymbol v=(v_x'+u,\ v_y',\ v_z')=\boldsymbol v'+u\boldsymbol i$，$\boldsymbol a=(a_x',a_y',a_z')=\boldsymbol a'$，即**一切力学定律在任何惯性系中形式相同**
- 依据上述变换，在绝对时空观中，空间间隔、时间间隔、时间、质量、一切力学定律都是绝对的，而速度是相对的

### 相对论实验基础

- 首先，依据电磁学理论的广泛应用，证明真空中的光速$\begin{align}c=\frac1{\sqrt{\varepsilon_0\mu_0}}\end{align}$，和伽利略变换中速度的相对性不符

- 迈克尔逊莫雷干涉实验：其本意为了验证绝对静止的惯性系——以太的存在，结果证明以太是不存在的

  即**任何惯性系都是等价的**，不存在任何一个惯性系优于其它惯性系

  - 实验内容：

### 洛伦兹变换

- 无论是经典力学还是狭义相对论，讨论的时空坐标变换都是关于惯性系的

- 两条基本假设：
  - 爱因斯坦相对性原理：**一切物理规律在所有的惯性系中形式相同**
    - 对照伽利略变换中的结论，可知相对性原理是其推广而不是其简单否定

  - 光速不变原理：在**任何惯性系**中，**真空**中的**光速不变**；即光速和光源的运动状态无关
    - 光速不变原理可视为一种物理规律，而经典力学中质量绝对等观念并不是物理规律

    - 另一表述：真空中的光速率是一切物体运动速率的极限

      注意是物体的速率，而非相对速率，相对速率可超过$c$
  
- 一切假设与伽利略变换相同，则有洛伦兹变换：$\begin{cases}x=\frac{\large x'+ut'}{\sqrt{1-\left(\Large\frac uc\right)^2}}\\y=y'\\z=z'\\t=\frac{\large t'+{\Large\frac u{c\normalsize ^2}}x'}{\sqrt{1-\left(\Large\frac uc\right)^2}}\end{cases}$
  - 推导：根据相对论假设，$\begin{cases}x=k'(x'+ut')\\x'=k(x-ut)\\k=k'\end{cases}$，且在初始时刻$x=ct,x'=ct'$
    故$xx'=k^2(x'+ut')(x-ut)=k^2(ct'+ut')(ct-ut)=k^2tt'(c+u)(c-u)=c^2tt'$
  
    解得$\begin{align}k=\sqrt\frac{c^2}{c^2-u^2}=\frac1{\sqrt{1-\left({\Large\frac uc}\right)^2}}\end{align}$

  - 当$u<<c$时，洛伦兹变换将退化成伽利略变换，即日常生活中的低速惯性系
  
- 在相对时空观中，时间与**运动方向上的空间位置**有关，时间、时间间隔、空间间隔、质量等都是相对的，而一切物理规律、光速是绝对的

### 相对论效应

- 以下讨论中，取$\begin{align}\gamma=\frac1{\sqrt{1-\left(\large\frac{u}{c}\right)^2}}\end{align}$

- **相对论中的速度**和**相对速度**的区别：

  - 沿惯性系相对运动方向上，给定$\begin{align}\frac{\mathrm dx}{\mathrm dt}=v_x\end{align}$，则有$\begin{align}v_x'=\frac{\mathrm dx'}{\mathrm dt'}=\frac{\mathrm dx'}{\mathrm dt}\frac{\mathrm dt}{\mathrm dt'}=\gamma^2(v_x-u)\left(1+\frac u{c^2}v_x'\right)\end{align}$

    整理得$\begin{align}v_x'=\frac{\gamma^2(v_x-u)}{1-{\large\frac{\gamma^2 u(v_x-u)}{c^2}}}=\frac{v_x-u}{1-{\large\frac{u}{c^2}}v_x}\end{align}$

    逆变换同理$\begin{align}v_x=\frac{v_x'+u}{1+{\large\frac{u}{c^2}}v_x'}\end{align}$

  - 一：在任意惯性系中测量任意物体的**绝对速率**，不可能超过$c$
    二：而**相对速度**是在**同一惯性系**中观测**不同事件的速度差**，只是简单的**速度叠加**，可能超过$c$

    因速度有相对性，相对速度自然也是相对的，在不同惯性系中观测会有不同

    讨论两个惯性系的相对运动速度$u$时，相当于物体一静止，故$u$等同于物体二的绝对速度，根据第一条，$u$是不可能超过$c$的(虽然它是相对速度)

  - 例如：两相对地面系$S$以速率$u_{1,2}\le c$相向运动的惯性系$S_{1,2}$，讨论$S_{1,2}$原点的运动速度：

    $\begin{align}&在S系中观测,S_1系运动速度为u_1,S_2系运动速度为-u_2,S_2相对S_1速度为u_1+u_2,绝对值可能超过c\\&在S_1系中观测,S_2系运动速度为u_2'=\frac{-u_2-u_1}{1-\frac{(-u_1)(-u_2)}{c^2}}=-\frac{(u_1+u_2)c}{c^2-u_1u_2}c,其绝对值不会超过c\\&仍在S_1系中观测,S_1系运动速度为0,故S_2相对S_1速度就是u_2',其绝对值不会超过c\end{align}$

- 同时性的相对性：在$S'$系中，**沿$u$方向上有相对位移**的不同地**同时发生**事件时，在$S$系中观测，在$S'$中沿$u$方向**后方的事件先发生**

  - 令$\Delta k$为前方事件坐标减后方事件坐标，则有$\begin{align}\begin{cases}\Delta t'=0\\\Delta x'>0\end{cases},\Delta t=\gamma\left(\Delta t'+\frac{u}{c^2}\Delta x'\right)>0\end{align}$，因此前方事件后发生，$\Delta t$即两事件在$S$系中的时间间隔

  - 当两个事件沿$u$方向上没有相对位移时，同时性是绝对的

    因此，不同地同时事件在另一惯性系中可能同时，也可能不同时

  - 时间悖论问题：既然上述推导中，$\Delta t>0而\Delta t'=0$，那么是否存在一种情况使$\Delta t>0而\Delta t'<0$(即颠倒时序)？

    - 因为$u$和任意信号传递速度$v$无法超过光速，因此$\begin{align}\Delta t'=\gamma\left(\Delta t-\frac u{c^2}\Delta x\right)=\gamma\Delta t\left(1-\frac u{c^2}\frac{\Delta x}{\Delta t}\right)=\gamma\Delta t\left(1-\frac{uv}{c^2}\right)\ge0\end{align}$

- 时间间隔的相对性(时间膨胀效应)：

  - 称在$S$中**相对$S$静止(同地)**的事件在该系中测得的时间间隔为**原时**，它是最短的

  - 在任意其它相对$S$系匀速直线运动的惯性系中观测，时间间隔会变长，称为**测时**，因为$\Delta t'=\gamma(\Delta t)$，其中$\gamma>1$

  - 注意原时的判断，考察的**不是$S$系动或不动**，而是该**事件的空间坐标在$S$系中不变**

    例如在**飞船上的时钟**在地面观测会变慢，其中原时为在飞船系中的时间，测时为地面系中测得时间

- 在运动的惯性系中，长度会收缩：

  - 因为时间间隔本身具有相对性，因此测量长度时要求**同时测量**以保证只探讨空间间隔的相对性

  - 若待测量物体的各空间坐标在$S$系中保持静止，则称在该惯性系中测得的长度为**原长**，它是最长的

  - 在任意其它相对$S$系匀速直线运动的惯性系中**同时测量**，**沿运动方向**上的空间间隔会变短，称为**测长**，因为$\begin{align}L_x'=\frac{L_x}{\gamma}=L_x\sqrt{1-\left(\frac{u}{c}\right)^2}\end{align}$

- 质速方程：

  - 在**同一惯性系中**观测时，运动中物体的质量比在静止时大，$\begin{align}m=\frac{m_0}{\sqrt{1-\large\frac{v^2}{c^2}}}\end{align}$，其中$v$为**物体的速度**、$m_0$为静止时的质量

  - 相对论的动量定理：$\begin{align}\boldsymbol{F}=\frac{\mathrm d(m\boldsymbol{v})}{\mathrm dt}=m_t'\boldsymbol{v}+m\boldsymbol{a}\end{align}$，在经典力学中，$m$为常数，第一项为$0$

    质速方程很好地阐明了为什么牛顿运动定律形式仍然相同而速率无法超过$c$

  - 所有物体的能量等于静止能量加上动能，$E=E_k+E_0$，且$\begin{cases}E=mc^2&相对论中的总能量\\E_0=m_0c^2&静能\\E_k=\Delta mc^2&动能\end{cases}$

    - 推导：$\begin{align}&\mathrm dA=\boldsymbol F·\mathrm d\boldsymbol r=\frac{\mathrm d\boldsymbol p}{\mathrm dt}·\mathrm d\boldsymbol r=\boldsymbol v·\mathrm d\boldsymbol p\\&\mathrm d\boldsymbol p=\mathrm d(m\boldsymbol v)=\boldsymbol v\mathrm dm+m\mathrm d\boldsymbol v\\&因此\mathrm dA=\boldsymbol{v·v}\mathrm dm+m\boldsymbol {v·}\mathrm d\boldsymbol v=v^2\mathrm dm+m\left(\sum_{i=x,y,z}v_i\mathrm dv_i\right)\\&=v^2\mathrm dm+m\left(\sum_{i=x,y,z}\mathrm d\left(\frac{v_i^2}{2}\right)\right)=v^2\mathrm dm+mv\mathrm dv\\&由质速方程m=\frac{m_0}{\sqrt{1-\left(\large\frac{v}{c}\right)^2}}\Rightarrow m^2c^2-m^2v^2=m_0^2c^2\\&两边求全微分得2mc^2\mathrm dm-(2mv^2\mathrm dm+2m^2v\mathrm dv)=0\\&代回原式得\mathrm dA=c^2\mathrm dm\\&动能定义E_k=A=\int_1^2\mathrm dA=\int_{m_0}^mc^2\mathrm dm=\Delta mc^2=mc^2-m_0c^2\end{align}$

- 相对论中能量和动量的关系：$\begin{align}E_k=\frac{p^2}{m+m_0}、E^2=p^2c^2+E_0^2\end{align}$
  
  - $\begin{align}&由m=\frac{m_0}{\sqrt{1-\left({\large\frac vc}\right)^2}}\Rightarrow m^2c^4-m^2v^2c^2=m_0^2c^4\\&\Rightarrow E^2=E_0^2+p^2\Rightarrow E_k^2+2E_km_0c^2+m_0^2c^4=m_0^2c^4+p^2c^2\\&\Rightarrow E_k(mc^2-m_0c^2+2m_0c^2)=p^2c^2\\&\Rightarrow E_k=\frac{p^2}{m+m_0}\end{align}$
  
- 光的波粒二象性：光子的速度为$c$，质量为$0$，设光的波长为$\lambda$，频率为$v$，普朗克常量为$h$，则

  - 光速$c=\lambda v$
  - 光子的能量$E=mc^2=pc=hv$
  - 光子的动量$\begin{align}p=\frac{hv}c=\frac{h}{\lambda}\end{align}$
  - 光子的质量$\begin{align}m=\frac E{c^2}=\frac{hv}c\end{align}$
