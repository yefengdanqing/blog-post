---
title: redis--基本数据结构
toc: true
date: 2022-02-24 15:26:49
tags:
- redis
categories:
- 组件
typora-root-url: ..\img
---

**redis五种基本数据结构**：**string、list、hash、set、zset**

**redis底层数据结构：简单动态字符串、双端连表、字典、压缩列表、整数集合**

###### **链表**

1. 代码定义

   ```c++
   typedef struct list{
        //表头节点
        listNode *head;
        //表尾节点
        listNode *tail;
        //链表所包含的节点数量
        unsigned long len;
        //节点值复制函数
        void (*free) (void *ptr);
        //节点值释放函数
        void (*free) (void *ptr);
        //节点值对比函数
        int (*match) (void *ptr,void *key);
   }list;
   ```

<!-- more -->

###### **跳跃表（跳表）**

1. 效率：跳跃表效率和平衡二叉树一样

2. 上层数据结构：有序集合、集群节点中用到了跳表

3. 结构

4. 1. head：记录跳跃表的头结点；tail： 记录跳跃表的尾结点；level：记录最高层数;length：记录跳跃表节点个数。
   2. zplist
   3. zpnode

###### **整数集合（intset)**

1. 上层数据结构：集合键；前提：当元素都是整数，并且个数不多的时候使用
2. 升级：扩大数据的类型，就是扩容保证有更大的元素能放下。
3. 降级：不存在的
4. 

###### **压缩链表（ziplist)**

1. 效率：

2. 上层数据结构：哈希键和列表键的底层数据结构

3. 1. 哈希键：当键值对数量较少(hash的kv个数，下同），并且都是整型或者较短的字符串。

   2. 图示

   3. 1. ![zlist](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/zlist.jpg)
      2. ![ziplist](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/ziplist.jpg)
      3. 

   4. 

   5. 结构：

   6. 1. zlbyte:压缩列表的的长度

      2. zltail:压缩列表到尾结点的长度

      3. zllen:压缩列表节点的数量

      4. entryX:节点对象

      5. 1. prev_length表示上一个节点的长度【大小占1字节或者5字节，5字节时，第一个字节为0xFE;其他为】
         2. encodeing表示类型和长度，本身占的大小有1个字节、2个字节和5个字节；

      6. zlend 特殊值（0xFF)

         

###### **字典（hash）**

1. 单独章节

###### **对象结构**

1. 分类：字符串对象、列表对象、哈希对象、集合对象和有序集合对象

2. 代码结构

3. ```c++
   typedef struct redisObject { 
   // 类型 
   unsigned type:4; 
   // 编码 
   unsigned encoding:4; 
   // 指向底层实现数据结构的指针 
   void *ptr;
   //对象的引用计数
   int refcount;
   //对象最后访问的时间
   unsigned lur;
   } obj;
   ```

4. 1. encoding 表示对象用什么底层数据结构来实现的

   2. | 编码常量                | 底层数据结构                      |
      | ----------------------- | --------------------------------- |
      | REDIS_ENCODING_INT      | long                              |
      | REDIS_ENCODING_EMBSTR   | emb的简单string[sds](小于32字节） |
      | REDIS_ENCODING_RAW      | 简单动态string                    |
      | REDIS_ENCODING_ht       | 字典                              |
      | REDIS_ENCODING_list     | 双端链表                          |
      | REDIS_ENCODING_intset   | 整数集合                          |
      | REDIS_ENCODING_skiplist | 跳跃表                            |
      | REDIS_ENCODING_zplist   | 压缩列表                          |

   3. 字符串使用：int、raw、embstr；embstr适用于短字符串类型

   4. 

5. 内存回收：给对象建一个引用计数，通过引用计数来确定

6. 对象共享：会有一些常量的对象被所有的命令共享

总结：

### 列表

![list](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/list.png)

### hash

![hash](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/hash.png)

### set

![set](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/set.png)

### zset

![zset](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/zset.png)

###### **过期删除策略**

1. 定时删除：在设置键的过期时间的时候，设置定时器，当过期时间快到的时候，删除过期键

2. 定期删除：每隔一段时间就数据库进行遍历，删除里面的过期键。至于扫描策略，由算法定【每次遍历16个库，每个库20个元素，这过程是有时间限制的】。

3. 惰性删除：当再次获取该key的时候根据是否过期来进行是否删除

4. 主服务器在rbd、aof持久化的时候，都会删除过期键；主服务器在载入rdb（增加del命令）和aof的时候，都会删除过期键；从服务器在载入rdb文件的时候不会删除过期键，全部载入；但是在主从同步的时候，会清空从库，没啥影响；在复制的过程中，只有主服务器显示的给从服务器发送del命令的时候，从才会删除；所以在没有删除的瞬间，客户端访问从服务器，主从会得到不一样的结果。

5. 同时使用两种持久化**
###### redis淘汰策略

- noevication 不使用任何淘汰策略，超过限制时返回错误

-  allkeys-lru：加入键的时候，如果过限，首先通过LRU算法驱逐最久没有使用的键

- volatile-lru：加入键的时候如果过限，首先从设置了过期时间的键集合中驱逐最久没有使用的键

-  allkeys-random：加入键的时候如果过限，从所有key随机删除

  \5. volatile-random：加入键的时候如果过限，从过期键的集合中随机驱逐

  \6. volatile-ttl：从配置了过期时间的键中驱逐马上就要过期的键

  \7. volatile-lfu：从所有配置了过期时间的键中驱逐使用频率最少的键

  \8. allkeys-lfu：从所有键中驱逐使用频率最少的键

redis-hash
redis-zset
