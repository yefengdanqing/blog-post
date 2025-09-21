---
title: epoll
toc: true
date: 2022-03-04 22:26:59
tags:
- 
categories:
- 
typora-root-url: ..\img
---

多路复用：一个进程处理多个网络socket，一个socket有哪些信息；

###### 边缘触发和水平触发

边缘触发和水平触发：状态的变化；边缘触发就是对一个socket 在缓存区中有数据的时候，只触发一次；水平触发就是会一直不断的触发；

<!-- more -->

###### 水平触发

1. socket接收缓冲区不为空 有数据可读 读事件一直触发
2. socket发送缓冲区不满 可以继续写入数据 写事件一直触发
    备注：符合思维习惯，epoll_wait返回的事件就是socket的状态

###### 边缘触发

1. socket的接收缓冲区状态变化时触发读事件，即空的接收缓冲区刚接收到数据时触发读事件
2. socket的发送缓冲区状态变化时触发写事件，即满的缓冲区刚空出空间时触发读事件
    备注：仅在状态变化时触发事件

![img](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/20180531185610344)

总结一下：

1. 对于监听的 sockfd，最好使用水平触发模式，边缘触发模式会导致高并发情况下，有的客户端会连接不上。如果非要使用边缘触发，可以用 while 来循环 accept()。

2. 对于读写的 connfd，水平触发模式下，阻塞和非阻塞效果都一样，因为在阻塞模式下，如果数据读取不完全则返回继续触发，反之读取完则返回继续等待。全建议设置非阻塞。

3. 对于读写的 connfd，边缘触发模式下，必须使用非阻塞 IO，并要求一次性地完整读写全部数据。

https://blog.csdn.net/zxm342698145/article/details/80524331/

https://cloud.tencent.com/developer/article/1736793

https://blog.csdn.net/dongfuye/article/details/50880251

https://www.zhihu.com/question/20502870

https://blog.lucode.net/linux/epoll-tutorial.html

https://plantegg.github.io/2019/12/09/epoll%E7%9A%84LT%E5%92%8CET/

<!-- more -->
