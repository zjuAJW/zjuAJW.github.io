---
layout:     post
title:      "Mongo 单个collection大数据量插入性能测试"
subtitle:   " \"该死的mongo\""
date:       2017-03-14 17:27:00
author:     "AJW"
header-img: "img/mongodb.jpg"
tags:
    - Mongodb
---

Mongo 单个collection大数据量插入性能测试  
---
由于之前发现不能在一个mongo实例下建立太多的collection，所以只能把数据存在一个collction中咯。所以想测试一下Mongo在单个collection中大数据量下的插入性能。


## 实验设置  
利用mongoimport向单个collection中导入100万条8个int的数据，重复300次。collection上建立了4个键的复合索引。  
## 实验结果
![单个collection大数据量下的插入性能](https://github.com/zjuAJW/MarkdownPhoto/blob/master/mongo.png?raw=true)

![第二次，先把mongo服务关掉](https://github.com/zjuAJW/MarkdownPhoto/blob/master/mongo_huge_data_2.png?raw=true)

图中横坐标是插入次数（1～300），纵坐标是每次插入的用时。开始的时候用时35s左右，后期最高点用时2181s。  
数据导入过程CPU使用率一直高达300%多，数据导入结束后内存占用保持在在40%左右（不过貌似mongo是会一直占内存直到内存占满的？）  
出现的问题是：存在数据丢失现象（按道理应该有300×100万的数据，可是最后count了一下貌似只有296000000左右的数据？具体的数值不记得了，
但是少的还是挺多的），而且时间波动明显，有很多向下的尖刺。

## 速度变慢的问题
这个是在意料之中的，按照mongo的内存使用，在用完可用的内存后，其插入速度会有明显的下降，确实在曲线上升的那段是内存用完的时间。  

## 数据丢失问题
这个问题有点严重，搞了好久没找到原因，也许我的实验方法就是错的。

### 错误日志
看了一下mongo的日志，有很多连接错误：

	[conn1343] AssertionException handling request, closing client connection: 6 socket exception [SEND_ERROR] for 127.0.0.1:47164

看mongoimport的输出，确实在某几次会出现连接不了的情况：

	Failed: Closed explicitly

	Failed: error connecting to db server: no reachable servers

会报这两种错误，第一个是在插入过程中连接挂掉了，第二种是在mongoimport连接时就没连上。

### 网上的类似问题及解决方案  
网上找了一下，有[类似的问题](http://blog.csdn.net/u010443481/article/details/50912752)，不过也没写具体原因。只是说：
>mongo是先存储在缓存中然后在存入数据库，但是在存入数据库的过程中有可能会对数据库连接出现问题。
  
StackOverflow上有在导入比较大的数据文件时[无法连接的问题](http://stackoverflow.com/questions/33475505/mongodb-mongoimport-loses-connection-when-importing-big-files)：  
>you can use mongoimport option -j. Try increment if not work with 4. i.e, 4,8,16, depend of the number of core you have in your cpu.  
>  
>mongoimport --help
>    
>-j, --numInsertionWorkers= number of insert operations to run concurrently (defaults to 1)    
>mongoimport -d mietscraping -c mails -j 4 < mails.json  

但是试了一下好像没什么作用。

### 一些分析和实验
根据上面的报错，soecket exception， 感觉是连接或者网络上的问题，难道是连接数满了？mongoimport之后没有释放连接？  
但是看了一下，import之后，连接的状态都是TIME_WAIT,所以连接是关闭了的。  

![TIME_WAIT状态](https://github.com/zjuAJW/MarkdownPhoto/blob/master/TIME_WAIT.PNG?raw=true)

但是TIME\_WAIT的连接也是要占用连接数的啊，之前看过，ulimit -n 最大的open files 限制是1024，难道是这里超了？
做了一下实验，发现当我连接数（包括TIME\_WAIT状态的）远超1024的时候，竟然还能建立连接插入数据！！难道我理解错了这个1024的意义？手动捂脸。。。。。。  

>**小tips**:mongo的配置文件里有net.maxIncomingConnections选项来限制mongo的可用连接数，但注意，这个连接数会有一个封顶值，就是系统open files 的80%，如果你设置的值大于这个值的话，是没有用的。  

既然这样，那我就建立很多连接但是先不断开，看报的错误是否和之前是一样的。  
当所建连接数大于mongo本身的连接数（maxIncomingConnections）时，会报错

	2017-03-09T22:18:29.084+0800 I NETWORK  [thread1] connection refused because too many open connections: 819

当所建连接数大于open files的限制时（这里由于collecton也要占用文件句柄，所以，建立的连接数其实是少于这个限制的），会报错：
![too many open files](https://github.com/zjuAJW/MarkdownPhoto/blob/master/too%20many%20open%20files.PNG?raw=true)
  

所以二者的报错都和之前的错误是不一样的，应该并不是这两个问题。

感觉还是内存的问题，内存使用增长到40%左右就变了想想就感觉很奇怪啊！！40%这个数跟官方的说法对不上！

利用cacheSizeGB将mongo能用的内存调整到8GB，做了一次实验，成功导入的数据次数确实增加了一些，但是不明显。内存占用上，比之前多了大概1GB，也许应该再增加一些，这样确认一下是不是内存的原因。于是将内存使用增加到了12G，结果。。。并没有什么卵用，该丢的还是丢了，而且是在内存还没占满的时候就丢了，所以，不是内存的问题？

另外，将所有数据都整合到一个大的数据文件中时，没有出现数据丢失的现象。

在网上也寻求了很多帮助，在csdn的论坛上发帖基本只有版主在回复，也没解决什么问题。偶然间进了这个[萌阔论坛](http://forum.foxera.com/mongodb/),看氛围还不错，就[发帖求助了一下](http://forum.foxera.com/mongodb/topic/633/mongoimport%E8%BF%9E%E7%BB%AD%E5%AF%BC%E5%85%A5%E5%A4%9A%E4%B8%AAcsv%E6%97%B6-%E5%A4%B1%E5%8E%BB%E8%BF%9E%E6%8E%A5%E9%80%A0%E6%88%90%E9%83%A8%E5%88%86%E6%95%B0%E6%8D%AE%E4%B8%A2%E5%A4%B1)，很多热心的人帮助，还有人做了同样的实验，结果他们并没有出现问题。
>MongoDB單機版本  
>一樣300次的匯入 每次匯入資料大小約為59M (150000筆*399bytes)  
>匯入每次資料的時間約為6.7s，從頭到為時間都沒有減慢

我还能说什么呢？可能我们俩的唯一区别就在于索引了，他估计没有索引，而我是有一个四个字段的复合索引的。难道这个索引会造成这么大的问题？  

今天突然想起来，开发机上也装了mongo，于是就在开发机（windows系统，16G内存）上测试了一下，结果并没有出现这个问题，我真的要怀疑是测试机上的mongo没有正确安装了。。。