---
title: brpc_bthread
toc: true
date: 2023-05-04 22:26:59
tags:
- 
categories:
- 
typora-root-url: ..\img
---

#### 概念

bthread是一个M:N线程库，目的是在提高程序的并发度的同时，降低编码难度，并在核数日益增多的CPU上提供更好的scalability（可扩展性）和cache locality(局部缓存行）。M:N是M个bthread 对应N个pthread,一般M远大于N;

一个bthread被卡住不会影响其他bthread。关键技术两点：work stealing调度和bmutex

work stealing调度和butex，前者让bthread更快地被调度到更多的核心上，后者让bthread和pthread可以相互等待和唤醒。这两点协程都不需要。





#### 举例

```c++
// 实现一个继承自ExampleService的服务类
// example::ExampleService由protobuf生成
class ExampleServiceImple : public example::ExampleService {
    public:
    // 实现SayHello方法
    void SayHello(google::protobuf::RpcController* cntl_base,
                  const example::Request* request,
                  example::Response* response,
                  google::protobuf::Closure* done) {
        // 从Controller中获取请求信息
        brpc::Controller* cntl = static_cast<brpc::Controller*>(cntl_base);
        // 设置响应信息
        response->set_reply("Hello " + request->message());
        response->set_request_id(request->message());
        // 调用done->Run()表示响应已经完成
        done->Run();
    }

};
int main() {
    brpc::Server server;
    ExampleServiceImple service_impl;
    // 注册服务,可以注册多个，
    if (server.AddService(&service_impl,
                          brpc::SERVER_DOESNT_OWN_SERVICE) != 0) {
        LOG(ERROR) << "Fail to add service";
        return -1;
    }
    brpc::ServerOptions options;
    options.idle_timeout_sec = FLAGS_idle_timeout_s;
    // options.enabled_protocols = "baidu_std_reuse";
    // 设置监听地址
    if (server.Start(12345, &options) != 0) {
        LOG(ERROR) << "Fail to start EchoServer";
        return -1;
    }
    // 服务启动后，会一直运行，直到调用server.stop(0)停止服务
    server.RunUntilAskedToQuit();
    return 0;
}


```

##### 梳理流程

<!-- more -->

###### AddService

这里需要add的service是由protobuf实现的service,protobuf只是定义了这个service,并没有具体的实现，业务代码要实现具体的rpc 接口

![image-20240422151442717](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/image-20240422151442717.png)

##### 实际场景

假如王二狗和牛翠花约好在天安门约会，两人从起点到目的地可以借助的通行方式比较多，至于选那种方式完全因人而异。

##### 使用方法（核心代码）

```c++
class Context；
```



#### 

#### 问题

* google::protobuf::Closure* done->Run()的作用？

Server端的done的逻辑主要是发送response回client，当其发现用户调用了SetFailed()后，会把错误信息送回client。client收到后，它的Controller::Failed()会为true（成功时为false），Controller::ErrorCode()和Controller::ErrorText()则分别是错误码和错误信息。



<!-- more -->
