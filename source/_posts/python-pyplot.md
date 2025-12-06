---
title: "Python: 使用pyplot可视化"
date: 2024-01-11
categories: [Programming, Python, Data Science Utils]
tags: [pyplot]
---
<!-- placeholder -->
<!-- more -->
# `matplotlib`数据可视化

## 基本用法

`matplotlib`用于将数据呈现为图，它并非`python`自带的库，需要`pip install`

生成一条直线的步骤如下，它的原理是将提供的数组中**各点连接**，所以提供的**数组越大，线越光滑**



```python
import matplotlib.pyplot as plt  # 常用的是pyplot模块
import numpy as np

# x,y可以是数字或字符串，为字符串时默认等距。
x = np.linspace(1,10,500) # linspace:在1-10区间内生成500个等距点,返回数组
y = 2*x + 1     # 直线方程

plt.figure()    # 创建新图,方便开多个figure
# plt.figure(num=4)     可以传数字参数num=?,空则默认按顺序命名
# plt.figure(figsize=(5,6))   自定义宽,高(但实际上可以用鼠标拉长)
plt.title('a line')   # 图的名称
plt.plot(x,y)    # 生成图,x轴数组为x,y轴数组为y
plt.show()     # 将该图展现出来
```

<img src="./python-pyplot/plt_figure.png" style="zoom: 33%;" />

如图，底栏所示功能编号1-7

1. 返回首页(即一开始看到的页面)

​     2-3. 后退/前进

4. 移动
5. 放大窗口
6. 调整图的边界参数
7. 保存当前窗口

## 自定义线

如果想在一个`figure`内画多条线，`plt.plot`还可以接受其他参数用于**区分两条线**



```python
y = exp(x)
plt.figure()
plt.plot(x,y)
plt.plot(y,x,color='red',linestyle='--')# 默认蓝色,这里改为红色,虚线
# linestyle可简写为ls,color可简写为c
plt.plot(x,y,linestyle=(0,(1,2,3)))  # 元组定义线型
```

<img src="./python-pyplot/plt_expFigure.png" style="zoom:33%;" />

关于元组定义线型，`(num,(n1,n2,n3,...))`表示以后面元组的值**作为单位**画线
例如`(0,(1,2,3))`指的是`(1磅实线,2磅空白,3磅实线)`，因为是**奇数**个，所以下一组变为`(1磅空白,2磅实线,3磅空白)`，所以一般来说，常用的是偶数个元素的元组
关于`num`，它用于在**显示标签**时**偏移**的长度，例如：

<img src="./python-pyplot/plt_linestyle.png" alt="plt_linestyle" style="zoom:50%;" />

`color`参数可用简写如`'r'(红色),'g'(绿色)`，多条曲线不指定颜色时会**自动选择不同颜色**，其它颜色如下：

- `'m'`：洋红色
- `'w'`：白色
- `'k'`：黑色
- `'y'`：黄色
- `'g'`：绿色
- `'#000000'`：自定义`RGB`颜色字符串->[RGB颜色查询对照表](https://www.sojson.com/rgb.html)



其它常用`linestyle`如下：

- `'-'/'solid'`：实线，默认
- `'--'/'doshed'`：虚线
- `':'/'dotted'`：点线
- `'-.'/dashdot`：点划线

其它参数：

- `label='name'`：设置这条线的标签，可用`plt.legend()`显示
- `linewidth=num`：线的宽度，默认为1，可简写为`lw`
- `alpha`：线的透明度，范围`0~1`
- `'bo'`：改为用蓝色**实心圈**绘制而非连线，常用于绘制单独的点，通过**改变首字母来改变颜色**，以下标记也可以在前面添加表示颜色的字母，不再赘述
- `'.'/','`：点/像素点(很小的点)绘制
- `'<'/'>'/'^'/'v'`：各种三角![m03](https://www.runoob.com/images/m03.png)
- `'1'/'2'/'3'/'4'`：各种三叉![m07](https://www.runoob.com/images/m07.png)
- `'*'`：星号绘制
- `'+'`：**+**号绘制，上面标记都**只能跟在数组后且不能重复**

如果只提供一个数组，则x会**默认为`[0,1,...,N-1]`**，`N`为提供的数组的元素个数

## 绘图标记

除了改变线型，`pyplot`还可以用`marker`更改**点的样式**

```python
plt.plot(x,y,marker='o') # 每个点用实心圆代替(但是线型还是直线)
plt.plot(x,y,marker=r'$\alpha$',markersize=10,label='name')  # markersize可简写为ms
plt.plot(x,y,marker='*',mec='r',mfc='k') # mec设置点边框的颜色,mfc设置点内部颜色
# mec:markeredgecolor, mfc:markerfacecolor
```

除了线型所支持的标记外，点的标记还**支持`Latex`语法**

可以用`fmt='[mk][line][color]'`参数快速设置点、线、色

```python
plt.plot(x,y,fmt='o:k')  # 实心圆(点),虚线(线),黑色(色)
```



## 自定义坐标轴

### 设置坐标轴的值

```python
plt.xlim((0,1))  # 限制x轴的范围
plt.ylim((1,2))  # 限制y轴的范围(也可以用[]括起)
plt.xlabel('label') # 设定x轴的标签,即说明x轴是什么,同理可用plt.ylabel()设置y轴的标签
# loc属性:设置标签的位置,有right/left/center 默认center ---> title(),legend()都有该属性
# ylabel的loc属性有top/bottom/center

plt.xticks(np.linspace(0,1,5)) # 更改x轴的单位,默认每0.5为一个节点,这里设置为将0-1分割为5点
# 后面数组对应前面数组的值,可以改成自己需要的名称
plt.xticks([0,1,5],['none','middle','pretty good'])
```

<img src="./python-pyplot/plt_xticks.png" style="zoom:33%;" />

此外，后面数组还支持`Latex`语法，因为可能要用到转义字符等，所以这种情况需要在字符串前加上`'r'/'R'`，将它视为原始字符串`(raw string)`来解析，例如`plt.xticks([0,1,5],[r'$\theta$','$x$',r'$\Theta$'])`的效果为：

<img src="./python-pyplot/plt_Latex.png" style="zoom:25%;" />



### 移动坐标轴

需要用到`gca()`方法，全称`get current axis`，一般将返回的对象称为`ax`，以下是隐藏右轴及上轴的代码

```python
ax = plt.gca() # 获取前面这整张图的坐标轴,有上下左右四条轴
for i in ['right','top']:
 ax.spines[i].set_color('none') # 可以通过spines[]更改对应轴的属性
```

四条轴分别为：`top,bottom,left,right`

需要移动坐标轴，则要用到`set_position()`方法，它接受元组参数，效果如下

```python
ax.spines['left'].set_position(('data',0)) # 将y轴移动到x=0的位置
```

<img src="./python-pyplot/plt_spines.png" style="zoom:33%;" />

一般来说，将坐标轴交点设置在`(0,0)`处的方法如下

```python
for i in ['left','bottom']:
 ax.spines[i].set_position(('data',0))
```

## 网格线

可用`grid`方法设置网格线：

```python
plt.grid(None,which='major',axis='both',...) # 其它参数和设置线型时类似
```

- `None`：是否显示网格线，如果后面接了其它参数，则默认为1，可设置为0
- `which`：显示主要/次要网格线，默认只显示主要，可改为`minor`或`both`
- `axis`：显示某方向的网格线，默认为`both`，可改为`x`或`y`



## 绘制子图

`pyplot`提供`subplot()`和`subplots()`方法，它能在一张画布中绘制多张图

### `subplot`

`subplot`有三个必选参数，从左到右分别是**行、列、标号**，可以加可选参数例如共享轴、标签、投影类型等，返回**一个对象**，在`subplots`里将详细讲述对象作图
或者，把逗号删去，使用一个三位数的参数，例如`plt.subplot(111)`

子图里可以改变图的投影类型，如：

```python
plt.figure()        # 创建画布,或:
fig = plt.figure()       # 将画布赋给fig
ax = plt.subplot(111,projection='polar') # 极坐标投影,或:
ax = plt.subplot(111,polar=1)    # 等效
```

其它投影类型还有：

- `aitoff`
- `hammer`
- `lambert`
- `mollweide`

这些投影类型需借助其它库，超出了`pyplot`的范围，且都用作绘制世界地图，故不细讲

注意到这个`fig`对象，它让切分画布变得便利：

```python
ax1 = fig.add_subplot(111)   # 将左上角的区域切分出来
```

### `subplots`

`subplots`用于一次性创建多个子图，并返回数组对象方便操作，现在介绍不同于上述用**函数作图**的**对象作图**方法，这种方法更**常用**，便于进行精细化操作。当然，如果只需要简单的图形，用函数作图的方式也无妨

它利用`subplots`返回画布和图对象，通过**改变对象的属性**，进行对特定画布中特定部分的修改：

```python
fig, ax = plt.subplots()  # 返回画布,图数组(这里没给参数,默认生成一张图)
ax.plot(x,y)
```

这里的`ax`指的是`axes`，即画布中的一部分。这里因为只生成一张图，所以只有一个`axes`



这种方法就不需要`plt.figure()`了，因为`subplots()`返回时就已创建一张画布了

此外，还可以设置共享轴与投影形式：

```python
fig,(ax1,ax2) = plt.subplots(1,2,sharex=1,sharey=1) # 1即所有子图共享,0即所有子图独立
# 改变图形与subplot不同,需要将参数转换为字典传进去
fig,ax3 = plt.subplots(1,1,subplot_kw=dict(projection='polar'))
fig,ax3 = plt.subplots(1,1,subplot_kw=dict(polar=1))
```

还可设置为`col`(每列子图共享，列与列间独立)，`row`(每行子图共享，行与行间独立)

其它设置与函数作图类似但部分有所不同：

```python
fig,axs = plt.subplots(2,2)      # 两行两列,返回数组,用坐标访问
axs[0,0].set_title('titleName')     # 标题
axs[0,0].plot(x,y,lw=1,c='r',marker='*',...) # plot
axs[0,0].set_xticks(array)      # 轴的单位
axs[0,0].set_xlim(-1,1)       # 轴的范围
axs[0,0].spines['right'].set_color('none')  # 隐藏轴
axs[0,0].axis('off')       # 隐藏所有轴,可以用xaxis/yaxis
axs[0,0]1.set_xlabel('labelName')    # 给x轴加标注
```

## 其它图形

接下来简略介绍其它图形的绘制

### 散点图`(scatter)`

除了`x、y`外，其它参数可以有：`marker(改变点的样式),s(点的大小),c(点的颜色),lw`等已经讲过的，和：

- `cmap(colormap)`：调色盘，只有当`c`是一个浮点数数组时启用，会将这些数值对应到调色盘内的某个颜色。默认为`viridis`，可以通过以下命令查看颜色条

  ```python
  # c=y,则y的数值大小可以清晰地映射到颜色条上
  plt.scatter(x,y,c=y,cmap='viridis')
  plt.colorbar()           # 函数作图
  ax.figure.colorbar(ax.scatter(x,y,c=y,cmap='viridis')) # 对象作图
  ```

  其它可选调色盘：


  
  <img src="./python-pyplot/plt_colorbar.png" style="zoom: 50%;" />
  
- `vmin,vmax`：调节颜色条的范围，大于`vmax`的数值将被强制映射到`vmax`上，`vmin`同理

- `norm`：也是用于调节范围的，优先级高于上一条，参数没搞懂，有如下：

  - `linear`
  - `log`
  - `symlog`
  - `asinh`
  - `logit`
  - `function`
  - `functionlog`

### 柱形图`(bar/barh)`

`bar`即纵向柱形图，`barh`即横向柱形图。除`x,y`外的其它参数有：

- `width`：浮点数数组，表示柱子宽度
- `height`：浮点数数组，表示横向柱子的宽度，它是`barh()`的参数
- `bottom`：底线值，即`y=?`，默认0
- `color`：颜色，不能缩写为`c`，可接受颜色数组
- `align`：对齐方式，默认`center`，即值的中间，可改为`edge`(左对齐)，如要实现右对齐，可以传递负数宽度值



### 饼图`(pie)`

饼图的参数有点不同，它只需要一个`x`的参数，且它必须是非负数、非字符串的值。其它参数如下：

- `explode`：一个数组，表示各个扇形的间隔。一般来说，没有显式说明的第二个参数默认为它
- `labels`：一个列表，表示各个扇形的标签
- `colors`：自定义颜色
- `autopct`：数值显示形式，与`C`语言类似，`%d`表示整数，`%0.1f`表示一位小数，在后面加上`%%`表示百分比形式显示
- `labeldistance`：标签的位置，默认1.1，即饼图外侧
- `pctdistance`：与上一条类似，表示数值的位置
- `shadow`：饼图是否有阴影，默认为0
- `radius`：饼图半径

其它参数实用性不强，这里不再说明

### 直方图`(hist)`

直方图用于统计数的个数，也只需要提供`x`。其它参数如下：

- `bins`：箱数，默认为10。`hist`将直方图分为`bins`个等距区间(即箱)，每个箱子的柱高表示这个区间内数的个数
- `density`：是否以频率显示。默认为0，即默认为统计个数而非频率
- `weights`：数组，表示每个箱子的权重，箱子中每有一个数就会乘以这个权重值
- `bottom`：与柱形图功能相同
- `histtype`：直方图样式，可选参数有(一些没有区别的已省去)：
  - `bar`：默认值
  - `step`：空心柱

## 其它装饰

### 复杂子图

在讲述`subplots`时，只是提到了一个简便的画图方法，如果需要更加精细地切分画布，这种规规整整的方式就不行了。于是我们需要用到网格：



```python
plt.figure()
ax1 = plt.subplot2grid((2,2),(0,0)) # 将画布切分为2行2列,并把(0,0)网格返回给ax对象
ax2 = plt.subplot2grid((2,2),(1,0),colspan=2) # ax2占满第二行
ax3 = plt.subplot2grid((2,2),(1,1),rowspan=2) # ax3占满第二列
```

也可以用`matplotlib.gridspec`模块实现：

```python
import matplotlib.gridspec as gridspec
plt.figure()
gs = gridspec.GridSpec(3,3)  # 将画布切分成3行3列
ax1 = plt.subplot(gs[0,0])
ax2 = plt.subplot(gs[1:,1])
ax3 = plt.subplot(gs[-1,-1])
```

网格坐标写法有如下：

1. `start:end`：从`start`开始占到`end`，例如`[1:,1]`为占了第一列的第二行到最后一行
2. `-num`：倒数第`num`行

这还是有点方正了，如果想随心所欲改变图的长宽，可以用`update`方法或`plt.subplots_adjust`，它们的参数是类似的：

```python
gs1 = gridspec.GridSpec(3,3)
gs1.update(left=0,right=0.5) # 限定gs1网格只占画布的左边
```

其它参数：

- `wspace`：横向子图的间距，默认`0.2`
- `hspace`：纵向子图的间距，默认`0.2`
- `top,bottom`：上边界和下边界

### 中文显示

需要在开头加上：

```python
plt.rcParams["font.family"]="SimHei" # 将字体名称设置为"SimHei"(黑体)
```

关于其它字体的设置：[链接](https://blog.csdn.net/qq_35240689/article/details/130924160)

但是在正则表达式中似乎无效



### `tick_params()`

通过这个方法可以让刻度线消失：

```python
ax.tick_params(length=0) # 直接设置刻度线的长度,或
ax.tick_params(bottom=0) # 不显示底轴的刻度线
```

## 实例

`code`：

```python
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.gridspec as gridspec
def main():
    x = np.linspace(-5,5,30)
    xSin = np.linspace(0, 2*np.pi,30)
    theta = np.linspace(0, 2*np.pi,30)
    yExp = np.exp(x)
    ySin = np.sin(xSin)
    rHeart = np.sin(theta)
    plt.figure()
    # 左网格
    gsL = gridspec.GridSpec(2,2)
    gsL.update(left=0.05,right=0.4,wspace=0.4)
    axSin = plt.subplot(gsL[0,0])
    axPol = plt.subplot(gsL[0,1], polar=1)  # 心形线极坐标映射
    axExp = plt.subplot(gsL[1,:])           # 占满第二行

    axSin.plot(xSin,ySin)
    axSin.set_xticks([0,np.pi,2*np.pi],[0,r'$\pi$',r'$2\pi$'])
    axSin.set_title('y=sinx',backgroundcolor='skyblue')

    axPol.plot(theta, rHeart)
    axPol.set_title(r'$\rm r=sin\theta$',backgroundcolor='skyblue')

    axExp.plot(x,yExp,label=r'$y=e^x$')
    axExp.plot(yExp,x,ls=(0,(0,1,2,1)),label='y=lnx')
    axExp.plot([0,1],[1,0],'ko')
    axExp.set_xlim(-0.5,2)
    axExp.set_ylim(-5,10)
    axExp.set_xticks(np.linspace(0,2,3))
    axExp.set_yticks(np.linspace(-5,10,4))
    axExp.tick_params(length=0)
    axExp.set_title(r'$\rm y=e^x\ &\ y=\ln x$',backgroundcolor='pink')
    plt.legend(loc='upper right')
    for i in ['right','top']:
        axSin.spines[i].set_color('none')
        axExp.spines[i].set_color('none')
    for i in ['left','bottom']:
        axSin.spines[i].set_position(('data',0))
        axExp.spines[i].set_position(('data',0))
    # 右网格
    gsR = gridspec.GridSpec(2,2,left=0.5,right=0.9,hspace=0.3)
    axDens = plt.subplot(gsR[0,0])
    axNums = plt.subplot(gsR[0,1])
    axPie = plt.subplot(gsR[1,:])

    rng = np.random
    xNums1 = rng.randn(1000)
    xNums2 = rng.randn(1000)
    axDens.hist(xNums1,alpha=0.7,color='skyblue',
                bins=100,weights=[np.array(2)for _ in range(1000)],density=1)
    axDens.hist(xNums2,alpha=0.7,color='pink',bins=100,density=1)
    axDens.tick_params(length=0)
    axDens.set_title('two groups\'s density',backgroundcolor='orange')
    
    axNums.hist(xNums1,alpha=0.7,color='skyblue',
                bins=100,weights=[np.array(2)for _ in range(1000)])
    axNums.hist(xNums2,alpha=0.7,color='pink',bins=100)
    axNums.tick_params(length=0)
    axNums.set_title('two groups',backgroundcolor='red')
    
    axPie.pie([40,30,20,10],colors=['#a4f9e4','#f9c5a4','#c9a4f9','#d1f9a4'],
              autopct="%d%%",radius=1.5,
              labels=['yorushika','atarayo','zutomayo','yoasobi'],
              labeldistance=0.7,
              pctdistance=0.3,
              shadow=1,
              explode=[0.1,0,0,0])
    axPie.set_title("The four 'YoRu's in my mind")

    plt.show()

if __name__ == '__main__':
    main()
```

`run`：



<img src="plt_run.png" alt="plt_run" style="zoom:80%;" />


