---
layout:     post
title:      "机器学习基石笔记(1)"
subtitle:   "VC维理论"
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



## Lecture 5 Training versus Testing 

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

因此，对于平面上的二分类问题，表面上看我们是要从无限多条直线中去选择一条出来，但其实有相当多的直线产生的结果是一样的。这些直线也将同时遇到Bad Data或者不遇到Bad Data。因此，之前我们估计的遭遇Bad Sample的概率是明显被夸大了的，我们的M可以用**Effective number of Lines**，也就是可能的直线的种类数，来替代，这样子，在有无限多直线的情况下，我们似乎也可以做到learning了。

![5-8](\img\in-post\machine-learning-fundation\5-8.PNG)

### Effective Number of Hypothesis

这里我们首先给出一个定义，dichotomy。简单来说，dichotomy就是给定一组输入数据x，任意从$$\cal H$$中选择一个h，来对x进行分类，得到的一个输出就是一个dichotomy。不难得出，一个h必定会对应一个dichotomy，但一个dichotomy可能会对应多个h，我们把一个dichotomy对应的所有h视为一类，那么其实上一节提到的Effective Number of Lines其实就等于不同dichotomy的数量。

那么$$\cal H$$作用与$$\cal D$$最多能产生多少种不同的dichotomy呢，显然，这和$$\cal H$$有关，和数据量N有关，也和具体的输入数据有关。比如上一节中，当N =1时，dichotomy有两种；当N =2时，dichotomy有4种；当N=3时，dichotomy**最多**有8种，但也可能只有6种；当N = 4时，dichotomy最多有14种。这里，我们把这个**最多**的数量称为**成长函数(Growth function)，$$m_{\cal H}(N)$$**，显然，当$$\cal H$$固定的情况下，成长函数是关于N的一个函数，而且对于我们的二分类问题来说，一定是小于等于$$2^N$$的。如果$$m_{\cal H}(N) =2^N$$，那么我们称这N个输入被$$\cal H$$**shatter**掉了

![5-9](\img\in-post\machine-learning-fundation\5-9.PNG)

下面给出几个成长函数的例子，这里只给出结果，具体的推导可以看一下PPT或者视频哈

![5-10](\img\in-post\machine-learning-fundation\5-10.PNG)

OK，那像我们上一节对平面线形二分类的讨论一样，对于一个一般化的问题我们可以用这个成长函数**$$m_{\cal H}(N)$$**来替代公式里的M

![5-11](\img\in-post\machine-learning-fundation\5-11.PNG)

那么，可以看到，如果$$m_{\cal H}(N)$$是一个多项式的话，它的增长速度是没有后边的指数项的衰减速度快的，所以不等式的右边随着N的增大可以衰减到接近于０，这对我们来说是好的。之前列出的Positive rays，positive intervals等等，它们的$$m_{\cal H}(N)$$可以很容易得到，但是有些$$\cal H$$的成长函数则没有那么容易得到，比如我们一直在讨论的线性二分类问题。那么对于这样的$$\cal H$$，我们怎么办呢？

### Break Point

虽然现在我们不太容易得到2D perceptron成长函数的解析形式，不知道它是不是多项式型的，但是我们知道在N为某些值时，它是小于$$2^N$$的（也就是不能被shatter），我们称这样的N为**Break Point**。比如，2D perceptron在N=4的时候，是一个Break Point，那么其实在N > 4的时候，所有的N都是Break Point，所以后文我们的Break Point都是指最小的那个。

同样的，对于我们之前讨论的几个问题，都可以得到它们的Break Point

![5-11](\img\in-post\machine-learning-fundation\5-12.PNG)

从这里，我们似乎可以发现一线希望，对于有Break Point的$$\cal H$$来说，好像他们的$$m_{\cal H}(N)$$都是多项式的，而且多项式的最高次是$$N^{k-1}$$。那么，这个结论是不是对的呢，且听下回分解。

## Lecture 6 Theory of Generalization

上节说到，对于有Break Point的$$\cal H$$，我们似乎可以用这个break point来做一点事情。所以这一节，我们关注的重点集中在这个break point上。

### Restriction of Break Point

先来看一个例子。我们知道，不管我的$$\cal H$$是什么，如果它最小的break point为2（$$k=2$$），那么说明$$m_{\cal H}(2)<4$$，也就是说$$m_{\cal H}(2)$$最大也就是3。那么N=3的时候呢？从break point的定义来说，$$k=2$$意味着，任意两个数据点都不能被shatter，还记得shatter的概念吗？意思就是我产生的dichotomies不能完全包含任何2个数据点所有的排列组合。 我们一个一个地来添加dichotomy，看看N=3的时候dichotomy的数量会有什么限制。

![6-1](\img\in-post\machine-learning-fundation\6-1.PNG)

![6-2](\img\in-post\machine-learning-fundation\6-2.PNG)

![6-3](\img\in-post\machine-learning-fundation\6-3.PNG)

当有三个dichotomy的时候，任意两个数据点都不能被shatter。但当加入第四个的时候，就可能会有问题了。

![6-4](\img\in-post\machine-learning-fundation\6-4.PNG)

这时候可以发现，$$x_2$$和$$x_3$$全部可能的排列组合都出现了，也就是被shatter了，这是不符合break point为2这个条件的，所以我们不能产生这个dichotomy。但是我可以换另一种dichotomy，就不会有问题。

![6-5](\img\in-post\machine-learning-fundation\6-5.PNG)

当我们继续添加第五个dichotomy的时候，你会发现，无论添加什么，都会导致有两个数据点被shatter的情况，所以，在N=3， k=2的情况下，可能产生的dichotomy的数量，最多最多只有4个。可见，这个break point对于$$m_{\cal H}(N)$$可能的最大值还是有一定的限制作用的。也就是说，虽然我们不能直接得到$$m_{\cal H}(N)$$的表达式，但是在知道break point的情况下，我们似乎能够得到$$m_{\cal H}(N)$$的一个上界。我们记这个上界为$$B(N, k)$$，称它为Bounding function。上边的讨论显示，$$B(2,2)=3$$，而$$B(3,2)=4$$。

### Bounding Function

那么，美妙的事情就发生了，我们不能得到$$m_{\cal H}(N)$$，但我们可以去研究这个Bounding Function，只要$$B(N,k)$$是多项式的，那么我们的$$m_{\cal H}(N)$$就也是多项式的了。

不难知道：

- $$k=1$$时，1个点(2种排列组合)都没有办法shatter，因此$$B(N,1)$$恒等于1。
- $$N<k$$时，$$\cal H$$一定能shatter掉$$N$$个点，因此它产生的dichotomies的种类等于这$$N$$个点所有的排列组合数$$2^N$$。
- $$k=N$$时，从$$N^2$$个排列组合中移除掉一个，剩下的都可以作为dichotomies，因此它产生的dichotomies的数量**最多最多**可以是$$2^{N−1}$$。

加上之前我们得到的结论$$B(3,2)=4$$，我们可以得到下面这个表

![6-6](\img\in-post\machine-learning-fundation\6-6.PNG)

接下来的问题就是，整个表的 左下这部分的值是多少？比如$$B(4,3)$$的值，我们能够根据其他值来推导出来吗？

算不出来我们可以穷举一下嘛！$$N=4$$的情况下，各种各样dichotomy的组合一共也就$$2^{2^4}$$种。经过一番穷举之后，我们发现，$$B(4, 3)=11$$，具体的dichotomy的组合如下图所示。

![6-7](\img\in-post\machine-learning-fundation\6-7.PNG)

我们把结果做一个简单的顺序变化，变成右图，其中橙色部分是成对出现的，紫色部分是单独出现的。我们先不看$$x_4$$，橙色部分因此可以分为两部分，写为$$2\alpha$$，紫色部分写为$$\beta$$， 那么$$B(4,3)=11=2\alpha + \beta$$。

根据定义，$$B(4,3)$$是不允许任意三个点被shatter的。我们拿出一个$$\alpha$$和$$\beta$$组成的这一部分，可以看作是对三个点的dichotomies，而这三个点是不能被shatter的，所以可以得到$$\alpha + \beta \le B(3,3)$$。

![6-8](\img\in-post\machine-learning-fundation\6-8.PNG)

对于剩下的另一部分$$\alpha$$，它们的$$x_4$$是成对的，所以，它们中的任意2个点都不能被shatter，否则，加上$$x_4$$，就会有三个点被shatter了，所以$$\alpha \le B(3,2)$$。

![6-9](\img\in-post\machine-learning-fundation\6-9.PNG)

将两部分合起来，就可以得到$$B(4,3)=2\alpha + \beta \le B(3,3)+B(3,2)$$

这样一来，我们就可以把之前那张表填完整了

![6-9](\img\in-post\machine-learning-fundation\6-10.PNG)

最终，通过数学归纳法，其实可以证明，$$B(N,k)\le \sum_{i=0}^{k-1}\binom {N}{i}$$

当$$k=1$$时不等式恒成立，因此只要讨论$$k\ge 2$$的情形。$$N=1$$时，不等式成立，假设$$N\le N_o$$时对于所有的$$k$$不等式都成立，则我们需要证明当$$N=N_o+1$$时，不等式也成立。根据前面得到的结论，有： 


$$
\begin{aligned}
B(N_{o}+1,k) &\leq B(N_{o},k) + B(N_{o},k-1) \\\
&\leq \sum_{i=0}^{k-1}\binom{N_{o}}{i}+\sum_{i=0}^{k-2}\binom{N_{o}}{i} \\\
&=1+\sum_{i=1}^{k-1}\binom{N_{o}}{i}+\sum_{i=1}^{k-1}\binom{N_{o}}{i-1} \\\
&=1+\sum_{i=1}^{k-1}[\binom{N_{o}}{i}+\binom{N_{o}}{i-1}] \\\
&=1+\sum_{i=1}^{k-1}\binom{N_{o}+1}{i}=\sum_{i=0}^{k-1}\binom{N_{o}+1}{i}
\end{aligned}
$$


因此当$$N=N_o+1$$时，不等式也成立。 

这样，我们成长函数的上界Bounding Function被bound住了，我们的成长函数自然也就被bound住，因此对于break point为k的成长函数而言，有：$$m_{\cal {H}}\le \sum_{i=0}^{k-1}\binom {N}{i}$$。这个公式的右边其实是一个最高次为$$k-1$$的多项式，所以对于我们最一开始讨论的2D perceptron而言，它的break point是4，所以它的成长函数会被$$B(N,4)$$给bound住。
$$
m_{\mathcal{H}}\leq \sum_{i=0}^{4-1}\binom {N}{i}=\frac{1}{6}N^3+\frac{5}{6}N+1
$$
所以，lecture 5和lecture 6差不多帮我们解决了$$\cal H$$无限大的问题：虽然$$\cal H$$看上去是无限大的，但实际有效的h是有限的，我们用成长函数来描述有效的h的数量，并通过break point去寻找它的一个上界，从而将成长函数bound在一个多项式的成长速度上。

## Lecture 7 The VC Dimension

上一节我们说到，我们可以利用有限的$$m_{\cal H}(N)$$来替代无限大的M，得到$$\cal H$$遇到bad data的概率上界：


$$
\mathbb{P}_\mathcal{D}[BAD\ D]\leq 2m_{\mathcal{H}}(N)\cdot exp(-2\epsilon ^2N)
$$
其中$$\mathbb{P}_{\cal D}[BAD\ D]$$是所有有效的方程(Effective Hypotheses)遇到Bad Sample的联合概率，即$$\cal H$$中存在一个方程遇上bad sample，则说$$\cal H$$遇上bad sample。用更加精准的数学符号来表示上面的不等式： 
$$
\mathbb{P}[\exists h \in \mathcal{H}\text{ s.t. } |E_{in}(h)-E_{out}(h)|\gt \epsilon]\leq 2m_{\mathcal{H}}(N)\cdot exp(-2\epsilon ^2N)
$$


但事实上上面的不等式是不严谨的，为什么呢？$$m_{\cal H}(N)$$描述的是$$\cal H$$作用于数据量为$$N$$的资料$$D$$，有效的方程数，因此$$H$$当中每一个$$h$$作用于$$D$$都能算出一个$$E_{in}(h)$$来，一共能有$$m_{\cal H}(N)$$个不同的$$E_{in}(h)$$，是一个有限的数。但在out of sample的世界里(总体)，往往存在无限多个点，平面中任意一条直线，随便转一转动一动，就能产生一个不同的$$E_{out}(h)$$来。$$E_{in}(h)$$的可能取值是有限个的，而$$E_{out}(h)$$的可能取值是无限的，无法直接套用union bound，我们得先把上面那个无限多种可能的$$E_{out}(h)$$换掉。那么如何把$$E_{out}(h)$$变成有限个呢？ 

这里我想偷个懒了，因为整个证明实在是太需要技巧性了，说实话我一个数学渣渣也没怎么看下去，所以这里直接给出一个结论，就是**VC-Bound**:



$$
\begin{aligned}
\mathbb{P}[BAD] &= \mathbb{P}[\exists h \in \mathcal{H}\text{ s.t. } |E_{in}(h)-E_{out}(h)|\gt \epsilon] \\\
&\leq 4m_{\mathcal{H}}(2N)exp(-\frac{1}{8}\epsilon^2N）
\end{aligned}
$$



所以，有了以上的这些讨论，我们可以得到，如果我们的$$m_{\cal H}(N)$$有breaks at k，并且N足够大，那么我们就可以保证$$E_{in}=E_{out}$$，如果我们的算法$$\cal A$$又能够选择到一个$$E_{in}$$很小的h的话，我们就可以做到学习啦：-）！

### VC dimension

VC dimension的定义其实就是最大的非break point值，也就是使得$$m_{\cal H}(N)=2^N$$的最大的N的值。我们记VC dimension为$$d_{vc}$$

![7-1](\img\in-post\machine-learning-fundation\7-1.PNG)

下图是我们之前讨论过的一些$$\cal H$$的VC维

![7-2](\img\in-post\machine-learning-fundation\7-2.PNG)

但这里我们要注意，通过成长函数来求VC维没什么太大的意义，因为我们通常很难得到某个$$\cal H$$的成长函数，而是希望能够得到VC维，从而来用$$N_{d_{vc}}$$bound住成长函数。那么如何来求$$d_{vc}$$呢？

从物理意义上来讲，VC维可以近似看作是$$\cal H$$的自由度大小，或者说参数的个数，对于上面所说的几个$$\cal H$$a都是成立的。

回到我们机器学习的两个关键问题，$$E_{in}$$和$$E_{out}$$是否接近，以及$$E_{in}$$能否足够小。那么我们的$$d_{vc}$$是如何影响这两个问题的呢？

![7-3](\img\in-post\machine-learning-fundation\7-3.PNG)

和之前分析M的大小类似，这两个条件似乎总是相抵触的，所以我们需要找到一个平衡点。

### VC维与模型复杂度

$$
\begin{aligned}
\mathbb{P}[\exists h \in \mathcal{H}\text{ s.t. } |E_{in}(h)-E_{out}(h)|\gt \epsilon] \leq 4m_{\mathcal{H}}(2N)exp(-\frac{1}{8}\epsilon^2N)
\end{aligned}
$$

回忆我们VC Bound的不等式，我们另不等式右边为$$\delta$$，也就是说坏事发生的概率小于$$\delta$$，那么好事发生的概率就会大于$$1-\delta$$。这里我们做一个简单的数学推导，可以得到$$\epsilon$$的表达式。

![7-4](\img\in-post\machine-learning-fundation\7-4.PNG)

因此$$E_{in}$$和$$E_{out}$$又会有下面的关系

![7-5](\img\in-post\machine-learning-fundation\7-5.PNG)

我们把根号那一项看作是模型的复杂度，可以看到，$$d_{vc}$$越大，模型的复杂度越高，但是$$d_{vc}$$越大，$$E_{in}$$会越小，所以，我们需要在两者之间找到一个平衡点，找到一个合适的$$d_{vc}$$

![7-6](\img\in-post\machine-learning-fundation\7-6.PNG)



反过来，如果我们需要使用$$d_{vc}=3$$这种复杂程度的模型，并且想保证$$\epsilon=0.1$$，置信度$$1-\delta=90%$$，我们也可以通过VC Bound来求得大致需要的数据量$$N$$。通过简单的计算可以得到理论上，我们需要$$Napprox 10,000d_{vc}$$笔数据，但VC Bound事实上是一个极为宽松的bound，因为它对于任何演算法$$\cal A$$，任何分布的数据，任何目标函数$$f$$都成立，所以经验上，常常认为$$N\approx 10d_{vc}$$就可以有不错的结果。 

## Lecture 8 Noise and Error    

### 确定性 v.s. 概率性

![8-1](\img\in-post\machine-learning-fundation\8-1.PNG)

回忆我们在课程一开始就建立的机器学习的整个框架，我们是假设有一个确定的目标函数$$f$$，而我们的目标是找到一个$$g$$，使得$$g$$尽可能和$$f$$接近。在这个过程中，我们是默认数据$$D$$是由$$f$$产生的，但是实际上，我们的数据通常是有噪声存在的，主要表现在三个方面：

* noise in y：本来应该是圈圈的样本，结果标成了叉叉
* noise in y：相同的$$x$$，结果标记不同
* noise in x：$$x$$本身存在一些记录错误

在确定的$$f$$上加了一些随机性的东西，就变成了一个概率性的问题，我们的target function也就变成了一个target distribution。

![8-2](\img\in-post\machine-learning-fundation\8-2.PNG)

对于某个样本$$x$$，理想状态下，应该有$$y=f(x)=+1$$，但由于某种noise的存在，该noise会有30%的概率转换$$f(x)$$的结果(把+1变成-1或把-1变成+1)。因此在$$\cal D$$中，该样本有70%的概率表现出$$y=+1$$，30%的概率表现出$$y=−1$$. 

![8-3](\img\in-post\machine-learning-fundation\8-3.PNG)

### 误差的衡量

误差的定义会影响训练过程，所以我们要根据目标定义好误差函数。这里需要注意的就是不同的错误，代价可能是不同的，因此，在误差的衡量上可以有不同的权重。

这一节很多知识这里就不详细记录了，可以参考讲义。