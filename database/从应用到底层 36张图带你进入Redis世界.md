

>高清思维导图已同步Git：https://github.com/SoWhat1412/xmindfile，关注公众号sowhat1412获取海量资源

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218101648866.png#pic_center)


# 1、基本类型及底层实现
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218101608676.png#pic_center)

### 1.1、String
用途：
> 适用于简单key-value存储、setnx key value实现分布式锁、计数器(原子性)、分布式全局唯一ID。

底层：
C语言中String用char[]数组表示，源码中用`SDS`(simple dynamic string)封装char[]，这是是Redis存储的`最小单元`，一个SDS最大可以存储512M信息。
```c
struct sdshdr{
  unsigned int len; // 标记char[]的长度
  unsigned int free; //标记char[]中未使用的元素个数
  char buf[]; // 存放元素的坑
}
```
Redis对SDS再次封装生成了`RedisObject`，核心有两个作用：
>1. 说明是5种类型哪一种。
>2. 里面有指针用来指向 SDS

当你执行`set name sowhat`的时候，其实Redis会创建两个RedisObject对象，键的RedisObject 和 值的RedisOjbect 其中它们type = REDIS_STRING，而SDS分别存储的就是 name 跟 sowhat 字符串咯。

并且Redis底层对SDS有如下优化：
> 1. SDS修改后大小 > 1M时 系统会多分配空间来进行`空间预分配`。
> 2. SDS是`惰性释放空间`的，你free了空间，可是系统把数据记录下来下次想用时候可直接使用。不用新申请空间。

### 1.2、List

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218110450176.png#pic_center)

查看源码底层 `adlist.h` 会发现底层就是个 **双端链表**，该链表最大长度为2^32-1。常用就这几个组合。
> lpush + lpop = stack 先进后出的栈
 lpush + rpop = queue 先进先出的队列
 lpush + ltrim = capped collection 有限集合
 lpush + brpop = message queue 消息队列

一般可以用来做简单的消息队列，并且当数据量小的时候可能用到独有的压缩列表来提升性能。当然专业点还是要 [RabbitMQ](https://mp.weixin.qq.com/s/DUhHC2Oum7LNJnY76pbrxQ)、ActiveMQ等

### 1.3、Hash
散列非常适用于将一些相关的数据存储在一起，比如用户的购物车。该类型在日常用途还是挺多的。

这里需要明确一点： Redis中只有一个K，一个V。其中 K 绝对是字符串对象，而 V 可以是String、List、Hash、Set、ZSet任意一种。

hash的底层主要是采用字典dict的结构，整体呈现层层封装。从小到大如下：
##### 1.3.1、dictEntry
>真正的数据节点，包括key、value 和 next 节点。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218110536782.png#pic_center)

##### 1.3.2、dictht
>1、数据 dictEntry 类型的数组，每个数组的item可能都指向一个链表。
2、数组长度 size。
3、sizemask 等于 size - 1。
4、当前 dictEntry 数组中包含总共多少节点。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210165853649.png#pic_center)

##### 1.3.3、dict
>1、[dictType](https://www.cnblogs.com/yangming1996/p/11567856.html) 类型，包括一些自定义函数，这些函数使得key和value能够存储 
2、rehashidx 其实是一个标志量，如果为`-1`说明当前没有扩容，如果`不为 -1` 则记录扩容位置。
3、dictht数组，两个Hash表。
4、iterators 记录了当前字典正在进行中的迭代器

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218101833785.png#pic_center)

**组合后结构就是如下**：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020121811063120.png#pic_center)

##### 1.3.4、渐进式扩容
为什么 dictht ht[2]是两个呢？**目的是在扩容的同时不影响前端的CURD**，慢慢的把数据从ht[0]转移到ht[1]中，同时`rehashindex`来记录转移的情况，当全部转移完成，将ht[1]改成ht[0]使用。

rehashidx = -1说明当前没有扩容，rehashidx != -1则表示扩容到数组中的第几个了。

扩容之后的数组大小为大于used*2的**2的n次方**的最小值，跟 [HashMap](https://mp.weixin.qq.com/s/XGTNaOddY3elcumcPyO1KA) 类似。然后挨个遍历数组同时调整rehashidx的值，对每个dictEntry[i] 再挨个遍历链表将数据 Hash 后重新映射到 dictht[1]里面。并且 **dictht[0].use** 跟 **dictht[1].use** 是动态变化的。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020121810202290.png#pic_center)
整个过程的重点在于`rehashidx`，其为第一个数组正在移动的下标位置，如果当前内存不够，或者操作系统繁忙，扩容的过程可以随时停止。

停止之后如果对该对象进行操作，那是什么样子的呢？
>1、如果是新增，则直接新增后第二个数组，因为如果新增到第一个数组，以后还是要移过来，没必要浪费时间
>2、如果是删除，更新，查询，则先查找第一个数组，如果没找到，则再查询第二个数组。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218102053376.png#pic_center)

### 1.4、Set
如果你明白Java中HashSet是[HashMap](https://mp.weixin.qq.com/s/XGTNaOddY3elcumcPyO1KA)的简化版那么这个Set应该也理解了。都是一样的套路而已。这里你可以认为是没有Value的Dict。看源码 `t.set.c` 就可以了解本质了。
```c
int setTypeAdd(robj *subject, robj *value) {
    long long llval;
    if (subject->encoding == REDIS_ENCODING_HT) {
         // 看到底层调用的还是dictAdd，只不过第三个参数= NULL
         if (dictAdd(subject->ptr,value,NULL) == DICT_OK) {
            incrRefCount(value);
            return 1;
        }
        ....
```

### 1.5、ZSet
范围查找 的天敌就是 有序集合，看底层 `redis.h` 后就会发现 Zset用的就是可以跟二叉树媲美的`跳跃表`来实现有序。跳表就是多层**链表**的结合体，跳表分为许多层(level)，每一层都可以看作是数据的**索引**，**这些索引的意义就是加快跳表查找数据速度**。

每一层的数据都是有序的，上一层数据是下一层数据的子集，并且第一层(level 1)包含了全部的数据；层次越高，跳跃性越大，包含的数据越少。并且随便插入一个数据该数据是否会是跳表索引完全随机的跟玩骰子一样。

跳表包含一个表头，它查找数据时，是`从上往下，从左往右`进行查找。现在找出值为37的节点为例，来对比说明跳表和普遍的链表。

1. 没有跳表查询
比如我查询数据37，如果没有上面的索引时候路线如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200328222024546.png#pic_center#pic_center)

3. 有跳表查询
有跳表查询37的时候路线如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200328222121177.png#pic_center#pic_center)

应用场景：
>积分排行榜、时间排序新闻、延时队列。

### 1.6、Redis Geo
以前写过[Redis Geo核心原理解析](https://mp.weixin.qq.com/s/F4EvBqxEMDF4ksSGLAmXyA)，想看的直接跳转即可。他的核心思想就是将地球近似为球体来看待，然后 GEO利用 GeoHash 将二维的经纬度转换成字符串，来实现位置的划分跟指定距离的查询。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210190628614.jpg)

### 1.7、HyperLogLog
HyperLogLog ：是一种`概率`数据结构，它使用概率算法来统计集合的近似基数。而它算法的最本源则是`伯努利过程 + 分桶 + 调和平均数`。具体实现可看  [HyperLogLog 讲解](https://sowhat.blog.csdn.net/article/details/103027371)。

**功能**：误差允许范围内做基数统计 (基数就是指一个集合中不同值的个数) 的时候非常有用，每个HyperLogLog的键可以计算接近**2^64**不同元素的基数，而大小只需要12KB。错误率大概在0.81%。所以如果用做 UV 统计很合适。

HyperLogLog底层 一共分了 **2^14** 个桶，也就是 16384 个桶。每个(registers)桶中是一个 6 bit 的数组，这里有个骚操作就是一般人可能直接用一个字节当桶浪费2个bit空间，但是Redis底层只用6个然后通过前后拼接实现对内存用到了极致，最终就是 16384*6/8/1024 = 12KB。

### 1.8、bitmap
BitMap 原本的含义是用一个比特位来映射某个元素的状态。由于一个比特位只能表示 0 和 1 两种状态，所以 BitMap 能映射的状态有限，但是使用比特位的优势是能大量的节省内存空间。

在 Redis 中BitMap 底层是基于字符串类型实现的，可以把 Bitmaps 想象成一个以比特位为单位的数组，数组的每个单元只能存储0和1，数组的下标在 Bitmaps 中叫做偏移量，BitMap 的 offset 值上限 **2^32 - 1**。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218103804269.png#pic_center)

1. 用户签到
> key = 年份：用户id  offset = （今天是一年中的第几天） % （今年的天数）
2. 统计活跃用户
> 使用日期作为 key，然后用户 id 为 offset 设置不同offset为0 1 即可。


**PS** : Redis 它的通讯协议是基于TCP的应用层协议 [RESP](https://blog.csdn.net/qq_31821675/article/details/104565540)(REdis Serialization Protocol)。

### 1.9、Bloom Filter
使用布隆过滤器得到的判断结果： `不存在的一定不存在，存在的不一定存在`。

布隆过滤器 原理：
>当一个元素被加入集合时，通过K个散列函数将这个元素映射成一个位数组中的K个点(有效降低冲突概率)，把它们置为1。检索时，我们只要看看这些点是不是都是1就知道集合中有没有它了：如果这些点有任何一个为0，则被检元素一定不在；如果都是1，则被检元素很可能在。这就是布隆过滤器的基本思想。

想玩的话可以用Google的`guava`包玩耍一番。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201210202435111.png#pic_center#pic_center)

### 1.10 发布订阅
redis提供了`发布、订阅`模式的消息机制，其中消息订阅者与发布者不直接通信，发布者向指定的频道（channel）发布消息，订阅该频道的每个客户端都可以接收到消息。不过比专业的MQ(RabbitMQ RocketMQ ActiveMQ Kafka)相比不值一提，这个功能就算球了。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218104327704.png#pic_center)

# 2、持久化

因为Redis数据在内存，断电既丢，因此持久化到磁盘是必须得有的，Redis提供了RDB跟AOF两种模式。
### 2.1、RDB
RDB 持久化机制，是对 Redis 中的数据执行周期性的持久化。更适合做冷备。
优点：
>1、压缩后的二进制文，适用于备份、全量复制，用于灾难恢复加载RDB恢复数据远快于AOF方式，适合大规模的数据恢复。
2、如果业务对数据完整性和一致性要求不高，RDB是很好的选择。数据恢复比AOF快。

缺点：
>1、RDB是**周期间隔性的快照文件**，数据的完整性和一致性不高，因为RDB可能在最后一次备份时宕机了。
2、备份时占用内存，因为Redis 在备份时会独立fork一个**子进程**，将数据写入到一个临时文件（此时内存中的数据是原来的两倍哦），最后再将临时文件替换之前的备份文件。所以要考虑到大概两倍的数据膨胀性。

注意手动触发及[COW](https://blog.csdn.net/lh87270202/article/details/106430154)：

> 1、`SAVE` 直接调用 rdbSave ，`阻塞` Redis 主进程，导致无法提供服务。
> 2、`BGSAVE` 则 fork 出一个子进程，子进程负责调用 rdbSave ，在保存完成后向主进程发送信号告知完成。 在BGSAVE 执行期间**仍可以继续处理客户端的请求**。
> 3、Copy On Write 机制，备份的是开始那个时刻内存中的数据，只复制被修改内存页数据，不是全部内存数据。
4、Copy On Write 时如果父子进程大量写操作会导致分页错误。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218104350970.png#pic_center)

### 2.2、AOF
AOF 机制对每条写入命令作为日志，以 append-only 的模式写入一个日志文件中，因为这个模式是**只追加**的方式，所以没有任何磁盘寻址的开销，所以很快，有点像 Mysql 中的binlog。AOF更适合做热备。

优点：
> AOF是一秒一次去通过一个后台的线程fsync操作，数据丢失不用怕。

缺点：
>1、对于相同数量的数据集而言，AOF文件通常要大于RDB文件。RDB 在**恢复**大数据集时的速度比 AOF 的恢复速度要快。
2、根据同步策略的不同，AOF在运行效率上往往会慢于RDB。总之，每秒同步策略的效率是比较高的。
 
**AOF整个流程分两步**：
第一步是命令的实时写入，不同级别可能有1秒数据损失。命令先追加到`aof_buf`然后再同步到AO磁盘，**如果实时写入磁盘会带来非常高的磁盘IO，影响整体性能**。

第二步是对aof文件的**重写**，目的是为了减少AOF文件的大小，可以自动触发或者手动触发(**BGREWRITEAOF**)，是Fork出子进程操作，期间Redis服务仍可用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218104409814.png#pic_center)

>1、在重写期间，由于主进程依然在响应命令，为了保证最终备份的完整性；它`依然会写入旧`的AOF中，如果重写失败，能够保证数据不丢失。
2、为了把重写期间响应的写入信息也写入到新的文件中，因此也会`为子进程保留一个buf`，防止新写的file丢失数据。
3、重写是直接把`当前内存的数据生成对应命令`，并不需要读取老的AOF文件进行分析、命令合并。
4、**无论是 RDB 还是 AOF 都是先写入一个临时文件，然后通过` rename `完成文件的替换工作**。

关于Fork的建议：
>1、降低fork的频率，比如可以手动来触发RDB生成快照、与AOF重写；
2、控制Redis最大使用内存，防止fork耗时过长；
3、配置牛逼点，合理配置Linux的内存分配策略，避免因为物理内存不足导致fork失败。
4、Redis在执行`BGSAVE`和`BGREWRITEAOF`命令时，哈希表的负载因子>=5，而未执行这两个命令时>=1。目的是**尽量减少写操作**，避免不必要的内存写入操作。
5、**哈希表的扩展因子**：哈希表已保存节点数量 / 哈希表大小。因子决定了是否扩展哈希表。

### 2.3、恢复
启动时会先检查AOF(数据更完整)文件是否存在，如果不存在就尝试加载RDB。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218104438213.png#pic_center)

### 2.4、建议
既然单独用RDB会丢失很多数据。单独用AOF，数据恢复没RDB来的快，所以出现问题了第一时间用RDB恢复，然后AOF做数据补全才说王道。

# 3、Redis为什么那么快
### 3.1、 基于内存实现：
> 数据都存储在内存里，相比磁盘IO操作快百倍，操作速率很快。

### 3.2、高效的数据结构：
>Redis底层多种数据结构支持不同的数据类型，比如HyperLogLog它连2个字节都不想浪费。

### 3.3、丰富而合理的编码：
Redis底层提供了 [丰富而合理的编码](https://mp.weixin.qq.com/s/b_yzbLeQh57oYjqlIgPiYQ)  ，五种数据类型根据长度及元素的个数适配不同的编码格式。
> String：自动存储int类型，非int类型用raw编码。
List：字符串长度且元素个数小于一定范围使用 **ziplist** 编码，否则转化为 **linkedlist** 编码。
Hash：hash 对象保存的键值对内的键和值字符串长度小于一定值及键值对。
Set：保存元素为整数及元素个数小于一定范围使用 intset 编码，任意条件不满足，则使用 **hashtable** 编码。
Zset：保存的元素个数小于定值且成员长度小于定值使用 **ziplist** 编码，任意条件不满足，则使用 **skiplist** 编码。

### 3.4、合适的线程模型：
> `I/O 多路复用`模型同时监听客户端连接，多线程是需要上下文切换的，对于内存数据库来说这点很致命。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218104723400.png#pic_center)

### 3.5、 Redis6.0后引入`多线程`提速：
要知道 读写网络的read/write系统耗时 `>>` Redis运行执行耗时，Redis的瓶颈主要在于**网络的 IO 消耗**, 优化主要有两个方向:
> 提高网络 IO 性能，典型的实现比如使用 DPDK 来替代内核网络栈的方式
使用多线程充分利用多核，典型的实现比如 Memcached。

协议栈优化的这种方式跟 Redis 关系不大，支持多线程是一种最有效最便捷的操作方式。所以Redis支持多线程主要就是两个原因：
> 可以充分利用服务器 CPU 资源，目前主线程只能利用一个核
多线程任务可以分摊 Redis 同步 IO 读写负荷
   
关于多线程须知:
> 1. Redis 6.0 版本 默认多线程是关闭的 io-threads-do-reads no
> 2. Redis 6.0 版本 开启多线程后 线程数也要 [谨慎设置](https://www.cnblogs.com/madashu/p/12832766.html)。
> 3. 多线程可以使得性能翻倍，但是多线程只是用来处理网络数据的读写和协议解析，**执行命令仍然是单线程顺序执行**。

# 4、常见问题
### 4.1、缓存雪崩

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201211093708734.png#pic_center#pic_center)

雪崩定义：
> Redis中大批量key在同一时间同时失效导致所有请求都打到了MySQL。而MySQL扛不住导致大面积崩塌。

雪崩解决方案：
>1、缓存数据的过期时间加上个随机值，防止同一时间大量数据过期现象发生。
>2、如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
>3、设置热点数据永远不过期。

### 4.2、缓存穿透
穿透定义：
> 缓存穿透 是 指缓存和数据库中`都没有`的数据，比如ID默认>0，黑客一直 请求ID= -12的数据那么就会导致数据库压力过大，严重会击垮数据库。

穿透解决方案：
> 1、后端接口层增加 用户**鉴权校验**，**参数做校验**等。
> 2、单个IP每秒访问次数超过阈值**直接拉黑IP**，关进小黑屋1天，在获取IP代理池的时候我就被拉黑过。
> 3、从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null 失效时间可以为15秒**防止恶意攻击**。
> 4、用Redis提供的  **Bloom Filter** 特性也OK。

### 4.3、缓存击穿

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201211100750411.png#pic_center#pic_center)

击穿定义：
>现象：大并发集中对这一个热点key进行访问，当这个Key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库。
>
击穿解决：
> 设置热点数据永远不过期
> 加上互斥锁也能搞定了

### 4.4、双写一致性
双写：`缓存`跟`数据库`均更新数据，如何保证数据一致性？

1、先更新数据库，再更新缓存
> 安全问题：线程A更新数据库->线程B更新数据库->线程B更新缓存->线程A更新缓存。`导致脏读`。
> 业务场景：读多写少场景，频繁更新数据库而缓存根本没用。更何况如果缓存是叠加计算后结果更`浪费性能`。

2、先删缓存，再更新数据库
> A 请求写来更新缓存。
> B 发现缓存不在去数据查询旧值后写入缓存。
> A 将数据写入数据库，此时缓存跟数据库**不一致**。

因此 **FackBook** 提出了  [Cache Aside Pattern](https://mp.weixin.qq.com/s/-fk-cEIo3iDCUSwT_l8d2w)
>失效：应用程序先从cache取数据，没有得到，则从数据库中取数据，成功后，放到缓存中。
命中：应用程序从cache中取数据，取到后返回。
更新：`先把数据存到数据库中，成功后，再让缓存失效`。

### 4.5、脑裂

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201212111426950.png#pic_center#pic_center)

脑裂是指因为网络原因，导致master节点、slave节点 和 sentinel集群处于不用的网络分区，此时因为sentinel集群**无法感知**到master的存在，所以将slave节点提升为master节点 此时存在两个不同的master节点就像一个大脑分裂成了两个。其实在`Hadoop` 、`Spark`集群中都会出现这样的情况，只是解决方法不同而已(用ZK配合强制杀死)。

集群脑裂问题中，如果客户端还在基于原来的master节点继续写入数据那么新的master节点将无法同步这些数据，当网络问题解决后sentinel集群将原先的master节点降为slave节点，此时再从新的master中同步数据将造成大量的数据丢失。

Redis处理方案是redis的配置文件中存在两个参数
```bash
min-replicas-to-write 3  表示连接到master的最少slave数量
min-replicas-max-lag 10  表示slave连接到master的最大延迟时间
```
如果连接到master的slave数量 < 第一个参数 且 ping的延迟时间 <= 第二个参数那么master就会拒绝写请求，配置了这两个参数后如果发生了集群脑裂则原先的master节点接收到客户端的写入请求会拒绝就可以减少数据同步之后的数据丢失。

### 4.6、事务
[MySQL](https://mp.weixin.qq.com/s/O_NHjv_YVUi4lSqXnhx5Mg)中的事务还是挺多道道的还要，而在Redis中的事务只要有如下三步：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218104854212.png#pic_center)

关于事务具体结论：
>1、redis事务就是一次性、顺序性、排他性的执行一个队列中的**一系列命令**。　
2、Redis事务**没有隔离级别**的概念：批量操作在发送 EXEC 命令前被放入队列缓存，并不会被实际执行，也就**不存在事务内的查询要看到事务里的更新，事务外查询不能看到**。
3、Redis**不保证原子性**：Redis中单条命令是原子性执行的，但事务不保证原子性。
4、Redis编译型错误事务中所有代码均不执行，指令使用错误。运行时异常是错误命令导致异常，其他命令可正常执行。
5、watch指令类似于**乐观锁**，在事务提交时，如果watch监控的多个KEY中任何KEY的值已经被其他客户端更改，则使用EXEC执行事务时，事务队列将不会被执行。

### 4.7、常用客户端
> 1.  jedis：经典工具，提供了全面Redis操作指令。
> 2. Redisson：提供了分布式操作跟可扩展Java数据结构，有分布式锁跟分布式集合。
> 3. Lettuce：基于Netty实现，底层是异步调动，感兴趣可以一试。

### 4.8 缓存预热跟降级
缓存预热：
>系统上线后先将相关缓存数据加载到缓存系统中防止MySQL压力过大，

缓存降级：
> 当缓存失效后或者无法访问的时候，作出正确的举措，不给数据库过大压力，而是直接返回默认值或者返回设定的默认值。
### 4.9、正确开发步骤
>`上线前`：Redis **高可用**，主从+哨兵，Redis cluster，避免全盘崩溃。
`上线时`：本地 ehcache 缓存 + Hystrix 限流 + 降级，避免MySQL扛不住。
`上线后`：Redis **持久化**采用 RDB + AOF 来保证断点后自动从磁盘上加载数据，快速恢复缓存数据。

# 5、分布式锁
日常开发中我们可以用 [synchronized](https://mp.weixin.qq.com/s/e_fYFWK5Qnxjmz6Abi7uqw) 、[Lock](https://mp.weixin.qq.com/s/kvuPxn-vc8dke093XSE5IQ)  实现并发编程。但是Java中的锁**只能保证在同一个JVM进程内中执行**。如果在分布式集群环境下用锁呢？日常一般有两种选择方案。
### 5.1、 Zookeeper实现分布式锁
你需要知道一点基本`zookeeper`知识：
>1、持久节点：客户端断开连接zk不删除persistent类型节点
2、临时节点：客户端断开连接zk删除ephemeral类型节点
3、顺序节点：节点后面会自动生成类似0000001的数字表示顺序
4、节点变化的通知：客户端注册了监听节点变化的时候，会**调用回调方法**

大致流程如下，其中注意每个节点`只`监控它前面那个节点状态，从而避免`羊群效应`。关于模板代码百度即可。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218111439150.png#pic_center)

缺点：
> 频繁的创建删除节点，加上注册watch事件，对于zookeeper集群的压力比较大，性能也比不上Redis实现的分布式锁。

### 5.2、 Redis实现分布式锁
本身原理也比较简单，Redis 自身就是一个单线程处理器，具备互斥的特性，通过setNX，exist等命令就可以完成简单的分布式锁，处理好超时释放锁的逻辑即可。

SETNX 
>SETNX 是SET if Not eXists的简写，日常指令是`SETNX key value`，如果 key 不存在则set成功返回 1，如果这个key已经存在了返回0。

SETEX
> SETEX key seconds value 表达的意思是 将值 value 关联到 key ，并将 key 的生存时间设为多少秒。如果 key 已经存在，setex命令将覆写旧值。并且 setex是一个`原子性`(atomic)操作。

加锁：
> 一般就是用一个标识唯一性的字符串比如UUID 配合 SETNX 实现加锁。

解锁：
>这里用到了LUA脚本，LUA可以保证是**原子性**的，思路就是判断一下Key和入参是否相等，是的话就删除，返回成功1，0就是失败。

缺点：
> 这个锁是**无法重入的**，且自己实心的话各种边边角角都要考虑到，所以了解个大致思路流程即可，**工程化还是用开源工具包就行**。
### 5.3、 Redisson实现分布式锁
**Redisson** 是在Redis基础上的一个服务，采用了基于NIO的Netty框架，不仅能作为Redis底层驱动**客户端**，还能将原生的RedisHash，List，Set，String，Geo，HyperLogLog等数据结构封装为Java里大家最熟悉的映射（Map），列表（List），集（Set），通用对象桶（Object Bucket），地理空间对象桶（Geospatial Bucket），基数估计算法（HyperLogLog）等结构。

这里我们只是用到了关于分布式锁的几个指令，他的大致底层原理：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218105550958.png#pic_center)
[Redisson加锁解锁](https://mp.weixin.qq.com/s/y_Uw3P2Ll7wvk_j5Fdlusw) 大致流程图如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218105613825.png#pic_center)

# 6、Redis 过期策略和内存淘汰策略
### 6.1、Redis的过期策略
Redis中 [过期策略](https://blog.csdn.net/weixin_42777004/article/details/108734354) 通常有以下三种：

1、**定时过期**：
>每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即对key进行清除。该策略可以立即清除过期的数据，对内存很友好；但是**会占用大量的CPU资源去处理过期的数据**，从而影响缓存的响应时间和吞吐量。

2、**惰性过期**：
>只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却**对内存非常不友好**。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。

3、**定期过期**：
>每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源**达到最优**的平衡效果。
>expires字典会保存所有设置了过期时间的key的过期时间数据，其中 key 是指向键空间中的某个键的指针，value是该键的毫秒精度的UNIX时间戳表示的过期时间。键空间是指该Redis集群中保存的所有键。


Redis采用的过期策略：`惰性删除` + `定期删除`。memcached采用的过期策略：`惰性删除`。

### 6.2、6种内存淘汰策略
Redis的内存淘汰策略是指在Redis的用于缓存的内存不足时，怎么处理需要新写入且需要申请额外空间的数据。
>1、volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选**最近最少使用**的数据淘汰
2、volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选**将要过期**的数据淘汰
3、volatile-random：从已设置过期时间的数据集（server.db[i].expires）中**任意选择**数据淘汰
4、allkeys-lru：从数据集（server.db[i].dict）中挑选**最近最少使用**的数据淘汰
5、allkeys-random：从数据集（server.db[i].dict）中**任意选择数**据淘汰
6、no-enviction（驱逐）：禁止驱逐数据，**不删除**的意思。

面试常问常考的也就是**LRU**了，大家熟悉的`LinkedHashMap`中也实现了`LRU`算法的，LinkedHashMap 是通过双向链表和散列表这两种数据结构组合实现的。LinkedHashMap 中的`Linked`实际上是指的是双向链表，并非指用链表法解决散列冲突。稍微修改就是个LRU：

```java
class SelfLRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int CACHE_SIZE;
    /**
     * 传递进来最多能缓存多少数据
     * @param cacheSize 缓存大小
     */
    public SelfLRUCache(int cacheSize) {
  // true 表示让 linkedHashMap 按照访问顺序来进行排序，最近访问的放在头部，最老访问的放在尾部。
        super((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, true);
        CACHE_SIZE = cacheSize;
    }
    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // 当 map中的数据量大于指定的缓存个数的时候，就自动删除最老的数据。
        return size() > CACHE_SIZE;
    }
}
```
### 6.2、总结
**Redis的内存淘汰策略的选取并不会影响过期的key的处理。内存淘汰策略用于处理内存不足时的需要申请额外空间的数据，过期策略用于处理过期的缓存数据**。
# 7、Redis 集群高可用
单机问题有机器故障、容量瓶颈、QPS瓶颈。在实际应用中，Redis的多机部署时候会涉及到`redis主从复制`、`Sentinel哨兵模式`、`Redis Cluster`。

|模式|优点  | 缺点|
|--|--| --|
| 单机版 | 架构简单，部署方便 | 机器故障、容量瓶颈、QPS瓶颈 |
| 主从复制 | 高可靠性，读写分离 | 故障恢复复杂，主库的写跟存受单机限制 |
|  Sentinel 哨兵|  集群部署简单，HA| 原理繁琐，slave存在资源浪费，不能解决读写分离问题 |
| Redis Cluster|  数据动态存储solt，可扩展，高可用| 客户端动态感知后端变更，批量操作支持查 |

###  7.1、redis主从复制
该模式下 具有高可用性且读写分离， 会采用 `增量同步` 跟 `全量同步` 两种机制。
##### 7.1.1、全量同步

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218111505771.png#pic_center)

Redis全量复制一般发生在**Slave初始化阶段**，这时Slave需要将Master上的**所有数据**都复制一份： 
>1、slave连接master，发送`psync`命令。
2、master接收到`psync`命名后，开始执行bgsave命令生成RDB文件并使用缓冲区记录此后执行的所有写命令。
3、master发送快照文件到slave，并在发送期间继续记录被执行的写命令。
4、slave收到快照文件后丢弃所有旧数据，载入收到的快照。
5、master快照发送完毕后开始向slave发送缓冲区中的写命令。
6、slave完成对快照的载入，开始接收命令请求，并执行来自master缓冲区的写命令。

##### 7.1.2、增量同步
也叫**指令同步**，就是从库重放在主库中进行的指令。Redis会把指令存放在一个**环形队列**当中，因为内存容量有限，如果备机一直起不来，不可能把所有的内存都去存指令，也就是说，如果备机一直未同步，指令可能会被覆盖掉。

Redis增量复制是指Slave初始化后开始正常工作时master发生的写操作同步到slave的过程。 
增量复制的过程主要是master每执行一个写命令就会向slave发送相同的写命令。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201212114228811.png#pic_center#pic_center)


##### 7.1.3、Redis主从同步策略：
> 1、`主从刚刚连接的时候，进行全量同步；全同步结束后，进行增量同步`。当然，如果有需要，slave 在任何时候都可以发起全量同步。redis 策略是，无论如何，首先会尝试进行增量同步，如不成功，要求从机进行全量同步。
> 2、slave在同步master数据时候如果slave丢失连接不用怕，slave在重新连接之后`丢失重补`。
> 3、一般通过主从来实现读写分离，但是如果master挂掉后如何保证Redis的 HA呢？ 引入`Sentinel`进行master的选择。
### 7.2、高可用之哨兵模式

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218105855507.png#pic_center)

[Redis-sentinel](https://www.cnblogs.com/guolianyu/p/10345387.html)  本身是一个**独立**运行的进程，一般sentinel集群 节点数至少三个且奇数个，它能监控多个master-slave集群，sentinel节点发现master宕机后能进行自动切换。Sentinel可以监视任意多个主服务器以及主服务器属下的从服务器，并在被监视的主服务器下线时，**自动执行故障转移操作**。这里需注意` sentinel`也有`single-point-of-failure`问题。大致罗列下哨兵用途：
> 集群监控：循环监控master跟slave节点。
> 消息通知：当它发现有redis实例有故障的话，就会发送消息给管理员
> 故障转移：这里分为主观下线(单独一个哨兵发现master故障了)。客观下线(多个哨兵进行抉择发现达到quorum数时候开始进行切换)。
> 配置中心：如果发生了故障转移，它会通知将master的新地址写在配置中心告诉客户端。

### 7.3、Redis Cluster
RedisCluster是Redis的分布式解决方案，在3.0版本后推出的方案，有效地解决了Redis分布式的需求。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218105911970.png#pic_center)

##### 7.3.1、分区规则

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218105947607.png#pic_center)

常见的分区规则
> 1. `节点取余`：hash(key) % N
> 2. `一致性哈希`： 一致性哈希环
> 3. `虚拟槽哈希`：CRC16[key] & 16383
> 
[Redis Cluster](https://www.cnblogs.com/ymaa/p/13420334.html)采用了`虚拟槽分区`方式，具题的实现细节如下：
>1、采用去**中心化**的思想，它使用**虚拟槽solt分区**覆盖到所有节点上，取数据一样的流程，节点之间使用轻量协议通信**Gossip**来减少带宽占用所以性能很高，
2、自动实现**负载均衡与高可用**，自动实现**failover**并且支持**动态扩展**，官方已经玩到可以1000个节点 实现的复杂度低。
3、每个Master也需要配置主从，并且内部也是采用**哨兵模式**，如果有半数节点发现某个异常节点会共同决定更改异常节点的状态。
4、如果集群中的master没有slave节点，则master挂掉后整个集群就会进入**fail**状态，因为集群的slot映射不完整。**如果集群超过半数以上的master挂掉，集群都会进入fail状态**。
5、官方推荐 **集群部署至少要3台以上的master节点**。


 
# 8、Redis 限流
经常乘坐北京西二旗地铁或者在北京西站乘坐的时候经常会遇到一种情况就是如果人很多，地铁的工作人员拿个小牌前面一档让你等会儿再检票，这就是实际生活应对人流量巨大的措施。

在开发高并发系统时，有三把利器用来保护系统：`缓存`、`降级`和`限流`。那么何为限流呢？顾名思义，限流就是限制流量，就像你宽带包了1个G的流量，用完了就没了。通过限流，我们可以很好地控制系统的qps，从而达到保护系统的目的。

### 1、基于Redis的setnx、zset
##### 1.2、setnx
比如我们需要在10秒内限定20个请求，那么我们在setnx的时候可以设置过期时间10，当请求的[setnx](https://blog.csdn.net/Wisimer/article/details/110259465)数量达到20时候即达到了限流效果。

缺点：比如当统计1-10秒的时候，无法统计2-11秒之内，如果需要统计N秒内的M个请求，那么我们的Redis中**需要保持N个key等等问题**。
##### 1.3、zset
其实限流涉及的最主要的就是滑动窗口，上面也提到1-10怎么变成2-11。其实也就是起始值和末端值都各+1即可。 我们可以将请求打造成一个**zset数组**，当每一次请求进来的时候，value保持唯一，可以用UUID生成，而score可以用当前时间戳表示，因为score我们可以用来计算当前时间戳之内有多少的请求数量。而zset数据结构也提供了**range**方法让我们可以很轻易的获取到2个时间戳内有多少请求，

缺点：就是zset的数据结构会越来越大。
### 2、漏桶算法
漏桶算法思路：把水比作是请求，漏桶比作是系统处理能力极限，水先进入到漏桶里，漏桶里的水**按一定速率流出**，当流出的速率小于流入的速率时，由于漏桶容量有限，后续进入的水直接溢出（拒绝请求），以此实现限流。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218110047561.png#pic_center)

### 3、令牌桶算法
令牌桶算法的原理：可以理解成医院的挂号看病，只有拿到号以后才可以进行诊病。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201218110119377.png#pic_center)

细节流程大致：
>1、所有的请求在处理之前都需要**拿到一个可用的令牌才会被处理**。
2、根据限流大小，设置按照一定的速率往桶里添加令牌。
3、设置桶最大可容纳值，当桶满时新添加的令牌就被丢弃或者拒绝。
4、请求达到后首先要获取令牌桶中的令牌，拿着令牌才可以进行其他的业务逻辑，处理完业务逻辑之后，将令牌直接删除。
5、令牌桶有最低限额，当桶中的令牌达到最低限额的时候，请求处理完之后将不会删除令牌，以此保证足够的限流。

工程化：
>1、[自定义注解、aop、Redis + Lua](https://mp.weixin.qq.com/s/kyFAWH3mVNJvurQDt4vchA) 实现限流。
>2、推荐 **guava** 的**RateLimiter**实现。
>3、令牌法可能允许瞬间高并发的小访问量，因为拿到令牌就好。而漏斗法则是永远固定速度的那种。

# 9、常见知识点
1. 字符串模糊查询时用`Keys`可能导致线程阻塞，尽量用`scan`指令进行无阻塞的取出数据然后去重下即可。
2. 多个操作的情况下记得用`pipeLine`把所有的命令一次发过去，避免频繁的发送、接收带来的网络开销，提升性能。
3. bigkeys可以扫描redis中的大key，底层是使用scan命令去遍历所有的键，对每个键根据其类型执行STRLEN、LLEN、SCARD、HLEN、ZCARD这些命令获取其长度或者元素个数。缺陷是线上试用并且个数多不一定空间大，
4. 线上应用记得开启Redis慢查询日志哦，基本思路跟MySQL类似。
5. Redis中因为内存分配策略跟增删数据是会导致`内存碎片`，你可以重启服务也可以执行`activedefrag yes`进行内存重新整理来解决此问题。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201212173251409.png#pic_center#pic_center)
> Ratio >1 表明有内存碎片，越大表明越多严重。
> Ratio < 1 表明正在使用虚拟内存，虚拟内存其实就是硬盘，性能比内存低得多，这是应该增强机器的内存以提高性能。
> 一般来说，mem_fragmentation_ratio的数值在1 ~ 1.5之间是比较健康的。

# 10、End

关于Redis先吹逼这么多(本来想写秒杀的，怕写太长)，如果你感觉没看够那`得价钱`。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201212114652736.gif)

---
### 往期精选：
1. [顺丰快递：请签收MySQL灵魂十连](https://mp.weixin.qq.com/s/O_NHjv_YVUi4lSqXnhx5Mg)
1. [面试HashMap看这篇就够了](https://mp.weixin.qq.com/s/XGTNaOddY3elcumcPyO1KA) 
2. [烂大街的Spring循环依赖如何说](https://mp.weixin.qq.com/s/Y4xCFTphtc_NectWB005ew)
3. [由浅入深逐步了解 Synchronized](https://mp.weixin.qq.com/s/e_fYFWK5Qnxjmz6Abi7uqw)
4. [快速上手JUC下常见并发容器](https://mp.weixin.qq.com/s/TlnAeajB8hAfImvuB6yGag)
5. [3W字玩转SpringCloud](https://mp.weixin.qq.com/s/A4yRiLBM9JTcPV8nulln2A)  

--- 




