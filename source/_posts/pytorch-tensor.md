---
title: "Pytorch: Tensor(张量)"
date: 2025-01-15
categories: [Programming, Python, Pytorch]
tags: [pytorch, big data]
---
<!-- placeholder -->
<!-- more -->
### `Tensor`(张量)

- 张量源于数学，并在微分几何中得到应用，其实早在线性代数中就已经学过，向量(一维张量)可以通过线性变换(乘上坐标变换矩阵，即一个二维张量)，在不同的坐标系下表示同一个坐标
  在物理中，张量的特性与相对论假设一拍即合，用于描述多个物理量的排列组合，并可以通过线性变换在不同参考系下保持不变，看过广义相对论的科普视频可以知道，这种帮助其它张量线性变换的张量还叫度规张量，并且非常常用
  在机器学习中，张量就是一个多维数组，整个张量是表示数据的基本单位，如果具体到数组中的每一个元素，可能它们并没有什么实际意义
  引入张量是为了提供一个便于用户计算的接口：例如前向传播中，每一步都要进行矩阵运算，如果只提供一个一维数组，那功能也太弱了；更何况，图片、音频、视频等数据在编码后也需要更高维的数组进行存储

- `torch`是在神经网络领域上能够完全替代`numpy`的库，除了一些小众的数学计算，许多数学函数、多维数组操作`torch`都能提供，并在此基础上添加了便于深度学习的函数
  如果你学过`numpy`，可以跳过一大部分这里介绍的方法和机制，很多时候将`np.`换成`torch.`其效果是一样的

- 张量的属性：

  - `dtype`：一个张量的所有元素类型必须是一致的，被封装为`torch.dtype`，支持各种位数的整数、浮点数等类型
    只能通过张量的实例属性`dtype`访问该张量的元素类型，因为当你试图通过索引访问张量的元素时，返回的仍是`torch.Tensor`(即使只访问一个元素)，`type()`是看不到内部的元素属性的
    很多深度学习相关方法要求张量的`dtype`为浮点类型

  - **`device`**：`device`对象，标记张量部署的设备，如`cpu、cuda`等

  - `shape`：返回`torch.Size`对象，是封装后的元组，表示张量的尺寸
    元组的长度表示张量的维度，元组的第`i`个元素表示这个维度上的元素个数，例如：

    ```python
    tensor(1)   # Size([])  元组为空, 表示标量
    tensor([1])   # Size([1])  元组长1, 表示一维、1个元素
    tensor([1, 2])  # Size([2])  元组长1, 表示一维、2个元素
    tensor([[1, 2]]) # Size([1, 2]) 元组长2, 表示二维、一行两列
    ```

    可以用切片符获取`Size`元组的信息，和内置元组操作一致
    实例方法`size()`返回值和`shape`一致、`size(dim=num)`和`shape[num]`一致
    <img src=".\pictures\torch_Tensor_size.png" alt="torch_Tensor_size" style="zoom:40%;" />
    顺便提一下`ndim`属性，即维度大小

  - **`requires_grad`**：布尔值，标记这个张量是否需要梯度；要想让`torch`在反向传播中自动跟踪并计算梯度，就必须使`requires_grad=True`
    后续，所有基于`requires_grad=True`的张量运算而得的张量，`requires_grad`自动设置为`True`
    梯度会被保存在`grad`属性中(如果是叶子张量)
    需要注意，只有浮点类型的张量才允许需要梯度

  - `is_leaf`：布尔值，标记这个张量是否为叶子张量
    在创建的一瞬间，如果这个张量不需要梯度，或者需要梯度但由用户直接创建，则为叶子张量
    否则，如果这个张量需要梯度、并且由其它张量运算得出，则不是叶子张量

    ```python
    a = torch.tensor([1])      # 默认不需要梯度
    b = torch.tensor([1], requires_grad=True) # 需要梯度
    aa = a * 2
    bb = b * 2  # requires_grad自动设置为True
    # obj   is_leaf
    # a    True
    # b    True 需要梯度, 且由用户直接创建, 为叶子张量
    # aa   True 不需要梯度, 为叶子张量
    # bb   False 需要梯度, 但不由用户创建, 而是运算得出, 不是叶子张量
    ```

    开发者不需要过于关心`is_leaf`的值，但要知道，**只有`is_leaf=True`的张量的梯度会被保留**
    **`requires_grad`决定是否计算该张量的梯度、`is_leaf`决定一个张量的梯度是否释放**
    这是理所当然的，一般我们只需要权重、偏置张量的梯度，而不需要中间结点值的梯度
    <img src=".\pictures\Tensor_is_leaf.png" alt="Tensor_is_leaf" style="zoom:50%;" />

  - `grad_fn`：布尔值，标记创建该张量的函数对象，所有叶子张量的`grad_fn`均为`None`
    如果这个张量需要梯度，且由其它张量运算得出，则`grad_fn`指向这个运算函数

  - `layout`：`layout`对象，表示张量的内存布局，默认`strided`(密集张量)、可选`aparse_coo`(`COO`格式的[稀疏矩阵](https://www.cnblogs.com/xbinworld/p/4273506.html)
    `stride`(步长)表明，无论维度如何，所有数据的物理地址是连续的，通过步长来访问维度
    例如对于三行三列的二维张量，当用户访问第三行第三列的元素(第`9`个元素)时，指针(初始指向第`1`个元素)先增加两次第一维度的步长`3`、再增加两次第二维度的步长`1`，找到目标的实际物理地址
    实际上，张量在实现上就是用步长元组`stride`和一维列表`UntypedStorage`存储的
    你可以用`untyped_storage()`返回张量的一维映射，用于理解其实现

- **创建张量**：

  - 深拷贝：使用工厂函数**`torch.tensor(data)`**
    通过**创建副本**的方式返回`Tensor`对象
    `data`可以是列表、元组、`np`数组等，由`tensor()`挑选一个最适合的`dtype`作为元素类型
    如果将元组传递给`tensor()`，后续修改张量也没有影响，因为会转换为可读写张量
    允许用户自定义`dtype、device、requires_grad`，默认不需要梯度

  - 浅拷贝：使用工厂函数`torch.as_tensor(data)`
    默认情况下，对于除`np`数组、张量以外的对象，和`tensor()`一致，**对于`np`数组以及张量，通过浅拷贝返回张量(前者调用`from_numpy()`)**
    但用户可以指定`dtype、device`，如果指定的这两个属性和`np`数组、张量这些默认进行浅拷贝行为的数据不一致，则进行深拷贝而不是浅拷贝
    <img src=".\pictures\Tensor_as_tensor.png" alt="Tensor_as_tensor" style="zoom:35%;" />

  - 其它特殊的方式：

    - **`torch.detach(Tensor)`或`self.detach()`**：产生和原张量共享内存的新张量，这个新张量的`requires_grad=False`，但是对其数据的改动会影响原张量

    - `zeros(Seq)、ones(Seq)、empty(Seq)、full(Seq, fill_val)`：创建全为零、全为一、未初始化、全为指定值的张量
      其中`Seq`是表示其维度的序列
      其它参数的使用与`tensor()`类似

    - `rand(Seq)、randn(Seq)`：创建其值的分布满足$U(0,1)、N(0,1)$的张量
      `self.uniform_(beg, end)、normal(size、mean、std)`：创建其值的分布满足$U({\rm beg,\ end})、N({\rm mean、std^2})$的张量，前者是张量的实例方法(依赖已有张量的尺寸)、后者是工厂函数(用户指定尺寸)

    - `arange()、linspace()、logspace()`：用法和内置方法差不多，返回值变成张量而已

      ```python
      # beg->(endnum<end)的间隔为step的等差数列
      torch.arange(beg=0, end, step=1)
      # beg->end的元素个数为steps的等差数列
      torch.linspace(beg, end, steps)
      # 以base为底, 以linspace(b,e,steps)返回的序列作为幂, 得到的张量
      torch.logspace(b, e, steps, base)
      ```

    - `eye()、diag()、triu()、tril()`：创建单位、对角、上三角、下三角矩阵

  - 其它功能有限但也有人用的方式：
    `Tensor()`：构造方法，使用`float32`作为`dtype`，且用户不能指定
    各种指定类型的构造方法，如`FloatTensor、BoolTensor`，可能有人喜欢用
    `from_numpy()`：只能传递`np`数组作为`data`，与`as_tensor(np)`效果一致

- 张量的梯度相关操作：

  - `backward()`：对该张量所在计算图进行反向传播，计算所有前置结点的梯度
    当张量为标量时，无需提供任何参数
    当张量不为标量时，需要提供`gradient:Tensor`表示求和时的权重系数
    两者其实没什么区别；例如，在多输出神经网络中，习惯对各个输出结点产生的损失进行求和得到总损失再向前求梯度，此时相当于手动地进行了`gradient=ones(n)`的求和，让损失从向量变成了标量，例如：

    ```python
    # 输出结果相同
    b = torch.tensor([[1, 2], [3, 4]], requires_grad=True, dtype=torch.float32)
    c = b.pow(4) # 向量
    c.backward(torch.ones(c.shape)) # 初始权重系数全为1, 等效于单纯的求和
    print(b.grad)
    b.grad = None
    d = b.pow(4).sum() # 标量, 原向量单纯的求和
    d.backward()
    print(b.grad)
    ```

    自动求导的好处是，不用再计算什么$\boldsymbol\delta$，只需要调用`backward()`即可

  - `requires_grad_()`：设置张量为需要梯度(是原位赋值方法)

  - `with torch.inference_mode():`：创建不追踪梯度的上下文环境，在其下的代码块中所有的张量运算不记录到计算图里
    是新版本中`with torch.no_grad():`的上位替代

  - `retain_grad()`：之前学过`is_leaf`，所有该属性为`False`的张量的梯度不会被保留
    使用该方法可以在计算后保留非叶子结点的梯度

  - `detach()`：产生张量的一个映射，该映射不需要梯度，不再赘述

  - 关于梯度清零：在若干次计算后，历史梯度会直接累加，应该尝试对梯度清零，不过这是优化器的事了

- 张量的类`numpy`操作：具体的参数都不重要，看到别人代码能看懂即可

  - **切片索引符**：基于内置列表的切片索引符，用逗号来分隔不同维度的切片，例如

    ```python
    # a是二维张量
    a[0:2, 0:3]   # 第一维度切0:2, 第二维度切0:3, 所以返回前两行、前三列围成的张量
    ```

  - `add()、sub()、mul()、div()`对应`+、-、*、/`，为张量逐元素加减乘除
    `matmul()`对应`@`，为二维张量的矩阵相乘
    `exp()、log()、pow()`：逐元素`e`的指数、`e`的对数、幂运算
    `t()`对应属性`T`，返回二维张量的转置(和原张量共享内存)
    所有上述方法都有方法名末尾加后缀`'_'`的实例方法版本，它们不创建新张量对象而是直接修改原有张量，称为**原位赋值**；之前的`zeros()`等也有原位赋值版本

  - **条件操作**：张量配合条件运算符(例如比较运算符)，将逐元素判断并返回`dtype=bool`的张量
    这个布尔张量可以用作掩码并经过索引符来筛选并返回一维张量，按顺序存储通过筛选的元素
    **`torch.where(cond,input,other)`或`self.where(cond,other)`**则是更通用的方法，类似于`C`的`?:`三元运算符的多维数组版本，`cond`是一个布尔张量，逐元素判断，当条件满足时返回`input`在该位置的值、不满足时返回`other`在该位置上的值
    `torch.where(cond,self,other)`和`self.where(cond,other)`效果一致

    ```python
    a = torch.tensor([[1, 2],[3,4]])
    m = a > 1  # 布尔张量 Tensor([[False,True],[True,True]])
    a[m]   # 一维张量 Tensor([2, 3, 4])
    a[m] = 1  # 支持原位赋值, 现在a为Tensor([[1,1],[1,1]])
    # a[m]=1和a=torch.where(a>1,1,a)效果一致,区别在于前者是原位赋值

  - `sum()、max()、min()、mean()、std()、var()`求张量的和、最大值、最小值、均值、标准差、方差

  - `dot()、cross()`：一维张量的点积、叉积

  - `self.view(Size)、reshape(Size)`：改变张量的`shpae`并返回新张量
    这个新张量和原有张量共享内存，`reshape()`在`view()`的基础上添加了保底(无法共享内存时进行深拷贝创建新张量)

  - 广播机制：当张量运算时，如果维度不相符，并且符合广播的要求，则会根据广播机制扩展某一方的维度
    其实和`numpy`的广播机制一个样：[`Broadcasting — NumPy`](https://numpy.org/doc/stable/user/basics.broadcasting.html)

  - `save()、load()`：张量的序列化、反序列化，人话说就是内存存到外存、外存加载到内存

  - `to()`：可以修改张量的类型、部署的设备，返回一个对原数据进行深拷贝后的新张量

  - 更多矩阵操作(例如求秩)在`torch.linalg`中
