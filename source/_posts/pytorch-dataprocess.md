---
title: "Pytorch: 使用torch.utils.data封装数据"
date: 2025-11-26
categories: [Programming, Python, Pytorch]
tags: [big data]
---
<!-- placeholder -->
<!-- more -->
### 数据集集成框架

- 数据的流向：先将数据交给自定义的数据集类(其任务为对源数据预处理，包装成数据加载器能够访问的数据集)，通常转化为张量
  通过数据加载类进行采样并送给模型计算并优化(其任务为根据默认或自定义的采样器`Sampler`进行真正的采样)

- `map-style`和`iterable-style`：索引式数据集和迭代式数据集

  - 索引式数据集的做法是将整个数据集读取到内存，采样的时候通过`[]`访问即可
    适用于批量读取、小数据集，由于数据均在内存中，往往更快
  - 迭代式数据集的做法是事先定义迭代的行为，采样的时候从硬盘中取出数据并作为迭代器返回
    适用于各种数据无法事先全部读入内存的情况(例如训练过程中会产生新样本、或内存过小)，当然是比较慢的

- `torch.utils.data.Dataset`(简称`Dataset`)是所有**数据集类型的基类**，用户如果需要构建自己的索引式数据集，必须直接继承该类并重载以下方法(其实就是封装成一个索引式数据集)：

  - `__len__()`：应返回数据集的长度，以便可以用`len(obj)`访问数据集长度
  - `__getitem__(idx)`：应返回该索引指向的数据，以便可以用`[]`访问数据

- `torch.utils.data.IterableDataset`(简称`IterableDataset`)是所有**迭代式数据集的基类**，继承该类后必须重载：

  - `__iter__()`：`yield`一个迭代器，一般是在方法内临时读取文件并返回一条数据

- `torch.utils.data.DataLoader`(简称`DataLoader`)是**数据集对象的迭代器**，相比诸多语言的内置迭代器封装了很多功能，不需要手动实现乱序、分批读取等，加载数据集十分方便，其构造方法`__init__()`参数用法为：

  - `dataset`：`Dataset`(或子类)对象，表示要遍历的数据集
  - **`batch_size`**：批数据的大小，即通过该迭代器遍历时返回的**样本个数**；将其设置为`None`以关闭自动批处理
  - `sampler`：可以**指定采样器进行更多样的采样行为**
  - `batch_sampler`：类似`sampler`，但要求用户提供的采样器必须**返回一批张量的索引**而不是一个张量的索引，这一批张量均用用户提供的`Sampler`对象来采样，它和`sampler、batch_size`是冲突的(因为认为用户已经在`sampler`里定义了批量大小)
  - **`shuffle`**：布尔值，决定**是否乱序采样**，和`sampler、batch_sampler`冲突(因为要使用默认的`RandomSampler`)，没有更高需求的采样行为而懒得自定义采样器时，使`shuffle=True`就足够了

  只提供`dataset`，其它参数保持默认下，迭代器会顺序地进行批量为一的自动批处理采样
  <img src="DataLoader_init.png" alt="DataLoader_init" style="zoom:40%;" />
  其它不常用的参数：

  - `num_workers`：将数据分给多个子进程加载，默认只用主进程加载
  - `collate_fn`：是一个可调用对象，表示对数据集采样后，返回给用户前进行的行为
    默认的`collate_fn`常在一个开启了自动批处理的索引式数据集中用到，会**将采样的数据合并为一批**
    因为迭代式数据集的批量采样等行为通常是由用户在`__iter__()`中实现的，所以对迭代式数据集不会提供默认的`collate_fn`
    用户可以自定义`collate_fn`
  - `pin_memory`：由于`GPU`中的内存全是锁页内存，但`CPU`中一般只是不锁页内存(即会被交换到虚存中)，设置`pin_memory=True`，将使读入的样本变成锁页内存，通过**消耗`CPU`内存的方式加快其载入`GPU`的过程**
  - `drop_last`：布尔值，表示是否丢弃最后一个不完整的批次
  - `timeout`：表示数据加载的时限

- `torch.utils.data.Sampler`(简称`Sampler`)是采样器类，表示采样的策略，具体为提供一个索引序列
  内置的`Sampler`有：

  - `SequentialSampler`：顺序采样，每次返回一个索引，是未提供`sampler`且`shuffle=False`时的默认采样器
  - `RandomSampler`：全集的随机采样，是未提供`sampler`且`shuffle=True`时的默认采样器
  - `WeightedRandomSampler`：根据权重的随机采样
  - `SubsetRandomSampler`：对数据集子集的随机采样
  - `BatchSampler`：对其它`Sampler`产生的索引序列分批，顺序地提供的各个批次的索引序列(一次返回整个批次的所有索引)
    开启自动批处理时，最终会经过这个采样器进行分批

  用户可以继承`Sampler`或以上这些特化的采样器类，来自定义一个`Sampler`
  不过大多数情况下，上述采样器已经足够了
