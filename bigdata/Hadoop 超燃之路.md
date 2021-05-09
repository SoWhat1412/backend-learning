# 1  Hadoop 简介
### 1.1 Hadoop 由来
![数据容量](https://img-blog.csdnimg.cn/20210114202727804.png)
大数据时代数据量超级大，数据具有如下特性：
1. Volume(大量)
2. Velocity(高速)
3. Variety(多样)
4. Value(低价值密度)

以前的存储手段跟分析方法现在行不通了！Hadoop 就是用来解决海量数据的 **存储** 跟海量数据的 **分析计算** 问题的，创始人 Doug Cutting 在创建 Hadoop 时主要思想源头是 Google 三辆马车
> 1. 第一辆 GFS 产生了 HDFS。
> 2. 第二辆  MapReduce 产生了MR。
> 3. 第三辆 BigTable 产生了HBase。

现在说的 Hadoop 通常指的是 **Hadoop 生态圈** 这样一个广义概念，如下：
![大数据知识体系](https://img-blog.csdnimg.cn/20210114205043616.png)
### 1.2  Hadoop 特点

##### 1.2.1  Hadoop 特点
1. 高可用
> Hadoop 底层对同一个数据维护这多个复本，即使Hadoop某个计算元素或者存储出现问题，也不会导致数据的丢失。

2. 高扩展
>在集群之间分配任务数据，可以方便的扩展跟删除多个节点，比如美团节点就在3K~5k 个节点

3. 高效性
>在MapReduce的思想下 Hadoop是并行工作的，以加快任务的处理速度

4. 高容错性
>如果一个子任务速度过慢或者任务失败 Hadoop会有响应策略会自动重试跟任务分配。

##### 1.2.2 Hadoop 架构设计
Hadoop 的 1.x 跟 2.x 区别挺大，2.x 主要是将1.x  MapReduce中资源调度的任务解耦合出来交 Yarn 来管理了(接下来本文以2.7开展探索)。
![1.x跟2.x变化](https://img-blog.csdnimg.cn/20210114210829727.png)
1. **HDFS**
> Hadoop Distributed File System 简称 HDFS，是一个分布式文件系统。HDFS 有着高容错性，被设计用来部署在低廉的硬件上来提供高吞吐量的访问应用程序的数据，适合超大数据集的应用程序。

2. **MapReduce**
>MapReduce是一种编程模型，包含Map(映射) 跟 Reduce(归约)。你可以认为是归并排序的深入化思想。

3. **Yarn**
> Apache Hadoop YARN （Yet Another Resource Negotiator，另一种资源协调者）是一种新的 Hadoop 资源管理器，它是一个**通用资源管理系统**，可**为上层应用提供统一的资源管理和调度**，它的引入为集群在利用率、资源统一管理和数据共享等方面带来了巨大好处。

4. **Common** 组件
> 1. log组件。
> 2. 独有RPC体系ipc、I/O系统、序列化、压缩。
> 3. 配置文件conf。
> 4. 公共方法类，比如checkSum校验。

# 2 HDFS
产生背景：
>随着数据量变大，数据在一个OS的磁盘无法存储了，需要将数据分配到多个OS管理的磁盘中，为了方面管理多个OS下的磁盘文件，迫切需要一种系统来管理多台机器上的文件，这就是**分布式文件管理系统**，HDFS 是通过目录树定位文件。需注意 HDFS 只是分布式文件系统中的其中一种。

### 2.1 HDFS 优缺点
#####  2.1.1 优点

1. 高容错性
> 1. 数据会自动保存多个副本，默认为3个，通过增加副本来提高容错性。
> 2. 某个副本丢失后系统会自动恢复。

2. 高扩展性
> HDFS集群规模是可以动态伸缩的。

4. 适合大数据处理
> 1. 数据规模达到GB/TB/PB级别。
> 2. 文件规模达到百万规模以上。
5. 流式访问
> 他能保证数据的一致性。
6. 低成本
> 部署廉价机器 提高了商业化能了。
7. 统一对外接口
> Hadoop本身用Java编写，但基于此的应用程序可以用其他语言编写调用。

##### 2.1.1 缺点
1. 做不到低延时
> Hadoop对高吞吐做了优化，牺牲了获取数据的延迟，比如毫秒级获取数据在Hadoop上做不到。

2. 不适合存储大量小文件
> 1. 存储大量小文件的话，它会占用  NameNode 大量的内存来存储文件、目录和块信息。因此该文件系统所能存储的文件总数受限于 NameNode 的内存容量，根据经验，每个文件、目录和数据块的存储信息大约占**150**字节。
> 2. 小文件存储的寻道时间会超过读取时间，它违反了HDFS的设计目标。

3. 无法修改文件
> 对于上传到HDFS上的文件，不支持修改文件，仅支持追加。HDFS适合一次写入，多次读取的场景。

4. 无法并发写入
> HDFS不支持多用户同时执行写操作，即同一时间，只能有一个用户执行写操作。

### 2.2  HDFS 组成架构

##### 2.2.1 Client
客户端主要有如下功能：
1. 文件切分：文件上传 HDFS 的时候，Client 将文件切分成一个一个的Block，然后进行存储。
2. 与 NameNode 交互，获取文件的位置信息。
3. 与 DataNode 交互，读取或者写入数据。
4. Client 提供一些命令来管理 HDFS，比如启动或者关闭 HDFS。
5. Client 可以通过一些命令来访问 HDFS。

##### 2.2.2 NameNode
NameNode 简称NN，就是HDFS中的 Master，是个管理者，主要有如下功能：
1. 管理HDFS的名称空间。
2. 配置副本策略
3. 处理客户端读写请求。
4.  管理数据块（Block）映射信息，比如
> 映射信息： NameNode(文件路径，副本数，{Block1，Block2}，[Block1:[三个副本路径],Block2:[三个副本路径]])

##### 2.2.3 DataNode
DataNode简称DN 就是HDFS集群中的Slave，NameNode负责下达命令，DataNode执行实际的操作。
1. 存储实际的数据块。
2. 执行数据块的读/写操作。

上面说过数据目录信息存储在NN中，而具体信息存储在DN中，很形象的比喻如下
![NN跟DN对比](https://img-blog.csdnimg.cn/20210115144958622.png)
**DataNode 的工作机制**
1. 数据块存储在磁盘信息 包括 数据 + 数据长度 + 校验和 + 时间戳。
2. DataNode 启动后向 NameNode注册，周期性（1小时）的向 NameNode 上报所有的块信息。
3. NN 跟 DN 之间 心跳 3秒一次，心跳返回结果带有 NameNode 给该 DataNode 的命令如复制块数据到另一台机器，或删除某个数据块。如果超过10分钟没有收到某个 DataNode 的心跳，则认为该节点不可用。
4. 集群运行中可以安全加入和退出一些机器。

**DataNode 确保数据完整性**
1. 当 DataNode 读取 Block 的时候，它会计算 CheckSum。
2. 如果计算后的 CheckSum，与 Block 创建时值不一样，说明 Block 已经损坏。
3. Client 读取其他 DataNode 上的 Block。
4. DataNode 在其文件创建后周期验证 CheckSum

DN 进程死亡或无法跟 NN 通信后 NN 不会立即将 DN 判死，一般经过十分钟 + 30秒再判刑。
##### 2.2.4 Secondary NameNode
当 NameNode 挂掉的时候，它并不能马上替换 NameNode 并提供服务。需要通过 **HA** 等手段实现自动切换。SNN 主要提供如下功能：
1. 辅助 NameNode，分担其工作量。
2. 定期合并 Fsimage 和 Edits，并推送给 NameNode。
3. 在紧急情况下，可辅助恢复 NameNode。

##### 2.2.5  Block
HDFS中的文件在物理上是分块Block存储的，在1.x版本中块 = 64M，2.x中块 = 128M。块不是越大越好，也不是越小越好。因为用户获取数据信息时间 = 寻址块时间 + 磁盘传输时间。
1. 块太小会增加寻址时间，程序大部分耗时在寻址上了。
2. 快太大则会导致磁盘传输时间明显大于寻址时间，程序处理块数据时较慢。
### 2.3 HDFS 写流程
##### 2.3.1 具体写流程
![写流程](https://img-blog.csdnimg.cn/20210115155401438.png)
1. 客户端通过 Distributed FileSystem 模块向 NameNode 请求上传文件，NameNode检查目标文件是否已存在，父目录是否存在。
2. NameNode 返回是否可以上传。
3. 客户端请求第一个 Block上传到哪几个 DataNode 服务器上。
4. NameNode 返回3个 DataNode 节点，分别为dn1、dn2、dn3。
5. 客户端通过 FSDataOutputStream 模块请求dn1上传数据，dn1收到请求会继续调用dn2，然后dn2调用dn3，将这个通信管道建立完成。
6. dn1、dn2、dn3逐级应答客户端。
7. 客户端开始往dn1上传第一个Block（先从磁盘读取数据放到一个本地内存缓存），以Packet为单位，dn1收到一个Packet就会传给dn2，dn2传给dn3；dn1每传一个packet会放入一个应答队列等待应答。
8. 当一个 Block 传输完成之后，客户端再次请求NameNode上传第二个Block的服务器。（重复执行3-7步）。
##### 2.3.2 节点距离计算
在 HDFS 写数据的过程中，NameNode 会选择**距离待上传数据最近距离**的DataNode接收数据。

最近距离 = **两个节点到达最近的共同祖先的距离总和**。

![节点距离计算](https://img-blog.csdnimg.cn/20210115161451856.png)
1. Distance(/d1/r1/n0,/d1/r1/n0) = 0 同一节点上的进程
1. Distance(/d1/r1/n1,/d1/r1/n2) = 2 同一机架上不同节点
1. Distance(/d1/r2/n0,/d1/r3/n2) = 4 同一数据中心不同机架节点
1. Distance(/d1/r2/n1,/d2/r4/n1) = 6 不同数据中心

##### 2.3.3 副本节点选择
1. 第一个副本在Client所在节点上，如果在集群外则随机选个。
2. 第二个副本跟第一个副本位于同机架不同节点
3. 第三个部分位于不同机架，随机节点。
![机架感知](https://img-blog.csdnimg.cn/202101151628282.png)
### 2.4 HDFS 读流程
![读流程](https://img-blog.csdnimg.cn/20210115164457386.png)
1. 客户端通过 Distributed FileSystem 向 NameNode 请求下载文件，NameNode 通过查询元数据，找到文件块所在的 DataNode 地址。
2. 挑选一台 DataNode（就近原则，然后随机）服务器，请求读取数据。
3. DataNode 开始传输数据给客户端（从磁盘里面读取数据输入流，以Packet为单位来做校验）。
4. 客户端以 **Packet** 为单位接收，先在本地缓存，然后写入目标文件。
### 2.5 NameNode 和 Secondary NameNode
##### 2.5.1  NN 和 2NN 工作机制
NameNode 中**元数据**单独存到磁盘不方便读写。单独存到内存时，断电会丢失。Hadoop 采用的是如下方式。
1. **FsImage** : 
> 元数据序列化后在磁盘存储的地方。包含HDFS文件系统的所有目录跟文件inode序列化信息。

2. **Memory**：
> 元数据在内存中存储的地方。

3. **Edit 文件**：
> 1. Edit 记录客户端更新元数据信息的每一步操作（可通过Edits运算出元数据）。
> 2. 一旦元数据有更新跟添加，元数据修改追加到Edits中然后修改内存中的元数据，这样一旦NameNode 节点断电，通过 FsImage 跟 Edits 的合并生成元数据。
> 3. Edits文件不要过大，系统会定期的由 Secondary Namenode 完成 FsImage 和 Edits 的合并。

![NN跟2NN工作机制](https://img-blog.csdnimg.cn/20210115180154557.png)

**第一阶段：NameNode 启动**
1. 第一次启动 NameNode 格式化后，创建 Fsimage 和 Edits 文件。如果不是第一次启动，直接加载编辑日志和镜像文件到内存。
2. 客户端对元数据进行增删改的请求。
3. NameNode 记录操作日志，更新滚动日志。
4. NameNode 在内存中对数据进行增删改。

**第二阶段：Secondary NameNode 工作**
1. Secondary NameNode 询问 NameNode 是否需要 CheckPoint。直接带回 NameNode 是否检查结果。一般下面条件任意满足即可：
> 1. CheckPoint 默认1小时执行一次。
> 2. 一分钟检查一次Edits文件操作次数，达阈值 CheckPoint 。
2. Secondary NameNode 请求执行 CheckPoint。
3. NameNode 滚动正在写的 Edits 日志。
4. 将滚动前的编辑日志Edit_001 和 镜像文件FsImage 拷贝到 Secondary NameNode。
5. Secondary NameNode 加载编辑日志和镜像文件到内存并合并。
6. 生成新的镜像文件 FsImage.chkpoint。
7. 拷贝 FsImage.chkpoint 到 NameNode。
8. NameNode 将 FsImage.chkpoint 重新命名成 FsImage。

### 2.6 安全模式
NameNode 刚启动时候系统进入安全模式(只读)，如果整个文件系统中99.9%块满足最小副本，NameNode 会30秒后退出安全模式。

##### 2.6.1 NameNode 启动
1. 将 FsImage 文件载入内存再执行Edits文件各种操作，最终内存生成完整的元数据镜像。
2. 创建个新的 FsImage 跟空 Edits 文件。
3. NameNode 开始监听 DataNode。
4. 整个过程 NameNode 一直运行在安全模式，NameNode 对于 Client 是只读的。

##### 2.6.2 DataNode 启动
1. 系统数据块位置不是由 NameNode 维护的，而是以块列表形式存储在 DataNode 中。
2. 安全模式下 DataNode 向 NameNode 发送最新块列表信息，促使 NameNode 高效运行。
3. 正常运行期 NameNode 内存中保留所有块位置映射信息。
### 2.7 HDFS-HA
HDFS 集群中 NameNode存在单点故障（SPOF），为了实现 High Available，其实包括 HDFS-HA 和YARN-HA。  HDFS 可以 通过配置Active/Standby 两个 NameNodes 实现在集群中对 NameNode 的热备来解决上述问题。如果出现故障，如机器崩溃或机器需要升级维护，可将NameNode很快的切换到另外一台机器。实现 HA 功能主要依赖**ZooKeeper** 跟 **ZKFC** 进程。
![HA故障转移](https://img-blog.csdnimg.cn/20210115205200585.png)

##### 2.7.1 HDFS-HA工作要点
1.	元数据管理方式需要改变
> 1. 内存中各自保存一份元数据。
> 1. Edits 日志只有 Active 状态的 NameNode 节点可以做写操作。
> 1. 两个 NameNode 都可以读取 Edits。
> 1. 共享的 Edits 放在一个共享存储中管理（qjournal 或 NFS）。
2.	需要一个状态管理功能模块
> 1. 实现了一个ZKFC，常驻在每一个namenode所在的节点，每一个ZKFC负责监控自己所在NameNode节点，利用zk进行状态标识，当需要进行状态切换时，由ZKFC来负责切换，切换时需要防止brain split现象的发生。
3.	必须保证两个 NameNode 之间能够ssh无密码登录
4.	防脑裂，同一时刻仅仅有一个 NameNode 对外提供服务。

##### 2.7.2 ZooKeeper
ZooKeeper提供如下功能：
1. **故障检测**：集群中每个 NameNode 在 ZooKeeper 中维护一个持久会话，如果机器崩溃，ZooKeeper中的会话将终止，ZooKeeper通知另一个NameNode需要触发故障转移。
2. **现役NameNode选择**：ZooKeeper提供了一个简单的机制用于唯一的选择一个节点为active状态。如果目前现役NameNode崩溃，另一个节点可能从ZooKeeper获得特殊的排外锁以表明它应该成为现役NameNode。

##### 2.7.3 ZKFC进程
在 NameNode 主机上有个 ZKFC(ZKFailoverController) 这样的ZK客户端，负责监视管理 NameNode 状态。ZKFC负责：
1. **健康监测**：ZKFC周期性检测同主机下NameNode监控撞库。
2. **ZooKeeper会话管理**：NameNode健康时候ZKFC保持跟ZK集群会话打开状态，ZKFC还持有个znode锁，如果会话终止，锁节点将自动删除。
3. **基于ZooKeeper的选择**：  ZKFC发现本地NameNode健康前提下会尝试获取znode锁，获得成功则Active状态。 

# 3 MapReduce
MapReduce是个**分布式运算程序的编程框架**，是基于 Hadoop 的 数据分析计算核心框架。处理过程分为两个阶段：Map 阶段跟 Reduce 阶段。
> 1. Map 负责把一个任务分解成多个任务。该阶段的 MapTask 并发实例，完全并行运行，互不相干。
> 2. Reduce 负责把多个任务处理结果汇总。该阶段的 ReduceTask 并发实例互不相干，但是他们的数据依赖于上一个阶段的所有 MapTask 并发实例的输出。
> 3. MapReduce 编程模型只能包含一个 Map 阶段和一个 Reduce 阶段，如果用户的业务逻辑非常复杂，那就只能多个MapReduce程序串行运行。

用户编写MR任务时候 程序实现部分分成三个部分：Mapper、Reducer、Driver(提交运行mr程序的客户端)。

### 3.1 优缺点

##### 3.1.1 优点
1. **易于编程**
> 简单实现了一些接口就可以完成个分布式程序，你写个分布式程序跟写个串行化程序一样，类似八股文编程。

2. **良好的扩展**
>  计算资源不足时可以简单的增加机器来扩展计算能力。

3. **高容错性**
> MapReduce任务部署在多台机器上后如果其中一台挂了，系统会进行自动化的任务转移来保证任务正确执行。

4. **适合PB级数据离线处理**
> 比如 美团3K个节点的集群并发，提供超大数据处理能力。

##### 3.1.2 缺点

1. 不擅长**实时**计算
> MapReduce 不会想 MySQL 一样毫秒级返回结果。

2. 不擅长**流式**计算
> 流式计算的 输入数据是动态的，而 MapReduce 的输入数据集是静态的。

3. 不擅长**DAG**计算
> 多个应用程序存在依赖关系，MapReduce的作业结果会落盘导致大量磁盘IO，性能贼低，此时上Spark吧！

### 3.2 序列化
**序列化**
>把内存中的对象，转换成字节序列（或其他数据传输协议）以便于存储（持久化）和网络传输。 

**反序列化**
>将收到字节序列（或其他数据传输协议）或者是硬盘的持久化数据，转换成内存中的对象。

因为 Hadoop 在集群之间进行通讯或者 RPC 调用时是需要序列化的，而且要求序列化要快、且体积要小、占用带宽要小。而Java自带的序列化是重量级框架，对象序列化后会附带额外信息，比如各种校验信息，header，继承体系等。所以 Hadoop `自研`了序列化框架。
| Java类型 |Hadoop Writable类型  |
|--|--|
| boolean| 	BooleanWritable| 
| byte| 	ByteWritable| 
| int| 	IntWritable| 
| float| 	FloatWritable| 
| long	| LongWritable| 
| double| 	DoubleWritable| 
| String	| Text| 
| map| 	MapWritable| 
| array	| ArrayWritable| 

### 3.3 MapTask 并行度
**数据块**：Block 是 HDFS 物理上把数据分成一块一块。
**数据切片**：数据切片只是在**逻辑**上对输入进行分片，并不会在磁盘上将其切分成片进行存储。

切片核心注意点：
1. 一个 Job 的 Map 阶段并行度又客户端提交Job时的切片数决定
2. 每个 Split 切片分配个 MapTask 并行实例处理
3. 模型情况下 切片大小 =  BlockSize
4. 切片时不会考虑数据集整体大小，而是逐个针对每个文件单独切片的。

##### 3.3.1 FileInputFormat 切片源码追踪
FileInputFormat切片源码追踪
1. 程序先找到目标数据存储目录
2. 开始遍历目录下每个文件。每个文件都会做如下操作
3. 获取切片大小，默认情况下切片大小 =  blocksize 
4. 开始切片，每次切片都要判断剩余部分是否大于块的1.1倍，不大于则就划分到一个切片。
5. 切片信息写到切片规划文件中。
6. 切片核心过程在getSplit方法完成。
7. InputSplit只是记录了切片元数据信息，如起始位置、长度跟所在节点列表等。

##### 3.3.2 切片大小计算 
**SplitSize**= Math.max(`minSize`,Math.min(`maxSize`,`blockSize`))
> mapreduce.input.fileinputformat.split.`minsize` 默认 1
> mapreduce.input.fileinputformat.split.`maxsize` 默认 Long.MAXValue
> blockSize 默认128M
> maxsize ：该参数如果比blockSize小灰导致切片变小，且就等于配置的整个参数。
minsize ：该参数如果调的比blockSize大，则切片大小会比blockSize还大。

##### 3.3.3 切片举例
![切片举例](https://img-blog.csdnimg.cn/20210116113902252.png)

### 3.4 FileInputFormat

##### 3.4.1 实现类简介
MR任务输入文件个数各有不同，针对不同类型MR定义了一个接口跟若干实现类来读取不同的数据。
![input继承关系](https://img-blog.csdnimg.cn/20210116121516600.png)

1. TextInputFormat
> 默认使用类，按行读取每条数据，Key是该行数据的 offset，Value = 行内容。

2. KeyValueTExtInputFormat
> 每行都是一条记录，被指定分隔符分割为Key跟Value，默认是 \t 。

3. NLineInputFormat
> 该模型下每个 map 处理 InputSplit 时不再按照 Block 块去划分，而是按照指定的行数N来划分文件。

4. 自定义InputFormat
> 基础接口，改写 RecordReader，实现一次读取一个完整文件封装为 KV，使用 SequenceFileOutPutFormat 输出合并文件。 

5. CombineTextInputFormat
>  用于小文件过多场景，逻辑上合并多个小文件个一个切片任务。较重要 中

##### 3.4.2 CombineTextInputFormat
默认框架 TextInputFormat 切片机制是对任务按文件规划切片，不管文件多小，都会是一个单独的切片，都会交给一个MapTask，这样如果有大量小文件，就会产生大量的MapTask，处理效率极其低下。CombineTextInputFormat 可以将多个小文件从逻辑上规划到一个切片中，这样多个小文件就可以交给一个MapTask处理。主要包含 **虚拟存储过程** 跟 **切片过程**。
> CombineTextInputFormat.setMaxInputSplitSize(job, 4194304); // 4m

**虚拟存储过程：**
1. 文件 <= SplitSize 则单独一块。
2.  1 * SplitSize < 文件  < 2 * SplitSize 时对半分。
3. 文件 >= 2*SplitSize时，以 SplitSize 切割一块，剩余部分若  < 2 * SplitSize 则对半分。

**切片过程：**
1. 判断虚拟存储的文件大小是否大于setMaxInputSplitSize值，大于等于则单独形成一个切片。
2. 如果不大于则跟下一个虚拟存储文件进行合并，共同形成一个切片。
![切片过程](https://img-blog.csdnimg.cn/20210116161800191.png)

### 3.6 OutputFormat
 OutputFormat 是 MapReduce 输出的基类，常见的实现类如下：

##### 3.5.1 TextOutputFormat
 系统默认输出格式，把每条记录写为文本行，他的K跟V是任意类型，系统在写入时候会统一转化为字符串。

##### 3.5.2 SequenceFileOutputFormat 
 此模式下的输出结果作为后续MapReduce任务的输入，该模式下数据格式紧凑，很容易被压缩。

##### 3.5.3 自定义OutputFormat 
如果需求不满足可按需求进行自定义。
1. 自定义类继承自FileOutputFormat。
2. 重写RecordWriter，改写具体输出数据的方法write。

### 3.6 MapReduce 流程

##### 3.6.1 整体流程图
![MapReduce流程](https://img-blog.csdnimg.cn/20210118120902504.png)
**MapTask 工作机制**
1. **Read阶段**：MapTask 通过用户编写的RecordReader，从输入InputSplit中解析出一个个key/value。
2. **Map阶段**：将解析出的key/value交给用户编写map()函数处理，并产生一系列新的key/value。
3. **Collect收集阶段**：它会将生成的key/value分区（调用Partitioner），并写入一个环形内存缓冲区中。
4. **Spill阶段**：先按照分区进行排序，然后区内按照字典对key进行快排，并在必要时对数据进行合并、压缩等操作。
5. **Combine阶段**：选择性可进行MapTask内的优化提速。

**ReduceTask 工作机制**
1. **Copy阶段**：从所有的MapTask中收集结果然后决定将数据放入缓存还是磁盘。 
2. **Merge阶段**：copy数据时后天会对磁盘还有内存数据进行Merge。
3. **Sort阶段**：ReduceTask需对所有数据进行一次归并排序，方便执行reduce 函数。
4. **Reduce阶段**：调用用户 reduce() 函数将计算结果写到HDFS上。

##### 3.6.2 Shuffle 
![Shuffle机制](https://img-blog.csdnimg.cn/20210116190335558.png)
 MapReduce 的核心就是  Shuffle 过程，Shuffle 过程是贯穿于 map 和 reduce 两个过程的！ 在Map端包括Spill过程，在Reduce端包括copy和sort过程。  具体Shuffle过程如下：
1. MapTask 收集我们的map()方法输出的kv对，放到内存缓冲区中。
2. 从内存缓冲区不断溢出本地磁盘文件，可能会溢出多个文件，溢出前会按照分区针对key进行区内快排。
3. 多个溢出文件会被合并成大的溢出文件。
4. 在溢出过程及合并的过程中，都要调用 Partitioner 进行分区和针对key进行排序。
5. ReduceTask 根据自己的分区号，去各个 MapTask 机器上取相应的结果分区数据。
6. ReduceTask 对收集后的数据进行合并跟归并排序。
7. 进入 ReduceTask 的逻辑运算过程，调用用户自定义的reduce()方法。
8. Shuffle 中的缓冲区大小会影响到 MapReduce 程序的执行效率，原则上说，缓冲区越大，磁盘io的次数越少，执行速度就越快。

##### 3.6.3 Partition
MapReduce  默认的分区方式是hashPartition，在这种分区方式下，KV 对根据 key 的 hashcode 值与reduceTask个数进行取模，决定该键值对该要访问哪个ReduceTask。
```java
public int getPartition(K2 key, V2 value, int numReduceTasks) {
    return (key.hashCode() & Integer.MAX_VALUE) % numReduceTasks;
    // numReduceTasks 默认 = 1 所以导致默认的reduce结果 = 1
  }
```
自定义的时候一般就是类继承Partitioner然后重写`getPartition` 方法。用户也可以设置ReduceTask数量，不过会遵循如下规则。
1. 如果 ReduceTask 数  > getPartition 数， 会多产生几个空的输出part-r-000xx。
2. 如果 1 < ReduceTask < getPartition 数，会有部分数据无法安放导致报错。
3. 如果ReduceTask = 1，不管MapTask端输出多少分区文件结果都是一个文件。
4. 分区必须从0开始，逐步累加。

比如 假设自定义分区数为5。
> 1. job.setNumReduceTasks(1)：会正常运行，只不过会产生一个输出文件。
> 2. job.setNumReduceTasks(2)：会报错。
>3. job.setNumReduceTasks(6)：大于5，程序会正常运行，会产生空文件。

##### 3.6.4 环形缓冲区
Map 的输出结果由 Collector 处理，每个 Map 任务不断地将键值对输出到在内存中构造的一个环形数据结构中。使用环形数据结构是为了更有效地使用内存空间，在内存中放置尽可能多的数据。

环形数据结构其实就是个字节数组byte[]，叫kvbuffer，默认值100M。里面主要存储 `数据` 跟 `元数据`。中间有个分界点，并且分界点是变化的。当环形缓冲区写入的buffer的大小达到 80%  满足溢写条件的时候，开始溢写spill。系统有两个线程一个负责写入数据，一个负责spill数据。

**数据：**
>存储 Key +  Value + bufindex。其中 bufindex（即数据的存储方向）是一直闷着头地向上增长，比如bufindex初始值为0，一个Int型的key写完之后，bufindex增长为4，一个Int型的value写完之后，bufindex增长为8。

**元数据：**
>1. 元数据是为了排序而生，是关于数据描述的数据。

>2. Kvmeta =  Partition  + keystart + valstart +  valLength ， 共占用4个Int长度，其中K的长度 = V的起点 -  K的起点。

>3. Kvmeta 的存放指针 Kvindex 每次都是向下跳四个 格子，然后再向上一个格子一个格子地填充四元组的数据。比如Kvindex初始位置是-4，当第一个键值对写完之后，(Kvindex+0)的位置存放partition的起始位置、(Kvindex+1)的位置存放keystart、(Kvindex+2)的位置存放valstart、(Kvindex+3)的位置存放value length，然后Kvindex跳到 -8位置，等第二个键值对和索引写完之后，Kvindex跳到-12位置。

```java
kvmeta.put(kvindex + PARTITION, partition);
kvmeta.put(kvindex + KEYSTART, keystart);
kvmeta.put(kvindex + VALSTART, valstart);
kvmeta.put(kvindex + VALLEN, distanceTo(valstart, valend));
// advance kvindex 改变每次index的值 每次4个位置!
kvindex = (kvindex - NMETA + kvmeta.capacity()) % kvmeta.capacity();
```
![环形缓冲区](https://img-blog.csdnimg.cn/20210116180105481.png)

##### 3.6.5 Combiner 合并
1. Combiner 是 MR 程序中 Mapper 跟 Reducer 之外的组件。
2. Combiner 是在每一个MapTask 所在节点运行，Reducer 是接受全部 Mapper 输出结果。
3. Combiner 属于局部汇总的意思，来减少网络传输。
4. Combiner 用的时候要注意不能影响最终业务逻辑！比如求平均值就不能用。求和就OK。

##### 3.6.6 关于 MapReduce 排序
MapReduce框架最重要的操作就是排序，MapTask 跟 ReduceTask 都会根据key进行按照字典顺序进行快排。
1. MapTask 将缓冲区数据快排后写入到磁盘，然后磁盘文件会进行归并排序。
2. ReduceTask统一对内存跟磁盘所有数据进行归并排序。

##### 3.6.7  ReduceJoin 跟 MapJoin 
**Reducejoin**
`思路`：通过将关联条件作为Map 输出的 Key，将两表满足 Join条件的数据并携带数据源文件发送同一个ReduceTask，在Reduce端进行数据串联信息合并。
`缺点`：合并操作在Reduce端完成，Reduce 端处理压力太大，并且Reduce端易产生数据倾斜。
**MapJoin** 
`适用`：适用于一张表十分小、一张表很大的场景。
`思路`：在 Map 端缓存多张表，提前处理业务逻辑，这样增加 Map 端业务，减少 Reduce 端数据的压力，尽可能的减少数据倾斜。

##### 3.6.8 注意点
1. ReduceTask = 0 说明没有Reduce节点，输出文件个数和 Map 个数一样。
2. ReduceTask 默认= 1，所以结果是一个文件。
3. ReduceTask 的个数不是任意设置的，需跟集群性能还有结果需求而定。
4. 逻辑处理 Mapper 时候可根据业务需求实现其中三个方法，map、setup、cleanup。
### 3.7 压缩
压缩是提高Hadoop运行效率的一种优化策略，通过在Mapper、Reducer运行过程的数据进行压缩来减少磁盘空间跟网络传输，最终实现提高MR运行速度。但需注意压缩也给CPU运算带来了负担。
压缩的基本原则：
1. 运算密集型任务 ，少压缩。
2. IO密集型任务，多压缩。

|压缩格式|系统自带|	算法|	文件扩展名|	是否可切分|	换成压缩格式后，原来的程序是否需要修改|
|--|--|-- |-- |-- |-- |
|DEFLATE|	是|	DEFLATE|	.deflate	|否	|和文本处理一样，不需要修改|
|Gzip|	是	|DEFLATE|	.gz	|否	|和文本处理一样，不需要修改|
|bzip2	|是|	bzip2|	.bz2|	是|	和文本处理一样，不需要修改|
|Snappy	|否|	Snappy|	.snappy|	否|	和文本处理一样，不需要修改|
|LZO	|否|	LZO	|.lzo|	是|	需要建索引，还需要指定输入格式|

# 4 YARN
Yarn 是一个**资源调度平台**，负责为运算程序提供服务器运算资源，相当于一个分布式的操作系统平台，而 MapReduce 等运算程序则相当于运行于操作系统之上的应用程序。

### 4.1 基本组成
![Yarn架构](https://img-blog.csdnimg.cn/20210118123433359.png)
YARN主要由 ResourceManager、NodeManager、ApplicationMaster 和 Container 等组件构成。

1. **ResourceManager**
 > 1. 处理客户端请求
> 2. 监控NodeMananger
> 3. 启动或监控ApplicationMaster 
> 4. 计算资源的分配跟调度

2. **NodeManager**
>1. 管理单个节点上资源
>2. 处理来着ResourceManager的命令
>3. 处理来自ApplicationMaster的命令

3. **ApplicationMaster**
>1. 负责数据切分。
>2. 为应用程序申请资源并分配给内部任务。
>3. 任务的监控跟容错。

4.  **Container** 
>1. Container 是 YARN 中资源的抽象，封装了某个节点上的多维度资源，比如内存、CPU、磁盘、网络等。

>2. YarnChild 其实它就是一个运行程序的进程。 MrAppMaster 运行程序时向 Resouce Manager 请求的 Maptask / ReduceTask。

### 4.2 Yarn 调度 MapReduce 任务
![Yarn调度流程](https://img-blog.csdnimg.cn/20210118154717695.png)
当 MR 程序提交到客户端所在的节点时后 大致运行流程如下：
**作业提交**
>1. Client 调用 job.waitForCompletion 方法 [YarnRunner](https://www.ituring.com.cn/article/212225?utm_source=tuicool&utm_medium=referral) ，向整个集群提交MapReduce作业。 Client 向 RM 申请一个作业id。
>2. RM 给 Client 返回该 job 资源的提交路径和作业 id。
>3. Client 提交jar包、切片信息和配置文件到指定的资源提交路径。
>4. Client 提交完资源后，向 RM 申请运行 MrAppMaster。

**作业初始化**
>5. 当 RM 收到 Client 的请求后，将该 Task 添加到容量调度器中。
>6. 某一个空闲的 NodeManager 领取到该 Task 。
>7. 该 NodeManager  创建 Container，并产生 MRAppMaster。
>8. 下载 Client 提交的资源 到本地。

**任务分配**
>9. MRAppMaster 向 RM 申请运行多个 MapTask 任务资源。
>10.  RM 将运行 MapTask 任务分配给俩 NodeManager。其中分配原则 是优先 jar 跟 数据在一台机器上，其次就尽可能在一个机房。最后 随便来个空闲机器。

**任务运行**
>11. MR 向两个接收到任务的 NodeManager 发送程序启动脚本，这两个 NodeManager 分别启动MapTask，MapTask 对数据分区排序。
>12. MrAppMaster 等待所有 MapTask 运行完毕后，向RM申请容器 运行ReduceTask。
>13. ReduceTask 向 MapTask 获取相应分区的数据。
>14. 程序运行完毕后，MR会向RM申请注销自己。

**进度和状态更新**
>YARN 中的任务将其进度和状态(包括counter)返回给应用管理器， 客户端每秒向应用管理器请求进度更新来展示给用户。

**作业完成**
>除了向应用管理器请求作业进度外, 客户端每5秒都会通过调用 waitForCompletion() 来检查作业是否完成。作业完成之后, 应用管理器和Container会清理工作状态。作业的信息会被作业历史服务器存储以备之后用户核查。

### 4.3 资源调度器
目前，Hadoop作业调度器主要有三种：FIFO、Capacity Scheduler 和 Fair Scheduler。Hadoop2.7.2默认的资源调度器是Capacity Scheduler。

##### 4.3.1 FIFO
![FIFO调度](https://img-blog.csdnimg.cn/20210118155509840.png)

##### 4.3.2  容量调度器 Capacity Scheduler
![容量调度器](https://img-blog.csdnimg.cn/20210118160631240.png)
1. 支持多个队列，每个队列配置一定资源，每个队列采用FIFO策略。
2. 为防止同一个童虎作业独占队列资源，会对同一用户提交作业所占资源量限制。
3. 计算每个队列中在跑任务数与其应该分得的计算只有比值，选择个比值最小的队列(最闲的)。
4. 按照作业优先级跟提交时间，同时还考虑用户资源限制跟内存限制对队列任务排序。
5. 比如job1、job2、job3分配排在最前面也是并行运行。

##### 4.3.3 公平调度器 Fair Scheduler
支持多队列多用户，每个队列中资源可以配置，同一队列中作业公平共享队列中所有资源。
![公平调度器](https://img-blog.csdnimg.cn/20210118160911790.png)
比如有queue1、queue2、queue3三个任务队列，每个队列中的job按照优先级分配资源，优先级高获得资源多，但会确保每个任务被分配到资源。
每个任务理想所需资源跟实际获得资源的差距叫缺额，同一个队列中是按照缺额高低来先后执行的，缺额越大越优先获得资源。

### 4.4 任务推测执行
作业完成时间取决于**最慢**的任务完成时间。系统中有99%的Map任务都完成了，只有少数几个Map老是进度很慢，此时系统会发现拖后腿的任务，比如某个任务运行速度远慢于任务平均速度。为拖后腿任务启动一个**备份任务**，同时运行。谁先运行完，则采用谁的结果。

# 5 MapReduce 优化方法
MapReduce优化方法主要从六个方面考虑：数据输入、Map阶段、Reduce阶段、IO传输、数据倾斜问题和常用的调优参数。

### 5.1 数据输入
1. 数据采集时，用 **Hadoop  Archive** 将多个小文件打包成一个Har文件。
2. 业务处理前，SequenceFile 由一系列KV组成，key=文件名，value=文件内容，将大批小文件合并成大文件。
3. 在 MapReduce 处理时，采用**CombineTextInputFormat**来作为输入，解决输入端大量小文件场景。
4. 对于大量小文件任务开启JVM 重用可提速，JVM 重用可以使得 JVM 实例在同一个 job 中重新使用N次。N的值可以在Hadoop的mapred-site.xml文件中进行配置，通常在10-20之间。

### 5.2 Map 阶段
1. 减少溢写 Spill 次数，调整循环缓存区大小，减少磁盘IO。
2. 减少合并 Merge 次数，增大Merge文件大小减少次数。
3. 在不影响业务的情况下在Map端进行Combine处理。

### 5.3 Reduce 阶段
1. 设置合理的Map跟REduce数，太少会导致Task等待。太多会导致竞争资源激烈。
2. 设置Map跟Reduce阶段共存，map运行一定程度后Reduce 也可以运行。
3. 规避使用Reduce，Reduce 端的Buffer也要合理设置，尽量防止溢写到磁盘。

### 5.4 IO 传输
1. 采用数据压缩方式来减少网络IO时间。
2. 使用SequenceFile二进制文件。

### 5.5 数据倾斜
1. 通过对数据抽样得到结果集来设置分区边界值。
2. 自定义分区。
3. 使用Combine来减少数据倾斜。
4. 采用MapJoin，尽量避免ReduceJoin。

# 参考
> HDFS-Shell 指令： http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_shell.html
