# 《Netty4核心原理与手写RPC框架实战》读书笔记

## Java I/O 演进之路

### 什么是 I/O

在操作系统之中我们都知道在 UNIX 世界里一切皆文件，而文件呢就是一串二进制流而已，其实不管是 Socket，还是 FIFO、管道、终端。对计算机来说一切都是文件，一切都是流。**在信息交换的过程中，计算机都是对这些流进行数据的收发操作，简称 I/O 操作（Input and Output）。**

### I/O 交互流程

通过用户进程中的一次完整的 I/O 交互流程分为两阶段，首先是经过内核空间，也就是由操作系统处理；紧接着就是到用户空间，也就是交由应用程序。具体流程如下图所示。

[![image-20221006194022868](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210061940952.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210061940952.png)

I/O 有内存 I/O、网络 I/O 和磁盘 I/O 三种，通常我们说的 I/O 指的是后两者。如下图所示是 I/O 通信过程的调度示意。

[![image-20221006194305227](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210061943276.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210061943276.png)

## 五种 I/O 通信模型

在网络环境下，通俗地讲，将 I/O 分为两步：**第一步是等待；第二步是数据搬迁。**

如果想要提高 I/O 效率，需要将**等待时间降低**。因此发展出来五种 I/O 模型，分别是：**阻塞 I/O 模型、非阻塞 I/O 模型、多路复用 I/O 模型、信号驱动 I/O 模型、异步 I/O 模型。其中前四种被称为同步 I/O**，下面对每一种 I/O 模型进行详细分析。

### 阻塞 I/O 模型

阻塞 I/O 模型的通信过程示意如下图所示。

[![image-20221006205039163](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210062050240.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210062050240.png)

我们第一次接触的到的网络编程都是从 `listen()`、`send()`、`recv()` 等接口开始的，这些接口都是阻塞型的。都属于阻塞 I/O 模型

**在调用函数到数据返回的期间阻塞的。**在服务器实现模式为**一个连接对应一个线程**，即客户端有连接请求时服务器就需要启动一个线程进行处理，如果这个连接不做任何事情就会造成不必要的线程开销，可以通过**线程池机制改善（只能改善减少创建关闭线程的开销，但不能改善 BIO 本身的缺点）**。

[![image-20221006205154439](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210062051489.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210062051489.png)

### 非阻塞 I/O 模型

示意图如下。

[![image-20221006205243625](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210062052703.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210062052703.png)

当用户进程发出 read 操作时，如果内核中的数据还没有准备好，那么它并不会阻塞用户进程，而是立刻返回一个 error。从用户进程的角度讲，他发起一个 read 操作后，并不需要等待，而是马上就得到了一个结果，用户进程判断结果是一个 error 时，他就知道数据还没有准备好。于是它可以再次发送 read 操作，一旦内核中的数据准备好了，并且再次收到了用户进程的系统调用，那么它会马上将数据拷贝到用户内存，然后返回，非阻塞接口相比于阻塞接口的显著差异在于，在被调用之后立即返回。

[![image-20221006205945061](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210062059124.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210062059124.png)

> 非阻塞模式套接字与阻塞模式相比，不容易使用，使用非阻塞模式套接字，要编写更多的代码，但是，非阻塞模式套接字在控制建立多个链接、时间不定时，具有明显优势。

### 多路复用 I/O 模型

[![image-20221008183741246](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081837376.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081837376.png)

多个进程的 I/O 可以注册到一个复用器（Selector）上，当用户进程调用该 Selector，Selector 会监听注册进来的所有 I/O，如果Selector 监听的所有 I/O 在内核缓冲区都没有可读数据，select 调用进程会被阻塞，而当任一 I/O 在内核缓冲区中有可读数据时，select 调用进程就会返回，而后 select 调用进程可以自己或通知另外的进程（注册进程）再次发起读取 I/O，读取内核中准备好的数据，多个进程注册 I/O 后，只有一个 select 调用进程被阻塞。

> 其实多路复用 I/O 模型和阻塞 I/O 模型并没有太大的不同，事实上由于这里要使用两个系统调用而比阻塞 I/O 模型的性能还要差些。
>
> 多路复用 I/O 不一定比使用多线程加阻塞 I/O 的模式更优，甚至性能更佳，多路复用的优势在于可以处理更多的连接，而不是单个连接处理更快。

[![image-20221008184759480](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081847520.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081847520.png)

### 信号驱动 I/O 模型

[![image-20221008184819467](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081848507.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081848507.png)

信号驱动 I/O 是指进程预先告知内核，向内核注册一个信号处理函数，然后用户进程返回**不阻塞**，当内核**数据就绪时会发送一个信号给进程**，用户进程便在信号处理函数中调用 I/O 读取数据，从上图可以看出，**实际上 I/O 内核拷贝到用户进程的过程还是阻塞的，信号驱动 I/O 并没有实现真正的异步，因为通知到进程后，依然由进程来完成 I/O 操作。**

[![image-20221008185312018](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081853074.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081853074.png)

### 异步 I/O 模型

[![image-20221008185359939](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081853986.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081853986.png)

用户进程发起 aio_read 操作后，给内核传递与 read 相同的描述符、缓冲区指针、缓冲区大小三个参数及文件偏移，告诉内核当整个操作完成时，如何通知我们立刻就可以开始去做其他的事；而另一方面，从内核的角度，当他收到一个 aio_read 之后，首先他会立刻返回，所以不会对用户进程产生任何阻塞，内核会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，内核会给用户进程发送一个信号，告诉它 aio_read 操作完成。

> 异步 I/O 的工作机制是：告知内核启动某个操作，并让内核在整个操作完成后通知我们，这种模型与信号驱动 I/O 模型的区别在于，**信号驱动 I/O 模型是由内核通知我们何时可以启动一个 I/O 操作，这个 I/O 操作由用户自定义的信号函数来实现，而异步 I/O 模型由内核告知我们 I/O 操作何时完成。**

[![image-20221008192424929](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081924965.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081924965.png)

### 各 I/O 模型的对比与总结

前四种 I/O 模型都是同步 I/O 操作，它们的区别在于第一阶段，而第二阶段是一样的：数据（准备好后）从内核拷贝到应用缓冲区期间（用户空间），进程阻塞于 `recvfrom` 调用。

> recvfrom 会将数据从内核（Kernel）拷贝到用户内存中，这个时候进程就被阻塞了。在这段时间内，进程是被阻塞的。

[![image.png](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081929728.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081929728.png)

由上图可以看出，阻塞程度：阻塞 I/O > 非阻塞 I/O > 多路复用 I/O > 信号驱动 I/O > 异步 I/O，效率是由低到高的。

[![image-20221008193056732](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081930764.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081930764.png)

Java BIO 和 NIO 之间的主要差异。

[![image-20221008195234349](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081952389.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081952389.png)

## 易混淆概念解释

- 同步与异步：主要看请求发起方对消息结果的获取是主动发起还是被动通知的。

[![image-20221008193426801](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081934838.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081934838.png)

- 阻塞与非阻塞：调用一个函数后，在等待这个函数返回结果之前，当前的线程是处于挂起状态还是运行状态。

[![image-20221008193611650](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081936694.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081936694.png)

- 同步阻塞：请求方主动发起的，一直等待应答结果（用户线程阻塞挂起）；
- 异步非阻塞：请求方主动发起，但是可以去做其他的事情，但是需要不断轮询查看发起的请求是否有结果；
- 异步阻塞：请求方发起请求，一直阻塞等待答应结果（实际不应用）；
- 异步非阻塞：请求方发起请求，可以去干自己的事，服务会主动通知该请求已完成。

## NIO 介绍

### 缓冲区（Buffer）

在谈到缓冲区，**我们说缓冲区对象本质上是一个数组，但它其实是一个特殊的数组，缓冲区对象内置了一些机制，能够追踪和记录缓冲区的状态变化情况**，如果我们使用 `get()` 方法从缓冲区获取数据或者使用 `put()` 方法把数据写入缓冲区，都会引起缓冲区状态的变化。

> 缓冲区三个重要属性：
>
> - position：指定下一个将要被写入或者读取的元素索引，它的值由 get()/put() 方法自动更新，在新创建一个 Buffer 对象时，position 被初始化为 0。
> - limit：指定还有多少数据需要取出（在从缓冲区写入通道时），或者还有多少空间可以放入数据（在从通道读入缓冲区时）。
> - capacity：指定了可以存储在缓冲区的最大数据容量，实际上，它指定了底层数据的大小，或者至少时指定了准许我们使用的底层数组的容量。‘
>
> 注：0<= position <= limit <= capacity

在 NIO 中，所有的缓冲区类型都继承与抽象类 Buffer，最常用的就是 ByteBuffer，对于 Java 中的基本类型，基本都有一个具体 Buffer 类型与之相对应。

[![image-20221008201920235](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210082019274.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210082019274.png)

- **缓存区的分配**：可以通过调用静态方法 `allocate()` 来指定缓冲区的容量，其实调用 allocate 方法相当于**创建了一个指定大小的数组，并把它包装为缓冲区对象**。我们也可以**自己创建一个数组通过调用静态方法 `wrap()` 来将其包装为缓冲区对象。**
- **缓冲区分片**：根据现有的缓冲区对象创建一个子缓冲区，**即在现有缓冲区上切出一片作为一个新的缓冲区，但现有的缓冲区与创建的子缓冲区在底层数面上是数据共享的（子缓冲区相当于现有缓冲区的一个视图窗口）。**可以通过调用缓冲区对象的 `slice()` 创建。
- **只读缓冲区**：通过调用缓冲区对象的 `asReadOnlyBuffer()` 方法，将任何**常规缓冲区转换为只读缓冲区**，这个方法返回一个与原缓冲区**完全相同**的缓冲区，并与原缓冲区**共享数据**，只不过它是只读的。如果**原缓冲区的内容发生了变化，只读缓冲区的内容也随之发生变化**。**注意：尝试修改只读缓冲区的内容，则会报 ReadOnlyBufferException 异常；只可以 常规–> 只读 不可以 只读 –> 可写**
- **直接缓冲区**：直接缓冲区是为了加快 I/O 速度，使用一种特殊方式为其分配内存的缓冲区。**该缓冲区会在每一次调用底层操作系统的本机 I/O 操作之前（或之后），尝试避免将缓冲区内容拷贝到一个中间缓冲区拷贝数据。**通过调用静态方法 `allocateDirect()` 方法
- **内存映射**：比常规的基于流或者基于通道的 I/O 快得多。 **内存映射文件 I/O 通过使文件的数据表现为内存数组的内容来完成**。一般来说，**只有文件中实际读取或写入的部分才会映射到内存中**。

### 选择器（Selector）

NIO 中非阻塞 I/O 采用了基于 Reactor 模式的工作方式， I/O 调用不会被阻塞，而是注册感兴趣的特定 I/O 事件，如可读数据到达、新的套接字连接等，在发生特定事件时，系统再通知我们。NIO 中实现非阻塞 I/O 的核心对象是 Selector，Selector 是注册各种 I/O 事件的地方，而且当那些事情发生时，就是 Selector 告诉我们所发生的事件。

### 通道（Channel）

通道是一个对象，通过它可以读取和写入数据，当然所有数据都通过 Buffer 对象来处理。我们永远不会将字节直接写入通道，而是将数据写入包含一个或者多个字节的缓冲区。同样也不会直接从通道中读取字节，而是通过数据从通道读入缓冲区，再从缓冲区获取这个字节。

[![img](https://pic2.zhimg.com/80/v2-537ed6de4ca2cfeefd0dd11654519b65_720w.webp)](https://pic2.zhimg.com/80/v2-537ed6de4ca2cfeefd0dd11654519b65_720w.webp)

### 反应堆

阻塞 I/O 的通信模型如下图所示。

[![image-20221009180839283](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210091808348.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210091808348.png)

每个客户端连接成功后，服务端都会启动一个线程区处理该客户端请求。

**阻塞 I/O 通信模型缺点**

1. 当客户端多时，会创建大量的处理线程。且每个线程都要占用栈空间和一些 CPU 时间。
2. 阻塞可能带来频繁的上下文切换，且大部分上下文切换可能是无意义的。

在这种情况下非阻塞 I/O 就有了它的应用前景。

**Java NIO 工作原理。**

1. **有一个专门的线程来处理所有 I/O 事件，并负责分发。**
2. **事件驱动机制**：事件到的时候出发，而不是同步地去监视事件。
3. **线程通信**：线程之间通过 wait、notify 等方式通信。保证每次上下文切换都是**有意义的**，**减少无谓的线程切换**。

> Java NIO 反应堆工作原理图。
>
> [![image-20221009181444168](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210091814247.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210091814247.png)
>
> （注：每个线程的处理流程大概都是读取数据、解码、计算处理、编码和发送响应。）

### Netty 与 NIO

#### Netty 支持的功能与特性

根据定义，Netty 是一个异步的、事件驱动的、用来做高性能高可靠的网络应用的框架。优点如下：

1. 框架设计优雅，底层模型随意切换，适应不同的网络协议要求。
2. 提供了很多的协议、安全、编解码的支持。
3. 解决了很多 NIO 不易用的问题。
4. 社区更为活跃。

Netty 支持的功能与特性如下图所示。

[![image-20221009181932653](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210091819746.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210091819746.png)

1. 底层核心有：Zero-Copy-Capable Buffer，非常易用的零拷贝 Buffer；统一的 API；标准可扩展的事件模型。
2. 传输方面支持的有：管道通信；HTTP 隧道；TCP 与 UDP。
3. 协议方面的支持有：基于原始文本和二进制的协议；解压缩；大文件传输；流媒体传输；ProtoBuf 编解码；安全认证；HTTP 和 WebSocket。

#### Netty 采用 NIO 而非 AIO 的理由

> 1. Netty 不看重 Windows 上的使用，在 Linux 系统上，AIO 的底层实现仍使用 `epoll`，没有很好地实现 AIO，因此在性能上没有明显又是，且被 JDK 封装了一层，不容易深度优化。
> 2. Netty 整体架构采用 Reactor 模型，而 AIO 采用 Proactor 模型，混在一起会非常混乱，把 AIO 也改造成 Reactor 模型，看起来是把 Epoll 绕了个弯又绕回来。
> 3. AIO 还有个缺点是接受数据需要预先分配缓存， 而 NIO 是需要接收时才分配，所以对连接数量非常大但流量小的情况，AIO 会浪费很多内存。
> 4. Linux上 AIO 不够成熟，处理回调结果的速度跟不上处理需求。

## Nettty 高性能之道

### 传统 RPC 调用性能差的三大问题

#### 1. 网络传输方式存在弊端

传统的 RPC 框架或者居于 RMI 等方式的远程服务（过程）调用都是采用 BIO，当客户端的并发压力或者网络时延 增大的时候，BIO 会因频繁的 “wait” 导致 I/O 线程经常出席那阻塞的情况，由于线程本省无法高效地工作，I/O 处理能力自然就会下降。

**采用 BIO 通信模型的服务端**，通常由一个独立的 Acceptor 线程负责监听客户端的连接，接收到客户端连接之后为客户端创建一个新的线程处理请求消息，处理完成之后，返回应答消息给客户端，线程销毁，这就是典型的一请求一应答模型。**这样的架构设计，最大的问题就是无法进行弹性伸缩。当用户访问量剧增时，并发量自然上升，而服务端的线程个数和并发访问数成线性正比，由于线程是 JVM 非常宝贵的系统资源，所以随着并发量的持续增加、线程数急剧膨胀，系统的性能也急剧下降，可能会发生句柄和线程堆栈溢出等问题，最终可能导致服务器宕机。**

[![image-20221010194225522](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210101942644.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210101942644.png)

#### 2. 序列化方式存在弊端

Java 序列化存在如下几个较为典型的问题：

- 无法跨语言使用
- 比起第三方序列化框架，序列化后的字节流占用的空间太大（传输带宽占用太大）。
- 序列化性能较差，序列化时会占用较多的 CPU 资源。

#### 3. 线程模型存在弊端

由于传统的 RPC 框架均采用 BIO 模型，这使得每个 TCP 链家都需要分配 1 个线程，而线程资源是 JVM 非常宝贵的系统资源，当 I/O 读写阻塞时无法及时释放时，会导致系统性能急剧下降，甚至会导致虚拟机无法创建新的线程。

### Netty 高性能的三个主题

[![image-20221010194940812](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210101949866.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210101949866.png)

#### I/O 传输模型

用什么样的通道将数据发送给对方，是 BIO、NIO 还是 AIO，I/O 传输模型在很大程度上决定了框架的性能。

#### 数据协议

采用什么样的通信协议，是 HTTP 还是内部私有协议。协议的选择不同，性能模型也就不同。一般来说内部私有协议比公有协议的性能更高。

#### 线程模型

线程模型涉及如何读取数据包，读取之后的编解码在哪个线程中进行，编解码后的消息如何派发等方面。线程模型设计得不同，对性能也会产生非常大得影响。

### 异步非阻塞通信

与 Socket 类和ServerSocket 类相对应，NIO 也提供了 SocketChannel 和 ServerSocketChannel 两种不同的套接字通道实现。这两种新增的 Channel 都支持阻塞和非阻塞两种 I/O 模式。

1. 服务端得通行步骤

   [![image-20221010200208068](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102002142.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102002142.png)

2. 客户端通信步骤：

[![image-20221010200238215](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102002287.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102002287.png)

通过上面的序列图，我们大概能够了解到 Netty 的 I/O 线程 NioEventLoop 聚合了 Selector，可以同时并发处理成百上千个客户端 Channel，而且它的**读写操作都是非阻塞的**，这可以大幅提升 I/O 线程的运行效率，**避免由于频繁 I/O 阻塞导致的线程挂起**。另外，由于 Netty 采用的是**异步通信模式**，**单个 I/O 线程也可以并发处理多个用户端连接和读写操作**，所以从根本上解决了传统 BIO 的但连接单线程模型的弊端，使整个系统的性能、弹性伸缩性能和可靠性都得到了极大的提升。

### 零拷贝

在**操作系统**的层面上**零拷贝**是指**避免**在**用户态**(User-space) 与**内核态**(Kernel-space)之间**来回拷贝**数据的技术。Netty 中零拷贝与操作系统层面上的零拷贝是完全不一样的，**Netty 的零拷贝完全是在用户态（java层面）的，更多的是数据操作的优化。**

Netty 的零拷贝主要体现在如下五个方面。

1. Netty 接收和发送 ByteBuffer 采用 DirectBuffer，使用堆外直接内存进行 Socket 读写，不需要进行字节缓冲区的二次拷贝。**如果使用传统的堆存（Heap Buffer）进行 Socket 的读写。那么 JVM 会将推存拷贝一份到直接内存中，然后才写入 Socket。**相比于堆外直接内存，消息在发送过程中多了一次缓冲区的内存拷贝。
2. Netty 提供了多种组合 Buffer 对象，可以聚合多个 ByteBuffer 对象，用户可以像操作一个 Buffer 那样方面地对组合 Buffer 进行操作，避免了传统的通过内存拷贝的方式将几个小 Buffer 合并成一个大 Buffer 的繁琐操作。
3. Netty 中文件传输采用 `transferTo()` 方法，它可以直接将文件缓冲区的数据发送到目标 `Channel`，避免了传统通过循环 `write()` 方式导致的内存拷贝问题。
4. 通过 wrap 操作，我们可以将 `byte[]` 数组、ByteBuf、ByteBuffer 等包装成要给 Netty ByteBuf 对象，进而避免了拷贝技术。
5. ByteBuf 支持 slice 操作，可以将 ByteBuf分解为多个共享同一个存储区域的 ByteBuf，避免内存的拷贝。

> 对于很多操作系统，它接收文件缓冲区的内容直接发送给目标 Channel，而不需要从内核拷贝到应用程序内存，这种更加高效的传输实现了文件传输的零拷贝

> 这一块的详细文章推荐：[NIO效率高的原理之零拷贝与直接内存映射](https://mp.weixin.qq.com/s?__biz=MzUyNzgyNzAwNg==&mid=2247483933&idx=1&sn=d9776b9efe054b30523adbe60cb7524a&scene=21#wechat_redirect)

#### 内存池

随着技术的发展，对象的分配和回收已经是一个非常轻量级的工作了。但是对于缓冲区来说还是有些特殊，尤其是对于堆外直接内存的分配和回收，是一种耗时的操作。**为了尽量重复例用缓冲区内存**，Netty 设计了一套**基于内存池的缓冲区重用机制**。

```JAVA
package top.devildyw.netty;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.PooledByteBufAllocator;
import io.netty.buffer.Unpooled;

/**
 * Netty 内存池技术测试
 *
 * @author Devil
 * @since 2022-10-10-20:42
 */
public class PoolBufferTest {
    public static void main(String[] args) {
        final byte[] CONTENT = new byte[1024];
        int loop = 1800000;
        long startTime = System.currentTimeMillis();
        ByteBuf poolBuffer = null;

        System.out.println("----------------------采用内存池分配器创建直接缓冲区----------------------");
        for (int i = 0; i < loop; i++) {
            poolBuffer = PooledByteBufAllocator.DEFAULT.directBuffer(1024);
            poolBuffer.writeBytes(CONTENT);
            poolBuffer.release();
        }
        long endTime = System.currentTimeMillis();
        System.out.println("内存池分配缓冲区耗时"+(endTime-startTime)+"ms.");

        System.out.println("----------------------采用非 堆内存分配器创建直接缓冲区----------------------");

        long startTime2 = System.currentTimeMillis();
        ByteBuf buffer = null;
        for (int i = 0; i < loop; i++) {
            buffer = Unpooled.directBuffer();
            buffer.writeBytes(CONTENT);
            buffer.release();
        }
        endTime = System.currentTimeMillis();
        System.out.println("非内存池分配缓冲区耗时"+(endTime-startTime2)+"ms.");
    }
}
```

[![image-20221010210500830](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102105891.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102105891.png)

### 高效的 Reactor 线程模型

1. Reactor 单线程模型

[![image-20221010210701111](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102107177.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102107177.png)

`Acceptor` 负责接收客户端的 TCP 连接请求消息，链路建立成功之后，通过 `Dispatcher` 将对应的 `ByteBuffer` 派发到指定的 `Handler` 上进行消息解码，用户 `Handler` 通过 NIO 线程将消息发送给客户端。

对于并发量较小的业务场景，可以使用单线程模型。但单线程模型不适合高负载、高并发的场景。

1. Reactor 多线程模型

[![image-20221010211346809](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102113881.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102113881.png)

- 有一个专门的 NIO 线程 Acceptor 用于监听服务端、接收服务端的 TCP 连接请求。
- 网络 I/O 读、写等操作只有一个 NIO 线程池负责，可以采用标准的 JDK 线程池来实现，它包含一个任务队列和多个可用的线程，由这些 NIO 线程负责消息的读取、节码、编码和发送。
- 一个 NIO 线程可用同时处理多条请求链路，但是一条链路只对应一个 NIO 线程，防止发生并发串行。

1. 主从 Reactor 多线成模型

[![image-20221010211724752](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102117819.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102117819.png)

- 服务端用于接收客户端连接的不再是单个 NIO 线程，而是分配了一个独立的 NIO 线程池。Acceptor 接收到客户端 TCP 连接请求并处理完成后（可能包含接入认证等），将新创建的 SocketChannel 注册到 I/O 线程池（Sub Reactor 子线程池）的某个 I/O 线程上，由它负责 SocketChannel 的读写和编解码工作。
- Acceptor 线程仅仅用于客户端的登录、握手和安全认证，一旦链路建立成功，就将链路注册到后端 Sub Reactor 子线程池的 I/O 线程上，再由 I/O 线程负责后续的 I/O 操作。

利用主从Reactor多线程模型可以解决一个服务端监听线程无法有效处理所有客户端连接的性能不足的问题。因此，在Netty的官方Demo中，推荐使用该线程模型。

### 无锁化的串行设计理念

为了尽可能避免锁竞争带来的性能损耗，**可用通过串行化设计来避免多线程竞争和同步锁，即消息的处理尽可能在同一个线程内完成，不进行线程的切换。（减少上下文切换）**

**为了尽可能提升性能，Netty 采用了无锁化串行设计，在 I/O 线程内部进行串行操作，避免多线程竞争导致的性能下降。**表面上看似乎串行化设计对 CPU 利用率不高，并发程度不够。**但是通过调整 NIO 线程池的线程参数，可用同时启动多个串行的线程并行运行，这种局部无锁化的串行线程设计相比一个队列——多个工作线程的模型更优。**

[![image.png](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102137205.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102137205.png)

### 高效的并发编程

Netty 的高效并发编程主要体现在如下几点。

1. **volatile 关键字的大量且正确的使用。**
2. **CAS 和原子类的广泛使用。**
3. **线程安全容器的使用**
4. **通过读写锁提升并发性能。**

### 对高性能序列化框架的支持

Netty默认提供了对Google Protobuf的支持，用户也可以通过**扩展Netty的编解码接口接入其他高性能的序列化框架进行编解码**，例如Thrift的压缩二进制编解码框架。

### 灵活的 TCP 参数配置能力

合理设置 TCP 参数在某些场景下对性能的提升具有显著的效果，例如 `SO_RCVBUF` 和 `SO_SNDBUF`：通常建议值为 128KB 或者 256KB。如果设置不当，对性能的影响也是非常大的。

对性能影响比较大的几个配置项。

1. `SO_RCVBUF` 和 `SO_SNDBUF`：通常建议值为128KB或者256KB。
2. `SO_TCPNODELAY`：`Nagle` 算法通过将缓冲区内的小封包自动相连，组成较大的封包，阻止大量小封包的发阻塞网络，从而提高网络应用效率。但是对于延时敏感的应用场景需要关闭该优化算法。

> `Nagle` 算法是以其发明人 John Nagle 的名字命名的，**它用于将小的碎片数据连接成更大的保温来最小化所发送的报文数量。如果需要发送一些较小的保温，则需要禁用该算法**。Netty **默认禁用该算法**，从而使得传输的**延时最小化**。

1. 软中断：如果 Linux 内核版本支持 RPS（2.6.35 版本以上），开启 RPS 可以实现软中断，提升网络吞吐量。RPS 会**根据数据包的源地址、目的地址，已经源端口和目标端口进行计算得到一个 hash 值，然后根据这个 hash 值来选择软中断 CPU 的运行**。从**上层来看，也就是将每个连接和 CPU 绑定，通过这个 Hash 值在多个 CPU 上均衡软中断，提升网络并行处理性能。**

> Netty 在启动辅助类中可以灵活地配置 TCP 参数，满足不同的用户场景。相关配置如下表所示。（此表还有不详尽之处，大概了解，用作以后备）

[![image-20221010214936680](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102149808.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102149808.png)

[![image-20221010214957049](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102149175.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102149175.png)

[![image-20221010215112965](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102151047.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210102151047.png)