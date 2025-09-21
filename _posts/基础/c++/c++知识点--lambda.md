---
title: c++知识点--lambda
date: 2022-02-19 11:42:47
tags:
- 语言
categories: 
- c++
toc: true
typora-root-url: ..\img
---
### Lambda

###### [capture list] (para list) mutable exception -> returntype {function body}

- 如果是值传递，body不能对值做改变；如果你想对其做改变；可以加上mutable；
- 返回类型可以自动推导，不用带auto
- 如果你想有返回类型，必须有参数列表；返回类型必须跟在参数列表的后面，并且必须在返回类型前面包含尾随返回类型关键字字->
- 捕获的变量是lambda定义之前的所有局部变量（包括lambda所在的类内），局部静态变量可以直接使用
<!-- more -->
###### 编译器实现原理

- 编译器创建一个lambda类,实现构造函数；重载函数调用运算符（其实就是扩号）

- 创建lambda对象

- 通过这个对象调用operator()

- 代码实例

  ```
  class lambda_xxxx
  {
  private:
      int c;
      int b;
  public:
      lambda_xxxx(int _a, int _b) :c(_a), b(_b)
      {
      }
      bool operator()(int x, int y) throw()
      {
          return c + b > x + y;
      }
  };
  void LambdaDemo()
  {
      int a = 1;
      int b = 2;
      lambda_xxxx lambda = lambda_xxxx(a, b);
      bool ret = lambda.operator()(3, 4);
  }
  //其中，类名 lambda_xxxx 的 xxxx 是为了防止命名冲突加上的
  ```

- lambda_xxxx 与 lambda 表达式 的对应关系

  - lambda 表达式中的**捕获列表**，对应 lambda_xxxx 类的 **private 成员**
  - lambda 表达式中的**形参列表**，对应 lambda_xxxx 类成员函数 **operator() 的形参列表**
  - lambda 表达式中的 **mutable**，对应 lambda_xxxx 类成员函数 **operator() 的常属性 const**，即是否是 **常成员函数**
  - lambda 表达式中的**返回类型**，对应 lambda_xxxx 类成员函数 **operator() 的返回类型**
  - lambda 表达式中的**函数体**，对应 lambda_xxxx 类成员函数 **operator() 的函数体**
  - 另外，lambda 表达 捕获列表的捕获方式，也影响 对应 lambda_xxxx 类的 private 成员 的类型
  - 值捕获：private 成员 的类型与捕获变量的类型一致
  - 引用捕获：private 成员 的类型是捕获变量的引用类型

<!-- more -->

### 右值相关

- 右值要么是常量，要么是临时表达式

- 右值表示放在等号右边，不能放在等号左边的量，一般都是常量，右值不具有名称（匿名变量）；【左值和右值的区分办法，能不能对这个表达式取地址】

- 右值引用就是绑定到右值或者临时对象上，如：int&& A = 0; int && a =geta();

- c++11之前更偏向于右值是一种常量引用。右值引用相当于把他两细化开了。

- 引用折叠：

  - 所有的右值引用叠加到右值引用上仍然还是一个右值引用；

  - 所有的其他引用类型之间的叠加都将变成左值引用

  - 引用不是对象，所以没有引用的引用，因此间接定义时，一旦出现引用的引用，就要进行折叠

  - X& &, X& &&, X&& &都折叠成X&

  - X&& &&折叠成X&&

- 右值引用解决了对象深拷贝的问题，因为传右值引用调用移动赋值用算符和移动构造函数。

- ```c++
  #include<string>
  #include<vector>
  #include<iostream>
  using namespace std;
  class RightValueTest {
  private:
      std::string str;
  public:
      RightValueTest(const string& s) : str(s) {
          std::cout <<"RightValueTest con" << std::endl;
      }
      RightValueTest(const RightValueTest& obj) {
          std::cout <<"RightValueTest copy" << std::endl;
          str = obj.str;
      }
      RightValueTest(RightValueTest&& obj) {
          std::cout <<"RightValueTest move";
          str = std::move(obj.str);
          std::cout <<" ,after origin obj.str:" << obj.str << "--" << std::endl;
      }
      RightValueTest& operator= (const RightValueTest& obj) {
          std::cout <<"left value = " << std::endl;
          str = obj.str;
          return *this;
      }
      RightValueTest& operator= (RightValueTest&& obj) {
          std::cout<<"right value = ";
          str = std::move(obj.str);
          std::cout<<" ,after origin obj.str:" << obj.str << std::endl;
          return *this;
      }
      void print() {
          std::cout << "print str value:" << str << endl;
      }
      ~RightValueTest() {
      	std::cout << "xigou RightValueTest" << std::endl;
      }
  };
  int func(std::vector<RightValueTest>& vect) {
  	std::vector<RightValueTest> tmp_vect;
  	tmp_vect.reserve(5);
  	RightValueTest test6("1314");
  	tmp_vect.push_back(test6);
  	tmp_vect.push_back(test6);
  	tmp_vect.push_back(test6);
  	tmp_vect.push_back(test6);
  	tmp_vect.push_back(test6);
  	for(auto& ele : tmp_vect) {
  		vect.emplace_back(std::move(ele));
  	}
  
  }
  int main() {
      std::vector<RightValueTest> vect;
      //RightValueTest test1(std::move("abc"));
     /* RightValueTest test1("abc");
      RightValueTest test2 = std::move(test1);
      test2.print();
      test1.print();
  
      RightValueTest test3("xxx");
      RightValueTest test4("ooo");
      test4 = std::move(test3);
      test3.print();
      test4.print();
      vect.reserve(5);
      cout << "size: " << vect.size() << ", capacity: " << vect.capacity() << endl;
      vect.emplace_back(std::move(test4));*/
      RightValueTest test5("520");
      test5.print();
      vect.push_back(test5);
      cout << "size: " << vect.size() << ", capacity: " << vect.capacity() << endl;
      //func(vect);
  
      //cout << "size: " << vect.size() << ", capacity: " << vect.capacity() << endl;
      //for(auto& ele : vect) {
  //	    ele.print();
    //  }
      //vect.emplace_back(test4);
  }
  ```

- 

- std::move()是将一个变量强制转化为一个右值

  ```c++
  //在返回类型和类型转换中也要用到typename
  template <typename T>
  typename remove_reference<T>::type&& move(T&& t)
  {
      return static_cast<typename remove_reference<T>::type&&>(t);
  }
  ```

- 完美转义：通过std::forward()进行完美转义

- 右值的特点：

  - 右值引用使右值和其变量生命周期一样长；
  - 只有在泛型编程中，右值引用的参数才是通用的应用类型，他有时候是左值引用；有时候是右值引用，具体看传入的参数。
