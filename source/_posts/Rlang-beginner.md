---
title: "Rlang: 虽然不知道为什么要学"
date: 2025-11-26
categories: [Programming, Rlang]
tags: [R, stat]
---
<!-- placeholder -->
<!-- more -->
# 统计学习

## 线性回归

### `stats`包

- `c(...)`：使参数拼接为一个向量
- `sample(tot, sz)`：生成一组索引，从`tot`个样本中随机采样`sz`个样本
- `poly(x, k, raw, ...)`：生成关于`x`的`k`阶多项式，`raw`布尔值表示`TRUE`(正交多项式)、`FALSE`(原始多项式)，正交多项式的每一项是原始多项式($x^i$)的线性组合
- `lm()`：
  - `formula`：模型公式，例如`y ~ x1 + x2`
    - `+`：线性相加
    - `.`：通配符，包括所有变量的`+`
    - `-`：去除变量，通常配合`.`
    - `:`：交互项
    - `*`：表示包含两者以及两者的交互项，例如`x1*x2`等价于`x1+x2+x1:x2`
    - `0`或`-1`：表示去除截距项
    - `I(x^k)`：多项式项(`k`次幂)
  - `data`：所用的数据，必须包含`formula`中使用的变量
  - `subset`：逻辑表达式或者索引，在训练前先作用于`data`
- `glm()`：广义线性模型，相比`lm()`添加了`family`参数
  - `gaussian`：默认值，进行线性拟合
  - `binomial`：逻辑回归
- `summary(model)`：获取模型的总结，包括系数、标准差、$t$值、$t$假设检验的`p`值、$RSE$、$R^2$(`adj`表示加了惩罚项)、$F$统计量、$F$假设检验的`p`值
  - 逻辑回归的总结还包括零模型偏差和完整模型偏差，以及`AIC`估计
- `coef(model)`：返回更少的内容，只返回系数的估计值
- `anova(models...)`：通过方差分析，判断更复杂的模型新增的变量是否有显著影响
- `predict(model, data, interval, level, type)`：使用模型进行预测
  - `data`：待预测的数据
  - `interval`：指定预测区间还是置信区间，可选`confidence`、`prediction`，默认为`none`(点估计)
  - `level`：指定置信水平，默认`0.95`
  - `type`：指定预测类型，可选`response`、`link`，指定为`response`来获得响应值，因为默认的`link`难以解释
- `which.min(vec)`：求出最小值对应的下标，`max()`类似

## 可视化

- `plot(x, y)`：绘制散点图
- `abline(model)`：绘制模型的最小二乘线

## `MASS`(现代应用统计学)

- `lda()`：线性判别分析
- `qda()`：二次判别分析

## `boot`

- `cv.glm(data, glmfit, K, cost)`：对`glm()`的模型进行`k`折交叉验证，返回值的`delta`列的第一项是标准的交叉验证估计值(`res$delta[1]`)
- `boot(data, statistic, R, ...)`：自助法估计指定`statistic`统计量的某个指标
  - `data`：原始数据集
  - `statistic(data, indices)`：自定义函数，在里面指定模型、返回感兴趣的统计指标，例如：

    ```R
    function(data, indices)
       d <- data[, indices]        # indices是boot()提供的索引组
       model <- lm(y ~ x, data=d)
       pred <- predict(model, d, type="response")
       return mean(pred == d$Y)
    ```

## `leaps`

- `regsubsets(y ~ ., data, method, nvmax, ...)`：默认为最优子集法，`method`可选`forward`和`backward`
  - `summary()`可返回`rsq, rss, bic, adjr2, cp`统计量

## `glmnet`

- `glmnet()`相比`glm()`新增了`alpha`和`lambda`参数，`alpha=0`表示岭回归(默认值)，`alpha=1`表示`Lasso`回归，`lambda`为惩罚系数，可以通过交叉验证求得
- 一般通过`predict()`使`type="coeficcients"`获取系数，因为使用压缩估计的目的是变量选择，所以不使用`summary()`
- 使用`cv.glmnet()`来选择超参数`lambda`，它会在内部自动调用`glmnet()`

## `pls`

- `pcr()`：`scale`参数
- `validationplot()`
- `plsr()`：偏最小二乘回归
- `scale=TRUE`参数表示标准化，`validation="cv"`表示使用交叉验证来估计测试误差

## 非线性模型

- 通过`cut(x, sz)`划分变量范围，来拟合阶梯函数

## `splines`

- `bs()`表示样条函数，默认为三次样条
- `smooth.spline()`：光滑样条
- `loess()`：局部回归

## `gam`

- `gam()`：广义可加模型
