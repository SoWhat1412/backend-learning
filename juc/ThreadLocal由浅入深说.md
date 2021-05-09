> SoWhat：[麦叔](https://blog.csdn.net/u010632165)，听说你面阿里被挂这事还被APP推荐了，最近面别的公司没？
> 麦叔：老子反手就是一个回收🍑，我沉淀了一礼拜面头条去了。
> SoWhat：哎呦我去！麦叔你这头条都面上了，面了几轮，搞红黑树没？
> 麦叔：刚刚两轮，一面红黑树轻松搞定了！面我关于Java的JVM跟并发的时候我看你水的那个[JVM系列](https://blog.csdn.net/qq_31821675/category_9788753.html)还有[并发系列](https://blog.csdn.net/qq_31821675/category_9769077.html)都过了。最后还问了我点`ThreadLocal`的问题。
> SoWhat：擦，`ThreadLocal`有啥好问的就是个底层Map啊！并且日常我写数据库事务跟Spring的时候也没见用啊！问那么偏门干什么他们。
> 麦叔：擦。。。。你关于`ThreadLocal`知道的那么点啊？Spring的灵魂除了[IOC跟AOP](https://sowhat.blog.csdn.net/article/details/104491509)就是`ThreadLocal`了！
> SoWhat：真的么，麦叔你给我讲讲要不？
> 麦叔：好今天让你开开眼。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040517314897.gif#pic_center)
# 介绍
我们看下JDK文档的官方描述：`ThreadLocal`类用来提供线程内部等局部变量，这种变量在多线程环境下访问(get,set)时能保证各个线程的变量相对独立于其他线程内的变量，`ThreadLocal`实例通常来说都是`private static`类型，用于关联线程的上下文。
ThreadLocal作用：提供线程内部的局部变量，不同线程之间不会被相互干扰，该变量在线程生命周期内起作用，可以减少同一个线程内多个函数或者组件之间一些公共变量传递的复杂度。
> 1. 线程并发：在多线程并发环境下用
> 2. 传递数据：通过ThrealLocal在同一个线程，不同组件中传递公共变量。
> 3. 线程隔离：每个线程内变量是独立的，不会相互影响。

# 初探使用
使用的时候可以简单的理解为`ThreadLocal`维护这一个`HashMap`,其中key = 当前线程，value = 当前线程绑定的局部变量。
| 方法| 用途|
|--|--|
| ThreadLocal |  创建ThreadLocal对象|
| set(T value) | 设置当前线程绑定的局部变量 |
| T get() |获得当前线程绑定的局部变量  |
| remove() |移除当前线程绑定的局部变量  |

ThreadLocal使用 
1. 先是不用
```java
public class UserThreadLocal {
    private String str = "";
    public String getStr() {return str;}
    public void setStr(String j) {this.str = j;}
    public static void main(String[] args) {
        UserThreadLocal userThreadLocal = new UserThreadLocal();
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    userThreadLocal.setStr(Thread.currentThread().getName() + "的数据");
                    System.out.println(Thread.currentThread().getName() + " 编号 " + userThreadLocal.getStr());
                }
            });
            thread.setName("线程" + i);
            thread.start();
        }
    }
}
```
重复执行几次会出现如下结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200404225932306.png)
2. 用Synchronized
```java
  synchronized (UserThreadLocal.class) { 
  // 唯一区别就是用了同步方法块
   userThreadLocal.setStr(Thread.currentThread().getName() + "的数据");
 System.out.println(Thread.currentThread().getName() + " 编号 " + userThreadLocal.getStr());
  }
 }
```
多执行几次结果总能正确：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200404230541135.png)
3. 用了ThreadLocal
```java
public class UserThreadLocal {
    static ThreadLocal<String> str = new ThreadLocal<>();
    public String getStr() {return str.get();}
    public void setStr(String j) {str.set(j);}
    public static void main(String[] args) {
        UserThreadLocal userThreadLocal = new UserThreadLocal();
        for (int i = 0; i < 5; i++) {
            Thread thread = new Thread(new Runnable() {
                @Override
                public void run() {
                    userThreadLocal.setStr(Thread.currentThread().getName() + "的数据");
                    System.out.println(Thread.currentThread().getName() + " 编号 " + userThreadLocal.getStr());
                }
            });
            thread.setName("线程" + i);
            thread.start();
        }
    }
}
```
重复执行结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200404230149714.png)
结论： 多个线程同时对同一个共享变量里对一些属性赋值会产生不同步跟数据混乱，加锁通过现在同步使用可以实现有效性，通过`ThreadLocal`也可以实现。
|对比 |synchronized |ThreadLocal |
|-- |-- |-- |
| 原理|以时间换正确性，不同线程排队访问 |以空间换取准确性，为每一个线程都提供了一份变量副本，从而实现访问互不干扰 |
|侧重点 |多个线程之间访问资源对同步 |多线程中让每个线程之间的数据相互隔离|

# 再度使用
数据库转账系统，一定要确保`转出`跟`转入`具备事务性，JDBC中关于事务的API。
|Connection接口方法| 作用 |
|--|--|
| setAutoCommit(false) |禁止事务自动提交，默认是自动的  |
| commit() | 提交事务 |
|  rollback()|  回滚事务|
### 代码实现
分析转账业务，我们先将业务分4层。
1. dao层：连接数据库进行数据库的crud。
```java
public class AccountDao {
    public void out(String outUser, int money) throws SQLException {
        String sql = "update account set money = money - ?  where name = ?";
        Connection conn = JdbcUtils.getConnection();// 数据库连接池获取连接
        PreparedStatement preparedStatement = conn.prepareStatement(sql);
        preparedStatement.setInt(1, money);
        preparedStatement.setString(2, outUser);
        preparedStatement.executeUpdate();
        JdbcUtils.release(preparedStatement, conn);
    }

    public void in(String inUser, int money) throws SQLException {
        String sql = "update account set money = money + ?  where name = ?";
        Connection conn = JdbcUtils.getConnection();//数据库连接池获得连接
        PreparedStatement preparedStatement = conn.prepareStatement(sql);
        preparedStatement.setInt(1, money);
        preparedStatement.setString(2, inUser);
        preparedStatement.executeUpdate();
        JdbcUtils.release(preparedStatement, conn);
    }
}
```
2. service层：开启跟关闭事务，调用dao层。
```java
public class AccountService {
    public boolean transfer(String outUser, String inUser, int money) {
        AccountDao ad = new AccountDao(); // service 调用dao层
        Connection conn = null;
        try {
            // 开启事务
            conn = JdbcUtils.getConnection();// 数据库连接池获得连接
            conn.setAutoCommit(false);// 关闭自动提交

            ad.out(outUser, money);//转出
            int i = 1/0;// 此时故意用一个异常来检查数据库的事务性。
            ad.in(inUser, money);//转入
            // 上面这两个要有原子性
            JdbcUtils.commitAndClose(conn);//成功提交
        } catch (SQLException e) {
            e.printStackTrace();
            JdbcUtils.rollbackAndClose(conn);//失败回滚
            return false;
        }
        return true;
    }
}
```
3. utils层：数据库连接池的关闭跟获取。
```java
public class JdbcUtils {
    private static final ComboBoxPopupControl ds = new ComboPooledDataSource();
    public static Connection getConnection() throws SQLException {
        return ds.getConnection();// 从数据库连接池获得一个连接
    }
    public static void release(AutoCloseable... ios) {
        for (AutoCloseable io : ios) {
            if (io != null) {
                try {
                    io.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }
    public static void commitAndClose(Connection conn) {
        try {// 提交跟关闭
            if (conn != null) {
                conn.commit();
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public  static void rollbackAndClose(Connection conn){
        try{//回滚跟关闭
            if(conn!=null){
                conn.rollback();
                conn.close();
            }
        }catch (SQLException e){
            e.printStackTrace();
        }
    }
}
```
5. web层： 真正的调用入口。
```java

public class AccountWeb {
    public static void main(String[] args) {
        String outUser = "SoWhat";
        String inUser = "小麦";
        int money = 100;
        AccountService as = new AccountService();
        boolean result =  as.transfer(outUser,inUser,money);
        if(result == false){
            System.out.println("转账失败");
        }
        else{
            System.out.println("转账成功");
        }
    }
}
```
### 注意点
1. 为了保证所以操作在一个事务中，案例中连接必须是同一个,`service`层开启事务的`connection`需要跟`dao`层访问数据库的`connection`**保持一致**。
2. 线程并发的情况下，每个线程只能操作各自的`connection`。
上述注意点在代码中的体现为service层获取连接开启事务的要跟dao层的连接一致，并且在当前线程只能操作自己的连接。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405093352769.png)
### 寻常思路
>1. 传参：将service层connection对象直接传递到dao层，
>2. 加锁
常规代码更改如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405094328808.png)
弊端：
>1. 提高代码耦合度：service层connection对象传递到dao层了。
>2. 降低了程序到性能：因为加锁降低了系统性能。
>3. Spring采用`Threadlocal`的方式，来保证单个线程中的数据库操作使用的是同一个数据库连接，同时，采用这种方式可以使业务层使用事务时不需要感知并管理connection对象，通过传播级别，巧妙地管理多个事务配置之间的切换，挂起和恢复。

### ThreadLocal思路
用`ThreadLocal`来实现，核心思想就是`service`跟`dao`从数据库连接确保用到同一个。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020040509564529.png)
utils修改部分代码如下：
```java
    static ThreadLocal<Connection> tl = new ThreadLocal<>();

    private static final ComboBoxPopupControl ds = new ComboPooledDataSource();

    public static Connection getConnection() throws SQLException {
        Connection conn = tl.get();
        if (conn == null) {
            conn = ds.getConnection();
            tl.set(conn);
        }
        return conn;
    }

    public static void commitAndClose(Connection conn) {
        try {
            if (conn != null) {
                conn.commit();
                tl.remove(); //类似IO流操作 用完释放 避免内存泄漏 详情看下面分析
                conn.close();
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
```
ThreadLocal优势：
> 1.数据传递：保存每个线程绑定的数据，在需要的地方直接获取，避免参数传递带来的耦合性。
> 2. 线程隔离：各个线程之间的数据相互隔离又具有并发性，避免同步加锁带来的性能损失。


# 底层
### 误解
不看源码仅仅从我们使用跟别人告诉我们的角度去考虑我们会认为`ThreadLocal`设计的思路：一个共享的Map，其中每一个子线程=Key，该子线程对应存储的ThreadLocal值=Value。JDK早期确实是如下这样设计的，不过现在早已不是！
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405103541692.png#pic_center)

### JDK8中设计
在JDK8中`ThreadLocal`的设计是：每一个`Thread`维护一个`Map`，这个`Map`的`key`是`ThreadLocal`对象，value才是真正要存储的object，过程如下：
> 1. **每一个Thread线程内部都有一个Map(ThreadLocalMap)**，一个线程可以有多个TreadLocal来存放不同类型的对象的，但是他们都将放到你当前线程的ThreadLocalMap里，所以肯定要数组来存。
> 2. Map里存储ThreadLocal对象为key，线程的变量副本为value。
> 3.  Thread内部的Map是由ThreadLocal类维护的，由ThreadLocal负责向map获取跟设置线程变量值。
> 4. 不同线程每次获取副本值时，别的线程无法获得当前线程的副本值，形成副本隔离，互不干扰。
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405105330413.png#pic_center)
### 优势
JDK8设计比JDK早期设计的优势，我们可以看到早期跟现在主要的变化就是`Thread`跟`ThreadLocal`**调换**了位置。

老版：`ThreadLocal`维护着一个`ThreadLocalMap`，由`Thread`来当做这个map里的key。
新版：`Thread`维护这一个`ThreadLocalMap`，由当前的`ThreadLocal`作为key。

1.  每个Map存储的KV数据变小了，以前是线程个数多则`ThreadLocal`存储的KV数就变多。现在的K是用`ThreadLocal`实例化对象来当key的，多线程情况下`ThreadLocal`实例化个数一般都比线程数少！
2.  以前线程销毁后`ThreadLocal`这个Map还是存在的，现在当Thread销毁时候，`ThreadLocalMap`也会随之销毁，减少内存使用。

### ThreadLocal核心方法
ThreadLocal对外暴露的方法有4个：
| 方法| 用途|
|--|--|
| initialValue() |  返回当前线程局部变量初始化值|
| set(T value) | 设置当前线程绑定的局部变量 |
| T get() |获得当前线程绑定的局部变量  |
| remove() |移除当前线程绑定的局部变量  |

##### set方法：
```java
// 设置当前线程对应的ThreadLocal值
public void set(T value) {
    Thread t = Thread.currentThread(); // 获取当前线程对象
    ThreadLocalMap map = getMap(t);
    if (map != null) // 判断map是否存在
        map.set(this, value); 
        // 调用map.set 将当前value赋值给当前threadLocal。
    else
        createMap(t, value);
        // 如果当前对象没有ThreadLocalMap 对象。
        // 创建一个对象 赋值给当前线程
}

// 获取当前线程对象维护的ThreadLocalMap
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
// 给传入的线程 配置一个threadlocals
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```
执行流程：
> 1. 获得当前线程，根据当前线程获得map。
> 2. map不为空则将参数设置到map中，当前到Threadlocal作为key。
> 3. 如果map为空，给该线程创建map，设置初始值。

##### get方法
```java
public T get() {
    Thread t = Thread.currentThread();//获得当前线程对象
    ThreadLocalMap map = getMap(t);//线程对象对应的map
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);// 以当前threadlocal为key,尝试获得实体
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    // 如果当前线程对应map不存在
    // 如果map存在但是当前threadlocal没有关连的entry。
    return setInitialValue();
}

// 初始化
private T setInitialValue() {
    T value = initialValue();
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
    return value;
}
```
> 1. 先尝试获得当前线程，获得当前线程对应的map。
>2. 如果获得的map不为空，以当前threadlocal为key尝试获得entry。
>3. 如果entry不为空，返回值。
> 4. 但凡2跟3 出现无法获得则通过initialValue函数获得初始值，然后给当前线程创建新map。

##### remove
首先尝试获取当前线程，然后根据当前线程获得map，从map中尝试删除enrty。
```java
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }
```
##### initialValue
1. 如果没有调用set直接get，则会调用此方法，该方法只会被调用一次，
2. 返回一个缺省值null。
3. 如果不想返回null，可以Override 进行覆盖。
```java
   protected T initialValue() {
        return null;
    }
```
# ThreadLocalMap源码分析
在分析`ThreadLocal`重要方法时，可以知道`ThreadLocal`的操作都是围绕`ThreadLocalMap`展开的，其中2包含3，1包含2。
>1. public class ThreadLocal <T> 
>2.  static class ThreadLocalMap
>3.  static class Entry extends WeakReference<ThreadLocal<?>>
>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405121513996.png?)
###  ThreadLocalMap成员变量
跟[HashMap](https://sowhat.blog.csdn.net/article/details/105000051)一样的参数，此处不再重复。
```java
// 跟hashmap类似的一些参数
private static final int INITIAL_CAPACITY = 16;

private Entry[] table;

private int size = 0;

private int threshold; // Default to 0
```
### ThreadLocalMap主要函数：
刚刚说的`ThreadLocal`中的一些`get`、`set`、`remove`方法底层调用的都是下面这几个函数
```java
set(ThreadLocal,Object)
remove(ThreadLocal)
getEntry(ThreadLocal)
```
###  内部类Entry
```java
// Entry 继承子WeakReference,并且key 必须说ThreadLocal
// 如果key是null，意味着key不再被引用，这是好entry可以从table清除
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```
在`ThreadLocalMap`中，用`Entry`来保存KV结构，同时`Entry`中的key(`Threadlocal`)是弱引用，目的是将ThreadLocal对象生命周期跟线程周期解绑。
[弱引用](https://blog.csdn.net/qq_31821675/article/details/104741947):
>WeakReference :一些有用（程度比软引用更低）但是并非必需，用弱引用关联的对象，只能生存到下一次垃圾回收之前，GC发生时，不管内存够不够，都会被回收。

### 弱引用跟内存泄漏
可能有些人认为使用`ThreadLocal`的过程中发生了内存泄漏跟`Entry`中使用弱引用`key`有关，结论是不对的。
##### 如果Key是强引用
> 1. 如果在业务代码中使用完`ThreadLocal`则此时，Stack中的`ThreadLocalRef`就会被`回收`了。
> 2. 但是此时`ThreadLocalMap`中的Entry中的Key是强引用`ThreadLocal`的，会造成`ThreadLocal`实例`无法回收`。
> 3. 如果我们没有删除Entry并且CurrentThread依然运行的情况下，强引用链如下图红色，会导致Entry内存泄漏。
> 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405143711277.png?)

结论： 强引用无法避免内存泄漏。

##### 如果key是弱引用
> 1. 如果在业务代码中使用完来`ThreadLocal`则此时，Stack中的`ThreadLocalRef`就会被`回收`了。
> 2. 但是此时`ThreadLocalMap`中的Entry中的Key是弱引用`ThreadLocal`的，会造成`ThreadLocal`实`回收`，此时Entry中的key = null。
> 3. 但是当我们没有手动删除Entry以及CurrentThread依然运行的时候还是存在强引用链，因为`ThreadLocalRef`已经被回收了，那么此时的value就无法访问到了，导致value内存泄漏！
> 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405144313247.png?)

结论：弱引用也无法避免内存泄漏。

### 内存泄漏原因
上面分析后知道内存泄漏跟强/弱应用无关，内存泄漏的前提有两个。
> 1. `ThreadLocalRef`用完后`Entry`没有手动删除。
> 2. `ThreadLocalRef`用完后`CurrentThread`依然在运行ing。

 - 第一点表明当我们在使用完毕`ThreadLocal`后，调用其对应的`remove`方法删除对应的`Entry`就可以避免内存泄漏。
 - 第二点是由于`ThreadLocalMap`是`CurrentThread`的一个属性，被当前线程引用，生命周期跟`CurrentThread`一样，如果当前线程结束`ThreadLocalMap`被回收，自然里面的Entry也被回收了，单问题是如果此时的线程不一样会被回收啊！，如果是线程池呢，用完就放回池子里了。

结论：`ThreadLocal`内存泄漏根源是由于`ThreadLocalMap`生命周期跟`Thread`一样，如果用完`ThreadLocal`没有手动删除就回内存泄漏。

### 为什么用弱引用
前面分析后知道内存泄漏跟强弱引用无关，那么为什么还要用弱引用？我们知道避免内存泄漏的方式有两个。
> 1. `ThreadLocal`使用完毕后调用`remove`方法删除对应的Entry。
> 2. `ThreadLocal`使用完毕后，当前的`Thread`也随之结束。

第一种方法容易实现，第二站不好搞啊！尤其是如果线程是从线程池拿的用完后是要放回线程池的，不会被销毁。

事实上在`ThreadLocalMap`中的set/getEntry方法中，我们会对key = null (也就是`ThreadLocal`为null)进行判定，如果key = null,则系统认为value没用了也会设置为null。

这意味着当我们使用完毕`ThreadLocal`，`Thread`仍然运行的前提下即使我们忘记调用`remove`， 弱引用也会比强引用多一层保障，弱引用的`ThreadLocal`会被收回然后key就是`null`了，对应的value会在我们下一次调用`ThreadLocal`的`set/get/remove`任意一个方法的时候都会调用到底层`ThreadLocalMap`中的对应方法。无用的value会被清除从而避免内存泄漏。对应的具体函数为`expungeStaleEntry`。

### Hash冲突
#####  构造方法
我们看下`ThreadLocalMap`构造方法：
```java
ThreadLocalMap(ThreadLocal<?> firstKey, Object firstValue) {
    table = new Entry[INITIAL_CAPACITY];//新建table
    int i = firstKey.threadLocalHashCode & (INITIAL_CAPACITY - 1); //找到位置
    table[i] = new Entry(firstKey, firstValue);//放置新的entry
    size = 1;// 容量初始化
    setThreshold(INITIAL_CAPACITY);// 设置扩容阈值
}

threadLocalHashCode = nextHashCode();

private static int nextHashCode() {
    return nextHashCode.getAndAdd(HASH_INCREMENT);
}

private static AtomicInteger nextHashCode =
    new AtomicInteger();

private static final int HASH_INCREMENT = 0x61c88647;
// 避免哈希冲突尽量
```
其实构造方法跟位细节运算看[HashMap](https://sowhat.blog.csdn.net/article/details/105000051)，写过的不再重复。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200730161951186.png#pic_center)

##### set方法
流程大致如下：
> 1. 根据key获得对应的索引i,查找i位置上的Entry
> 2. 如果Entry已存在并key也相等则直接进行值的覆盖。
> 3.  如果Entry存在，但是key为空，调用`replaceStaleEntry`替换key为空的Entry
> 4. 如果遇到了`table[i]`为null的时候则需要在`table[i]`出创建一个新的Entry，并且插入，同时size+1。
> 5. 调用`cleanSomeSlots`清理key为null的Entry，再`rehash`。
```java


private void set(ThreadLocal<?> key, Object value) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);//计算索引位置

    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) { // 开放定值法解决哈希冲突
        ThreadLocal<?> k = e.get();

        if (k == key) {//直接覆盖
            e.value = value;
            return;
        }

        if (k == null) {// 如果key不是空value是空，垃圾清除内存泄漏防止。
            replaceStaleEntry(key, value, i);
            return;
        }
    }
    // 如果ThreadLocal对应的key不存在并且没找到旧元素，则在空元素位置创建个新Entry
    tab[i] = new Entry(key, value);
    int sz = ++size;
    if (!cleanSomeSlots(i, sz) && sz >= threshold)
        rehash();
}

// 环形数组 下一个索引
private static int nextIndex(int i, int len) {
    return ((i + 1 < len) ? i + 1 : 0);
}
```
PS:
> 1. 每个ThreadLocal只能保存一个变量**副本**，如果想要上线一个线程能够保存多个副本以上，就需要创建多个ThreadLocal。
> 2. ThreadLocal内部的ThreadLocalMap键为**弱**引用，会有内存泄漏的风险，用完记得擦屁股。
>3. 适用于无状态，副本变量独立后不影响业务逻辑的高并发场景。如果如果业务逻辑强依赖于副本变量，则不适合用ThreadLocal解决，需要另寻解决方案
### 如果想共享线程的ThreadLocal数据怎么办？
使用 [InheritableThreadLocal](https://www.jianshu.com/p/94ba4a918ff5) 可以实现多个线程访问`ThreadLocal`的值，我们在主线程中创建一个`InheritableThreadLocal`的实例，然后在子线程中得到这个`InheritableThreadLocal`实例设置的值。
```java
private void test() {    
final ThreadLocal threadLocal = new InheritableThreadLocal();       
threadLocal.set("帅得一匹");    
Thread t = new Thread() {        
    @Override        
    public void run() {            
      super.run();            
      Log.i( "张三帅么 =" + threadLocal.get());        
    }    
  };          
  t.start(); 
} 
```
###  为什么一般用`ThreadLocal`都要用`Static`?
阿里规范有云：
> `ThreadLocal `无法解决共享对象的更新问题，`ThreadLocal `对象建议使用 `static`修饰。这个变量是针对一个线程内所有操作**共享**的，所以设置为静态变量，所有此类实例共享此静态变量 ，也就是说在类第一次被使用时装载，只分配一块存储空间，所有此类的对象(只要是这个线程内定义的)都可以操控这个变量。

JDK官方规范有云：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405170500461.png)
# 参考
[黑马老师讲解](https://www.bilibili.com/video/BV1N741127FH?from=search&seid=16136985564278830426)

