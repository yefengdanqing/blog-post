---
title: gdb调试相关
date: 2022-08-17 18:42:47
tags:
- gcc
categories: 
- 工具
toc: true
typora-root-url: ..\img
---
###### core文件定位，使用这些工具和指令排查基本步骤包括：

- 通过 top 找到内存泄露的进程


- 通过 pmap 找到内存泄露的地址及范围

  <!-- more -->

- 通过 gcore 对进程内存进行快照，gcore 产生 coredump 文件

- 通过 gdb 加载内存信息，gdb 加载 coredump 文件

- 通过gdb的 dump binary 导出泄露内存的内容，(gdb) dump binary memory result.bin {start_addr} {end_addr}命令 dump 内存

- 通过 vim 查看内存内容，根据内存中的内容，锁定对应的代码段，进行排查修复，使用 :%!xxd 的方式查看内容

- 常用命令：
  - top -b -n 1 -H -p 12877
  - pmap -x -p 12877

<!-- more -->

###### 多线程

分为两种模式:all-stop模式和no-stop模式.（gdb7.0之前不支持no-stop模式）

- all-stop：当程序在gdb因某种原因停止时,所有的线程都会停止.一般来说,gdb不能够单步所有线程,因为线程调度室gdb无法控制的(???).无论何时,当gdb停止你的程序时.它都会自动切换到触发断点的那个线程.
- no-stop(网络编程常用)：只是当前线程会被停止,而其他线程将会继续运行.此时step,next只对当前线程起作用.
- gdb调试多线程常用命令
  - info threads : 显示可以调试的所有线程
  - thread ID : 切换到指定ID的线程
  - break FileName.cpp:LinuNum thread all：所有线程都在文件FileName.cpp的第LineNum行有断点。
  - thread apply ID1 ID2 IDN command: 让线程编号是ID1，ID2…等等的线程都执行command命令。
  - thread apply all command：所有线程都执行command命令。
  - set scheduler-locking off|on|step： 在调式某一个线程时，其他线程是否执行。在使用step或continue命令调试当前被调试线程的时候，其他线程也是同时执行的，如果我们只想要被调试的线程执行，而其他线程停止等待，那就要锁定要调试的线程，只让他运行。off:不锁定任何线程，默认值。 on:锁定其他线程，只有当前线程执行。
  - step:在step（单步）时，只有被调试线程运行。
  - set non-stop on/off: 当调式一个线程时，其他线程是否运行。
  - set pagination on/off: 在使用backtrace时，在分页时是否停止。
  - set target-async on/off: 同步和异步。同步，gdb在输出提示符之前等待程序报告一些线程已经终止的信息。而异步的则是直接返回。
  - show scheduler-locking： 查看当前锁定线程的模式
  
  ###### 其他
  
  ```gdb
  set print pretty on 
  set print object on 
  set print static-members on 
  set print vtbl on 
  set print demangle on 
  set demangle-style gnu-v3 
  set print sevenbit-strings off
  ```
  
  
