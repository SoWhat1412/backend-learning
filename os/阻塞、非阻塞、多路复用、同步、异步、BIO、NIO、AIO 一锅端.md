因为申请较晚，本公众号没留言，想交流的欢迎添加私人微信，一起相互吹捧，共同进步。
 
![](https://img-blog.csdnimg.cn/img_convert/831ac5bfa9d05c8df6e2735e9a404580.png)
 
承接上文的[操作系统](https://mp.weixin.qq.com/s?__biz=MzI4NjI1OTI4Nw%3D%3D&idx=1&mid=2247491874&scene=21&sn=02d78d2b1a9252756a589d63470b10e1#wechat_redirect)，关于IO会涉及到阻塞、非阻塞、多路复用、同步、异步、BIO、NIO、AIO等几个知识点。知识点虽然不难但平常经常容易搞混，特此Mark下，与君共勉。
 
# 1 阻塞跟非阻塞
### 1.1 阻塞
 
![**阻塞IO**](https://img-blog.csdnimg.cn/img_convert/cf3baeb6d1cedd62873aff404b0e637c.png)
 
阻塞IO情况下，当用户调用`read`后，用户线程会被阻塞，等内核数据准备好并且数据从内核缓冲区拷贝到用户态缓存区后`read`才会返回。可以看到是阻塞的两个部分。
 
1.  CPU把数据从磁盘读到内核缓冲区。
    
2.  CPU把数据从内核缓冲区拷贝到用户缓冲区。
    
 
### 1.2 非阻塞
 
![**非阻塞IO**](https://img-blog.csdnimg.cn/img_convert/112a78d5db64eadeb4eddf51259af63f.png)
 

  
非阻塞IO发出read请求后发现数据没准备好，会继续往下执行，此时应用程序会不断轮询polling内核询问数据是否准备好，当数据没有准备好时，内核立即返回EWOULDBLOCK错误。直到数据被拷贝到应用程序缓冲区，read请求才获取到结果。并且你要注意！这里最后一次 read 调用获取数据的过程，是一个同步的过程，是需要等待的过程。这里的同步指的是**内核态的数据拷贝到用户程序的缓存区这个过程**。
 
### 1.3 IO多路复用
 
![**IO多路复用**](https://img-blog.csdnimg.cn/img_convert/9d2d910c8ecacb99900a68495ca4c2f2.png)
 

非阻塞情况下无可用数据时，应用程序每次轮询内核看数据是否准备好了也耗费CPU，能否不让它轮询，当内核缓冲区数据准备好了，以事件通知当机制告知应用进程数据准备好了呢？应用进程在没有收到数据准备好的事件通知信号时可以忙写其他的工作。此时**IO多路复用**就派上用场了。
 
IO多路复用中文比较让人头大，IO多路复用的原文叫 I/O multiplexing，这里的 multiplexing 指的其实是在单个线程通过记录跟踪每一个Sock(I/O流)的状态来同时管理多个I/O流. 发明它的目的是尽量多的提高服务器的吞吐能力。实现一个线程监控多个IO请求，哪个IO有请求就把数据从内核拷贝到进程缓冲区，拷贝期间是阻塞的！现在已经可以通过采用mmap地址映射的方法，达到内存共享效果，避免真复制，提高效率。
 
![**IO多路复用**](https://img-blog.csdnimg.cn/img_convert/b93a42f7ae22c52d9e603a1ae1c0cfe8.png)
 

  
像**select、poll、epoll **都是I/O多路复用的具体的实现。
 
1.3.1 select
 
select是第一版IO复用，提出后暴漏了很多问题。
 
1.  select 会修改传入的参数数组，这个对于一个需要调用很多次的函数，是非常不友好的。
    
2.  select 如果任何一个sock(I/O stream)出现了数据，select 仅仅会返回，但不会告诉是那个sock上有数据，只能自己遍历查找。
    
3.  select 只能监视1024个链接。
    
4.  select 不是线程安全的，如果你把一个sock加入到select, 然后突然另外一个线程发现这个sock不用，要收回，这个select 不支持的。
    
 
1.3.2 **poll**
 
poll 修复了 select 的很多问题。
 
1.  poll 去掉了1024个链接的限制。
    
2.  poll 从设计上来说不再修改传入数组。
    
 
但是poll仍然不是线程安全的， 这就意味着不管服务器有多强悍，你也只能在一个线程里面处理一组 I/O 流。你当然可以拿多进程来配合了，不过然后你就有了多进程的各种问题。
 
1.3.3 epoll
 
epoll 可以说是 I/O  多路复用最新的一个实现，epoll 修复了poll 和select绝大部分问题， 比如：
 
1.  epoll 现在是线程安全的。
    
2.  epoll 现在不仅告诉你sock组里面数据，还会告诉你具体哪个sock有数据，你不用自己去找了。
    
3.  epoll 内核态管理了各种IO文件描述符， 以前用户态发送所有文件描述符到内核态，然后内核态负责筛选返回可用数组，现在epoll模式下所有文件描述符在内核态有存，查询时不用传文件描述符进去了。
    
 
1.3.4 三者对比
 
![对比图](https://img-blog.csdnimg.cn/img_convert/3e7c62be7ed33e01390d3297086d96bc.png)
 

横轴 Dead connections 是链接数的意思，叫这个名字只是它的测试工具叫deadcon。纵轴是每秒处理请求的数量，可看到epoll每秒处理请求的数量基本不会随着链接变多而下降的。poll 和/dev/poll 就很惨了。但 epoll 有个致命的缺点是只有**linux**支持。
 
比如平常Nginx为何可以支持4W的QPS是因为它会使用目标平台上面最高效的I/O多路复用模型。
 
 多路复用产生的效果，完全可以由用户态去遍历文件描述符并调用其非阻塞的 read 函数实现。而多路复用快的**原因**在于，操作系统提供了这样的系统调用，使得原来的 while 循环里多次系统调用，变成了一次系统调用 + 内核层遍历这些文件描述符。
### 1.4 异步IO
 
![**异步IO**](https://img-blog.csdnimg.cn/img_convert/454d953ddb562886af44e4f6d709a06e.png)
 

然后你会发现上面的提到过的操作都不是真正的异步，因为两个阶段总要等待会儿！而真正的异步 I/O 是内核数据准备好和数据从内核态拷贝到用户态这两个过程都不用等待。
 
很庆幸，Linux给我们准备了**aio_read**跟**aio_write**函数实现真实的异步，当用户发起aio_read请求后就会自动返回。内核会自动将数据从内核缓冲区拷贝到用户进程空间，应用进程啥都不用管。
 
# 2 同步跟异步
 
### 2.1 同步
 
同步跟异步的区别在于`数据从内核空间拷贝到用户空间是否由用户线程完成`，这里又分为同步阻塞跟同步非阻塞两种。
 
1.  同步阻塞：此时一个线程维护一个连接，该线程完成数据到读写跟处理到全部过程，数据读写时时线程是被阻塞的。
    
2.  同步非阻塞：非阻塞的意思是用户线程发出读请求后，读请求不会阻塞当前用户线程，不过用户线程还是要不断的去主动判断数据是否准备OK了。此时还是会阻塞等待内核复制数据到用户进程。他与同步BIO区别是使用一个连接全程等待
    
 
我们以同步非阻塞为例，如下可看到，在将数据从内核拷贝到用户空间这一过程，是由用户线程阻塞完成的。
 
![同步非阻塞](https://img-blog.csdnimg.cn/img_convert/4c6485a783708a11928f5139b0a09222.png)
 
### 2.2 异步
 
对于异步来说，用户进行读或者写后，将立刻返回，由内核去完成数据读取以及拷贝工作，完成后通知用户，并执行回调函数（用户提供的callback），此时数据已从内核拷贝到用户空间，用户线程只需要对数据进行处理即可，不需要关注读写，用户不需要等待内核对数据的复制操作，用户在得到通知时数据已经被复制到用户空间。我们以如下的真实异步非阻塞为例。
 
![异步IO](https://img-blog.csdnimg.cn/img_convert/5e8bf8f3cd0c468e9cb1b99edb801826.png)
 

  
可发现，用户在调用之后会立即返回，由内核完成数据的拷贝工作，并通知用户线程，进行回调。
 
### 2.3 同步跟异步对比
 
同步关注的消息通信机制synchronous communication，在发出一个调用时，在没有得到结果之前，该调用就不返回。但是一旦调用返回，就得到返回值了。换句话说，就是由调用者主动等待这个调用的结果。
 
异步关注消息通信机制asynchronous communication，调用在发出之后，这个调用就直接返回了，所以没有返回结果。换句话说，当一个异步过程调用发出后，调用者不会立刻得到结果。而是在调用发出后，被调用者通过状态、通知来通知调用者，或通过回调函数处理这个调用。
 
# 3 Java IO
 
在Java中，我们使用socket进行网络通信，IO主要有三种模式，主要看**内核支持**哪些。
 
1.  `BIO`：同步阻塞IO。
    
2.  `NIO`：同步非阻塞IO。
    
3.  `AIO`：异步非阻塞IO。
    
 
### 3.1 BIO
 
**同步阻塞IO**，每个客户端的Socket连接请求，服务端都会对应有个处理线程与之对应，对于没有分配到处理线程的连接就会被阻塞或者拒绝。相当于是`一个连接一个线程`。
 
![BIO ](https://img-blog.csdnimg.cn/img_convert/40b00a0f3ab627e864f4693548cdecf6.png)
 

  
**BIO特点**：
 
1.  使用一个独立的线程维护一个socket连接，随着连接数量的增多，对虚拟机造成一定压力。
    
2.  使用流来读取数据，流是阻塞的，当没有可读／可写数据时，线程等待，会造成资源的浪费。
    
 
3.1.1 BIO 样例
 
常量：
 
```java
public class Constant {
    public static final String HOST = "127.0.0.1";
    public static final int PORT = 8080;
}
```
 
主类：
 
```java
public class ClientMain {
    public static void main(String\[\] args) {
        //开启服务
        System.out.println("开启服务,监听端口:" + Constant.PORT);
        new Thread(new ServerThread()).start();
        //建立一个socket客户端,发起请求
        System.out.println("客户端,请求连接,并发送数据");
        try {
            Socket socket = new Socket(Constant.HOST,Constant.PORT);
            //开启新的线程处理socket连接
            new Thread(new ClientProcessThread(socket)).start();
        } catch (IOException e) {
            e.printStackTrace();
        }
 
    }
}
```
 
服务端监听线程：
 
```java
// 开启服务监听线程,当收到连接请求后,开启新的线程进行处理
public class ServerThread implements Runnable{
    @Override
    public void run() {
        try {
            ServerSocket serverSocket = new ServerSocket(Constant.PORT);
            while (true){
                Socket socket = serverSocket.accept();
                new Thread(new ServerProcessThread(socket)).start();
                //开启新的线程进行连接请求的处理
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
 
服务端处理线程：
 
```java
import java.io.*;
import java.net.Socket;
/**
 * 服务端收到连接请求后,处理请求的线程,阻塞式IO
 */
public class ServerProcessThread implements Runnable {
    private Socket socket;
    public ServerProcessThread(Socket socket){
        this.socket = socket;
    }
    @Override
    public void run() {
        //获取客户端的数据,并写回
        //等待响应
        try {
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            String line = "";
            String requestStr = "";
            System.out.println("来自客户端的数据:"); // 读取客户端数据
            while((line = bufferedReader.readLine()) != null){
                requestStr += line;
                System.out.println(line);
            }
            //  从服务端发给客户端数据
            Writer writer = new OutputStreamWriter(socket.getOutputStream());
            writer.write("data from server " + requestStr + "\\r\\n");
            writer.flush();
            writer.close();
            bufferedReader.close();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
 
客户端：
 
```java
/**
 * 维护客户端socket连接的线程,阻塞式IO
 */
public class ClientProcessThread implements Runnable {
    private Socket socket;
    public ClientProcessThread(Socket socket){
        this.socket = socket;
    }
    @Override
    public void run() {
        //写数据,等待响应,输出响应
        String requestStr = "data from client \\r\\n";
        try {
            Writer writer = new OutputStreamWriter(socket.getOutputStream());
            writer.write(requestStr);
            writer.flush();
            socket.shutdownOutput();
            //等待响应
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            String line;
            System.out.println("来自服务端的响应:");
            while((line = bufferedReader.readLine()) != null){
                System.out.println(line);
            }
            writer.close();
            bufferedReader.close();
            socket.close();
 
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
 
输出结果：![](https://img-blog.csdnimg.cn/img_convert/91c469ba3712979ae20850add24f92c2.png)
 
### 3.2 NIO
 
**同步非阻塞IO之NIO**：服务器端保存一个Socket连接列表，然后对这个列表进行轮询，如果发现某个Socket端口上有数据可读时说明读就绪，则调用该socket连接的相应读操作。如果发现某个 Socket端口上有数据可写时说明写就绪，则调用该socket连接的相应写操作。如果某个端口的Socket连接已经中断，则调用相应的析构方法关闭该端口。这样能充分利用服务器资源，效率得到了很大提高，在进行IO操作请求时候再用个线程去处理，是`一个请求一个线程`。Java中使用Selector、Channel、Buffer来实现上述效果。
 
![NIO](https://img-blog.csdnimg.cn/img_convert/7e53a36f694ff2b7e5de1152d7c5046e.png)
 

 
  
每个线程中包含一个`Selector`对象，它相当于一个通道管理器，可以实现在一个线程中处理多个通道的目的，减少线程的创建数量。远程连接对应一个channel，数据的读写通过buffer均在同一个`channel`中完成，并且数据的读写是非阻塞的。通道创建后需要注册在`selector`中，同时需要为该通道注册感兴趣事件（客户端连接服务端事件、服务端接收客户端连接事件、读事件、写事件），`selector`线程需要采用`轮训`的方式调用`selector`的`select`函数，直到所有注册通道中有兴趣的事件发生，则返回，否则一直阻塞。而后循环处理所有就绪的感兴趣事件。以上步骤解决BIO的两个瓶颈：
 
1.  不必对每个连接分别创建线程。
    
2.  数据读写非阻塞。
    
 
下面对以下三个概念做一个简单介绍，Java NIO由以下三个核心部分组成：
 
1.  `**selector**`：Selector 允许单线程处理多个Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。要使用Selector，得向Selector注册Channel，然后调用他的select方法，这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，事件的例子入有新连接接进来，数据接收等。
    
2.  **`Channel`**：基本上所有的IO在NIO中都从一个Channel开始。Channel有点像流，数据可以从channel**读**到buffer，也可以从buffer**写**到channel。
    
3.  **`Buffer`**：缓冲区本质上是一个可以读写数据的内存块，可以理解成是一个容器对象(含数组)，该对象提供了一组方法，可以更轻松的使用内存块，缓冲区对象内置了一些机制，能够跟踪和记录缓冲区的状态变换情况，Channel提供从文件，网络读取数据的渠道，但是读取或者写入的数据都必须经由Buffer。
    
 
channel和buffer有好几种类型。下面是Java NIO中的一些主要channel的实现：
 
```
FileChannel
DatagramChannel
SocketChannel
ServerSocketChannel
```
 
正如你所看到的，这些通道涵盖了UDP和TCP网络IO，以及文件IO。以下是Java NIO里关键的buffer实现：
 
```
ByteBuffer
CharBuffer
FloatBuffer
IntBuffer
LongBuffer
ShortBuffer
```
 
在微服务阶段，一个请求可能涉及到多个不同服务之间的跨服务器调用，如果你想实现高性能的PRC框架来进行数据传输，那就可以基于Java NIO做个支持长连接、自定义协议、高并发的框架，比如Netty。Netty本身就是一个基于NIO的网络框架， 封装了Java NIO那些复杂的底层细节，给你提供简单好用的抽象概念来编程。比如[Dubbo](https://mp.weixin.qq.com/s?__biz=MzI4NjI1OTI4Nw%3D%3D&idx=1&mid=2247489782&scene=21&sn=efbe773e62b12ad291d7801e3affc014#wechat_redirect)底层就是用的Netty。
 
![Netty通讯模式](https://img-blog.csdnimg.cn/img_convert/484bec66199d4cfc50ff9b22487639ff.png)
 

### 3.3 AIO
 
AIO是异步非阻塞IO，相比NIO更进一步，进程读取数据时只负责发送跟接收指令，数据的准备工作完全由操作系统来处理。
 
# 4 参考

 
1.  IO说：https://blog.csdn.net/u013177446/article/details/65936341
    
2.  爆赞TCP讲解：https://b23.tv/tMxwQV
    
3.  通俗说IO：https://www.cnblogs.com/LBSer/p/4622749.html
    
4.  小仙IO：https://t.1yb.co/iEAW
5.  
    
