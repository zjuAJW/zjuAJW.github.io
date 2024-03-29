---
layout:     post
title:      "决策树test2"
subtitle:   "你真的理解了吗"
date:       2018-06-27 15:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
    - 机器学习
    - 算法
---

## 各种决策树的区别

总得来说：

1. ID3和C4.5只能做分类，而CART可以做回归
2. ID3和C4.5可以是多叉的，而CART是二叉树
3. ID3只能处理离散变量，而C4.5和CART可以处理连续变量
4. ID3对缺失值敏感，而CART和C4.5可以处理缺失值
5. ID3和C4.5对特征只能使用一次，而CART则可以使用多次

ID3在分裂时采用的是信息增益，这样会倾向于选择取值较多的属性，因为这样每一个属性当中的样本数会比较少，纯度也会比较大。C4.5在这方面做出了改进，采用信息增益比而不是信息增益来选择分裂特征。因为取值较多的属性总体的信息熵也会较大，所以信息增益比上信息原本的信息熵值就不会那么大了。

CART树分裂时采用的则是gini系数。

C4.5算法相比ID3主要做出了以下方面的改进 

1. 可以处理连续数值型属性 

对于离散值，C4.5和ID3的处理方法相同，对于某个属性的值连续时，假设这这个节点上的数据集合样本为total，C4.5算法进行如下处理： 

- 将样本数据该属性A上的具体数值按照升序排列，得到属性序列值：$$A_1,A_2,A_3,...,A_{total}$$
- 在上一步生成的序列值中生成total-1个分割点。第i个分割点的取值为$$A_i$$和$$A_i+1$$的均值，每个分割点都将属性序列划分为两个子集。
- 计算每个分割点的信息增益(Information Gain),得到$$total-1$$个信息增益。
- 对分裂点的信息增益进行修正：减去$$\frac{log_2(N-1)}{\vert D\vert}$$，其中N为可能的分裂点个数，D为数据集合大小。
- 选择修正后的信息增益值最大的分类点作为该属性的最佳分类点
- 计算最佳分裂点的信息增益率(Gain Ratio)作为该属性的Gain Ratio
- 选择Gain Ratio最大的属性作为分类属性。

用信息增益率(Information Gain Ratio)来选择属性 

2. 克服了用信息增益来选择属性时偏向选择值多的属性的不足。信息增益率定义为： 

![img](\img\in-post\Decision Tree\1.png)

其中Gain(S,A)和ID3算法中的信息增益计算相同，而SplitInfo(S,A)代表了按照属性A分裂样本集合S的广度和均匀性。

![img](\img\in-post\Decision Tree\2.png)

其中Si表示根据属性A分割S而成的样本子集。 

其实这里的分母就相当于是按照属性A来对数据集进行了类别划分，然后求了信息熵。

3. 缺失值处理

对于某些采样数据，可能会缺少属性值。在这种情况下，处理缺少属性值的通常做法是赋予该属性的常见值，或者属性均值。另外一种比较好的方法是为该属性的每个可能值赋予一个概率，即将该属性以概率形式赋值。例如给定Boolean属性B，已知采样数据有12个B=0和88个B=1实例，那么在赋值过程中，B属性的缺失值被赋值为B(0)=0.12、B(1)=0.88；所以属性B的缺失值以12%概率被分到False的分支，以88%概率被分到True的分支。这种处理的目的是计算信息增益，使得这种属性值缺失的样本也能处理。

## 剪枝的处理

### ID3和C4.5

首先明确决策树$$T$$的**损失函数**： 

$$C_\alpha(T)=N_tH_t(T)+\alpha|T|$$

其中$$\vert T\vert$$为叶节点的数量，$$H_t(T)$$为叶节点t的信息熵，$$N_t$$为叶节点t的样本点数量，$$\alpha\ge0$$为参数。损失函数主要由两部分组成，第一部分可以看作是模型的预测误差，而第二部分则是对模型复杂度的惩罚项。

ID3和C4.5的剪枝过程是**固定$$\alpha$$的值**，然后选择损失函数最小的模型的过程。具体地，对于算法生成的整个决策树T，给定$$\alpha$$的值，递归地从叶节点向上回缩，即剪掉某些叶节点；如果剪掉节点后树的损失函数减小，那么就进行剪枝，将父节点变为新的叶节点；重复整个过程，直至不能继续为止。

### CART

CART树的剪枝和ID3、C4.5的主要区别就在于，它没有提前固定$$\alpha$$的值，而是通过不断生成原树的子树，来求得相应的$$\alpha$$值。

同样的，我们仍旧从叶节点开始向上回缩决策树，叶节点回缩到父节点t后，这**单个父节点t**的损失函数为

$$C_\alpha(t)=C(t) +\alpha$$

而回缩前，以t为根节点的子树$$T_t$$的损失函数为

$$C_\alpha(T_t)=C(T_t)+\alpha\vert T_t\vert$$

当$$\alpha$$比较小的时候，剪枝前的树必定表达能力更强，所以有$$C_\alpha(T_t)<C_\alpha(t)$$，随着$$\alpha$$的增大，$$C_\alpha(T_t)$$的大小会逐渐超过$$C_\alpha(t)$$，当$$\alpha=\frac{C(t)-C(T_t)}{\vert T_t \vert -1}$$的时候，两者相等，而这个临界点也就是剪枝与不剪枝的临界点。

由此，随着$$\alpha$$的增加，我们的决策树就会一点一点地被剪掉，得到一系列的子决策树，而且每个决策树都对应着一个$$\alpha$$的值。对这些子树，采用交叉验证的方法选择最优的一个，就得到剪枝之后的决策树。

## 决策树的优缺点

决策树有很多优点，比如：

- 易于理解、易于解释
- 可视化
- 对与缺失值的处理比较容易
- 很容易进行多类别的分类
- 非常高效的非线性模型
- 使用决策树（预测数据）的成本是训练决策时所用数据的对数量级。

但这些模型往往不直接使用，决策树一些常见的缺陷是：

- 构建的树过于复杂，无法很好地在数据上实现泛化。

- 数据的微小变动可能导致生成的树完全不同，因此决策树不够稳定。

- 决策树学习算法在实践中通常基于启发式算法，如贪婪算法，在每一个结点作出局部最优决策。此类算法无法确保返回全局最优决策树。

- 如果某些类别占据主导地位，则决策树学习器构建的决策树会有偏差。因此推荐做法是在数据集与决策树拟合之前先使数据集保持均衡。

- 某些类别的函数很难使用决策树模型来建模，如 XOR、奇偶校验函数（parity）和数据选择器函数（multiplexer）。

  

##  随机森林

随机森林的主要优点体现在随机的行抽样和列抽样上。行抽样是有放回的进行的，而列抽样则要注意，**不是说每棵树采用一个特征的子集，而是在每个节点分裂的时候，随机选择一部分特征来进行节点的分裂**。

随机森林的主要优点在于，随机性的引入使得它更不容易过拟合，而且对噪声有一定的抗性，而且可以并行。



## GBDT和Xgboost

GBDT和Xgboost是以决策树为基础的两个重要算法，两者单独在另一篇文章中进行讨论，详见[GBDT和Xgboost](https://zjuajw.github.io/2018/02/28/GBDT-and-Xgboost/)

