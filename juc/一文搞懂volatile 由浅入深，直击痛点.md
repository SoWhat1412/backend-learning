今天闲来无事跟同事[小麦大叔](https://blog.csdn.net/u010632165)闲聊，
> SoWhat：麦叔听说你偷偷面阿里啦，面的咋样？
> 小麦大叔： 一面挺简单的，主要问了一些基本的数据结构跟算法，还问了下 [HashMap](https://blog.csdn.net/qq_31821675/article/details/105000051)的十大常见基本问题。我都答案上来了，还问了我JDK7环，幸亏你那个[HashMap环](https://blog.csdn.net/qq_31821675/article/details/105221231)绘制的牛逼，我答的不错就让我准备二面了。
> SoWhat:二面类？
> 小麦大叔：二面问了我一些JVM的问题，问我对于JVM内存模型的理解，还有GC的常见理解，最终还问了我下类加载机制，我看你之前水过这个 [JVM系列](https://blog.csdn.net/qq_31821675/category_9788753.html)，就依葫芦画瓢答上来了，让我准备三面。
> SoWhat：麦叔这波可以啊，三面问的啥啊？
> 小麦大叔：三面问了我一些CAS、Lock、AQS跟 [ConcurrentHashMap](https://sowhat.blog.csdn.net/article/details/105070823) 的底层实现什么的，还问了我下[线程池](https://blog.csdn.net/qq_31821675/article/details/105189304)的七大参数跟四大拒绝策略，以及使用注意事项。我看你水过 [并发编程系列](https://blog.csdn.net/qq_31821675/category_9769077.html)，也就答上来了。
> Sowhat：厉害啊这是要过的节奏阿！
> 小麦大叔：过个锤子，三面的这个总监最后竟然问了我下我对`volatile`的底层原理。你妹的你么水，我就答了一些基本的可见性跟弱原子性，然后我感觉面试官不太满意啊！
> Sowhat：额好吧，那我抓紧再水文写下个关于`volatile`的使用。

### 使用
`volatile`变量自身具有下列特性相信大家都知道：
> 1. 可见性。对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
> 2. 原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不具有原子性。

其中第二点可以理解为把对 [volatile](https://juejin.im/post/590f451c44d904007beaba1b#heading-3) 变量的单个读/写，看成是使用同一个锁对这些单个读/写操作做了同步,就跟下面的`SoWhat`跟`SynSoWhat`功能类似哦。
```java
class SoWhat{
    volatile int i = 0; // volatile修饰的变量
    public int getI(){
        return i;// 单个volatile变量的读
    }
    public  void setI(int j){
        this.i = j; // 单个volatile 变量的写
    }
    public void inc(){
        i++;//复合多个volatile 变量
    }
}
class SynSoWhat{
     int i = 0;
    public synchronized int getI(){
        return i;
    }
    public  synchronized void setI(int j){
        this.i = j;
    }
    public void inc(){ // 普通方法调用
        int tmp = getI(); // 调用已同步方法
        tmp = tmp + 1;//普通写方法
        setI(tmp);// 调用已同步方法
    }
}
```
### 写理解
volatile写的内存语义如下：
> 当写一个`volatile`变量时，JMM会把该线程对应的本地中的共享变量值`刷新`到主内存。
```java
public class VolaSemanteme {
	int a = 0;
	volatile boolean flag = false; // 这是重点哦
	public void init() {
		a = 1; 
		flag = true; 
		//.......
	}
	public void use() {
		if (flag) { 
			int i = a * a; 
		}
		//.......
	}
}
```
线程A调用`init`方法，线程B调用`use`方法。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403104611714.png#pic_center)
### 读理解
`volatile`读的内存语义如下：
> 当读一个volatile变量时，JMM会把该线程对应的本地内存置为`无效`。线程接下来将从主内存中读取共享变量。
```java
public class VolaSemanteme {
	int a = 0;
	volatile boolean flag = false; // 这是重点哦
	public void init() {
		a = 1; 
		flag = true; 
		//.......
	}
	public void use() {
		if (flag) { 
			int i = a * a; 
		}
		//.......
	}
}
```
流程图大致是这样的：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403104838822.png#pic_center)
### volatile 指令重排
`volatile` 变量的内存可见性是基于内存屏障（Memory Barrier）实现。关于内存屏障的具体讲解以前写过不再重复，[JMM装逼于无形](https://sowhat.blog.csdn.net/article/details/105257876)这里说过。总结来说就是JMM内部会有指令重排，并且会有`af-if-serial`跟`happen-before`的理念来保证指令重拍的正确性。内存屏障就是基于4个汇编级别的关键字来禁止指令重排的，其中volatile的重拍规则如下：
> 1. 第一个为读操作时，第二个任何操作不可重排序到第一个前面。
> 2. 第二个为写操作时，第一个任何操作不可重排序到第二个后面。
>3. 第一个为写操作时，第二个的读写操作也不运行重排序。
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403100754657.png#pic_center)
### volatile写底层实现
JMM对volatile的内存屏障插入策略
>在每个volatile写操作的前面插入一个StoreStore屏障。在每个volatile写操作的后面插入一个StoreLoad屏障。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403113444239.png#pic_center)

### volatile 读底层
JMM对volatile的内存屏障插入策略
> 在每个volatile读操作的后面插入一个LoadLoad屏障。在每个volatile读操作的后面插入一个LoadStore屏障。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403114745179.png#pic_center)

其中重点说下`volatile`读后面为什么跟了个`LoadLoad`。加入我有如下代码 AB两个线程执行，B线程的flag获取下面的读被提前了。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403122015710.png#pic_center)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403120139849.png#pic_center)
### volatile的实现原理
有volatile变量修饰的共享变量进行写操作的时候会使用`CPU`提供的`Lock`前缀指令。在CPU级别的功能如下：
> 1. 将当前处理器缓存行的数据写回到**系统内存**
> 2. 这个写回内存的操作会告知在其他CPU你们拿到的变量是无效的下一次使用时候要重新共享内存拿。

我们可以通过[jitwatch](https://github.com/AdoptOpenJDK/jitwatch)对简单的代码进行详细的[反汇编](https://blog.csdn.net/haihui_yang/article/details/103789384)看一下。
```java
package com.sowhat.demo;

public class VolaSemanteme {
    int unvloatileVal = 0;
    volatile boolean flag = false;

    public void init() {
        unvloatileVal = 1;
        flag = true; // 第九行哦
    }
    public void use() {

        if (flag) {
            int LocalA = unvloatileVal;
            if (LocalA == 0) {
                throw new RuntimeException("error");
            }
        }
    }
    public static void main(String[] args) {
        VolaSemanteme volaSemanteme = new VolaSemanteme();
        volaSemanteme.init();
        volaSemanteme.use();
    }
}
```
对普通变量的赋值操作：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403160105933.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxODIxNjc1,size_16,color_FFFFFF,t_70)
对`volatile`变量的赋值操作。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200403160203184.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzMxODIxNjc1,size_16,color_FFFFFF,t_70)
可以对比得出，volatile 修饰的变量确实会多一个 lock addl $0x0,(%rsp) 指令。
```java
0x0000000114ce95cb: lock addl $0x0,(%rsp)  ;*putfield flag
  ; - com.sowhat.demo.VolaSemanteme::init@7 (line 9)
```


