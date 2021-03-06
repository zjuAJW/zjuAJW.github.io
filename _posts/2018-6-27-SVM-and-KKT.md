---
layout:     post
title:      "SVM以及KKT条件的理解"
subtitle:   "优化还是很重要啊"
date:       2018-06-27 15:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
    - 机器学习
    - 算法
---

SVM一直是机器学习方法里比较难以理解的一个，这篇文章不是SVM的入门教程，只是自己在学习SVM过程中的一些理解和收获，想从头了解SVM还是要看书或者网上的其他博客。

## SVM

SVM其实也没有多么高深，它和感知器、逻辑回归等线性分类器一样，归根到底它还是找一条线来把不同类别的样本分开来，区别就在于直线好坏的判断标准。之前我们学过感知器，直线的好坏就是看有么有分类错误或者分类错误的数量尽可能少；逻辑回归，直线的好坏通过logloss损失函数来判断，越小就越好，通过梯度下降等方法就可以求解。那么SVM呢，从几何的直观上来看，就是找到一条直线，能够使得这条线到两个类别的样本的距离能尽可能大。

![1](\img\in-post\SVM\1.png)

如图，能够分开平面的直线是有无数条的，那么哪一条是最优的呢，根据SVM的条件，是使得Gap最大的那条，也就是红色的这条。

以上是几何上的直观解释，如何表达成数学上的式子呢？其实也很简单，我们把Gap的值算出来然后让它最大就好了，其实也就是用了一个点到直线的距离公式，这是初高中的数学知识。

假设我们的直线是$$\omega x+b$$，那么某个点到直线的距离就是$$\gamma_i = \frac{\vert \omega x+b\vert}{\Vert \omega\Vert}$$，在我们的直线分类正确的前提下，可以写成$$\gamma_i = \frac{y_i(\omega x_i+b)}{\Vert \omega\Vert}$$,我们记所有点中，距离直线距离最小的那个值为$$\gamma$$，即$$\gamma = min \frac{y_i(\omega x_i+b)}{\Vert \omega\Vert} i = 1...n$$，那我们的目标函数就是使这个距离最大。

$$max_{\omega,b}\ \ \gamma $$

$$s.t. \gamma_i \ge \gamma \ \ \ \ i=1..n$$

注意到$$y_i(\omega x_i+b)$$的值是受$$\omega$$和$$b$$的大小的影响的（这个值其实就是函数间隔，而除了$$\Vert \omega \Vert$$的是几何间隔），只要它们成倍的放大或者缩小，$$y_i(\omega x_i+b)$$的值也会成倍的放大或者缩小，所以我们可以把$$\gamma$$对应的$$y_i(\omega x_i+b)$$的值标准化为1，这不影响我们最优化问题的求解，所以，我们的问题最就成了

$$max_{\omega,b}\ \frac{1}{\Vert \omega \Vert} $$

$$s.t. y_i(\omega x_i+b)\ge 1$$

而这个问题和下面这个问题是等价的

$$max_{\omega,b}\ \frac{1}{2}\Vert\omega \Vert^2 $$

$$s.t. y_i(\omega x_i+b)\ge 1$$

这也就是SVM最基本的数学表述，而求解上述的这个凸二次规划问题是很方便的，有很多现成的工具包，所以，问题似乎到此为止就可以结束了。但是，我们可以有更加高效的方法来求解这个问题，这也就是为什么会有对偶问题、KKT条件等这一系列后续的原因

## 对偶问题

通过拉格朗日乘子法，我们可以将上述问题转化为

$$L(\omega, b, \alpha) = \frac{1}{2}\Vert\omega\Vert^2-\sum_{i=1}^{n}\alpha_i(y_i(\omega x_i+b)-1)$$

那么原问题的对偶问题就是

$$max_\alpha min_{\omega, b}\ \ L(\omega, b,\alpha)$$

先求解里边的min问题，L分别对$$\omega$$和$$b$$求偏导，可以得到

![2](\img\in-post\SVM\2.PNG)

带回到原式子，可以得到关于$$\alpha$$的优化问题：

![3](\img\in-post\SVM\3.PNG)

求解出$$\alpha$$后，我们就可以算出$$\omega$$和$$b$$，我们的直线方程也就出来啦

![4](\img\in-post\SVM\4.PNG)

当然，由于我们的原始问题中是存在不等式约束的，所以，以上推导都要满足KKT条件。

关于KKT条件，这里先抛掉SVM，单独看一下这个KKT条件是什么。

## KKT条件

我们先从最基本的只有等式约束的最优化问题开始，考虑问题$$min f(x) \ s.t. h(x) = 0$$

我们知道，这种问题用拉格朗日乘子法可以很好的解决，参照下图

![5](\img\in-post\SVM\5.PNG)

在等式约束条件下，极值点必定在约束条件和等高线相切的地方（假设为$$x^*$$）取到，这时约束条件和目标函数的梯度是共线的， 也就是说$$\nabla f(x^*) + \lambda \nabla h(x^*) = 0$$，我们成为拉格朗日条件。所以，为了方便，我们定义了一个拉格朗日函数$$L(x, \lambda)=f(x)+\lambda h(x)$$，令其偏导等于0，就得到了拉格朗日条件。

那么对于不等式约束来说，其实如果只有一个，那么和一个等式约束是一样的，极值点还是切点，只是可行域扩大了。

![6](\img\in-post\SVM\6.PNG)

当多个不等式起作用的时候，麻烦就来了。考虑以下问题：

$$min f(x)$$

$$s.t.\  g_1(x) \le0,\ g_2(x)\le 0$$

![7](\img\in-post\SVM\7.PNG)

这时极值点不再是切点，这时候就需要KKT条件了。这里的“条件”是指：某一个点它如果是最小值点的话，就必须满足这个条件（在含不等式约束的优化问题里），这是个必要条件。

对于这个问题而言，KKT条件为：

1. $$\mu_1\ge 0, \mu_2\ge 0$$
2. $$\nabla f(x^*)+\mu_1\nabla g_1(x^*)+\mu_2\nabla g_2(x^*)=0$$
3. $$\mu_1g_1(x^*)+\mu_2g_2(x^*)=0$$

其中的$$\mu_1,\mu_2$$为KKT乘子，可以认为是和拉格朗日乘子差不多的东西。

下面看看这些个奇怪的条件是怎么来的。首先，我们从上一个简单的问题中可以得到，约束曲线的梯度方向与曲线垂直，我在上图画出了两条约束曲线的负梯度方向（绿色箭头）和等高线的负梯度方向（红色箭头）。如果这个顶点是满足约束的最小值点，那么该点处（红点），红色箭头一定在两个绿色箭头之间(可以参考[浅谈最优化问题的KKT条件 - 力学渣的文章 - 知乎 ](https://zhuanlan.zhihu.com/p/26514613 )），即$$\nabla f(x^*)$$能被$$-\nabla g_1(x^*)$$和$$-\nabla g_2(x^*)$$线性表出($$\nabla f(x^*)=-\mu_1\nabla g_1(x^*)-\mu_2\nabla g_2(x^*)=0$$)，且系数必非负($$\mu_1\ge 0, \mu_2\ge 0$$)，这就是KKT条件中的前两个。

然而有时候，不等式约束是不起作用的，比如

![8](\img\in-post\SVM\8.PNG)

这里的$$g_3(x)$$其实是没有用的，因为最小值不在$$g_3(x)=0$$上，它对解其实没有贡献。这时我们的KKT条件1和2为：

1. $$\mu_1\ge 0, \mu_2\ge 0， \mu_3\ge 0$$
2. $$\nabla f(x^*)+\mu_1\nabla g_1(x^*)+\mu_2\nabla g_2(x^*)+\mu_3\nabla g_3(x^*)=0$$

这里条件2中的$$\mu_3\nabla g_3(x^*)$$这一项就很烦人了，因为它跟“红色箭头”没啥关系，所以我们加上了条件3：

3. $$\mu_1g_1(x^*) +\mu_2g_2(x^*) +\mu_3g_3(x^*) =0 $$

因为$$g_1(x^*)=0,g_2(x^*)=0$$，所以前两项等于0，第三项的系数$$\mu_3=0$$。

总结来说，就是起作用的约束条件，其KKT乘子不为0；而不起作用的约束条件，其KKT乘子为0。

同样的，如果我们也定义一个拉格朗日函数，再令偏导等于0，就可以得到KKT条件中的第二个了。

## KKT条件与支持向量

那我们再回头来看看SVM中的KKT条件。在SVM中，KKT条件可以写成

1. $$\alpha_i\ge 0$$（KKT乘子要大于等于0）
2. $$\sum \alpha_i (y_i(\omega x_i+b)-1)=0$$

这其中第2条跟我们”支持向量”的概念是分不开的。在SVM中，$$y_i(\omega x_i+b)-1=0$$的点是哪些呢？就是我们距离直线距离最近的点，所以，有KKT条件，其实只有这几个点的$$\alpha$$是不为0的。再观察我们之前得到的SVM的解的形式：

![4](\img\in-post\SVM\4.PNG)

其实，最终我们的解只与这些支持向量有关，与其他的数据点没有关系。

## 核函数

SVM另一点比较重要的就是核函数的问题。之前讨论的都是线性SVM，对于非线性可分的数据，我们可以将数据映射到高维空间，然后再进行线性分类。但是问题是，根据之前我们得到的公式，我们需要计算映射后数据点之间的内积。这个内积是不好计算的，因为映射后的空间维度可能很大，甚至是无穷维。所以，我们引入了核函数。

核函数的作用就是，样本$$x_i$$和$$x_j$$在映射后空间的内积，与它们在原始空间内通过核函数$$\kappa(x_i, x_j)$$计算得到的结果是一样的，这样我们就不必去直接计算高维空间的内积，而是采用核函数来简化计算。

这看起来很美妙，如果有线性不可分的数据，我们只需要找到一个合适的映射，将其映射到一个线性可分的空间中，然后通过这个映射，得出对应的核函数，就可以了。但是，这个过程其实是很困难的，且不说从映射得到对应的核函数就很难，光是找到一个合适的映射可能都不是那么容易办到。

所以，我们在实际使用SVM的时候，都是直接选一个核函数就来用了，然而我们并不知道这个核函数对应的映射到底会把数据映射成啥样子，所以，核函数的选择是SVM训练过程中一个非常重要的参数。

当然，这里有一个问题要注意，并不是所有的二元函数都可以作为核函数来使用，是有一定的要求的。有以下定理：

![9](\img\in-post\SVM\9.PNG)

当然了，给我们一个函数，让我们来判断它是不是核函数其实也挺困难的，所以我们基本上是采用一些常用的核函数，比如高斯核、多项式核等。

### 高斯核函数

这里单独拿出高斯核函数来说一说，因为高斯核是我们用的比较多的，而且它有一个特点，就是会把数据映射到无穷维。

高斯核的形式为：$$\kappa(x_i, x_j)=exp(-\frac{\Vert x_i-x_j\Vert^2}{2\sigma^2})$$,那么它对应的映射是什么样子的呢，参考下图：

![9](\img\in-post\SVM\10.PNG)

可以看到，确实是映射到了无穷维，当然图中没有加上$$\sigma$$，但是这个$$\sigma$$是很重要的一个参数。对于高斯核而言，如果$$\sigma$$选得很大的话，高次特征上的权重实际上衰减得非常快，所以实际上（数值上近似一下）相当于一个低维的子空间；反过来，如果$$\sigma$$选得很小，则可以将任意的数据映射为线性可分——当然，这并不一定是好事，因为随之而来的可能是非常严重的过拟合问题。



## 参考资料

[如何通俗地讲解对偶问题？尤其是拉格朗日对偶lagrangian duality？ - 彭一洋的回答 - 知乎](https://www.zhihu.com/question/58584814/answer/159863739)

[Free Mind 支持向量机](http://blog.pluskid.org/?p=682)