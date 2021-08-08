开发人员开发完一个电商项目，该 [Jar](https://mp.weixin.qq.com/mp/appmsgalbum?action=getalbum&album_id=1663052626025840644&__biz=MzI4NjI1OTI4Nw==&uin=&key=&devicetype=Windows%207%20x64&version=63010029&lang=zh_CN&ascene=7&fontgear=2) 项目包含 [Redis](https://mp.weixin.qq.com/s/UfJE6V45MoAQK2RpmNbvhA)、[MySQL](https://mp.weixin.qq.com/s/O_NHjv_YVUi4lSqXnhx5Mg)、ES、Haddop等若干组件。开发人员自测无误后提交给测试进行预生产测试了。
> **测试**：你的这个服务，我在进行单元测试跟数据核对的时候总是出现不知名的bug！你要不要来看下啊？
> **开发**：你咋测试的？是按照操作文档一步步来的么？
> **测试**：绝对是按照文档来的啊！
> **开发**：你重启了吗？清缓存了吗？代码是最新版吗？你用的是Chrome浏览器? 你是不是动啥东西了？ 
> **测试**：这 这 这  我啥也没干啊！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210110114728686.gif)
至此，开发跟测试之间的爱恨情仇正式开始！
# 1 Docker 简介
### 1.1 Docker 由来
Docker 是基于 **Go** 语言开发的一个容器引擎，Docker是应用程序与系统之间的**隔离层**。通常应用程序对安装的系统环境会有各种严格要求，当服务器很多时部署时系统环境的配置工作是非常繁琐的。Docker让应用程序不必再关心主机环境，各个应用安装在Docker镜像里，Docker引擎负责运行包裹了应用程序的docker镜像。

Docker的理念是让开发人员可以简单地把应用程序及依赖装载到容器中，然后轻松地部署到任何地方，Docker具有如下特性。
>1. Docker容器是轻量级的虚拟技术，占用更少系统资源。
>2. 使用 Docker容器，不同团队（如开发、测试，运维）之间更容易合作。
>3. 可以在任何地方部署 Docker 容器，比如在任何物理和虚拟机上，甚至在云上。
>4. 由于Docker容器非常轻量级，因此可扩展性很强。

### 1.2 Docker 基本组成
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111122346545.jpg)
镜像（image）：
>Docker 镜像就好比是一个目标，可以通过这个目标来创建容器服务，可以简单的理解为编程语言中的类。

容器（container）:
>Docker 利用容器技术，独立运行一个或者一组应用，容器是通过镜像来创建的，在容器中可执行启动、停止、删除等基本命令，最终服务运行或者项目运行就是在容器中的，可理解为是类的实例。

仓库（repository）:
>仓库就是存放镜像的地方！仓库分为公有仓库和私有仓库，类似Git。一般我们用的时候都是用国内docker镜像来加速。
### 1.3 VM 跟 Docker
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111094931409.png)
**虚拟机**：
>传统的虚拟机需要模拟整台机器包括硬件，每台虚拟机都需要有自己的操作系统，虚拟机一旦被开启，预分配给他的资源将全部被占用。每一个虚拟机包括应用，必要的二进制和库，以及一个完整的用户操作系统。

**Docker**：
>容器技术是和我们的宿主机共享硬件资源及操作系统可以实现资源的动态分配。容器包含应用和其所有的依赖包，但是与其他容器**共享内核**。容器在宿主机操作系统中，在用户空间以分离的进程运行。

|比对项	|Container（容器）|	VM（虚拟机）|
|-- |--|--|
|启动速度|	秒级|	分钟级|
|运行性能	|接近原生|	有所损失|
|磁盘占用|	MB|	GB|
|数量|	成百上千|	一般几十台|
|隔离性|	进程级别|	系统级别|
|操作系统|	只支持Linux|	几乎所有|
|封装程度|	只打包项目代码和依赖关系，共享宿主机内核|	完整的操作系统|
### 1.4 Docker 跟 DevOps 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111110552828.jpg)
**DevOps** 是一组过程、方法与系统的统称，用于促进开发（应用程序/软件工程）、技术运营和质量保障（QA）部门之间的沟通、协作与整合。
>DevOps 是两个传统角色 Dev(Development) 和 Ops(Operations) 的结合，Dev 负责开发，Ops 负责部署上线，但 Ops 对 Dev 开发的应用缺少足够的了解，而 Dev 来负责上线，很多服务软件不知如何部署运行，二者中间有一道明显的鸿沟，DevOps 就是为了弥补这道鸿沟。DevOps 要做的事，是偏 Ops 的；但是做这个事的人，是偏 Dev 的, 说白了就是要有一个了解 Dev 的人能把 Ops 的事干了。而Docker 是适合 DevOps 的。

### 1.5 Docker 跟 k8s
k8s的全称是kubernete，它是基于容器的集群管理平台，是管理应用的全生命周期的一个工具，从创建应用、应用的部署、应用提供服务、扩容缩容应用、应用更新、都非常的方便，而且可以做到故障自愈，例如一个服务器挂了，可以自动将这个服务器上的服务调度到另外一个主机上进行运行，无需进行人工干涉。k8s依托于Google自家的强大实践应用，目前市场占有率已经超过Docker自带的Swarm了。

一句话：如果你有很多 Docker 容器要启动、维护、监控，那就上k8s吧！
### 1.6 hello world
docker run hello-world 的大致流程图如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111124331727.png)
# 2 Docker 常见指令
**官方文档**：
> [https://docs.docker.com/engine/reference/commandline/build/](https://docs.docker.com/engine/reference/commandline/build/)
> 
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111142254600.png)

# 3 Docker 运行原理
Docker 只提供一个运行环境，他跟 VM 不一样，是不需要运行一个独立的 OS，容器中的系统内核跟宿主机的内核是**公用**的。**docker容器本质上是宿主机的进程**。对 Docker 项目来说，它最核心的原理实际上就是为待创建的用户进程做如下操作：
> 1. 启用 **Linux Namespace** 配置。
> 2. 设置指定的 **Cgroups** 参数。
> 3. 切换进程的根目录(**Change Root**)，优先使用 **pivot_root** 系统调用，如果系统不支持，才会使用 **chroot**。
### 3.1 namespace 进程隔离
Linux [Namespaces](https://blog.csdn.net/zhonglinzhang/article/details/64441263) 机制提供一种`进程资源隔离`方案。PID、IPC、Network 等系统资源不再是全局性的，而是属于某个特定的**Namespace**。每个**namespace**下的资源对于其他 **namespace** 下的资源都是透明，不可见的。系统中可以同时存在两个进程号为0、1、2的进程，由于属于不同的namespace，所以它们之间并不冲突。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210111153921901.png)

PS：**Linux** 内核提拱了6种 **namespace** 隔离的系统调用，如下图所示。![在这里插入图片描述](https://img-blog.csdnimg.cn/20210110141413247.png)
### 3.2 CGroup 分配资源
Docker 通过 **Cgroup** 来控制容器使用的资源配额，一旦超过这个配额就发出**OOM**。配额主要包括 CPU、内存、磁盘三大方面， 基本覆盖了常见的资源配额和使用量控制。  
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210110145814232.png)
[Cgroup](https://blog.csdn.net/ningyuxuan123/article/details/81981835) 是 Control Groups 的缩写，是Linux `内核`提供的一种可以限制、记录、隔离进程组所使用的物理资源(如 CPU、内存、磁盘 IO 等等)的机制，被 LXC(Linux container)、Docker 等很多项目用于实现进程资源控制。Cgroup 本身是提供将进程进行分组化管理的功能和接口的基础结构，I/O 或内存的分配控制等具体的资源管理是通过该功能来实现的，这些具体的资源 管理功能称为 Cgroup 子系统。
### 3.3  chroot 跟 pivot_root 文件系统
[chroot](https://segmentfault.com/a/1190000020700272)(change root file system)命令的功能是 **改变进程的根目录到指定的位置**。比如我们现在有一个` $HOME/test `目录，想要把它作为一个 `/bin/bash` 进程的根目录。
> 1. 首先，创建一个$HOME/test目录和几个lib文件夹 $HOME/test/{bin,lib64,lib}
> 2. 把bash命令拷贝到test目录对应的bin路径下 cp -v /bin/{bash,ls} $HOME/test/bin
> 3. 把bash命令需要的所有so文件，也拷贝到test目录对应的lib路径下
> 4. 执行chroot命令，告诉操作系统，我们将使用$HOME/test目录作为/bin/bash进程的根目录  chroot $HOME/test /bin/bash

被chroot的进程此时执行 `ls / ` 返回的都是`$HOME/test`目录下面的内容，Docker就是这样实现容器根目录的。为了能够让容器的这个根目录看起来更`真实`，一般在容器的根目录下挂载一个完整操作系统的文件系统，比如Ubuntu16.04的ISO。这样在容器启动之后，容器里执行`ls /`查看到的就是Ubuntu 16.04的所有目录和文件。

而挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的`容器镜像`。更专业的名字叫作：**rootfs**（根文件系统）。所以一个最常见的 **rootfs** 会包括如下所示的一些目录和文件：
```bash
$ ls /
bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var
```
**chroot** 只改变当前进程的 /，**pivot_root**改变当前 **mount namespace**的 / 。**pivot_root** 可以认为是 **chroot** 的改良版。
### 3.4 一致性
由于 **rootfs** 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着应用以及它运行所需要的所有依赖都被封装在了一起。有了容器镜像`打包操作系统`的能力，这个最基础的依赖环境也终于变成了**应用沙盒**的一部分。这就赋予了容器所谓的一致性：
> 无论在本地、云端，还是在一台任何地方的机器上，用户只需要解压打包好的容器镜像，那么这个应用运行所需要的完整的执行环境就被重现出来了。
### 3.5 UnionFS 联合文件系统
如何实现**rootfs**的高效可重复利用呢？Docker在镜像的设计中引入了层（**layer**）的概念。也就是说用户制作镜像的每一步操作都会生成一个层，也就是一个增量**rootfs**。介绍分层前我们先说个重要知识点，联合文件系统。

联合文件系统（UnionFS）是一种**分层、轻量级并且高性能的文件系统**，它支持对文件系统的修改作为一次提交来一层层的叠加，同时可以**将不同目录挂载到同一个虚拟文件系统**下。比如现在有水果fruits、蔬菜vegetables两个目录，其中水果中有苹果和蕃茄，蔬菜有胡萝卜和蕃茄：
```bash
$ tree
.
├── fruits
│  ├── apple
│  └── tomato
└── vegetables
  ├── carrots
  └── tomato
```
然后使用联合挂载的方式将这两个目录挂载到一个公共的目录 mnt 上：
```bash
$ mkdir mnt
$ sudo mount -t aufs -o dirs=./fruits:./vegetables none ./mnt
```
这时再查看目录 mnt 的内容，就能看到目录 fruits 和 vegetables 下的文件被合并到了一起：
```bash
$ tree ./mnt
./mnt
├── apple
├── carrots
└── tomato
```
可以看到在 mnt 目录下有三个文件，苹果apple、胡萝卜carrots和蕃茄tomato。水果和蔬菜的目录被union到了 mnt 目录下了。
```bash
 $ echo mnt > ./mnt/apple
 $ cat ./mnt/apple
 mnt
 $ cat ./fruits/apple
 mnt
```
可以看到./mnt/apple的内容改了，./fruits/apple的内容也改了。
```bash
 $ echo mnt_carrots > ./mnt/carrots
 $ cat ./vegetables/carrots
 old
 $ cat ./fruits/carrots
 mnt_carrots
```
./vegetables/carrots 并没有变化，反而是 ./fruits/carrots 的目录中出现了 carrots 文件，其内容是我们在 ./mnt/carrots 里的内容。

**结论**：
>在mount aufs命令时候，没有对 vegetables 跟 fruits 设置权限，默认命令行上第一个的目录是可读可写的，后面的全都是只读的。有重复的文件名，在mount命令行上，越往前的被操作的优先级越高。

### 3.6 layer 分层 
说完联合文件系统后我们再说下Docker中的分层，镜像可以通过分层来进行继承，基于基础镜像(没有父镜像)用户可以制作各种具体的应用镜像。不同 Docker 容器可以**共享**一些基础的文件系统层，同时再加上自己独有的改动层，大大提高了存储的效率。

Docker 中使用一种叫 **AUFS**（Anothe rUnionFS）的联合文件系统。 AUFS 支持为每一个成员目录设定不同的读写权限。
1.  rw 表示可写可读read-write。
2. ro 表示read-only，如果你不指权限，那么除了第一个外，ro是默认值，对于ro分支，其永远不会收到写操作，也不会收到查找whiteout的操作。
3.  rr 表示 real-read-only，与read-only不同的是，rr 标记的是天生就是只读的分支，这样，AUFS可以提高性能，比如不再设置inotify来检查文件变动通知。
 
当我们想修改**ro**层的文件时咋办？因为ro是不允许修改的啊！**Docker**中一般**ro**层还带个**wh**的能力。我们就需要对这个**ro**目录里的文件作**whiteout**。**AUFS**的**whiteout**的实现是通过**在上层的可写的目录下建立对应的whiteout隐藏文件来实现的**。比如我们有三个目录和文件如下所示：
```bash
$ tree
.
├── fruits
│   ├── apple
│   └── tomato
├── test #目录为空
└── vegetables
    ├── carrots
    └── tomato
```
 执行如下：
```bash
 $ mkdir mnt
 $ mount -t aufs -o dirs=./test=rw:./fruits=ro:./vegetables=ro none ./mnt
 $ ls ./mnt/
 apple  carrots  tomato
```
在权限为 rw 的 test 目录下建个 whiteout 的隐藏文件 `.wh.apple`，你就会发现 ./mnt/apple 这个文件就消失了，跟执行了 rm ./mnt/apple 是一样的结果：
```bash
 $ touch ./test/.wh.apple
 $ ls ./mnt
 carrots  tomato
```

对于[AUFS](https://segmentfault.com/a/1190000020700272)来说镜像的若干基础层放置在`/var/lib/docker/aufs/diff`目录下，然后通过查询` /sys/fs/aufs` 查看被联合挂载在一起的各个层的信息，多个基础层最终被联合挂载在`/var/lib/docker/aufs/mnt`里面，这里面存储的就是一个成品。

Docker 目前支持的联合文件系统包括 OverlayFS, AUFS, Btrfs, VFS, ZFS 和 Device Mapper。推荐使用 [overlay2](https://blog.csdn.net/qq_30164225/article/details/79516237) 存储驱动，overlay2 是目前 Docker 默认的存储驱动，以前则是 AUFS。
##### 3.6.1 只读层
我们以Ubuntu为例，当执行`docker image inspect ubuntu:latest` 会发现容器的**rootfs**最下面的四层，对应的正是ubuntu:latest镜像的四层。它们的挂载方式都是只读的(ro+wh)，都以增量的方式分别包含了Ubuntu操作系统的一部分，四层联合起来组成了一个成品。
##### 3.6.2 可读写层
**rootfs** 最上层的操作权限为 **rw**， 在没有写入文件之前，这个目录是空的。而一旦在容器里做了写操作，你修改产生的内容就会以增量的方式出现在这个层中。如果你想删除只读层里的文件，咋办呢？这个问题上面已经讲解过了。

最上面这个可读写层就是专门用来存放修改 **rootfs** 后产生的增量，无论是增、删、改，都发生在这里。而当我们使用完了这个被修改过的容器之后，还可以使用 docker commit 和 push 指令，保存这个被修改过的可读写层，并上传到 Docker Hub上，供其他人使用。并且原先的只读层里的内容则不会有任何变化，这就是**增量 rootfs** 的好处。
##### 3.6.3 init 层
它是一个以`-init`结尾的层，夹在只读层和读写层之间。Init层是Docker项目单独生成的一个内部层，专门用来存放 /etc/hosts 等信息。

需要这样一层的原因是这些文件本来属于只读的Ubuntu镜像的一部分，但是用户往往需要在启动容器时写入一些指定的值比如 hostname，那就需要在可读写层对它们进行修改。可是，这些修改往往只对当前的容器有效，我们并不希望执行 docker commit 时，把 **init** 层的信息连同可读写层一起提交。

最后这6层被组合起来形成了一个完整的 **Ubuntu** 操作系统供容器使用。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210110181735725.png)
# 4 Docker 网络
由上面的 Docker 原理可知 Docker 使用了 Linux 的 Namespaces 技术来进行资源隔离，如 PID Namespace 隔离进程，Mount Namespace 隔离文件系统，Network Namespace 隔离网络等。一个Network Namespace 提供了一份独立的网络环境(包括网卡、路由、Iptable规则)与其他的Network Namespace隔离，一个Docker容器一般会分配一个独立的Network Namespace。

当你安装Docker时，执行`docker network ls`会发现它会自动创建三个网络。
```bash
[root@server1 ~]$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
0147b8d16c64        bridge              bridge              local
2da931af3f0b        host                host                local
63d31338bcd9        none                null                local
```
我们在使用docker run创建Docker容器时，可以用 --net 选项指定容器的网络模式，Docker可以有以下4种网络模式：
|网络模式| 使用注意 |
|--|--|
|  host| 和宿主机共享网络 |
| none | 不配置网络 |
| bridge | docker默认，也可自创 |
| container | 容器网络连通，容器直接互联，用的很少 |

### 4.1 Host 模式
等价于Vmware中的桥接模式，当启动容器的时候用host模式，容器将不会虚拟出自己的网卡，配置自己的IP等，而是使用宿主机的IP和端口。但是容器的其他方面，如文件系统、进程列表等还是和宿主机隔离的。
### 4.2 Container 模式
Container 模式指定新创建的容器和已经存在的一个容器共享一个 Network Namespace，而不是和宿主机共享。新创建的容器不会创建自己的网卡，配置自己的IP，而是和一个指定的容器共享IP、端口范围等。同样，两个容器除了网络方面，其他的如文件系统、进程列表等还是隔离的。两个容器的进程可以通过lo网卡设备通信。
### 4.3 None 模式
None 模式将容器放置在它自己的网络栈中，并不进行任何配置。实际上，该模式关闭了容器的网络功能，该模式下容器并不需要网络（例如只需要写磁盘卷的批处理任务）。
### 4.4 Bridge 模式
Bridge 模式是 Docker 默认的网络设置，此模式会为每一个容器分配 Network Namespace、设置IP等。当Docker Server启动时，会在主机上创建一个名为**docker0**的虚拟网桥，此主机上启动的Docker容器会连接到这个虚拟网桥上。虚拟网桥的工作方式和物理交换机类似，主机上的所有容器就通过交换机连在了一个二层网络中。
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011119450868.png)
Docker 会从RFC1918所定义的私有IP网段中，选择一个和宿主机不同的**IP地址**和**子网**分配给docker0，连接到 docker0 的容器从子网中选择个未占用 IP 使用。一般 Docker 会用 172.17.0.0/16 这个网段，并将172.17.0.1/16 分配给 docker0 网桥（在主机上使用ifconfig命令是可以看到docker0的，可以认为它是网桥的管理接口，在宿主机上作为一块虚拟网卡使用）。

**网络配置的过程大致3步**：
1. 在主机上创建一对虚拟网卡 veth pair 设备。veth设备总是成对出现的，它们组成了一个数据的通道，数据从一个设备进入，就会从另一个设备出来。因此veth设备常用来连接两个网络设备。
2. Docker 将 veth pair 设备的一端放在新创建的容器中，并命名为eth0。另一端放在主机中，以veth65f9 这样类似的名字命名，并将这个网络设备加入到docker0网桥中，可以通过brctl show命令查看。
3. 从 docker0 子网中分配一个IP给容器使用，并设置 docker0 的IP地址为容器的默认网关。

**Bridge 模式下容器的通信**
1. 容器访问外部
>假设主机网卡为eth0，IP地址10.10.101.105/24，网关10.10.101.254。从主机上一个IP为172.17.0.1/16 的容器中ping百度（180.76.3.151）。首先IP包从容器发往自己的默认网关 docker0，包到达docker0后，会查询主机的路由表，发现包应该从主机的 eth0 发往主机的网关10.10.105.254/24。接着包会转发给eth0，并从eth0发出去。这时Iptable规则就会起作用，将源地址换为 eth0 的地址。这样，在外界看来，这个包就是从10.10.101.105上发出来的，Docker容器对外是不可见的。

2. 外部访问容器
> 创建容器并将容器的80端口映射到主机的80端口。当我们对主机 eth0 收到的目的端口为80的访问时候，Iptable规则会进行DNAT转换，将流量发往172.17.0.2:80，也就是我们上面创建的Docker容器。所以，外界只需访问10.10.101.105:80就可以访问到容器中的服务。
### 4.5 --link
容器创建后我们想通过容器名字来ping。此时需要用到--link，如下：
```java
docker run -d -P --name linux03 --link linux02 linux
docker exec -it linux03 ping linux02 可ping通。
docker exec -it linux02 ping linux03 不可ping通。
```
追本溯源 看下 linux03 的 /etc/hosts 会发现本质只是做了个host映射。
```java
172.17.0.3 linux03 12ft4tesa # 跟Windows的host文件一样，只是做了地址绑定
```
### 4.6 自建Bridge
我们之前直接启动的命令 (默认是使用--net bridge，可省)，这个bridge就是我们的docker0。下面俩是等价的。
```bash
docker run -d -P --name linux01 LinuxSelf
docker run -d -P --name linux01 --net bridge LinuxSelf
```
docker0默认不支持域名访问 ， 只能用 --link 打通连接。如果我们使用自定义的网络时，docker底层已经帮我们维护好了对应关系，可以实现域名访问。
```java
# --driver bridge 网络模式定义为 ：桥接 
# --subnet 192.168.0.0/16 定义子网 ，范围为：192.168.0.2 ~ 192.168.255.255 
# --gateway 192.168.0.1 子网网关设为： 192.168.0.1 
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
```
接下来
```java
docker run -d -P --name linux-net-01 --net mynet LinuxSelf
docker run -d -P --name linux-net-02 --net mynet LinuxSelf
docker exec -it linux-net-01 ping linux-net-02的IP  # 结果OK
docker exec -it linux-net-01 ping linux-net-02     # 结果OK
```
# 5 可视化界面
### 5.1 Portainer
[Portainer](https://gitee.com/mirrors/portainer) 是 Docker 的图形化管理工具，提供状态显示面板、应用模板快速部署、容器镜像网络数据卷的基本操作（包括上传下载镜像，创建容器等操作）、事件日志显示、容器控制台操作、Swarm集群和服务等集中管理和操作、登录用户管理和控制等功能。功能十分全面，基本能满足中小型单位对容器管理的全部需求。

### 5.2 DockerUI
DockerUI基于Docker API，提供等同Docker命令行的大部分功能，支持container管理，image管理。不过DockerUI一个致命的缺点就是 不支持多主机。
### 5.3 Shipyard
Shipyard 是一个集成管理docker容器、镜像、Registries的系统,它可以简化对横跨多个主机的Docker容器集群进行管理. 通过Web用户界面，你可以大致浏览相关信息，比如你的容器在使用多少处理器和内存资源、在运行哪些容器，还可以检查所有集群上的事件日志。
# 6 Docker 学习指南
本来也想写常见指令、Dockerfile、Docker Compose、Docker Swarm的，不过感觉还是拉闸吧，官方文档他不香啊！推荐几个学习指南。
>1. 官方文档：[https://docs.docker.com/engine/reference/commandline/build/](https://docs.docker.com/engine/reference/commandline/build/)
>2. 从入门到实践：[https://github.com/yeasy/docker_practice](https://github.com/yeasy/docker_practice)
>3. 在线教程：[https://vuepress.mirror.docker-practice.com](https://vuepress.mirror.docker-practice.com)
>4. PDF：公众号回复 docker


