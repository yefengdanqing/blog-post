---
title: c++编译汇总
toc: true
date: 2022-03-04 22:26:59
tags:
- 
categories:
- 
typora-root-url: ..\img
---

1. 库编译问题，glog依赖gflags需要编译成动态库

   <!-- more -->

   ![](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/image-20230302102258655.png)

2. protobuf的三个动态库都要链接，并且要先加dependence的pb依赖

   ![image-20230302113818125](/../../../../../Library/Application Support/typora-user-images/image-20230302113818125.png)

3. 在开启单元测试的时候，glog和gflags 都依赖gtest，不然会有链接问题

4. cmake错误

   ![image-20230306175300742](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/image-20230306175300742.png)

   在文件flags.make:8；在第8行有格式错误导致报错；

5. ![image-20230419183158371](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/image-20230419183158371.png)

   由于没有链接cpp文件导致找不到定义，cmake 写的有问题

6. 编译器构造函数优化问题：直接优化为一次构造，通过开启和禁用-fno-elide-constructors关闭返回值优化

   ![image-20230517103335987](/../../../../../Library/Application Support/typora-user-images/image-20230517103335987.png)

   <!-- more -->

7. 编译器右值优化

8. google::SetCommandLineOption[abi:cxx11](char const*, char const*)和google::SetCommandLineOption(char const*, char const*) 的区别

   

9. 可能有多个gcc/g++版本，或者对应的c++ std库链接的还是旧的

10. ![image-20240416145259200](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/image-20240416145259200.png)

![image-20240416151316248](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/image-20240416151316248.png)



