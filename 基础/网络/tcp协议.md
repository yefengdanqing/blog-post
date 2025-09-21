---
title: rpc、socket、tcp/udp整理梳理
toc: true
date: 2022-03-04 22:26:59
tags:
- 
categories:
- 
typora-root-url: ..\img
---

##### 概念

**RPC**:远程过程调用（分布式、微服务间的方法调用）

**HTTP**:无状态，每次请求都要发送一个request，服务器响应之后就断掉（http header中的keep-alive指的是tcp）

**TCP**:面向连接，三次握手保证通信可靠

**UDP**:非面向连接，不可靠，速度快（可以手动对数据收发进行验证，IM系统多采用，QQ）

<!-- more -->

**Socket**:TCP协议的接口实现，面向传输层进行网络编程, socket并不是一种协议，是在程序员层面上对TCP/IP协议的封装和应用。其实是一个调用接口，方便程序员使用TCP/IP协议栈而已。程序员通过socket来使用tcp/ip协议。但是socket并不是一定要使用tcp/ip协议，Socket编程接口在设计的时候，就希望也能适应其他的网络协议。

**RPC(Remote Procedure Call)是远程过程调用**，比如说现在有两台服务器A, B，一个在A服务器上的应用想要调用B服务器上的应用提供的某个，由于不在两个方法不在一个内存空间，不能直接调用，需要通过网络表达调用的语义和传达调用的数据。常存在于分布式系统中。

RPC跟HTTP不是对立面，RPC中可以使用HTTP作为通讯协议。**RPC是一种设计、实现框架**，通讯协议只是其中一部分。
