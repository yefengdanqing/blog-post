---
title: docker了解
toc: true
date: 2022-08-1822:26:59
tags:
- docker
categories:
- 工具
typora-root-url: ..\img
---

### Docker了解

<!-- more -->

### 基本概念

**镜像**（`Image`）

一堆压缩在一起的静态软件包

**容器**（`Container`）

这堆静态包执行的过程

**仓库**（`Repository`）

镜像的存放地方

#### 命令

###### docker stop $(docker ps -a -q)

停止所有容器运行

###### docker rm $(docker ps -a -q)

删除所有停止运行的容器

###### **docker ps -a -q** 

- `docker ps` 列出容器。
- `-a` 这个选项用于列出所有容器，包括停止运行的。如果没有这个选项，则默认只列出在运行的容器。
- `-q` 这个选项列出容器的数字 ID，而不是容器的所有信息。

###### docker rmi 

删除镜像

###### docker commit

Docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。以后我们运行这个新镜像的时候，就会拥有原有容器最后的文件变化。

`docker commit` 的语法格式为：

```dockerfile
docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
//会记录下所有的改动，可能会导致冗余
```



###### docker build

```dockerfile
FROM amazonlinux:2.0.20200722.0
RUN yum install -y epel-releas jq initscripts sudo nc gdb net-tools make vim lsof which tar tree              \
    procps python3 openssh-clients unzip && pip3 install requests awscli supervisor pipenv
//docker build -f docker_file -t sunketo-redhat:v0.0.1 .
```

###### docker push

```docker
docker push yefengdanqing/sunketo-redhat:v0.0.1
```

必须先要docker tag，然后才能带上用户名再进行push

```docker
docker tag sunketo-redhat:v0.0.1 yefengdanqing/sunketo-redhat:v0.0.1
```



```do
docker ps #查看运行的docker
docker run -t #绑定一个伪终端运行;-i则让容器的标准输入保持打开，可以进行交互;-d Run container in background and print container ID,-v绑定一个目录，-w在容器里面的
docker exec 进入一个以backgroud方式运行的容器

```



<!-- more -->



sudo chmod 666 /var/run/docker.sock

第一步：sudo gpasswd -a username docker  #将普通用户username加入到docker组中，username这个字段也可以直接换成$USER。

第二步：newgrp docker  #更新docker组

第三步：再执行你报错的命令，此时就不会报错了。
————————————————
版权声明：本文为CSDN博主「术业还未专攻」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/BaoITcore/article/details/127736052
