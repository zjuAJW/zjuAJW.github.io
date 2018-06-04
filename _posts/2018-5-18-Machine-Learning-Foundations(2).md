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
&=\frac{1}{N}(N-(d+1))||noise||^2\\\

\end{aligned}
$$
至于这里这个$$trace(I-H)=N-(d+1)$$的结论，课程里也没有详细说，这里也就不给出证明了。

所以我们可以得到

![9-3](\img\in-post\machine-learning-fundation\9-3.PNG)

这个$$\bar E_{out}$$是怎么来的，课程里也没有说，这里就当结论给出好了。这两个公式说明了啥呢，我们把它们的图像画出来

![9-4](\img\in-post\machine-learning-fundation\9-4.PNG)

可以看到它们都是趋向于$$\sigma ^2$$（noise level）的，也就说明$$E_{in}$$和$$E_{out}$$的误差是被大概$$\frac{2(d+1)}{N}$$bound住的，这有点类似于之前说的VC bound，也是提供了一个学习可行性的保障。

