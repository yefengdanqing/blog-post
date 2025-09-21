---
title: redis--hash
toc: true
date: 2022-02-22 15:26:49
tags:
- redis
categories:
- 组件
typora-root-url: ..\img
---

###### Hash

redis的hash结构是指某个key对应的值是hash，值为key-value的形似；

![hash111](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/hash111.jpg)

<!-- more -->

(ps:图是盗取的）

其中ht是大小为2的指针数组，分表指向两个hashtable 表；当key发生冲突的时候，通过链地址法解决冲突，在dictEntry结构体中有节点的next指针，用来指向冲突的下一个节点，每次有新节点时，插在连表的头部。

redis哈希算法：使用了murmurhash2算法

rehash：

![hash222](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/hash222.jpg)

(ps:盗图）

其中rehash分为两种情况：扩容和缩容

rehash的限制：当在进行bgsave或者background时并且装载因子大于1时不进行rehash；

rehash条件：【扩容】当进行bgsave或者时并且装载因子大于5的时候进行rehash;正常情况是装载因子大于1就进行rehash（其中不包括在bgrewriteaof或者bgsava的；【缩容】：当hash因子是小于0.1（100个节点，不到10个在用）的时候，执行缩容

rehash的过程：是一种渐进式的hash过程，当rehashidx=-1的时候不进行hash； 

首先分配2的n次幂（刚大于已有的2倍）的大小；

其次从0开始，每rehash一个指针数组元素时。rehashidx就进行++操作

第三直到所有的节点都rehash 完，其中有size和sizemask；再讲rehashidx重置为-1；

最后将ht[0]指向新的hash表，ht[1]置为空

rehash时的增删改查：所有的增删改查都在新的hash表上进行，旧的hash表只是做迁移，将其节点链在指针数据下面。
