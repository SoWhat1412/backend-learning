![在这里插入图片描述](https://img-blog.csdnimg.cn/cover1/247466910341660722.jpg#pic_center)
# 分布式ID必要性。
业务量**小于500W或数据容量小于2G**的时候单独一个mysql即可提供服务，再大点的时候就进行读写分离也可以应付过来。但当主从同步也扛不住的是就需要分表分库了，但分库分表后**需要有一个唯一ID来标识一条数据**，数据库的自增ID显然不能满足需求；特别一点的如订单、优惠券也都需要有唯一ID做标识。此时一个能够生成全局唯一ID的系统是非常必要的。那么这个全局唯一ID就叫分布式ID。

分布式ID需满足那些条件
- 全局唯一：基本要求就是必须保证ID是全局性唯一的。
- 高性能：高可用低延时，ID生成响应要快。
- 高可用：无限接近于100%的可用性
- 好接入：遵循拿来主义原则，在系统设计和实现上要尽可能的简单
- 趋势递增：最好趋势递增，这个要求就得看具体业务场景了，一般不严格要求

# 1. UUID
[UUID](https://www.jianshu.com/p/da6dae36c290) 是指Universally Unique Identifier，翻译为中文是**通用唯一识别码**，UUID 的目的是让分布式系统中的所有元素都能有唯一的识别信息。形式为 8-4-4-4-12，总共有 36个字符。用起来非常简单
```java
import java.util.UUID;
	public static void main(String[] args) {
		String uuid = UUID.randomUUID().toString().replaceAll("-","");
		System.out.println(uuid);
	}
```
输出结果 `99a7d0925b294a53b2f4db9d5a3fb798`，但UUID却并不适用于实际的业务需求。订单号用UUID这样的字符串没有丝毫的意义，看不出和订单相关的有用信息；而对于数据库来说用作业务主键ID，它不仅是**太长**还是**字符串**，存储性能差查询也很耗时，所以不推荐用作分布式ID。

`优点`：生成足够简单，本地生成无网络消耗，具有唯一性
`缺点`：无序的字符串，不具备趋势自增特性，没有具体的业务含义。如此长的字符串当MySQL主键并非明智选择。
# 2. 基于数据库自增ID
基于数据库的auto_increment自增ID完全可以充当分布式ID，具体实现：需要一个单独的MySQL实例用来生成ID，建表结构如下：
```java
CREATE DATABASE `SoWhat_ID`;
CREATE TABLE SoWhat_ID.SEQUENCE_ID (
    `id` bigint(20) unsigned NOT NULL auto_increment, 
    `value` char(10) NOT NULL default '',
    `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (id),
) ENGINE=MyISAM;
insert into SEQUENCE_ID(value) VALUES ('values');
```
当我们需要一个ID的时候，向表中插入一条记录返回主键ID，但这种方式有一个比较致命的缺点，访问量激增时`MySQL本身就是系统的瓶颈`，用它来实现分布式服务风险比较大，不推荐！

`优点`：实现简单，ID单调自增，数值类型查询速度快

`缺点`：DB单点存在宕机风险，无法扛住高并发场景

# 
# 3. 基于数据库集群模式
前边说了单点数据库方式不可取，那对上边的方式做一些高可用优化，换成主从模式集群。害怕一个主节点挂掉没法用，那就做双主模式集群，也就是两个Mysql实例都能单独的生产自增ID。那这样还会有个问题，两个MySQL实例的自增ID都从1开始，会生成重复的ID怎么办？
`解决方案`：[设置](https://www.cnblogs.com/olinux/p/6518766.html)起始值和自增步长

MySQL_1 配置：
```sql 
set @@auto_increment_offset = 1;     -- 起始值
set @@auto_increment_increment = 2;  -- 步长
```
MySQL_2 配置：
```sql
set @@auto_increment_offset = 2;     -- 起始值
set @@auto_increment_increment = 2;  -- 步长
```
这样两个MySQL实例的自增ID分别就是：
```
1、3、5、7、9 
2、4、6、8、10
```
但是如果两个还是无法满足咋办呢？增加第三台MySQL实例需要人工修改一、二两台MySQL实例的起始值和步长，把第三台机器的ID起始生成位置设定在比现有最大自增ID的位置远一些，但必须在一、二两台MySQL实例ID还没有增长到第三台MySQL实例的起始ID值的时候，否则自增ID就要出现重复了，必要时可能还需要停机修改。

`优点`：解决DB单点问题

`缺点`：不利于后续扩容，而且实际上单个数据库自身压力还是大，依旧无法满足高并发场景。

# 4. 基于数据库的号段模式
号段模式是当下分布式ID生成器的主流实现方式之一，号段模式可以理解为`从数据库批量的获取自增ID`，每次从数据库取出一个号段范围，例如 (1,1000] 代表1000个ID，具体的业务服务将本号段，生成1~1000的自增ID并加载到内存。表结构如下：
```sql
CREATE TABLE id_generator (
  `id` int(10) NOT NULL,
  `max_id` bigint(20) NOT NULL COMMENT '当前最大id',
  `step` int(20) NOT NULL COMMENT '号段的步长',
  `biz_type`    int(20) NOT NULL COMMENT '业务类型',
  `version` int(20) NOT NULL COMMENT '版本号',
  PRIMARY KEY (`id`)
)
```
- max_id ：当前最大的可用id
- step ：代表号段的长度
- biz_type ：代表不同业务类型
- version ：是一个乐观锁，每次都更新version，保证并发时数据的正确性


| id | biz_type | max_id	|step	 | version| 
|--|--|--|--|--|
|1	|101|	1000|	2000|	0|
等这批号段ID用完，再次向数据库申请新号段，对max_id字段做一次`update`操作，update max_id= max_id + step，update成功则说明新号段获取成功，新的号段范围是(max_id ,max_id +step]。
```sql
update id_generator set max_id = {max_id+step}, version = version + 1
 where version =  {version} and biz_type = XX
```
由于多业务端可能同时操作，所以采用版本号 version `乐观锁`方式更新，这种分布式ID生成方式不强依赖于数据库，不会频繁的访问数据库，对数据库的压力小很多。但是如果遇到了双十一或者秒杀类似的活动还是会对数据库有比较高的访问。
# 5. 基于Redis模式
[Redis](https://www.cnblogs.com/huigelaile/p/10888339.html) 也同样可以实现，原理就是Redis 是单线程的，因此我们可以利用redis的` incr `命令实现ID的原子性自增。
```bash
127.0.0.1:6379> set seq_id 1     // 初始化自增ID为1
OK
127.0.0.1:6379> incr seq_id      // 增加1，并返回递增后的数值
(integer) 2
```
用redis实现需要注意一点，要考虑到redis持久化的问题。redis有两种持久化方式RDB和AOF。
# 6. 基于雪花算法（Snowflake）模式
SnowFlake 算法，是 Twitter 开源的分布式 id 生成算法。其核心思想就是：**使用一个 64 bit 的 long 型的数字作为全局唯一 id**。在分布式系统中的应用十分广泛，且ID 引入了时间戳，为什么叫雪花算法呢？私以为众所周知**世界上没有一对相同的雪花**。雪花算法基本上保持自增的，后面的代码中有详细的注解。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728151027172.png#pic_center)
这 64 个 bit 中，其中 1 个 bit 是不用的，然后用其中的 41 bit 作为毫秒数，用 10 bit 作为工作机器 id，12 bit 作为序列号。举例如上图：
1. 第一个部分是 1 个 bit：0，
这个是无意义的。因为二进制里第一个 bit 位如果是 1，那么都是负数，但是我们生成的 id 都是正数，所以第一个 bit 统一都是 0。
2. 第二个部分是 41 个 bit：表示的是时间戳。单位是毫秒。
41 bit 可以表示的数字多达 2^41 - 1，也就是可以标识 2 ^ 41 - 1 个毫秒值，换算成年就是表示 `69` 年的时间。
3. 第三个部分是 5  个 bit：表示的是机房 id
5 个 bit 代表机器 id。意思就是最多代表 2 ^ 5 个机房（32 个机房）
4. 第四个部分是 5  个 bit：表示的是机器 id。每个机房里可以代表 2 ^ 5 个机器（32 台机器），也可以根据自己公司的实际情况确定。
5. 第五个部分是 12 个 bit：表示的序号，就是某个机房某台机器上这**一毫秒**内同时生成的 id 的序号。
12 bit 可以代表的最大正整数是 2 ^ 12 - 1 = 4096，也就是说可以用这个 12 bit 代表的数字来区分同一个毫秒内的 4096 个不同的 id。

总结：
简单来说，你的某个服务假设要生成一个全局唯一 id，那么就可以发送一个请求给部署了 SnowFlake 算法的系统，由这个 SnowFlake 算法系统来生成唯一 id。

这个 SnowFlake 算法系统首先肯定是知道自己所在的机房和机器的，比如机房 id = 17，机器 id = 12。

接着 SnowFlake 算法系统接收到这个请求之后，首先就会用二进制位运算的方式生成一个 64 bit 的 long 型 id，64 个 bit 中的第一个 bit 是无意义的。

接着 41 个 bit，就可以用当前时间戳（单位到毫秒），然后接着 5 个 bit 设置上这个机房 id，还有 5 个 bit 设置上机器 id。

最后再判断一下，当前这台机房的这台机器上这一毫秒内，这是**第几个请求**，给这次生成 id 的请求累加一个序号，作为最后的 12 个 bit。最终一个 64 个 bit 的 id 就出来了，类似于：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728151027172.png#pic_center)
这个算法可以保证一个机房的一台机器在同一毫秒内，生成了一个唯一的 id。可能一个毫秒内会生成多个 id，但是有最后 12 个 bit 的序号来区分开来。

总结：就是用一个 64 bit 的数字中各个 bit 位来设置不同的标志位，区分每一个 id。
 
SnowFlake 算法的实现代码如下：
```java
/**
 * 雪花算法相对来说如果思绪捋顺了实现起来比较简单，前提熟悉位运算。
 */
public class SnowFlake
{
	/**
	 * 开始时间截 (2015-01-01)
	 */
	private final long twepoch = 1420041600000L;

	/**
	 * 机器id所占的位数
	 */
	private final long workerIdBits = 5L;

	/**
	 * 数据标识id所占的位数
	 */
	private final long dataCenterIdBits = 5L;

	/**
	 * 支持的最大机器id，结果是31 (这个移位算法可以很快的计算出几位二进制数所能表示的最大十进制数)
	 */
	private final long maxWorkerId = ~(-1L << workerIdBits);

	/**
	 * 支持的最大机房标识id，结果是31
	 */
	private final long maxDataCenterId = ~(-1L << dataCenterIdBits);

	/**
	 * 序列在id中占的位数
	 */
	private final long sequenceBits = 12L;

	/**
	 * 机器ID向左移12位
	 */
	private final long workerIdShift = sequenceBits;

	/**
	 * 机房标识id向左移17位(12+5)
	 */
	private final long dataCenterIdShift = sequenceBits + workerIdBits;

	/**
	 * 时间截向左移22位(5+5+12)
	 */
	private final long timestampLeftShift = sequenceBits + workerIdBits + dataCenterIdBits;

	/**
	 * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
	 */
	private final long sequenceMask = ~(-1L << sequenceBits);

	/**
	 * 工作机器ID(0~31)
	 */
	private volatile long workerId;

	/**
	 * 机房中心ID(0~31)
	 */
	private volatile long dataCenterId;

	/**
	 * 毫秒内序列(0~4095)
	 */
	private volatile long sequence = 0L;

	/**
	 * 上次生成ID的时间截
	 */
	private volatile long lastTimestamp = -1L;

	//==============================Constructors=====================================

	/**
	 * 构造函数
	 *
	 * @param workerId     工作ID (0~31)
	 * @param dataCenterId 机房中心ID (0~31)
	 */

	public SnowFlake(long workerId, long dataCenterId)
	{
		if (workerId > maxWorkerId || workerId < 0)
		{
			throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
		}
		if (dataCenterId > maxDataCenterId || dataCenterId < 0)
		{
			throw new IllegalArgumentException(String.format("dataCenter Id can't be greater than %d or less than 0", maxDataCenterId));
		}
		this.workerId = workerId;
		this.dataCenterId = dataCenterId;
	}

	// ==============================Methods==========================================

	/**
	 * 获得下一个ID (该方法是线程安全的)
	 * 如果一个线程反复获取Synchronized锁，那么synchronized锁将变成偏向锁。
	 *
	 * @return SnowflakeId
	 */
	public synchronized long nextId() throws RuntimeException
	{
		long timestamp = timeGen();

		//如果当前时间小于上一次ID生成的时间戳，说明系统时钟回退过这个时候应当抛出异常
		if (timestamp < lastTimestamp)
		{
			throw new RuntimeException((String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp)));

		}

		//如果是毫秒级别内是同一时间生成的，则进行毫秒内序列生成
		if (lastTimestamp == timestamp)
		{
			sequence = (sequence + 1) & sequenceMask;
			//毫秒内序列溢出，一毫秒内超过了4095个
			if (sequence == 0)
			{
				//阻塞到下一个毫秒,获得新的时间戳
				timestamp = tilNextMillis(lastTimestamp);
			}
		}
		else
		{
			//时间戳改变，毫秒内序列重置
			sequence = 0L;
		}

		//上次生成ID的时间截
		lastTimestamp = timestamp;

		//移位并通过或运算拼到一起组成64位的ID
		return ((timestamp - twepoch) << timestampLeftShift)
				| (dataCenterId << dataCenterIdShift)
				| (workerId << workerIdShift)
				| sequence;
	}

	/**
	 * 阻塞到下一个毫秒，直到获得新的时间戳
	 * @param lastTimestamp 上次生成ID的时间截
	 * @return 当前时间戳
	 */
	private long tilNextMillis(long lastTimestamp)
	{
		long timestamp = timeGen();
		while (timestamp <= lastTimestamp)
		{
			timestamp = timeGen();
		}
		return timestamp;
	}

	/**
	 * 返回以毫秒为单位的当前时间
	 * @return 当前时间(毫秒)
	 */
	private long timeGen()
	{
		return System.currentTimeMillis();
	}
}
```
**SnowFlake算法的优点**：
- 高性能高可用：生成时不依赖于数据库，完全在内存中生成。
- 容量大：每秒中能生成数百万的自增ID。
- ID自增：存入数据库中，索引效率高。

**SnowFlake算法的缺点**：
- 依赖与系统时间的一致性，如果系统时间被回调，或者改变，可能会造成id冲突或者重复。
 
实际中我们的机房并没有那么多，我们可以改进改算法，将10bit的机器id优化成业务表或者和我们系统相关的业务。

# 7. 百度uid-generator
项目GitHub地址： [https://github.com/baidu/uid-generator](https://github.com/baidu/uid-generator)，uid-generator是由百度技术部开发，基于Snowflake算法实现的，与原始的snowflake算法不同在于，uid-generator支持自定义时间戳、工作机器ID和 序列号等各部分的位数，而且uid-generator中采用用户自定义workId的生成策略。

uid-generator需要与数据库配合使用，需要新增一个WORKER_NODE表。当应用启动时会向数据库表中去插入一条数据，插入成功后返回的自增ID就是该机器的workId数据由host，port组成。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728182153779.png#pic_center)
由上图可知，UidGenerator的时间部分只有28位，这就意味着UidGenerator默认只能承受8.5年（2^28-1/86400/365）。当然，根据你业务的需求，UidGenerator可以适当调整delta seconds、worker node id和sequence占用位数。

接下来分析百度UidGenerator的实现。需要说明的是UidGenerator有两种方式提供：和`DefaultUidGenerator`和`CachedUidGenerator`。我们先分析比较容易理解的`DefaultUidGenerator`。
### DefaultUidGenerator
**delta seconds**
这个值是指当前时间与epoch时间的时间差，且单位为**秒**。epoch时间就是指集成UidGenerator生成分布式ID服务第一次上线的时间，可配置，也一定要根据你的上线时间进行配置，因为默认的epoch时间可是2016-09-20，不配置的话，会浪费好几年的可用时间。

**worker id**
接下来说一下UidGenerator是如何给worker id赋值的，搭建UidGenerator的话，需要创建一个表：![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728182425588.png#pic_center)
`UidGenerator`会在集成用它生成分布式ID的实例启动的时候，往这个表中插入一行数据，得到的id值就是准备赋给workerId的值。由于workerId默认22位，那么，集成UidGenerator生成分布式ID的所有实例重启次数是不允许超过4194303次（即2^22-1），否则会抛出异常。

这段逻辑的核心代码来自DisposableWorkerIdAssigner.java中，当然，你也可以实现`WorkerIdAssigner.java`接口，自定义生成workerId。
**sequence**
核心代码如下，几个实现的关键点：
- synchronized保证线程安全。
- 如果时间有任何的回拨，那么直接抛出异常。
- 如果当前时间和上一次是同一秒时间，那么sequence自增。如果同一秒内自增值超过2^13-1，那么就-- 会自旋等待下一秒（getNextSecond）。
- 如果是新的一秒，那么sequence重新从0开始。
```java
/**
     * Get UID
     *
     * @return UID
     * @throws UidGenerateException in the case: Clock moved backwards; Exceeds the max timestamp
     */
    protected synchronized long nextId() {
        long currentSecond = getCurrentSecond();
        // Clock moved backwards, refuse to generate uid
        if (currentSecond < lastSecond) {
            long refusedSeconds = lastSecond - currentSecond;
            throw new UidGenerateException("Clock moved backwards. Refusing for %d seconds", refusedSeconds);
        }
        // At the same second, increase sequence
        if (currentSecond == lastSecond) {
            sequence = (sequence + 1) & bitsAllocator.getMaxSequence();
            // Exceed the max sequence, we wait the next second to generate uid
            if (sequence == 0) {
                currentSecond = getNextSecond(lastSecond);
            }
        // At the different second, sequence restart from zero
        } else {
            sequence = 0L;
        }
        lastSecond = currentSecond;
        // Allocate bits for UID
        return bitsAllocator.allocate(currentSecond - epochSeconds, workerId, sequence);
    }
```
**总结**
通过DefaultUidGenerator的实现可知，它对时钟回拨的处理比较简单粗暴。另外如果使用UidGenerator的DefaultUidGenerator方式生成分布式ID，一定要根据你的业务的情况和特点，调整各个字段占用的位数：
```java
<property name="timeBits" value="28"/>
<property name="workerBits" value="22"/>
<property name="seqBits" value="13"/>
<property name="epochStr" value="2016-09-20"/>
```
### CachedUidGenerator
`CachedUidGenerator`是`UidGenerator`的重要改进实现。它的核心利用了`RingBuffer`，如下图所示，它本质上是一个数组，数组中每个项被称为slot。UidGenerator设计了两个RingBuffer，一个保存唯一ID，一个保存flag。RingBuffer的尺寸是2^n，n必须是正整数：![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728182927511.png#pic_center)
具体细节阅读Git源码即可，可以直接通过 [SpringBoot](https://blog.csdn.net/m0_37367413/article/details/87341352) 集成开发使用。

# 8. 美团（Leaf）
Leaf由美团开发，github地址：[https://github.com/Meituan-Dianping/Leaf](https://github.com/Meituan-Dianping/Leaf)，Leaf同时支持号段模式和snowflake算法模式，可以 [切换使用](https://www.jianshu.com/p/bd6b00e5f5ac)。
### 号段模式
先导入源码 https://github.com/Meituan-Dianping/Leaf ，在建一张表leaf_alloc
```sql
DROP TABLE IF EXISTS `leaf_alloc`;
CREATE TABLE `leaf_alloc` (
  `biz_tag` varchar(128)  NOT NULL DEFAULT '' COMMENT '业务key',
  `max_id` bigint(20) NOT NULL DEFAULT '1' COMMENT '当前已经分配了的最大id',
  `step` int(11) NOT NULL COMMENT '初始步长，也是动态调整的最小步长',
  `description` varchar(256)  DEFAULT NULL COMMENT '业务key的描述',
  `update_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '数据库维护的更新时间',
  PRIMARY KEY (`biz_tag`)
) ENGINE=InnoDB;
```
然后在项目中开启号段模式，配置对应的数据库信息，并关闭snowflake模式
```xml
leaf.name=com.sankuai.leaf.opensource.test
leaf.segment.enable=true
leaf.jdbc.url=jdbc:mysql://localhost:3306/leaf_test?useUnicode=true&characterEncoding=utf8&characterSetResults=utf8
leaf.jdbc.username=root
leaf.jdbc.password=root

leaf.snowflake.enable=false
#leaf.snowflake.zk.address=
#leaf.snowflake.port=
```
启动leaf-server 模块的 LeafServerApplication项目就跑起来了
号段模式获取分布式自增ID的测试url ：http：//localhost：8080/api/segment/get/leaf-segment-test
监控号段模式：http://localhost:8080/cache

### snowflake模式
Leaf的snowflake模式依赖于`ZooKeeper`，不同于原始snowflake算法也主要是在workId的生成上，Leaf中workId是基于ZooKeeper的顺序Id来生成的，每个应用在使用Leaf-snowflake时，启动时都会都在Zookeeper中生成一个顺序Id，相当于一台机器对应一个顺序节点，也就是一个workId。
```xml
leaf.snowflake.enable=true
leaf.snowflake.zk.address=127.0.0.1
leaf.snowflake.port=2181
```
snowflake模式获取分布式自增ID的测试url：http://localhost:8080/api/snowflake/get/test
# 9. 滴滴（Tinyid）
[Tinyid](https://mp.weixin.qq.com/s/P6LG8HQpQ1og3eIuNKQcYw) 由滴滴开发，Github地址：[https://github.com/didi/tinyid](https://github.com/didi/tinyid)

Tinyid是一个ID生成器服务，它提供了REST API和Java客户端两种获取方式，如果使用Java客户端获取方式的话，官方宣称能单实例能达到1kw QPS（Over10 million QPSper single instance when using the java client.）

[Tinyid教程](https://www.jianshu.com/p/01d8365d383a) 的原理非常简单，通过数据库表中的数据基本是就能猜出个八九不离十，就是经典的segment模式，和美团的leaf原理几乎一致。原理图如下所示，以同一个bizType为例，每个tinyid-server会分配到不同的segment，例如第一个tinyid-server分配到(1000, 2000]，第二个tinyid-server分配到(2000, 3000]，第3个tinyid-server分配到(3000, 4000]：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200728185804974.png#pic_center)
再以第一个tinyid-server为例，当它的segment用了20%（核心源码：segmentId.setLoadingId(segmentId.getCurrentId().get() + idInfo.getStep() * Constants.LOADING_PERCENT / 100);，LOADING_PERCENT的值就是20），即设定loadingId为20%的阈值，
例如当前id是10000，步长为10000，那么loadingId=12000。那么当请求分布式ID分配到12001时（或者重启后），即超过loadingId，就会返回一个特殊code：new Result(ResultCode.LOADING, id);tinyid-server根据ResultCode.LOADING这个响应码就会异步分配下一个segment(4000, 5000]，以此类推。具体使用 [参考](https://zhuanlan.zhihu.com/p/98657689)

