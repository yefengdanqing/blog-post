---
title: c++智能指针
toc: true
date: 2022-02-22 22:26:59
tags:
- 
categories:
- 
typora-root-url: ..\img
---

enbale_shared_from_this是一个模板类，std::enable_shared_from_this 能让其一个对象（假设其名为 t ，且已被一个 std::shared_ptr 对象 pt 管理）安全地生成其他额外的 std::shared_ptr 实例（假设名为 pt1, pt2, … ） ，它们与 pt 共享对象 t 的所有权。为什么要用 `enable_shared_from_this`？

什么时候使用enable_shared_from_this

- 需要在类对象的内部中获得一个指向当前对象的 shared_ptr 对象。
- 如果在一个程序中，对象内存的生命周期全部由智能指针来管理。在这种情况下，要在一个类的成员函数中，对外部返回this指针就成了一个很棘手的问题。

<!-- more -->

```c++
template <typename T>
class SharedPointer
{
private:

    class Implement
    {
    public:
        Implement(T* p) : mPointer(p), mRefs(1){}
        ~Implement(){ delete mPointer;}

        T* mPointer;  //实际指针
        size_t mRefs;  // 引用计数
    };

    Implement* mImplPtr;

public:

    explicit SharedPointer(T* p)
      : mImplPtr(new Implement(p)){}

    ~SharedPointer()
    {
        decrease();  // 计数递减
    }

    SharedPointer(const SharedPointer& other)
      : mImplPtr(other.mImplPtr)
    {
        increase();  // 计数递增
    }

    SharedPointer& operator = (const SharedPointer& other)
    {
        if(mImplPtr != other.mImplPtr)  // 避免自赋值
        {
            decrease();
            mImplPtr = other.mImplPtr;
            increase();
        }

        return *this;
    }

    T* operator -> () const
    {
        return mImplPtr->mPointer;
    }

    T& operator * () const
    {
        return *(mImplPtr->mPointer);
    }

private:

    void decrease()
    {
        if(--(mImplPtr->mRefs) == 0)
        {
            delete mImplPtr;
        }
    }

    void increase()
    {
        ++(mImplPtr->mRefs);
    }
};
```

