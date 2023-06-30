# NIO

## NIO 与 BIO的区别

### BIO

Java IO 核心就是流。流只能单向，要么输入，要么输出。只能选其一。

Java IO 就是典型的 BIO 模型，即面向流编程，一个流要么是输入，要么是输出。

BIO 是阻塞的，即在准备读取数据到数据返回期间需要等待内核将数据准备完毕，再通过 IO 阻塞传输到用户空间。

### NIO

Java NIO 与 BIO 不同，NIO 有三个核心组件 Channel、Buffer、Selector，在 NIO 中我们是**面向块(block) \**或是\**缓冲区(buffer)** 编程的。与 Stream 不同的是，Channel 是**双向的**，流只能**单向**所以区分 **In** 和 **Out**，所以 Channel 打开后可以进行**读取、写入或是读写**。

NIO 是非阻塞的，在数据准备阶段不需要等待，需要启动一个线程一直监听内核是否将数据准备完毕，准备完毕后，监听线程通知 IO 线程阻塞读取数据（这一块还是阻塞的）。

> 由于 Channel 是双向的，因此它能更好地反映出底层操作系统的真实情况；在 Linux 系统中，底层操作系统的通道就是双向的。

## NIO 核心组件介绍

**NIO 包含3个核心的组件：**

- Channel(通道)
- Buffer(缓冲区)
- Selector(选择器)

[![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0facff245004ef7adb37a1846c176fb~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a0facff245004ef7adb37a1846c176fb~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

### 缓冲区（Buffer）

在谈到缓冲区，**我们说缓冲区对象本质上是一个数组，但它其实是一个特殊的数组，缓冲区对象内置了一些机制，能够追踪和记录缓冲区的状态变化情况**，如果我们使用 `get()` 方法从缓冲区获取数据或者使用 `put()` 方法把数据写入缓冲区，都会引起缓冲区状态的变化。

> 缓冲区三个重要属性：
>
> - position：**指定下一个将要被写入或者读取的元素索引，它的值由 get()/put() 方法自动更新，在新创建一个 Buffer 对象时，position 被初始化为 0。**
> - limit：**指定还有多少数据需要取出（在从缓冲区写入通道时），或者还有多少空间可以放入数据（在从通道读入缓冲区时）。**
> - capacity：**指定了可以存储在缓冲区的最大数据容量，实际上，它指定了底层数据的大小，或者至少时指定了准许我们使用的底层数组的容量。**
>
> 注：**0<= position <= limit <= capacity**

缓冲区的容量（capacity）是不变的，而位置（position）和上限（limit）以根据实际需要改变。也就是说可以通过改变当前位置和上限来操作缓冲区内任意位置的数据。

通过源码控制初始化时的上限（limit）和容量（capacity）是相同的，而位置（position）则是被初始化为了 0。

```JAVA
public static ByteBuffer allocate(int capacity) {
    if (capacity < 0)
        throw new IllegalArgumentException();
    //调用初始化的方法
    return new HeapByteBuffer(capacity, capacity);
}

HeapByteBuffer(int cap, int lim) {            // package-private
    //传入了一个初始化容量为 cap 的数组
    super(-1, 0, lim, cap, new byte[cap], 0);
    /*
    hb = new byte[cap];
    offset = 0;
    */
```

[![在这里插入图片描述](https://img-blog.csdnimg.cn/72fdf9989461473a9f481e9882a99785.png)](https://img-blog.csdnimg.cn/72fdf9989461473a9f481e9882a99785.png)

------

在 NIO 中，所有的缓冲区类型都继承与抽象类 Buffer，最常用的就是 ByteBuffer，对于 Java 中的基本类型，基本都有一个具体 Buffer 类型与之相对应。

- **缓存区的分配**：可以通过调用静态方法 `allocate()` 来指定缓冲区的容量，其实调用 allocate 方法相当于**创建了一个指定大小的数组，并把它包装为缓冲区对象**。我们也可以**自己创建一个数组通过调用静态方法 `wrap()` 来将其包装为缓冲区对象。**
- **缓冲区分片**：根据现有的缓冲区对象创建一个子缓冲区，**即在现有缓冲区上切出一片作为一个新的缓冲区，但现有的缓冲区与创建的子缓冲区在底层数面上是数据共享的（子缓冲区相当于现有缓冲区的一个视图窗口）。**可以通过调用缓冲区对象的 `slice()` 创建。
- **只读缓冲区**：通过调用缓冲区对象的 `asReadOnlyBuffer()` 方法，将任何**常规缓冲区转换为只读缓冲区**，这个方法返回一个与原缓冲区**完全相同**的缓冲区，并与原缓冲区**共享数据**，只不过它是只读的。如果**原缓冲区的内容发生了变化，只读缓冲区的内容也随之发生变化**。**注意：尝试修改只读缓冲区的内容，则会报 ReadOnlyBufferException 异常；只可以 常规–> 只读 不可以 只读 –> 可写**
- **直接缓冲区**：直接缓冲区是为了加快 I/O 速度，使用一种特殊方式为其分配内存的缓冲区。**该缓冲区会在每一次调用底层操作系统的本机 I/O 操作之前（或之后），尝试避免将缓冲区内容拷贝到一个中间缓冲区拷贝数据。**通过调用静态方法 `allocateDirect()` 方法
- **内存映射**：比常规的基于流或者基于通道的 I/O 快得多。 **内存映射文件 I/O 通过使文件的数据表现为内存数组的内容来完成**。一般来说，**只有文件中实际读取或写入的部分才会映射到内存中**。

------

#### Buffer 数据类型[![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2570db1b8d254311a4287daf087d585a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2570db1b8d254311a4287daf087d585a~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

从类图中可以看到，7中数据类型对应着 7 中子类，这些名字是 Heap 开头子类，数据是存放在 JVM 堆中的。

##### MappedByteBuffer

与 HeapByteBuffer 数据存放在 JVM 堆中的不同，MappedByteBuffer 是将数据存放在**堆以外**的**直接内存**的，可以映射到文件。

通过 java.nio 包和 MappedByteBuffer 允许 Java 程序直接从内存中读取文件内容，通过整个或部分文件映射到内存，由操作系统来处理加载请求和写文件，应用只需要和内存打交道，这使得 IO 很快。

Mmap 内存映射和普通标准 IO 操作的本质区别在于**它并不需要将文件中的数据先拷贝至 OS 的内核 IO 缓冲区**，而是可以**直接将用户进程私有地址空间一块区域与文件对象建立映射关系**，这样程序就好像可以**直接从内存中完成对文件 读/写 操作一样**。

[![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed673b99773a48bbbdc1d99182e01e50~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed673b99773a48bbbdc1d99182e01e50~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

采用Mmap的方式其读/写的效率和性能都非常高，大家熟知的 **RocketMQ** 就使用了该技术。

------

### 选择器（Selector）

NIO 中非阻塞 I/O 采用了基于 Reactor 模式的工作方式， I/O 调用不会被阻塞，而是注册感兴趣的特定 I/O 事件，如可读数据到达、新的套接字连接等，在发生特定事件时，系统再通知我们。NIO 中实现非阻塞 I/O 的核心对象是 Selector，Selector 是注册各种 I/O 事件的地方，而且当那些事情发生时，就是 Selector 告诉我们所发生的事件。

Selector 会不断地轮询注册在上面所有 Channel，如果某个 channel 为读写等事件做好准备，那么就处于就绪状态，通过 Selector 可以不断轮询发现出就绪的 channel，进行后续的 IO 操作。

[![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b35e7714003440ad93c6695772295de2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b35e7714003440ad93c6695772295de2~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

一个Selector能够同时轮询多个channel，这样，一个单独的线程就可以管理多个channel，从而管理多个网络连接，这样就不用为每一个连接都创建一个线程，同时也避免了多线程之间上下文切换导致的开销。（较与 BIO 的优点）。

### 通道（Channel）

通道是一个对象，通过它可以读取和写入数据，当然所有数据都通过 Buffer 对象来处理。我们永远不会将字节直接写入通道，而是将数据写入包含一个或者多个字节的缓冲区。同样也不会直接从通道中读取字节，而是通过数据从通道读入缓冲区，再从缓冲区获取这个字节。

[![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f2e88ae541f420d9f59c433b68cdcb8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7f2e88ae541f420d9f59c433b68cdcb8~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp)

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

## NIO 理解与使用

### Buffer 的常用方法

NIO提供一系列方法来操作Buffer的位置（position）和上限（limit），以及向缓冲区读写数据。

```JAVA
put() //向缓冲区position位置添加数据。并且position往后移动，不能超过limit上限。
get() //读取当前position位置的数据。并且position往后移动，不能超过limit上限。
flip() //将limit置位为当前position位置，再讲position设置为0 切换读写模式。
rewind() //仅将当前position位置设置为0
remaining //获取缓冲区中当前position位置和limit上限之间的元素数（有效的元素数）
hasRemaining() //判断当前缓冲区是否存在有效的元素数
mark() //在当前position位置打一个标记
reset() //将当前position位置恢复到mark标记的位置。
duplicate() //复制缓冲区
```

------

#### 创建缓冲区

```JAVA
//创建一个容量为10的缓冲区
ByteBuffer byteBuffer1 = ByteBuffer.allocate(10);

//使用线程的数组将其包装为缓冲区
ByteBuffer byteBuffer2 = ByteBuffer.wrap("abcdef".getBytes());
```

–

#### 获取/设置缓冲区参数

```JAVA
ByteBuffer byteBuffer = ByteBuffer.allocate(10);

System.out.println("位置："+byteBuffer.position());
System.out.println("上限："+byteBuffer.limit());
System.out.println("容量："+byteBuffer.capacity());
```

[![在这里插入图片描述](https://img-blog.csdnimg.cn/e0edccc64c2742eb93d99a090bb419ea.png)](https://img-blog.csdnimg.cn/e0edccc64c2742eb93d99a090bb419ea.png)

------

#### 添加数据到缓冲区

```JAVA
ByteBuffer byteBuffer = ByteBuffer.allocate(10);

//添加数据到缓冲区
byteBuffer.put("abcde".getBytes());
System.out.println("position位置:"+byteBuffer.position()); //5
System.out.println("limit上限:"+byteBuffer.limit()); //10
System.out.println("capacity容量:"+byteBuffer.capacity()); //10
```

[![在这里插入图片描述](https://img-blog.csdnimg.cn/cbbd61adc5d04be882e4943bd2cabe65.png)](https://img-blog.csdnimg.cn/cbbd61adc5d04be882e4943bd2cabe65.png)

#### rewind 重置缓冲区

rewind 函数将 position 置为 0 位置，并清除标记。

```JAVA
ByteBuffer byteBuffer = ByteBuffer.allocate(10);

//添加数据到缓冲区
byteBuffer.put("abcde".getBytes());

System.out.println("position位置:"+byteBuffer.position()); //5
System.out.println("limit上限:"+byteBuffer.limit()); //10
System.out.println("capacity容量:"+byteBuffer.capacity()); //10

System.out.println("---------------------------------------");

//重置缓冲区
byteBuffer.rewind();

System.out.println("position位置:"+byteBuffer.position()); //0
System.out.println("limit上限:"+byteBuffer.limit()); //10
System.out.println("capacity容量:"+byteBuffer.capacity()); //10
```

[![在这里插入图片描述](https://img-blog.csdnimg.cn/ba388b104135439b87f08419cafb7549.png)](https://img-blog.csdnimg.cn/ba388b104135439b87f08419cafb7549.png)

#### `flip()` 重置缓冲区

flip 函数将 limit 设置为 position 位置，再将 position 置为 0 位置，并清除 mar 标记。

```JAVA
ByteBuffer byteBuffer = ByteBuffer.allocate(10);

//添加数据到缓冲区
byteBuffer.put("abcde".getBytes());

System.out.println("position位置:"+byteBuffer.position()); //5
System.out.println("limit上限:"+byteBuffer.limit()); //10
System.out.println("capacity容量:"+byteBuffer.capacity()); //10

System.out.println("---------------------------------------");

//重置缓冲区
byteBuffer.rewind();

System.out.println("position位置:"+byteBuffer.position()); //0
System.out.println("limit上限:"+byteBuffer.limit()); //5
System.out.println("capacity容量:"+byteBuffer.capacity()); //10
```

[![在这里插入图片描述](https://img-blog.csdnimg.cn/15d5036a46674cd68d1ba24c94af9170.png)](https://img-blog.csdnimg.cn/15d5036a46674cd68d1ba24c94af9170.png)

------

#### `clear()` 清空缓冲区

`clear()` 方法也将 position 置为0，同时将 limit 置为 capacity 的大小，并清除 mark 标记。

```JAVA
//创建一个容量为 10 的缓冲区
ByteBuffer byteBuffer = ByteBuffer.allocate(10);

//设置上限为5
byteBuffer.limit(5);

//添加数据到缓冲区
byteBuffer.put("abcde".getBytes());

System.out.println("position位置:"+byteBuffer.position()); //5
System.out.println("limit上限:"+byteBuffer.limit()); //5
System.out.println("capacity容量:"+byteBuffer.capacity()); //10

System.out.println("---------------------------------------");

//清空缓冲区
byteBuffer.clear();

System.out.println("position位置:"+byteBuffer.position()); //0
System.out.println("limit上限:"+byteBuffer.limit()); //10
System.out.println("capacity容量:"+byteBuffer.capacity()); //10
```

[![在这里插入图片描述](https://img-blog.csdnimg.cn/3b16d8a5eb8e4e73ad0c552f5da62867.png)](https://img-blog.csdnimg.cn/3b16d8a5eb8e4e73ad0c552f5da62867.png)

------

#### 标记和恢复

```JAVA
//创建一个容量为 10 的缓冲区
ByteBuffer byteBuffer = ByteBuffer.allocate(10);

//添加数据到缓冲区
byteBuffer.put("abcde".getBytes());

//打一个标记
byteBuffer.mark();
System.out.println("标记位置:"+byteBuffer.position()); //5

//再添加5个字节
byteBuffer.put("fijkl".getBytes());
System.out.println("标记位置:"+byteBuffer.position()); //10

//将position恢复到mark标记位置
byteBuffer.reset();
System.out.println("恢复标记位置:"+byteBuffer.position()); //5
```

### FileChannel 通道

本地文件 IO 通道，用于读取、写入、映射和操作文件的通道。

```JAVA
//创建读取文件通道
FileChannel fisChannel = new FileInputStream("day05/src/a.txt").getChannel();
//创建写入文件的通道
FileChannel fosChannel = new FileOutputStream("day05/src/b.txt").getChannel();
//创建缓冲区
ByteBuffer buffer = ByteBuffer.allocate(2);
while (fisChannel.read(buffer)!=-1){
    System.out.println("position:"+buffer.position()); //0
    System.out.println("limit:"+buffer.limit());//2
    //锁定缓冲区(为输出buffer数据做准备) limit 指向 position 位置 准备读取 
    buffer.flip();
    //读取
    fosChannel.write(buffer);
    //重置缓冲区(为输入buffer数据做准备) 将 limit指向 cap position 指向 0 重置为初始状态 准备下一次读取
    buffer.clear();
}
//关闭通道
fisChannel.close();
fosChannel.close();
```

[![在这里插入图片描述](https://img-blog.csdnimg.cn/ee6c83e973094678b1ddbdf7855b330c.png)](https://img-blog.csdnimg.cn/ee6c83e973094678b1ddbdf7855b330c.png)

### SocketChannel 通道

使用 SocketChannel 通道上传文件到服务器。

```JAVA
public class SocketChannelDemo {

    public static void main(String[] args) throws IOException {
        //创建通道
        SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("localhost", 8080));

        //创建缓冲区
        ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
        //读取本地文件通过管道读取数据到缓冲区
        FileChannel fisChannel = new FileInputStream("day05/src/a.txt").getChannel();

        while (fisChannel.read(byteBuffer)!=-1){
            byteBuffer.flip();//为写入做准备
            socketChannel.write(byteBuffer);
            byteBuffer.clear(); //清除缓冲区为读取做准备
        }

        //关闭本地通道
        fisChannel.close();

        //读取服务器回写的数据
        byteBuffer.clear();
        int read = socketChannel.read(byteBuffer);
        System.out.println(new String(byteBuffer.array(),0,read));

        //关闭socket通道
        socketChannel.close();
    }

}
```

### ServerSocketChannel 通道

使用 ServerSocketChannel 通道接收文件并保存在服务器。

```JAVA
public class ServerSocketChannelDemo {
    public static void main(String[] args) throws IOException {
        //创建ServerSocketChannel通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();

        //绑定端口号
        serverSocketChannel.bind(new InetSocketAddress(8080));

        //设置为非阻塞
        serverSocketChannel.configureBlocking(false);
        System.out.println("服务器已开启");

        while (true){
            //获取客户端通道，如果有客户端连接直接返回客户端通道，否则直接返回false
            SocketChannel socketChannel = serverSocketChannel.accept();
            //创建本地通道，用于往文件中写数据
            UUID uuid = UUID.randomUUID();
            FileChannel fosChannel=new FileOutputStream("day05/src/"+uuid+".txt").getChannel();
            ByteBuffer buffer = ByteBuffer.allocate(1024);

            while (socketChannel.read(buffer)!=-1){
                buffer.flip(); //准备吧缓冲区数据输出
                //把缓冲区数据写入文件
                fosChannel.write(buffer);
                //清除缓冲区 重置 limit 和 pos
                buffer.clear(); //方便下一次读取
            }
            fosChannel.close();

            //会写数据到客户端
            ByteBuffer resultBuffer = ByteBuffer.wrap("上传文件成功".getBytes());
            //将缓冲区中的数据写入socketChannel通道。
            socketChannel.write(resultBuffer);

            //关闭客户端通道。
            socketChannel.close();
        }
    }
}
```

### NIO Selector 的服务器

```JAVA
public class Server {
    public static void main(String[] args) throws IOException {
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.socket().bind(new InetSocketAddress("localhost", 8080));
        serverSocketChannel.configureBlocking(false);

        Selector selector = Selector.open();
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (true) { //循环阻塞读取就绪事件
            //获取就绪事件的个数
            int readyNum = selector.select();
           	
            if (readyNum == 0) {
                continue;
            }
            //如果就绪事件个数不为0 就取出就绪事件遍历 执行相应逻辑业务
            Set<SelectionKey> selectionKeys = selector.selectedKeys();
            Iterator<SelectionKey> it = selectionKeys.iterator();
            while (it.hasNext()) {
                SelectionKey key = it.next();
                if (key.isAcceptable()) {
                    //接受链接
                } else if (key.isReadable()){
                    //通道可读
                } else if (key.isWritable()) {
                    //通道可写
                }
                it.remove();
            }
        }
    }
}
```

上面的代码可以当作是一个模板。