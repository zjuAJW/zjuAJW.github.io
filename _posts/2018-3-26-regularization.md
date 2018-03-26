---
layout:     post
title:      "关于正则化"
subtitle:   "多种角度的理解"
date:       2018-03-26 15:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
    - 机器学习
    - 数据挖掘
---

正则化是机器学习里经常要提到的一个问题，它最大的作用是限制了模型的复杂度，从而降低了过拟合的风险。道理想想是简单的，就是在损失函数后边加了一个与模型结构或者参数相关的惩罚项，常见的有L1正则和L2正则。但是为什么加了之后就可以防止过拟合了？而且为什么L1正则更容易得到稀疏解？

## 理解正则化的两个角度

### 从优化的角度

这个角度是最直接也是被说的最多的，下面这张图我想试图了解过正则化的人已经看过很多遍了

![L1 and L2](\img\in-post\regularization\L1_and_L2.jpg)

正则化的加入，相当于对模型参数加了一个约束，所以限制了解空间的范围，L1正则约束是一个正方形，而L2是一个圆形。

但是还是有很多人有疑问，我们要优化的形式不是$$min_w E_D(\omega)+\lambda E_R(\omega)$$吗，为什么在图里把它给拆开成两项了？图中的优化问题好像用下面这个式子表达更合适：

$$min_\omega E_D(\omega)$$

$$s.t. E_R(\omega) \le \eta $$

其实这两个问题是等价的，即对一个特定的$\lambda$总存在一个$\eta$ 使得这两个问题是等价的。

或者从拉格朗日乘子法的角度，下边这个问题其实可以转化成$$min_\omega E_D(\omega) + \lambda (E_R(\omega)-\eta )$$，从这个角度来看，正则化其实也就等价于对参数加了约束。

### 贝叶斯角度

从贝叶斯的角度来说，正则化相当于是假设参数服从某个先验分布，然后通过最大化后验概率估计参数$\omega$

![贝叶斯](\img\in-post\regularization\bayes1.png)

其中$p(D|\omega)$是似然函数：参数向量w的情况下，观测数据D出现的概率

$p(\omega)$是参数向量的先验概率(prior)

对于似然函数部分有$p(D|\omega) = \Pi_{k =1}^{n}p(D_i|\omega)$

则，对后验概率取对数有

![bayes](\img\in-post\regularization\bayes2.jpg)

当先验概率满足正态分布时

![bayes](\img\in-post\regularization\bayes3.png)

代入式子展开可以得到

![bayes](\img\in-post\regularization\bayes4.jpg)

对比下式

![bayes](\img\in-post\regularization\bayes5.png)

可以看到，似然函数部分对应于损失函数（经验风险），而先验概率部分对应于正则项。所以L2正则，等价于参数$\omega$的先验概率分布满足一个零均值，方差为$1/\lambda$正态分布。

同理，对于L1正则，相当于参数$\omega$的先验概率满足拉普拉斯分布。

![bayes](\img\in-post\regularization\bayes6.png)

![bayes](\img\in-post\regularization\bayes7.jpg)

## 为什么L1比L2更容易得到稀疏解

这个问题同样可以从上面的两个角度来分别说明。

从优化的角度，从最上方那张图中可以看到，L1正则化中损失函数和约束条件的交点更有可能出现在坐标轴上，也就是最优值处的参数更有可能是0，所以模型也就更加稀疏。而L2正则交点则不是很容易到坐标轴上。

从贝叶斯的角度，看看正态分布和拉普拉斯分布的图像就知道了。

![laplace](\img\in-post\regularization\laplace.jpg)

![正态分布和拉普拉斯分布](\img\in-post\regularization\normal_laplace.jpg)

拉普拉斯在0值附近非常突出，而正态分布则对比较大的值乘法更重。



在知乎上看到了另外一个角度的解释，感觉也很好。

假设原损失函数为$L(\omega)$, 那么L1正则为$L_1(\omega) = L(\omega) + C\lvert \omega\rvert$, L2正则化为$L_2(\omega) = L(\omega) + C\omega^2$

两种正则化能不能把最优的$\omega$变成 0，取决于原先的费用函数在 0 点处的导数。

如果本来导数$L'(0)​$不为 0，那么施加 L2 正则后导数$L_2'(0) = L'(0) + 2C * 0​$依然不为 0，最优的 $\omega​$也不会变成 0。

而施加 L1 正则时，只要正则项的系数 C 大于原损失函数$L(\omega)$在 0 点处的导数的绝对值， $\omega = 0$ 就会变成一个极小值点。这里要解释一下，要使得$L_1(\omega)$在$\omega = 0$处取得极值，那么需要$\omega = 0$两边的导数异号。$\omega$从左边趋近于0 时，$C\lvert \omega\rvert$的导数是$-C$，从右边趋近于0时，$C\lvert\omega\rvert$的导数是$C$，所以$L_1$左右两边的导数分别为$L'(0) - C$和$L'(0) + C$，要保证两者异号，只需要保证$C\gt \lvert L'(0)\rvert$就可以了。

所以L1正则的最优值更容易在参数为0时取得，这也就造成了L1比L2更容易得到稀疏解。



#### 参考资料

1. [机器学习中常常提到的正则化到底是什么意思](https://www.zhihu.com/question/20924039)
2. [l1 相比于 l2 为什么容易获得稀疏解？ - 王小明的回答 - 知乎](https://www.zhihu.com/question/37096933/answer/189905987)
3. [从贝叶斯角度深入理解正则化](https://blog.csdn.net/zhuxiaodong030/article/details/54408786)
4. [l1 相比于 l2 为什么容易获得稀疏解？ - 王赟 Maigo的回答 - 知乎](https://www.zhihu.com/question/37096933/answer/70426653),