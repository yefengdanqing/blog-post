---
title: kafka概念
date: 2022-02-17 18:42:47
tags:
- kafka
categories: 
- 组件
toc: true
typora-root-url: ..\img
---
- ISR、OSR、AR
  - ISR：In-Sync Replicas 副本同步队列
  - OSR：Out-of-Sync Replicas
  - AR：Assigned Replicas 所有副本
  - ISR是由leader维护，follower从leader同步数据有一些延迟（具体可以参见 图文了解 Kafka 的副本复制机制），超过相应的阈值会把 follower 剔除出 ISR, 存入OSR（Out-of-Sync Replicas ）列表，新加入的follower也会先存放在OSR中。AR=ISR+OSR。

- Kafka的日志目录结构

  - 每个partition一个文件夹，包含四类文件.index .log .timeindex leader-epoch-checkpoint

    <!-- more -->

  - .index .log .timeindex 三个文件成对出现，前缀为上一个segment的最后一个消息的偏移；log文件中保存了所有的具体消息；index文件中保存了稀疏的相对偏移的索引；timeindex保存的则是时间索引；通过2分查找；

  	| offset1 | 消息在log中的物理磁盘偏移1 |
    | ------- | -------------------------- |
    | offset2 | 消息在log中的物理磁盘偏移2 |
    
  - leader-epoch-checkpoint中保存了每一任leader开始写入消息时的offset 会定时更新 follower被选为leader时会根据这个确定哪些消息可用

- Kafka的control

  - 作用：通过zookeeper管理和协调整个kafka集群
  - 选择：第一个在zk上创建临时节点的broker，
  - 功能：
    - 创建topic
    - 分区重分配
    - preferred领导选举
    - 集群成员管理
    - 数据传输


Kafka的特性:⾼吞吐量、低延迟，可扩展，持久性、可靠性，容错性，⾼并发kafka消息格式转换 只会在Broker端发⽣了消息格式转换。1.如果⽣产者 和 Kafka 集群版本的消息格式不⼀致，（因为在Kafka更新的过程中，进⾏了⼀次消息格式的修改，）那 么 Broker端为了兼容考虑， 会将⽣产者的消息格式修改为当前版本的消息格式， ⽽转换消息格式是必然涉及 解压缩 和 重压缩的2. 如果消费者版本和Kafka 集群版本不⼀致，那么Broker也会进⾏消息格式转换 在partition中如何通过offffset查找message1. 根据offffset ，⼆分查找⽂件列表，就可以快速定位到index⽂件（index⽂件采⽤稀疏索引存储⽅式）2. 根据offffset，定位到index的元数据物理位置和log的物理偏移地址3. 在稠密索引中，⽂件中的每个搜索码值都对应⼀个索引值，在稀疏索引中，只为索引码的某些值建⽴索引项。为什么 kafka 要⽤ page cache1. 如果使⽤ JVM 来管理这些内存 参考[1.a]1. 对象头会带来很多空间浪费 2. 过⼤的堆会让 JVM GC 负担太重, 影响回收效率 3. 当重启 kafka 服务时, 内存上重建缓存10GB 的告诉缓存可能需要很长时间 参考[1]1. ⽽使⽤ page cache , 即使重启 kafka 也能够直接利⽤其中的缓存 2. 使⽤ page cache 的另外⼀个好处就是可以使⽤ Zero-Copy 零拷贝 1. zero-copy 可以降低系统调⽤的次数 (减少内核态/⽤户态上下⽂切换) 2. 可以减少在不同缓冲区之间的 copying3. 如果Kafka producer的⽣产速率与consumer的消费速率相差不⼤，那么就能⼏乎只靠对broker page cache的 读写完成整个⽣产-消费过程, 所有的数据都在内存中,这是 kafka 的⾼吞吐量的保证 参考[2.a]4. 这也是⼀个权衡的⽅案, 能够在开发量适当的情况下, 尽可能地提⾼IO效率 参考[5.a]关于零拷贝：producer⽣产消息时，会使⽤pwrite()系统调⽤【对应到Java NIO中是FileChannel.write() API】按偏移量写⼊数据，并 且都会先写⼊page cache⾥。consumer消费消息时，会使⽤sendfifile()系统调⽤【对应FileChannel.transferTo()API】，零拷贝地将数据从page cache传输到broker的Socket buffffer，再通过⽹络传输。注意事项：1. 对于单纯运⾏Kafka的集群⽽⾔，⾸先要注意的就是为Kafka设置合适（不那么⼤）的JVM堆⼤⼩。从上⾯的分析可 知，Kafka的性能与堆内存关系并不⼤，⽽对page cache需求巨⼤。根据经验值，为Kafka分配5~8GB的堆内存就已经⾜ ⾜够⽤了，将剩下的系统内存都作为page cache空间，可以最⼤化I/O效率。2. 另⼀个需要特别注意的问题是lagging consumer，即那些消费速率慢、明显落后的consumer。它们要读取的数据 有较⼤概率不在broker page cache中，因此会增加很多不必要的读盘操作。⽐这更坏的是，lagging consumer读取 的“冷”数据仍然会进⼊page cache，污染了多数正常consumer要读取的“热”数据，连带着正常consumer的性能变差。在⽣产环境中，这个问题尤为重要。3. 同时如果有其他进程申请内存，会回收抢占⼀部分PageCache，但也会导致Kafka吞吐量下降会不稳定https://my.oschina.net/vivotech/blog/4524883https://giraffffetree.me/2020/11/16/Kafka-%E8%AE%BE%E8%AE%A1%E6%80%9D%E6%83%B3-%E9%A1%B5%E9%9D%A2%E7%BC%93%E5%AD%98-page-cache/https://cloud.tencent.com/developer/article/1488144https://shiyueqi.github.io/2017/04/27/Kafka-Pagecache%E5%8E%9F%E7%90%86/kafka如何保证可靠性1. topic分区副本Kafka 可以保证单个分区⾥的事件是有序的，在众多的分区副本⾥⾯有⼀个副本是 Leader，其余的副本是 follower，所 有的读写操作都是经过 Leader 进⾏的，同时 follower 会定期地去 leader 上的复制数据。当 Leader 挂了的时候，其中 ⼀个 follower 会重新成为新的 Leader。通过分区副本，引⼊了数据冗余，同时也提供了 Kafka 的数据可靠性。2. Producer 往 Broker 发送消息根据实际的应⽤场景，我们设置不同的 acks，以此保证数据的可靠性。3. kafka副本同步机制https://mp.weixin.qq.com/s?__biz=MzA5MTc0NTMwNQ==&mid=2650716970&idx=1&sn=3875dd83ca35c683bfa42135c55a03ab&chksm=887da65cbf  何时产⽣压缩 1. ⽣产者 为了数据在传输到 Kafka 可以更快， 那么在⽣产者启动压缩⾃然是很正常的。2. Broker端 Broker 主要是负责储存数据， 压缩能够很好的减少磁盘的占⽤。⼀般情况⽽⾔， 如果数据已经在 ⽣产 者端压缩了， 那么其实就不需要在Broker端再做处理， 实际上也确实是这样， 但是如果发⽣以下这些情况， 那么Broker端会再进⾏压缩， 这样⽆疑会导致性能问题， 所以应该尽量避免：Broker端指定了和Producer端不同的压缩算法， 这很好理解，因为压缩算法不⼀致， Broker 就需要解压 缩，并在此压缩成设定好的算法， 所以⼀定要避免这种情况。Broker端发⽣了消息格式转换。这⾥所谓的消息格式转换，是因为在Kafka更新的过程中，进⾏了⼀次消息格式的修改， 如果⽣产者 和 Kafka 集群版本的消息格式不⼀致， 那么 Broker端为了兼容考虑， 会将 ⽣产 者的消息格式修改为当前版本的消息格式， ⽽转换消息格式是必然涉及 解压缩 和 重压缩的， 何时解压缩？1. Consumer端 消费数据⾃然需要将数据解压缩，这个没什么好说的。2. Broker端 这⾥可能你要奇怪了， 为什么Broker端还要解压缩呢？实际上Broker端只是为了进⾏消息的校检， 以 保证数据的正确性， 这样必然会给Broker端的性能带来⼀定的影响， 但是就⽬前来说，好像也没什么好的解决办 法。当kafka遇到如下四种情况的时候，kafka会触发消费组Rebalance机制：1. 消费组成员发⽣了变更，⽐如有新的消费者加⼊了消费组组或者有消费者宕机 2. 消费者⽆法在指定的时间之内完成消息的消费 3. 消费组订阅的Topic发⽣了变化 4. 订阅的Topic的partition发⽣了变化kafka的zookeeper存储结构kafka的选举机制1. 控制器（Broker）选主1. 集群中第⼀个启动的broker会通过在zookeeper中创建临时节点/controller来让⾃⼰成为控制器，其他broker启动 时也会在zookeeper中创建临时节点，但是发现节点已经存在，所以它们会收到⼀个异常，意识到控制器已经存在，那么 就会在zookeeper中创建watch对象，便于它们收到控制器变更的通知。2. 集群中每选举⼀次控制器，就会通过zookeeper创建⼀个 controller epoch ，每⼀个选举都会创建⼀个更⼤，包含最新信息的 epoch ，如果有broker收到⽐这个 epoch 旧的数据，就会忽略它们，kafka也通过这个 epoch 来防⽌集群产⽣“脑 裂”。3. 如果有⼀个broker加⼊集群中，那么控制器就会通过Broker ID去判断新加⼊的broker中是否含有现有分区的副本， 如果有，就会从分区副本中去同步数据。4. 如果集群中有⼀个broker发⽣异常退出了，那么控制器就会检查这个broker是否有分区的副本leader，如果有那么这个分区就需要⼀个新的leader，此时控制器就会去遍历其他副本，决定哪⼀个成为新的leader，同时更新分区的ISR集 合。5. 负责删除topic以及副本重分配2. 分区多副本选主 ⾸领副本（leader）：也就是leader主副本，每个分区都有⼀个⾸领副本，为了保证数据⼀致性，所有的⽣产者与消费者 的请求都会经过该副本来处理。跟随者副本（follower）：除了⾸领副本外的其他所有副本都是跟随者副本，跟随者副本不处理来⾃客户端的任何请求， 只负责从⾸领副本同步数据，保证与⾸领保持⼀致。如果⾸领副本发⽣崩溃，就会从这其中选举出⼀个leader。⾸选⾸领副本：创建分区时指定的⾸选⾸领。如果不指定，则为分区的第⼀个副本。1. 第⼀次选主，Kafka会将副本中清单AR⾥的第⼀个同步副本（preferred replica）选为⾸领2. 当主异常挂了，会从isr列表中选择⼀个作为主，如果isr列表⾥没有副本，那么检查是否允许允许脏主选举，允许的 话，会从ar列表⾥选择⼀个作为主3. 消费组选主在kafka的消费端，会有⼀个消费者协调器以及消费组，组协调器GroupCoordinator需要为消费组内的消费者选举出⼀ 个消费组的leader如果消费组内还没有leader，那么第⼀个加⼊消费组的消费者即为消费组的leader如果某⼀个时刻leader消费者由于某些原因退出了消费组，那么就会重新选举leader，map中的第⼀个消费者作为新的leaderkafka同步机制 写是都往leader上写，读也只在leader上读，flflower只是数据的⼀个备份，保证leader被挂掉后顶上来，并不往外提供 服务。Follower通过拉取的⽅式从Leader中同步数据。消费者和⽣产者都是从Leader中读取数据，不与Follower交互。利⽤ISR列表的机制实现了1. 当⼀个⽣产者要⽣产⼀个消息到topic中的某个partition中时，这条消息⾸先会被分区的leader副本追加到它的log中2. follower副本持续的从leader上拉去消息，follower从leader同步数据有⼀些延迟（超过延迟时间replica.lag.time.max.ms）后会被踢出ISR列表3. 每个follower都有⼀个LEO，取⼀个partition对应的ISR中最⼩的LEO作为HW，HW之后的数据消费者看不到，HW之 前的消息成为已提交的消息，HW也会持续的⼴播给followers，followers持久化到磁盘，恢复时⽤。4. 当⼀个失败的replica重启后，⾸先读取磁盘中的HW然后将log截断到HW，然后成为follower开始向leader拉去HW之 后的消息，⼀旦追赶上leader的LEO，这个replica⼜重新被加⼊到ISRhttps://javazhiyin.blog.csdn.net/article/details/115258060https://www.kancloud.cn/nicefo71/kafka/1470863https://www.jianshu.com/p/7008d2a1e320https://github.com/Snailclimb/JavaGuide/blob/master/docs/system-design/distributed-system/messagequeue/Kafka%E5%B8%B8%E8%A7%81%E9%9D%A2%E8%AF%95%E9%A2%98%E6%80%BB%E7%BB%93.md
