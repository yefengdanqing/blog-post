---
title: redis--哨兵
toc: true
date: 2022-02-22 23:50:59
tags:
- redis
categories:
- 组件
typora-root-url: ..\img
---

###### **目的**

解决主从挂了只能手动去拉起的问题，哨兵是redis高可用（ha)的方案。

###### **Sentinel**

启动：sentinel是一个redis server，只是他有自己特有的代码，相当于一个定制化的redis，用于监控 

命令

```
struct redisCommand sentinelcmds[] = {
{"ping",pingCommand,1,"",0,NULL,0,0,0,0,0}, 
{"sentinel",sentinelCommand,-2,"",0,NULL,0,0,0,0,0}, 
{"subscribe",subscribeCommand,-2,"",0,NULL,0,0,0,0,0}, 
{"unsubscribe",unsubscribeCommand,-1,"",0,NULL,0,0,0,0,0}, 
{"psubscribe",psubscribeCommand,-2,"",0,NULL,0,0,0,0,0},
{"punsubscribe",punsubscribeCommand,-1,"",0,NULL,0,0,0,0,0}, 
{"info",sentinelInfoCommand,-1,"",0,NULL,0,0,0,0,0} };
```

<!-- more -->

结构

```c++
struct sentinelState { 
    // 当前纪元，用于实现故障转移 
    uint64_t current_epoch; 
    // 保存了所有被这个sentinel 监视的主服务器 
    // 字典的键是主服务器的名字 
    // 字典的值则是一个指向sentinelRedisInstance 结构的指针 
    dict *masters; // 是否进入了TILT 模式？int tilt; 
    // 目前正在执行的脚本的数量 
    int running_scripts;
    // 进入TILT 模式的时间 mstime_t tilt_start_time;
    // 最后一次执行时间处理器的时间 
    mstime_t previous_time; 
    // 一个FIFO 队列，包含了所有需要执行的用户脚本 
    list *scripts_queue; 
} sentinel;
```

```c++
typedef struct sentinelRedisInstance { 
    // 标识值，记录了实例的类型，以及该实例的当前状态 
    int flags; 
    // 实例的名字 
    // 主服务器的名字由用户在配置文件中设置 
    // 从服务器以及Sentinel 的名字由Sentinel 自动设置
    // 格式为ip:port ，例如"127.0.0.1:26379" 
    char *name; // 实例的运行ID char *runid;
    // 配置纪元，用于实现故障转移 
    uint64_t config_epoch; 
    // 实例的地址 
    sentinelAddr *addr; 
    // SENTINEL down-after-milliseconds 选项设定的值 
    // 实例无响应多少毫秒之后才会被判断为主观下线（subjectively down ） mstime_t down_after_period; 
    // SENTINEL monitor <master-name> <IP> <port> <quorum> 选项中的quorum 参数 
    // 判断这个实例为客观下线（objectively down ）所需的支持投票数量 
    int quorum; 
    // SENTINEL parallel-syncs <master-name> <number> 选项的值
    // 在执行故障转移操作时，可以同时对新的主服务器进行同步的从服务器数量 
    int parallel_syncs;
    // SENTINEL failover-timeout <master-name> <ms> 选项的值// 刷新故障迁移状态的最大时限 
    mstime_t failover_timeout; 
    // ... 
} sentinelRedisInstance;
```

```shell
##################### # master1 configure # ##################### 
#master1-->name
sentinel monitor master1 127.0.0.1 6379 2
sentinel down-after-milliseconds master1 30000
sentinel parallel-syncs master1 1
sentinel failover-timeout master1 900000
##################### # master2 configure # #####################
sentinel monitor master2 127.0.0.1 12345 5 
sentinel down-after-milliseconds master2 50000 
sentinel parallel-syncs master2 5 
sentinel failover-timeout master2 450000
```

- 配置含义
  - down-after-milliseconds:redis server多少秒后无响应，代表主观下线
  - SENTINEL monitor <master-name> <IP> <port> <quorum>，quorum标识客观下线的票数
- 创建连接
  - 命令连接：通过异步方式，命令连接用于向主服务器发送命令，并接主服务器的返回
  - 订阅连接：异步，订阅主服务器的__sentinel__:hello频道【数据信息没有保存在内存中，所以要单独建立；通过订阅链接可以得知其他sentinel信息】

###### **Sentinel监控流程**

- 获取主服务器信息：sentinel每10s(秒）一次向主服务器发送info命令，sentinel根据返回信息能拿到主服务器和从服务器的信息。当sentinel根据当前保存的name【指定】信息能知道当前是否有新的从服务器，或者服务器是否重启了【主服务器是启动sentinel的时候手动配置的name等信息,主服务器实例结构的name属性的值是用户使用Sentinel配置文件设置的，而从服务器实 例结构的name属性的值则是Sentinel根据从服务器的IP地址和端口号自动设置的】。


- 获取从服务器信息：通过上一步得到从服务器信息的时候，sentinel会建立和从服务器的命令连接和订阅连接。之后以10s（秒）一次的频率向从服务发送info命令


- 发送信息：
  - 2s(秒)一次通过命令连接向主服务器和从服务器发送信息
  - PUBLISH __sentinel__:hello "<s_ip>,<s_port>,<s_runid>, <s_epoch>,<m_name>,<m_ip>,<m_port>,<m_epoch>"

- 订阅信息：
  - 通过订阅链接订阅；SUBSCRIBE __sentinel__:hello
  - 每个sentinel会根据频道获得其他sentinel的信息
  - Sentinel更新自己的sentinel 对象信息，如果存在更新；不存在新建。
  - 创建向其他sentinel的命令链接

- 检测主观下线：【领头sentinel】每秒向所有的主、从、sentinel 发送ping命令并根据得到的结果判断对应的实例是否在线。
  - 返回有效结果：+PONG、-LOADING、+MASTERDOWN的正常返回
  - 其他为无效结果
  - 如果在配置的下线时间内一直返回无效结果，则这个server对应的结构信息的flags设置为SIR_S_DOWN。
  - 多个sentinel设置的主观下线的时间不一样
- 检测客观下线流程：
  - 当一个sentinel （源）判断一个主服务器下线后，会向其他sentinel（目的）询问是否下线，发送以下命令
    - eg1：SENTINEL is-master-down-by- addr <ip> <port> <current_epoch> <runid>
    - eg2：SENTINEL is-master-down-by-addr 127.0.0.1 6379 0 *
  - 向源Sentinel返回一条包含三个参数 的Multi Bulk回复作为SENTINEL is-master-down-by命令的回复<down_state><leader_runid><leader_epoch>；eg:1  * 0
  - 当Sentinel得到的下线回复数大于配置的quorum数的时候，将flags置为SIR_O_DOWN
  - 每个sentinel对下线的认知不同，其实就是配置内容的配置项的不同（下线个数和时间）

###### **选举领头Sentinel**

- 背景：当sentinel发现一个server处于客观下线的时候，监视这个server的sentinel就会触发领头Sentinel选举


- 选举过程:参见raft


###### **故障转移**

- 领头Sentinel挑选从下线主服务器的从服务器中挑选一个当做新的主服务器

  - 在下线主服务器的从服务器列表中删除已下线或者断开的从服务器项


  - 删除最近5s(秒）内没有回复领头sentinel的从服务器


  - 删除那些与主服务器连接断开超过down-after-milliseconds*10毫秒的从服务器


  - 以上都是为了保证最新


  - 将以上从服务器列表根据优先级进行排序，如果优先级一样；再按照偏移量进行排序(最大的偏移量）；否则根据运行id进行排序，最小的是最新的


  - 领头Sentinel向被选中的从服务器发送salve of one,让其成为主服务器


  - 领头Sentinel在故障迁移时以1s(秒）的频率发送info，检查role是否为master


- 让下线主服务器的从服务器复制新的主服务器

  - 发送salveof命令复制新的主服务器


- 如果这个下线主服务器重新上线，将（salveof)成为这个新主服务器的从服务器


问题汇总：

哨兵节点挂了怎么办
