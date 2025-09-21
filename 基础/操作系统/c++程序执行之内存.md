---
title: c++程序执行之内存
date: 2022-02-18 18:42:47
tags:
- 语言
categories: 
- c++
toc: true
typora-root-url: ..\img
---
#### main函数之前和之后

- 分配空间
- 初始化静态和全局变量
- 加载配置参数
- 之后可以析构回调函数
- 注册一个main之后的回调函数atexit(func)
- 可以通过 **attribute** 关键字，声明 **constructor** 和 **destructor** 来实现

<!-- more -->

#### malloc实现

- 当申请小内存（小于128K）的时候，malloc使用sbrk分配内存；当申请大内存时，使用mmap函数申请内存；但是这只是分配了虚拟内存，还没有映射到物理内存，当访问申请的内存时，才会因为缺页异常，内核分配物理内存
- new实现的原理
  - 底层调用malloc：当大于128K时，用mmap在堆和栈的虚拟内存中找一块空间分配；否则调用brk在堆上有高到低的分配；
  - 对比malloc：对于malloc分配失败需要用remalloc
  - stl 设计当大于128B的时候用maclloc，否则用stl的内存池；
    new是在自由存储区，而malloc是堆分配

#### c++源码编译

![64022](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/64022.jpg

#### 函数执行过程



#### 虚函数

- 单个继承的话，每个类对应一个虚函数表，子类覆盖子类的虚函数


- 多继承，一个对象有多个虚函数表指针，指向多张虚函数表，对于当前类新的虚函数，会增加到第一个虚函数指针对应的虚函数表里面。并且最后一个虚函数表会有一个“0”的标记，标识后面没有了虚函数。

![virtual](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/virtual.jpg)

![virtual-func](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/img/virtual-func.jpg)

https://www.cnblogs.com/malecrab/p/5573368.html

【https://blog.csdn.net/qq_36359022/article/details/81870219】【-fdump-class-hierarchy打印虚表】

###### c++执行的过程
