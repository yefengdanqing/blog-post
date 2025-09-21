---
title: c++知识点--模板
date: 2022-02-21 11:42:47
tags:
- 语言
categories: 
- c++
toc: true
typora-root-url: ..\img
---

### 类

***

###### 类在堆上构建

- 按照单例或者把析构函数设置为私有的

###### 类在栈上构建

只要禁用new运算符就可以实现类对象只能建立在栈上。将operator new()设为私有即可

<!-- more -->

### 模板类

*****

- 模板类：模板定义只是声明一个蓝图或者公式；只有在实例化的时候才会生成真正的代码（也就是调用的接口处）

   - 特化：就是按照特定类型去实现一个类模板。

   - 偏特化：模板偏特化（Template Partitial Specialization）是模板特化的一种特殊情况，指显示指定部分模板参数而非全部模板参数，或者指定模板参数的部分特性分而非全部特性，也称为模板部分特化。与模板偏特化相对的是模板全特化，指对所有的模板参数进行特化。模板全特化与模板偏特化共同组成模板特化。模板偏特化主要分为两种，一种是指对部分模板参数进行全特化，另一种是对模板参数特性进行特化，包括将模板参数特化为指针、引用或是另外一个模板类。

   - typename在下面情况下禁止使用：
     - 模板定义之外，即typename只能用于模板的定义中
     - 非限定类型，比如前面介绍过的`int`，`vector<int>`之类
     - 基类列表中，比如`template <class T> class C1 : T::InnerType`不能在`T::InnerType`前面加typename
     - 构造函数的初始化列表中
     <!-- more -->
     
   - 类可以包含本身是模板的成员函数。这种成员称为成员模板。成员模板不能是虚函数

   - 因为模板只有使用时才会实例化，所以相同的实例如果出现在不同的对象文件中，多个源文件使用了相同的模板，并提供了相同的模板参数时，就会导致每个文件中都有该模板的一个实例。这会引起代码体积膨胀，C++11以后允许通过显式实例化来避免这种开销

     ```c++
     extern template class Blob<string>;  //声明
     template int compare(const int&, const int&);  //定义
     ```

   - 一些情况下编译器无法推导出模板实参的类型，这时我们可以显式地控制模板实例化

     ```c++
     template <typename T1, typename T2, typename T3>
     T1 sum(T2, T3);
     //无法推断出T1的类型，因为T1不在函数参数中出现//T1是显式指定的，T2和T3是从函数实参类型推断出来的
     auto val3 = sum<long long>(i, lng);  //long long sum(int ,long)
     //显式指定模板实参要严格按照自左向右的顺序来匹配，可以指定全部，也可以指定部分。
     ```

   - 尾置返回类型与类型转换

     ```c++
     template <typename It>
     auto fcn(It beg, It end) -> decltype(*beg)
     {
         //处理
         return *beg;  //返回序列第一个元素的引用
     }
     ​
     vector<int> fi={1,2,3,4,5};
     Blob<string> ca={"hi","bye"};
     auto &i = fcn(vi.begin(), vi.end());//fcn返回int&
     auto &s = fcn(ca.begin(), ca.end());//fcn返回string&
     //decltype作用于表达式时，如果表达式返回的是左值，则decltype返回左值引用。
     template <typename It>
     auto fcn2(It beg, It end) -> typename remove_reference<decltype(*beg)>::type
     {
       //处理
       return *beg;  //返回序列中一个元素的拷贝
     }
     //remove_reference获得元素类型。它有一个模板类型参数和一个名为type的类型成员。如果用引用类型实例化remove_reference，则type就表示被引用的类型。
     ```

   - 函数指针和实参推导利用std::forward() 可以兼容形参类型透传

   - 函数模板可以被模板函数或者普通函数重载

     - 函数匹配规则

   - 可变参数模板

     ```c++
     template <typename T>
     ostream &print(ostream &os, const T &t)
     {
         return os << t;  //包中最后一个元素之后不打印分隔符
     }
     ​
     template <typename T, typename... Args>
     ostream& print(ostream &os, const T &t, const Args&...reset)
     {
         os << t << ", ";//打印第一个实参
         return print(os, reset...);  //递归调用，打印其他参数
     } 
     ```

     - 包扩展就是加上3个点

   - 模板特例化

     - 模板函数不提供偏特化版本，通过重载实现
     - 可以偏特化、特化和普通的都实现

   - 模板类限制类型：c++11提供enable_if和is_base_of（判断一个是否是另一个的基类）

