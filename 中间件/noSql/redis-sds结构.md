---
title: redis--sds结构
toc: true
date: 2022-02-22 15:26:49
tags:
- redis
categories:
- 组件
typora-root-url: ..\img
---

SDS是redis的字符串struct体，所有的数据结构都依赖于SDS，它是可以简单的理解为char数据或者string类型，但是有区别：SDS是一个结构体，

struct SDS {

  int alloc;     //标识有最大的空间大小，剩余空间(alloc-len);不包括head                //和'\0'

  int len;       //标识使用了多少字节的空间，其中不包括'\0'

  unsinged int flag; //标识 struct 的大小类型，为了让空间更加紧密

  char buf[];    //保存原始字符的数据

};

<!-- more -->

SDS是通过长度判断字符的完成性，所以可以保存不可显示的字符串，这个点std::string 类似；而c字符串是通过'\0'判断，所以可能发生截断；

| len  | alloc | flag | r    | e    | d    | i    | s    | \0填充 |      |      | \0   | \0   |
| ---- | ----- | ---- | ---- | ---- | ---- | ---- | ---- | ------ | ---- | ---- | ---- | ---- |
|      |       |      |      |      |      |      |      |        |      |      |      |      |

通过flag更快的找到更合适的字符串结构，提升写入和查找性能，并且能保证内存的最大化利用；

```c++
sds sdscatlen(sds s, const void *t, size_t len) {
    size_t curlen = sdslen(s);

    s = sdsMakeRoomFor(s,len);
    if (s == NULL) return NULL;
    memcpy(s+curlen, t, len);
    sdssetlen(s, curlen+len);
    s[curlen+len] = '\0';
    return s;
}
```

直接将字符串填补在len长度之后，从最后的'\0'开始memcopy,直接覆盖'\0',然后再重置最后一个'\0'
