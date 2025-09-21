---
title: 原子操作相关
toc: true
date: 2023-05-04 22:26:59
tags:
- 
categories:
- 
typora-root-url: ..\img
---

#### 概念

在 x86/x64 架构上，原子加载（load）和存储（store）操作通常具有 acquire 和 release 语义，这意味着它们可以用作同步原语，以确保对共享数据的正确访问顺序。

- **Acquire 语义**：当一个线程执行具有 acquire 语义的原子加载操作时，它确保在该操作之前的所有读取和写入都已完成，且结果已对其他线程可见。换句话说，在 acquire 操作之后，该线程能够看到在 acquire 操作之前由其他线程对共享数据所做的修改。
- **Release 语义**：当一个线程执行具有 release 语义的原子存储操作时，它确保在该操作之后的所有读取和写入都不会在该存储操作之前执行。换句话说，在 release 操作之前的所有修改对其他线程可见，而在 release 操作之后的任何修改对其他线程不可见。

在 C++11 中，使用 `std::atomic` 类模板可以实现带有 acquire 和 release 语义的原子操作。

```c++
#include <atomic>

std::atomic<int> data;
std::atomic<bool> ready;

void thread1() {
    while (!ready.load(std::memory_order_acquire)) {
        // 等待数据准备
    }
    int value = data.load(std::memory_order_relaxed);
    // 使用 value
}

void thread2() {
    // 准备数据
    data.store(42, std::memory_order_relaxed);
    ready.store(true, std::memory_order_release);
}

```



#### 举例

##### 动机

<!-- more -->

##### 实际场景

假如王二狗和牛翠花约好在天安门约会，两人从起点到目的地可以借助的通行方式比较多，至于选那种方式完全因人而异。

##### 使用方法（核心代码）

```c++
class Context；
```



#### 模板方法模式适合应用场景

#### 实现方式（类图）



#### 模板方法模式优缺点

优点：

缺点：

#### 与其他模式的关系

<!-- more -->
