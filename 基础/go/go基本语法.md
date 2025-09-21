---
title: go基本语法
toc: true
date: 2023-03-07 22:26:59
tags:
- 语言
categories:
- go
typora-root-url: ..\img
---

##### 包

这是因为在Golang中，**没有强制要求包名和目录名称一致**。也就是说，在上面的例子中，我们引用路径中的文件夹名称是`package2`，而在这个文件夹下面的两个文件，他们的包名，被设置成了`同一个。而在Golang的引用中，我们需要填写的是**源文件所在的相对路径**。也就是说，我们可以理解为，包名和路径其实是两个概念，文件名在Golang中不会被显式的引用，通常的引用格式是`packageName.FunctionName`。

<!-- more -->

结论如下：

- 导入包的时候，除了路径，还要加上module的名字
- import导入的是文件夹的名字，而不是包名
- 在习惯上将包名和文件夹名设置为一样，但这不是强制规定（但是不建议这么做，容易不能发现是在哪个文件夹中）
- 在使用函数或者变量的时候，用的是包名，而不是文件夹的名字
- 对于导入的包，例如tempconv导入的fmt包，则是对应源文件级的作用域，因此只能在当前的文件中访问导入的fmt包，当前包的其它源文件无法访问在当前源文件导入的包

###### 系统包

* strconv包提供了布尔型、整型数、浮点数和对应字符串的相互转换，还提供了双引号转义相关的转换

* unicode包提供了IsDigit、IsLetter、IsUpper和IsLower等类似功能，它们用于给字符分类。
* fmt.Printf函数的%b、%d、%o和%x等参数提供功能往往比strconv包的Format函数方便很多，特别是在需要包含附加额外信息的时候




##### nil

nil` 是一个 `Type`，根据源码 `var nil Type`，它其实也是 **Golang** 中的一中类型，nil 的类型必须是一个指针，通道，函数，接口，字典，切片类型,`nil` 只能赋值给指针、`channel`、`func`、`interface`、`map` 或 `slice` 类型的变量

##### 作用域

* 控制流标号，就是break、continue或goto语句后面跟着的那种标号，则是函数级的作用域

摘录来自
gopl-zh
yar999
此材料可能受版权保护。

##### 声明

###### 变量声明

* var para_name type
* var a int
* var a bool
* var a *int
* var a []int
* var a map[int] string
* var a chan
* var a error

var a fun(string) int

var a slice

const a int = 20

匿名变量：用_ 标识一个占位符，在被赋值后，会立即被操作系统回收，通常在批量赋值的时候使用

对于变量的声明也可以用:= 来定义自动推导，在之前声明的变量使用:=会报错;:=不能用于全局变量的声明

可以用括号声明多个全局变量

```go
var (
		a int
  	b int
)
```



##### 函数声明

###### 匿名函数

```go
func squares() func() int {
	var x int
	//
	return func() int {
		x++
		return x * x
	}
}
func Test_anonymous(t *testing.T) {
	Convey("anonynous", t, func() {
		f := squares()
		So(f(), ShouldEqual, 1)
		So(f(), ShouldEqual, 4)
		g := squares()
		So(g(), ShouldEqual, 1)
		So(g(), ShouldEqual, 4)

	})
}
```



函数名的首字母要大写,对包来说就是public 的函数；函数名的首字母是小写，对包来说是private；

func Func_name(para1, para2) return_type {

​	func_body

} 

调用的时候通过包名

返回多个参数的时候用括号括起来

##### 数组和切片

数组是一个由固定长度的特定类型元素组成的序列，一个数组可以由零个或多个元素组成。因为数组的长度是固定的，因此在Go语言中很少直接使用数组。和数组对应的类型是Slice（切片），它是可以增长和收缩动态序列，slice功能也更灵活，但是要理解slice工作原理的话需要先理解数组

###### 数组

数组大小不能被改变

```go
var array [10]int //必须被使用，size也是常量
var array = [10]int{1,2,3,4,5}
```

如果一个数组的元素类型是可以相互比较的，那么数组类型也是可以相互比较的，这时候我们可以直接通过==比较运算符来比较两个数组，只有当两个数组的所有元素都是相等的时候数组才是相等的。不相等比较运算符!=遵循同样的规则。


###### 切片（相当于动态数组，c++--vector)

含义：是按照需要切成自己想要的部分；一个slice由三个部分构成：指针、长度和容量。指针指向第一个slice元素对应的底层数组元素的地址，要注意的是slice的第一个元素并不一定就是数组的第一个元素。长度对应slice中元素的数目；长度不能超过容量，容量一般是从slice的开始位置到底层数据的结尾位置。内置的len和cap函数分别返回slice的长度和容量。

slice之间不能比较，因此我们不能使用==操作符来判断两个slice是否含有全部相等元素。不过标准库提供了高度优化的bytes.Equal函数来判断两个字节型slice是否相等（[]byte），但是对于其他类型的slice，我们必须自己展开每个元素进行比较

由于数组是固定大小的，切片可以提供一个动态的、灵活的动态数组结构；

切片通过一个左闭右开的半开区间来标识；slices[1:3]

```go
var arr1 [5]int
for i := 0; i < len(arr1); i++ {
		fmt.Printf("%d ", i)
		arr1[i] = i
	}
for ele, _ := range arr1 {
  fmt.Printf("%d", ele)
}
	slices1 := arr1[1:3]
	fmt.Println("\nslices: ", slices1)
	slices2 := arr1[0:5]
	//slices2[0] = 100 会改变原始的数组，指针的关系
	slices2 = append(slices2, 106, 107, 108)
	slices2[0] = 100 //不会改变，slices2的指针有变化
```

对切片进行赋值的时候，也会改变底层的数据的值；对切片进行append时候，如果当前*切片*空间不足的时候，会新建一个切片的空间然后将旧空间的元素拷贝过来，这样并不会改变原始数据的值；

通过make函数可以创建切片,相当于创建

```go
slices1 := make([]int, 10) // len是10个
slices2 ：= make([]int, 10, 20) // len是10，cap是20
```

数组在定义的时候会指定其大小；切片不用指定大小

```go
//数组
arr1 := [4]int{1, 2, 3, 4}
//切片
slices1 := []int{4,3,2,1}
// nil切片
var slices1 []int 
```

###### 指针数组

一个数组，数据类型是地址

```go
var ptrs [size]*int
// var ptrArr [size] *Type

var ptrs [4]*int
a, b, c, d := 1, 2, 3, 4
ptrs = [4]*int {&a, &b, &c, &d}

```

###### 数组指针

一个指针，一个地址；数据还是数组，指向数组

```go
var arrptr *[size]int
// *arrPtr[0]
// (*arrPtr)[0]* 寻址运算符和 [] 中括号运算符的优先级是不同的,加括号
```

##### map[string]string

* 使用内置的delete函数可以删除元素
* K对应的key必须是支持==比较运算符的数据类型，所以map可以通过测试key是否相等来判断是否已经存在
* Go 语言在运行时为哈希表的遍历引入了不确定性，也是告诉所有 Go 语言的使用者，程序不要依赖于哈希表的稳定遍历

由于map 的底层是一个hashmap,当map作为一个参数传入函数的时候，在函数内对map做操作的时候，调用层最终也能知道这hashmap有变化 

```go
map_variable := make(map[KeyType]ValueType, initialCapacity)
```

使用方式

```go
v1 := map[string]int {
  "111": 111,
  "222": 222,
  "333":333,
}
```



```go
AdInfo struct {
  
}
var testMap map[string]AdInfo

```

###### 结构体嵌入和匿名成员

Go语言有一个特性让我们只声明一个成员对应的数据类型而不指名成员的名字；这类成员就叫匿名成员

得意于匿名嵌入的特性，我们可以直接访问叶子属性而不需要给出完整的路径

结构体字面值必须遵循形状类型声明时的结构，所以我们只能用下面的两种语法，它们彼此是等价的

```
w = Wheel{8, 8, 5, 20}                       // compile error: unknown fields
w = Wheel{X: 8, Y: 8, Radius: 5, Spokes: 20} // compile error: unknown fields
fmt.Printf("%#v\n", w)
```







##### defer

defer的执行推迟到defer所在函数（外层函数）执行完毕才执行，并且defer后面只能是函数调用，defer用于资源的释放，defer是在return之前执行的;你只需要在调用普通函数或方法前加上关键字defer，就完成了defer所需要的语法。当defer语句被执行时，跟在defer后面的函数会被延迟执行。直到包含该defer语句的函数执行完毕时，defer后的函数才会被执行，不论包含defer语句的函数是通过return正常结束，还是由于panic导致的异常结束。你可以在一个函数中执行多条defer语句，它们的执行顺序与声明顺序相反

```go
func f() (r int) {
    defer func(r int) {
          r = r + 5
    }(r)
    return 1
}
```



##### 指针



go 没有指针运算；&取地址；*获取值

#####  panic 和 recover

###### panic

由于panic会引起程序的崩溃，因此panic一般用于严重错误，如程序内部的逻辑不一致。勤奋的程序员认为任何崩溃都表明代码中存在漏洞，所以对于大部分漏洞，我们应该使用Go提供的错误机制，而不是panic，尽量避免程序的崩溃。在健壮的程序中，任何可以预料到的错误，如不正确的输入、错误的配置或是失败的I/O操作都应该被优雅的处理，最好的处理方式，就是使用Go的错误机制

###### recover

如果在deferred函数中调用了内置函数recover，并且定义该defer语句的函数发生了panic异常，recover会使程序从panic中恢复，并返回panic value。导致panic异常的函数不会继续运行，但能正常返回。在未发生panic时调用recover，recover会返回nil。”





##### make和new



##### struct

结构体的定义也遵循首字规则；包外访问的struct，定义的时候必须是名字首字母大写，否则小写；如果是struct的成员变量，和struct遵循一样的规则;

在struct中，无论使用的是指针的方式声明还是普通方式，访问其成员都使用"."，在访问的时候编译器会自动把 stu2.name 转为 (*stu2).name。

struct分配内存使用new，返回的是指针。

struct没有构造函数，但是我们可以自己定义“构造函数”。

struct是我们自己定义的类型，不能和其他类型进行强制转换。

###### 定义和使用

使用字面量创建变量，这种使用方式，可以在大括号中为结构体的成员赋初始值，有两种赋初始值的方式，一种是按字段在结构体中的顺序赋值，下面代码中`m2`就是使用这种方式，这种方式要求所有的字段都必须赋值，因此如果字段太多，每个字段都要赋值，会很繁琐，另一种则使用字段名为指定字段赋值，如下面代码中变量`m3`的创建，使用这种方式，对于其他没有指定的字段，则使用该字段类型的零值作为初始化值。

```go
type Books struct {
  english string
  history string
}
type Student struct {
		id     int
    name   string
    email  string
    gender int
    age    int
    prices []float64
    knowlege map[string]*Books
  
}
```

```go
var m2 = Student{1,"小明","xiaoming@163.com",1,18} // 简短变量声明方式：m2 := Member{1,"小明","xiaoming@163.com",1,18}
var m3 = Student{id:2,"name":"小红"}// 简短变量声明方式：m3 := Member{id:2,"name":"小红"}
var m4 = Student{id:2}
```

结构体也可以不包含任何字段，称为`空结构体`，struct{}表示一个空的结构体，注意，直接定义一个空的结构体并没有意义，但在并发编程中，channel之间的通讯，可以使用一个struct{}作为信号量。

```go
ch := chan(struct{})
ch <- struct{}
```

###### 指针结构体

构造函数

###### tag

只要是可导出成员（变量首字母大写），都可以用json、pb、xml等tag，并且支持同时多个tag

```go
type User struct {
    UserId   int    `json:"user_id" bson:"b_user_id"`
    UserName string `json:"user_name" bson:"b_user_name"`
}
```

在定义结构体字段时，除字段名称和数据类型外，还可以使用反引号为结构体字段声明元信息，这种元信息称为Tag，用于编译阶段关联到字段当中,如我们将上面例子中的结构体修改为

tag可以为结构体的成员添加说明或者标签便于使用,这些说明可以通过反射获取到。

在前面提到了，结构体中的成员首字母小写对外不可见，但是我们把成员定义为首字母大写这样与外界进行数据交互会带来极大的不便，此时tag带来了解决方法。



###### 匿名结构体和匿名字段（字段、属性)

匿名字段实现is a的继承方式



###### 组合

组合实现has a的继承方式







###### 特性

1. 值传递
   1. 结构体是复合类型，只能通过复制一个副本来作为函数参数或者作为赋值的对象
2. 不能继承
   1. 
3. 结构体不能包含自己
   1. 不能包含自己的对象，可以包含对象的指针

###### 方法

通过给一个struct声明一个

在现实的程序里，一般会约定如果Point这个类有一个指针作为接收器的方法，那么所有Point的方法都必须有一个指针接收器，即使是那些并不需要这个指针接收器的函数

不管你的method的receiver是指针类型还是非指针类型，都是可以通过指针/非指针类型进行调用的，编译器会帮你做类型转换。
在声明一个method的receiver该是指针还是非指针类型时，你需要考虑两方面的内部，

第一方面是这个对象本身是不是特别大，如果声明为非指针变量时，调用会产生一次拷贝；

第二方面是如果你用指针类型作为receiver，那么你一定要注意，这种指针类型指向的始终是一块内存地址，就算你对其进行了拷贝。熟悉C或者C艹的人这里应该很快能明白。

* 一个对象的变量或者方法如果对调用方是不可见的话，一般就被定义为“封装”。封装有时候也被叫做信息隐藏，同时也是面向对象编程最关键的一个方面

###### 结构体模拟继承性











##### proto

proto.Unmarshal（json, 



##### go test

###### 使用

文件名必须是以`_test`结尾的源码；函数必须`Test`开头的函数

```go
package xxx

import "testing"

func TestHelloWorld(t *testing.T) {
    t.Log("hello world")
}
```

###### 运行

```go
go test helloworld_test.go
//指定 TestA函数 进行测试：
go test -v -run TestA select_test.go
//-run跟随的测试用例的名称支持正则表达式，使用-run TestA$即可只执行 TestA 测试用例
go test -v .
```

###### 单元测试日志

每个测试用例可能并发执行，使用 testing.T 提供的日志输出可以保证日志跟随这个测试上下文一起打印输出。testing.T 提供了几种日志输出方法，详见下表所示。

| 方  法 | 备  注                           |
| ------ | -------------------------------- |
| Log    | 打印日志，同时结束测试           |
| Logf   | 格式化打印日志，同时结束测试     |
| Error  | 打印错误日志，同时结束测试       |
| Errorf | 格式化打印错误日志，同时结束测试 |
| Fatal  | 打印致命日志，同时结束测试       |
| Fatalf | 格式化打印致命日志，同时结束测试 |

##### 基准测试——获得代码内存占用和运行效率的性能数据

基准测试可以测试一段程序的运行性能及耗费 CPU 的程度。Go语言中提供了基准测试框架，使用方法类似于单元测试，使用者无须准备高精度的计时器和各种分析工具，基准测试本身即可以打印出非常标准的测试报告。















<!-- more -->

