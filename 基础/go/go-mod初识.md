---
title: go初始了解
toc: true
date: 2022-10-13 15:26:59
tags:
- 语言
categories:
- go
typora-root-url: ..\img
---

##### go module

###### 简介

若是想要在其他位置新建go项目，就不得不更改系统变量GOPATH，将其设为新项目的位置，可能还要改goland中的GOPATH设置。

go语言一直到1.10，都是使用GOPATH设置模块搜索路径，但从1.11开始，引入了新的Go模块管理机制（go modules），不过一直到1.15，默认的模块管理方式仍然是GOPATH，直到Go1.16开始，将默认的模块管理方式改成了go modules，在这种工作模式下，每一个模块都必须使用go.mod文件指定模块的位置。

最大的问题是如果go.mod文件中使用了绝对路径指定了模块路径，如果在git push时将每个模块的go.mod文件都上传到了[服务器](https://cloud.tencent.com/act/pro/promotion-cvm?from_column=20065&from=20065)，那么在git pull到其他机器，由于路径可能不一样，如果进行git push操作的是macOS或Linux，而进行git pull操作的是Windows，那路径肯定是不一样的。

为了解决go modules的这个饱受诟病的问题，从Go1.18开始，推出了工作区的概念，基本思路就是，每一个模块仍然需要一个go.mod文件，但这个文件主要用于指定模块名和go的版本，并不需要指定引用模块的路径，而所有模块的路径统一由工程根目录的go.work文件指定。go.work文件的语法与go.mod文件的语法类似，但可以通过use指定模块的路径。

之前主要是用GOPATH 和 Vendor，vendor相对主流，但现在官方更提倡go mod。

<!-- more -->

###### 初始化

​		go mod init modulename

```go
module github.com/yefengdanqing/go_freshman

go 1.19

require (
	github.com/gin-gonic/gin v1.9.0
)
```



###### 环境变量

GO111MODULE

在 GOPATH 目录之外新建一个目录，并使用`go mod init`初始化生成 go.mod 文件。go.mod文件一旦创建后，它的内容将会被go toolchain全面掌控。go toolchain会在各类命令执行时，比如go get、go build、go mod等修改和维护go.mod文件。将需要引入外部包的go文件置于项目目录下，编译文件，就会把外部包下载到本地的GOPATH/pkg/mod目录下。
set GO111MODULE=off，GOPATH mode，查找vendor和GOPATH目录。
set GO111MODULE=auto，如果当前目录不在$GOPATH 并且 当前目录（或者父目录）下有go.mod文件，则使用 GO111MODULE， 否则仍旧使用 GOPATH mode

###### go.mod

* 项目名字
* 版本
* 第三方包

go.mod 提供了 module、require、replace 和 exclude 四个命令[第三方包使用方式]：

- require 语句指定的依赖项模块；
- replace 语句可以替换依赖项模块；替换无法直接获取的package
  - 当项目没上传到git的时候，可以用replace 替换成本地项目

- exclude 语句可以忽略依赖项模块。

go mod edit -require="xxxx@v.xxx"修改版本

go get使用该命令下载公开库，会把依赖下载到第一个gopath下

go mod vendor

###### go.sum

##### 导入import

###### 路径

###### 点操作

这个点操作的含义就是这个包导入之后在你调用这个包的函数时，你可以省略前缀的包名，也就是前面你调用的

```go
import . “fmt”
fmt.Println(“hello world”) 
//可以省略的写成
Println(“hello world”)
```



###### 别名

别名操作顾名思义可以把包命名成另一个自己绝对方便的名字

```go
import( f “fmt” )  
//别名操作调用包函数时前缀变成了重命名的前缀，即
f.Println(“hello world”)
```



###### _ 操作

**_** 操作其实只是引入该包，只是使用该包的init函数，并不显示的使用该包的其他内容。注意：这种形式的import，当import时就执行了fmt包中的init函数，而不能够使用该包的其他函数。

```go
import (
    _ "github.com/go-sql-driver/mysql"
)
```



##### 编译和运行

go build xxx.go

###### 运行

go run xxx.go

##### 报错

1. ![](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/image-20221014171252113-20221014171459118.png)

   Go mod init modulename(项目名字)

2. ![image-20221014180545475](/../../../../../Library/Application Support/typora-user-images/image-20221014180545475.png)

   配置GO111MODULE=auto

3. go的import的包一定要使用，不然就会被go mod tidy清理、优化掉

4. ![](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/image-20221014202125815.png)读了sum.goland.org,需要配置，GONOPROXY="gitlab.xxx.com"，GONOSUMDB="gitlab.xxxx.com"，GOPRIVATE="gitlab.xxx.com"

5. 东方饭店

6. 烦啥科技

###### 引用

https://juejin.cn/post/7026523806730551333

<!-- more -->



1. 任务难点 1.1 标准及统一的协议设计 项目设计多个模块的协议设计，包括 feature-extraction-lib、feature-generation-lib、feature-generator-server。另外，为统一协议，ranker server 协议语言也由 flatbuffer 改为 protobuf。 1.2 代码改动量大 ranker server 在经历几年的迭代后，代码逻辑复杂，可读性差，项目的改动量极大，近乎代码重构，对代码能力和业务逻辑理解深度有更高要求。 1.3 服务性能难预期 由于涉及大量业务逻辑改动、协议变动，对最终的服务性能影响较大且不好预估。 2. 关键点 2.1 抽取 feature-extraction-lib 将 feature extraction 相关的操作，从 ranker server 中剥离，抽取为独立的代码逻辑。 2.2 抽取 feature-generation-server (FGS) 将 feature-extraction-lib 的输出，即 raw feature，作为 FGS 的输入，调用 feature-generation-lib，完成终态特征的生成，将相关的代码抽离作为独立的代码逻辑，为后续服务化做准备。
