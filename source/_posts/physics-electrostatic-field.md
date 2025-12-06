---
title: "大学物理: 静电场"
date: 2024-10-06
categories: [SE Courses, Physics]
tags: []
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 静电场

## 基本定律及常量

- 电荷的性质：
  - 量子性：基本电荷$e=1.602\times10^{-19}\rm C$，所有基本粒子所带电荷均为$e$的整数倍
  - 电荷守恒定律：在孤立系统内发生的任何过程中，总电荷数(所有电荷代数和)不变
  - 正负性、相对论不变性

- 库仑定律：电荷间的相互作用力为$\begin{align}\boldsymbol{F}=\frac1{4\pi\varepsilon_0}\frac{q_1q_2}{r^3}\boldsymbol{r}=\frac{q_1q_2}{4\pi\varepsilon_0r^2}\boldsymbol{r}_0\end{align}$
  - 其中，真空介电常量$\varepsilon_0=8.85\times10^{-12}\rm C^2·N^{-1}·m^{-2}$
  - 没有合并进去的$4\pi$是方便工程中计算球体而引入的

## 场论

- 场是一种中介物质，物质间的相互作用要靠场来传递

- 电磁场是量子化的，具有波粒二象性，对电场来说：
  - 相对于观察者静止的电荷产生静电场
  - 电场是有能量的
  
- 电场强度(场强)$\boldsymbol{E}$：
  - 反映了电场的固有属性，即空间中**某一点的场强**与在**该点上的电荷无关**，计算时需要剔除场点产生的场强

  - 由于电场力满足叠加原理，故电场强度也满足叠加原理

  - 要求某一点的场强，就是求空间中**除该点外**，所有电荷产生的场强的矢量和

    场强第一种求解方式(场强叠加原理)：$\begin{align}\boldsymbol{E}=\int\mathrm d\boldsymbol{E}=\int\frac{\mathrm dq}{4\pi\varepsilon_0r^3}\boldsymbol{r}=\int_\Omega\frac{\rho\ \boldsymbol{r}}{4\pi\varepsilon_0r^3}\mathrm d\Omega\end{align}$
    具体来说，**大小通过积分**来求，而**方向通过划分坐标分量**，各自求解并组合

- 电通量$\varPhi$和静电场中的高斯定理：
  - 指垂直通过某面的电场强度的代数和(电场线的条数)，和电场强度、穿过的面积成正比
  - $\begin{align}\varPhi=\iint_S\boldsymbol{E}\ \mathrm d\boldsymbol{S}=\iiint_V\div{\boldsymbol E}\end{align}$(第二型曲面积分和高斯公式)，散度$\begin{align}\div{\boldsymbol E}=\frac{\lambda}{\varepsilon_0}\end{align}$
  - 高斯定理：真空中，通过封闭曲面的电通量等于$\begin{align}\frac{\sum q}{\varepsilon_0}\end{align}$，其中$q$包括所有自由、极化电荷
  - 高斯定理对所有电场都成立，但只能用于求解高度对称场的场强，例如(场强第二种求解方式)：
    - 球对称图形的外部电场：$\begin{align}\boldsymbol E=\frac{q}{4\pi\varepsilon_0r^2}{\boldsymbol r_0}\end{align}$
    - 轴对称图形的外部电场：$\begin{align}\boldsymbol E=\frac{\rho}{2\pi\varepsilon_0 r}\boldsymbol r_0\end{align}$，$\rho$为电荷线密度
    - 面对称的匀强场：$\begin{align}\boldsymbol E=\pm\frac{\lambda}{2\varepsilon_0}\boldsymbol n_0\end{align}$，这里的面**无厚度**
  - 高斯定理揭示出的**电场线性质**：
    - 有头有尾，始于正电荷、终于负电荷
    - 任意两电场线不相交
    - 任意电场线不闭合
    - 任何正电荷(包括极化电荷)都能发出电场线，即电介质中需要画两种电场线
  - 通过高斯定理求解步骤：
    - 阐明对称性，选取合适高斯面以至于能快速去除积分号
    - 求出电通量表达式和面内净电荷
  
- 电场力做功：
  - 电荷在电场中从$a$到$b$，$\begin{align}A=\int_a^b\boldsymbol{F·}\mathrm d\boldsymbol{r}\end{align}$
  - $\begin{align}A=q\oint_L\boldsymbol{E·}\mathrm d\boldsymbol{r}=q\iint_S\boldsymbol{\grad\times E·}\mathrm d\boldsymbol{S}\end{align}$(斯托克斯公式)，其中$\boldsymbol{\grad\times E}$即旋度
  - 电场力是保守力，做功与路径无关，沿闭合路径做功为$0$
  
- 场强环路定理：
  - 静电场是**无旋场**，是保守力场
  - 电势能：定义电荷在$a$处的电势能为$\begin{align}W_{a}=q\int_a^{0势}\boldsymbol{E·}\mathrm d\boldsymbol{r}\end{align}$
  - 电势：$a$处电势为$\begin{align}U_a=\int_a^{0势}\boldsymbol{E·}\mathrm d\boldsymbol{r}\end{align}$，积分上下限通常为点到对称中心的距离
    - 电势是**相对量**
    - 电势差：定义$a$和$b$点间的电势差为$\begin{align}U_a-U_b=\int_a^b\boldsymbol{E·}\mathrm d\boldsymbol{r}\end{align}$
  - 微分形式：$\grad U=-\boldsymbol E$，电场方向指向电势减小的方向
    - 场强均匀的区域，电势不一定为定值
    - 电势为定值的**区域**(不是等势面)，场强一定为零
    - 通常作为定性考点
  - 常见对称电场的电势(以无穷远为零势点)：
    - 球对称图形的外部：$\begin{align}\frac{q}{4\pi\varepsilon r}\end{align}$
    - 轴对称图形的外部：$\begin{align}\frac{\lambda}{2\pi\varepsilon}\end{align}$
    - 无厚度面：只能求两面/两薄面的电势差
  - 电势两种计算方式：定义法(对称场)、叠加法
  
- 导体：

  - 导体静电平衡条件：内部场强为零，表面附近场强处处与面元垂直
    - 若有空腔且空腔内有电荷$Q$，则内表面感应出$-Q$，外表面表现为$q+Q$，此时空腔内有电场

  - 非对称导体表面带电情况只能通过实验测量，电荷面密度与表面曲率成正比

  - 导体是等势体

- 电介质：

  - 两种分子：正负电荷中心重合/不重合$\rightarrow$非极性分子/极性分子

  - 极化方式：位移极化/取向极化，极化电场只影响电介质内部电场

  - 电极化率：$\chi_e$

  - 电极化强度：$\boldsymbol P=\alpha\boldsymbol E$、$\alpha=\varepsilon_0\chi_e$，$P_n=\sigma_{极化电荷}$

  - 相对介电常量：$\varepsilon_r=1+\chi_e$，无量纲；一般大于$1$，因此往电容两极板间加入电介质将增大电容

  - 绝对介电常量：$\varepsilon=\varepsilon_0\varepsilon_r$，有量纲

  - 电位移矢量：$\boldsymbol D=\varepsilon\boldsymbol E$

  - 电介质的高斯定理：$\begin{align}\oint_S\boldsymbol{D·}\mathrm d\boldsymbol S=\sum q_{自由电荷}\end{align}$，因此电位移线由正自由电荷发出
    - 推导：$\begin{align}&\oint_S\boldsymbol{P·}\mathrm d\boldsymbol S=-\sum q_{极化电荷}\\&\oint_S\boldsymbol{E·}\mathrm d\boldsymbol S=\frac1{\varepsilon_0}\sum (q_{极化电荷}+q_{自由电荷})\\&\boldsymbol P=\frac{\varepsilon_r-1}{\varepsilon_r}\boldsymbol D=\varepsilon_0(\varepsilon_r-1)\boldsymbol E\end{align}$

    - 分别求极化电荷或自由电荷产生的场强时，不能通过电介质的高斯定理求解，该定理用于快速求解电介质中的合场强

      换句话说，不同种类电荷产生的场强和$\varepsilon_r$无关

      事实上，$\begin{align}E_0=\frac{\sigma}{\varepsilon_0}、E'=\frac{\sigma'}{\varepsilon_0}\end{align}$

- 电容：

  - $\begin{align}C=\frac{Q}{\Delta U}\end{align}$，两极相距单位电势差时正极板所带的电荷
  - 常见电容器的电容：
    - 孤立导体球：$\begin{align}C=4\pi\varepsilon R\end{align}$
    - 平行板电容器：$\begin{align}C=\frac{\varepsilon S}{d}\end{align}$
    - 球形电容器(两同心球面)：$\begin{align}C=\frac{4\pi\varepsilon R_1R_2}{R_2-R_1}\end{align}$($R_2>R_1$)
    - 圆筒形电容器(两同轴圆柱面)：$\begin{align}C=\frac{2\pi\varepsilon l}{\ln{\frac{R_2}{R_1}}}\end{align}$($R_2>R_1$)
  - 电容并联：$C=\sum C_i$，串联：$C^{-1}=\sum C_i^{-1}$
  - 电容率：通常指两极板间介质的相对介电常量

- 电场能量：

  - 电场能量的性质：
    - 只要该点存在电场，就有电场能量
    - 称某电场的能量为范围内所有点电场能量的总和

  - 电容的能量：$\begin{align}W_e=\frac{CU^2}{2}\end{align}$
  - 能量密度(一般电场某点的能量)：$\begin{align}w_e=\frac{\varepsilon E^2}{2}\end{align}$
  - 一般电场存储的能量：$\begin{align}W_e=\int_Vw_e\mathrm dV\end{align}$

## 疑问知识点

### 平行板电容器补充

易知平行板电容器的电容为$\begin{align}C=\frac{\varepsilon S}{d}\end{align}$

- 一：往平行板电容器中平行于两板插入电介质和垂直于两板插入电介质的区别

  答：前者为两电容器串联，后者为两电容器并联

- 二：插入介质后，平行板电容器间的场强在有/无电源时的区别

  答：无电源时，电势会改变，是因为插入介质后场强改变了，$\begin{align}E=\frac{E_0}{\varepsilon_r}\end{align}$

  有电源时，电势不改变，则场强不改变(无论中间介质如何改变)，原理是由电源输送的自由电荷场强和插入介质中产生的极化电荷场强抵消

- 三：插入介质后，平行板电容器的能量在无电源时的表现

  答：能量会减少，根据$\begin{align}W_e=\frac{Q^2}{2C}\end{align}$可知

  原理是极化电荷削弱了电场，使其能量减少，但电容会增加

### 电极化强度和电位移矢量的含义

- 电极化强度是单位体积内的电偶极矩的矢量和，且$P_n=\sigma_{极化电荷}$；因为未极化电介质的电偶极矩矢量和为零，极化后不为零，故以此作为区分
- 电位移矢量是辅助矢量，是经数学简化后的叠加场
