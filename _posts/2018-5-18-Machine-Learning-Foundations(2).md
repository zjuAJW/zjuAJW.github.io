---
layout:     post
title:      "机器学习基石笔记(2)"
subtitle:   "线性回归和逻辑回归"
date:       2018-06-01 15:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
    - 机器学习
    - 数据挖掘
---

之前的课程我们讨论的是机器学习基础的理论，包括什么是机器学习以及为什么机器可以学习，接下来几节课会具体介绍几个机器学习算法，主要是线性回归和逻辑回归。

## Lecture 9 Linear Regression

线性回归这里介绍的是最经典的最小二乘，我们的损失函数为

$$E_{in}(\omega_{lin}) = \frac{1}{N}\sum_{i=1}^N(x_i^T\omega_{lin}-y_i)^2=\frac{1}{N}\Vert X\omega_{lin}-y\Vert ^2$$

直接求导然后令导数为0，我们可以得到

$$X^TX\omega_{lin} -X^Ty=0$$

$$\omega_{lin} = (X^TX)^{-1}X^Ty$$

这个推导之前已经很熟悉了，不过课程里对这个结果做了一个几何意义的解释。

根据推导的结果，我们的预测值$$\hat y=X\omega_{lin}$$，这可以看作是对$$X$$中列向量的一个线性组合，也就是$$\hat y$$是$$X$$的列向量张成的空间中的一个向量。而我们的目标是使$$y-\hat y$$最小，那么很明显，$$\hat y$$应该是$$y$$在span of X上的垂直投影，这样$$y-\hat y$$是垂直于sapn of X的，距离才会最小。

![9-1](\img\in-post\machine-learning-fundation\9-1.PNG)

所以，我们可以把结果写成$$\hat y=X\omega_{lin}=X(X^TX)^{-1}X^Ty=Hy$$，这里的$$H$$又被叫做Hat Matrix（给$$y$$加了一个帽子），它的作用就是做了投影这个动作。而$$y-\hat y = (I-H)y$$，也就是说，$$I-H$$是将$$y$$变成了垂直于span of X的那个向量。

如果我们的数据中有噪声存在呢？如下图，假设我们的$$y$$是由$$f(X)+noise$$产生的，注意这里的$$f(X)\in span\ of\ X$$。

![9-2](\img\in-post\machine-learning-fundation\9-2.PNG)

那么其实$$y-\hat y=(I-H)y=(I-H)noise$$（y和noise做投影时的垂线是一样的），所以我们可以得到：


$$
\begin{aligned}
\\\
E_{in}(\color{blue}{w_{LIN}})&=\frac{1}{N}||\color{green}{y-\hat{y}}||^2\\\
&=\frac{1}{N}||(I-\color{orange}{H})noise||^2 \\\
&=\frac{1}{N}trace(I-\color{orange}{H})||noise||^2 \\\
&=\frac{1}{N}(N-(d+1))||noise||^2
\end{aligned}
$$


至于这里这个$$trace(I-H)=N-(d+1)$$的结论，课程里也没有详细说，这里也就不给出证明了。

所以我们可以得到

![9-3](\img\in-post\machine-learning-fundation\9-3.PNG)

这个$$\bar E_{out}$$是怎么来的，课程里也没有说，这里就当结论给出好了。这两个公式说明了啥呢，我们把它们的图像画出来

![9-4](\img\in-post\machine-learning-fundation\9-4.PNG)

可以看到它们都是趋向于$$\sigma ^2$$（noise level）的，也就说明$$E_{in}$$和$$E_{out}$$的误差是被大概$$\frac{2(d+1)}{N}$$bound住的，这有点类似于之前说的VC bound，也是提供了一个学习可行性的保障。

## Lecture 10 Logistic Regression

逻辑回归这里也不多讲太多了，都很熟悉。

通过一个sigmoid函数将输出从实数范围转化到$$(0,1)$$，作为一个分类概率输出。模型的损失函数为交叉熵，采用梯度下降的方法来进行优化求解

![10-1](\img\in-post\machine-learning-fundation\10-1.PNG)

当然注意这里的类标记是+1或者-1，如果是1和0的话，交叉熵的形式可能会有所变化。

## Lecture 11 Linear Models for Classification

我们到目前为止一共学习了三个线性模型：PLA（其实严格来说PLA是一个算法，不过这里用它来代表最简单的线性分类器这个模型），Linear Regression，Logistic Regression，三个模型的主要区别就在于它们的损失函数不同。

PLA的输出就是一个样本的标签，但是它的求解过程是一个NP-hard的，不是很容易。而其他两个问题优化起来都相对比较容易，所以我们能不能用回归来解决分类问题呢？我们来看一下三者损失函数的对比。

![11-1](\img\in-post\machine-learning-fundation\11-1.PNG)

可以看到三者都和$$ys$$有关，我们以$$ys$$为横坐标，损失函数的值为纵坐标，画出它们的图像

![11-2](\img\in-post\machine-learning-fundation\11-2.PNG)

注意这里稍微做了一些改变，我们把逻辑回归的损失函数做了一个换底操作，换成了以2为底的对数，也就是将原损失函数除了一个$$ln2$$，这样做的目的是逻辑回归的损失函数曲线就会和PLA的曲线在$$ys = 0$$的地方相切，方便后续的分析和推导。

可以看到linear regression和logistic regression的损失函数是PLA的上界，这样可以确定的是，linear regression和logistic regression认为做的好的事，PLA也认为是做的好的，所以我们就可以用回归来做分类了。

这里也放上三个算法的对比和优缺点

![11-3](\img\in-post\machine-learning-fundation\11-3.PNG)

通常，逻辑回归还是我们更常用的算法。

### 随机梯度下降

上面我们分析了用逻辑回归做分类的可能性，但是有一个问题是，逻辑回归每轮迭代的时候要利用所有的样本数据来计算梯度信息，而PLA每轮迭代只需要一个样本点，那么有什么办法能让逻辑回归也把复杂度降到O(1)吗？

这里我们就用到了随机梯度下降，每次计算梯度的时候，不是用全部样本数据，而是只用一个样本数据，这样计算量就会小很多，对于数据量非常大或者在线学习的情况，随机梯度下降就显得更加实用。

随机梯度下降其实和PLA是非常相似的，PLA每次将直线往样本的方向进行偏移，而随机梯度下降同样也是进行偏移，只不过偏移量的大小跟错误的程度有关，错误比较大，就多移一点，错误比较小，就少移一点。

### 多分类问题

对于多分类，有两种解决方法，OVA(One vs. ALL)和OVO(One vs. One) 。

OVA就是每次将一个类别作为圈圈，其他都当作叉叉，来训练二元分类器，这样，有几个类别，就会有几个模型出来。预测的时候，分别用每个模型来预测，哪个模型预测结果为圈圈，那么结果就是对应的类别。当然，我们也可以输出各个类别的概率，以防出现多个模型预测为圈圈或者没有圈圈的情况。

OVA的缺点在于如果我们的类别很多的话，那么单个类的数据样本相对来说就很少，会有数据的不平衡问题。因此，OVO可以来解决这个问题。OVO每次只用两个类别的数据来进行模型的训练，预测时在所有模型中得票最多的类别作为最后的输出，但是OVO的缺点是所需要训练的模型数量更多，因此占用的空间更多。