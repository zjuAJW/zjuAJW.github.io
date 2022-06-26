---
layout:     post
title:      "推荐系统论文精读——Airbnb的Embedding方法"
subtitle:   "KDD2018 Best Paper"
date:       2019-09-09 17:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
    - CTR
	- 推荐系统
---

[**Real-time Personalization using Embeddings for Search Ranking at Airbnb**](https://www.kdd.org/kdd2018/accepted-papers/view/real-time-personalization-using-embeddings-for-search-ranking-at-airbnb)

Airbnb的一篇文章，重点介绍了Airbnb在进行实时排序的过程中采用的embedding方法。

文中分两个部分分别介绍了两种embedding，一是通过用户的click session来生成listing（注意文中的listing实际就是一个个房屋）的embedding，二是通过book session来生成user-type和listing-type的embedding，两种embedding可以看作是分别是用来捕捉用户的短期和长期兴趣。

### Listing Embedding

文中的embedding方法都是基于skip-gram模型，同时采用了负采样的优化方法。

![listing_embedding](\img\in-post\airbnb-embedding\listing.png)

原始的经过负采样的skip-gram模型的损失函数为

![skip-gram](\img\in-post\airbnb-embedding\skip-gram.png)

这个损失函数具体咋来的，可以参考[负采样损失函数计算](https://arxiv.org/pdf/1402.3722.pdf)

文章的一个改进在于，对于存在booking操作的session，不管booking的listing是否在当前中心listing计算的滑动窗口中，都会将其加到最后的损失函数中去，所以，损失函数就变成了

![booking-loss](\img\in-post\airbnb-embedding\booking-loss.png)

另一个改进在于，考虑到短租用户通常都是搜索在同一个目的地，或者同一市场的住房，为了能够学习到同一市场listing的差异，又在损失函数中加入了一组负采样样本，就是在central listing同一市场的listing中进行了随机负采样，最后损失函数就变成了这样：

![marketplace](\img\in-post\airbnb-embedding\marketplace.png)

以上基本就是文章在Listing Embedding上的做法，之后文章还提到了冷启动的问题，对于未在训练集中出现的listing，文章选取了距离该listing最近的，并且同一类型而且价格在同一 price bucket的三个listing，然后取平均，作为该listing的embedding。

### User-type & Listing-type Embedding

之前Listing的embedding更多的是利用用户短期兴趣的推荐，接下来文章希望能将用户的booking信息融入进来，这些信息反映了用户长期的兴趣，以及一定的跨market的相似性。

但是用用户的booking行为的列表来进行embedding会有一些问题：

1. booking行为比click要少非常多，很多用户只有一次booking
2. 要得到比较好的embedding效果，每个listing至少要出现5~10次，但是很多listing只被booking了一次
3. 用户的两次booking之间可能会隔了非常久的时间，这期间用户的很多信息和偏好会发生变化

为了解决以上这些问题，文章提出不使用uid或者listing id来进行embedding，而是采用user-type和listing-type，type的类型由自定义的一些规则来得到。同时，文章将user-type和listing-type放到了一起训练，为的是能够使两者在同一个vector space里，这样就可以根据两者的余弦相似度来为用户进行推荐。

![type-embedding](\img\in-post\airbnb-embedding\type-embedding.png)

具体地，文章拿到的是（user-type，listing-type）对的序列，然后仍然采用skip-gram加负采样的方式，来进行训练，如左图所示。对应的损失函数根据中心的item是user-type还是listing-type分为两个：

![loss-user-type](\img\in-post\airbnb-embedding\loss-user-type.png)

![loss-list-type](\img\in-post\airbnb-embedding\loss-list-type.png)

但其实两个是一样的，只不过中心item不同所以符号不太一样罢了。

*这个做法看起来总是感觉怪怪的，把两个不一样东西放到一起去做embedding？*

booking操作还涉及到host也就是房东的操作，他们可以拒绝用户的租住请求，文章认为这也是一个很重要的信息，所以将“拒绝”这一操作加入到了embedding中，理由是这样可以有效区分一些对用户信息没那么敏感的listing，这样可以降低用户租住时的拒绝率。具体的做法同样是加入了一个新的负采样集合。按照文中的说法，这个负采样集合是booking操作之前的rejection，加入这个负采样集合之后，损失函数变为：

![image-20190911154358844](\img\in-post\airbnb-embedding\loss-user-rejection.png)

![image-20190911154453002](\img\in-post\airbnb-embedding\loss-listing-rejection.png)

### 工程细节

#### Training Listing Embeddings

在训练listing embedding的时候，文章做了一些数据上的处理。

首先，那些停留时间少于30s的click被去掉了；第二，只保留有两个及以上点击的session；第三，booking结尾的这种session进行了5倍的过采样。

训练的时候，embedding每天都是重新训练的，数据会加入新一天的数据，而去掉最老一天的，使得数据维持在一个窗口中，文中说这样取得的效果比增量训练要更好。而且，由于后续使用的大多是embedding的余弦相似度，所以embedding值本身的变化并不会影响后续的使用。

### 特征设计

在进行了embedding之后，文章中并不是直接将embedding用与模型，而是设计了一系列的特征，主要包括以下几个

![feature](\img\in-post\airbnb-embedding\feature.png)

主要的思想就是计算当前的listing和用户好多个历史行为列表中的listing的相似度，然后作为特征。