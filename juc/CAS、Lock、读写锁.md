
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200316230626942.png)
# CAS
什么是原子(atomic)操作：
>多线程中的原子操作类似于数据库中的同时执行AB两个语句，要么同时执行成功，要么同时执行失败。
>
[synchronized](https://mp.weixin.qq.com/s/e_fYFWK5Qnxjmz6Abi7uqw) 的不足：
> 1. syn是基于阻塞的锁机制，颗粒度还是比较大 的。
> 2.  如果被阻塞的线程优先级很高怎么办。
> 3. 拿到锁的线程一直不释放锁怎么办。
> 4. 如果出现大量竞争会消耗CPU，同时带来死锁或其他安全隐患。

用syn也可以实现原子操作不过不太合适，目前CPU指令级别实现了原子性的比较和交换（`Conmpare And Swap`）操作(CAS`不是锁`只是CPU提供的一个原子性操作指令哦切记)。

CAS的实现步骤如下
> 1. 获得L(内存地址)上的数据初始值D1
> 2. 对D1的数据进行增减后最终等到D2
> 3. 尝试将D2 放到原来L的位置上
> 4. 放之前先比较目前L里的数据是否跟我之前取出的D1值跟版本号都对应。
> 5. 对应了 我就将数据放到L中，但有一个不对应则写入失败。重新执行步骤1.
> 6.  上面的步骤如果失败了就会重复进入一个1～5的死循环，俗称`自旋`。
> 
`CAS`在语言层面不进行任何处理，直接将原则操作实现在`硬件`级别实现，只所以可以实现`硬件级别`的操作核心是因为CAS操作类中有个核心类`UnSafe`类，

`Java`和`C++`语言的一个重要区别就是Java中我们无法直接操作一块内存区域，不能像C++中那样可以自己申请内存和释放内存。`Java`中的`Unsafe`类为我们提供了类似C++手动管理内存的能力。`Unsafe`类，全限定名是`sun.misc.Unsafe`，`UnSafe`类中所有的方法都是`native`修饰的，也就是说`UnSafe`类中的方法都是`直接`调用操作底层资源执行响应的任务。主要功能如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200316223535907.png)
用CAS的弊端：
1. ABA 问题
>  现象：在内存中数据变化为A==>B==>A,这样如何判别，因为这样其实数据已经修改过了。
>  解决方法：引入`版本号`
2. 开销问题
> 如果长期不成功那就会进入自旋。
> JVM支持处理器提供的**pause**指令，使得效率会有一定的提升，pause指令有两个作用：
> 1. 它可以**延迟**流水线执行指令，使CPU不会消耗过多的执行资源，
> 2. 它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。
3.  只能保证一个共享变量之间的原则性操作
>  问题描述：当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对**多个**共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁来保证原子性。
解决办法：从JDK5开始提供了**AtomicReference**类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行**CAS**操作。

JDK中相关原子操作类的使用
> 1. 更新基本类型类：AtomicBoolean，AtomicInteger，AtomicLong
> 2. 更新数组类：AtomicIntegerArray，AtomicLongArray，AtomicReferenceArray
	> 3. 更新引用类型：AtomicStampedReference，AtomicMarkableReference，AtomicReference
> 4. 原子更新字段类： AtomicReferenceFieldUpdater，AtomicIntegerFieldUpdater，AtomicLongFieldUpdater
> 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200316155113261.png)
相互之间差别不太，我们以AtomicInteger为例，常用方法:
> 1. get()
> 2. set(int)
> 3. getAndIncrement()
> 4. incrementAndGet()
> ...

AtomicInteger 例子：
```java
    static AtomicInteger ai = new AtomicInteger(10);
    public static void main(String[] args) {
        System.out.println(ai.getAndIncrement());
        //10--->11
        System.out.println(ai.incrementAndGet());
        //11--->12--->out
        System.out.println(ai.get());
    }
```
### LongAdder
本来的初衷是通过CAS操作来进行原子性的简单累加计数功能，但是在并发很大的情况下，因为每次CAS都只有一个线程能成功，竞争失败的线程会非常多。失败次数越多，循环次数就越多，很多线程的CAS操作越来越接近 自旋锁（spin lock）。计数操作本来是一个很简单的操作，实际需要耗费的cpu时间应该是越少越好，AtomicXXX在高并发计数时，大量的cpu时间都浪费会在 自旋 上了，这很浪费，也降低了实际的计数效率。
```java
// jdk1.8的AtomicLong的实现代码，这段代码在sun.misc.Unsafe中
// 当线程竞争很激烈时，while判断条件中的CAS会连续多次返回false，这样就会造成无用的循环，循环中读取volatile变量的开销本来就是比较高的
// 因为这样，在高并发时，AtomicXXX并不是那么理想的计数方式
public final long getAndAddLong(Object o, long offset, long delta) {
long v;
do {
v = getLongVolatile(o, offset);
} while (!compareAndSwapLong(o, offset, v, v + delta));// 自旋
return v;
}
```
[LongAdder](https://www.cnblogs.com/yaowen/p/11250204.html) 是根据 [ConcurrentHashMap](https://mp.weixin.qq.com/s/CjuT8SGKNS4eR_gjhrubGg)这类为并发设计的类的基本原理(锁分段)来实现的，它里面维护一组按需分配的计数单元，并发计数时，不同的线程可以在不同的计数单元上进行计数，这样减少了线程竞争，提高了并发效率。本质上是用`空间换时间`的思想，不过在实际高并发情况中消耗的空间可以忽略不计。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020032810135578.png)

用引用类型`AtomicReference`包装`user`对象，然后修改包装后的对象，`user`本身参数是不变的这点要切记。
```java
public class UseAtomicReference {
	static AtomicReference<UserInfo> userRef = new AtomicReference<UserInfo>();
    public static void main(String[] args) {
        UserInfo user = new UserInfo("sowhat", 14);//要修改的实体的实例
        userRef.set(user); // 引用包装后，包装里面的类跟包装前是两个不同的对象。
        
        UserInfo updateUser = new UserInfo("liu", 12);//要变化的新实例
        userRef.compareAndSet(user, updateUser);
        System.out.println(userRef.get().getName());
        System.out.println(userRef.get().getAge());

        System.out.println(user.getName()); // 注意此时的user 属性
        System.out.println(user.getAge());        
    }
    //定义一个实体类
    static class UserInfo {
        private String name;
        private int age;
        public UserInfo(String name, int age) {
            this.name = name;
            this.age = age;
        }
        public String getName() {
            return name;
        }
        public int getAge() {
            return age;
        }
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203201415692.png)

ABA问题：JDK提供了两个类
> 1. AtomicStampedReference： 返回Boolean值，关心的是动没动过。
> 2. AtomicMarkableReference：关心的是动过几次。

我们以`AtomicStampedReference`为例分析：
重点函数如下：
1.  `AtomicStampedReference(V initialRef, int initialStamp)`，V表示要CAS的数据，int表示初始化版本。
2. `public V getReference() ` 表示获得CAS里面的数据
3. `public int getStamp()` 表示获得当前CAS版本号
4.  第一个参数是原来的CAS中原来参数，第二个参数是要替换后的新参数，第三个参数是原来CAS数据对于版本号，第四个参数表示替换后的新参数版本号。
```java
public boolean compareAndSet(V   expectedReference,
                                 V   newReference,
                                 int expectedStamp,
                                 int newStamp)
```
具体demo加深理解如下：
```java
public class UseAtomicStampedReference {
static AtomicStampedReference<String> asr = new AtomicStampedReference<>("sowhat", 0);
    public static void main(String[] args) throws InterruptedException {
        final int oldStamp = asr.getStamp();  // 那初始的版本号
        final String oldReferenc = asr.getReference(); // 初始数据

        System.out.println(oldReferenc + "----------" + oldStamp);

        Thread rightStampThread = new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println(Thread.currentThread().getName()
                        + "当前变量值：" + oldReferenc + "当前版本戳：" + oldStamp + "-"
                        + asr.compareAndSet(oldReferenc, oldReferenc + "Java",
                        oldStamp, oldStamp + 1));
            }
        });

        Thread errorStampThread = new Thread(new Runnable() {
            @Override
            public void run() {
                String reference = asr.getReference();
                System.out.println(Thread.currentThread().getName()
                        + "当前变量值：" + reference + "当前版本戳：" + asr.getStamp() + "-"
                        + asr.compareAndSet(reference, reference + "C",
                        oldStamp, oldStamp + 1)); //此处版本号用错了
            }
        });

        rightStampThread.setName("对的线程");
        rightStampThread.start();
        rightStampThread.join();

        errorStampThread.setName("错的线程");
        errorStampThread.start();
        errorStampThread.join();
        System.out.println(asr.getReference() + "------------" + asr.getStamp());

    }
}
```
# 显示锁Lock
在JDK5以后引入了`Lock`这个`interface`，跟syn对比如下：
> 1、Synchronized 是Java内置关键字，Lock是Java接口
2、Synchronized 无法判断获取锁状态，Lock可以判断是否获取到锁了
3、Synchronized 会自动释放锁，Lock必须手动释放锁。
4、Synchronized 后来线程会傻傻等待锁，Lock锁可能存在加塞 不会苦等。
5、Synchronized 是可重入锁，不可以中断。Lock 可重入锁，可以设置锁是否公平。
6、Synchronized 适合少量代码块同步，Lock适合大量代码块。
7、Synchronized 不可timeout，Lock可以timeout
8、Synchronized 可能涉及到monitor 用户态到CPU态切换，Lock不会涉及切换，效率更好些。

**重点**：`ReentrantLock`底层实现依赖于特殊的CPU指令，比如发送lock指令和unlock指令，不需要用户态和内核态的切换，所以效率高（这里和[volatile](https://mp.weixin.qq.com/s/MY7moP4SNnJnKaSKy7WuPA)底层原理类似），而 [synchronized](https://mp.weixin.qq.com/s/e_fYFWK5Qnxjmz6Abi7uqw) 底层由监视器锁（monitor）是依赖于底层的操作系统的Mutex Lock需要用户态和内核态的切换，所以效率低。


PS 可重入含义：
>同一线程外层函数获得锁后，内层**递归函数**仍能获取该锁的代码。在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。也就是说，线程可以进入任何一个它已经拥有的锁所同步的代码块。
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200316192101579.png)
###  ReentrantLock
日常经常用Lock的实现类`ReentrantLock`：
```java
public class LockDemo {
	private Lock lock  = new ReentrantLock();
	private int count;
	public void increament() {
		lock.lock();
		try {
			count++;
		}finally {
			lock.unlock();
		}
	}
	public synchronized void incr2() {
		count++;
		incr2();
	}
	// 可重入锁  底层类似 累加器 锁的调用
	public synchronized void test3() {
		incr2();
	}
}
```
Lock还有一个重点是可以实现`公平锁`跟`非公平锁`：
>如果在时间上，先对锁进行获取对请求，一定被先满足则锁**公平**的。如果不满足就是**非公平**的。
1. 公平锁 ：比如ABC三个任务去抢同一个锁，A先获得 BC就要被依此挂起，BC被挂起就相当于秋名山上的AE86被喊停了，等A再用完锁，BC才获得锁再起步这个过程对于CPU来说是很**耗时**的。
>挂起：主动的行为，需要的时候再让其工作，操作系统就不给这个现成分配cpu资源
睡眠：主动的行为，到时间后自动回复。
阻塞：被动的状态，在等待某个事件或者资源表现，一旦成立则回归就绪状态。

对线程的控制就好比你控制了一个雇工为你干活。你对雇工的控制是通过编程来实现的。
>1. 挂起线程的意思就是：你对主动对雇工说：“你睡觉去吧，用着你的时候我主动去叫你，然后接着干活”。
>2. 线程睡眠的意思就是：你主动对雇工说 “你睡觉去吧，**某时某刻过来报到**，然后接着干活”。
>3. 线程阻塞的意思就是：你突然发现，你的雇工不知道在什么时候没经过你允许，自己睡觉呢，但是你不能怪雇工，肯定你这个雇主没注意，本来你让雇工扫地，结果扫帚被偷了或被邻居家借去了，你又没让雇工继续干别的活，他就只好睡觉了。至于扫帚回来后，雇工会不会知道，会不会继续干活，你不用担心，雇工一旦发现扫帚回来了，他就会自己去干活的。因为雇工受过良好的培训。这个培训机构就是操作系统。


2. 非公平锁
比如ABC三个任务抢同一个锁，A获得锁在运行但时间长，而B提交后由于非公平机制会**直接**进行抢锁再执行，如果尝试失败，就再采用类似公平锁那种方式。所以非公平锁相对来说性能会更好些。
ReentrantLock 底层默认实现为`非公平锁`：
```java
   public ReentrantLock() {
        sync = new NonfairSync();// 默认非公平锁
    }
 public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```
syn跟Lock 使用建议：
> 1. 能用syn就用syn，代码更简洁。
> 2. 需要锁可中断，超时获取锁，尝试获取锁时候 用Lock。

### Condition
`synchronized`可用`wait()`和`notify()/notifyAll()`方法相结合可以实现等待/通知模式。`ReentrantLock`也提供了`Condition`来提供类似的功能。
```java
    public Condition newCondition() {
        return sync.newCondition();
    }
```
其中	`Condition`主要函数如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200316213319538.png)
基本跟`syn`的操作差别不大，唯一区别可能就是多来个`a`在方法前面。

#  读写锁
先普及下几个常见的关于锁的名词

互斥锁：
> 在访问共享资源之前会对资源进行加锁操作，在访问完成之后进行解锁操作。 加锁后，任何其他试图再次加锁的线程会被**阻塞**，直到当前进程解锁。通俗来说就是共享资源某一时刻只能有一个线程访问，其余等待。
> 
共享锁：
> 共享锁从字面来看也即是允许多个线程共同访问资源。
>
读写锁：
> 读写锁既是**互斥锁**，又是共享锁，read模式是共享，write是互斥(排它锁)的。
> 一次只有一个线程可以占有写模式的读写锁，但是多个线程可以同时占有读模式的读写锁。

前面说到的`syn`跟`Lock`都是独占锁,JDK 还专门给我们提供了更细致的读写锁，对于读操作因为不改变值可以多个线程同时进行读数据，但是对于出现写操作的时候则将该对象进行Lock，JDK中读写锁的接口是`ReadWriteLock`，该接口其实底层实现就是有**两个锁**，一个**管读**操作，一个管**写操**作，对于多度少写的场景一般比`syn`性能可提速10倍。
```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```
具体实现类是`ReentrantReadWriteLock`。
```java
public class ReentrantReadWriteLock
     implements ReadWriteLock, java.io.Serializable {
    private static final long serialVersionUID = -6992448646407690164L;
    /** Inner class providing readlock  单独的 读锁 */
    private final ReentrantReadWriteLock.ReadLock readerLock;
    /** Inner class providing writelock单独的写锁  */
    private final ReentrantReadWriteLock.WriteLock writerLock;
```
### 锁的降级跟升级
要实现读写锁需要考虑一个问题就是锁升级和锁降级的问题，`ReadWriteLock`的`javadoc`中说明如下：
> Can the write lock be downgraded to a read lock without allowing an intervening writer? Can a read lock be upgraded to a write lock, in preference to other waiting readers or writers?

简言之就是说 锁降级(从写锁变成读锁)跟 锁升级(从读锁变成写锁)，ReadWriteLock是否支持?
1. 锁降级
```java
public class Test {
    public static void main(String[] args) {
        ReentrantReadWriteLock rtLock = new ReentrantReadWriteLock();
        rtLock.writeLock().lock();
        System.out.println("writeLock");

        rtLock.readLock().lock();
        System.out.println("get read lock");
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203224057439.png)

**结论**：`ReentrantReadWriteLock`支持锁降级，上面代码不会产生死锁。这段代码虽然不会导致死锁，但没有正确的释放锁。从写锁降级成读锁，并不会自动释放当前线程获取的写锁，**仍然需要显示的释放**，否则别的线程永远也获取不到写锁。

2. 锁升级
```java
public class Test {

    public static void main(String[] args) {
        ReentrantReadWriteLock rtLock = new ReentrantReadWriteLock();
        rtLock.readLock().lock();
        System.out.println("get readLock.");
        rtLock.writeLock().lock();
        System.out.println("blocking");
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020120322401842.png)

**结论**：结果直接卡死，因为同一个线程中，在没有释放读锁的情况下，就去申请写锁，这属于**锁升级**，ReentrantReadWriteLock是不支持的。

### 读写锁跟Syn性能测试
前面说过对于多度少写的场景一般比`syn`性能可提速10倍。接下来测试读写锁跟syn性能，比如我们是卖娃娃的，有总销售额跟库存数，每减少库存则销售额会增加，我们用多线程来执行。

GoodsInfo 商品信息类
```java
public class GoodsInfo {
    private final String name;
    private double totalMoney;//总销售额
    private int storeNumber;//库存数
    public GoodsInfo(String name, int totalMoney, int storeNumber) {
        this.name = name;
        this.totalMoney = totalMoney;
        this.storeNumber = storeNumber;
    }
    public void changeNumber(int sellNumber){
        this.totalMoney += sellNumber*25;
        this.storeNumber -= sellNumber;
    }
}
```
操作类接口
```java
public interface GoodsService {
    GoodsInfo getNum() throws Exception;//获得商品的信息
    void setNum(int number) throws Exception;//设置商品的数量
}
```
操作类实现Syn
```java
public class UseSyn implements GoodsService {
	private GoodsInfo goodsInfo;
	public UseSyn(GoodsInfo goodsInfo) {
		this.goodsInfo = goodsInfo;
	}
	@Override
	public synchronized GoodsInfo getNum() throws Exception {
		TimeUnit.MILLISECONDS.sleep(5);
		return this.goodsInfo;
	}
	@Override
	public synchronized void setNum(int number) throws Exception {
		TimeUnit.MILLISECONDS.sleep(5);
		goodsInfo.changeNumber(number);
	}
}
```
操作类实现读写锁
```java
public class UseRwLock implements GoodsService {
    private GoodsInfo goodsInfo;
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final Lock getLock = lock.readLock(); //读锁
    private final Lock setLock = lock.writeLock(); //写锁

    public UseRwLock(GoodsInfo goodsInfo) {
        this.goodsInfo = goodsInfo;
    }
    @Override
    public GoodsInfo getNum() {
        getLock.lock();// 加读锁
        try {
            SleepTools.ms(5);
            return this.goodsInfo;
        } finally {
            getLock.unlock();
        }
    }
    @Override
    public void setNum(int number) {
        setLock.lock(); //加写锁
        try {
            SleepTools.ms(5);
            goodsInfo.changeNumber(number);
        } finally {
            setLock.unlock();
        }
    }
}
```
多读少写情况下并发测试。
```java
public class BusiApp {
    static final int readWriteRatio = 10;//读写线程的比例
    static final int minthreadCount = 3;//最少线程数

    //读操作
    private static class GetThread implements Runnable {
        private GoodsService goodsService;
        public GetThread(GoodsService goodsService) {
            this.goodsService = goodsService;
        }
        @Override
        public void run() {
            long start = System.currentTimeMillis();
            for (int i = 0; i < 100; i++) {
                try {
                    goodsService.getNum();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "读取商品数据耗时："
                    + (System.currentTimeMillis() - start) + "ms");
        }
    }

    //写操做
    private static class SetThread implements Runnable {
        private GoodsService goodsService;
        public SetThread(GoodsService goodsService) {
            this.goodsService = goodsService;
        }
        @Override
        public void run() {
            long start = System.currentTimeMillis();
            Random r = new Random();
            for (int i = 0; i < 10; i++) {
                SleepTools.ms(50);
                try {
                    goodsService.setNum(r.nextInt(10));
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            System.out.println(Thread.currentThread().getName() + "写商品数据耗时：" + (System.currentTimeMillis() - start) + "ms---------");
        }
    }

    public static void main(String[] args) throws InterruptedException {
        GoodsInfo goodsInfo = new GoodsInfo("Cup", 100000, 10000);
        //GoodsService goodsService = new UseRwLock(goodsInfo); //单次耗时770ms  用读写锁实现
        GoodsService goodsService =new UseSyn(goodsInfo); //单次耗时 17000ms  用syn实现
        for (int i = 0; i < minthreadCount; i++) {
            Thread setT = new Thread(new SetThread(goodsService));
            for (int j = 0; j < readWriteRatio; j++) {
                Thread getT = new Thread(new GetThread(goodsService));
                getT.start();
            }
            SleepTools.ms(100);
            setT.start();
        }
    }
}
```
Syn性能
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203223856931.png)

读写锁性能
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203223938745.png)

### 互斥还是共享
通过`Syn`跟`ReetrantReadWriteLock`的测试我们可以了解到，读写锁中的读锁使用**共享模式**，也就是说可以同时有多个线程并发地读数据，读锁跟写锁之间是**互斥模式**。

公用代码部分：
```java
public class Test {
    // 读操作
    public static void readFile(Thread thread) {
        lock.readLock().lock();
        boolean readLock = lock.isWriteLocked();
        if (!readLock) {
            System.out.println("当前为读锁！");
        }
        try {
            for (int i = 0; i < 3; i++) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(thread.getName() + ":正在进行读操作……");
            }
            System.out.println(thread.getName() + ":读操作完毕！");
        } finally {
            System.out.println("释放读锁！");
            lock.readLock().unlock();
        }
    }

    // 写操作
    public static void writeFile(Thread thread) {
        lock.writeLock().lock();
        boolean writeLock = lock.isWriteLocked();
        if (writeLock) {
            System.out.println("当前为写锁！");
        }
        try {
            for (int i = 0; i < 3; i++) {
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(thread.getName() + ":正在进行写操作……");
            }
            System.out.println(thread.getName() + ":写操作完毕！");
        } finally {
            System.out.println("释放写锁！");
            lock.writeLock().unlock();
        }
    }
}
```
同时调用读锁：
```java
    public static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    public static void main(String[] args) {
        //同时读、写
        ExecutorService service = Executors.newCachedThreadPool();
        service.execute(new Runnable() {
            @Override
            public void run() {
                readFile(Thread.currentThread());
            }
        });
        service.execute(new Runnable() {
            @Override
            public void run() {
                readFile(Thread.currentThread());
            }
        });
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203225300445.png)
一个读锁一个写锁调用：
```java
    public static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    public static void main(String[] args) {
        //同时读、写
        ExecutorService service = Executors.newCachedThreadPool();
        service.execute(new Runnable() {
            @Override
            public void run() {
                readFile(Thread.currentThread());
            }
        });
        service.execute(new Runnable() {
            @Override
            public void run() {
                writeFile(Thread.currentThread());
            }
        });
    }
```

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203224204971.png)

写锁跟写锁之间是**互斥模式**，跟`Syn`还有`ReentrantLock`一样。
```java
    public static ReentrantReadWriteLock lock = new ReentrantReadWriteLock();
    public static void main(String[] args) {
        //同时读、写
        ExecutorService service = Executors.newCachedThreadPool();
        service.execute(new Runnable() {
            @Override
            public void run() {
                writeFile(Thread.currentThread());
            }
        });
        service.execute(new Runnable() {
            @Override
            public void run() {
                writeFile(Thread.currentThread());
            }
        });
    }
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201203224238251.png)

### 结论
> 1. JUC中`ReetrantReadWriteLock`实现了ReadWriteLock接口并添加了可重入的特性
>2. `ReetrantReadWriteLock`读写锁的效率明显高于 [synchronized](https://mp.weixin.qq.com/s/e_fYFWK5Qnxjmz6Abi7uqw) 关键字，引入如果存在多度少写情况尽量用读写锁。
> 3. `ReetrantReadWriteLock`读写锁的实现中，读锁使用**共享**模式；写锁使用**独占**模式，读锁跟写锁之间是锁**互斥**模式。
> 4. `ReetrantReadWriteLock`读写锁的实现中，需要注意当有读锁时，写锁就**不能**获得；而当有写锁时，除了获得写锁的这个线程可以获得读锁外，其他线程不能获得读锁。

# 参考
[UnSafe](https://www.jianshu.com/p/db8dce09232d)
[CAS](https://blog.csdn.net/lixinkuan328/article/details/94319775)
