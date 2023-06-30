# Netty

## 1. 概述

1. Netty 是由 JBOSS 提供的一个 java 开源框架。它是一个**异步的**、**基于事件驱动**的**网络**应用框架，开源快速开发高性能、高可靠性的网络 IO 程序。
2. Netty 主要针对在 **TCP** 协议下，面向 Clients 端的**高并发**应用。或者 Peer-to-Peer 场景下的大量数据持续传输的应用。
3. Netty 是一个 NIO 框架，将 Java 底层的 NIO API 再次做了一个封装和优化，简化和流程化了 NIO 的开发过程。可以帮助你快速、简单的开发出一个网络应用。
4. Netty 是目前最流行的 NIO 框架，Netty 在互联网领域、大数据分布式计算领域、游戏行业、通信行业等获得了广泛的应用，知名的 Elasticsearch 、Dubbo（RPC） 框架内部都采用了 Netty。

## 2. 五种 I/O 通信模型

在网络环境下，通俗地讲，将 I/O 分为两步：**第一步是等待；第二步是数据搬迁。**

如果想要提高 I/O 效率，需要将**等待时间降低**。因此发展出来五种 I/O 模型，分别是：**阻塞 I/O 模型、非阻塞 I/O 模型、多路复用 I/O 模型、信号驱动 I/O 模型、异步 I/O 模型。其中前四种被称为同步 I/O**，下面对每一种 I/O 模型进行详细分析。

### 阻塞 I/O 模型

阻塞 I/O 模型的通信过程示意如下图所示。

[![image-20221006205039163](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224203.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224203.png)

我们第一次接触的到的网络编程都是从 `listen()`、`send()`、`recv()` 等接口开始的，这些接口都是阻塞型的。都属于阻塞 I/O 模型。

[![image-20221006205154439](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224250.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224250.png)

[![image-20221025131137955](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251311000.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251311000.png)

> 后续模型都是在 BIO 的基础上演化而来的。

### 非阻塞 I/O 模型

示意图如下。

[![image-20221006205243625](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224321.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224321.png)

当用户进程发出 read 操作时，如果内核中的数据还没有准备好，那么它并不会阻塞用户进程，而是立刻返回一个 error。从用户进程的角度讲，他发起一个 read 操作后，并不需要等待，而是马上就得到了一个结果，用户进程判断结果是一个 error 时，他就知道数据还没有准备好。于是它可以再次发送 read 操作，一旦内核中的数据准备好了，并且再次收到了用户进程的系统调用，那么它会马上将数据拷贝到用户内存，然后返回，非阻塞接口相比于阻塞接口的显著差异在于，在被调用之后立即返回。

[![image-20221006205945061](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224275.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224275.png)

> 非阻塞模式套接字与阻塞模式相比，不容易使用，使用非阻塞模式套接字，要编写更多的代码，但是，非阻塞模式套接字在控制建立多个链接、时间不定时，具有明显优势。

### 多路复用 I/O 模型

[![image-20221008183741246](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224314.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224314.png)

多个进程的 I/O 可以注册到一个复用器（Selector）上，当用户进程调用该 Selector，Selector 会监听注册进来的所有 I/O，如果Selector 监听的所有 I/O 在内核缓冲区都没有可读数据，select 调用进程会被阻塞，而当任一 I/O 在内核缓冲区中有可读数据时，select 调用进程就会返回，而后 select 调用进程可以自己或通知另外的进程（注册进程）再次发起读取 I/O，读取内核中准备好的数据，多个进程注册 I/O 后，只有一个 select 调用进程被阻塞。

> 其实多路复用 I/O 模型和阻塞 I/O 模型并没有太大的不同，事实上由于这里要使用两个系统调用而比阻塞 I/O 模型的性能还要差些。
>
> 多路复用 I/O 不一定比使用多线程加阻塞 I/O 的模式更优，甚至性能更佳，多路复用的优势在于可以处理更多的连接，而不是单个连接处理更快。

[![image-20221008184759480](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224264.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224264.png)

### 信号驱动 I/O 模型

[![image-20221008184819467](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224247.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224247.png)

信号驱动 I/O 是指进程预先告知内核，向内核注册一个信号处理函数，然后用户进程返回**不阻塞**，当内核**数据就绪时会发送一个信号给进程**，用户进程便在信号处理函数中调用 I/O 读取数据，从上图可以看出，**实际上 I/O 内核拷贝到用户进程的过程还是阻塞的，信号驱动 I/O 并没有实现真正的异步，因为通知到进程后，依然由进程来完成 I/O 操作。**

[![image-20221008185312018](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224253.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224253.png)

### 异步 I/O 模型

[![image-20221008185359939](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224250.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224250.png)

用户进程发起 aio_read 操作后，给内核传递与 read 相同的描述符、缓冲区指针、缓冲区大小三个参数及文件偏移，告诉内核当整个操作完成时，如何通知我们立刻就可以开始去做其他的事；而另一方面，从内核的角度，当他收到一个 aio_read 之后，首先他会立刻返回，所以不会对用户进程产生任何阻塞，内核会等待数据准备完成，然后将数据拷贝到用户内存，当这一切都完成之后，内核会给用户进程发送一个信号，告诉它 aio_read 操作完成。

> 异步 I/O 的工作机制是：告知内核启动某个操作，并让内核在整个操作完成后通知我们，这种模型与信号驱动 I/O 模型的区别在于，**信号驱动 I/O 模型是由内核通知我们何时可以启动一个 I/O 操作，这个 I/O 操作由用户自定义的信号函数来实现，而异步 I/O 模型由内核告知我们 I/O 操作何时完成。**

[![image-20221008192424929](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224272.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224272.png)

### 各 I/O 模型的对比与总结

前四种 I/O 模型都是同步 I/O 操作，它们的区别在于第一阶段，而第二阶段是一样的：数据（准备好后）从内核拷贝到应用缓冲区期间（用户空间），进程阻塞于 `recvfrom` 调用。

> recvfrom 会将数据从内核（Kernel）拷贝到用户内存中，这个时候进程就被阻塞了。在这段时间内，进程是被阻塞的。

[![image.png](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224382.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224382.png)

由上图可以看出，阻塞程度：阻塞 I/O > 非阻塞 I/O > 多路复用 I/O > 信号驱动 I/O > 异步 I/O，效率是由低到高的。

[![image-20221008193056732](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224287.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224287.png)

Java BIO 和 NIO 之间的主要差异。

[![image-20221008195234349](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081952389.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081952389.png)

## 易混淆概念解释

- 同步与异步：主要看请求发起方对消息结果的获取是主动发起还是被动通知的。

[![image-20221008193426801](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224283.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210251224283.png)

- 阻塞与非阻塞：调用一个函数后，在等待这个函数返回结果之前，当前的线程是处于挂起状态还是运行状态。

[![image-20221008193611650](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081936694.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202210081936694.png)

- 同步阻塞：请求方主动发起的，一直等待应答结果（用户线程阻塞挂起）；
- 异步非阻塞：请求方主动发起，但是可以去做其他的事情，但是需要不断轮询查看发起的请求是否有结果；
- 异步阻塞：请求方发起请求，一直阻塞等待答应结果（实际不应用）；
- 异步非阻塞：请求方发起请求，可以去干自己的事，服务会主动通知该请求已完成。

## NIO 介绍

### 为什么不选择 Java 原生 NIO 编程的原因

一般不建议开发者直接使用 JDK 的 NIO 类库进行开发，具体原因如下：

- **NIO 的类库和 API 繁杂，使用麻烦**，你需要熟练掌握 Selector、ServerSocketChannel、SocketChannel、ByteBuffer 等。
- **需要熟练其他的额外技能作为铺垫**，例如 Java 多线程编程。这是因为 NIO 编程涉及到 Reactor 模式，你必须对多线程和网络编程都十分熟悉，才能编写出高质量的 NIO 程序。
- **可靠性能力补齐，工作量和难度都非常大**。例如客户端面临断连接、网络闪断、半包读写、失败缓存，网络阻塞和异常码流的处理等问题，NIO 编程的特点是功能开发相对容易，但是可靠性能力补齐的工作量和难度都会非常大。
- **JDK NIO 的 BUG，例如 epoll bug**，它会导致 Selector 空轮询，最终导致 CPU 100%，虽然官方声称会在 JDK 1.6 的 update 18 修复这些问题，但是到了 JDK 1.7 问题依然还存在，他根本没有得到解决，只是该 BUG 发生概率降低了一些。

综上原因，在大多数场景下，不建议直接使用 JDK 的 NIO 类库，除非在有精通 NIO 编程的实力或者特殊的需求下，在大多数的业务场景中，完美可以使用 NIO 框架 Netty 来进行 NIO 编程。下面来看看 Netty 作为基础通信框架有哪些优点。

### 为什么选择 Netty

Netty 是业界最流行的 NIO 框架之一，它的健壮性、功能、性能、可定制性和可扩展性在同类框架中都是首屈一指的，它已经得到成百上千的商业项目验证，例如 Hadoop 的 RPC 框架 avro 使用 Netty 作为底层通信框架；很多其他业界主流的 RPC 框架（Dubbo、RocketMQ），也是用 Netty 来构建高性能的异步通信能力。

Netty 优点总结如下：

- API 简单，开发门槛低；
- 功能强大，预置了多种编解码功能，支持多种主流协议；
- 定制能力强，可以通过 ChannelHandler 对通信框架进行灵活地扩展。
- 性能高，通过与其他业界主流的 NIO 框架对比，Netty 的综合性能最优；
- 成熟、稳定，Netty 修复了已经发现的所有 JDK NIO BUG，业务人员不需要再为 NIO 的 BUG 而发愁；
- 社区活跃，版本迭代周期短，发现的 BUG 可以被及时修复，同时，更多的新功能会加入；
- 经历了大规模的商业应用考验，质量得到验证。被多个行业商业应用使用，证明了它已经完全能够满足不能行业的商业需求了。

正因为有上述的这些优点，Netty 逐渐称为了 Java NIO 编程的首选框架。

## Netty 入门应用

导入 Netty 依赖即可。

```XML
<!-- https://mvnrepository.com/artifact/io.netty/netty-all -->
<dependency>
    <groupId>io.netty</groupId>
    <artifactId>netty-all</artifactId>
    <version>4.1.84.Final</version>
</dependency>
```

### TimeServer
`TimeServer.java`

```JAVA
package top.devildyw.netty.server;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;

/**
 * @author Devil
 * @since 2022-11-07-15:10
 */
public class TimeServer {

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;
        new TimeServer().bind(port);
    }

    public void bind(int port) throws InterruptedException {
        //配置服务端的 NIO 线程组 创建两个 bossGroup线程组专用于接受客户端的请求，而workerGroup则专注于对客户端Channel请求的读写
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            //ServerBootstrap 对象是用于启动 NIO 服务端的辅助启动类，目的是降低开发的复杂度
            ServerBootstrap bootstrap = new ServerBootstrap();
            //将刚刚创建的两个线程组作为参数加入group方法中 根据前后顺序 前面的group 为 parent 后面的group为 child
            bootstrap.group(bossGroup,workerGroup)
                    //设置创建的 Channel为 NioServerSocketChannel
                    .channel(NioServerSocketChannel.class)
                    //配置tcp参数，设置backlog参数为1024
                    .option(ChannelOption.SO_BACKLOG,1024)
                    //配置childGroup 的Handler 用来处理网络I/O 事件，例如记录日志、对消息进行编解码等。
                    .childHandler(new ChildChannelHandler());

            //绑定端口，同步等待成功 返回ChannelFuture，功能类似与 JDK的 java.util.concurrent.Future 主要用于异步操作的通知回调。
            ChannelFuture f = bootstrap.bind(port).sync();

            //阻塞等待服务器监听端口关闭
            f.channel().closeFuture().sync();
        }finally {
            //优雅退出，释放线程池资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    private class ChildChannelHandler extends ChannelInitializer<SocketChannel>{
        @Override
        protected void initChannel(SocketChannel ch) throws Exception {
            ch.pipeline().addLast(new TimeServerHandler());
        }
    }

}
```
`TimeServerHandler`
```JAVA
package top.devildyw.netty.server;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.util.Date;

/**
 * @author Devil
 * @since 2022-11-07-15:16
 */
public class TimeServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //ByteBuf 类似与 JDK 中的 java.nio.ByteBuffer 对象，不过它提供了更强大和灵活的功能。
        ByteBuf buf = (ByteBuf) msg;

        //ByteBuf 的 readableBytes 方法可以获取缓冲区可读字节数，
        byte[] req = new byte[buf.readableBytes()];
        //将缓冲区的字节数组复制到新建的byte数组中
        buf.readBytes(req);
        String body = new String(req, "UTF-8");
        System.out.println("The time server receive order : "+body);

        String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body)?new Date(System.currentTimeMillis()).toString():"BAD ORDER";

        ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
        //写回
        ctx.write(resp);

    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        //为了防止频繁地唤醒Selector进行消息的发送，Netty 的 write 方法并不直接地将消息发送到缓存数组中，再通过 flush 方法，将发送缓冲区中的消息全部写入 SocketChannel
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //发生异常时，关闭ChannelHandlerContext，释放相关联的资源。
        ctx.close();
    }
}
```

------

### TimeClient

`TimeClient.java`
```JAVA
package top.devildyw.netty.client;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;

/**
 * @author Devil
 * @since 2022-11-07-17:57
 */
public class TimeClient {

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;

        new TimeClient().connect("127.0.0.1",port);
    }

    private void connect(String host, int port) throws InterruptedException {
        //配置客户端 NIO 线程组 创建客户端处理 I/O 读写的 线程组
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            //创建客户端辅助启动类
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    //配置Channel 类型 为 NIOSocketChannel 与 JDK 中的 java.nio.SocketChannel 类似
                    .channel(NioSocketChannel.class)

                    .option(ChannelOption.TCP_NODELAY,true)
                    //添加处理 I/O 的handler 作用在初始化它的时候将它的 ChannelHandler 设置到 ChannelPipeline 中 用于处理网络 I/O 事件。
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new TimeClientHandler());
                        }
                    });
            //发起异步连接操作 ，调用同步方法等待连接成功
            ChannelFuture future = bootstrap.connect(host, port).sync();

            //阻塞等待客户端链路关闭
            future.channel().closeFuture().sync();
        }finally {
            //优雅退出，释放NIO 线程组
            group.shutdownGracefully();
        }
    }
}
```
`TimeClientHandler`
```JAVA
package top.devildyw.netty.client;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerAdapter;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * @author Devil
 * @since 2022-11-07-18:06
 */
public class TimeClientHandler extends ChannelInboundHandlerAdapter {

    private final ByteBuf firstMessage;

    public TimeClientHandler() {
        byte[] req = "QUERY TIME ORDER".getBytes();

        firstMessage = Unpooled.buffer(req.length);
        firstMessage.writeBytes(req);
    }

    /**
     * 当客户端和服务端 TCP 链路建立成功之后，Netty 的 NIO 线程会调用 channel Active 方法，发送查询时间的指令给服务段
     * 调用 ChannelHandlerContext 的 writeAndFlush 方法将请求消息发送给服务端。
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ctx.writeAndFlush(firstMessage);
    }

    /**
     * 当服务端返回应答消息时，channelRead 方法被调用
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req,"UTF-8");
        System.out.printf("Now is : "+body);
    }

    /**
     * 发生异常时，打印异常日志。
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //释放资源
        ctx.close();
    }
}
```

## TCP 粘包/拆包问题的解决之道

TCP 是一个”**流**“协议，所谓流，就是没有界限的一大串的数据，就像河里的水是连成一片的，其间并没有分界线。TCP 底层并不了解上层业务数据的具体含义，它会根据 TCP 缓冲区的实际情况进行包的划分，所以在业务上认为，**一个完成的包可能会被 TCP 拆分成多个包进行发送，也有可能把多个小的包封装成一个打的数据包发送，这就是所谓的 TCP 粘包和拆包问题。**

### TCP 粘包/拆包

#### TCP 粘包/拆包问题说明

[![image-20221108134049344](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108134049344.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108134049344.png)

问题描述：假设客户端分别发送了两个数据包 D1 和 D2 给服务端， 由于服务端一次读取到的字节数是不确定的，故可能存在以下四种情况。

1. 服务器分两次读取到了两个独立的数据包，分别是 D1 和 D2，没有粘包和拆包；
2. 服务端一次接收到了两个数据包，D1 和 D2 粘合在了一起，被称为 TCP 粘包；
3. 服务端分两次读取到了两个数据包，第一次读取到了完成的 D1 包和 D2 包的部分内容，第二次读取到了 D2 包的剩余内存，这被称为 TCP 拆包。
4. 服务端分两次读取到了两个数据包，第一次读取到了 D1 包的部分内容 D1_1，第二次读取到了 D1 包的剩余内容 D1_2 和 D2 包的整包。

如果此时服务端 TCP 接收滑动窗口非常小，而数据包 D1 和 D2 非常大，很有可能会发生第五种可能，即**服务端分多次才能将 D1 和 D2 包完全接收，期间发生多次拆包。**

#### TCP 粘包/拆包发生的原因

原因共有三点，分别如下：

1. 应用程序 write 写入的字节大小大于套接口发生缓冲区大小；

2. 进行 MSS 大小的 TCP 分段；

   > MSS——MSS（Maximum Segment Size，最大报文段大小）的概念是**指TCP层所能够接收的最大段大小，该值只包括TCP段的数据部分，不包括选项部分。**

3. 以太网帧的 payload 大于 MTU 进行 IP 分片。

   > MTU——***MTU***（Maximum Transmission Unit，最大传输单元）来***限制所能传输的数据包大小*** 。**MTU是指一次传送的数据最大长度，不包括数据链路层数据帧的帧头。**如以太网的MTU为1500字节，实际上数据帧的最大长度为1514字节，其中以太网数据帧的帧头为14字节。

[![image-20221108162011707](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108162011707.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108162011707.png)

#### 粘包问题的解决策略

由于底层的 TCP 无法理解上层的业务数据，所以在底层是无法保证数据包不被拆分和重组的，这个问题只能通过上层的应用协议栈设计来解决，根据业界主流协议的解决方案，可以归纳如下：

1. 消息定长，例如每个报文的大小为固定长度 200 字节，如果不够，空格补位。
2. 在包尾添加回车换行符进行分割，例如 FTP 协议；
3. 将消息分为消息头和消息体，消息头中包含表示消息总长度（或者消息体长度）的字段，通常设计思路为消息头的第一个字段使用一个 int32 来表示消息的总长度；
4. 此外就是更复杂的应用层协议。

下面我们来看看如何使用 Netty 提供的半包解码器来解决 TCP 粘包/拆包问题。

### 未考虑 TCP 粘包导致功能异常的案例

前面编写的 Netty 入门应用中，我们并没有考虑读半包问题，这在功能测试时往往是没有问题的，但是一旦压力上来，或者发送大报文之后，就会存在**粘包/拆包**问题。如果不考虑，往往就会出现节码错位或者错误，导致程序不能正常工作。

下面我们来模拟故障场景。

#### TimeServer 的改造

Netty 时间服务器服务端处理器 `TimeServerHandler.java`

```JAVA
package top.devildyw.netty.stick_unpacking.server;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.util.Date;

/**
 * @author Devil
 * @since 2022-11-07-15:16
 */
public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    private int counter;
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //ByteBuf 类似与 JDK 中的 java.nio.ByteBuffer 对象，不过它提供了更强大和灵活的功能。
        ByteBuf buf = (ByteBuf) msg;

        //ByteBuf 的 readableBytes 方法可以获取缓冲区可读字节数，
        byte[] req = new byte[buf.readableBytes()];
        //将缓冲区的字节数组复制到新建的byte数组中
        buf.readBytes(req);
        String body = new String(req, "UTF-8").substring(0,req.length - System.getProperty("line.separator").length());

        System.out.println("The time server receive order : "+body +" ; the counter is " + ++counter);

        String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body)?new Date(System.currentTimeMillis()).toString():"BAD ORDER";
        currentTime = currentTime+System.getProperty("line.separator");
        ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
        ctx.writeAndFlush(resp);

    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //发生异常时，关闭ChannelHandlerContext，释放相关联的资源。
        ctx.close();
    }
}
```

每读到一条消息后，就计数一次，然后发送应答消息给客户端。按照设计，服务端接收到消息总数应该是跟客户端发送的消息总数相同，而且请求消息删除回城换行符后应该为 “QUERY TIME ORDER”。

#### TimeClient 的改造

Netty 时间服务器客户端 `TimeClientHandler.java`

```JAVA
package top.devildyw.netty.stick_unpacking.client;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * @author Devil
 * @since 2022-11-07-18:06
 */
public class TimeClientHandler extends ChannelInboundHandlerAdapter {


    private int counter;

    private byte[] req;
    public TimeClientHandler() {
        req = ("QUERY TIME ORDER"+System.getProperty("line.separator")).getBytes();

    }

    /**
     * 当客户端和服务端 TCP 链路建立成功之后，Netty 的 NIO 线程会调用 channel Active 方法，发送查询时间的指令给服务段
     * 调用 ChannelHandlerContext 的 writeAndFlush 方法将请求消息发送给服务端。
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ByteBuf message = null;
        for (int i = 0; i < 1000; i++) {
            message = Unpooled.buffer(req.length);
            //将请求写入缓冲区
            message.writeBytes(req);
            //将缓冲区中的数据写入到客户端的通道中
            ctx.writeAndFlush(message);
        }
    }

    /**
     * 当服务端返回应答消息时，channelRead 方法被调用
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ByteBuf buf = (ByteBuf) msg;
        byte[] req = new byte[buf.readableBytes()];
        buf.readBytes(req);
        String body = new String(req,"UTF-8");
        System.out.printf("Now is : "+body+" ; the counter is : "+ ++counter);
    }

    /**
     * 发生异常时，打印异常日志。
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //释放资源
        ctx.close();
    }
}
```

主要是修改了发送请求那一块，客户端与服务端建立连接后，循环发送 1000 条消息，没发送一条就刷新一次，保证每条消息都会被写入到 Channel 中。按照我们的计划服务端应该接收到 100 条查询时间指令的请求消息。

#### 运行结果

因结果较长，这里只放出部分结果。

服务端

[![image-20221108171311075](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108171311075.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108171311075.png)

服务端明确表明了它只接收到了 283 条消息，每一条都包含了不同数量的 ”QUERY TIME ORDER“ 指令，总数正好是 1000 条，我们期待的是收到 1000 条消息，每条包含一条 “QUERY TIME ORDER” 指令。这说明发生了 **TCP 粘包**。(**一次读取 读取到了多个包**)

客户端

[![image-20221108171640166](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108171640166.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108171640166.png)

设计初衷，客户端应该收到 1000 条当前系统时间的消息，但实际上只收到了 2 条应答，由于请求消息不满足查询条件，所以返回许多条 “BAD ORDER” 应答消息。但是实际上客户端只收到了2条包含若干条 “BAD ORDER” 指令的消息，说明服务端返回的应答消息也发生了粘包。

------

由于上述案例没有考虑 TCP 的粘包/拆包，所以当发生 TCP 粘包时，我们的程序就不能正常工作了。下面将 演示如何通过 Netty 的 `LineBasedFrameDecoder` 和 `StringDecoder` 来解决 TCP 粘包问题。

### 利用 LineBasedFrameDecoder 解决 TCP 粘包问题

Netty 默认提供了多种编解码器用于处理半包，只要能熟练掌握这些类库的使用，TCP 粘包问题会从此变得非常容易，你甚至不需要要去关注他们，这也是其他 NIO 框架 和 JDK 原生 NIO API 所无法匹敌的。

下面就演示如何使用 Netty 默认提供的编解码器来解决上述我们遇到的 TCP 粘包问题。还是以修正时间服务器为目标进行演示。

#### 支持 TCP 粘包的 TimeServer

`TimeServer.java`
```JAVA
package top.devildyw.netty.slove_stick_unpacking.server;

import io.netty.bootstrap.ServerBootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioServerSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringEncoder;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

/**
 * @author Devil
 * @since 2022-11-07-15:10
 */
public class TimeServer {
    private static final Logger LOGGER = LogManager.getLogger(TimeServer.class);
    public static void main(String[] args) throws InterruptedException {
        int port = 8081;
        new TimeServer().bind(port);
    }

    public void bind(int port) throws InterruptedException {
        //省略重复代码
        ......

        try {
            //省略重复代码
        	......
                    //配置childGroup 的Handler 用来处理网络I/O 事件，例如记录日志、对消息进行编解码等。
                    .childHandler(new ChildChannelHandler());

        //省略重复代码
        ......
    }

    private class ChildChannelHandler extends ChannelInitializer<SocketChannel>{
        @Override
        protected void initChannel(SocketChannel ch) throws Exception {
            ch.pipeline().addLast(new LineBasedFrameDecoder(1024)); //新增ChannelHandler
            ch.pipeline().addLast(new StringDecoder()); //新增ChannelHandler
            ch.pipeline().addLast(new TimeServerHandler());
        }
    }

}
```

在原来的 TimeServerHandler 之前新增了两个解码器：第一个是 `LineBasedFrameDecoder`，第二个是 `StringDecoder`。

`TimeServerHandler.java`

```JAVA
package top.devildyw.netty.slove_stick_unpacking.server;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

import java.util.Date;

/**
 * @author Devil
 * @since 2022-11-07-15:16
 */
public class TimeServerHandler extends ChannelInboundHandlerAdapter {

    private int counter;
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //ByteBuf 类似与 JDK 中的 java.nio.ByteBuffer 对象，不过它提供了更强大和灵活的功能。
        String body = (String) msg; //仔细观察
        
        System.out.println("The time server receive order : "+body +" ; the counter is " + ++counter);

        String currentTime = "QUERY TIME ORDER".equalsIgnoreCase(body)?new Date(System.currentTimeMillis()).toString():"BAD ORDER";
        
        currentTime = currentTime+System.getProperty("line.separator");
        ByteBuf resp = Unpooled.copiedBuffer(currentTime.getBytes());
        ctx.writeAndFlush(resp);

    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        //发生异常时，关闭ChannelHandlerContext，释放相关联的资源。
        ctx.close();
    }
}
```

**可以发现接收到的 msg 就是删除回车换行符后的请求消息，不需要额外考虑处理读半包问题，也不需要对消息进行编码，代码非常简洁。**（这就是新加两个 ChannelHandler 的作用）

#### 支持 TCP 粘包的 TimeClient

`TimeClient.java`
```JAVA
package top.devildyw.netty.slove_stick_unpacking.client;

import io.netty.bootstrap.Bootstrap;
import io.netty.channel.ChannelFuture;
import io.netty.channel.ChannelInitializer;
import io.netty.channel.ChannelOption;
import io.netty.channel.nio.NioEventLoopGroup;
import io.netty.channel.socket.SocketChannel;
import io.netty.channel.socket.nio.NioSocketChannel;
import io.netty.handler.codec.LineBasedFrameDecoder;
import io.netty.handler.codec.string.StringDecoder;
import org.apache.logging.log4j.LogManager;
import org.apache.logging.log4j.Logger;

/**
 * @author Devil
 * @since 2022-11-07-17:57
 */
public class TimeClient {

    private static final Logger LOGGER = LogManager.getLogger(TimeClient.class);
    public static void main(String[] args) throws InterruptedException {
        int port = 8081;

        new TimeClient().connect("127.0.0.1",port);
    }

    private void connect(String host, int port) throws InterruptedException {
        //省略重复代码
        ......
        try {
            //省略重复代码
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new LineBasedFrameDecoder(1024)); //新增 ChannelHandler
                            ch.pipeline().addLast(new StringDecoder()); //新增 ChannelHandler
                            ch.pipeline().addLast(new TimeClientHandler());
                        }
                    });
        //省略重复代码
        ......
    }
}
```

同样是直接在 TimeClientHandler 之前新增 `LineBasedFrameDecoder` 和 `StringDecoder` 解码器。

`TimeClientHandler.java`
```JAVA
package top.devildyw.netty.slove_stick_unpacking.client;

import io.netty.buffer.ByteBuf;
import io.netty.buffer.Unpooled;
import io.netty.channel.ChannelHandlerContext;
import io.netty.channel.ChannelInboundHandlerAdapter;

/**
 * @author Devil
 * @since 2022-11-07-18:06
 */
public class TimeClientHandler extends ChannelInboundHandlerAdapter {
	//省略重复代码
        ......

    /**
     * 当服务端返回应答消息时，channelRead 方法被调用
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;
        System.out.printf("Now is : "+body+" ; the counter is : "+ ++counter);
    }
    
	//省略重复代码
        ......

}
```

可以发现 `channelRead(...)` 拿到的 msg **已经是解码成字符串之后的消息了**，相比于之前的代码简洁了许多。

#### 运行结果

客户端连续发送 1000 条请求给服务端，查看服务端和客户端的运行结果。

同样因为测试结果过长这里只放出关键部分。

**服务端**

[![image-20221108175300709](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108175300709.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108175300709.png)

**客户端**

[![image-20221108175518462](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108175518462.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221108175518462.png)

程序的运行结果完全符合预期，说明通过使用 `LineBasedFrameDecoder` 和 `StringDecoder` 成功解决了 TCP 粘包导致的读半包问题。对于使用者来说，**只要将支持半包解码的 handler 添加到 ChannelPipeline 中即可**，操作简单，代码简洁。

------

#### LineBasedFrameDecoder 和 StringDecoder 的原理分析

`LineBaseFrameDecoder` 的工作原理是它**依次遍历 ByteBuf 中的可读字节，判断看是否有 “\n” 或者 “\r\n”，如果有，就以此位置为结束位置，从可读索引到结束位置区间的字节就组成了一行。**

它是**以换行符为结束标志**的解码器，支持**携带结束符**或者**不携带结束符**两种解码方式，同时支持**配置单行最大长度**，如果**连续读取到最大长度**后仍然没有发现换行符，就会**抛出异常**，同时**忽略**掉之前读到的异常码流。

------

`StringDecoder` 的功能很简单，就是**将接收到的对象转换为字符串，然后继续调用后面的 handler。**

LineBasedFrameDecoder + StringDecoder 组合就是**按行切换的文本解码器**，它被设计用来支持 TCP 的粘包和拆包。

上述这种组合**只能针对以换行符或者回车换行符的消息解码**。其他类型的解码，Netty 也支持，Netty 提供了多种支持 TCP 粘包/拆包的解码器，用来满足用户的不同诉求。

## 分隔符和定长解码器的应用

TCP 以流的方式进行数据的传输，上层协议为了对消息进行区分，往往采用如下四种方式：

1. 消息定长，累计读到长度总和为定长 LEN 的报文后，就认为读到了一个完整的消息；将计数器置位，重新开始读取下一个数据报；
2. 将回车换行符作为消息结束符，例如 FTP 协议，就种方式再文本协议中应用比较广泛；
3. 将特殊的分割符作为消息阶数标志，回车换行符就是一种特殊的结束分隔符；
4. 通过在消息头中定义长度字段来表示消息的总长度。

本节介绍另外两种解码——DelimiterBasedFrameDecoder 和 FixedLengthFrameDecoder，前者可以自动完成以分隔符做结束标志的消息的解码，后者可以自动完成对定长消息的解码，他们都能解决 TCP 粘包/拆包导致的读半包问题。

### DelimiterBasedFrameDecoder 应用开发

Echo 服务，EchoServer 接收到 EchoClient 的请求消息后，将其打印出来，然后将原始消息返回给客户端，消息以 “$_” 作为分隔符。

#### 服务端

`EchoServer.java`
```JAVA
/*...*/

public class EchoServer {
    private static final Logger LOGGER = LogManager.getLogger(EchoServer.class);
    public static void main(String[] args) throws InterruptedException {
        int port = 8081;
        new EchoServer().bind(port);
    }

    public void bind(int port) throws InterruptedException {
        /*省略代码*/
        try {
            //ServerBootstrap 对象是用于启动 NIO 服务端的辅助启动类，目的是降低开发的复杂度
            ServerBootstrap bootstrap = new ServerBootstrap();
            //将刚刚创建的两个线程组作为参数加入group方法中 根据前后顺序 前面的group 为 parent 后面的group为 child
            bootstrap.group(bossGroup,workerGroup)
                    //设置创建的 Channel为 NioServerSocketChannel
                    .channel(NioServerSocketChannel.class)
                    //配置tcp参数，设置backlog参数为1024
                    .option(ChannelOption.SO_BACKLOG,1024)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    //配置childGroup 的Handler 用来处理网络I/O 事件，例如记录日志、对消息进行编解码等。
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            
                            ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
                            ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024,delimiter));
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new EchoServerHandler());
                        }
                    });

            /*省略代码*/
    }

}
```

本例程使用 “$_” 作为分隔符。创建 `DelimiterBasedFrameDecoder` 对象，将其加入到 `ChannelPipeline` 中，即完成配置。

`DelimiterBasedFrameDecoder` 有多个构造方法，这里我们传入两个参数，第一个 1024 表示**单条消息的长度**，当到达该长度后，仍然没有查找到分隔符，就抛出 `TooLongFrameException` 异常，**防止由于异常码流确实分隔符而导致的内存溢出，这是 Netty 解码器的可靠性保护**；第二个参数就是分隔符缓存对象。

`EchoServerHandler.java`
```JAVA
@ChannelHandler.Sharable //添加该注解可以将同一个ChannelHandler实例添加到多个 pipeline 中; 如果不添加则每次都要创建一个新的 ChannelHandler
public class EchoServerHandler extends ChannelInboundHandlerAdapter {

    private int counter = 0;
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        String body = (String) msg;

        /*返回给客户端时需要给消息尾部拼接分隔符 “$_”，最后创建 ByteBuf ，将原始消息重新返回给客户端。*/
        System.out.println("This is "+ ++counter + "times receiver cliet : {"+
                body+"}");
        body +="$_";
        ByteBuf echo = Unpooled.copiedBuffer(body.getBytes());
        ctx.writeAndFlush(echo);

    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        //发生异常时，关闭ChannelHandlerContext，释放相关联的资源。
        ctx.close();
    }
}
```

由于 DelimiterBasedFrameDecoder **自动对请求消息进行了解码**（以 “$_” 作为分隔符进行解码），后续的 ChannelHandler 接收到的 msg 对象就是个**完整的消息包**；由于第二个 ChannelHandler 是 StringDecoder，它**将 ByteBuf 解码成字符串对象**；所以我们自定义的 ChannelHandler 接收到的消息就是解码后的字符串对象。

#### 客户端

```JAVA
/*...*/

public class EchoClient {

    private static final Logger LOGGER = LogManager.getLogger(EchoClient.class);
    public static void main(String[] args) throws InterruptedException {
        int port = 8081;

        new EchoClient().connect("127.0.0.1",port);
    }

    private void connect(String host, int port) throws InterruptedException {
        /*省略代码*/
        
        try {
            //创建客户端辅助启动类
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    //配置Channel 类型 为 NIOSocketChannel 与 JDK 中的 java.nio.SocketChannel 类似
                    .channel(NioSocketChannel.class)

                    .option(ChannelOption.TCP_NODELAY,true)
                    //添加处理 I/O 的handler 作用在初始化它的时候将它的 ChannelHandler 设置到 ChannelPipeline 中 用于处理网络 I/O 事件。
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ByteBuf delimiter = Unpooled.copiedBuffer("$_".getBytes());
                            ch.pipeline().addLast(new DelimiterBasedFrameDecoder(1024,delimiter));
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new EchoClientHandler());
                        }
                    });
            
            /*省略代码*/
        }
    }
}
```

与服务端类似，分别将 `DelimiterBasedFrameDecoder` 和 `StringDecoder` 添加到客户端 ChannelPipeline 中，最后添加客户端 I/O 事件处理类 `EchoClientHandler`。

```JAVA
public class EchoClientHandler extends ChannelInboundHandlerAdapter {


    private int counter;

    static final String ECHO_REQ = "Hi Lilinfeng, Welcome to Netty.$_";


    /**
     * 当客户端和服务端 TCP 链路建立成功之后，Netty 的 NIO 线程会调用 channel Active 方法，发送查询时间的指令给服务段
     * 调用 ChannelHandlerContext 的 writeAndFlush 方法将请求消息发送给服务端。
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        ByteBuf message = null;
        //发送 1000 条消息到服务端
        for (int i = 0; i < 1000; i++) {
            ctx.writeAndFlush(Unpooled.copiedBuffer(ECHO_REQ.getBytes()));
        }
    }

    /**
     * 当服务端返回应答消息时，channelRead 方法被调用
     * @param ctx
     * @param msg
     * @throws Exception
     */
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("This is "+ ++counter + "times receive server :{"
        + msg + "}");
    }


    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    /**
     * 发生异常时，打印异常日志。
     * @param ctx
     * @param cause
     * @throws Exception
     */
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        //释放资源
        ctx.close();
    }
}
```

#### 运行结果

**服务端**

[![image-20221109181435846](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221109181435846.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221109181435846.png)

**客户端**

[![image-20221109181459800](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221109181459800.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221109181459800.png)

可以发现 `DelimiterBasedFrameDecoder` 将我们的分隔符 “$_” 过滤掉，而解决了 TCP 粘包/拆包的问题。

### FixedLengthFrameDecoder 应用开发

`FixedLengthFrameDecoder` 是固定长度解码器，它能够按照指定的长度对消息进行自动解码，开发者不需要考虑 TCP 的粘包/拆包问题，非常实用。

#### 服务端

在服务端的 `ChannelPipeline` 中新增 `FixedLengthFrameDecoder` ，长度设置为 20，然后依次增加字符串解码器和 `EchoServerHandler`。

`EchoServer.java`
```JAVA
/*...*/

public class EchoServer {
    private static final Logger LOGGER = LogManager.getLogger(EchoServer.class);
    public static void main(String[] args) throws InterruptedException {
        int port = 8081;
        new EchoServer().bind(port);
    }

    public void bind(int port) throws InterruptedException {
       /*省略代码*/
        
        try {
            //ServerBootstrap 对象是用于启动 NIO 服务端的辅助启动类，目的是降低开发的复杂度
            ServerBootstrap bootstrap = new ServerBootstrap();
            //将刚刚创建的两个线程组作为参数加入group方法中 根据前后顺序 前面的group 为 parent 后面的group为 child
            bootstrap.group(bossGroup,workerGroup)
                    //设置创建的 Channel为 NioServerSocketChannel
                    .channel(NioServerSocketChannel.class)
                    //配置tcp参数，设置backlog参数为1024
                    .option(ChannelOption.SO_BACKLOG,1024)
                    .handler(new LoggingHandler(LogLevel.INFO))
                    //配置childGroup 的Handler 用来处理网络I/O 事件，例如记录日志、对消息进行编解码等。
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new FixedLengthFrameDecoder(20));
                            ch.pipeline().addLast(new StringDecoder());
                            ch.pipeline().addLast(new EchoServerHandler());
                        }
                    });

            /*省略代码*/
    }

}
```

EchoServerHandler 的功能比较简单，直接将读取到的消息打印出来。

`EchoServerHandler.java`
```JAVA
/*...*/

public class EchoServerHandler extends ChannelInboundHandlerAdapter {

    private int counter = 0;
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        System.out.println("Receive client : {"+ msg + "}");

    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        //发生异常时，关闭ChannelHandlerContext，释放相关联的资源。
        ctx.close();
    }
}
```

利用 `FixedLengthFrameDecoder` 解码器，无论 一次接收到了多少数据报，它都会按照构造函数中设置的固定长度进行解码，如果是半包消息，`FixedLengthFrameDecoder` 会**缓存半包消息并等待下个包到达后进行拆包，知道读取到一个完整的包**。

#### 运行结果

我们通过 telnet 命令行来测试 `EchoServer` 服务端，就可以省去编写客户端的步骤。

[![image-20221109192258617](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221109192258617.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221109192258617.png)

输入`ffffffffffffffffff`

服务端接收到的

[![image-20221109192456534](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221109192456534.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221109192456534.png)

服务端的运行结果完全符合预期，FixedLengthFrameDecoder 解码器按照 20 个字节长度对请求消息进行截取。

## 编解码技术

基于 Java 提供的对象输入/输出流 `ObjectInputStream` 和 `ObjectOutputStream`，可以直接**把 Java 对象作为可存储的字节数组写入文件**，也可以**传输到网络上**。对于程序员来说，基于 JDK 默认的序列化方式机制可以避免操作底层数组，从而提升开发效率。

Java 序列化的目的主要有两个：

- **网络传输**
- **对象持久化**

关于网络传输，当进行远程跨进程服务调用，需要把被传输的 Java 对象编码为 字节数组或者 ByteBuffer 对象。而当远程服务读取到 ByteBuffer 对象或者字节数组时，需要将其解码为发送时的 Java 对象，这被称为 **Java 对象编解码技术**。

对象持久化，就是将 Java 对象编码为一种可以持久化存储在磁盘文件上的格式，当需要读取时，需要将其解码为存储前的 Java 对象。

Java 序列化仅仅是 Java 编解码技术的一种，由于**它有着种种的缺陷**，衍生出了多中编解码技术和框架。

------

评判一个编解码框架的优劣时，往往会考虑以下几个因素。

- **是否支持跨语言，支持的语言种类是否丰富；**
- **编码后的码流大小；**
- **编解码的性能；**
- **类库是否小巧，API 使用是否方便；**
- **使用者需要手工开发的工作量和难度。**

### Java 序列化的框架

Java 的序列化从 JDK 1.1 版本就已经提供，它不需要添加额外的类库，只需要实现 java.io.Serializable 并生成序列 ID 即可，因此它从诞生之初就得到了广泛应用。

但是因为 Java 原生序列化机制的缺陷，我们在远程调用服务（RPC）时，很少使用，原因如下：

- **无法跨语言**
- **序列化后的码流太大**
- **序列化性能太低**

------

#### 无法跨语言

对于跨进程的服务调用时，服务提供者可能会使用其他非 Java 语言进行开发，**当我们需要和异构语言进程交互时，Java 序列化就难以胜任**。

Java 序列化技术是 Java 语言**内部的私有协议**，其他语言不支持，对于用户来说它完全是黑盒。对于 Java 序列化后的字节数组，别的语言无法进行反序列化，这就是严重阻碍了它的应用。

事实上，目前几乎所有主流的 Java RPC 通信框架，都没有使用 Java 序列化作为编解码框架，主要原因就是它无法跨语言，而这些 RPC 框架往往需要支持跨语言调用。

#### 序列化后的码流太大

POJO 对象类 UserInfo 该类的实例就是我们要序列化的对象

```JAVA
public class UserInfo implements Serializable {
    /**
    * 默认的序列号
    */
    private static final long serialVersionUID = 1L;

    private String userName;

    private int userID;

    public UserInfo buildUserName(String userName){
        this.userName = userName;
        return this;
    }

    public UserInfo buildUserID(int userID){
        this.userID = userID;
        return this;
    }

    public final void setUserName(String userName) {
        this.userName = userName;
    }

    public final int getUserID() {
        return userID;
    }

    public final void setUserID(int userID) {
        this.userID = userID;
    }

    public final String getUserName(){
        return userName;
    }

    public byte[] codeC() {
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        byte[] value = this.userName.getBytes();
        buffer.putInt(value.length);
        buffer.put(value);
        buffer.putInt(this.userID);
        buffer.flip();
        value = null;
        byte[] result = new byte[buffer.remaining()];
        buffer.get(result);
        return result;
    }

}
```

UserInfo 对象是一个普通的 POJO 对象，它实现了 java.io.Serializable 接口，并且生成了一个默认的序列化号 serialVersionUID = 1L，这说明 UserInfo 对象可以通过 JDK 默认的序列化机制进行序列化和反序列化。

在该类的 `codeC()` 方法中，我们使用 ByteBuffer 的通用二进制编解码技术对 UserInfo 对象进行了编码，编码结果仍然是 byte 数组。可以与传统的 JDK 序列化后的码流大小进行对比。

`TestUserInfo.java`
```JAVA
public class TestUserInfo {

    public static void main(String[] args) throws IOException {
        UserInfo info = new UserInfo();
        info.buildUserID(100).buildUserName("Welcome to Netty");

        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream os = new ObjectOutputStream(bos);
        os.writeObject(info);
        os.flush();
        os.close();
        byte[] b = bos.toByteArray();

        System.out.println("The jdk serializable length is : " +b.length);
        bos.close();
        System.out.println("--------------------------------------------");
        System.out.println("The byte array serializable length is : "+ info.codeC().length);

    }
}
```

运行结果

[![image-20221110002831188](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221110002831188.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221110002831188.png)

JDK 序列化机制编码后的二进制数组大小竟然是二进制编码 5.x 倍。

**在同等情况下，编码后的字节数组越大，存储的时候就约占空间，存储的硬件成本就越高，并且在网络传输时更占带宽，导致系统吞吐量降低。**

#### 序列化性能太低

修改代码为如下：

`UserInfo.java`

```JAVA
public class UserInfo implements Serializable {
    private static final long serialVersionUID = 1L;

    private String userName;

    private int userID;

    public UserInfo buildUserName(String userName){
        this.userName = userName;
        return this;
    }

    public UserInfo buildUserID(int userID){
        this.userID = userID;
        return this;
    }

    public final void setUserName(String userName) {
        this.userName = userName;
    }

    public final int getUserID() {
        return userID;
    }

    public final void setUserID(int userID) {
        this.userID = userID;
    }

    public final String getUserName(){
        return userName;
    }

    public byte[] codeC(ByteBuffer buffer) {
        buffer.clear();
        byte[] value = this.userName.getBytes();
        buffer.putInt(value.length);
        buffer.put(value);
        buffer.putInt(this.userID);
        buffer.flip();
        value = null;
        byte[] result = new byte[buffer.remaining()];
        buffer.get(result);
        return result;
    }

}
```
`TestUserInfo.java`
```JAVA
public class TestUserInfo {

    public static void main(String[] args) throws IOException {
        UserInfo info = new UserInfo();
        info.buildUserID(100).buildUserName("Welcome to Netty");

        int loop = 1000000;
        ByteArrayOutputStream bos = null;
        ObjectOutputStream os = null;
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < loop; i++) {
            bos = new ByteArrayOutputStream();
            os = new ObjectOutputStream(bos);
            os.writeObject(info);
            os.flush();
            os.close();
            bos.toByteArray();
            bos.close();
        }


        long endTime = System.currentTimeMillis();
        System.out.println("The jdk serializable cost time is : "+(endTime - startTime) + " ms");

        System.out.println("--------------------------------------------------");
        ByteBuffer buffer = ByteBuffer.allocate(1024);
        startTime = System.currentTimeMillis();
        for (int i = 0; i < loop; i++) {
            byte[] bytes = info.codeC(buffer);
        }
        endTime = System.currentTimeMillis();
        System.out.println("The byte array serializable cost time is : "+ (endTime-startTime) + " ms");
    }
}
```

**运行结果**

[![image-20221110160953859](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221110160953859.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221110160953859.png)

通过运行结果可以发现，Java序列化的性能只有二进制编码的1/13左右，可见 Java 原生序列化的性能是在太差了。

Java 序列化和二进制编码的性能的差异比对图：

[![image-20221110161803025](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221110161803025.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221110161803025.png)

由图可知，无论是序列化后的码流大小，还是序列化的性能，JDK 默认的序列化机制表现得都很差。因此我们不会选择 Java 序列化作为远程跨节点调用得编解码框架。

### 业界主流得编解码框架

#### Google 的 Protobuf

Protobuf 全称 Google Protocol Buffers，它由谷歌开源而来，在谷歌内部经久考研。它将数据结构以.proto 文件进行描述，通过代码可以生成对应数据结构的 POJO 对象和 Protobuf 相关的方法和属性。

特点如下：

- **结构化数据存储格式（XML，JSON等）；**
- **高效的编解码性能；**
- **语言无关、平台无关、拓展性好；**
- **官方支持 Java、C++ 和 Python 三种语言。**

> 首先我们来看下为什么不适用 XML，尽管 XML 的可读性和可扩展性非常好，也非常适合描述数据结构，但是 XML 解析的时间开销和 XML 为了可读性而牺牲的空间开销都非常大，因此不适合做高性能的通信协议。Protobuf 使用 二进制编码，在空间和性能上具有更大的优势。

Protobuf 另一个吸引人的地方就是它的**数据描述文件和代码生成机制**，利用数据描述文件对数据结构进行说明的优点如下。

- **文本化的数据结构描述语言，可以实现语言和平台无关，特别适合异构系统间的集成；**
- **通过标识字段的顺序，可以实现协议的向前兼容。**
- **自动代码生成，不需要手工编写同样数据结构的 C++ 和 Java 版本；**
- **方便后续的管理和维护。相比于代码，结构化的文档更容易管理和维护。**

Protobuf 编解码和其他几种序列化框架的性能对比数据如下：

**Protobuf 编解码和其他几种序列化框架的响应时间对比**：

[![查看源图像](https://m.haicoder.net/uploads/pic/architecture/netty/netty-codec/01%20protobuf%20%E5%93%8D%E5%BA%94%E6%97%B6%E9%97%B4%E5%AF%B9%E6%AF%94.png)](https://m.haicoder.net/uploads/pic/architecture/netty/netty-codec/01 protobuf 响应时间对比.png)

**Protobuf 和其他几种序列化框架的字节数对比：**

[![查看源图像](https://haicoder.net/uploads/pic/architecture/netty/netty-codec/02%20protobuf%20%E5%92%8C%E5%85%B6%E4%BB%96%E5%BA%8F%E5%88%97%E5%8C%96%E6%A1%86%E6%9E%B6%E5%AD%97%E8%8A%82%E6%95%B0%E5%AF%B9%E6%AF%94.png)](https://haicoder.net/uploads/pic/architecture/netty/netty-codec/02 protobuf 和其他序列化框架字节数对比.png)

可以看到 Protobuf 的编解码性能远高于其他几种序列化框架，这也是很多 RPC 框架选用 Protobuf 做编解码框架的原因。

#### Facebook 的 Thrift

Thrift 源于 Facebook。创造 Thrift 是为了解决 Facebook 各系统间大数据量的传输通信以及系统间语言环境不同需要跨平台的特性，因此 Thrift 可以支持多种程序语言（包括 C++、C#、Java 和 Python 等）。

在多种不同的语言之间通信，Thrift 可以作为高性能的通信中间件使用，它支持数据（对象）序列化和多种类型的 RPC 服务。**Thrift 是用于静态的数据交换，需要先确定好它的数据结构，当数据结构发生变化时，必须重新编辑 IDL 文件，生成代码和编译**，这一点跟其他 IDL 工具相比可以视为是 Thrift 的弱项。**Thrift 适用于搭建大型呼叫交换及存储和通用工具，对于大型系统中的内部数据传输，相对于 JSON 和 XML 在性能和传输大小上都有明显的优势。**

Thrift 由五部分组成。

1. **语言系统以及 IDL 编译器**：**负责由用户给定的 IDL 文件生成相应的语言的接口代码**
2. **TProtocol**：RPC 的协议层，可以选择多种不同对象的序列化方式，如 JSON 和 XML；
3. **TTransport**：RPC 的传输层，同样可以选择不同的传输层实现，如 Socket、NIO、MemoryBuffer 等。
4. **TProcessor**：作为协议层和用户提供的服务实现之间的纽带，负责调用服务实现的接口；
5. **TServer**：聚合 TProtocol、TTransport 和 TProcessor 等对象。

> 由于 **Thrift 的 RPC 服务调用和编解码绑定在一起**，所以，通常我们使用 Thrift 的时候会采用 RPC 框架的方式。但是，它的 **TProtocol 编解码框架还是可以以类库的方式独立使用的**。

**与 Protobuf 比较类似的是，Thrift 通过 IDL 描述接口和数据结构定义**，它支持 8 中 Java 基本类型、Map、Set 和 List，支持可选和必选定义，功能非常强大。因为可以**定义数据结构中字段的顺序，所以它也可以支持协议的前后兼容。**

Thrift 支持三种比较典型的编解码方式。

- **通用的二进制编解码；**
- **压缩二进制编解码；**
- **优化的可选字段压缩编解码。**

由于支持二进制压缩编解码，Thrift 的编解码性能表现也相当优异，远超 Java 序列化和 RMI 等。

[![image-20221110203807601](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221110203807601.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221110203807601.png)

#### JBoss Marshalling 介绍

JBoss Marshalling 是一个 Java 对象的序列化 API 包，**修正了 JDK 自带的序列化包的很多问题**，**但又保持跟 `java.io.Serializable` 接口的兼容**；同时增加了一些可调的参数和附加的特性，并且这些参数和特性可通过工厂类进行配置。

相较于传统的 Java 序列化机制，它的优点如下：

- **可插拔的类解析器，提供更加便捷的类加载定制策略，通过一个接口即可实现定制；**
- **可插拔的对象替换技术，不需要通过继承的方式；**
- **可插拔的预定义缓存表，可以减小序列化的字节数组长度，提升常用类型的对象序列化性能；**
- **无需实现 java.io.Serializable 接口，即可实现 Java 序列化；**
- **通过缓存技术提升对象的序列化性能。**

相较于前面两种编解码框架，JBoss Marshalling 更多是在 JBoss 内部使用，应用范围有限，

## Java 序列化

### Netty Java 序列化服务端开发

服务端开发的场景如下：Netty 服务端接收到客户端的用户订购请求消息，消息定义如下：

`SubscribeReq` 消息定义：

```JAVA
@Data
public class SubscribeReq implements Serializable {//继承 java.io.Serializable 接口
    /**
     * 默认的序列号
     */
    private static final long serialVersionUID = 1L;

    private int subReqId;

    private String userName;

    private String productName;

    private String phoneNumber;

    private String address;
    
}
```

服务端接收到消息，对用户名进行合法性校验，如果合法，则构造订单成功的应答消息返回给客户端。订购应答消息的定义如下：

`SubscibeResp` 消息定义：

```JAVA
@Data
public class SubscribeResp implements Serializable {

    /**
     * 默认序列ID
     */
    private static final long serialVersionUID = 1L;
    
    private int subReqId;
    
    private int respCode;
    
    private String desc;

}
```

使用 Netty 对 POJO 对象进行序列化的开发步骤如下。

1. 在服务端 ChannelPipeline 中新增解码器 `io.netty.handler.codec.serialization.ObjectDecoder`；
2. 在服务端 ChannelPipeline 中新增对象编码器 `io.netty.handler.codec.serialization.ObjectEncoder`；
3. 需要进行 Java 序列化的 POJO 对象必须实现 `java.io.Serializable` 接口。

**服务端**

订购服务主函数 `SubReqServer`

```JAVA
public class SubReqServer {
    public void bind(int port) throws InterruptedException {
        //配置服务端的 NIO 线程组
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        
        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG,100)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(
                                    new ObjectDecoder(
                                            1024*1024,
                                            ClassResolvers
                                                    .weakCachingConcurrentResolver(this.getClass()
                                                            .getClassLoader())
                                    )
                            );
                            ch.pipeline().addLast(new ObjectEncoder());
                            ch.pipeline().addLast(new SubReqServerHandler());
                        }
                    });
            
            //绑定端口，同步等待成功
            ChannelFuture future = bootstrap.bind(port).sync();

            //等待服务端监听端口关闭
            future.channel().closeFuture().sync();
        }finally {
            //优雅退出，释放线程池资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;
        
        new SubReqServer().bind(port);
    }
}
```

> **`ObjectDecoder`\**，主要负责对实现 Serializable 接口的 POJO 对象进行\**解码**，它有多个构造函数，支持不同的 ClassResovler，在此我们使用 **weakCacheingConcurrentResolver 创建线程安全的 WeakReferenceMap 对类加载器进行缓存**，它支持多线程并发访问，**当虚拟机内存不足时，会释放缓存中的内存，防止内存泄漏**。为了防止**异常码流和解码错位导致的内存溢出**，这里将单个对象最大序列化后的字节数组长度设置为了 1 M。
>
> **`ObjectEncoder`** ，它可以在消息发送的时候自动将实现 Serializable 接口的 POJO 对象进行**编码**，因此用户无需亲自对对象进行手工序列化，只需要关注自己的业务逻辑即可。

`SubReqServerHandler`
```JAVA
public class SubReqServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //经过 handler ObjectDecoder 的解码,SubReqServerHandler 接收到的请求消息以及被自动解码为了 SubscribeReq 对象，可以直接使用。
        SubscribeReq req = (SubscribeReq) msg;
        //对用户名进行合法性校验。
        if ("Devildyw".equals(req.getUserName())){
            System.out.println("Service accept client subscribe req : {"+req.toString()+"}");
            ctx.writeAndFlush(resp(req.getSubReqId()));
        }
    }

    private SubscribeResp resp(int subReqId) {
        SubscribeResp resp = new SubscribeResp();
        resp.setSubReqId(subReqId);
        resp.setRespCode(0);
        resp.setDesc("Netty Book order succeed, 3 days later, sent to the designated address");
        return resp;
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close(); //发生异常，关闭链路
    }
}
```

### Netty Java 序列化 客户端开发

客户端设计思路如下：

1. 创建客户端的时候将 Netty 对象解码器 和编码器添加到 ChannelPipeline；
2. 链路被激活的时候构造订购请求消息发送，为了检验 Netty 的 Java 序列化功能是否支持 TCP 粘包/拆包，客户端一次构造 100 条消息，最后一次性发送给服务端；
3. 客户端订购处理 Handler 将接收到的 订单响应消息打印出来。

产品订购客户端 **客户端**

```JAVA
public class SubReqClient {
    public void connect(String host, int port) throws InterruptedException {
        //配置客户端 NIO 线程组
        NioEventLoopGroup group = new NioEventLoopGroup();

        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .option(ChannelOption.TCP_NODELAY,true)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(
                                    new ObjectDecoder(
                                            1024,
                                            //禁止对类加载器进行缓存，它在基于 OSGI 的动态模块化编程中经常使用。
                                            ClassResolvers.cacheDisabled(this.getClass()
                                                    .getClassLoader())
                                    )
                            );
                            ch.pipeline().addLast(new ObjectEncoder());
                            ch.pipeline().addLast(new SubReqClientHandler());
                        }
                    });
            //发起异步链接操作
            ChannelFuture future = bootstrap.connect(host, port).sync();

            //等待客户端链路关闭
            future.channel().closeFuture().sync();
        }finally {
            //优雅关闭资源
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;

        new SubReqClient().connect("127.0.0.1",port);
    }
}
```

> 这里我们禁止了对类加载器的缓存，它在基于 OSGI 的动态模块化编程中经常使用。由于 OSGI 的bundle 可以进行热部署和热升级，当某个 bundle 升级后，它对应的类加载器也将一同升级，因此**在动态模块化编程过程中，很少对某个类加载器进行缓存，因为它随时可能发生变化。**

`SubReqClientHandler`
```JAVA
public class SubReqClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        for (int i = 0; i < 100; i++) {
            ctx.write(subReq(i));
        }
        //将100条消息一次性地发送给服务端
        ctx.flush();
    }

    private SubscribeReq subReq(int i) {
        SubscribeReq req = new SubscribeReq();
        req.setSubReqId(i);
        req.setAddress("cuit");
        req.setPhoneNumber("XXXXXXXXXXXXXXXXXXXXX");
        req.setUserName("Netty 权威指南");
        req.setUserName("Devildyw");
        return req;
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //自动解码，所以接收到的已经是解码后的订单应答消息了。
        System.out.println("Receive server response : ["+(SubscribeResp)msg+"]");
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

### 运行结果

**服务端**

[![image-20221111164800421](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111164800421.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111164800421.png)

**客户端**

[![image-20221111164818348](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111164818348.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111164818348.png)

与预取结果一致，ObjectDecoder 会在接收消息时自动解码反序列化为 POJO 对象，ObjectEncoder 会在发送消息之前将 POJO 对象编码为进行序列化，除此之外，**Netty 的 Java 序列化功能还支持解决 TCP 粘包/拆包问题**。

## Google Protobuf 编解码

### Protobuf 的入门

#### Protobuf 开发环境搭建

首先在 github 上下载 protobuf 最新 windows 版本，网址如下：

https://github.com/protocolbuffers/protobuf/releases

[![image-20221111183959658](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111183959658.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111183959658.png)

再 Maven 项目的 pom.xml 中配置如下，即可免去手动编译，通过插件来编译更为方便。

```XML
<dependencies>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
        <version>4.1.78.Final</version>
    </dependency>
    <dependency>
        <groupId>com.google.protobuf</groupId>
        <artifactId>protobuf-java</artifactId>
        <version>3.19.6</version>
    </dependency>
</dependencies>
```
```XML
<build>
    <plugins>
        <plugin>
            <groupId>org.xolstice.maven.plugins</groupId>
            <artifactId>protobuf-maven-plugin</artifactId>
            <version>0.6.1</version>
            <configuration>
                <protocExecutable>
                    D:\Utils\protoc\bin\protoc.exe  <!-- 刚刚安装的位置 -->
                </protocExecutable>
                <pluginId>protoc-java</pluginId>
                <!-- proto文件放置的目录 -->
                <protoSourceRoot>${project.basedir}/src/main/java/top/devildyw/netty/serialization/protobuf/proto</protoSourceRoot>
                <!-- 生成文件的目录 -->
                <outputDirectory>${project.basedir}/src/main/java</outputDirectory>
                <!-- 生成文件前是否把目标目录清空，这个最好设置为false，以免误删项目文件 -->
                <clearOutputDirectory>false</clearOutputDirectory>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>compile</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>

</build>
```

protoc.exe 工具主要根据 .proto 文件生成代码，下面我们定义 SubscribeReq.ptoto 和 SubsrcibeResp.proto 来测试生成代码。

数据文件定义如下：
`SubscribeReq.proto`

```JAVA
syntax = "proto3";
option java_package = "top.devildyw.netty.serialization.protobuf.POJO";
option java_multiple_files = true;
option java_outer_classname = "SubscribeReqProto";


message SubscribeReq{
   int32 subReqId = 1;

   string userName = 2;

   string productName = 3;

   string address = 4;
}
SubscribeResp.proto
JAVA
syntax = "proto3";
option java_package = "top.devildyw.netty.serialization.protobuf.POJO";
option java_multiple_files = true;
option java_outer_classname = "SubscribeRespProto";

message SubscribeResp{
  int32 subReqID = 1;
  int32 respCode = 2;

  string desc = 3;
}
```

通过执行 maven 插件 `protobuf:compile` 命令生成 Java 代码。

[![image-20221111185756195](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111185756195.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111185756195.png)

可以看到指定文件夹下已经有了生成好的 Java 代码了。

[![image-20221111190018948](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111190018948.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111190018948.png)

### Protobuf 编解码开发

Protobuf 的类库使用比较简单，下面就通过 SubscribeReqProto 进行编解码来介绍。

```JAVA
public class TestSubscribeReqProto {
    private static byte[] encode(SubscribeReq req){
        return req.toByteArray();
    }

    private static SubscribeReq decode(byte[] body) throws InvalidProtocolBufferException {
        return SubscribeReq.parseFrom(body);
    }

    private static SubscribeReq createSubscribeReq(){
        SubscribeReq.Builder builder = SubscribeReq.newBuilder();
        builder.setSubReqId(1);
        builder.setUserName("Devildyw");
        builder.setProductName("Netty Book");
        ArrayList<String> address = new ArrayList<>();
        address.add("NanJing YuHuaTai");
        address.add("BeiJing LiuLiChang");
        builder.addAllAddress(address);
        return builder.build();
    }

    public static void main(String[] args) throws InvalidProtocolBufferException {
        SubscribeReq req = createSubscribeReq();
        System.out.println("Before encode : "+req.toString());
        SubscribeReq req2 = decode(encode(req));
        System.out.println("After decode : "+ req.toString());
        System.out.println("Assert equal : -->" + req2.equals(req));
    }
}
```

通过 SubscribeReq 的静态方法 newBuilder 创建 SubscribeReq 的 Builder 实例，通过 Builder 构建器对 SubscribeReq 的属性进行设置，对于集合类型，通过 addAllXXXA() 方法可以将集合对象设置到对应的属性中。

**编码时**通过 SubscribeReq 实例的 **toByteArray** 即可将 SubscribeReq 编码为 byte 数组。

**解码时**通过 SubscribeReq 的静态方法 **parseFrom** 将二进制 byte 数组解码为原始的对象。

> Protobuf 的编解码接口非常简单和实用，但是功能和性能却非常强大，这也是它流行的一个重要原因。

运行结果：

[![image-20221111200306184](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111200306184.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221111200306184.png)

结果表明，经过 Protobuf 编解码后，生成的 SubscribeReq 与编码前原始的 SubscribeReq **等价**。

### Netty 的 Protobuf 服务端开发

开发一个 Protobuf 版本的订书系统。

#### Protobuf 版本的图书订购服务端开发

```JAVA
public class SubReqServer {
    public void bind(int port) throws InterruptedException {
        //配置服务端的 NIO 线程组
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup, workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG, 100)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new ProtobufVarint32FrameDecoder());

                            ch.pipeline().addLast(
                                    new ProtobufDecoder(
                                            SubscribeReq.getDefaultInstance()
                                    )
                            );
                            ch.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
                            ch.pipeline().addLast(new ProtobufEncoder());
                            ch.pipeline().addLast(new SubReqServerHandler());
                        }
                    });

            //绑定端口，同步等待成功
            ChannelFuture future = bootstrap.bind(port).sync();

            //等待服务端监听端口关闭
            future.channel().closeFuture().sync();
        } finally {
            //优雅退出，释放线程池资源
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;

        new SubReqServer().bind(port);
    }
}
```

在 channelPipeline 中我们添加了 `ProtobufVarint32FrameDecoder`，它主要用于半包处理，随后继续添加 `ProtobufDecoder` 解码器，它的参数是 `com.google.protobuf.MessageLite`，实际上是要告诉 ProtobufDecoder 需要**解码的目标是什么**。

`SubReqServerHandler`
```JAVA
public class SubReqServerHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //经过 handler ObjectDecoder 的解码,SubReqServerHandler 接收到的请求消息以及被自动解码为了 SubscribeReq 对象，可以直接使用。
        SubscribeReq req = (SubscribeReq) msg;
        //对用户名进行合法性校验。
        if ("Devildyw".equals(req.getUserName())){
            System.out.println("Service accept client subscribe req : {"+req.toString()+"}");
            //由于使用了 ProtobufEncoder，所以不需要对 SubscribeResp 进行手工编码
            ctx.writeAndFlush(resp(req.getSubReqId()));
        }
    }

    private SubscribeResp resp(int subReqId) {
        SubscribeResp.Builder builder = SubscribeResp.newBuilder();
        builder.setSubReqID(subReqId);
        builder.setRespCode(0);
        builder.setDesc("Netty Book order succeed, 3 days later, sent to the designated address");
        return builder.build();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close(); //发生异常，关闭链路
    }
}
```

#### Protobuf 版本的图书订购客户端开发

`SubReqClient`
```JAVA
public class SubReqClient {
    public void connect(String host, int port) throws InterruptedException {
        //配置客户端 NIO 线程组
        NioEventLoopGroup group = new NioEventLoopGroup();

        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .option(ChannelOption.TCP_NODELAY,true)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            ch.pipeline().addLast(new ProtobufVarint32FrameDecoder());
                            ch.pipeline().addLast(
                                    new ProtobufDecoder(
                                            SubscribeResp.getDefaultInstance()
                                    )
                            );
                            ch.pipeline().addLast(new ProtobufVarint32LengthFieldPrepender());
                            ch.pipeline().addLast(new ProtobufEncoder());
                            ch.pipeline().addLast(new SubReqClientHandler());
                        }
                    });
            //发起异步链接操作
            ChannelFuture future = bootstrap.connect(host, port).sync();

            //等待客户端链路关闭
            future.channel().closeFuture().sync();
        }finally {
            //优雅关闭资源
            group.shutdownGracefully();
        }
    }

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;

        new SubReqClient().connect("127.0.0.1",port);
    }
}
```

与服务端一样需要在 ChannelPipeline 中添加 Protobuf 相关的半包处理器，编/解码器，需要配置对什么消息对象进行解码。

`SubReqClientHandler`
```JAVA
public class SubReqClientHandler extends ChannelInboundHandlerAdapter {
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        //将100条消息写入缓冲区中
        for (int i = 0; i < 100; i++) {
            ctx.write(subReq(i));
        }
        //将100条消息一次性地发送给服务端
        ctx.flush();
    }

    private SubscribeReq subReq(int i) {
        SubscribeReq.Builder builder = SubscribeReq.newBuilder();
        builder.setSubReqId(i);
        ArrayList<String> list = new ArrayList<>();
        list.add("cuit");
        list.add("cdu");
        builder.addAllAddress(list);
        builder.setProductName("XXXXXXXXXXXXXXXXXXXXX");
        builder.setUserName("Netty 权威指南");
        builder.setUserName("Devildyw");
        return builder.build();
    }

    @Override
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        //自动解码，所以接收到的已经是解码后的订单应答消息了。
        System.out.println("Receive server response : ["+(SubscribeResp)msg+"]");
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

#### 运行结果

**客户端**

[![image-20221114170555109](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221114170555109.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221114170555109.png)

**服务端**

[![image-20221114170940457](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221114170940457.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221114170940457.png)

**总结**：利用 Netty 提供的 Protobuf 编解码能力，我们不需要了解 Protobuf 实现和使用细节的请况下就能轻松支持 Protobuf 编解码，可以方便地实现跨语言的远程服务调用和与周边的异构系统进行通信对接。

### Netty 使用 Protobuf 的注意事项

`ProtobufDecoder` **仅仅负责解码，它不支持读半包**。因此，在 `ProtobufDecoder` **前面**，一定要有能够读半包的解码器，有如下三种方式可选：

- 使用 Netty 提供的 ProtobufVarint32FrameDecoder，它可以处理半包消息；
- 继承 Netty 提供的通用半包解码器 LengthFieldBasedFrameDecoder；
- 继承 ByteToMessageDecoder 类，自己处理半包消息。

注意：**如果你只使用 ProtobufDecoder 解码器而忽略对半包消息的处理，程序是不能正常工作的。**

## HTTP 协议开发应用

HTTP 协议是目前 Web 开发的主流协议，基于 HTTP 应用非常广泛，因此，掌握 HTTP 的开发非常重要。由于 Netty 的 HTTP 协议栈是基于 Netty 的 NIO 通信框架开发的，因此，Netty 的 HTTP 协议也是异步非阻塞的。

[计算机网络](https://devildyw.github.io/2022/10/18/计算机网络/)

### Netty HTTP 服务端入门开发

由于 Netty 天生是异步事件驱动的架构，因此基于 NIO TCP 协议栈开发的 HTTP 协议栈也是**异步非阻塞**的。

Netty 的 HTTP 协议栈无论在性能还是可靠性上，都表现优异，非常适合在非 Web 容器的场景下应用，相比于传统的 Tomcat、Jetty 等 Web 容器，它更加**轻量和小巧，灵活性和定制性也更好**。

#### HTTP 服务端例程场景描述

我们以文件服务器为例，例程场景如下：文件服务器使用 HTTP 协议对外提供服务，当客户端通过浏览器访问文件服务器时，对访问路径进行检查，检查失败时返回 HTTP 403 错误，该页无法访问；如果校验通过，以链接的方式打开当前文件目录，每个目录或者文件都是个超链接，可以递归访问。

如果是目录，可以继续递归访问它下面的子目录或者文件，如果是文件且可读，则可以在浏览器端直接打开，或者通过【目标另存为】下载该文件。

#### HTTP 服务端开发

```JAVA
public class HttpFileServer {

    private static final String DEFAULT_URL = "";

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;
        String url = DEFAULT_URL;
        new HttpFileServer().run(port,url);
    }

    private void run(int port, String url) throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            //添加 HTTP 消息解码器
                            ch.pipeline().addLast("http-decoder",new HttpRequestDecoder());
                            //添加 HttpObjectAggregator解码器，它的作用是将多个消息转换为单一的 FullHttpRequest 或者 FullHttpResponse ，原因是 Http 解码器在每个HTTP 消息中会生成多个消息对象。
                            ch.pipeline().addLast("http-aggregator",new HttpObjectAggregator(65536));
                            //添加 HTTP 响应编码器，对 HTTP 响应消息进行编码；
                            ch.pipeline().addLast("http-encoder",new HttpResponseEncoder());
                            //新增 ChunkedHandler 它的主要作用是支持异步发送大的码流（例如大的问价你传输），但不占用过多的内存，防止发生java内存溢出的错误。
                            ch.pipeline().addLast("http-chunked",new ChunkedWriteHandler());
                            //自定义 ChannelHandler 用于文件服务器的业务逻辑处理。
                            ch.pipeline().addLast("fileServerHandler",new HttpFileServerHandler(url));
                        }
                    });

            ChannelFuture future = bootstrap.bind("localhost", port).sync();
            System.out.println("HTTP 文件目录服务器启动，网址是 ： "+"http://localhost:"+port+url);
            future.channel().closeFuture().sync();
        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```
`HttpFileServerHandler`
```JAVA
import javax.activation.MimetypesFileTypeMap;
import java.io.File;
import java.io.FileNotFoundException;
import java.io.RandomAccessFile;
import java.io.UnsupportedEncodingException;
import java.net.URLDecoder;
import java.util.regex.Pattern;

import static io.netty.handler.codec.http.HttpHeaderNames.*;
import static io.netty.handler.codec.http.HttpHeaderValues.KEEP_ALIVE;
import static io.netty.handler.codec.http.HttpResponseStatus.*;
import static io.netty.handler.codec.http.HttpVersion.HTTP_1_1;
import static io.netty.util.CharsetUtil.UTF_8;


/**
 * @author Devil
 * @since 2022-11-15-16:54
 */
@ChannelHandler.Sharable
public class HttpFileServerHandler extends SimpleChannelInboundHandler<FullHttpRequest> {


    private final String url;

    public HttpFileServerHandler(String url) {
        this.url = url;
    }


    @Override
    protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest request) throws Exception {
        //首先对http请求消息的解码结果进行判断
        if (!request.decoderResult().isSuccess()) {
            //如果解码失败，直接构造 HTTP 400 错误返回。
            sendError(ctx, BAD_REQUEST);
            return;
        }
        //对 HTTP 请求方法进行判断
        if (request.method() != HttpMethod.GET) {
            //如果不是 GET 请求 则构造 405 错误
            sendError(ctx, METHOD_NOT_ALLOWED);
            return;
        }

        //对 URL 进行包装
        final String uri = request.uri();
        final String path = sanitizeUri(uri);
        //如果构造的 URI 不合法，则返回 HTTP 403 错误。
        if (path == null) {
            sendError(ctx, FORBIDDEN);
            return;
        }

        //使用新组装的URI 构造 File 对象。
        File file = new File(path);
        //如果文件不存在或者是系统隐藏文件，则构造 404 异常返回
        if (file.isHidden() || !file.exists()) {
            sendError(ctx, NOT_FOUND);
            return;
        }
        //如果文件是一个目录，则发送目录的链接给客户端浏览器。。
        if (file.isDirectory()) {
            if (uri.endsWith("/")) {
                sendListing(ctx, file);
            } else {
                sendRedirect(ctx, uri + '/');
            }
            return;
        }

        //如果不是目录
        //如果构造的文件对象不是文件 则返回 403 异常并返回
        if (!file.isFile()) {
            sendError(ctx, FORBIDDEN);
            return;
        }
        //如果是文件的话 点击超链接会下载
        RandomAccessFile randomAccessFile = null;
        try {
            //通过随机文件读写类以只读的方式打开文件 
            randomAccessFile = new RandomAccessFile(file, "r");//以只读的

        } catch (FileNotFoundException fnfe) {
            //如果打开失败则返回 HTTP 404 的异常错误
            sendError(ctx, HttpResponseStatus.NOT_FOUND);
            return;
        }

        //构造HTTP应答消息
        //获取文件长度
        long fileLength = randomAccessFile.length();
        DefaultHttpResponse response = new DefaultHttpResponse(HTTP_1_1, OK);
        HttpUtil.setContentLength(response, fileLength);
        setContentTypeHeader(response, file);
        //开启长连接传输
        if (HttpUtil.isKeepAlive(request)) {
            response.headers().set(CONNECTION, KEEP_ALIVE);

        }
        
        ctx.write(response);
        ChannelFuture sendFileFuture;
        //通过 Netty 的 ChunkedFile 对象直接将文件分片写入到发送缓冲区中，最后增加 GenericFutureListener，如果发送完成打印 "Transfer complete"
        sendFileFuture = ctx.write(new ChunkedFile(randomAccessFile, 0, fileLength, 8192), ctx.newProgressivePromise());
        sendFileFuture.addListener(new ChannelProgressiveFutureListener() {
            @Override
            public void operationProgressed(ChannelProgressiveFuture future, long progress, long total) throws Exception {
                if (total < 0) {
                    System.err.println("Transfer progress: " + progress);
                } else {
                    System.err.println("Transfer progress: " + progress + "/" + total);
                }
            }

            @Override
            public void operationComplete(ChannelProgressiveFuture future) throws Exception {
                System.out.println("Transfer complete.");
            }
        });

        //如果使用 chunked 编码，最后需要发送一个编码结束的空消息体，将 LastHttpContent的 EMPTY_LAST_ CONTENT 发送到缓冲区中 标识所有的消息体已经发送完成，同时flush刷新到SocketChannel中
        ChannelFuture lastContentFuture = ctx.writeAndFlush(LastHttpContent.EMPTY_LAST_CONTENT);
        if (!HttpUtil.isKeepAlive(request)) {
            lastContentFuture.addListener(ChannelFutureListener.CLOSE);
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {

        cause.printStackTrace();
        if (ctx.channel().isActive()) {
            sendError(ctx, INTERNAL_SERVER_ERROR);

        }

    }

    private static final Pattern INSECURE_URL = Pattern.compile(".*[<>$\"].*");

    private String sanitizeUri(String uri) {
        try {
            //使用 java.net.URLDecoder 对 URL 进行编码 使用 utf-8
            uri = URLDecoder.decode(uri, "utf-8");
        } catch (UnsupportedEncodingException e) {
            try {
                uri = URLDecoder.decode(uri, "ISO-8859-1");
            } catch (UnsupportedEncodingException ex) {
                throw new Error();
            }
        }
        //如果 URI 与允许访问的 URI 一直或者是其子目录，则校验通过 否则返回空
        if (!uri.startsWith(url)) {
            return null;
        }
        if (!uri.startsWith("/")) {
            return null;
        }
        //二次合法性校验，如果校验失败则直接返回空。
        uri = uri.replace('/', File.separatorChar);
        if (uri.contains(File.separator + '.')
                || uri.contains('.' + File.separator) || uri.startsWith(".")
                || uri.endsWith(".") || INSECURE_URL.matcher(uri).matches()) {
            return null;
        }

        //校验成功对文件路径进行拼接
        return System.getProperty("user.dir") + File.separator + uri;
    }

    private static final Pattern ALLOWED_FILE_NAME = Pattern.compile("[A-Za-z0-9][-_A-Za-z0-9\\.]*");

    private static void sendListing(ChannelHandlerContext ctx, File dir) {
        FullHttpResponse response = new DefaultFullHttpResponse(HTTP_1_1, OK);
        //设置消息头为 "text/html charset = utf-8". 由于要将响应结果显示在浏览器上，所以采用html格式 并且编码字符集为 utf-8
        response.headers().set(CONTENT_TYPE, "text/html;charset=UTF-8");
        StringBuilder buf = new StringBuilder();
        String dirPath = dir.getPath();
        //html语法
        buf.append("<!DOCTYPE html>\r\n");
        buf.append("<html><head><title>");
        buf.append(dirPath);
        buf.append(" 目录: ");
        buf.append("</title></head><body>\r\n");
        buf.append("<h3>");
        buf.append(dirPath).append(" 目录：");
        buf.append("</h3>\r\n");
        buf.append("<ul>");
        //超链接格式
        buf.append("<li>链接 : <a href=\"../\">..</a><li>\r\n");
        for (File file : dir.listFiles()) {
            if (file.isHidden() || !file.canRead()) {
                continue;
            }
            String name = file.getName();

            if (!ALLOWED_FILE_NAME.matcher(name).matches()) {
                continue;
            }
            buf.append("<li>链接：<a href=\"");
            buf.append(name);
            buf.append("\">");
            buf.append(name);
            buf.append("</ul></body>\r\n");
        }
        buf.append("</ul></body></html>\r\n");
        //分配对应消息的缓冲对象
        ByteBuf buffer = Unpooled.copiedBuffer(buf, UTF_8);
        //将缓冲区中的响应消息存放到 HTTP 应答消息中，然后释放缓冲区
        response.content().writeBytes(buffer);
        buffer.release();
        //调用writeAndFlush 将响应消息发送到缓冲区并刷新到 SocketChannel 中
        ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);
    }

    private static void sendRedirect(ChannelHandlerContext ctx, String newUri) {
        DefaultHttpResponse response = new DefaultHttpResponse(HTTP_1_1, FOUND);
        response.headers().set(LOCATION, newUri);
        ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);
    }

    private static void sendError(ChannelHandlerContext ctx, HttpResponseStatus status) {
        FullHttpResponse response = new DefaultFullHttpResponse(HTTP_1_1, status, Unpooled.copiedBuffer("Failure: " + status.toString() + "\r\n", UTF_8));

        response.headers().set(CONTENT_TYPE, "text/plain;charset=UTF-8");
        ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);
    }


    private static void setContentTypeHeader(HttpResponse response, File file) {
        MimetypesFileTypeMap mimetypesFileTypeMap = new MimetypesFileTypeMap();
        response.headers().set(CONTENT_TYPE, mimetypesFileTypeMap.getContentType(file.getPath()));
    }
}
```

#### 运行结果

**服务端**

[![image-20221115184425769](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221115184425769.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221115184425769.png)

客户端（浏览器）

[![image-20221115184459734](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221115184459734.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221115184459734.png)

------

### Netty HTTP+JSON 协议栈开发

由于 HTTP 协议的通用性，很多异构系统间的通信交互采用 HTTP 协议，通过 HTTP 协议承载业务数据进行消息交互，例如非常流行的 HTTP + XML 或者 RESTful + JSON。

#### 开发场景介绍

模拟一个简单的用户订购系统。客户端填写订单，通过 HTTP 客户端向服务端发送订购请求，请求消息放在 HTTP 消息体中，**以 JSON 承载**，即**采用 HTTP + JSON 方式进行通信**。HTTP 服务端接收到订购请求后，对订单请求进行修改，然后通过 HTTP+JSON 的方式返回应答消息。双方采用 HTTP 1.1 协议，连接类型为 **CLOSE 方式，即双方交互完成，由 HTTP 服务端主动关闭链路，随后客户端也关闭链路并退出**。

订购请求消息定义如表：

| 字段名称 | 类型     | 备注                                        |
| -------- | -------- | ------------------------------------------- |
| 订购数量 | Int64    | 订购的商品数量                              |
| 客户信息 | Customer | 客户信息，负责 POJO 对象                    |
| 账单地址 | Address  | 账单的地址                                  |
| 寄送方式 | Shipping | 枚举类型如下：普通邮递、宅急送、国际邮递….. |
| 送货地址 | Address  | 送货地址                                    |
| 总价     | float    | 商品总价                                    |

客户信息定义如表：

| 字段名称 | 类型            | 备注               |
| -------- | --------------- | ------------------ |
| 客户ID   | Int64           | 客户端ID，长整型   |
| 姓       | String          | 客户端姓氏，字符串 |
| 名       | String          | 客户名字，字符串   |
| 全名     | List < String > | 客户全称，字符列表 |

地址信息定义如表：

| 字段名称 | 类型   | 备注 |
| -------- | ------ | ---- |
| 街道1    | String |      |
| 街道2    | String |      |
| 城市     | String |      |
| 省份     | String |      |
| 邮政编码 | String |      |
| 国家     | String |      |

邮递方式定义如表：

| 字段名称 | 枚举类型 | 备注 |
| -------- | -------- | ---- |
| 普通邮递 | String   |      |
| 宅急送   | String   |      |
| 国际邮递 | String   |      |
| 国内快递 | String   |      |
| 国际快递 | String   |      |

------

#### HTTP+JSON 协议栈设计

步骤一：构造请求消息并将其编码为 HTTP+JSON 形式，**Netty 的 HTTP 协议栈提供了构造 HTTP 消息的相关接口**，但是**无法将普通的 POJO 对象转换为 HTTP + JSON 的 HTTP 请求消息**，需要**自定义** HTTP+JSON格式的请求消息编码器。

步骤二：利用 Netty 的 HTTP 协议栈，可以支持 HTTP 链路的建立和请求消息的发送，所以不需要额外开发，直接重用 Netty 的能力即可。

步骤三：HTTP 服务端需要将 HTTP + JSON 格式的订购消息解码为订购请求 POJO 对象，同时获取 HTTP 请求头信息。利用 Netty 的 HTTP 协议栈服务端，可以完成 HTTP 请求消息的解码，但是消息体为 JSON Netty 无法支持解码为 POJO 对象，需要在 Netty 协议栈的基础上扩展实现。

步骤四：服务端对订购请求消息处理完成后，重新将其封装为 JSON，通过 HTTP 应答消息体携带给客户端，**Netty 的 HTTP 协议栈不支持将 POJO 对象以 XML 方式发送，需要定制。**

步骤五：HTTP 客户端需要将 HTTP+JSON 格式的应答消息解码为 POJO 对象，同时能够获取应答消息的 HTTP 头信息，**Netty 的协议栈不支持自动的 HTTP + JSON 消息解码**。

**设计思路：**

1. 需要一套通用的、高性能的 JSON 序列化框架，它能够灵活地实现 POJO-JSON 的互相转换；
2. 作为通用的 HTTP+JSON 协议栈，JSON—POJO 对象的映射关系应该非常灵活，支持命名空间和自定义标签；
3. 提供 HTTP+JSON 的请求/响应消息编/解码器，供 HTTP 客户端/服务端能够自动编码/解码使用；
4. 协议栈使用者不需要关心 HTTP+JSON 的编解码，对上层业务零侵入，业务只需要对上层的业务 POJO 对象进行编排。

> 这里的 JSON 序列化框架使用 fastjson2.
>
> ```XML
> <dependency>
>     <groupId>com.alibaba.fastjson2</groupId>
>     <artifactId>fastjson2</artifactId>
>     <version>2.0.17.graal</version>
> </dependency>
> ```

#### HTTP+JSON 编解码框架开发

**POJO 对象定义**

`Order`
```JAVA
@Data
public class Order {

    private long orderNumber;

    private Customer customer;

    private Address billTo;

    private Shipping shipping;

    private Address shipTo;

    private Float total;


}
```
`Customer`
```JAVA
@Data
public class Customer {
    /**
     * id
     */
    private long customerNumber;

    private String firstName;

    private String lastName;

    private List<String> middleNames;


}
```
`Address`
```JAVA
@Data
public class Address {
    private String street1;

    private String street2;

    private String city;

    private String state;

    private String postCode;

    private String country;
}
```
`Shipping`
```JAVA
public enum Shipping {
    STANDARD_MAIL, PRIORITY_MAIL, INTERNATIONAL_MAIL, DOMESTIC_EXPRESS, INTERNATIONAL_EXPRESS
}
OrderFactory
JAVA
public class OrderFactory {
    public static Order create(long orderId){
        Order order = new Order();
        order.setOrderNumber(orderId);
        order.setTotal(9999.999f);
        Address address = new Address();
        address.setCity("成都");
        address.setCountry("中国");
        address.setPostCode("123456");
        address.setState("四川");
        address.setStreet1("成华大道");
        order.setBillTo(address);
        Customer customer = new Customer();
        customer.setCustomerNumber(orderId);
        customer.setFirstName("阿");
        customer.setLastName("黄");
        order.setCustomer(customer);
        order.setShipping(Shipping.INTERNATIONAL_MAIL);
        order.setShipTo(address);
        return order;
    }
}
```

**HTTP + JSON 请求消息编码类**

`AbstractHttpJsonEncoder`
```JAVA
public abstract class AbstractHttpJsonEncoder<T> extends MessageToMessageEncoder<T> {
    final static String CHARSET_NAME = "UTF-8";

    final static Charset UTF_8 = Charset.forName(CHARSET_NAME);

    protected ByteBuf encode0(ChannelHandlerContext ctx, Object body){
        String jsonString = null;
        ByteBuf buffer = null;
        try {
            //调用 fastjson2 json序列化框架将目标对象序列化为 json 字符串
            jsonString = JSON.toJSONString(body);
            buffer = Unpooled.copiedBuffer(jsonString, UTF_8);

        }catch (Exception e){
            e.printStackTrace();
            ctx.fireExceptionCaught(e);
        }
        return buffer;
    }

}
```
`HttpJsonRequestEncoder`
```JAVA
public class HttpJsonRequestEncoder extends AbstractHttpJsonEncoder<HttpJsonRequest>{

    @Override
    protected void encode(ChannelHandlerContext ctx, HttpJsonRequest msg, List<Object> out) throws Exception {
        //调用父类的 encoder0 ，将业务需要发送的 POJO 对象 Order 实例通过 fastjson2 序列化为 json 字符串，随后将其封装为 Netty 的 ByteBuf
        ByteBuf buf = encode0(ctx, msg.getBody());
        FullHttpRequest request = msg.getRequest();
        //如果消息为空则构造新的 HTTP 请求消息并且构造请求消息头
        if (request==null){
            //构造请求消息头 这里设置了默认的HTTP消息头，由于通常情况下 HTTP 通信双方更关注消息体本身 所以这里采用了硬编码的方式。如果想灵活扩展可以使用配置文件的方式
            request = new DefaultFullHttpRequest(HttpVersion.HTTP_1_1, HttpMethod.GET,"/do",buf);
            HttpHeaders headers = request.headers();
            headers.set(HttpHeaderNames.HOST, InetAddress.getLocalHost().getHostAddress());
            headers.set(HttpHeaderNames.ACCEPT_ENCODING,HttpHeaderValues.GZIP.toString()+","+HttpHeaderValues.DEFLATE.toString());
            headers.set(HttpHeaderNames.ACCEPT_CHARSET,"ISO-8859-1,utf-8;q=0.7,*;q=0.7");
            headers.set(HttpHeaderNames.USER_AGENT,"Netty Json Http Client side");
            headers.set(HttpHeaderNames.ACCEPT,"text/html,application/json;q=0.9,*/*;q=0.8");
        }
        //设置消息体的长度 防止TCP 粘包/拆包问题
        HttpUtil.setContentLength(request,buf.readableBytes());
        //将重新构造的 HTTP 请求消息加入到 out中，由后序 Netty 的 HTTP 请求编码器继续对 HTTP 请求消息进行编码。
        out.add(request);
    }
}
```

**HTTP + JSON 请求消息编码类**

`HttpJsonRequest`
```JAVA
@Data
@NoArgsConstructor
public class HttpJsonRequest {

    private FullHttpRequest request;

    private Object body;

    public HttpJsonRequest(FullHttpRequest request, Object body) {
        this.request = request;
        this.body = body;
    }

}
```

它包含两个成员变量 FullHttpRequest 和编码对象 Object，**用于实现和协议栈之间的解耦**。

**HTTP + JSON 请求消息解码类**

`AbstractHttpJsonDecoder`
```JAVA
public abstract class AbstractHttpJsonDecoder<T> extends MessageToMessageDecoder<T> {

    private Class<?> clazz;

    private boolean isPrint;

    private final static String CHARSET_NAME = "UTF-8";

    final static Charset UTF_8 = Charset.forName(CHARSET_NAME);

    protected AbstractHttpJsonDecoder(Class<?> clazz) {
        this(clazz,false);
    }

    protected AbstractHttpJsonDecoder(Class<?> clazz, boolean isPrint) {
        this.clazz = clazz;
        this.isPrint = isPrint;
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }

    protected Object decoder0(ChannelHandlerContext ctx, ByteBuf body){
        Object JSONObject = null;
        
        String jsonString = body.toString(UTF_8);
        if (isPrint){
            System.out.println("The body is : "+jsonString);
        }

        //通过fastjson2 将 json 字符串转换为 POJO 对象。
        JSONObject = JSON.parseObject(jsonString, clazz);
        return JSONObject;
    }
}
```
`HttpJsonRequestDecoder`
```JAVA
public class HttpJsonRequestDecoder extends AbstractHttpJsonDecoder<FullHttpRequest>{
    protected HttpJsonRequestDecoder(Class<?> clazz) {
        super(clazz);
    }

    //对象类型和是否打印HTTP消息体码流的码流开关
    public HttpJsonRequestDecoder(Class<?> clazz, boolean isPrint) {
        super(clazz, isPrint);
    }

    @Override
    protected void decode(ChannelHandlerContext ctx, FullHttpRequest msg, List<Object> out) throws Exception {
        //首先判断 HTTP 消息解码是否成功 如果解码失败 则构造处理结果异常的 HTTP 应答消息返回给客户端。
        if (!msg.decoderResult().isSuccess()){
            sendError(ctx,BAD_REQUEST);
            return;
        }
        //但没有考虑 JSON 消息异常解码失败后的异常封装和处理。后续可以完善 以提升协议栈的健壮性和可靠性。
        HttpJsonRequest request = new HttpJsonRequest(msg,decoder0(ctx,msg.content()));
        //通过 HttpJsonRequest 和 反序列化后的 Order对象构造 HttpJsonRequest 实例。
        out.add(request);

    }

    private void sendError(ChannelHandlerContext ctx, HttpResponseStatus status) {
        DefaultFullHttpResponse resp = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, status, Unpooled.copiedBuffer("Failure: " + status.toString() + "\r\n", CharsetUtil.UTF_8));
        resp.headers().set(CONTENT_TYPE,"text/plain;charset=utf-8");

        ctx.writeAndFlush(resp).addListener(ChannelFutureListener.CLOSE);
    }
}
```

**HTTP + JSON 响应消息编码类**

`HttpJsonResponse`
```JAVA
@Data
@NoArgsConstructor
public class HttpJsonResponse {
    private FullHttpResponse response;
    private Object result;

    public HttpJsonResponse(FullHttpResponse response, Object result) {
        this.response = response;
        this.result = result;
    }
}
```

它包含两个成员变量：FullHttpResponse 和 Object，Object 就是业务需要发送的应答 POJO 对象。

`HttpXmlResponseEncoder`
```JAVA
public class HttpJsonResponseEncoder extends AbstractHttpJsonEncoder<HttpJsonResponse> {
    @Override
    protected void encode(ChannelHandlerContext ctx, HttpJsonResponse msg, List<Object> out) throws Exception {
        ByteBuf body = encode0(ctx, msg.getResult());
        FullHttpResponse response = msg.getResponse();
        //对应答消息进行判断如果为空
        if (response==null){
            //则构造构造一个
            response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.OK,body);
        }else {
            //否则利用已有的应答消息重新复制一个新的 HTTP 应答消息。
            response = new DefaultFullHttpResponse(msg.getResponse().protocolVersion(),msg.getResponse().status(),body);
        }
        /**
         * 因为DefaultFullHttpResponse没有提供动态设置消息体的 content 的接口 所以才要向上述那样实现
         */
        
        //设置消息类型
        response.headers().set(CONTENT_TYPE,"application/json");
        //设置消息体长度
        HttpUtil.setContentLength(response,body.readableBytes());
        //交给 Netty 后序的 HTTP 编码器二次编码
        out.add(response);
    }
}
```

**HTTP + JSON 应答消息解码**

客户端接收到 HTTP + XML 应答消息后，对消息进行解码，获得 HttpJsonResponse 对象。

`HttpJsonResponseDecoder`
```JAVA
public class HttpJsonResponseDecoder extends AbstractHttpJsonDecoder<DefaultFullHttpResponse> {
    protected HttpJsonResponseDecoder(Class<?> clazz) {
        super(clazz);
    }

    protected HttpJsonResponseDecoder(Class<?> clazz, boolean isPrint) {
        super(clazz, isPrint);
    }

    @Override
    protected void decode(ChannelHandlerContext ctx, DefaultFullHttpResponse msg, List<Object> out) throws Exception {
        HttpJsonResponse response = new HttpJsonResponse(msg, decoder0(ctx, msg.content()));
        out.add(response);
    }
}
```

通过 DefaultFullHttpResponse 和 HTTP 应答消息反序列化后的 POJO 对象构造 HttpJsonResponse，并将其添加进解码列表中。

##### HTTP + JSON 客户端开发

客户端的功能如下：

1. 发起 HTTP 连接请求；
2. 构造订购请求消息，将其编码为 JSON，通过 HTTP 协议发送给服务端；
3. 接收 HTTP 服务端的应答消息，将 JSON 反序列化解码为订购消息 POJO 对象；
4. 关闭 HTTP 连接。

`HttpJsonClient`
```JAVA
public class HttpJsonClient {

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;
        new HttpJsonClient().connect(port);
    }

    private void connect(int port) throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    .option(ChannelOption.TCP_NODELAY,true)
                    .channel(NioSocketChannel.class)
                    .handler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            //添加HTTP应答消息解码器 负责将二进制流解码为 HTTP 的应答消息
                            ch.pipeline().addLast("http-decoder",new HttpResponseDecoder());
                            //负责将1个HTTP 请求消息的多个部分合并成一条完整的 HTTP 消息
                            ch.pipeline().addLast("http-aggregator",new HttpObjectAggregator(65536));
                            //我们开发 JSON 解码器 实现 HTTP +  JSON 应答消息的自动解码。
                            ch.pipeline().addLast("json-decoder",new HttpJsonResponseDecoder(Order.class,true));
                            /**
                             * 注意顺序编码时是按照从尾到头的顺序
                             */
                            //HTTP请求消息编码器 将HTTP请求消息编码为 二进制流
                            ch.pipeline().addLast("http-encoder",new HttpRequestEncoder());
                            //消息编码将对象序列化为 JSON 串
                            ch.pipeline().addLast("json-encoder",new HttpJsonRequestEncoder());
                            //业务逻辑实现类
                            ch.pipeline().addLast("jsonClientHandler",new HttpJsonClientHandler());

                        }
                    });

            //发起异步连接操作
            ChannelFuture future = bootstrap.connect("localhost",port).sync();

            //等待客户端链路关闭
            future.channel().closeFuture().sync();
        }finally {
            //优雅退出，释放NIO线程组
            group.shutdownGracefully();
        }
    }
}
```
`HttpJsonClientHandler`
```JAVA
public class HttpJsonClientHandler extends SimpleChannelInboundHandler<HttpJsonResponse> {
    /**
     * 构造HttpJsonRequest请求对象，调用 writeAndFlush 发送
     * @param ctx
     * @throws Exception
     */
    @Override
    public void channelActive(ChannelHandlerContext ctx) throws Exception {
        try {
            HttpJsonRequest request = new HttpJsonRequest(null, OrderFactory.create(123));
            System.out.println(request);
            ctx.writeAndFlush(request);
        }catch (Exception e){
            e.printStackTrace();
            ctx.fireExceptionCaught(e);
        }
    }

    /**
     * 将接收到的应答消息答应（分消息头和消息体）
     * @param ctx           the {@link ChannelHandlerContext} which this {@link SimpleChannelInboundHandler}
     *                      belongs to
     * @param msg           the message to handle
     * @throws Exception
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpJsonResponse msg) throws Exception {
        System.out.println("The client receive response of http header is : "+msg.getResponse().headers().names());
        System.out.println("The client receive response of http body is : "+ msg.getResult());
    }
}
```

##### HTTP + JSON 服务端开发

服务端功能如下：

1. 接收 HTTP 客户端的连接；
2. 接收 HTTP 客户端的 JSON 请求消息，并将其解码为 POJO 对象；
3. 对 POJO 对象进行业务处理，构造应答消息返回；
4. 通过 HTTP + JSON 的格式返回应答消息；
5. 主动关闭 HTTP 连接。


`HttpJsonServer`
```JAVA
public class HttpJsonServer {

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;

        new HttpJsonServer().run(port);
    }



    private void run(int port) throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            /**
                             * 与客户端同理 但编解码器使用顺序不同
                             */
                            ch.pipeline().addLast("http-decoder",new HttpRequestDecoder());
                            ch.pipeline().addLast("http-aggregator",new HttpObjectAggregator(65536));
                            ch.pipeline().addLast("json-decoder",new HttpJsonRequestDecoder(Order.class,true));
                            ch.pipeline().addLast("http-encoder",new HttpResponseEncoder());
                            ch.pipeline().addLast("json-encoder",new HttpJsonResponseEncoder());
                            ch.pipeline().addLast("jsonServerHandler",new HttpJsonServerHandler());
                        }
                    });

            ChannelFuture future = bootstrap.bind(new InetSocketAddress(port)).sync();

            System.out.println("HTTP 订购服务器启动，网址是 ： "+"http://localhost:"+port);
            future.channel().closeFuture().sync();
        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }

    }
}
```
`HttpJsonServerHandler`
```JAVA
public class HttpJsonServerHandler extends SimpleChannelInboundHandler<HttpJsonRequest> {
    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        if (ctx.channel().isActive()){
            sendError(ctx,INTERNAL_SERVER_ERROR);
        }
    }

    private void sendError(ChannelHandlerContext ctx, HttpResponseStatus status) {
        DefaultFullHttpResponse response = new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, status, Unpooled.copiedBuffer("失败: " + status.toString() + "\r\n", CharsetUtil.UTF_8));
        response.headers().set(CONTENT_TYPE,"text/plain;charset=UTF-8");
        ctx.writeAndFlush(response).addListener(ChannelFutureListener.CLOSE);

    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, HttpJsonRequest msg) throws Exception {
        FullHttpRequest request = msg.getRequest();
        Order order = (Order)msg.getBody();
        //对解码后的消息进行打印
        System.out.println("Http server receive request : "+ order);
        //处理order对象
        doBusiness(order);
        //将处理后的order对象构造HttpJsonResponse 响应并发送响应
        ChannelFuture future = ctx.writeAndFlush(new HttpJsonResponse(null, order));
        if (!isKeepAlive(request)){
            future.addListener(new GenericFutureListener<Future<? super Void>>() {
                @Override
                public void operationComplete(Future<? super Void> future) throws Exception {
                    //在发送成功之后关闭 HTTP 链路。
                    ctx.close();
                }
            });
        }
    }

    private void doBusiness(Order order) {
        order.getCustomer().setFirstName("狄");
        order.getCustomer().setLastName("仁杰");
        ArrayList<String> midNames = new ArrayList<>();
        midNames.add("李元芳");
        order.getCustomer().setMiddleNames(midNames);
        Address address = order.getBillTo();
        address.setCity("资阳");
        address.setCountry("中国");
        address.setState("四川");
        address.setPostCode("123123");
        order.setBillTo(address);
        order.setShipTo(address);

    }
}
```

##### HTTP + JSON 协议栈测试

**客户端**

[![image-20221116194953254](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221116194953499.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221116194953499.png)

**服务端**

[![image-20221116195006434](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221116195006434.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221116195006434.png)

结果表明，HTTP + XML 协议栈功能正常，达到了设计预期。

------

## Netty WebSocket 协议开发

### WebSocket 入门

#### HTTP 协议弊端

HTTP 协议的主要弊端如下：

1. HTTP 协议是半双工协议。半双工协议指的是数据可以在客户端和服务端两个方向上传输，但是不能同时传输。**它意味着在同一时刻，只有一个方向上的数据传输；**

2. **HTTP 消息冗长而繁杂**。HTTP 消息包含消息头、消息体、换行符等，通常情况下采用文本方式传输，想比于其他的二进制传输协议，冗长而繁杂。

3. 针对服务器推送的黑客攻击。例如长时间轮询。

   现在，很多网址巍峨了实现消息推送，所用的技术都是轮询。轮询是在特定的时间间隔（如每一秒），由浏览器向服务端发送 HTTP 请求，然后由服务器端返回最新的数据给客户端。这种传统的模式有一个很明显的缺点，即浏览器需要不断地向服务器发出请求，然而 HTTP request 的 header 是非常冗长的，里面包含的可用数据比例可能非常低，这会占用很多的带宽和服务器的资源。

   虽然有一些比较新的技术。这种技术虽然可以达到双向通信，但依然需要发出请求，而在在如 Comet 中，普遍采用的是长连接，这也会大量消耗服务器带宽和资源。

> 为了解决 HTTP 协议效率低下的问题，HTML 5 定义了 **WebSocket 协议**，能更好地节省服务器的资源和带宽并达到式时通信。

------

#### WebSocket

WebSocket 是 HTML 5 开始提供的一种浏览器与服务器之间进行全双工通信的网络技术。

在 WebSocket API 中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成一条快速通道，两者可以**直接**相互传输数据了。**WebSocket 基于 TCP 双向全双工进行消息传递**，在同一时刻，可以发送消息，也可以接收消息，相比 HTTP 的半双工协议，性能大幅提升。

WebSocket 的特点：

- 单一的 TCP 连接，采用全双工模式通信；
- 对代理、防火墙和路由器透明；
- 无头部信息、Cookie 和身份验证；
- 无安全开销；
- 通过 ”ping/pong“ 帧保持链路激活；
- 服务器可以主动传递消息给客户端，不再需要客户端轮询。

> **WebSocket 本质上就是一个 TCP 连接，所以在数据传输的稳定性和数据传输量的大小方面，和轮询技术相比，具有很大的性能优势。**

##### WebSocket 连接建立

[![image-20221117190823057](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117190823057.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117190823057.png)

建立 WebSocket 连接时，需要通过客户端或者浏览器发出握手请求，发送消息如下图所示。

[![image-20221117191111589](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117191111589.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117191111589.png)

为了建立一个 WebSocket 连接，客户端浏览器首先要向服务器发送一个 **HTTP 请求**，这个请求和通常的 HTTP 请求不同，包含了一些附加头信息 **”Upgrade: WebSocket“ 表明这是一个申请协议升级的 HTTP 请求** 。服务器端解析这些附加的头信息，然后生成应答消息返回给客户端，客户端和服务端的 WebSocket 连接就建立起来了，双方可以通过这个连接通道自由地传输信息，并且**这个连接会持续存在知道客户端或服务端的某一方主动关闭连接。**

客户端返回给客户端的应答消息如下：

[![image-20221117192555175](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117192555175.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117192555175.png)

请求消息中的 ”Sec-WebSocket-Key“ 是**随机的**，服务器端会用这些数据来构造一个 **SHA-1 的信息摘要**，把 ”Sec-WebSocket-Key“ 加上一个字符串 ”258EAFA5-E914-47DA-95CA-C5AB0DCB1“。使用 SHA-1 加密，然后进行 BASE-64 编码，**将结果作为 ”Sec-WebSocket-Accept“ 头的值，返回给客户端**。

------

##### 生命周期

握手成功后，服务端和客户端就可以通过 ”messages“ 的方式进行通信了，**一个消息由一个或者多个帧组成，WebSocket 的消息不一定对应一个特定网络层的帧，它可以被分割成多个帧或者被合并。**

**帧都有自己对应的类型，属于同一个消息的多个帧具有相同类型的数据。从广义上讲，数据类型可以是文本类型、二进制数据和控制帧（协议级信令，如信号）。**

WebSocket 声明周期如图：

[![image-20221117193410108](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117193410108.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117193410108.png)

------

##### WebSocket 连接关闭

为关闭 WebSocket 连接，客户端和服务端需要通过一个安全的方法关闭底层 TCP 连接已经 TLS 会话。如果合适，丢弃任何可能已经接收的字节；必要时（比如受到攻击），可以通过任何可用的手段关闭连接。

> 底层的 TCP 连接，在正常请款下，应该首先由服务器关闭。在异常情况下（例如在一个合理的时间周期后没有收到服务器连接的 TCP close），客户端可以发起 TCP close。因此，当服务器被指示关闭 WebSocket 连接时，它应该立即发起一个 TCP Close 操作了客户端应该等待服务器的 TCP Close。

WebSocket 的握手关闭消息带有一个状态码和一个可选的关闭原因，他必须按照协议要求发送一个 Close 控制帧，当对端接收到关闭控制帧指令时，需要主动关闭 WebSocket 连接。

#### Netty WebSocket 协议开发

**服务端**

```JAVA
public class WebSocketServer {
    private final static Logger LOGGER = LogManager.getLogger(WebSocketServer.class);

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;

        new WebSocketServer().run(port);
    }

    private void run(int port) throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();

        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {
                            //添加 HttpServerCodec 将请求和应答消息编码或者解码为 HTTP 消息；
                            ch.pipeline().addLast("http-codec",new HttpServerCodec());
                            //将一个HTTP 消息的多个部分组装成一个
                            ch.pipeline().addLast("aggregator",new HttpObjectAggregator(65536));
                            //ChunkedWriteHandler 向客户端发送HTML5 文件，它主要的作用于支持浏览器和服务端进行 WebSocket 通信。
                            ch.pipeline().addLast("http-chunked",new ChunkedWriteHandler());
                            //自定义业务类
                            ch.pipeline().addLast("handler",new WebSocketHandler());
                        }
                    });
            Channel ch = bootstrap.bind(port).sync().channel();

            LOGGER.info("Web Socket server started at port "+ port + ".");

            LOGGER.info("Open your browser and navigate to http://localhost:"+port+"/");

            ch.closeFuture().sync();
        }finally {
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```
`WebSocketHandler`
```JAVA
public class WebSocketHandler extends SimpleChannelInboundHandler<Object> {

    private static final Logger LOGGER = LogManager.getLogger(WebSocketServer.class);

    private WebSocketServerHandshaker handshaker;

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, Object msg) throws Exception {
        //WebSocket第一次握手是由 HTTP 承载，所以它是一个HTTP 消息
        //传统的 HTTP 接入
        if (msg instanceof FullHttpRequest){
            //执行 handlerHttpRequest 方法来处理 WebSocket 握手请求。
            handleHttpRequest(ctx,(FullHttpRequest) msg);
        }
        //WebSocket 接入
        else if (msg instanceof WebSocketFrame){
            handleWebSocketFrame(ctx,(WebSocketFrame)msg);
        }
    }

    /**
     * 链路建立成功后，客户端通过webSocket通道提交请求消息（帧）给服务端 就由该处理器处理
     * @param ctx
     * @param frame
     */
    private void handleWebSocketFrame(ChannelHandlerContext ctx, WebSocketFrame frame) {
        //首先对控制帧进行判断

        //判断是否是关闭链路的指令
        if (frame instanceof CloseWebSocketFrame){
            handshaker.close(ctx.channel(),((CloseWebSocketFrame) frame).retain());
            return;
        }


        //判断是否是ping消息
        if (frame instanceof PingWebSocketFrame){
            ctx.channel().write(new PongWebSocketFrame(frame.content().retain()));
            return;
        }

        //本例程仅支持文本消息,不支持二进制消息
        if (!(frame instanceof TextWebSocketFrame)){
            throw new UnsupportedOperationException(String.format("%s frame types not supported",frame.getClass().getName()));
        }

        //获取请求消息字符串，对它处理后通过构造新的 TextWebSocketFrame 消息返回给客户端 握手时动态新增了TextWebSocketFrame编码类，所以可以直接发送该对象实例
        //返回响应消息
        String request = ((TextWebSocketFrame) frame).text();
        LOGGER.info(String.format("%s received %s",ctx.channel(),request));

        ctx.channel().write(new TextWebSocketFrame(request+" , 欢迎使用 Netty WebSocket 服务, 现在时刻："+new Date()));
    }

    private void handleHttpRequest(ChannelHandlerContext ctx, FullHttpRequest request) {
        //如果HTTP解码失败，返回HTTP异常 如果HTTP消息头Upgrade字段不是websocket 也返回HTTP异常(code = 400)
        if (!request.decoderResult().isSuccess()||(!"websocket".equals(request.headers().get("Upgrade")))){
            sendHttpResponse(ctx,request,new DefaultFullHttpResponse(HttpVersion.HTTP_1_1, HttpResponseStatus.BAD_REQUEST));
            return;
        }


        //构造握手响应，本机测试
        WebSocketServerHandshakerFactory wsFactory = new WebSocketServerHandshakerFactory(
                "ws://localhost:8080/websocket",null,false
        );

        //新建握手处理类 WebSocketServerHandshaker 通过它构造响应消息返回给客户端，
        //同时将WebSocket相关的编码和解码类动态添加到 ChannelPipeline中，用于WebSocket消息的编解码
        handshaker = wsFactory.newHandshaker(request);
        if (handshaker==null){
            WebSocketServerHandshakerFactory.sendUnsupportedVersionResponse(ctx.channel());
        }else {
            handshaker.handshake(ctx.channel(),request);
        }
    }

    private void sendHttpResponse(ChannelHandlerContext ctx, FullHttpRequest request, DefaultFullHttpResponse response) {
        //返回应答给客户端
        if (response.status().code()!=200){
            ByteBuf buf = Unpooled.copiedBuffer(response.status().toString(), CharsetUtil.UTF_8);
            response.content().writeBytes(buf);
            buf.release();
            HttpUtil.setContentLength(response,response.content().readableBytes());
        }

        //如果是非keep-Alive 关闭连接
        ChannelFuture future = ctx.channel().writeAndFlush(response);
        if (!isKeepAlive(request)||response.status().code()!=200){
            future.addListener(ChannelFutureListener.CLOSE);
        }
    }

    @Override
    public void channelReadComplete(ChannelHandlerContext ctx) throws Exception {
        ctx.flush();
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

**客户端**

```HTML
<html>
<head>
    <meta charset="UTF-8">
    Netty WebSocket 时间服务器
</head>
<br>
<body>
<br>
<script type="text/javascript">
    var socket;
    if (!window.WebSocket) {
        //
    }

    if (window.WebSocket) {
        socket = new WebSocket("ws://localhost:8080/webSocket");
        socket.onmessage = function (event) {
            var ta = document.getElementById("responseText");
            ta.value = "";
            ta.value = event.data;
        };
        socket.onopen = function (event) {
            var ta = document.getElementById("responseText");
            ta.value = "打开 WebSocket 服务正常，浏览器支持 WebSocket！";
        };
        socket.onclose = function (event) {
            var ta = document.getElementById("responseText");
            ta.value = "";
            ta.value = "WebSocket 关闭";
        }
    } else {
        alert("抱歉, 你的浏览器不支持 WebSocket 协议!");
    }

    function send(message) {
        if (!window.WebSocket) {
            return;
        }
        if (socket.readyState == WebSocket.OPEN) {
            socket.send(message);
        } else {
            alert("WebSocket 连接没有建立成功!");
        }


    }

</script>
<form onsubmit="return false;">
    <input type="text" name="message" value="Netty 最佳实践"/>
    <br>
    <input type="button" value="发送 WebSocket 请求消息" onclick="send(this.form.message.value)"/>
    <hr color="blue"/>
    <h3>服务端返回的应答消息</h3>
    <textarea id="responseText" style="width:500px;height: 300px"></textarea>
</form>
</body>
</html>
```

**运行结果**

[![image-20221117201002473](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117201002473.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117201002473.png)

[![image-20221117201012671](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117201012671.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221117201012671.png)

## UDP 协议开发

### 协议简介

**UDP 是无连接的，通信双方不需要建立任何物理链路连接**。在网络中它用于处理数据包，在 OSI 模型中，它处于第四层传输层，即位于 IP 协议的上一层。它不对数据报分组、组装、校验和排序，因此是不可靠的。报文的发送者不知道报文是否被对方正确接收。

UDP 数据报格式有首部和数据两个部分，首部很简单，为 8 个字节，包括如下部分。

1. 源端口：源端口号，2 个字节，最大值 65535；
2. 目的端口：目的端口号，2个字节，最大值 65535；
3. 长度：2个字节，UDP 用户数据报的总长度；
4. 校验和：2个字节，用于校验 UDP 数据报的数字段和包含 UDP 数据报首部的 “伪首部”。其校验方法类似于 IP 分组首部中的首部校验和。

伪首部，又称为伪包头（Pseudo Header）：是指在 TCP 分段或 UDP 的数据报格式中，**在数据包首部前面增加源 IP 地址、目的 IP 地址、 IP 分组的协议字段、TCP 或 UDP 数据报的总长度，共 12 字节，所构成的扩展首部结构**。此伪首部是一个**临时的结构**，它既不向上也不向下传递，**仅仅是为了保证校验套接字的正确性**。

UDP 协议数据报格式示意图如下：

[![image-20221118184618660](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118184618660.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118184618660.png)

UDP 协议的特点如下：

1. **UDP 传送数据前并不与对方建立连接，即 UDP 是无连接的**。在传输数据前，发送方和接收方相互交换信息使双方同步；
2. **UDP 对接收到的数据报不发送确认信号，发送端不知道数据是否被正确接收，也不会重发数据；**
3. UDP 传输数据比 TCP 快速，系统开销也少；UDP 比较简单，UDP 头包含了源端口、目的端口、消息长度和校验和等很少的字节。**由于 UDP 比 TCP 简单、灵活，常用于可靠性要求不高的数据传输**，如视频、图片以及简单文件传输系统（TFTP）等。**TCP 则是用于可靠性要求很高但实时性要求不高的应用**，如文件传输协议 FTP、超文本传输协议 HTTP、简单邮件传输协议 SMTP 等。

### UDP 服务端开发

`ChineseProverbServer`
```JAVA
public class ChineseProverbServer {

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;
        new ChineseProverbServer().run(port);
    }

    private void run(int port) throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap bootstrap = new Bootstrap();
            bootstrap.group(group)
                    //设置支持UDP
                    .channel(NioDatagramChannel.class)
                    //设置支持广播
                    .option(ChannelOption.SO_BROADCAST,true)
                    //因为 UDP 不存在客户端和服务端的实际连接，因此不需要为连接（ChannelPipeline）设置 handler 只需要设置启动辅助类的 handler 即可
                    .handler(new ChineseProverbServerHandler());

            bootstrap.bind(port).sync().channel().closeFuture().await();
        }finally {
            group.shutdownGracefully();
        }
    }
}
```
`ChineseProverbServerHandler`
```JAVA
@ChannelHandler.Sharable
public class ChineseProverbServerHandler extends SimpleChannelInboundHandler<DatagramPacket> {

    //谚语列表
    private static final String[] DICTIONARY = {"好茶不怕细品，好事不怕细论。"
            , "人心隔肚皮，看人看行为。"
            , "碾谷要碾出米来，说话要说出理来。"
            , "宁肯给君子提鞋，不肯和小人同财。"
            , "鸟贵有翼，人贵有志。"};


    private String nextQuote(){
        //这里之所以使用 ThreadLocalRandom 是因为该服务器存在多线程并发的可能 Netty 的线程安全随机类
         int quoteId = ThreadLocalRandom.current().nextInt(DICTIONARY.length);
         return DICTIONARY[quoteId];
    }

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, DatagramPacket msg) throws Exception {
        /**
         * Netty 对 UDP 进行了封装，因此接收到的是 Netty 封装后的 io.netty.channel.socket.DatagramPacket 对象
         */
        String req = msg.content().toString(CharsetUtil.UTF_8); // 将接收到的datagrampakcet 内容转换为 字符串（ByteBuf 的 toString(Charset)方法）
        System.out.println(req);
        if ("谚语字典查询？".equals(req)){
            //发送响应消息 构造 DatagramPacket 应答消息返回 第一个是需要发送的内容，为 ByteBuf 第二个为目的地址包括 IP 端口，可以直接从发送的报文 DatagramPacket 中获取。
            ctx.writeAndFlush(new DatagramPacket(Unpooled.copiedBuffer("谚语查询结果："+nextQuote(),CharsetUtil.UTF_8),msg.sender()));
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

UDP 服务器处理流程图，如图：

[![image-20221118185807303](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118185807303.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118185807303.png)

### UDP 客户端开发

客户端需要主动构建请求消息，向本网段的所有主机广播请求消息，对于服务端而言，接收到广播消息之后会向广播消息的发起方进行定点发送响应。

`ChineseProverbClient`
```JAVA
public class ChineseProverClient {
    public static void main(String[] args) throws InterruptedException {
        int port = 8080;
        new ChineseProverClient().run(port);
    }

    private void run(int port) throws InterruptedException {
        NioEventLoopGroup group = new NioEventLoopGroup();
        try {
            Bootstrap b = new Bootstrap();
            b.group(group)
                    //支持 UDP
                    .channel(NioDatagramChannel.class)
                    //设置支持广播
                    .option(ChannelOption.SO_BROADCAST,true)
                    //不需要建立连接，只需要配置辅助启动类的handler即可
                    .handler(new ChineseProverbClientHandler());
            Channel ch = b.bind(0).sync().channel();
            
            //向网段内的所有及其广播 UDP 消息
            ch.writeAndFlush(new DatagramPacket(Unpooled.copiedBuffer("谚语字典查询？",CharsetUtil.UTF_8)
                    ,new InetSocketAddress("255.255.255.255",port))).sync();
            
            //等待15秒 用于接收服务端的应答消息，然后退出并释放资源。
            if (!ch.closeFuture().await(15000)){
                System.out.println("查询超时");
            }
        }finally {
            group.shutdownGracefully();
        }
    }
}
```
`ChineseProverbClientHandler`
```JAVA
public class ChineseProverbClientHandler extends SimpleChannelInboundHandler<DatagramPacket> {


    /**
     * 打印应答消息
     */
    @Override
    protected void channelRead0(ChannelHandlerContext ctx, DatagramPacket msg) throws Exception {
        String response = msg.content().toString(CharsetUtil.UTF_8);
        if (response.startsWith("谚语查询结果：")){
            System.out.println(response);
            ctx.close();
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

### 运行 UDP 例程

客户端运行两次， 查看运行结果

服务端

[![image-20221118190559355](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118190559355.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118190559355.png)

客户端

[![image-20221118190623972](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118190623972.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118190623972.png)

[![image-20221118190632901](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118190632901.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118190632901.png)

------

## 文件传输

在 NIO 类库提供之前，Java 所有的文件操作分为两大类：

- 基于字节流的 InputStream 和 OutputStream；
- 基于字符流的 Reader 和 Writer。

通过新提供的 NIO 类库的 FileChannel 类库可以方便地以 “管道” 方式对文件进行各种 I/O 操作，相比于传统以流的方式进行的 I/O 操作有了很大的变化和改进。

### FileChannel 简介

Java NIO 中的 File Channel 是一个连接到文件的通道，可以通过这个文件通道读写文件。JDK 1.7 之前 NIO 1.0 的 FileChannel 是同步阻塞的，JDK 1.7 版本对 NIO 类库进行了升级，升级后的 NIO 2.0 提供了异步文件通道 `AsynchronousFileChannel`，它支持**异步非阻塞文件操作**（AIO）。

在使用 FileChannel 之前必须先打开它，FileChannel 无法直接被打开，需要通过使用 InputStream、OutputStream 或 RandomAccessFile 来获取一个 FileChannel 实例。

```JAVA
RandomAccessFile randomAccessFile = new RandomAccessFile(msg, "r");
FileChannel channel = randomAccessFile.getChannel();
```

如果需要从 FileChannel 中读取数据，要申请一个 ByteBuffer，将数据从 FileChannel 中读取到字节缓冲区中。**`read()` 方法返回 int 值表示有多少字节被读取到了字节缓冲区中，如果返回 -1，表示读取到了文件末尾。**

如果需要通过 FileChannel 向文件中写入数据，需要将数据复制或者直接存放到 ByteBuffer 中，然后调用 `FileChannel.write()` 方法进行写入操作。

> 使用完 FileChannel 之后，需要通过 `close()` 方法关闭文件句柄，防止出现句柄泄露。
>
> 可以通过 FileChannel 的 `position(long pos)` 方法设置文件的位置指针，利用该特性可以实现文件的随机读写。

### Netty 文件传输开发

在实际项目中，文件传输通常采用 FTP 或者 HTTP 附件的方式。事实上通过 TCP Socket + File 的方式进行问价你传输也有一定的应用场景，尽管不是主流，但是掌握这种文件传输方式还是比较重要的，特别是针对两个跨主机的 JVM 进程之间进行持久化数据的相互交换。

具体场景如下：

1. Netty 文件服务器启动，绑定 8080 端口作为内部监听端口；
2. 在 CMD 控制台，通过 telnet 和文件服务器建立 TCP 连接；
3. 控制台输入需要下载的文件绝对路径；
4. 文件服务器接收到请求消息进行合法性判断，如果文件存在，则将文件发送给 CMD 控制台；
5. CMD 控制台打印文件名和文件内容。

#### 服务端

`FileServer`
```JAVA
public class FileServer {
    private static final Logger LOGGER = LogManager.getLogger(FileServer.class);

    public static void main(String[] args) throws InterruptedException {
        int port = 8080;

        new FileServer().run(port);
    }

    private void run(int port) throws InterruptedException {
        NioEventLoopGroup bossGroup = new NioEventLoopGroup();
        NioEventLoopGroup workerGroup = new NioEventLoopGroup();
        try {
            ServerBootstrap bootstrap = new ServerBootstrap();
            bootstrap.group(bossGroup,workerGroup)
                    .channel(NioServerSocketChannel.class)
                    .option(ChannelOption.SO_BACKLOG,100)
                    .childHandler(new ChannelInitializer<SocketChannel>() {
                        @Override
                        protected void initChannel(SocketChannel ch) throws Exception {

                            ch.pipeline().addLast(
                                    //将文件内容编码为字符串 方便CMD 控制台可见
                                    new StringEncoder(CharsetUtil.UTF_8),
                                    //根据回车换行符将 TCP 数据报解码 解决了 TCP 粘包/拆包问题
                                    new LineBasedFrameDecoder(1024),
                                    //将数据报解码为字符串
                                    new StringDecoder(CharsetUtil.UTF_8),
                                    new FileServerHandler());
                        }
                    });

            ChannelFuture future = bootstrap.bind(port).sync();
            LOGGER.info("Start file server at port : "+ port);
            future.channel().closeFuture().sync();

        }finally {
            //优雅停机
            bossGroup.shutdownGracefully();
            workerGroup.shutdownGracefully();
        }
    }
}
```
`FileServerHandler`
```JAVA
public class FileServerHandler extends SimpleChannelInboundHandler<String> {
    private static final Logger LOGGER = LogManager.getLogger(FileServerHandler.class);
    private static final String CR = System.getProperty("line.separator");

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, String msg) throws Exception {
        File file = new File(msg);

        //对文件合法性进行校验
        if (file.exists()) {
            if (!file.isFile()) {
                ctx.writeAndFlush("Not a file : " + file + CR);
                return;
            }
            ctx.write(file + " " + file.length() + CR);
            //使用 RandomAccessFile 以只读的方式打开文件
            RandomAccessFile randomAccessFile = new RandomAccessFile(msg, "r");

            /**
             * 通过 Netty 提供的 DefaultFileRegion 进行文件传输
             *
             * 参数如下:
             *
             * FileChannel: 文件通道，用于对文件进行读写操作；
             *
             * Position: 文件操作的指针位置，读取或写入的起始点；
             *
             * Count: 操作的总字节数；
             */
            DefaultFileRegion region = new DefaultFileRegion(
                    randomAccessFile.getChannel(), 0, randomAccessFile.length()
            );
            //构造完之后，直接调用 ctx 的 write 方法实现文件的传输，Netty 底层对文件写入进行了封装，上层应用不需要关心发送的细节。 
            ctx.write(region);
        
            //最后写入回车换行符告知 CMD 控制台：文件传输结束。
            ctx.writeAndFlush(CR);
            randomAccessFile.close();
        } else {
            //文件不存在，构造异常消息返回
            ctx.writeAndFlush("File not found: " + file + CR);
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        cause.printStackTrace();
        ctx.close();
    }
}
```

#### 运行 Netty 文件传输服务例程

启动文件服务器，打开 CMD 控制台，通过 telnet 命令连接到主机。

[![image-20221118200734637](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118200734637.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20221118200734637.png)

通过用例的运行结果可以看出，Netty 的文件服务器功能运行正常，可以实现文件的正确传输。