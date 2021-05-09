![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403233749811.jpg)
# 使用
悲观锁 `synchronized`关键字是并发编程中线程同步的常用手段之一，其作用有三个:
> 1. 互斥性：确保线程互斥的访问同步代，锁自动释放，多个线程操作同个代码块或函数必须排队获得锁，
> 2. 可见性：保证共享变量的修改能够及时可见，获得锁的线程操作完毕后会将所数据刷新到[共享内存区](https://sowhat.blog.csdn.net/article/details/105257876)
> 3. 有序性：有效解决重排序问题

`synchronized`用法有三个:
>  1. 修饰实例方法
>  2. 修饰静态方法
> 3. 修饰代码块

### 1. 修饰实例方法
`synchronized`关键词作用在方法的前面，用来锁定方法，其实默认锁定的是`this`对象。
```java
public class Thread1 implements Runnable{
    //共享资源(临界资源)
    static int i=0;
    //如果没有synchronized关键字，输出小于20000
    public synchronized void increase(){
        i++;
    }
    public void run() {
        for(int j=0;j<10000;j++){
            increase();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        Thread1 t=new Thread1();
        Thread t1=new Thread(t);
        Thread t2=new Thread(t);
        t1.start();
        t2.start();
        t1.join();//主线程等待t1执行完毕
        t2.join();//主线程等待t2执行完毕
        System.out.println(i);
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403180940471.png)
### 2. 修饰静态方法
`synchronized`还是修饰在方法上，不过修饰的是静态方法，等价于锁定的是`Class`对象，
```java
    public class Thread1 {
    //共享资源(临界资源)
    static int i = 0;
    //如果没有synchronized关键字，输出小于20000
    public static synchronized void increase() {
        i++;
    }
    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(new Runnable() {
            public void run() {
                for (int j = 0; j < 10000; j++) {
                    increase();
                }
            }
        });
        Thread t2 = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int j = 0; j < 10000; j++) {
                    increase();
                }
            }
        });
        t1.start();
        t2.start();
        t1.join();//主线程等待t1执行完毕
        t2.join();//主线程等待t2执行完毕
        System.out.println(i);
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403180940471.png)
### 3. 修饰代码块
用法是在函数体内部对于要修改的参数区间用`synchronized`来修饰，相比与锁定函数这个范围更小，可以指定锁定什么对象。
```java
public class Thread1 implements Runnable {
    //共享资源(临界资源)
    static int i = 0;

    @Override
    public void run() {
        for (int j = 0; j < 10000; j++) {
            //获得了String的类锁
            synchronized (String.class) {
                i++;
            }
        }
    }
    public static void main(String[] args) throws InterruptedException {
        Thread1 t = new Thread1();
        Thread t1 = new Thread(t);
        Thread t2 = new Thread(t);
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(i);
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403180940471.png)
总结：
> 1. synchronized修饰的实例方法，多线程并发访问时，`只能有一个线程进入`，获得对象内置锁，其他线程阻塞等待，但在此期间线程仍然可以访问其他方法。
>2. synchronized修饰的静态方法，多线程并发访问时，只能有一个线程进入，获得类锁，其他线程阻塞等待，但在此期间线程仍然可以访问其他方法。
>3. synchronized修饰的代码块，多线程并发访问时，只能有一个线程进入，根据括号中的对象或者是类，获得相应的对象内置锁或者是类锁
>4. 每个类都有一个类锁，类的每个对象也有一个内置锁，它们是`互不干扰`的，也就是说一个线程可以同时获得类锁和该类实例化对象的内置锁，当线程访问非synchronzied修饰的方法时，并不需要获得锁，因此不会产生阻塞。

# 管程
[管程](https://www.zhihu.com/question/30641734) (英语：Monitors，也称为监视器) 在操作系统中是很重要的概念，管程其实指的是`管理共享变量以及管理共享变量的操作过程`。有点扮演中介的意思，管程管理一堆对象，多个线程同一时候只能有一个线程来访问这些东西。
1. 管程可以看做一个软件模块，它是将共享的变量和对于这些共享变量的操作封装起来，形成一个具有一定接口的功能模块，进程可以调用`管程`来实现进程级别的并发控制。
2. 进程只能互斥的使用管程，即当一个进程使用管程时，另一个进程必须等待。当一个进程使用完管程后，它必须释放管程并唤醒等待管程的某一个进程。

管程解决互斥问题相对简单，把共享变量以及共享变量的操作都封装在一个类中
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403185647725.png)
当线程A和线程B需要获取共享变量count时，就需要调用get和set方法，而get和set方法则保证互斥性，保证每次只能有一个线程访问。

生活中举例管程比如链家店长分配给每一个中介管理一部分二手房源，多个客户通过中介进行房屋买卖。
> 1. 中介 就是管程。
> 2. 多个二手房源被一个中介管理中，就是一个管程管理着多个系统资源。
> 3.  多个客户就相当于多个线程。


# Synchronzied的底层原理
### 对象头解析
我们知道在Java的[JVM内存区域](https://sowhat.blog.csdn.net/article/details/104738411)中一个对象在堆区创建，创建后的对象由三部分组成。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040318214780.png)
这三部分功能如下：
1. `填充数据`：由于虚拟机要求对象起始地址必须是8字节的整数倍。填充数据不是必须存在的，仅仅是为了`字节对齐`。
2. `实例变量`：存放类的`属性数据`信息，包括`父类`的属性信息，这部分内存按4字节对齐。
3. `对象头`：主要包括两部分 `Klass Point`跟 `Mark Word`
>  1. `Klass Point`(类型指针)：是对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例。
> 2. `Mark Word`(标记字段)：这一部分用于储存对象自身的运行时数据，如`哈希码`，`GC`分代年龄，`锁状态标志`，`锁指针`等，这部分数据在32bit和64bit的虚拟机中大小分别为32bit和64bit，考虑到虚拟机的空间效率，Mark Word被设计成一个`非固定`的数据结构以便在极小的空间中存储尽量多的信息，它会根据对象的状态复用自己的存储空间(跟[ConcurrentHashMap](https://mp.weixin.qq.com/s/CjuT8SGKNS4eR_gjhrubGg)里的标志位类似)，详细情况如下图：
> 

`Mark Word`状态表示位如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200402191730608.png)
`synchronized`不论是修饰方法还是代码块，都是通过持有修饰对象的`锁`来实现同步，`synchronized`锁对象是存在对象头`Mark Word`。其中轻量级锁和偏向锁是`Java6`对`synchronized `锁进行优化后新增加的，这里我们主要分析一下重量级锁也就是通常说synchronized的对象锁，锁标识位为10，其中指针指向的是`monitor`对象（也称为管程或监视器锁）的起始地址。每个对象都存在着一个 [monitor](http://www.hollischuang.com/archives/2030) 与之关联。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403193325805.png)
### 反汇编查看
分析对象的`monitor`前我们先通过反汇编看下同步方法跟同步方法块在汇编语言级别是什么样的指令。
```java
public class SynchronizedTest {
    public synchronized void doSth(){
        System.out.println("Hello World");
    }
    public void doSth1(){
        synchronized (SynchronizedTest.class){
            System.out.println("Hello World");
        }
    }
}
```
`javac SynchronizedTest .java` 然后`javap -c SynchronizedTest `反编译后看汇编指令如下：
```java
 public synchronized void doSth();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED //  这是重点 方法锁
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  
         3: ldc           #3    
         5: invokevirtual #4                  
         8: return

  public void doSth1();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #5                 
         2: dup
         3: astore_1
         4: monitorenter  //   进入同步方法
         5: getstatic     #2                  
         8: ldc           #3                  
        10: invokevirtual #4                
        13: aload_1
        14: monitorexit  //正常时 退出同步方法
        15: goto          23
        18: astore_2
        19: aload_1
        20: monitorexit  // 异常时 退出同步方法
        21: aload_2
        22: athrow
        23: return
```
我们可以看到Java编译器为我们生成的字节码。在对于doSth和doSth1的处理上稍有不同。也就是说。JVM对于同步方法和同步代码块的处理方式不同。对于同步方法，JVM采用`ACC_SYNCHRONIZED`标记符来实现同步。 对于同步代码块。JVM采用`monitorenter`、`monitorexit`两个指令来实现同步。

- ACC_SYNCHRONIZED
> 方法级的同步是`隐式`的。同步方法的常量池中会有一个`ACC_SYNCHRONIZED`标志。当某个线程要访问某个方法的时候，会检查是否有`ACC_SYNCHRONIZED`，如果有设置`ACC_SYNCHRONIZED`会去隐式调用刚才的两个指令：monitorenter和monitorexit。则需要先获得`监视器锁`，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放。

- monitorenter跟monitorexit
>  可以把执行`monitorenter`指令理解为加锁，执行`monitorexit`理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行`monitorenter`）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行`monitorexit`指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。

结论：同步方法和同步代码块底层都是通过`monitor`来实现同步的。
两者区别：同步方式是通过方法中的`access_flags`中设置`ACC_SYNCHRONIZED`标志来实现，同步代码块是通过`monitorenter`和`monitorexit`来实现。
### monitor解析
每个对象都与一个`monitor`相关联，而`monitor`可以被线程拥有或释放，在Java虚拟机(HotSpot)中，`monitor`是由`ObjectMonitor`实现的，其主要数据结构如下（位于HotSpot虚拟机源码ObjectMonitor.hpp文件，C++实现的）。
```c
ObjectMonitor() {
    _count        = 0;      //记录数
    _recursions   = 0;      //锁的重入次数
    _owner        = NULL;   //指向持有ObjectMonitor对象的线程 
    _WaitSet      = NULL;   //调用wait后，线程会被加入到_WaitSet
    _EntryList    = NULL ;  //等待获取锁的线程，会被加入到该列表
}
```
monitor运行图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040219215578.png)
对于一个synchronized修饰的方法(代码块)来说：
1. 当多个线程同时访问该方法，那么这些线程会先被放进`_EntryLis`t队列，此时线程处于`blocked`状态
2. 当一个线程获取到了对象的`monitor`后，那么就可以进入`running`状态，执行方法块，此时，`ObjectMonitor`对象的`_owner`指向当前线程，`_count`加1表示当前对象锁被一个线程获取。
3. 当`running`状态的线程调用`wait()`方法，那么当前线程释放`monitor`对象，进入`waiting`状态，`ObjectMonitor`对象的`_owner变为`null，` _count`减1，同时线程进入`_WaitSet`队列，直到有线程调用`notify()`方法唤醒该线程，则该线程进入`_EntryList`队列，竞争到锁再进入`_owner`区。
4. 如果当前线程执行完毕，那么也释放`monitor`对象，`ObjectMonitor`对象的`_owner`变为null，`_count`减1。

因为监视器锁（monitor）是依赖于底层的操作系统的`Mutex Lock`来实现的，而操作系统实现线程之间的切换时需要从`用户态转换到核心态`(具体可看CXUAN写的OS哦)，这个状态之间的转换需要相对比较长的时间，时间成本相对较高，这也是早期的`synchronized`效率低的原因。庆幸在Java 6之后Java官方对从JVM层面对`synchronized`较大优化最终提升显著，Java 6之后，为了减少获得锁和释放锁所带来的性能消耗，引入了锁升级的概念。
# 锁升级
synchronized锁有四种状态，`无锁`、`偏向锁`、`轻量级锁`、`重量级锁`。这几个状态会随着竞争状态逐渐升级，锁可以`升级`但不能`降级`，但是偏向锁状态可以被重置为无锁状态。科学性的说这些锁之前我们先看个简单通俗的例子来加深印象。
### 通俗说各种锁
偏向锁、轻量级锁和重量级锁之间的关系，首先打个[比方](https://blog.csdn.net/xyh930929/article/details/84571805)：假设现在厕所只有一个位置，每个使用者都有打开门锁的钥匙。必须打开门锁才能使用厕所。其中小明、小红理解为两个线程，上厕所理解为执行同步代码，门锁理解为同步代码的锁

>1. 小明今天吃坏了东西需要反复去厕所，如果小明每次都要开锁就很耽误时间，于是门锁将小明的脸记录下来（假设那个锁是智能锁），下次小明再来的时候门锁会自动识别出是小明来了，然后自动开锁，这样就省去了小明拿钥匙开门的过程，此时门锁就是`偏向锁`，也可以理解为偏向小明的锁。

>2. 接下来，小红又去上厕所，试图将厕所的门锁设置为偏向自己的偏向锁，于是发现门锁无法偏向自己，因为此时门锁已是偏向小明的偏向锁。于是小红很生气，要求门锁撤销对小明的偏向，当然，小明也不同意门锁偏向小红。于是等小明用完厕所之后，门锁撤销了对任何人的偏向（只要出现竞争的情况，就会撤销偏向锁）。这个过程就是撤销偏向锁。此时`门锁升级为轻量级锁`。

>3. 等小明出来以后，轻量级锁正式生效 。下一次小明和小红同时来厕所，谁跑的快谁先走到门前，开门后将门锁拿进厕所，并将门锁打开以后拿进厕所里，将门反锁，于是在门外原来放门锁的位置放置了一个`有人`的标志（这个标识可以理解为指向门锁的指针，或者理解为作为锁的Java对象头的`Mark Word`值），这时，小红看到有人以后很着急，催着里面的人出来时马上进去，于是不断的来敲门，问小明什么时候出来。这个过程就是`自旋`。

>4. 反复敲了几次以后，小明受不了了，对小红喊话，说你别敲了，等我用完厕所我告诉你，于是小红去一边等着（线性阻塞）。此时门锁升级为`重量级锁`。升级为重量级锁的后果就是，小红不再反复敲门，小明在上完厕所以后必须告诉小红一声，否则小红就会一直等着。

结论：
>偏向锁在只有一个人上厕所时非常高效，省去了开门的过程。轻量级锁在有多人上厕所但是每个人使用的特别快的时候，比较高效，因为会出现这种现象，小红敲门的时候正好赶上小明出来，这样就省得小明出来告诉小红以后小红才能进去，但是这样可能会出现小红敲门失败的情况（就是敲门时小明还没用完）。重量级锁相比与轻量级锁的多了一步小明呼唤小红的步骤，但是却省掉了小红反复去敲门的过程，但是能保证小红去厕所时厕所一定是没人的。

### 偏向锁
经过HotSpot的作者大量的研究发现大多数时候是`不存在锁竞争`的，经常是一个线程多次获得同一个锁，因此如果每次都要`竞争锁`会增大很多没有必要付出的代价，为了降低获取锁的代价，才引入的偏向锁。
核心思想：
> 如果一个线程获得了锁，那么锁就进入偏向模式，此时`Mark Word` 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作，从而也就提供程序的性能。所以，对于没有锁竞争的场合，偏向锁有很好的优化效果，毕竟极有可能连续多次是同一个线程申请相同的锁。但是对于锁竞争比较激烈的场合，偏向锁就失效了，因为这样场合极有可能每次申请锁的线程都是不相同的，因此这种场合下不应该使用偏向锁，否则会得不偿失，需要注意的是，偏向锁失败后，并不会立即膨胀为重量级锁，而是先升级为轻量级锁。

具体流程：当线程1访问代码块并获取锁对象时，会在java对象头和栈帧中记录偏向的锁的`threadID`，因为`偏向锁不会主动释放锁`，因此以后线程1再次获取锁的时候，需要`比较`当前线程的`threadID`和Java对象头中的`threadID`是否一致，如果一致（还是线程1获取锁对象），则无需使用CAS来加锁、解锁；如果不一致（其他线程，如线程2要竞争锁对象，而偏向锁不会主动释放因此还是存储的线程1的threadID），那么需要查看Java对象头中记录的线程1`是否存活`，如果没有存活，那么锁对象被重置为无锁状态，其它线程（线程2）可以竞争将其设置为偏向锁；如果存活，那么立刻查找该线程（线程1）的栈帧信息，如果还是需要继续持有这个锁对象，那么暂停当前线程1，撤销偏向锁，升级为 `轻量级锁`，如果线程1不再使用该锁对象，那么将锁对象状态设为无锁状态，重新偏向新的线程。
### 轻量级锁
轻量级锁考虑的是竞争锁对象的`线程不多`，而且线程持有锁的`时间也不长`的情景。因为阻塞线程需要高昂的耗时实现CPU从`用户态转到内核态`的切换，如果刚刚阻塞不久这个锁就被释放了，那这个代价就有点得不偿失了，因此这个时候就干脆不阻塞这个线程，让它`自旋`这等待锁释放。

**原理跟升级**：
线程1获取轻量级锁时会先把锁对象的对象头`MarkWord`复制一份到线程1的栈帧中创建的用于存储锁记录的空间（称为DisplacedMarkWord），然后使用CAS把对象头中的内容替换为线程1存储的锁记录（DisplacedMarkWord）的地址；

如果在线程1复制对象头的同时（在线程1CAS之前），线程2也准备获取锁，复制了对象头到线程2的锁记录空间中，但是在线程2CAS的时候，发现线程1已经把对象头换了，**线程2的CAS失败，那么线程2就尝试使用自旋锁来等待线程1释放锁**。 自旋锁简单来说就是让线程2`在循环中不断CAS尝试获得锁对象`。

但是如果自旋的**时间太长**也不行，因为自旋是要消耗CPU的，因此**自旋的次数是有限制**的，比如10次或者100次，如果自旋次数到了线程1还没有释放锁，或者线程1还在执行，线程2还在自旋等待，那么这个时候轻量级锁就会膨胀为重量级锁。重量级锁把除了拥有锁的线程都阻塞，防止CPU空转。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040320532469.png)
### 锁消除
消除锁是虚拟机另外一种锁的优化，这种优化更彻底，Java虚拟机在JIT编译时通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消，除没有必要的锁，可以节省毫无意义的请求锁时间，我们知道`StringBuffer`是线程安全的，里面包含锁的存在，但是如果我们在函数内部使用`StringBuffer`那么代码会在JIT后会自动将锁释放掉哦。

 对比如下：
 
| 锁状态 | 优点 |缺点  |适用场景  |
|--|--|--|--|
| 偏向锁 | 加锁解锁无需额外消耗，跟非同步方法时间相差纳秒级别 | 如果竞争线程多，会带来额外的锁撤销的消耗 | 基本没有其他线程竞争的同步场景 |
| 轻量级锁 |竞争的线程不会阻塞而是在自旋，可提高程序响应速度  | 如果一直无法获得会自旋消耗CPU | 少量线程竞争，持有锁时间不长，追求响应速度 |
| 重量级锁 | 线程竞争不会导致CPU自旋跟消耗CPU资源 | 线程阻塞，响应时间长 | 很多线程竞争锁，切锁持有时间长，追求吞吐量时候 |

PS：ReentrantLock底层实现依赖于特殊的CPU指令，比如发送lock指令和unlock指令，不需要用户态和内核态的切换，所以效率高。而synchronized底层由监视器锁（monitor）是依赖于底层的操作系统的`Mutex Lock`需要用户态和内核态的切换，所以效率会低一些。

### 锁升级流程图
最后奉上[unbelievableme](https://www.cnblogs.com/kundeg/p/8422557.html)绘制的锁升级大图(如果看不清各种回复 syn 即可获得高清大图)：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403214122966.jpg)

# 参考
[头条大佬讲syn](https://blog.csdn.net/javazejian/article/details/72828483)
[阿里hollis将syn](http://www.hollischuang.com/archives/2030)



