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

## Lecture 12 Nonlinear Transformation

之前所讲的模型都是线性的，它们的优点很明显，就是VC维是可控的，我们可以确保$$E_{in}$$和$$E_{out}$$是接近的。但是缺点也很明显，对于实际上线性不可分的数据，我们的$$E_{in}$$可能一直很大。

那么如何处理这种情况呢？非线性变换是一个思路

![12-1](\img\in-post\machine-learning-fundation\12-1.PNG)

图中的数据是线性不可分的，但是通过一个非线性变换，可以将原坐标转换为线性可分的，在新的坐标空间下，就可以利用之前所使用过的线性分类器来进行数据的分类了。当然，我们这里的变换只能变换圆形的边界，我们还可以增加其他的交叉项来产生椭圆等边界。

这看上去很美好，但是要注意，这种变换额外引入了更多的参数，也就是说模型的复杂度或者说VC维其实增加了。假设我们的数据有$$d$$个维度，而我们变换后的多项式最高次为$$Q$$，那么变换后的参数个数是$$Q^d$$这个数量级的（准确的应该是$$C_{Q+d}^d$$或者$$C_{Q+d}^{Q}$$）,所以我们做了变换的代价是模型的复杂度可能会变得更高。

还记得我们之前做的VC dimension和error的关系图吗？

![12-2](\img\in-post\machine-learning-fundation\12-2.PNG)

诚然，我们采用维度更高的变换，可以得到更低的$$E_{in}$$，但是很可能过高的复杂度会导致$$E_{out}$$反而变大了。所以，我们应该从简单的模型做起，逐渐增加模型的复杂度。

## Lecture 13 Hazard of Overfitting

过拟合通常是指$$E_{in}$$很小，但是$$E_{out}$$很大的情况。

![13-1](\img\in-post\machine-learning-fundation\13-1.PNG)

如图所示，讨论了噪声和数据量对过拟合的影响。

在第一个图中，target function是一个10次多项式，另外加了一些噪声。我们分别用2次多项式和10次多项式的Hypothesis set来进行拟合，结果发现$$\cal H_{10}$$反而过拟合了。原因就在于我们的数据量不足。

![13-2](\img\in-post\machine-learning-fundation\13-2.PNG)

以上的学习曲线是我们在第九讲中提到的，在数据量非常大的时候，$$\cal H_{10}$$的$$E_{out}$$是更小的，但是在$$N$$比较小的时候，$$\cal H_{10}$$的$$E_{out}$$是要更大的，所以在数据量很少的时候，$$\cal H_{10}$$会过拟合。

这是在有噪声的情况下，那么对于没有噪声的情况呢？在本讲第一幅度的右边，target function是一个50次的多项式，但是没有添加噪声，而从结果来看，$$\cal H_{10}$$还是过拟合了。这种情况下过拟合的原因在于，我们的target function本身太过于复杂了，不管$$\cal H_{2}$$还是$$\cal H_{10}$$都没有办法很好的去拟合，这种复杂性本身就相当于是一种噪声，我们称之为**Deterministic Noise**，确定性噪声。确定性噪声可以认为是$$\cal H$$中最好的一个$$h^*$$与target function之间的差值，因此它是与$$\cal H$$ 有关的。

![13-3](\img\in-post\machine-learning-fundation\13-3.PNG)

关于这个确定性噪声，我感觉自己没有完全理解，，，，，，

最后给出一个更加细致的分析图：

![13-4](\img\in-post\machine-learning-fundation\13-4.PNG)

上图中，红色越深，代表overfit程度越高，蓝色越深，代表overfit程度越低。先看左边的图，左图中$$\cal H$$阶数$$Q_f$$固定为20，横坐标代表样本数量N，纵坐标代表噪声水平$$\sigma^2$$。红色区域集中在N很小或者$$\sigma^2$$很大的时候，也就是说N越大，$$\sigma^2$$越小，越不容易发生overfit。右边图中$$\sigma^2=0.1$$，横坐标代表样本数量N，纵坐标代表目标函数阶数$$Q_f$$。红色区域集中在N很小或者$$Q_f$$很大的时候，也就是说N越大，$$Q_f$$越小，越不容易发生overfit。上面两图基本相似。 

从图中我们也可以总结出会造成overfitting的几个原因：数据量少、噪声大、模型过于复杂都会导致过拟合。

那么我们如何避免过拟合呢？主要有以下几种方法：

1. data cleaning/pruning：对数据里label明显错误的样本进行修正或者去除
2. data hinting：数据量不够时，可以想办法获得更多数据，这里data hinting是指对原有数据进行简单的变换，比如一些平移、旋转等等。
3. regularization和validation：regularization主要用来控制模型复杂度，而vallidation可以看作是一种early stopping，这个我们在后续会详细展开。

## Lecture 14 Regularization

正则化的问题因为之前就了解过一些(详见[正则化](https://zjuajw.github.io/2018/03/26/regularization/))，所以这里就简单跳过了，没有什么太新的东西。

## Lecture 15 Validation

