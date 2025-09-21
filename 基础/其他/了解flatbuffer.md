---
title: 了解flatbuffer
toc: true
date: 2022-03-04 22:26:59
tags:
- 语言
categories:
- flatbuffers
typora-root-url: ..\img
---

### 简介

[FlatBuffers](https://github.com/google/flatbuffers)是为Google发布的一个跨平台，提供多种语言接口，注重性能和资源使用的序列化类库。目前该类库提供C++、C#、C、GO、Java、JavaScript、PHP、Python语言接口。该序列化库多用于移动端手游数据传输以及特定的对性能要求较高的应用。具有一下特点：

<!-- more -->

1、对序列化的数据不需要打包和拆包

2、内存和效率速度高，扩展灵活

3、代码依赖较少

4、强类型设计，编译期即可完成类型检查

5、使用简单、可跨平台使用

<!-- more -->

### 使用

通过flact 生成类似pb的.h/.cpp文件

```c++
/******
table AdvOfferTable;
******/
//创建指针
auto builder = std::make_shared<flatbuffers::FlatBufferBuilder>();
auto &table_builder = *(builder.get());
//中介获取数据，然后通过CreateXXX获得偏移量
auto offset = CreateCreativePackageTable(table_builder, cpId, whDcoId, status, setting, updated, updatedDate);
//完成序列化
builder->Finish(offset);

// 反序列化为结构体指针
auto table = flatbuffers::GetRoot<AdvOfferTable>(builder->GetBufferPointer());
auto tmp = table->xxx();
```
