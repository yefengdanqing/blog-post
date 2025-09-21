---
title: redis--持久化
toc: true
date: 2022-02-22 23:44:49
tags:
- redis
categories:
- 组件
typora-root-url: ..\img
---

Redis持久化分为两种：第一种是rdb(redis database);第二种是aof（append only file);

###### RDB

RDB 是一个二进制文件，可以压缩也可以不压缩

持久化的方式：手动save和自动bgsave

持久化的过程：手动save使redis服务器被hang住，不接受客户端的命令，直到save结束；bgsave可以手动触发，也可以通过配置特定的条件触发，比如：60s内执行了1000次命令；900s内执行了1次命令；bgsave是background save，在后台执行，通过子进程的方式

持久化文件的加载：在启动redis的加载rdb文件，在没有开启aof的时候，默认是开始加载rdb文件，没有特别的命令支持加载rdb

限制：在bgsave期间不执行其他客户端发来的save和bgsave;

 | REDIS | 过期时间 | key  | type | 编码类型的value |
 | ----- | -------- | ---- | ---- | --------------- |

注解：文件检验和是个好东西

<!-- more -->

###### AOF

AOF是保存所有执行的redis命令

持久化的条件：在配置中开启aof选项，选项为appendonly no/yes;当开启aof的时候优先加载aof；

aof持久化的方式：通过配置的形式自动化的同步，比如选项：appendsync:awalys/everysec;或者客户端通过bgrewrite命令进行

持久化的过程：将命令追加到aof buf里面，经过everysec 后flush到磁盘

限制：文件会过大

rewrite:存在aof文件过大，需要手动bgrewrite进行持久化或者通过aof percent 或者aof size 来进行自动同步；

在rewrite的开启一个没有网络链接的伪客户端，创建一个新文件，按照redis当天数据以命令的形势写如文件，同时开启一个aof 重写buf空间，保存在重新的过程中的这部分数据，当 当前redis的数据已经写入新aof文件后，我们就把aof重写buf空间的数据写入新的aof文件，然后原子的mv新的文件到旧文件。

对比：rdb的一致性比aof的高；rdb一般适合做永久备份，大规模的数据备份；在同步aof重写buf的之前，子进程会发信号给主进程，主进程会阻塞一下，导致redis服务器临时hold一下；fork子进程的时候也会hold子进程；重命名aof也会阻塞主进程

bgrewrite手动触发和自动触发【通过当前的大小和比例】

###### 比较

rdb的优点：适合大规模的数据备份，灾难恢复；二进制文件相对aof较小；rdb在备份和恢复的时候相对更稳定。

rdb的缺点：出现故障的时候容易丢数据，数据的完整性和一致性的更低

aof的有点：完整性和一致性相对较高， redis-check-aof 可以修复异常的aof文件；最多丢失1秒的数据

aof的缺点：文件越来越大，需要重写；写的速度慢于rdb;

同时使用两种持久化，新持久化的时候是复制之前的临时文件。

持久化的格式

