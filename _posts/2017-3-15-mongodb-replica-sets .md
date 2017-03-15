---
layout:     post
title:      "Mongodb副本集"
subtitle:   "数据冗余，降低风险"
date:       2017-03-15 16:20:00
author:     "AJW"
header-img: "img/mongodb.jpg"
tags:
    - Mongodb
---

副本集（Replica Set）的存在可以提高系统的稳定性，以防单台服务器上的数据丢失。在某些情形下，副本集的存在可以提升用户读取数据的性能，因为用户可以发送请求到不同的数据服务器上。而且在实际的生产部署过程中，都是要以副本集为基础的。这里记录一些mognodb副本集的注意事项。

## 副本集的组成
副本集由主节点(primary)、从节点(secondaries，或许应该翻译成备份节点，这不是重点)以及可选的仲裁节点(arbiter)组成，官方文档建议最少有3个存数据的节点（例如：1个主节点，2个从节点），这样冗余性较好。  
另外，副本集最多可以有50个成员，但是最多只有7个可以进行投票。
### 主节点
主节点是副本集中唯一接受写操作的节点，主节点会将所有的写操作记录在opolog中，从节点会复制log信息并将操作作用于自身的数据集。  
所有的成员都可以接受读操作，但是，默认会从主节点进行读取。  
一个副本集只能有一个主节点，当主节点出现问题时，会选举出一个新的主节点。

### 从节点
从节点存储主节点数据的一份复制，可以接受读操作，可以通过选举机制成为新的主节点。

从节点可以配置优先级、隐藏节点、延时节点等，可以根据需要进行相应的配置。具体的参考[官方文档](https://docs.mongodb.com/manual/core/replica-set-secondary/)。

### 仲裁节点
仲裁节点不存储数据，并且不能成为主节点，他的作用就是投票。按照官方文档的说法，仲裁节点的存在就是为了能让投票的总人数成为奇数，但是要注意，不要在主节点或者从节点的主机上运行仲裁节点（具体原因我也不清楚，个人理解可能是为了让仲裁节点以一个外部的视角看待副本集中的成员，也就是当有人挂掉了，不会影响到它，旁观者清～）
>Arbiters participate in elections in order to break ties. If a replica set has an even number of members, add an arbiter.

>IMPORTANT:  
>Do not run an arbiter on systems that also host the primary or the secondary members of the replica set.

## oplog
oplog是副本集之间保持同步的方式。
