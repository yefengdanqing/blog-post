---
title: go进阶
toc: true
date: 2023年03月07日 星期一 18时41分33秒
tags:
- 语言
categories:
- go
typora-root-url: ..\img
---

### 并发

<!-- more -->

#### 上下文和context

context 通常和select 结合使用，用来监听其是否结束或者取消

context可以在goroutine之间传递，

#### Interface

* 概念上讲一个接口的值，接口值，由两个部分组成，一个具体的类型和那个类型的值;
* Go 语言使用 [`runtime.iface`](https://draveness.me/golang/tree/runtime.iface) 表示第一种接口，使用 [`runtime.eface`](https://draveness.me/golang/tree/runtime.eface) 表示第二种不包含任何方法的接口 `interface{}`，两种接口虽然都使用 `interface` 声明，但是由于后者在 Go 语言中很常见，所以在实现时使用了特殊的类型。



##### 空接口类型的支持--interface{}

任何其他类型的数据都可以赋值给interface{}类型

```go
func ProcessInterface(value interface{}) interface{} {
	if num, ok := value.(int); ok {
		fmt.Println(num)
	} else if str, ok := value.(string); ok {
		fmt.Println(str)
	} else if arr, ok := value.([]float64); ok {
		fmt.Println(arr)
	} else {
		fmt.Println("type is unsupport")
	}
	return value
}
```

###### 接口断言

类型断言是一个使用在接口值上的操作。语法上它看起来像x.(T)被称为断言类型，这里x表示一个接口的类型和T表示一个类型。一个类型断言检查它操作对象的动态类型是否和断言的类型匹配。

```go
value, ok := interfaceValue.(ConcreteType)
// interfaceValue 是你想要断言的接口变量。
// ConcreteType 是你希望将接口转换成的具体类型。
// value 是转换成功后的值，ok 是一个布尔值，表示转换是否成功。
```



##### 接口实现泛型、继承等特性

定义:包含一组方法或者函数的集合，只有方法，没有数据（由struct承接）；

使用:接口是一组方法签名

```go
type ProcessType int

const (
	FbProcess ProcessType = iota + 1
	RsProcess
)

type IRequest interface {
	GetAdxId() string
	GetOneid() int
	GetProcessType() ProcessType
	SetAllValue(id string, oneId int)
}
type RsRequest struct {
	AdxId string
	OneId int
	Type  ProcessType
}

func (rsRequest *RsRequest) GetAdxId() string {
	return rsRequest.AdxId
}
func (rsRequest *RsRequest) GetOneid() int {
	return rsRequest.OneId
}
func (rsRequest *RsRequest) GetProcessType() ProcessType {
	return rsRequest.Type
}
func (rsRequest *RsRequest) SetAllValue(id string, oneId int) {
	rsRequest.AdxId = id
	rsRequest.OneId = oneId
	rsRequest.Type = RsProcess
}

type FbRequest struct {
	AdxId string
	OneId int
	Type  ProcessType
}

func (fbRequest *FbRequest) GetAdxId() string {
	return fbRequest.AdxId
}
func (fbRequest *FbRequest) GetOneid() int {
	return fbRequest.OneId
}
func (fbRequest *FbRequest) GetProcessType() ProcessType {
	return fbRequest.Type
}
func (fbRequest *FbRequest) SetAllValue(id string, oneId int) {
	fbRequest.AdxId = id
	fbRequest.OneId = oneId
	fbRequest.Type = FbProcess
}

func WriteProcess(request IRequest, id string, oneId int) {
	request.SetAllValue(id, oneId) //改成值传递就行
}
func ReadProcess(request IRequest) {
	fmt.Println(request.GetAdxId(), request.GetOneid(), request.GetProcessType()) //改成值传递就行
}
```

如下只需要将Irequest改成值传递就行

![image-20241021151115811](/../../../../../Library/Application Support/typora-user-images/image-20241021151115811.png)

**接口本身包含了类型信息**：

- 一个接口值由两个部分组成：一个指向具体类型的指针和一个指向该类型的方法集。通过值传递接口时，这两个信息已经包含在接口值内部。
- 这意味着接口值本质上已经是一个类似指针的结构，不需要再使用指针来引用接口。

**值类型接口更符合 Go 语言习惯**：

- 在 Go 中，值传递接口更加常见，并且保持了代码的一致性。对于大多数场景来说，值类型接口已经能够满足需求。

**避免不必要的解引用操作**：

- 如果使用接口的指针类型，你需要额外的解引用操作来访问接口的实际方法，这增加了代码复杂性。
- 指针指向接口时，Go 需要对指针进行额外的解析，通常没有必要。



定义小而专注的接口，只包含必要的方法。避免定义过于庞大的接口。

定义小接口有以下优点，首先小接口定义了有限的方法，使得接口的用途更加明确和易于理解。其次是由于小接口只定义了少量的方法，从而更容易遵循单一职责原则。同时由于小接口专注于特定的功能，因此具有更高的可复用性。

因此，在接口设计时，我们应该尽量定义小接口，然后通过组合接口来组装出更为复杂的接口。

下面是一些常见的规范，能够帮助我们定义出小接口:

1. 初期设计接口：思考接口需要具备哪些核心功能，只定义与这些功能相关的方法。避免将不必要或无关的方法包含在接口中，保持接口的简洁性。
2. 迭代接口: 分析接口的使用场景，思考是否可以将其抽取为多个接口，根据实际的使用情况和需求变化，对接口进行调整和优化。
3. 尽量满足单一职责原则: 在进行接口的迭代分析时，多思考其是否满足单一职责原则。
4. 考虑使用接口组合: 一个类型需要同时满足多个接口的功能，可以使用接口组合的方式。

从上面可以看出来，小接口的定义并非是一蹴而就的，也是随着需求的变化，对领域的理解越来越深刻，在不断变化的，这个需要我们不断思考演进的。

#### 反射

运行时反射是程序在运行期间检查其自身结构的一种方式。反射带来的灵活性是一把双刃剑，反射作为一种元编程方式可以减少重复代码，但是过量的使用反射会使我们的程序逻辑变得难以理解并且运行缓慢。我们在这一节中会介绍 Go 语言反射的三大法则[](https://draveness.me/golang/docs/part2-foundation/ch04-basic/golang-reflect/#fn:3)，其中包括：

1. 从 `interface{}` 变量可以反射出反射对象；
2. 从反射对象可以获取 `interface{}` 变量；
3. 要修改反射对象，其值必须可设置；

### 同步原语和锁

#### sync

##### sync.Mutex

所有编程里面的互斥锁，当一个线程直接锁住临界区时，其他线程等待中

##### sync.RWMutex

所有编程里面的读写锁，当一个线程获取读锁后，其他线程可以获取读锁，不能获取写锁（等待）；当一个线程获取到写锁后，其他线程获取读或者写都要等待

#### sync.WaitGroup

计数器

| 方法名                          | 功能                |
| ------------------------------- | ------------------- |
| (wg * WaitGroup) Add(delta int) | 计数器+delta        |
| (wg *WaitGroup) Done()          | 计数器-1            |
| (wg *WaitGroup) Wait()          | 阻塞直到计数器变为0 |

#### sync.Once

异步的只执行一次

在编程的很多场景下我们需要确保某些操作在高并发的场景下只执行一次，例如只加载一次配置文件、只关闭一次通道等。

Go语言中的sync包中提供了一个针对只执行一次场景的解决方案–sync.Once。

sync.Once只有一个Do方法，其签名如下：

```go
func (once *sync.Once)Do(f func()) {}
```

#### sync.Map

#### atomic

底层是自旋锁来控制并发的安全

#### GMP 原理与调度

![13](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/13.jpg)

```go
package main

import (
    "os"
    "fmt"
    "runtime/trace"
)

func main() {

    //创建trace文件
    f, err := os.Create("trace.out")
    if err != nil {
        panic(err)
    }

    defer f.Close()

    //启动trace goroutine
    err = trace.Start(f)
    if err != nil {
        panic(err)
    }
    defer trace.Stop()

    //main
    fmt.Println("Hello World")
}
```

会得到一个 trace.out 文件，然后我们可以用一个工具打开，来分析这个文件。

```
$ go tool trace trace.out 
2020/02/23 10:44:11 Parsing trace...
2020/02/23 10:44:11 Splitting trace...
2020/02/23 10:44:11 Opening browser. Trace viewer is listening on http://127.0.0.1:33479
```

我们可以通过浏览器打开 [http://127.0.0.1:33479](http://127.0.0.1:33479/) 网址，点击 view trace 能够看见可视化的调度流程。

### 定时器

#### timer

#### ticker

### channel

Go 语言中的 `select` 也能够让 Goroutine 同时等待多个 Channel 可读或者可写，在多个文件或者 Channel状态改变之前，`select` 会一直阻塞当前线程或者 Goroutine。

channel是一种类型，一种引用类型

```go
var 变量名字 chan 数据类型
var a chan int
a := make(chan int)
a<-100	发送数据给chan a
x := <-a  从a里面读取数据
```





### 调度器

##### 网络轮询器

##### 系统监控

#### 内存管理

##### 内存分配器

##### 垃圾收集器

##### 栈内存管理

#### 元编程

#### 系统库
