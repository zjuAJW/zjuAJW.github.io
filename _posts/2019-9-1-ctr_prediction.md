---
layout:     post
title:      "CTR预估相关论文记录"
subtitle:   "论文阅读计划"
date:       2019-09-01 15:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
    - CTR
---

## Learning Piece-wise Linear Models from Large Scale Data for Ad Click Prediction

这篇文章是阿里盖坤团队的MLR（Mixer Logistic Regression），虽然论文里好像起了个更高大上的名字。

文章的出发点就是，传统的LR模型对于广告这种非线性场景的拟合能力是不够的，业界需要一个能够捕捉非线性性质同时又能适合于大规模稀疏特征的模型，于是MLR就这样诞生了。

MLR的主要公式如下：

$$p(y=1\vert x)=g(\sum^m_{j=1}\sigma(u_j^T x)\eta(w_j^Tx) )$$

其中$$\sigma$$将”特征空间“分成了多份，而$$\eta$$是每个空间下的拟合函数。文章中说到通常情况下两个函数分别取softmax和sigmoid函数，而$$g$$取恒等映射函数即$$g(x)=x$$，所以上式会变成：

$$p(y=1\vert x)=\sum^m_{i=1}\frac{exp(u_i^T)}{\sum_{j=1}^mexp(u_j^Tx)}*\frac{1}{1+exp(-w_i^Tx)} $$

个人理解，MLP就是把整个样本空间分成了m份，当一个样本来了之后，先用$$\sigma$$函数给样本打个分，看它属于哪个空间，（也就是所谓的结构先验），然后再在这个空间上，用$$\eta$$函数来进行模型的拟合（比如LR）。

或者从公式的形式上来看，感觉就是将样本喂给了m个LR模型，然后进行了一个加权求和。

模型的损失函数是传统的交叉熵加上两个正则化项

$$arg min_\Theta f(\Theta)=loss(\Theta)+\lambda\Vert \Theta\Vert_{2,1}+\beta\Vert\Theta\Vert_1$$

$$loss(\Theta)=-\sum_{t=1}^n[y_tlog(p(y_t=1\vert x_t,\Theta))+(1-y_t)log(p(y_t=0\vert x_t,\Theta))]$$

其中的$$L_{2,1}$$正则化项$$\Vert\Theta\Vert_{2,1}=\sum_{i=1}^d\sqrt{\sum_{j=1}^{2m}{\theta_{ij}^2}}$$目的在于，使得某些空间下参数都尽可能为0，而$$L_{1}$$正则化项$$\Vert\Theta\Vert_1=\sum_{ij}\vert\theta_{ij}\vert$$就是使得每个空间下单独进行逻辑回归时，参数尽可能为0。

## Deep Interest Network for Click-Through Rate Prediction

### 背景

通常CTR预估的深度学习模型都是典型的Embedding&MLP模型，高纬度的稀疏特征首先通过embedding层映射到一个低维固定长度的向量上，然后再输入到多层神经网络中。在电商场景中，用户的兴趣是多种多样的（diversity），将用户的多样兴趣embedding到一个固定长度的向量中，可能会限制了模型的表达能力。如果要更好地刻画用户的多样兴趣，可以加大embedding向量的维度，但这样又会加大参数量，导致过拟合风险。

另一方面，影响用户点击行为的，可能只是用户历史行为中的一部分，比如，一个人点击了一个泳镜，更大程度上是因为他上周买了一套泳衣，而不是因为他上周买了一个鞋子。所以，我们也没有必要将用户所有的历史兴趣都embedding到向量中，而是只取其中一部分，或者给这部分更大的权重。

### Embedding&MLP

![image-20190901174302410](\img\in-post\ctr\mlp.png)

在Embedding&MLP模型中，用户行为的建模将所有用户点击历史goods的信息进行了pooling，得到了一个固定维度的向量。

### DIN

![image-20190901175248017](\img\in-post\ctr\din.png)

DIN的思路就是，对于用户历史行为，并不是每个goods都一样看待，而是通过一个local activation unit来计算每个goods在pooling中的权重。Local activation unit的输入为goods的embedding向量和广告的embedding向量。直观上理解，就是和广告相关性越强的商品，权重应该会越大，这个和attention的思想很像