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

![9-1](E:\zjuAJW.github.io\img\in-post\machine-learning-fundation\9-1.PNG)

所以，如果我们把结果写成$$\hat y=X\omega_{lin}=X(X^TX)^{-1}X^Ty=Hy$$，这里的H又被叫做Hat Matrix，它的作用就是做了投影这个动作。