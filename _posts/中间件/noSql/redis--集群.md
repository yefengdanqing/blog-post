---
title: redis--集群
toc: true
date: 2022-02-22 22:26:59
tags:
- redis
categories:
- 组件
typora-root-url: ..\img
---

###### **节点**

- 定义：每一个redis server当做是一个集群的节点

- 启动：在配置文件中通过cluster_enabled为yes

- 添加方式：cluster meet ip port（copy）
  - 节点A会为节点B创建一个clusterNode结构，并将该结构添加到自己的 clusterState.nodes字典里面。2）之后，节点A将根据CLUSTER MEET命令中给定的IP地址和端口号，向节点B发送一条 MEET消息（message）。 
  
  - 如果一切顺利，节点B将接收到节点A发送的MEET消息，节点B会为节点A创建一个clusterNode结构，并将该结构添加到自己的clusterState.nodes字典里面。 
  
  - 之后，节点B将向节点A返回一条PONG消息。
  
  - 如果一切顺利，节点A将接收到节点B返回的PONG消息，通过这条PONG消息节点A可以知道节点B已经成功地接收到了自己发送的MEET消息。 

  - 之后，节点A将向节点B返回一条PING消息。 
  
  - 如果一切顺利，节点B将接收到节点A返回的PING消息，通过这条PING消息节点B可以知道节点A已经成功地接收到了自己返回的PONG消息，握手完成。
  
  - 节点A会将节点B的信息通过Gossip协议传播给集群中的其他节点，让其他节点认识B【raft、pa'xos】
  
    <!-- more -->
  
- 数据结构

  ```c++
  struct clusterNode { 
      // 创建节点的时间 mstime_t ctime; 
      // 节点的名字，由40 个十六进制字符组成 
      // 例如68eef66df23420a5862208ef5b1a7005b806f2ff 
      char name[REDIS_CLUSTER_NAMELEN]; 
      // 节点标识 
      // 使用各种不同的标识值记录节点的角色（比如主节点或者从节点）， 
      // 以及节点目前所处的状态（比如在线或者下线）。
      int flags; 
      // 节点当前的配置纪元，用于实现故障转移 
      uint64_t configEpoch; 
      // 节点的IP 地址char ip[REDIS_IP_STR_LEN]; 
      // 节点的端口号 int port; 
      // 保存连接节点所需的有关信息 
      clusterLink *link; // ...
  };
  ```

  ```c++
  //clusterNode结构的link属性是一个clusterLink结构，该结构保存了连接节点所需的有关信 息，比如套接字描述符，输入缓冲区和输出缓冲区：
  typedef struct clusterLink { 
      // 连接的创建时间 
      mstime_t ctime; 
      // TCP 套接字描述符 
      int fd; // 输出缓冲区，保存着等待发送给其他节点的消息（message ）。
      sds sndbuf; 
      // 输入缓冲区，保存着从其他节点接收到的消息。
      sds rcvbuf; 
      // 与这个连接相关联的节点，如果没有的话就为NULL 
      struct clusterNode *node; 
  } clusterLink;
  ```

###### **槽**

- 槽数：16384（2048*8或者2k*8)


- 状态：如果16384个槽都有节点处理，那就是处于上线状态(ok);否则 下线（failed)


- 分配方式：CLUSTER ADDSLOTS

- ```
  struct clusterNode {
  // ... 
  unsigned char slots[16384/8];
  int numslots;
  //...
  };
  ```

  

- 每个节点都保存了所有的槽信息，只要数据到了任何一个节点，都能一次得到这个数据保存在哪个槽上

- 在执行命令时，如果key对应的value不在当前节点，则返回moved语义；


###### **重新分片**：Redis的集群管理软件redis-trib负责执行

###### **故障转移**

- 复制设置从节点：salveof replicate ip port


- 故障检测：周期性的发送ping命令，如果ping命令没有在规定的时间内回复，则认为这个主节点疑似下线，只有半数以上负责处理槽的主节点认为这个节点疑似下线。


- 故障转移：   当一个从节点发现自己正在复制的主节点进入了已下线状态时，从节点将开始对下线主节点进行故障转移，以下是故障转移的执行步骤： 

  - 复制下线主节点的所有从节点里面，会有一个从节点被选中。 


  - 被选中的从节点会执行SLAVEOF no one命令，成为新的主节点。 


  - 新的主节点会撤销所有对已下线主节点的槽指派，并将这些槽全部指派给自己。 


  - 新的主节点向集群广播一条PONG消息，这条PONG消息可以让集群中的其他节点立即知道这个节点已经由从节点变成了主节点，并且这个主节点已经接管了原本由已下线节点 负责处理的槽。


  - 新的主节点开始接收和自己负责处理的槽有关的命令请求，故障转移完成。


- 从节点选举：raft协议


###### **消息：ping、pong、fail**
