# 1、JUC的由来
[synchronized](https://mp.weixin.qq.com/s/e_fYFWK5Qnxjmz6Abi7uqw) 关键字是JDK官方人员用C++代码写的，在JDK6以前是重量级锁。Java大牛 **Doug Lea**对 [synchronized](https://mp.weixin.qq.com/s/e_fYFWK5Qnxjmz6Abi7uqw) 性能不满意就自己写了个JUC，以此来显著提升并发性能，本文要讲的就是JUC并发包下的 **AbstractQueuedSynchronizer**。

在JUC中 CountDownLatch、ThreadPoolExecutor、ReentrantLock、ReentrantReadWriteLock 等底层用的都是AQS，如果想要获取锁可以被中断、超时获取锁、尝试获取锁那就用AQS吧，**AQS几乎占据了JUC并发包里的半壁江山**。

> Doug Lea 杰作比如：  [HashMap](https://mp.weixin.qq.com/s/XGTNaOddY3elcumcPyO1KA)、[ConcurrentHashMap](https://mp.weixin.qq.com/s/CjuT8SGKNS4eR_gjhrubGg)、[JUC](https://mp.weixin.qq.com/s/TlnAeajB8hAfImvuB6yGag) 等。

**温馨提醒**：
> 内容有点长，涉及到AQS重要方法、lock、unlock、CountDownLatch、await、signal几个重要组件的底层讲解。
# 2、AQS前置知识点
###  2.1、模板方法
`AbstractQueuedSynchronizer`是个**抽象类**，所有用到方法的类都要继承此类的若干方法，对应的设计模式就是**模版模式**。
>定义：一个抽象类公开定义了执行它的方法的方式/模板。它的子类可以按需要重写方法实现，但调用将以抽象类中定义的方式进行。这种类型的设计模式属于行为型模式。。

抽象类：
```java
public abstract class SendCustom {
	public abstract void to();
	public abstract void from();
	public void date() {
		System.out.println(new Date());
	}
	public abstract void send();
	// 注意此处 框架方法-模板方法
	public void sendMessage() {
		to();
		from();
		date();
		send();
	}
}
```
模板方法派生类：
```java
public class SendSms extends SendCustom {

	@Override
	public void to() {
		System.out.println("sowhat");
	}

	@Override
	public void from() {
		System.out.println("xiaomai");
	}

	@Override
	public void send() {
		System.out.println("Send message");
	}
	
	public static void main(String[] args) {
		SendCustom sendC = new SendSms();
		sendC.sendMessage();
	}
}
```
### 2.2、LookSupport
**LockSupport** 是一个线程阻塞工具类，所有的方法都是静态方法，可以让线程在任意位置阻塞，当然阻塞之后肯定得有唤醒的方法。常用方法如下：
```java
public static void park(Object blocker); // 暂停当前线程
public static void parkNanos(Object blocker, long nanos); // 暂停当前线程，不过有超时时间的限制
public static void parkUntil(Object blocker, long deadline); // 暂停当前线程，直到某个时间
public static void park(); // 无期限暂停当前线程
public static void parkNanos(long nanos); // 暂停当前线程，不过有超时时间的限制
public static void parkUntil(long deadline); // 暂停当前线程，直到某个时间
public static void unpark(Thread thread); // 恢复当前线程
public static Object getBlocker(Thread t);
```
叫**park**是因为**park**英文意思为停车。我们如果把**Thread**看成一辆车的话，**park**就是让车停下，**unpark**就是让车启动然后跑起来。

与Object类的wait/notify机制相比，park/unpark有两个优点：
>1. 以thread为操作对象更符合阻塞线程的**直观定义**
>2. **操作更精准**，可以准确地唤醒某一个线程（notify随机唤醒一个线程，notifyAll唤醒所有等待的线程），增加了灵活性。


LockSupport.park() 和 LockSupport.unpark(Thread thread) 调用的是 **Unsafe**(提供CAS操作) 中的 **native**代码。

[park/unpark](https://www.jianshu.com/p/e3afe8ab8364) 功能在Linux系统下，是用的**Posix**线程库**pthread**中的**mutex**(互斥量)，**condition**(条件变量)来实现的。**mutex**和**condition**保护了一个 **_counter** 的变量，当 **park** 时，这个变量被设置为0。当**unpark**时，这个变量被设置为1。

### 2.3、CAS 
[CAS](https://mp.weixin.qq.com/s/kvuPxn-vc8dke093XSE5IQ) 是 CPU指令级别实现了原子性的比较和交换(Conmpare And Swap)操作，注意CAS不是锁只是CPU提供的一个原子性操作指令。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122014571496.png)
CAS在语言层面不进行任何处理，直接将原则操作实现在**硬件级别实现**，只所以可以实现硬件级别的操作核心是因为CAS操作类中有个核心类**UnSafe**类： 关于CAS引发的ABA问题、性能开销问题、只能保证一个共享变量之间的原则性操作问题。以前[CAS](https://mp.weixin.qq.com/s/kvuPxn-vc8dke093XSE5IQ)中写过，再次不再重复讲解。

# 3、AQS重要方法
模版方法分为`独占式`跟`共享式`，子类根据需要不同调用不同的模版方法(讲解有点多，想看底层可直接下滑到第四章节)。
### 3.1 模板方法
##### 3.1.1 独占式获取
###### 3.1.1.1 accquire 
不可中断获取锁`accquire`是获取独占锁方法，`acquire`尝试获取资源，成功则直接返回，不成功则进入等待队列，这个过程不会被线程中断，被外部中断也不响应，获取资源后才再进行自我中断`selfInterrupt()`。
```java
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
}
```
1. acquire(arg)
tryAcquire(arg) 顾名思义，它就是尝试获取锁，需要我们自己实现具体细节，一般要求是：
> 如果该锁没有被另一个线程保持，则获取该锁并立即返回，将锁的保持计数设置为 1。如果当前线程已经保持该锁，则将保持计数加 1，并且该方法立即返回。
如果该锁被另一个线程保持，则出于线程调度的目的，禁用当前线程，并且在获得锁之前，该线程将一直处于休眠状态，此时锁保持计数被设置为 1。

2. addWaiter(Node.EXCLUSIVE)
>主要功能是 一旦尝试获取锁未成功，就要使用该方法将其加入同步队列**尾部**，由于可能有多个线程并发加入队尾产生竞争，因此采用**compareAndSetTail**锁方法来保证同步
>
3. acquireQueued(addWaiter(Node.EXCLUSIVE), arg)
> 一旦加入同步队列，就需要使用该方法，**自旋阻塞** 唤醒来不断的尝试获取锁，直到被中断或获取到锁。
###### 3.1.1.2 acquireInterruptibly
可中断获取锁`acquireInterruptibly`相比于`acquire`支持响应中断。
>如果当前线程未被中断，则尝试获取锁。 
如果锁空闲则获锁并立即返回，state = 1。如果当前线程已持此锁，state + 1，并且该方法立即返回。
如果锁被另一个线程保持，出于线程调度目的，禁用当前线程，线程休眠ing，除非锁由当前线程获得或者当前线程被中断了，中断后会抛出InterruptedException，并且清除当前线程的已中断状态。
此方法是一个显式中断点，所以要优先考虑响应中断。

```java
 if (shouldParkAfterFailedAcquire(p, node) && parkAndCheckInterrupt())
     throw new InterruptedException(); // acquireInterruptibly 选择
      interrupted = true; // acquire 的选择
```


###### 3.1.1.3 tryAcquireNanos
该方法可以被中断，增加了超时则失败的功能。可以说该方法的实现与上述两方法没有任何区别。时间功能上就是用的标准超时功能，如果剩余时间小于0那么`acquire`失败，如果该时间大于一次自旋锁时间(**spinForTimeoutThreshold** = 1000)，并且可以被阻塞，那么调用`LockSupport.parkNanos`方法阻塞线程。

doAcquireNanos内部：
```java
  nanosTimeout = deadline - System.nanoTime();
  if (nanosTimeout <= 0L)
      return false;
  if (shouldParkAfterFailedAcquire(p, node) && nanosTimeout > spinForTimeoutThreshold)
      LockSupport.parkNanos(this, nanosTimeout);
  if (Thread.interrupted())
      throw new InterruptedException();
```
该方法一般会有以下几种情况产生：
> 1. 在指定时间内，线程获取到锁，返回true。
> 2. 当前线程在超时时间内被中断，抛中断异常后，线程退出。
>3. 到截止时间后线程仍未获取到锁，此时线程获得锁失败，不再等待直接返回false。

##### 3.1.2 共享式获取
###### 3.1.2.1 acquireShared
```java
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }
```
该模版方法的工作：
1. 调用`tryAcquireShared(arg)` 尝试获得资源，返回值代表如下含义：
> 负数表示失败。
> 0 表示成功，但没有剩余可用资源。
> 正数表示成功，且有剩余资源。

doAcquireShared作用：
>创建节点然后加入到队列中去，这一块和独占模式下的**addWaiter**代码差不多，不同的是结点的模式是Node.SHARED，在独占模式下是Node.EXCLUSIVE。
###### 3.1.2.2 acquireSharedInterruptibly
无非就是可中断性的共享方法
```java
public final void acquireSharedInterruptibly(long arg)  throws InterruptedException {
    if (Thread.interrupted()) // 如果线程被中断，则抛出异常
        throw new InterruptedException();
    if (tryAcquireShared(arg) < 0)  
        // 如果tryAcquireShared()方法获取失败，则调用如下的方法
        doAcquireSharedInterruptibly(arg);
}
```
###### 3.1.2.3. tryAcquireSharedNanos
尝试以共享模式获取，如果被中断则中止，如果超过给定超时期则失败。实现此方法首先要检查中断状态，然后至少调用一次 `tryacquireshared(long)`，并在成功时返回。否则，在成功、线程中断或超过超时期之前，线程将加入队列，可能反复处于阻塞或未阻塞状态，并一直调用 `tryacquireshared(long)`。
```java
    public final boolean tryAcquireSharedNanos(long arg, long nanosTimeout)
            throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        return tryAcquireShared(arg) >= 0 ||
            doAcquireSharedNanos(arg, nanosTimeout);
    }
```
##### 3.1.3 独占式释放
独占锁的释放调用`unlock`方法，而该方法实际调用了AQS的`release`方法，这段代码逻辑比较简单，如果同步状态释放成功（tryRelease返回true）则会执行if块中的代码，当head指向的头结点不为null，并且该节点的状态值不为0的话才会执行`unparkSuccessor()`方法。
```java
    public final boolean release(long arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }
```
##### 3.1.4 共享式释放
`releaseShared`首先去尝试释放资源`tryReleaseShared(arg)`，如果释放成功了，就代表有资源空闲出来，那么就用`doReleaseShared()`去唤醒后续结点。
```java
public final boolean releaseShared(int arg) {
    if (tryReleaseShared(arg)) {
        doReleaseShared();
        return true;
    }
    return false;
}
```
比如CountDownLatch的`countDown()`具体实现：
```java
    public void countDown() {
        sync.releaseShared(1);
    }
```
### 3.2 子类需实现方法
子类**要实现**父类方法也分为`独占式`跟`共享式`。
###### 3.2.1 独占式获取  
tryAcquire
顾名思义，就是**尝试获取锁**，AQS在这里没有对其进行功能的实现，只有一个抛出异常的语句，我们需要自己对其进行实现，可以对其重写实现公平锁、不公平锁、可重入锁、不可重入锁
```java
protected boolean tryAcquire(int arg) {
        throw new UnsupportedOperationException();
}
```
###### 3.2.2 独占式释放 
` tryRelease` 尝试释放 独占锁，需要子类实现。
```java
   protected boolean tryRelease(long arg) {
        throw new UnsupportedOperationException();
    }
```
###### 3.2.3 共享式获取
` tryAcquireShared` 尝试进行共享锁的获得，需要子类实现。
```java
protected long tryAcquireShared(long arg) {
        throw new UnsupportedOperationException();
    }
```
###### 3.2.4 共享式释放
`tryReleaseShared`尝试进行共享锁的释放，需要子类实现。
```java
    protected boolean tryReleaseShared(long arg) {
        throw new UnsupportedOperationException();
    }
```
### 3.3  状态标志位
`state`因为用 [volatile](https://mp.weixin.qq.com/s/MY7moP4SNnJnKaSKy7WuPA)修饰 保证了我们操作的**可见性**，所以任何线程通过`getState()`获得状态都是可以得到最新值，但是`setState()`无法保证原子性，因此AQS给我们提供了`compareAndSetState`方法利用底层`UnSafe`的CAS功能来实现原子性。
```java
    private volatile long state;

    protected final long getState() {
        return state;
    }

    protected final void setState(long newState) {
        state = newState;
    }

   protected final boolean compareAndSetState(long expect, long update) {
        return unsafe.compareAndSwapLong(this, stateOffset, expect, update);
    }
```
### 3.4 查询是否独占模式
`isHeldExclusively` 该函数的功能是查询当前的工作模式是否是独占模式。需要子类实现。
```java
    protected boolean isHeldExclusively() {
        throw new UnsupportedOperationException();
    }
```
### 3.5 自定义实现锁
这里需要重点说明一点，**JUC中一般是用一个子类继承自Lock，然后在子类中定义一个内部类来实现AQS的继承跟使用**。
```java
public class SowhatLock implements Lock
{
	private Sync sync = new Sync();

	@Override
	public void lock()
	{
		sync.acquire(1);
	}

	@Override
	public boolean tryLock()
	{
		return false;
	}

	@Override
	public boolean tryLock(long time, TimeUnit unit) throws InterruptedException
	{
		return sync.tryAcquireNanos(1,unit.toNanos(time));
	}

	@Override
	public void unlock()
	{
		sync.release(1);
	}

	@Override
	public Condition newCondition()
	{
		return sync.newCondition();
	}

	@Override
	public void lockInterruptibly() throws InterruptedException
	{
	}

	private class Sync extends AbstractQueuedSynchronizer
	{
		@Override
		protected boolean tryAcquire(int arg)
		{
			assert arg == 1;
			if (compareAndSetState(0, 1))
			{
				setExclusiveOwnerThread(Thread.currentThread());
				return true;
			}
			return false;
		}

		@Override
		protected boolean tryRelease(int arg)
		{
			assert arg == 1;
			if (!isHeldExclusively())
			{
				throw new IllegalMonitorStateException();
			}
			setExclusiveOwnerThread(null);
			setState(0);
			return true;
		}

		@Override
		protected boolean isHeldExclusively()
		{
			return getExclusiveOwnerThread() == Thread.currentThread();
		}

		Condition newCondition() {
			return new ConditionObject();
		}
	}
}
```
自定义实现类：
```java
public class SoWhatTest
{
	public static int m = 0;
	public  static CountDownLatch latch  = new CountDownLatch(50);
	public static Lock lock = new SowhatLock();

	public static void main(String[] args) throws  Exception
	{
		Thread[] threads = new Thread[50];
		for (int i = 0; i < threads.length ; i++)
		{
			threads[i] = new Thread(()->{
				try{
					lock.lock();
					for (int j = 0; j <100 ; j++)
					{
						m++;
					}
				}finally
				{
					lock.unlock();
				}
				latch.countDown();
		});
		}
		for(Thread t : threads) t.start();
		latch.await();
		System.out.println(m);
	}
}
```
# 4、AQS底层
### 4.1 CLH
**CLH**(Craig、 Landin、 Hagersten locks三个人名字综合而命名):
> 1. 是一个**自旋锁**，能确保无饥饿性，提供先来先服务的公平性。
> 2. **CLH**锁也是一种**基于链表**的可扩展、高性能、公平的自旋锁，申请线程只在本地变量上自旋，它不断轮询前驱的状态，如果发现前驱释放了锁就结束自旋。

### 4.2 Node
**CLH**队列由**Node**对象组成，其中Node是AQS中的内部类。
```java
static final class Node {
 // 标识共享锁
 static final Node SHARED = new Node();
 // 标识独占锁
 static final Node EXCLUSIVE = null;
 // 前驱节点
 volatile Node prev;
 // 后继节点
 volatile Node next;
 // 获取锁失败的线程保存在Node节点中。
 volatile Thread thread;
 // 当我们调用了Condition后他也有一个等待队列
 Node nextWaiter;
 //在Node节点中一般通过waitStatus获得下面节点不同的状态，状态对应下方。
 volatile int waitStatus;
 static final int CANCELLED =  1;
 static final int SIGNAL    = -1;
 static final int CONDITION = -2;
 static final int PROPAGATE = -3;
```
**waitStatus** 有如下5中状态：
1. CANCELLED = 1 
> 表示当前结点已取消调度。当超时或被中断（响应中断的情况下），会触发变更为此状态，进入该状态后的结点将不会再变化。
 2. SIGNAL = -1 
>  表示后继结点在等待当前结点唤醒。后继结点入队时，会将前继结点的状态更新为 SIGNAL。
3. CONDITION = -2
> 表示结点等待在 Condition 上，当其他线程调用了 Condition 的 signal() 方法后，CONDITION状态的结点将从等待队列转移到同步队列中，等待获取同步锁。
4. PROPAGATE = -3
>  共享模式下，前继结点不仅会唤醒其后继结点，同时也可能会唤醒后继的后继结点。
5. INITIAL = 0
> 新结点入队时的默认状态。

### 4.3 AQS实现
##### 4.3.1 公平锁和非公平锁
银行售票窗口营业中：
> **公平排队**：每个客户来了自动在最后面排队，轮到自己办理业务的时候拿出身份证等证件取票。
> **非公平排队**：有个旅客火车马上开车了，他拿着自己的各种证件着急这想跟窗口工作人员说是否可以加急办理下，可以的话则直接办理，不可以的话则去队尾排队去。

在JUC中同样存在`公平锁`跟`非公平锁`，**一般非公平锁效率好一些**。因为非公平锁状态下打算抢锁的线程不用**排队挂起了**。
##### 4.3.2 AQS细节
**AQS**内部维护着一个**FIFO**的队列，即**CLH**队列，提供先来先服务的公平性。**AQS**的同步机制就是依靠**CLH**队列实现的。**CLH**队列是**FIFO**的双端双向链表队列(方便尾部节点插入)。线程通过**AQS**获取锁失败，就会将线程封装成一个**Node**节点，通过**CAS**原子操作插入队列尾。当有线程释放锁时，会尝试让队头的next节点占用锁，个人理解AQS具有如下几个特点：
> 1. **在AQS 同步队列中 -1 表示线程在睡眠状态**
> 2. **当前Node节点线程会把前一个Node.ws = -1。当前节点把前面节点ws设置为-1，你可以理解为：你自己能知道自己睡着了吗？ 只能是别人看到了发现你睡眠了**！
> 3. **持有锁的线程永远不在队列中**。
> 4. **在AQS队列中第二个才是最先排队的线程**。
> 5. **如果是交替型任务或者单线程任务，即使用了Lock也不会涉及到AQS 队列**。
> 6. **不到万不得已不要轻易park线程，很耗时的！所以排队的头线程会自旋的尝试几个获取锁**。

### 4.4 加锁跟解锁流程图
以最经典的 **ReentrantLock** 为例逐步分析下 **lock** 跟 **unlock** 底层流程图(要原图的话公众号回复：`lock`)。
```java
private Lock lock = new ReentrantLock();
public void test(){
    lock.lock();
    try{
        doSomeThing();
    }catch (Exception e){
      ...
    }finally {
        lock.unlock();
    }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122017031630.png)
##### 4.4.1 独占式加入同步队列
同步器AQS中包含两个节点类型的引用：一个指向头结点的引用(head)，一个指向尾节点的引用(tail)，如果加入的节点是OK的则会直接运行该节点，当若干个线程抢锁失败了那么就会**抢着加入**到同步队列的尾部，因为是抢着加入这个时候用**CAS**来设置尾部节点。入口代码：
```java
public final void acquire(int arg) {
    if (!tryAcquire(arg) &&
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```
1. tryAcquire
> 该方法是需要自我实现的，在上面的demo中可见一斑，就是返回是否获得了锁。
```java
 protected final boolean tryAcquire(int acquires) {
     final Thread current = Thread.currentThread();
     int c = getState();
     if (c == 0) {
         //  是否需要加入队列，不需要的话则尝试CAS获得锁，获得成功后 设置当前锁的拥有者
         if (!hasQueuedPredecessors() &&
             compareAndSetState(0, acquires)) {
             setExclusiveOwnerThread(current);
             return true;
         }
     }
     else if (current == getExclusiveOwnerThread()) {
         // 这就是可重入锁的实现  
         int nextc = c + acquires;
         if (nextc < 0)
             throw new Error("Maximum lock count exceeded");
         setState(nextc);
         return true;
     }
     return false;
 }
```
2. addWaiter(Node.EXCLUSIVE,arg)
```java
/**
 * 如果尝试获取同步状态失败的话,则构造同步节点（独占式的Node.EXCLUSIVE），通过addWaiter(Node node,int args)方法将该节点加入到同步队列的队尾。
 */
 private Node addWaiter(Node mode) {
     // 用当前线程构造一个Node对象，mode是一个表示Node类型的字段，或者说是这个节点是独占的还是共享的
     Node node = new Node(Thread.currentThread(), mode);
     // 将目前队列中尾部节点给pred
     Node pred = tail;
     // 队列不为空的时候
     if (pred != null) {
         node.prev = pred;
         // 先尝试通过AQS方式修改尾节点为最新的节点，如果修改失败，意味着有并发，
         if (compareAndSetTail(pred, node)) {
             pred.next = node;
             return node;
         }
     }
     //第一次尝试添加尾部失败说明有并发，此时进入自旋
     enq(node);
     return node;
 }
```
3. 自旋enq
`enq`方法将并发添加节点的请求通过CAS跟自旋将尾节点的添加变得`串行化`起来。说白了就是让节点放到正确的队尾位置。
```java
/**
* 这里进行了循环，如果此时存在了tail就执行同上一步骤的添加队尾操作，如果依然不存在，
* 就把当前线程作为head结点。插入节点后，调用acquireQueued()进行阻塞
*/
private Node enq(final Node node) {
   for (;;) {
       Node t = tail;
       if (t == null) { // Must initialize
           if (compareAndSetHead(new Node()))
               tail = head;
       } else {
           node.prev = t;
           if (compareAndSetTail(t, node)) {
               t.next = node;
               return t;
           }
       }
   }
}
```
4. acquireQueued
`acquireQueued`是当前Node节点线程在死循环中获取同步状态，而只有前驱节点是**头节点**才能尝试获取锁，原因是：
> 1. 头结点是成功获取同步状态（锁）的节点，而头节点的线程释放了同步状态以后，将会唤醒其后继节点，后继节点的线程被唤醒后要检查自己的前驱节点是否为头结点。
>  2. 维护同步队列的**FIFO**原则，节点进入同步队列之后，会尝试自旋几次。
```java
final boolean acquireQueued(final Node node, int arg) {
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 自旋检查当前节点的前驱节点是否为头结点，才能获取锁
        for (;;) {
            // 获取节点的前驱节点
            final Node p = node.predecessor();
            if (p == head && tryAcquire(arg)) {
            // 节点中的线程循环的检查，自己的前驱节点是否为头节点
            // 只有当前节点 前驱节点是头节点才会 再次调用我们实现的方法tryAcquire
            // 接下来无非就是将当前节点设置为头结点，移除之前的头节点
                setHead(node);
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 否则检查前一个节点的状态，看当前获取锁失败的线程是否要挂起
            if (shouldParkAfterFailedAcquire(p, node) &&
           //如果需要挂起，借助JUC包下面的LockSupport类的静态方法park挂起当前线程，直到被唤醒
                parkAndCheckInterrupt())
                interrupted = true; // 两个判断都是true说明 则置true
        }
    } finally {
        //如果等待过程中没有成功获取资源（如timeout，或者可中断的情况下被中断了），那么取消结点在队列中的等待。
        if (failed)
           //取消请求，将当前节点从队列中移除
            cancelAcquire(node);
    }
}
```
如果成功就返回，否则就执行`shouldParkAfterFailedAcquire`、`parkAndCheckInterrupt`来达到阻塞效果。

5. shouldParkAfterFailedAcquire
第二步的`addWaiter()`构造的新节点，`waitStatus`的默认值是**0**。此时，会进入最后一个if判断，CAS设置`pred.waitStatus SIGNAL`，最后返回`false`。由于返回`false`，第四步的`acquireQueued`会继续进行循环。假设`node`的前继节点`pred`仍然不是头结点或锁获取失败，则会再次进入`shouldParkAfterFailedAcquire()`。上一轮循环中已经将`pred.waitStatu = -1`了，则这次会进入第一个判断条件，直接返回**true**，表示应该阻塞调用`parkAndCheckInterrupt`。
     
那么什么时候会遇到`ws > 0`呢？当`pred`所维护的获取请求被取消时（也就是node的waitStatus = CANCELLED），这时就会**循环移除所有被取消的前继节点pred**，直到找到未被取消的pred。移除所有被取消的前继节点后，直接返回false。
```java
private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
       int ws = pred.waitStatus; // 获得前驱节点的状态
       if (ws == Node.SIGNAL) //此处是第二次设置
           return true;
       if (ws > 0) {
          do {
               node.prev = pred = pred.prev;
           } while (pred.waitStatus > 0);
           pred.next = node;
       } else {
          //  此处是第一次设置 unsafe级别调用设置
          compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
       }
       return false;
   }
```
6. parkAndCheckInterrupt
主要任务是暂停当前线程然后查看是否已经暂停了。
```java
private final boolean parkAndCheckInterrupt() {
    // 调用park()使线程进入挂起状态，什么时候调用了unpark再继续执行下面
    LockSupport.park(this); 
    // 如果被唤醒，查看自己是不是已经被中断了。
    return Thread.interrupted();
}
```
7. cancelAcquire
` acquireQueued`方法的finally会判断 `failed`值，正常运行时候自旋出来的时候会是`false`，如果中断或者`timeout`了 则会是`true`，执行`cancelAcquire`，其中核心代码是`node.waitStatus = Node.CANCELLED`。
8. selfInterrupt
```java
static void selfInterrupt() {
      Thread.currentThread().interrupt();
  }
```
##### 4.4.2 独占式释放队列头节点
`release()`会调用`tryRelease`方法尝试释放当前线程持有的锁，成功的话唤醒后继线程，并返回true，否则直接返回false。
```java
public final boolean release(long arg) {
    if (tryRelease(arg)) {
        Node h = head;
        if (h != null && h.waitStatus != 0)
            unparkSuccessor(h);
        return true;
    }
    return false;
}
```
1. tryRelease
这个是子类需要自我实现的，没啥说的根据业务需要实现。
2. unparkSuccessor
唤醒头结点的后继节点。
```java
private void unparkSuccessor(Node node) {
   int ws = node.waitStatus; // 获得头节点状态
    if (ws < 0) //如果头节点装小于0 则将其置为0
        compareAndSetWaitStatus(node, ws, 0);
    Node s = node.next; //这个是新的头节点
    if (s == null || s.waitStatus > 0) { 
    // 如果新头节点不满足要求
        s = null;
        for (Node t = tail; t != null && t != node; t = t.prev)
        //从队列尾部开始往前去找最前面的一个waitStatus小于0的节点
            if (t.waitStatus <= 0)
                s = t;
    }
    if (s != null)//唤醒后继节点对应的线程
        LockSupport.unpark(s.thread);
}
```
##### 4.4.3 AQS 中增加跟删除形象图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201220162603820.png)
# 5、CountDownLatch底层
### 5.1 共享锁 CountDownLatch底层
**CountDownLatch** 虽然相对简单，但也实现了共享锁模型。但是如何正确的吹逼 [CountDownLatch](https://blog.csdn.net/wb_zjp283121/article/details/88693124)  呢？如果在理解了上述流程的基础上，从**CountDownLatch**入手来看 **AQS** 中关于**共享锁**的代码还比较好看懂，在看的时候可以 **以看懂大致内容为主，学习其设计的思路**，不要陷入所有条件处理细节中，多线程环境中，对与错有时候不是那么容易看出来的。个人追源码绘制了如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222095557219.png)
### 5.2 计数信号量Semaphore

[Semaphore](https://mp.weixin.qq.com/s/4E6d2Zou2CzOmW3KlTO8zw) 这就是共享锁的一个实现类，在初始化的时候就规定了共享锁池的大小N，有一个线程获得了锁，可用数就减少1个。有一个线程释放锁可用数就增加1个。如果有>=2的线程同时释放锁，则此时有多个锁可用。这个时候就可以 **同时唤醒** 两个锁 **setHeadAndPropagate** (流程图懒的绘制了)。
```java
 public final void acquireShared(int arg) {
     if (tryAcquireShared(arg) < 0)
         doAcquireShared(arg);
 }
```
```java
private void doAcquireShared(int arg) {
    final Node node = addWaiter(Node.SHARED);
    boolean failed = true;
    try {
        boolean interrupted = false;
        for (;;) {
            //找先驱结点
            final Node p = node.predecessor();
            if (p == head) {
                 // 尝试获取资源
                int r = tryAcquireShared(arg);
                if (r >= 0) {
                    // 设置当前结点为头结点，然后去唤醒后续结点。注意传播性 唤醒！
                    setHeadAndPropagate(node, r);
                    p.next = null; // help GC  释放头结点，等待GC
                    if (interrupted)
                        selfInterrupt();
                    failed = false;//获取到资源
                    return;
                }
            }
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)//如果最后没有获取到资源，则cancel
            cancelAcquire(node);
    }
}
```
### 5.3 ReentrantReadWriteLock
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/20200319231958968.png)
 在 [ReentrantReadWriteLock](https://mp.weixin.qq.com/s/TlnAeajB8hAfImvuB6yGag) 类中也是只有一个32位的`int state`来表示读锁跟写锁，如何实现的？
 > 1. 后16位用来保存独享的**写锁**个数，第一次获得就是01，第二次重入就是10了，这样的方式来保存。
 > 2. 但是多个线程都可以获得读锁，并且每个线程可能读多次，如何保存？我们用前16位来保存有多少个线程获得了**读锁**。
 > 3. 每个读锁线程获得的重入读锁个数 由内部类`HoldCounter`与读锁配套使用。
# 6、Condition
[synchronized](https://mp.weixin.qq.com/s/e_fYFWK5Qnxjmz6Abi7uqw) 可用 **wait()** 和 **notify()**/**notifyAll()** 方法相结合可以实现等待/通知模式。**Lock** 也提供了 **Condition** 来提供类似的功能。

`Condition`是JDK5后引入的`Interface`，它用来替代传统的Object的`wait()/notify()`实现线程间的协作，相比使用Object的`wait()/notify()`，使用`Condition`的`await()/signal()`这种方式 **实现线程间协作更加安全和高效**。简单说，他的作用是使得某些线程一起等待某个条件(Condition)，只有当该条件具备(signal 或者 signalAll方法被调用)时，这些等待线程才会被唤醒，从而重新争夺锁。`wait()/notify()`这些都更倾向于底层的实现开发，而Condition接口更倾向于代码实现的等待通知效果。两者之间的区别与共通点如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200319205701634.png)


### 6.1 条件等待队列
条件等待队列，指的是 [Condition](https://blog.csdn.net/u012420654/article/details/56496631) 内部自己维护的一个队列，不同于 AQS 的 同步等待队列。它具有以下特点：
> 要加入[条件等待队列]的节点，不能在 [同步等待队列]。
从 [条件等待队列] 移除的节点，会进入[同步等待队列]。
一个锁对象只能有一个[同步等待队列]，但可以有多个[条件等待队列]。

这里以 **AbstractQueuedSynchronizer** 的内部类 **ConditionObject** 为例(Condition 的实现类)来分析下它的具体实现过程。首先来看该类内部定义的几个成员变量：
```java
/** First node of condition queue. */
private transient Node firstWaiter;
/** Last node of condition queue. */
private transient Node lastWaiter;
```
它采用了 **AQS** 的 **Node** 节点构造(前面说过**Node**类有**nextWaiter**属性)，并定义了两个成员变量：**firstWaiter**、**lastWaiter** 。说明在 ConditionObject 内部也维护着一个自己的单向等待队列。目前可知它的结构如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222173034142.png)
### 6.2 await、signal 
比如有线程 1、2竞争锁，下面来说下具体过程
线程1：
> 线程1 调用 reentrantLock.lock时，持有锁。
线程1 调用 await 方法，进入 条件**等待队列** ，同时释放锁。
线程1 获取到线程2 signal 信号，从  条件等待队列 进入 同步等待队列 。

线程2：
>线程2 调用 reentrantLock.lock时，由于锁被线程1 持有，进入 同步**等待队列** 。
由于线程1 释放锁，线程2 从 同步等待队列  移除，获取到锁。线程2 调用 signal 方法，导致线程 1 被唤醒。
线程2 调用 reentrantLock.unlock 。线程1 获取锁，继续下走。
##### 6.2.1 await
当我们看await、signal 的源码时候不要认为等待队列跟同步队列是完全分开的，其实个人感觉底层源码是有点 [HashMap](https://mp.weixin.qq.com/s/XGTNaOddY3elcumcPyO1KA) 中的红黑树跟双向链表的意思。
当我们调用await方法时候，说明当前任务队列的头节点拿着锁呢，此时要把该Thread从任务队列挪到等待队列再唤醒任务队列最前面排队的运行任务，如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222160628868.png)
1. thread 表示节点存放的线程。
2. waitStatus 表示节点等待状态。条件等待队列中的节点等待状态都是 CONDITION，否则会被清除。
3. nextWaiter 表示后指针。
##### 6.2.2 signal
当我们调用signal方法的时候，我们要将等待队列中的头节点移出来，让其去抢锁，如果是公平模式就要去排队了，流程如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222160816487.png)
上面只是形象流程图，如果从代码级别看的话大致流程如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201222220955354.png)
##### 6.2.3 signalAll
**signalAll**与**signal**方法的区别体现在**doSignalAll**方法上，前面我们已经知道doSignal方法只会对等待队列的**头节点**进行操作，**doSignalAll**方法只不过将等待队列中的**每一个节点**都移入到同步队列中，即`通知`当前调用**condition.await()**方法的每一个线程。：
```java
private void doSignalAll(Node first) {
    lastWaiter = firstWaiter = null;
    do {
        Node next = first.nextWaiter;
        first.nextWaiter = null;
        transferForSignal(first);
        first = next;
    } while (first != null); // 循环
}
```
### 6.3 End
一个 [Condition](https://blog.csdn.net/u012420654/article/details/56496631) 对象就有一个单项的**等待任务队**列。在一个多线程任务中我们可以new出多个等待任务队列。比如我们new出来两个等待队列。
```java
 private Lock lock = new ReentrantLock();
 private Condition FirstCond = lock.newCondition();
 private Condition SecondCond = lock.newCondition();
```
所以真正的AQS任务中一般是**一个任务队列N个等待队列的**，因此我们尽量调用**signal**而少用**signalAll**，因为在指定的实例化等待队列中只有一个可以拿到锁的。
而 **wait** 跟 **notify** 底层代码的等待队列只有一个，多个线程调用**wait**的时候我们是无法知道头节点是那个具体线程的。因此只能**notifyAll**。
# 7、参考
> 详解Condition的await和signal：https://www.jianshu.com/p/28387056eeb4
> Condition的await和signal流程：https://www.cnblogs.com/insaneXs/p/12219097.html
---

