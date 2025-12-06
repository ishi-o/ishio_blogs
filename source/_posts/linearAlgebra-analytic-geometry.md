---
title: "向量与解析几何入门"
date: 2024-05-20
categories: [SE Courses, Linear Algebra]
tags: [math]
mathjax: true
---
<!-- placeholder -->
<!-- more -->
# 向量与解析几何

## 概述

### 向量的线性表示

在右手空间直角坐标系中，向量可表示为$\vec a=a_1\vec i+a_2\vec j+a_3\vec k$或$\vec a=(a_1,a_2,a_3)$

两点形成的向量$\overrightarrow {PQ}=(q_1-p_1,\ q_2-p_2,\ q_3-p_3)$

### 向量的运算与关系

- 加减法：平行四边形法则，线性表示为$\vec a+\vec b=(a_1+b_1,\ a_2+b_2,\ a_3+b_3)$
- 数乘：线性表示为$\lambda\vec a=(\lambda a_1,\ \lambda a_2,\ \lambda a_3)$，模长$|\lambda\vec a|=|\lambda||\vec a|$
- 求模：线性表示为$|\vec a|=\sqrt{a_1^2+a_2^2+a_3^2}$
- 点乘(数量积)：$\vec a·\vec b=|\vec a||\vec b|·\cos\theta=a_1b_1+a_2b_2+a_3b_3$，点乘满足交换、数乘结合、分配律
- 平行：线性表示为$\begin{align}&\vec a//\vec b\iff \frac{a_1}{b_1}=\frac{a_2}{b_2}=\frac{a_3}{b_3}\end{align}$，即叉乘的模长等于$0$，由行列式知识可知，平行时两向量各坐标值成比例
- 垂直：线性表示为$\vec a\perp\vec b\iff a_1b_1+a_2b_2+a_3b_3=0$，即点乘等于$0$
- 夹角：由点乘，$\begin{align}\cos \theta=\frac{\vec a·\vec b}{|\vec a||\vec b|}\end{align}$。一个向量的三个方向余弦$\cos^2\theta_1+\cos^2\theta_2+\cos^2\theta_3=1$
- 单位向量：$\begin{align}\vec e_{\vec a}=\frac{\vec a}{|\vec a|}\end{align}$，方向余弦$\begin{align}\cos\theta_{x,y,z}=\frac{\vec a_{x,y,z}}{|\vec a_{x,y,z}|}\end{align}$
- 投影：$\vec a$在$\vec b$上的投影为$\begin{align}(\vec a)_{\vec b}=|\vec a|\cos\theta\end{align}$，投影向量为$\begin{align}(\vec a)_{\vec b}\frac{\vec b}{|\vec b|}\end{align}$
- 叉乘(向量积)：方向为右手四指从$\vec a$到$\vec b$握后大拇指的方向，模长为$|\vec a||\vec b|\sin\theta$。叉乘满足分配、数乘结合律，交换后结果方向相反。可以视$\vec a\times\vec b$为三阶行列式$\left|\begin{matrix}\vec i&\vec j&\vec k\\a_1&a_2&a_3\\b_1&b_2&b_3\end{matrix}\right|$，整理后的线性表示为按第一行展开的结果，即$\vec a\times\vec b=(\left|\begin{matrix}a_2&a_3\\b_2&b_3\end{matrix}\right|,\ \left|\begin{matrix}a_1&a_3\\b_1&b_3\end{matrix}\right|,\ \left|\begin{matrix}a_1&a_2\\b_1&b_2\end{matrix}\right|)$，两个向量的叉乘的模长等于**平行四边形的面积**
- 混合积：称$(\vec a,\ \vec b,\ \vec c)$为$(\vec a\times\vec b)·\vec c$或$\vec a·(\vec b\times\vec c)$，叉乘运算必然优先。可表示为三阶行列式$\left|\begin{matrix}a_1&a_2&a_3\\b_1&b_2&b_3\\c_1&c_2&c_3\end{matrix}\right|$，由行列式性质得它等于偶数次交换行/列，或转置后的结果。混合积的几何意义为它的绝对值等于**平行六面体的体积**
- 自乘：$\vec a·\vec a=|\vec a|^2,\ \vec a\times\vec a=\vec0$
- 若$\vec a\sdot\vec b=0$，则两向量垂直；若$\vec a\times\vec b=0$，则两向量平行；若$(\vec a,\ \vec b,\ \vec c)=0$，则三向量共面

### 平面方程

- 点法式：$\begin{align}\frac{x-x_0}{A}=\frac{y-y_0}{B}=\frac{z-z_0}{C}\end{align}$，法向量为$(A,B,C)$，平面必过点$(x_0,y_0,z_0)$
- 一般式：$Ax+By+Cz+D=0$，法向量为$(A,B,C)$
- 截距式：$\begin{align}\frac xA+\frac yB+\frac zC=1\end{align}$

两个相交平面确定一条直线，这条直线的**同轴平面束**为$(A_1x+B_1y+C_1z+D_1)+\lambda(A_2x+B_2y+C_2z+D_2)$

#### 平面与平面的关系

- 两平面相交，则$\begin{align}A_1:B_1:C_1\ne A_2:B_2:C_2\end{align}$
- 两平面重合，则$\begin{align}\frac{A_1}{A_2}=\frac{B_1}{B_2}=\frac{C_1}{C_2}=\frac{D_1}{D_2}\end{align}$
- 两平面平行，则$\begin{align}\frac{A_1}{A_2}=\frac{B_1}{B_2}=\frac{C_1}{C_2}\ne\frac{D_1}{D_2}\end{align}$
- 平行平面间的**距离**：将$A,B,C$化为相同后，距离为$\begin{align}\frac{|D_2-D_1|}{\sqrt{A^2+B^2+C^2}}\end{align}$，由于距离公式含绝对值，需要注意有多个解
- 两平面间夹角：求两法向量间夹角的余弦值的绝对值，得到平面间夹角的余弦值

### 空间直线方程

- 点向式(对称式)：$\begin{align}\frac{x-x_0}A=\frac{y-y_0}B=\frac{z-z_0}C\end{align}$，方向向量为$(A,B,C)$，直线必过点$(x_0,y_0,z_0)$
- 参数式：$\begin{cases}x=x_0+At\\y=y_0+Bt\\z=z_0+Ct\end{cases}$，方向向量为$(A,B,C)$，直线必过点$(x_0,y_0,z_0)$
- 两点式：$\begin{align}\frac{x-x_0}{x_1-x_0}=\frac{y-y_0}{y_1-y_0}=\frac{z-z_0}{z_1-z_0}\end{align}$，方向向量为$\overrightarrow{P_0P_1}$
- 一般式：$\begin{cases}A_1x+B_1y+C_1z+D_1=0\\A_2x+B_2y+C_2z+D_2=0\end{cases}$，即两个平面的交线，方向向量为$(A_1,B_1,C_1)\times(A_2,B_2,C_2)$

#### 空间直线方程间的转换

由于点向式与参数式容易互相转换，在这里只讲述一般式与参数式、两点式的互换：

- 一般式化为参数式：联立两平面，令$x、y、z$任意一者为$t$，即得到参数式方程
- 快捷地，如果只需求一般式直线的方向向量，只需对两平面法向量进行叉乘即可
- 一般式化为两点式：令$x、y、z$任意两者为$0$，求出必过的两点即可
- 参数式化为一般式：三个方程两两联立除去$t$，求出两条关系式即可，点向式同理等式间两两联立整理即可

#### 直线与直线的关系

- 两直线异面：首先判断**方向向量是否平行**，若平行(叉乘为$\vec 0$)则两直线共面，否则应在直线中分别找出两点，得到向量$\overrightarrow{P_1P_2}$，与叉乘结果进行点乘运算，若不为$0$，则为异面；完整行为是求混合积，但有时可以不求$\overrightarrow{P_1P_2}$，并更容易判断共面情况
- 两直线共面且相交：在上述混合积为$0$的情况下，两方向向量不平行
- 两直线平行：两方向向量平行且都不与$\overrightarrow {P_1P_2}$平行
- 两直线重合：三个向量互相平行
- 平行直线间的距离：$\begin{align}d=\frac{|\vec {s_1}\times\overrightarrow {P_1P_2}|}{|\vec {s_1}|}\end{align}$，其中$\vec{s_1}$为某直线的方向向量，即利用了叉乘的正弦值
- 异面直线间的距离：$\begin{align}d=\frac{|(\vec{s_1},\vec{s_2},\overrightarrow{P_1P_2})|}{|\vec{s_1}\times\vec{s_2}|}\end{align}$，即利用混合积求出垂直于两条直线的公垂线

#### 直线与平面的关系

- 直线平行于平面：若直线非一般式，则方向向量与法向量垂直，且直线上点不在平面上，可利用$P_0$判断；若为一般式，可直接观察其中一条平面方程是否与该平面平行
- 直线在平面上：若直线非一般式，则方向向量与法向量垂直，且$P_0$应在平面上；若为一般式，可直接观察其中一条平面方程是否可与该平面化为一致
- 直线与平面相交于一点：若直线非一般式，则方向向量与法向量点乘不为$0$；若为一般式，可直接观察两条平面方程是否都不与该平面平行
- 直线与平面的夹角：利用方向向量与法向量的点乘求出直线与平面夹角的正弦值
- 平行直线离平面的距离：任意找出直线上一点，$\begin{align}d=\frac{|Ax_0+By_0+Cz_0+D|}{\sqrt{A^2+B^2+C^2}}\end{align}$，由于距离公式含绝对值，需要注意有**多个解**

### 二次曲面

一个二元函数就是一个面，若自变量与因变量均为$1$次，则这个面是一个平面；若含有任意一个变量为二次，则称其为二次曲面

类似空间直线，空间曲线由**两个空间曲面相交**而成，**截痕法**通过曲面在三个坐标平面上的空间曲线来研究曲面的形状，例如令$z=0$，可得到平面$Oxy$上的截痕

#### 常用二次曲面

- 球面：$(x-x_0)^2+(y-y_0)^2+(z-z_0)^2=r^2$，其中球心为$(x_0,y_0,z_0)$，球半径为$r$
- 椭球面：若球面中各系数存在不同，则为椭球面，化简为$\begin{align}\frac{(x-x_0)^2}{a^2}+\frac{(y-y_0)^2}{b^2}+\frac{(z-z_0)^2}{c^2}=1\end{align}$
  - 在三个坐标平面上的空间曲线均为椭圆(或圆)
- 柱面：柱面由母线绕**定曲线(即准线)**平行于**定直线$l$**移动而成，一个只有两个变量的曲面一定是柱面，且平行于缺失变量所在坐标轴，以下方程中，$(x_0,y_0)$表示准线的中心
  - 椭圆柱面：准线为椭圆，$\begin{align}\frac{(x-x_0)^2}{a^2}+\frac{(y-y_0)^2}{b^2}=1\end{align}$表示该柱面以一个椭圆为准线、且$l//z$轴；特别地，当$a^2=b^2$时，表示一个圆柱面
  - 双曲柱面：准线为双曲线，$\begin{align}\frac{(x-x_0)^2}{a^2}-\frac{(y-y_0)^2}{b^2}=1\end{align}$，$l//z$轴
  - 抛物柱面：准线为抛物线，$\begin{align}(x-x_0)^2-2py=0\end{align}$，$l//z$轴
- 二次锥面：$\begin{align}\frac{(x-x_0)^2}{a^2}+\frac{(y-y_0)^2}{b^2}-\frac{(z-z_0)^2}{c^2}=0\end{align}$表示在$x=x_0$与$y=y_0$上的截痕均为**一对直线**，而在$z=z_0$平面上截痕为**一点**，该点为$(x_0,y_0,z_0)$(两$\ge0$的数相加$=0$只有一个解)
- 单叶双曲面：$\begin{align}\frac{(x-x_0)^2}{a^2}+\frac{(y-y_0)^2}{b^2}-\frac{(z-z_0)^2}{c^2}=1\end{align}$表示在$x=x_0$与$y=y_0$上的截痕均为**双曲线**，而在$z=z_0$上截痕为**椭圆**，其中心为$(x_0,y_0,z_0)$
- 双叶双曲面：$\begin{align}\frac{(x-x_0)^2}{a^2}-\frac{(y-y_0)^2}{b^2}-\frac{(z-z_0)^2}{c^2}=1\end{align}$表示在$y=y_0$和$z=z_0$上的截痕为**双曲线**，$\begin{align}\frac{(x_1-x_0)^2}{a^2}-1=1\end{align}$时，在$x=x_1$上的截痕为**椭圆**；若$x_1>x_0$，因在$x_0<x<x_1$间**没有定义**，故称双叶
- 椭圆抛物面：$\begin{align}\pm(z-z_0)=\frac{(x-x_0)^2}{a^2}+\frac{(y-y_0)^2}{b^2}\end{align}$表示在$x=x_0$与$y=y_0$上截痕为**抛物线**，而在$z=z_0$上截痕为**椭圆**
- 双曲抛物面(马鞍面)：$\begin{align}z-z_0=\frac{(x-x_0)^2}{a^2}-\frac{(y-y_0)^2}{b^2}\end{align}$表示在$x=x_0$与$y=y_0$上截痕为**抛物线**，而在$z=z_1\ne z_0$上截痕为**双曲线**，在$z=z_0$上为**一对直线**

类似的，许多常见曲面应用截痕法记忆

由截痕法所求的在某个平面上的截痕方程，就是曲面在这个平面上的交线

而求曲线在坐标平面上的投影曲线，就是**消去该方向上的坐标变量**而合并成一条曲面方程后，和**该平面方程**组合而成的曲线方程

#### 一般形化简为标准形

假设一个曲面方程的一般形为$a_{11}x^2+a_{22}y^2+a_{33}z^2+2a_{12}xy+2a_{13}xz+2a_{23}yz+b_1x+b_2y+b_3z+c=0$，要将它化为$\lambda_1(x'-x_0)^2+\lambda_2(y'-y_0)^2+\lambda_3(z'-z_0)^2+c=0$的形式，需要消除$xy,xz,yz$项，则需：

- 将它的系数化为**对称矩阵**：$A=\left[\begin{matrix}a_{11}&a_{12}&a_{13}\\a_{12}&a_{22}&a_{23}\\a_{13}&a_{23}&a_{33}\end{matrix}\right]$，通过$\lambda E-A=0$，求出它的特征值$\lambda_{1,2,3}$
- 若$b_1=b_2=b_3=0$，则不需要作平移变换，且$x_0=y_0=z_0=0$
- 否则，需要平移变换，即找出正交阵$X$，使$X^TAX=\rm diag(\lambda_1,\lambda_2,\lambda_3)$
- 则$\left(\begin{matrix}b_1&b_2&b_3\end{matrix}\right)X=\left(\begin{matrix}d_1&d_2&d_3\end{matrix}\right)$

#### 求解未知曲面方程

在实际求解方程时，先设所求曲面上一点$M(x,y,z)$，然后根据条件求其一般形$F(x,y,z)=0$，根据需要进行正交变换转换为标准方程，再判断其形状

能利用的条件有许多：

- $M$距球面(球心为$O$)距离：$d=\Big||\overrightarrow{MO}|-R\Big|$
- 给出双参数方程，则联立消去参数，可转化为一般形
- 给出旋转轴和动曲线，先找到这条动曲线上任意点的表示形式，如$M_0(x_0,y_0,z_0)$，然后通过与旋转轴的关系(即在某一方向上，动曲线的值$k_0$与待求曲面的值$k$相同、以及$M_0$与轴的距离不变)，求出$M_0$与$M$的关系，再由$M_0$内部的关系得出$M$内部的关系
  - 具体来说，假设要求动直线$L$绕$z$轴旋转后的曲面方程，其中$L$方向向量为$\vec s$；$(1)$则$M_0$的三个坐标值与$\vec s$相关；$(2)$由于绕$z$轴，则在$z$轴方向上，$z=z_0$；$(3)$在同一个$z_0$上，有$r^2=x^2+y^2=x_0^2+y_0^2$；由$(1),(2),(3)$得到的式子联立，可求出未知曲面方程

## 练习

$\begin{align}&1.求过点M_1(-1,0,4),且与平面\pi_1:3x-4y+z-10=0平行,又与直线L_1:\frac{x+1}3=\frac{y-3}1=\frac z2\\&相交的直线L的方程.\\&解:\\&设L方向向量为(A,B,C),由\pi_1法向量为(3,-4,1),得3A-4B+C=0\\&又L_1的方向向量为(3,1,2),其中一点为M_2(-1,3,0),由它们相交得(\vec {s_{L}},\vec{s_{L_1}},\overrightarrow{M_1M_2})=0\\&即10A-12B-9C=0,令C=4,得A=48,B=37,即L:\frac{x+1}{48}=\frac{y}{37}=\frac{x-4}4\end{align}$

---

$\begin{align}&2.设两直线L_1:\begin{cases}x-3y+z=0\\2x-4y+z+1=0\end{cases};L_2:x=\frac{y+1}3=\frac{z-2}4\\&(1).证明异面.(2).求距离.(3).求过L_1且平行于L_2的平面方程\\&解:\\&(1).令z=t,化L_1为\begin{cases}x=\frac32+\frac t2\\y=\frac12+\frac t2\\z=t\end{cases},\vec {s_1}=(1,1,2),过点M_1(\frac32,\frac12,0)\\&则(\vec {s_1},\vec{s_2},\overrightarrow{M_1M_2})=2\ne 0,故异面.\\&(2).d=\Bigg|\frac{(\vec {s_1},\vec{s_2},\overrightarrow{M_1M_2})}{|\vec{s_1}\times\vec{s_2}|}\Bigg|=\frac1{\sqrt3}\\&(3).由两方向向量叉乘,且平面过M_1,得方程为x+y-z+2=0\end{align}$

---

$\begin{align}&3.已知点A,B的坐标分别为(1,0,0),(0,1,1).线段AB绕z轴旋转一周所成的曲面为S.\\&求S及平面z=0,z=1围成的立体体积.\\&解:\\&线段AB所在直线的方向向量为(1,-1,-1),设其上任意一点为(x_0,y_0,z_0)\\&则L_{AB}:x_0-1=-y_0=-z_0&(1).\\&设所求曲面任意一点为(x,y,z),绕z轴旋转,故z=z_0&(2).\\&由r^2=x^2+y^2=x_0^2+y_0^2与上式联立得,x^2+y^2=(1-z)^2+z^2\\&即x^2+y^2-2z^2+2z-1=0.\\&故在平行于Oxy平面上的截面面积为S=\pi(2z^2-2z+1),\\&V=\int_0^1Sdz=\frac23\pi\end{align}$
