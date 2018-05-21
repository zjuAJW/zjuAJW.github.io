---
layout:     post
title:      "机器学习基石笔记"
subtitle:   "夯实基础"
date:       2018-05-18 10:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
    - 机器学习
    - 数据挖掘
---

林轩田老师的课程是接触机器学习开始就知道的，但是当时对于初学者的我其实有点难，所以没有看下去，马上要到找工作的时候了，想要把这个经典课程看一下，顺便复习一些基础知识，夯实一下基础。

## Lecture 1 2 3

前三讲主要是一些基础的介绍，这里就不多说了，唯一需要注意的是整个课程对于machine learning的一个定义。

![machine learning](\img\in-post\machine-learning-fundation\ml.PNG)

所谓machine learning，就是利用数据data，采用某种算法$$\cal A$$，从hypothesis set（后文基本都用$$\cal H$$来表示）中选择一个g，使得g和目标函数f非常接近。

## Lecture 4: Feasibility of Learning 

### Learning is Impossible？

首先来看一个简单的例子

![4_1](\img\in-post\machine-learning-fundation\4_1.PNG)

如图所示这样一个二分类的问题，我的hypothesis set其实一共只可能有2^8种，那我们从中选出一个g，使得在我们看到的这5个数据上，$$g(x_n)=y_n$$, 那么g和f就是很接近的吗？答案是否定的

![4_2](\img\in-post\machine-learning-fundation\4_2.PNG)

在$$\cal H$$中，我们可以找到对于已知的五条数据，完全符合的g一共有8个，但是他们在未知的数据上，他们的表现就不一定了。也就是说，在已知数据D上，$$g \approx f$$是成立的，但是在D以外的数据上，$$g\approx f$$不一定成立。而机器学习目的，恰恰是希望我们选择的模型能在未知数据上的预测与真实结果是一致的，而不是在已知的数据集D上寻求最佳效果。 

这个例子告诉我们，我们想要在D以外的数据中更接近目标函数似乎是做不到的，只能保证对D有很好的分类结果。机器学习的这种特性被称为没有免费午餐（No Free Lunch）定理。 NFL说明了无法保证一个机器学习算法在D以外的数据集上一定能分类或预测正确，除非加上一些假设条件，我们以后会介绍。 

### Probability to the Rescue

上一节的结论似乎说明了在已知数据之外，机器学习的模型很难做到正确的分类或者预测，那么有没有一些工具能让我们做一些推论，使得机器学习变得有用呢？

我们考虑这样一个问题：

![4_3](\img\in-post\machine-learning-fundation\4_3.PNG)

假设一个罐子里装满了绿色和橙色的球，我们如何知道里边橙色球的比例？统计学上的做法是做一个抽样，然后用抽样出的球中橙色球的比例来估计真实的比例。为什么可以这样做？因为从概率的角度来看，样本中橙色球的比例$$\nu$$是很有可能接近于实际的橙色球比例$$\mu$$的，这就是**Hoeffding‘s Inequality**

![4_4](\img\in-post\machine-learning-fundation\4_4.PNG)

Hoeffding‘s Inequality保证了在抽样样本N非常大的时候，$$\nu$$和$$\mu$$是非常接近的，这时我们称，$$\nu = \mu $$ 是probably approximately correct (PAC) 的。

### Connection to Learning

那上边这个问题和machine learning有什么关系呢？我们可以把机器学习的概念对应过来。机器学习hypothesis与目标函数相等的可能性，类比于罐子中橙色球的概率问题；罐子里的一颗颗弹珠类比于机器学习样本空间x；橙色的弹珠类比于$$h(x)$$与$$f(x)$$不相等；绿色的弹珠类比于$$h(x)$$与$$f(x)$$相等；从罐子中抽取的N个球类比于机器学习的训练样本D，而且是独立同分布的。所以呢，如果样本N够大，且是独立同分布的，那么，从样本中$$h(x)\neq f(x)$$的概率就能推导在抽样样本外的所有样本中$$h(x)\neq f(x)$$的概率是多少。 

![4_5](\img\in-post\machine-learning-fundation\4_5.PNG)

这里定义两个量：$$E_{in}(h)$$表示在抽样样本上，$$h(x)\neq f(x)$$的概率，而$$E_{out}$$表示在所有样本上，$$h(x)\neq f(x)$$的概率。那么，对于某一个**固定的h**， 根据Hoeffding‘s Inequality， 我么可以得到，$$E_{in}(h)$$和$$E_{out}(h)$$在N很大的时候，是PAC的。

![4_6](\img\in-post\machine-learning-fundation\4_6.PNG)

但是要注意一点，这里只是说$$E_{in}(h)$$和$$E_{out}(h)$$非常接近，但$$E_{in}(h)$$并不一定是最小的，因此我们只是说可以利用这一点，来验证某一个h的表现是怎样的。

### Connection to Real Learning

之前我们一直都是在说固定一个h的情况，那么现在问题来了，现在有很多个假设h，其中一个h在样本数据上的输出全部都是正确的，我们要不要选择这个h来作为最后的g呢？

先来看一个例子，假设现在有150个人抛硬币，每人抛5次，其中一个人5次都是正面，我们能说这个人的硬币抛出正面的概率比较大吗？

![4_7](\img\in-post\machine-learning-fundation\4_7.PNG)

答案是否定的，我们可以看到，在每个硬币都均匀的情况下，150个人中至少有一个人抛出5个正面的概率其实是非常大的。对应到我们的学习过程中，可能每个h的$$E_{out}(h)$$其实都是1/2，但是其中某个h在样本上全部都是正确的，但我们并不能说这个h是好的。

Hoeffding不等式告诉我们什么呢？对于一个h来说，它在样本数据上的错误率和在所有样本上的错误率大概率是相等的。也就是说，假如我们从样本空间中抽样抽非常多次，得到很多很多不同的样本出来，那么在这些样本上，h只在很少一部分上会出现$$E_{in}(h)$$和$$E_{out}(h)$$差别很大的情况，我们称这样的抽样为BAD sample，对应的样本数据为BAD data。对应到下图，Hoeffding不等式保证了，**对于每一行**（也就是每一个h），其中BAD的格子数量是很少的。

![4_8](\img\in-post\machine-learning-fundation\4_8.PNG)

那如果现在我有很多h，对于表格中的某一列而言（也就是某种样本数据D），如果其中有一个格子是BAD的，那么我的算法就不能自由地从所有的h中去选择，因为可能踩到坑，也就是这个D对于某个h而言可能是BAD的，例如，可能$$h_1$$其实是一个很好的方程，但由于BAD sample，导致$$h_1$$在这个D上的$$E_{in}(h_1)$$很大，但$$E_{out}(h_1)$$其实很小，那么$$\cal A$$可能就不会选$$h_1$$了。同样的，可能$$h_2$$是一个不好的方程，但由于BAD sample，导致$$h_1$$在这个D上的$$E_{in}(h_2)$$很小，但其实$$E_{out}(h_2)$$很大，$$\cal A$$可能会误选$$h_2$$作为g。

如果$$\cal H$$中有任意个h遇到BAD data，我们的$$\cal A$$在挑选h的时候就会遇到麻烦，那么出现这种情况的概率是多少呢？见下图：

![4_9](\img\in-post\machine-learning-fundation\4_9.PNG)

其中M是h的数量，也就是$$\cal H$$的大小。

这样我们得到了一个上界，所以在M有限的情况下，N够大情况下，我们可以保证D对于每一个h都不是BAD的。

进而，我们似乎就可以做到一些Learning了：假如我们的**$$\cal H$$是有限的**，并且**数据量足够大**，那么不管我们的算法怎么选，都可以保证$$E_{in}(h)$$和$$E_{out}(h)$$都是接近的，那么我们就可以从所有的h中选择一个$$E_{in}(h)$$最小的来作为我们的g了。

那么问题又来了，大部分情况下，$$\cal H$$并不是有限的，这时候，我们还能够学习到知识吗？这个要等到后面的课程来解决了。



## Lecture 5

根据前几节的讨论，我们可以把机器学习转化为两个核心问题：

1. 我们能否保证$$E_{in}(h)$$和$$E_{out}(h)$$是接近的
2. 我们能不能使$$E_{in}(h)$$足够小

这里我们会遇到一个两难的问题：

* 要保证$$E_{in}(h)$$和$$E_{out}(h)$$是接近的，那么$$\cal H$$的大小M要是有限的，这样，我们可选的h就很少了，不能保证一定有$$E_{in}(h)$$很小
* 如果我们的M很大，那么我们可选的h就很多，选择到$$E_{in}(h)$$很小的h的可能性就很大，但是这样我们又不能够保证$$E_{in}(h)$$和$$E_{out}(h)$$是接近的。

可见这个**M**的值非常的重要，接下来的几节课也一直在围绕这个M的值展开，同时还有与M有关的这个关键的不等式，我在这里再写一遍：
$$
\mathbb{P}[| E_{in}(g) - E_{out}(g) | \gt \epsilon] \leq 2\cdot M \cdot exp(-2\epsilon ^2N)
$$

### Effective Number of Lines

首先我们来回忆一下这个M是从哪里来的。

在我们对上面那个公式进行推导的时候，我们用了一个Union Bound

![5-1](\img\in-post\machine-learning-fundation\5-1.PNG)

公式中等号成立的条件是什么？条件是各个事件是互斥的，也就是没有交集。但是实际上，我们的hypothesis有一些是非常相似的，对于这些相似的hypothesis，往往他们的Bad Data也是相似的，也就是说各个hypothesis遇到Bad Sample这件事并不是互斥的，是有overlapping的，在这种情况下，我们使用union bound其实是过分的估计了P的上限。

![5-2](\img\in-post\machine-learning-fundation\5-2.PNG)

既然如此，我们为什么不把相似的hypothesis归为一类呢？

比如，对于平面上的线性二分类问题，我们的$$\cal H$$是平面上的所有直线，这个集合是无穷大的。但如果现在我们只有一个数据点，那么这些直线其实可以分为两类，一类将点分为圈圈，一类分为叉叉。

![5-3](\img\in-post\machine-learning-fundation\5-3.PNG)

同样的，如果有两个点，那我们的直线可以分为4类

![5-4](\img\in-post\machine-learning-fundation\5-4.PNG)

三个点，我们的直线最多可以分为8类

![5-5](\img\in-post\machine-learning-fundation\5-5.PNG)

但是有一点要注意，如果三点共线的话，有两种情况我们是得不到的，但是我们这里考虑**最多**可能的情况。

![5-6](\img\in-post\machine-learning-fundation\5-6.PNG)

四个点的时候，同样的道理，我们最多可以得到14种不同的直线

![5-7](\img\in-post\machine-learning-fundation\5-7.PNG)

因此，对于平面上的二分类问题，表面上看我们是要从无限多条直线中去选择一条出来，但其实有相当多的直线产生的结果是一样的。这些直线也将同时遇到Bad Data或者不遇到Bad Data。因此，之前我们估计的遭遇Bad Sample的概率是明显被夸大了的，我们的M可以用**Effective number of Lines**，也就是可能的直线的种类数，来替代，这样子，在有无限多直线的情况下，我们也可以也可以做到learning了。

![5-8](\img\in-post\machine-learning-fundation\5-8.PNG)

### Effective Number of Hypothesis

这里我们首先给出一个定义，dichotomy。简单来说，dichotomy就是给定一组输入数据x，任意从$$\cal H$$中选择一个h，来对x进行分类，得到的一个输出就是一个dichotomy。不难得出，一个h必定会对应一个dichotomy，但一个dichotomy可能会对应多个h，我们把一个dichotomy对应的所有h视为一类，那么其实上一节提到的Effective Number of Lines其实就等于不同dichotomy的数量。

那么$$\cal H$$作用与$$\cal D$$最多能产生多少种不同的dichotomy呢，显然，这和$$\cal H$$有关，和数据量N有关，也和具体的输入数据有关。比如上一节中，当N =1时，dichotomy有两种；当N =2时，dichotomy有4种；当N=3时，dichotomy**最多**有8种，但也可能只有6种；当N = 4时，dichotomy最多有14种。这里，我们把这个**最多**的数量成为**成长函数**，显然，当$$\cal H$$固定的情况下，成长函数是关于N的一个函数，而且对于我们的二分类问题来说，一定是小于$$2^N$$的。

![5-9](\img\in-post\machine-learning-fundation\5-9.PNG)
