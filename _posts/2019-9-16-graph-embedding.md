---
layout:     post
title:      "Graph Embedding"
subtitle:   "另一种embedding方法"
date:       2019-09-16 17:00:00
author:     "AJW"
header-img: "img/tech.jpg"
tags:
    - CTR
	- 推荐系统
---

传统的embedding模型处理的都是序列化的数据，而在很多场景，比如电商中，人、物、场之间往往是复杂的图结构，那么这时候如何来进行合理的embedding以表示它们之间的相关度呢，这就是Graph Embedding做的事情。

### Deep Walk

deepwalk采用的是随机游走的方法，从图中生成序列，然后利用word2vec的方法来生成embedding，是早期比较简单的一种生成embedding的方法。

随机游走的概率由节点之间边的权重来获取：

![deep walk weight](\in-post\graph-embedding\deep-walk.png)

### Node2vec

Node2vec在deepwalk随机游走的基础上进行了改进，通过改变游走的权重使得embedding的结果能更好地反映出节点间的同质性（homophily）和结构相似性（structural equivalence）。

同质性指的是在网络中相互连接、属于相似网络、距离较近的这些节点，应该有相近的embedding表示（比如图中的s1和u节点，它们都同在一个子网络中），而结构相似性则代表着在网络中有相似功能的节点应该有相近的embedding表示（比如图中的u和s6节点，它们都是各自网络的中心）

![bfs and dfs](\img\in-post\graph-embedding\bfs-and-dfs.png)

为了使embedding能够更好的表达网络的**同质性**，我们再游走的时候需要使其更偏向于**深度优先搜索**，而为了能表达**结构相似性**，则需要让游走更偏向于**宽度优先搜索**。为什么是这样呢，可以参考 [关于Node2vec算法中Graph Embedding同质性和结构性的进一步探讨](https://zhuanlan.zhihu.com/p/64756917)

Node2vec通过两个参数$$p$$和$$q$$来控制游走时是偏向BFS还是DFS

![node2vec](\img\in-post\graph-embedding\node2vec1.png)

如上图所示，假设目前我们刚刚从t游走到v，那么下一步的概率由$$\pi_{vx}=\alpha(t,x)*\omega_{vx}$$决定，其中$$\omega_{vx}$$为边$$vx$$的权重，而$$\alpha(t,x)$$定义为：

![node2vec weight](\img\in-post\graph-embedding\node2vec-weight.png)

其中，$$d_{tx}$$表示t到x的距离，p和q控制着游走的概率，p越小，返回t的概率越大，那么游走更倾向于在局部进行，也就是BFS；而q越小，则游走到远方的概率越大，也就是DFS

