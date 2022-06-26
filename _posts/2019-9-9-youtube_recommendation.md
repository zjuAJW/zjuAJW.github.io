---
layout:     post
title:      "推荐系统论文精读——Youtube视频推荐"
subtitle:   "Deep Neural Networks for YouTube Recommendations"
date:       2019-09-09 17:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
   - CTR
	- 推荐系统
---

这篇2016年的论文，在现在看来好像已经没有什么新意，但是对于我这种小白来说，还是值得一读的。

首先来看整体的架构

![image-20190909165252924](\img\in-post\youtube-recommendation\structure.png)

从图中可以看到，youtube这篇文章中的推荐系统分为两个阶段，和搜索十分像，可以理解为召回和排序两个阶段，两个阶段分别使用了两个网络模型。

那么首先来看一下召回，也就是candidate generator

![image-20190909194848061](\img\in-post\youtube-recommendation\candidate_generator.png)

整个网络的结构和现在业界流行的十分像，底层是各个特征的embedding， 然后输入到上层的神经网络中。文章将这个问题看做是预测用户下一个观看的视频的问题，所以被当做了一个多分类问题来解，上边采用了一个softmax函数来产生每个候选视频的概率。

这个结构有几个点值得思考。

1. 输入的特征中有一个example age，原文中是这么写的

> We consistently observe that users prefer fresh content, though not at the expense of relevance
>
> The distribution of video popularity is highly non-stationary but the multinomial distribution over the corpus produced by our recommender will reflect the average watch likelihood in the training window of several weeks. To correct for this, we feed the age of the training example as a feature during training. At serving time, this feature is set to zero (or slightly negative) to reflect that the model is making predictions at the very end of the training window.

也就是说，这个特征的引入是为了解决用户对新视频有偏好的问题，如何理解它的作用呢？首先，这个example age的值是什么呢，按照文中的说法应该是训练时间减去样本log的时间，看起来和ctr预估里的position bias有异曲同工之妙。似乎可以理解为，显式地告诉模型，这条样本表示的是用户在哪个时间点的兴趣，而在线上，这个值置为0或者微负，告诉模型我要预测的是用户当前的兴趣。

2. 这里候选的视频可能有上百万个，softmax要为每个候选视频都算一个概率，性能如何保证。

   这个问题文中其实参考了word2vec里的做法，采用了负采用的做法，或者也可以尝试hierarchical softmax，但是文中说hierarchical softmax并没有取得预期的效果。

3. 为什么模型在线上的时候用的是一个左边最近邻的模型，而不是直接用的右边的softmax呢。

   softmax复杂度还是太高了，要算百万的概率，还要排序，所以线上使用的LSH的方法，也就是左边的结构，这样复杂度可以降到logn甚至常数级别。

召回之后，就是排序阶段了，模型如下

![image-20190909221959857](\img\in-post\youtube-recommendation\ranking.png)

排序阶段的模型有了更多更丰富的特征输入，但最关键的还是上层，在训练和serving的时候，又分开了，训练时采用的是加权的逻辑回归，而serving的时候又输出了一个奇怪的东西$$e^{Wx+b}$$，文中说到这个东西预测的是用户的观看时长，一脸懵逼。后来网上看了各路大神的解读，才略有理解，这里就直接放个链接[揭开YouTube深度推荐系统模型Serving之谜](https://zhuanlan.zhihu.com/p/61827629)。

理解这个问题还是要从逻辑回归来看。逻辑回归我们都知道是线性回归套了一个sigmoid函数，而通过推到可以发现，$$Wx=ln(Odds)=ln(\frac{p}{1-p})$$，这也就是文中Odds的来源，这里的p其实就是真正的逻辑回归输出的概率。

而文中采用的是用户观看时间加权的逻辑回归，我们简单推导一下，假设对于某个视频而言我们的正样本有k个，负样本一共x个，那么$$p=\frac{k}{k+x}$$，而$$Odds=\frac{k/k+x}{1-k/k+x}=\frac{k}{x}$$，如果我们采用了用户观看时间$$T_i$$作为正样本的加权，而负样本取1，可以看到$$Odds=\frac{\sum T_i/\sum T_i+x}{1-\sum T_i/\sum T_i+x}=\frac{\sum T_i}{x}$$，由于样本的点击率通常都很小，也就是p一般很小，所以可以认为负样本占了大多数，x基本等于样本数N，所以最后Odds基本就是每个视频的平均观看时间，所以这也就是为什么最后模型是输出了$$e^{Wx+b}$$。

总体看来，整篇文章还是很工程导向的，有很多实践的细节值得去深入思考。

