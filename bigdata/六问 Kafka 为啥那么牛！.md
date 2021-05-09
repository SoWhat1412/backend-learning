![](https://img-blog.csdnimg.cn/20210122214438783.png)
![Kafka架构](https://img-blog.csdnimg.cn/2021050917414331.png)

# 1 Kafka 简介
### 1.1 Kafka 概述

Kafka 是一个**分布式**的基于发布/订阅模式的消息队列，依靠其强悍的吞吐量，Kafka 主要应用于大数据实时处理领域。在数据采集、传输、存储的过程中发挥着举足轻重的作用。
1. Apache Kafka 由 **Scala** 写成，是由Apache软件基金会开发的一个开源消息系统项目。该项目的目标是为处理实时数据提供一个统一、高通量、低等待的平台。
2. Kafka 是一个分布式消息队列，Kafka 对消息保存时根据 Topic 进行归类，Kafka 集群有多个 Kafka实例组成，每个实例 Server 称为 broker。
3. 无论是 Kafka 集群还是 Consumer 都依赖于 ZooKeeper 集群保存一些 meta 信息，来保证系统可用性

### 1.2 Kafka 优点
1. [支持](https://blog.csdn.net/A529810922/article/details/104009512/)多个 生产者 和 消费者
1. 支持broker的横向拓展
1. 副本集机制，实现数据冗余，保证数据不丢失
1. 通过 topic 将数据进行分类
1. 通过**分批**发送压缩数据的方式，减少数据传输开销，提高吞高量
1. 支持多种模式的消息，消息是基于磁盘实现数据的持久化。
1. 高性能的处理信息，在大数据的情况下，可以保证亚秒级的消息延迟
1. 一个消费者可以支持多种 topic 的消息
1. 对CPU、内存、网络的消耗比较小
1. 支持跨数据中心的数据复制 跟 镜像集群
### 1.3 Kafka 缺点
1. 由于是批量发送，所以数据达不到真正的实时
1. 只能支持统一分区内消息有序，无法实现全局消息有序
1. 监控不完善，需要安装插件
1. 会丢失数据，并且不支持事务
1. 可能会**重复消费**数据，消息会乱序，可用保证一个固定的partition内部的消息是有序的，但是一个topic有多个partition的话，就不能保证有序了，需要zookeeper的支持，topic一般需要人工创建，部署和维护一般都比mq高
### 1.4 Kafka 架构
![Fafka架构图](https://img-blog.csdnimg.cn/20210121143446678.png)



1. **Broker**：一台 kafka 服务器就是一个broker。一个集群由多个broker组成。一个broker可以容纳多个topic。
2. **Producer** ：消息生产者，就是向 Kafka broker 发消息的客户端。
3. **Consumer** ：消息消费者，向 Kafka broker 拉取消息来消费。可以根据 Consumer 的消费能力以适当的速率消费消息。
4. **Topic** ：可以理解为一个队列，生产者和消费者面向的都是一个topic。
5. **Partition**：为了实现扩展性，一个非常大的 topic 可以分布到多个broker上，一个 topic 可以分为多个partition，每个 partition 是一个有序的队列，有点平衡分摊生产者机制。
6. **Replication**：为保证集群中的某个节点发生故障时，该节点上的partition数据不丢失，且kafka仍然能够继续工作，kafka提供了副本机制，一个topic的每个分区都有若干个副本，一个leader和若干个follower。
7. **leader**：一个分区有一个Leader，生产者发送数据的对象，以及消费者消费数据的对象都是leader。
8. **follower**：一个分区有一个Follower，实时从 leader 中同步数据，保持和 leader 数据的同步。leader 发生故障时，某个 follower 会成为新的 follower。注意Kafka中 副本数不能超过Broker数！
9. **Consumer Group** ：消费者组由多个 consumer 组成。组内每个消费者负责消费不同分区的数据，一个分区只能由一个组内消费者消费；消费者组之间互不影响。所有的消费者都属于某个消费者组，即消费者组是逻辑上的一个订阅者。
10. **offset**：消费者在具体消费某个 topic 中的消息时，可以指定起始偏移量。
### 1.5 ZooKeeper 作用
[ZooKeeper](https://www.jianshu.com/p/a036405f989c) 在 Kafka 中有举足轻重的地位，一般提供如下功能：
##### 1.5.1 Broker 注册
Broker 是分布式部署并且相互之间相互独立，但是需要有一个注册系统能够将整个集群中的Broker管理起来，比如用ZooKeeper。
##### 1.5.2 Topic注册
在 Kafka 中同一个 Topic 的消息会被分成多个 Partition 并将其分布在多个 Broker 上，这些 Partition 信息及与 Broker 的对应关系也都是由 Zookeeper 在维护，由专门的节点来记录。
##### 1.5.3 生产者负载均衡
同一个Topic消息会被分区并将其分布在多个Broker上，因此，生产者需要将消息合理地发送到这些分布式的Broker上。
1. 老式的四层负载均衡，根据生产者的IP地址和端口来为其确定一个相关联的Broker。一般一个生产者只会对应单个Broker，但实际系统中的每个生产者产生的消息量及每个Broker的消息存储量都是不一样的。
2. 使用 Zookeeper 进行负载均衡，由于每个Broker启动时，都会完成Broker注册过程，生产者会通过该节点的变化来动态地感知到Broker服务器列表的变更，这样就可以实现动态的负载均衡机制。
##### 1.5.4 消费者负载均衡
Kafka 中的消费者同样需要进行负载均衡来实现多个消费者合理地从对应的 Broker 服务器上接收消息，每个消费者分组包含若干消费者，`每条消息都只会发送给分组中的一个消费者`，不同的消费者分组消费自己特定的Topic下面的消息，互不干扰。
##### 1.5.5 分区 与 消费者 的关系
Kafka 会为每个 Consumer Group 分配个全局唯一 Group ID，Group 内的 Consumer 共享该 ID，Kafka规定 每个partition信息只能被同组的一个Consumer 消费，在Zk中记录partition 跟 Consumer关系，每个消费者一旦确定了对一个消息分区的消费权力，需要将其Consumer ID 写入到 Zookeeper 对应消息分区的临时节点上。
##### 1.5.6 消息消费进度 Offset 记录
Consumer 对指定消息分区进行消费的过程中，需要定时地将分区消息的消费进度 Offset 记录到 Zookeeper 上，以便在该 Consumer 进行重启或者其他 Consumer 重新接管该消息分区的消息消费后，能够从之前的进度开始继续进行消息消费。
##### 1.5 7 消费者注册
为让同一个 Topic 下不同分区的消息尽量均衡地被多个 Consumer 消费而进行 Consumer 与消息分区分配的过程。
1. Consumer 启动后在ZK下创建个节点，并且每个 Consumer 会对 Consumer Group 中的 Consumer 的变化注册监听，目的是为了保证 Consumer 负载均衡。
2. Consumer 会对Broker列表监听，发生变化会进行 Consumer 负载均衡。

# 2 Kafka 生成过程
### 2.1 写入方式
producer采用  push 模式将消息发布到 broker，每条消息都被 append 到 patition 中，属于 **顺序写磁盘** ，顺序写比随机写要起码提速3个数量级！
### 2.2 分区 Partition
##### 2.2.1  Partition 简介
消息发送时都被发送到一个topic，其本质就是一个目录，而topic是由一些 分区日志 Partition Logs 组成，其组织结构如下图所示：
![Partition发生](https://img-blog.csdnimg.cn/20210121152949639.png)
可以看到每个 Partition 中的消息都是有序的，生产的消息被不断追加到 Partition log 上，其中的每一个消息都被赋予了一个唯一的 offset 值。
![消费者](https://img-blog.csdnimg.cn/20210121153238183.png)
通过分区可以 方便在集群中扩展，可以提高并发。

形象理解：
> Kafka 的设计源自生活，好比为公路运输，不同的起始点和目的地需要修不同高速公路（主题），高速公路上可以提供多条车道（分区），流量大的公路(主题)多修几条车道(分区)保证畅通，流量小的公路少修几条车道避免浪费。收费站好比消费者，车多的时候多开几个一起收费避免堵在路上，车少的时候开几个让汽车并道就好了。 

##### 2.2.2 分区原则
我们需要将producer发送的数据封装成一个ProducerRecord对象。![数据封装](https://img-blog.csdnimg.cn/20210121192004209.png)
1. 指明 partition 的情况下，直接将指明的值直接作为 partiton 值。
2. 没有指明 partition 值但有 key 的情况下，将 key 的 hash 值与 topic 的 partition 数进行取余得到 partition 值。
3. 既没有 partition 值又没有 key 值的情况下，第一次调用时随机生成一个整数（后面每次调用在这个整数上自增），将这个值与 topic 可用的 partition 总数取余得到 partition 值，也就是常说的 **round-robin** 算法。
### 2.3 Kafka 文件存储机制
![Fafka存储结构](https://img-blog.csdnimg.cn/20210121164110664.png)
1. Kafka 中消息是以 topic 进行分类的，生产者跟消费者都是面向 topic 的，topic 只是逻辑上的概念，而 Partition 是物理上的概念，每个 Partition 对应个 log 文件，每个分区用 `*.index`存放数据索引，`*.log`存储数据。index文件中的元数据指向对应log文件中Message的物理偏移地址(参考 **kaldi**、**Neo4j**)。
2. 为防止 log 文件过大导致数据定位效率低下，Kafka采取了`分片`和`索引`机制，将每个partition分为多个**segment**。每个 segment 对应`*.index`跟`*.log`。这些文件位于一个文件夹下，该文件夹的命名规则为：topic名称 + 分区序号。例如 first 这个 topic 有三个分区，则其对应的文件夹为first-0、first-1、first-2。
```java
00000000000000000000.index
00000000000000000000.log
00000000000000170410.index
00000000000000170410.log
00000000000000239430.index
00000000000000239430.log
```
注意：index 和 log 文件以当前segment的第一条消息的 offset 命名。

![数据查找过程](https://img-blog.csdnimg.cn/20210121171721940.png)
### 2.4 如何保证消息顺序执行
##### 2.4.1 顺序错乱
1. Kafka 一个 topic，一个 partition，一个 Consumer，但是 Consumer 内部进行多线程消费，这样数据也会出现顺序错乱问题。![多线程消费](https://img-blog.csdnimg.cn/20210121175304561.png)
3. 数据有顺序的数据写入到了不同的 partition 里面，不同的消费者去消费，但是每个 Consumer 的执行时间是不固定的，无法保证先读到消息的 Consumer 一定先完成操作，这样就会出现消息并没有按照顺序执行，造成数据顺序错误。

![多个消费者](https://img-blog.csdnimg.cn/20210121175528841.png)
##### 2.4.2 解决办法
1. 确保同一个消息发送到同一个 partition，一个topic，一个partition，一个consumer，内部单线程消费。
> 1. 写入同一个Partition的信息一定有序。
> 2. 给信息指定一个key，key相同则一定写入同一个partition。
> 3. 从同一个Partition读取信息一定有序。

![单线程消费](https://img-blog.csdnimg.cn/20210121175929122.png)
2.  在1的基础上，在一个 Consumer 上根据信息ID映射到不同队列，以此加速消费。
![内存队列](https://img-blog.csdnimg.cn/20210121191251208.png)
# 3 同步策略
### 3.1 同步方式
![数据同步方式 ](https://img-blog.csdnimg.cn/20210121194905780.png)
为保证 producer 发送的数据，能可靠的发送到指定的 topic，topic的每个 partition 收到 producer 发送的数据后，都需要向 producer 发送ack，如果 producer 收到 ack，就会进行下一轮的发送，否则重新发送数据。问题是啥时候发ack呢？

|方案  | 优点 | 缺点 
|--|--| -- |
|半数以上完成同步，就发送ack|	延迟低	|选举新的leader时，容忍n台节点的故障，需要2n+1个副本|
|全部完成同步，才发送ack	|选举新的leader时，容忍n台节点的故障，需要n+1个副本|	延迟高|
Kafka选择了`第二种`方案，原因如下：
1. 同样为了容忍n台节点的故障，第一种方案需要2n+1个副本，而第二种方案只需要n+1个副本，而Kafka的每个分区都有大量的数据，第一种方案会造成`大量数据的冗余`。
2. 虽然第二种方案的网络延迟会比较高，但**网络延迟对Kafka的影响较小**。
### 3.2 ISR
副本数据同步策略是全部follower，如果有一个卡死咋办？系统等死么？

Leader 维护个动态的 in sync replica  set，意为和leader保持同步的follower集合。当 ISR 中的 follower 完成数据的同步之后，leader就会给 follower 发送ack。如果 follower 长时间未向leader同步数据，则该 follower 将被踢出 ISR，该时间阈值由replica.lag.time.max.ms参数设定。Leader发生故障之后，就会从 ISR 中选举新的 leader。
### 3.3 故障细节
![故障过程](https://img-blog.csdnimg.cn/20210121201401601.png)
**LEO**：Long End Offset 指每个副本最大的offset。
**HW**：High Watermark 指消费者能见到的最大的offset，ISR队列中最小的LEO。
1. follower 故障
>follower发生故障后会被临时踢出 ISR，待该 follower 恢复后，follower会读取本地磁盘记录的上次的HW，并将log文件高于HW的部分截取掉，从HW开始向leader进行同步。等 follower 追上 leader 之后重新加入ISR了。
2. leader 故障
>leader 故障之后，从 ISR 中选出leader，为保证多个副本之间的数据一致性，**其余的follower会先将各自的log文件高于HW的部分截掉**，然后从新的leader同步数据。
3. 注意
>这只能保证副本之间的数据一致性，并不能保证数据不丢失或者不重复(还要用ACK机制)。

# 4 数据可靠性
### 4.1 消息传递语义
消息传递语义 message delivery semantic ，Kafka 为确保消息在 producer 和 consumer 之间传输。有以下三种传输保障（delivery guarantee）：
1. **at most once**：最多一次，消息可能会丢，但绝不会重复传输。
2. **at least once**：至少一次，消息绝不会丢，但可能会重复传输。
3. **exactly once**：精确传递一次。消息被处理且只会被处理一次。不丢失不重复就一次。

理想情况下肯定希望系统的消息传递是严格 exactly once，但很难做到。接下来会按照 消息的传播流程大致说下。

### 4.2 信息从生产者到 Broker
##### 4.2.1 生产者信息发送至Broker
大致步骤如下：
1. producer 从 ZK 找到目标 Partition 的 Leader 元数据。
2. producer 发送消息给 Leader。
3. Leader 接受消息持久化，然后根据acks配置选择如何同步Follower。
4. Followder 按照前面说的同步数据后给Leader回复ack。
5. Leader 跟 Follower 同步完毕后 Leader 给 producer 回复 ack。

对于Leader回复 ack，Kafka 为用户提供了三种可靠性级别，用户根据对可靠性和延迟的要求进行权衡。
1. request.required.acks = 0
> 1. producer`不等待` broker 的ack，提供了一个最低的延迟，broker接收到还没有写入磁盘就已经返回，当broker故障时有可能丢失数据，对应 At Most Once 模式。
> 2. 但凡没落盘成功信息就丢失了，一般生产不用。

2. request.required.acks  = 1
> 1. 此乃默认值，producer 等待 broker 的 ack，partition 的`leader落盘成功后返回ack`，如果在follower同步成功之前leader故障，那么将会丢失数据；认为leader返回 信息就成功了。

3. request.required.acks = -1 / all
> 1. producer 等待 broker 的 ack，partition 的 leader 和 follower (ISR中的)`全部落盘`成功后才返回 ack。
> 2. 但如果在 leader 收到信息返回ok，follower 收到信息但是发送 ack 时 leader 故障，此时生产者会`重新`给follower 发送个信息。
> 3. 对应 At Least Once 模式。

##### 4.2.2 如何保证幂等性
如果业务需要数据  Exactly Once，在早期的 Kafka 版本中 只能在下游去重，现在引入了个幂等性，意思就是无论生产者发送多少个重复消息，Server端只会持久化一条数据，
> At Least Once + 幂等性 = Exactly Once

启动幂等性，在生产者参数中 enable.idompotence= true，开启幂等性的生产者在初始化时候会被分配一个PID，发送同一个Partition的消息会附带Sequence Number，Broker会对<PID，Partition，Sequence Number> 做缓存，以此来判断唯一性。但是如果PID重启就会发生变化，同时不同partition也具有不同的主键，幂等性无法保证跨分区会话的 Exactly Once。
### 4.3 Kafka Broker 信息落磁盘
![数据落盘过程](https://img-blog.csdnimg.cn/2021012220253770.png)

Kafka Broker 收到信息后，如何落盘是通过 producer.type 来设定的，一般两个值。
1. sync，默认模式，数据必须最终落盘才算OK。
2. async，异步模式，数据刷新到OS的 Page Cache就返回，此时如果机器突然出问题，信息就丢失了。
### 4.4 消费者从 Kafka Broker 消费数据
![消费数据](https://img-blog.csdnimg.cn/20210122122822123.png)
Consumer 是以 Consumer Group 消费者组的方式工作，由一个或者多个消费者组成一个组，共同消费一个topic。每个分区在同一时间只能由group中的一个消费者读取，但是多个group可以同时消费这个partition。如果一个消费者失败了，那么其他的 group 成员会自动负载均衡读取之前失败的消费者读取的分区。Consumer Group 从 Broker 拉取消息来消费主要分为两个阶段：
1. 获得数据，提交 Offset。
2. 开始处理数据。

如果你先提交 offset 再处理数据可能在处理数据时出现异常导致数据丢失。而如果你先处理数据再提交 offset， 如果提交 offset 失败可能导致信息重复消费。

PS：
> pull 模式不足之处是，如果 kafka 没有数据，消费者可能会陷入循环中，一直返回空数据。针对这一点，Kafka的消费者在消费数据时会传入一个时长参数 timeout，如果当前没有数据可供消费，consumer会等待一段时间之后再返回，这段时长即为 timeout。

# 5 Kafka 分区分配策略
同一个 group.id 中的消费者，对于一个 topic 中的多个 partition 中的消息消费，存在着一定的分区分配策略。

在 Kafka 中存在着两种分区分配策略，通过 partition.assignment.strategy 来设置。
1.  RangeAssignor 范围分区策略，也是默认模式。
2. RoundRobinAssignor 分配策略，轮询分区模式。


### 5.1  RangeAssignor 范围分区策略
Range 范围分区策略是对每个 topic 而言的。首先对同一个 topic 里面的分区按照序号进行排序，并对消费者按照字母顺序进行排序。假如现在有 10 个分区，3 个消费者，排序后的分区将会是p0~p9。消费者排序完之后将会是C1-0、C2-0、C3-0。通过 Partitions数 / Consumer数 来决定每个消费者应该消费几个分区。如果除不尽，那么前面几个消费者将会多消费 1 个分区。
|消费者| 消费的分区 |
|--|--|
| C1-0 | 消费 p0、1、2、3分区 |
| C2-0 | 消费 4、5、6分区 |
| C3-0 | 消费 7、8、9分区 |

**Range 范围分区的弊端**：
如上只是针对 1 个 topic 而言，C1-0 消费者多消费1个分区影响不是很大。如果有 N 多个 topic，那么针对每个 topic，消费者 C1-0 都将多消费 1 个分区，topic越多，C1-0 消费的分区会比其他消费者明显多消费 N 个分区。这就是 Range 范围分区的一个很明显的弊端了.
### 5.2  RoundRobinAssignor 轮询分区策略
RoundRobin 轮询分区策略是把所有的 partition 和所有的 consumer 都列出来，然后按照 hascode 进行排序，最后通过轮询算法来分配 partition 给到各个消费者。轮询分区分为如下两种情况：
1. 同一个 Consumer Group 内 Consumer  订阅信息相同
2. 同一个 Consumer Group 内 Consumer  订阅信息不相同
##### 5.2.1 Consumer Group 内 Consumer  订阅信息相同
如果同一消费组内，所有的消费者订阅的消息都是相同的，那么 RoundRobin 策略的分区分配会是均匀的。

例如同一消费者组中，有 3 个消费者C0、C1和C2，都订阅了 2 个主题 t0 和 t1，并且每个主题都有 3 个分区(p0、p1、p2)，那么所订阅的所以分区可以标识为t0p0、t0p1、t0p2、t1p0、t1p1、t1p2。最终分区分配结果如下：
|消费者| 消费的分区 |
|--|--|
| C0 | 消费 t0p0、t1p0 分区 |
| C1 | 消费 t0p1、t1p1 分区 |
| C2 | 消费 t0p2、t1p2 分区 |
##### 5.2.2 Consumer Group 内 Consumer  订阅信息不相同
同一消费者组内，所订阅的消息是不相同的，那么分区分配就不是完全的轮询分配，有可能会导致分区分配的不均匀。如果某个消费者没有订阅消费组内的某个 topic，那么在分配分区的时候，此消费者将不会分配到这个 topic 的任何分区。

例如同一消费者组中有3个消费者C0、C1、C2，他们共订阅了 3 个主题t0、t1、t2，这 3 个主题分别有 1、2、3 个分区(即t0有1个分区(p0)，t1有2个分区(p0、p1)，t2有3个分区(p0、p1、p2))，即整个消费者所订阅的所有分区可以标识为 t0p0、t1p0、t1p1、t2p0、t2p1、t2p2。然后消费者 C0 订阅的是主题t0，消费者C1订阅的是主题t0和t1，消费者C2订阅的是主题t0、t1和t2，最终分区分配结果如下：
|消费者| 消费的分区 |
|--|--|
| C0 | 消费 t0p0 分区 |
| C1 | 消费 t1p0 分区 |
| C2 | 消费 t1p1、 t2p0、 t2p1、 t2p2 分区 |

# 6 Kafka 高效读写
Kafka 可支持百万 TPS 跟如下几个特性有关。

### 6.1 顺序读写数据
信息存储在硬盘中，硬盘由很多盘片组成，显微镜观察盘片会看见盘片表面凹凸不平，凸起的地方被磁化代表数字1，凹的地方是没有被磁化代表数字0，因此硬盘可以以二进制来存储表示文字、图片等信息。
![磁盘平面图](https://img-blog.csdnimg.cn/20210122174034168.png)
上图是硬盘的实际图，可能无法理解内部构造，我们来看个形象的图：
![磁盘内部图](https://img-blog.csdnimg.cn/20210122175224301.png)
1. 系统通过`磁头`从`盘面`读取数据，  磁头在盘面上的飞行高度只是人类头发直径的千分之一。
2. 硬盘里的`盘片`跟CD光盘的长相类似，一个`盘片`有上下两个`盘面`，每个`盘面`都可以存储数据。
3. 每个`盘面`会被划分出超级多的同心圆`磁道`，同心圆的半径是不同的。
4. 所有磁盘上的同一磁道构成一个`柱面`，相同`磁道`的同一个`扇区`被称为`簇`。数据的读写按照`柱面`从上到下进行，一个柱面写满后，才移到下一个扇区开始写数据。
5. 一个磁道是被划分成一段段的`圆弧`(`扇区`)，每个`扇区`用来存储 512个字节跟其他信息。由于同心圆的扇区弧度相同而半径不同所以外圈线速度比内圈线速度大。
6. 系统每次读取一个扇区效率太低，所以操作系统是按照`block`来进行读取数据的，一个`block`(块)一般有多个扇区组成。在每块的大小是 4~64KB。
7. 页`page`，默认4KB，操作系统经常与内存和硬盘这两种存储设备进行通信，类似于块的概念，都需要一种虚拟的基本单位。所以与内存操作，是虚拟一个页的概念来作为最小单位。与硬盘打交道，就是以块为最小单位。
>1. 扇区： 硬盘的最小读写单元
>1. 块/簇： 是操作系统针对硬盘读写的最小单元
>1. page： 是内存与操作系统之间操作的最小单元。

一次访盘的读/写请求完成过程由三个动作组成：
1. 寻道：磁头从开始移动到数据所在磁道所需要的时间，平均10ms左右。
2. 旋转延迟：盘片旋转将请求数据所在扇区移至读写磁头下方所需要的时间，旋转延迟取决于磁盘转速。如果是 5400 转每分钟的磁盘，平均大概为 5 ms。
3. 数据传输：磁头位从目标扇区第一个位置，到访问完所有数据的耗时。假如5400转的磁道有400个扇区，我只访问一个则耗时 0.0278ms。

可以发现读取主要耗时是在前两个，如果我顺序读取则`寻道`跟`旋转延迟`只用一次即可。而如果随机读取呢则可能经历多次`寻道`跟`旋转延迟`，两者相差几乎 **3**个数量级。![随机跟顺序读写在磁盘跟内存中](https://img-blog.csdnimg.cn/20210122195144312.png)
### 6.2  内存映射文件
Memory Mapped Files：
> 1. 虚拟内存系统 通过将虚拟内存分割为称作虚拟页(Virtual Page，VP)大小固定的块，一般情况下，每个虚拟页的大小默认是4KB。同样的，物理内存也被分割为物理页(Physical Page，PP)，也为4KB。
>2. 服务器可直接用 操作系统的 Page 来实现物理内存到文件的映射，用户操作读写数据会直接到Page中，操作系统会根据映射自动的将对物理内存的操作同步到硬盘上。实现类似**顺序读写内存**的功能。
>3. 缺点在Broker信息落盘时候也说了，落的不是真正磁盘可能导致**数据丢失**。


![内存映射](https://img-blog.csdnimg.cn/20210122214007739.png)
### 6.3 Zero Copy
##### 6.3.1  直接内存存取 DMA
CPU 发出指令操作 IO 来进行读写操作，大部分情况下其实只是把数据读取到内存，然后从内存传到IO即可，所以数据其实可以不经过CPU的。

`Direct Memory Access` 的出现就是为批量数据的输入/输出而提速的。DMA 是指外部设备不通过CPU而直接与系统内存交换数据的接口技术。这样数据的传送速度就取决于存储器和外设的工作速度。

如果数据传输的时候只用到了 DMA 传输而没经过 CPU 复制数据，则我们称之为零拷贝 Zero Copy。用了 Zero Copy 技术耗时性能起码减半。

##### 6.3.2 Kafka 读写对比
![零拷贝](https://img-blog.csdnimg.cn/20210122210837614.png)
如上黑色流程是没用 Zero Copy 技术流程：
1. DMA传输，磁盘读取数据到操作系统内存 Page Cache 区。
2. CPU搬运，数据从 Page Cache 区数据复制到用户内存区。
3. CPU搬运，数据从用户内存区到  Socket Cache 区。
4. DMA传输，数据从 Socket Cache 区传输到 NIC网卡缓存区。

红色流程是用 Zero Copy 技术流程：
1. DMA传输，磁盘读取数据到操作系统内存 Page Cache 区。
2. DMA传输，数据从 系统内存 Page Cache 区传输到 NIC网卡缓存区。
### 6.4 Batch Deal 
消费者拉取数据的时候，Kafka 不是一个一个的来送数据的，而是批量发送来处理的，这样可以节省网络传输，增大系统的TPS，不过也有个缺点就是，我们的数据不是真正的实时处理的，而真正的实时还是要看 Flink。
# 7 参考
> 1. Kafka 为什么要分区 ：https://www.zhihu.com/question/28925721
> 2. 关于磁盘读取：https://blog.csdn.net/holybin/article/details/21175781
> 3. Kafka百万TPS：https://mp.weixin.qq.com/s/Fb1cW0oN7xYeb1oI2ixtgQ

---
