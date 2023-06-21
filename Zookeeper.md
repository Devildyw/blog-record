# Zookeeper

## 简介

 `ZooKeeper` 一个中心化的服务, 用于维护配置信息, 命名服务(`naming`), 提供分布式同步和集群服务(`group services`)。

 `Zookeeper`是一个开源的分布式应用程序协调服务，是雅虎公司对于Google的`Chubby`的一个开源实现，现已加入Apache开源，其最主要的核心协议ZAB（`Zookeeper`原子广播协议）是著名的`Paxos`算法的衍生`Mult Paxos`的工业实现。

 `Zookeeper`是 `Hadoop` 和 `Hbase` 的重要组件。 `ZooKeeper` 的目标是封装好复杂易出错的关键服务, 暴露简单易用、高效、稳定的接口给用户, 提供 `java` 和 `C` 接口。

 在立项初期，考虑到之前内部很多项目都是使用动物的名字来命名的（例如著名的Pig项目),雅虎的工程师希望给这个项目也取一个动物的名字。时任研究院的首席科学家`RaghuRamakrishnan`开玩笑地说：“在这样下去，我们这儿就变成动物园了！”此话一出，大家纷纷表示就叫动物园管理员吧一一一因为各个以动物命名的分布式组件放在一起，雅虎的整个分布式系统看上去就像一个大型的动物园了，而`Zookeeper`正好要用来进行分布式环境的协调一一于是，`Zookeeper`的名字也就由此诞生了。

## 工作机制

 `Zookeeper`从设计模式角度来理解：是一个基于观察者模式设计的分布式服务管理框架，它负责存储和管理大家都关心的数据，然后接受观察者的注册，一旦这些数据的状态发生变化，`Zookeeper`就将负责通知已经在`Zookeeper`上注册的那些观察者做出相应的反应。

[![image-20220714185103332](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714185103332.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714185103332.png)

## 特点

[![image-20220714185224752](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714185224752.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714185224752.png)

1. `Zookeeper`: 一个领导者（Leader），多个跟随者（Follower）组成的集群。

2. 集群中只要有

   半数以上

   节点存活，

   ```
   Zokeeper
   ```

   集群就能正常服务。所以

   ```
   Zookeeper
   ```

   适合安装奇数台服务器。

   1. 可见，与`Eureka`相比`Zookeeper`更倾向于满足集群节点之间的一致性即CAP中 `Zookeeper`更倾向于`CP`分支

   2. 为什么

      ```
      Zookeeper
      ```

      适合安装奇数台服务器？

      1. **防止由脑裂造成的集群不可用。**当集群节点发生脑裂分成了多个集群如果是奇数个节点的情况下，多个集群中总是会有一个小集群满足可用节点 > 总节点/2，也就是说这个小集群可以在脑裂的情况下重新选举leader，仍然能够对外提供服务；但是如果是偶数个节点的情况下，可能会出现分配十分均匀的小集群，比如说分成了两个集群，两个集群中的节点都是原来节点的一半，此时可用节点只能==总结点/2，不能对外提供服务，虽然这种情况是偶然情况，但是还是建议使用奇数个节点提高可用性。
      2. **在容错能力相同的情况下，奇数台更节省资源。**当容错能力相同的情况下，5台节点组成的集群对外正常提供服务至少需要大于5/2 = 2.5台 = 3台机器正常（反过来说就是挂三台就宕机了），而6台节点组成的集群对外正常提供服务至少需要大于6/3 = 3台机器正常（同样也是挂三台就宕机）。可知5台与6台对于容忍度并没有提升，所以推荐奇数个。

3. 全局数据一致；每个Server保存同一份相同的数据副本，Client无论连接到那个Server，数据都是一致的。

4. 更新请求顺序执行，来自同一个Client的更新请求按其发送顺序依次执行。

5. 数据更新原子性，一次数据更新要么成功，要么失败。

6. 实时性，在一定时间范围内，Client能读到最新数据

   1. 这个时间很短，因为Server保存的数据其实很小，更新/同步起来很快。

## 数据结构

 `Zookeeper`数据模型的结构与Unix文件系统很类似，整体上可用看作是一棵树，每个节点称作一个`ZNode`。每一个`ZNode`默认能够存储`1MB`的数据，每个`ZNode`都可以通过其路径唯一标识（通过一个路径能够找到唯一的一个`ZNode`）。

[![image-20220714192417943](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714192417943.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714192417943.png)

## 应用场景

 提供的服务包括：统一命名服务、统一配置管理、统一集群管理、服务器节点动态上下线、软负载均衡等。

### 统一命名服务

 在分布式环境下，经常需要对应用/服务进行统一命名，便于识别（便于负载均衡）

 例如：`IP`不好记住，而域名容易记住。

 类似的功能`Nginx`的负载均衡，以及其他框架如`Eureka`也有

[![image-20220714192903303](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714192903303.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714192903303.png)

### 统一配置管理

1. 分布式环境下，配置文件同步非常常见。

   1. 一般要求一个集群中，所有节点的配置信息是一致的，比如Kafka集群。
   2. 对配置文件修改后，希望能够快速同步到各个节点上。

2. 配置管理可交由

   ```
   Zookeeper
   ```

   实现

   1. 可将配置信息写入`Zookeeper`上的一个`ZNode`。
   2. 各个客户端服务器监听这个`ZNode`。
   3. 一旦`Znode`中的数据被修改，`Zookeeper`将通知各个客户端服务器。

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714193439722.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714193439722.png)

### 统一集群管理

1. 分布式环境中，实时掌握每个节点的状态是必要的。

   1. 可用根据节点实时状态做出一些调整

2. ```
   Zookeeper
   ```

   可用实时监控节点状态变化

   1. 可将节点信息写入`Zookeeper`上的`ZNode`。
   2. 监听这个`ZNode`可获取它的实时状态变化。

[![image-20220714193710640](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714193710640.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714193710640.png)

### 服务器动态上下线

 客户端能实时洞察到服务器的上下线的变化。

[![image-20220714193816296](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714193816296.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714193816296.png)

### 软负载均衡

在`Zookeeper`中记录每台服务器的访问数，让访问数最少的服务器去处理最新的客户端请求。

[![image-20220714193918850](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714193918850.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714193918850.png)

## 下载与安装

### 下载

官网首页：[`Apache ZooKeeper`](https://zookeeper.apache.org/)

[![image-20220714194148375](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714194148375.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714194148375.png)

点击`Download`选择你要下载的版本

[![image-20220714194308833](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714194308833.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714194308833.png)

版本自行选择

这里我们用云服务器Docker来部署

[Docker](https://devildyw.github.io/2022/05/15/Docker/)中有docker详细的安装教程。

### 单机版

#### 配置`Zookeeper`安装目录

1. 在宿主机中配置`Zookeeper`安装目录：`/home/docker/zookeeper`并且在文件夹中创建data和logs目录

   `mkdir -p /home/docker/zookeeper/data`

   `mkdir -p /home/docker/zookeeper/logs`

2. 授权文件目录：`chmod -R 777 /home/docker/zookeeper/`

#### 安装和部署`Zookeeper`

- 拉去`Zookeeper`镜像: `docker pull zookeeper`默认拉取最新版

- 启动一个临时的`Zookeeper`容器

  ```BASH
  docker run -d -p 2181:2181 --restart always --name=zookeeper --privileged=true zookeeper:latest
  ```

- 进入到刚刚创建的临时容器中

  ```BASH
  docker exec -it 容器id(通过docker ps查到) /bin/bash\
  ```

- 去到根目录`/`，发现根目录中有一个`conf`目录，进入后发现里面有我们需要的`zoo.cfg`文件

- 退出容器

- 将容器中的`Zookeeper`配置文件复制到宿主机的对应位置 `/home/docker/zookeeper`

  ```BASH
  docker cp zookeeperTemp容器的id:/conf /home/docker/zookeeper/
  ```

  > 将容器中有`zoo.cfg`文件的`conf`直接拷贝到容器外我们一开始建立的`zookeeper`目录下，此时`/docker/develop/zookeeper/`目录中应该有三个目录：`conf`，`data`和`logs`

- 这个时候正式地建立我们的`zookeeper`容器，命名为`zookeeper`，别忘了先停掉刚刚创建的容器`zookeeper`然后删掉容器。将我们宿主机中配置的文件目录挂在到容器对应的目录上。

  ```BASH
  docker run -d -p 2181:2181 --restart always --name=zookeeper  --privileged=true \
  -v /home/docker/zookeeper/conf:/conf \
  -v /home/docker/zookeeper/data:/data \
  -v /home/docker/zookeeper/logs:/datalog \
  -e "TZ=Asia/Shanghai" \
  -e "JAVA_OPTS=-server -Xms512m -Xmx512m -Xmn256m -Duser.home=/opt -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m -XX:+AlwaysPreTouch -XX:-UseBiasedLocking" \
  zookeeper:latest
  ```

  > ```PLAINTEXT
  > docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
  > ```
  >
  > OPTIONS说明：
  >
  > - **-a `stdin`:** 指定标准输入输出内容类型，可选 `STDIN/STDOUT/STDERR` 三项；
  > - **-d:** 后台运行容器，并返回容器ID；
  > - **-i:** 以交互模式运行容器，通常与 -t 同时使用；
  > - **-P:** 随机端口映射，容器内部端口**随机**映射到主机的端口
  > - **-p:** 指定端口映射，格式为：**主机(宿主)端口:容器端口**
  > - **`-t:`** 为容器重新分配一个伪输入终端，通常与 -i 同时使用；
  > - **`--name="nginx-lb":`** 为容器指定一个名称；
  > - **`--dns 8.8.8.8:`** 指定容器使用的`DNS`服务器，默认和宿主一致；
  > - **`--dns-search example.com:`** 指定容器`DNS`搜索域名，默认和宿主一致；
  > - **-h “mars”:** 指定容器的hostname；
  > - **-e username=”ritchie”:** 设置环境变量；
  > - **–env-file=[]:** 从指定文件读入环境变量；
  > - **–cpuset=”0-2” or –cpuset=”0,1,2”:** 绑定容器到指定CPU运行；
  > - **-m :**设置容器使用内存最大值；
  > - **–net=”bridge”:** 指定容器的网络连接类型，支持 bridge/host/none/container: 四种类型；
  > - **–link=[]:** 添加链接到另一个容器；
  > - **–expose=[]:** 开放一个端口或一组端口；
  > - **–volume , -v:** 绑定一个卷

#### 启动客户端连接Zookeeper

- 进入容器中

  ```BASH
  docker exec -it 容器id(通过docker ps查到) /bin/bash\
  ```

- 启动客户端

  ```BASH
  bin/zkCli.sh
  ```

- 如果出现以下信息表示`Zookeeper`部署启动成功

  ```BASH
  root@fb3e9f10fb70:/apache-zookeeper-3.7.0-bin# bin/zkCli.sh
  Connecting to localhost:2181
  2022-07-16 12:53:57,208 [myid:] - INFO  [main:Environment@98] - Client environment:zookeeper.version=3.7.0-e3704b390a6697bfdf4b0bef79e3da7a4f6bac4b, built on 2021-03-17 09:46 UTC
  2022-07-16 12:53:57,228 [myid:] - INFO  [main:Environment@98] - Client environment:host.name=fb3e9f10fb70
  2022-07-16 12:53:57,228 [myid:] - INFO  [main:Environment@98] - Client environment:java.version=11.0.13
  2022-07-16 12:53:57,230 [myid:] - INFO  [main:Environment@98] - Client environment:java.vendor=Oracle Corporation
  2022-07-16 12:53:57,230 [myid:] - INFO  [main:Environment@98] - Client environment:java.home=/usr/local/openjdk-11
  2022-07-16 12:53:57,230 [myid:] - INFO  [main:Environment@98] - Client environment:java.class.path=/apache-zookeeper-3.7.0-bin/bin/../zookeeper-server/target/classes:/apache-zookeeper-3.7.0-bin/bin/../build/classes:/apache-zookeeper-3.7.0-bin/bin/../zookeeper-server/target/lib/*.jar:/apache-zookeeper-3.7.0-bin/bin/../build/lib/*.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/zookeeper-prometheus-metrics-3.7.0.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/zookeeper-jute-3.7.0.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/zookeeper-3.7.0.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/snappy-java-1.1.7.7.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/slf4j-log4j12-1.7.30.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/slf4j-api-1.7.30.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/simpleclient_servlet-0.9.0.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/simpleclient_hotspot-0.9.0.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/simpleclient_common-0.9.0.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/simpleclient-0.9.0.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/netty-transport-native-unix-common-4.1.59.Final.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/netty-transport-native-epoll-4.1.59.Final.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/netty-transport-4.1.59.Final.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/netty-resolver-4.1.59.Final.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/netty-handler-4.1.59.Final.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/netty-common-4.1.59.Final.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/netty-codec-4.1.59.Final.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/netty-buffer-4.1.59.Final.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/metrics-core-4.1.12.1.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/log4j-1.2.17.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jline-2.14.6.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jetty-util-ajax-9.4.38.v20210224.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jetty-util-9.4.38.v20210224.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jetty-servlet-9.4.38.v20210224.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jetty-server-9.4.38.v20210224.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jetty-security-9.4.38.v20210224.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jetty-io-9.4.38.v20210224.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jetty-http-9.4.38.v20210224.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/javax.servlet-api-3.1.0.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jackson-databind-2.10.5.1.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jackson-core-2.10.5.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/jackson-annotations-2.10.5.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/commons-cli-1.4.jar:/apache-zookeeper-3.7.0-bin/bin/../lib/audience-annotations-0.12.0.jar:/apache-zookeeper-3.7.0-bin/bin/../zookeeper-*.jar:/apache-zookeeper-3.7.0-bin/bin/../zookeeper-server/src/main/resources/lib/*.jar:/conf:
  2022-07-16 12:53:57,230 [myid:] - INFO  [main:Environment@98] - Client environment:java.library.path=/usr/java/packages/lib:/usr/lib64:/lib64:/lib:/usr/lib
  2022-07-16 12:53:57,230 [myid:] - INFO  [main:Environment@98] - Client environment:java.io.tmpdir=/tmp
  2022-07-16 12:53:57,230 [myid:] - INFO  [main:Environment@98] - Client environment:java.compiler=<NA>
  2022-07-16 12:53:57,230 [myid:] - INFO  [main:Environment@98] - Client environment:os.name=Linux
  2022-07-16 12:53:57,230 [myid:] - INFO  [main:Environment@98] - Client environment:os.arch=amd64
  2022-07-16 12:53:57,231 [myid:] - INFO  [main:Environment@98] - Client environment:os.version=3.10.0-1160.25.1.el7.x86_64
  2022-07-16 12:53:57,231 [myid:] - INFO  [main:Environment@98] - Client environment:user.name=root
  2022-07-16 12:53:57,231 [myid:] - INFO  [main:Environment@98] - Client environment:user.home=/root
  2022-07-16 12:53:57,231 [myid:] - INFO  [main:Environment@98] - Client environment:user.dir=/apache-zookeeper-3.7.0-bin
  2022-07-16 12:53:57,231 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.free=56MB
  2022-07-16 12:53:57,232 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.max=256MB
  2022-07-16 12:53:57,233 [myid:] - INFO  [main:Environment@98] - Client environment:os.memory.total=64MB
  2022-07-16 12:53:57,236 [myid:] - INFO  [main:ZooKeeper@637] - Initiating client connection, connectString=localhost:2181 sessionTimeout=30000 watcher=org.apache.zookeeper.ZooKeeperMain$MyWatcher@7946e1f4
  2022-07-16 12:53:57,240 [myid:] - INFO  [main:X509Util@77] - Setting -D jdk.tls.rejectClientInitiatedRenegotiation=true to disable client-initiated TLS renegotiation
  2022-07-16 12:53:57,250 [myid:] - INFO  [main:ClientCnxnSocket@239] - jute.maxbuffer value is 1048575 Bytes
  2022-07-16 12:53:57,259 [myid:] - INFO  [main:ClientCnxn@1726] - zookeeper.request.timeout value is 0. feature enabled=false
  Welcome to ZooKeeper!
  JLine support is enabled
  2022-07-16 12:53:57,303 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1171] - Opening socket connection to server localhost/127.0.0.1:2181.
  2022-07-16 12:53:57,303 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1173] - SASL config status: Will not attempt to authenticate using SASL (unknown error)
  2022-07-16 12:53:57,317 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1005] - Socket connection established, initiating session, client: /127.0.0.1:51034, server: localhost/127.0.0.1:2181
  2022-07-16 12:53:57,334 [myid:localhost:2181] - INFO  [main-SendThread(localhost:2181):ClientCnxn$SendThread@1438] - Session establishment complete on server localhost/127.0.0.1:2181, session id = 0x100010b79380003, negotiated timeout = 30000
  
  WATCHER::
  
  WatchedEvent state:SyncConnected type:None path:null
  [zk: localhost:2181(CONNECTED) 0] 
  ```

### Zookeeper配置

下面是`Zookeeper`默认配置

```YAML
dataDir=/data  #默认的temp(临时)目录，容易被linux系统定期删除，所以一般不用默认的temp目录。
dataLogDir=/datalog
tickTime=2000 #通信心跳时间，Zookeeper服务器与客户端心跳时间，单位毫秒
initLimit=5 #LF初始通信时限(Leader和follower初始连接时能忍受的最多心跳数即tickTime的数量)
syncLimit=2 #LF(Leader和follower)同步时限(LF的通信时间如果超过syncLimit*tickTime,Leader认为Follower死掉，从服务器列表中删除Follower)
autopurge.snapRetainCount=3
autopurge.purgeInterval=0
maxClientCnxns=60
standaloneEnabled=true
admin.enableServer=true
server.1=localhost:2888:3888;2181 #2181客户端端口号
```

### 集群搭建

#### 搭建前准备

这里我们搭建三台`Zookeeper`组成集群

先像单机版搭建额外两台

1. 配置`Zookeeper`安装目录

2. 启动`Zookeeper02`、`Zookeeper03`两个容器(注意端口映射不要重复)

   ```BASH
   # 启动Zookeeper02容器
   docker run --privileged=true -d --name zookeeper02 --publish 2182:2181 -d zookeeper:latest
   
   # 启动Zookeeper03容器
   docker run --privileged=true -d --name zookeeper03 --publish 2183:2181 -d zookeeper:latest
   ```

3. 将容器中的对应的需要集群配置的文件复制出来

   ```BASH
   docker cp zookeeper02容器的id:/conf /home/docker/zookeeper02/
   docker cp zookeeper02容器的id:/data /home/docker/zookeeper02/
   
   docker cp zookeeper03容器的id:/conf /home/docker/zookeeper03/
   docker cp zookeeper03容器的id:/data /home/docker/zookeeper03/
   ```

#### 集群配置

修改配置文件前先获取各个容器的容器`ip`

`docker`查看容器`ip`的命令

```BASH
docker inspect --format='{{.NetworkSettings.IPAddress}}' 容器名/容器id
```

获取得到三个容器的ip分别为

```TEX
172.17.0.3
172.17.0.4
172.17.0.5
```

**修改配置文件(`zoo.cfg&myid`)**

1. 首先是`myid`文件，他被我们复制到了`zookeeper`文件夹下的data内，修改它，按照顺序`zookeeper01`为1、`zookeeper02`为2…..。（`myid`的值是`zoo.cfg`文件里定义的`server.A`项`A`的值，`Zookeeper` 启动时会读取这个文件，拿到里面的数据与 `zoo.cfg` 里面的配置信息比较从而判断到底是那个server，只是一个**标识作用**。）

2. 修改`zoo.cfg`在期末为将原来默认的`server.1=localhost:2888:3888;2181`根据`myid`的关系修改为

   ```BASH
   server.1=172.17.0.3:2888:3888;2181
   server.2=172.17.0.4:2888:3888;2181
   server.3=172.17.0.5:2888:3888;2181
   ```

3. 重新启动三个容器(这里可以做成一个脚本执行)

   ```BASH
   docker restart zookeeper01
   docker restart zookeeper02
   docker restart zookeeper03
   ```

4. 进入容器中查看

   ```BASH
   docker exec -it 容器id bash
   ```

   ```BASH
   zkServer.sh status #查看zookeeper容器的状态
   ```

   出现如下信息表示搭建集群成功

   [![image-20220717002844751](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220717002844751.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220717002844751.png)

   [![image-20220717002816617](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220717002816617.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220717002816617.png)

   [![image-20220717002933896](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220717002933896.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220717002933896.png)

5. 集群搭建成功



## 选取机制

**SID:**服务器ID。用来唯一标识一台`Zookeeper`集群中的机器，每台机器不能重复，和myid一样。

**`ZXID:`**事务ID。`ZXID`是一个事务ID，用来标识一次服务器状态的变更。在某一时刻，集群中的每台机器的`ZXID`值不一定完全一致，这和`ZooKeeper`服务器对于客户端“更新请求”的处理逻辑有关。(`ZXID`可以理解为服务器状态更新的次数，因为每次更新操作成功后事务id会递增。)

**Epoch:**每个Leader任期的代号。没有Leader时同一轮投票过程中的逻辑时钟值是相同的。每投完一次票这个数据就会增加。

### 第一次启动

我们需要知道`myid`小的会将票投给`myid`大的节点。

[![image-20220719180536531](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220719180536531.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220719180536531.png)

> 如图展示的是myid逐次增加的情况。

1. 服务器1启动，发起一次选举。服务器1投自己一票。此时服务器1票数1票，不够半数以上（3票），选举无法完成，服务器1状态保持LOOKING。
2. 服务器2启动，再发起一次选举，服务器1和服务器2分别投自己一票并交换选票信息：此时服务器1发现服务器2的`myid`比自己目前投票选举的（服务器1）大，更改选票为推举服务器2.此时服务器1票数为0，服务器2票数为2票，依旧没有半数以上，选举无法完成，服务器1，2状态保持LOOKING。
3. 服务器3启动，发起一次选举，此时服务器1和2都会更改选票为服务器3。此投票结果：服务器1为0票，服务器2为0票，服务器3为3票。此时服务器3的票数依旧超过半数，服务器3当选leader。服务器1，2更改状态为FOLLOWING，服务器3更改状态为LEADING。
4. 服务器4启动，发起一次选举。此时服务器1，2，3已经不是LOOKING状态，不会更改选票信息。交换选票信息结果：服务器3为3票，服务器4为1票。此时服务器4服从多数，更改选票信息为服务器3，并更改状态为FOLLOWING。
5. 服务器5重启，同4一样最后状态会编程FOLLOWING。

### 非第一次启动

[![image-20220719183330948](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220719183330948.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220719183330948.png)

1. 当`Zookeeper`集群中的一台服务器出现以下两种情况之一时，就会开始进入Leader选举：

   - 服务器初始化启动
   - 服务器运行期间无法和Leader保持连接（因为网络问题没有检测到Leader）

2. 当一台机器进入Leader选举流程时，当前集群也可能会处于以下两种状态：

   - 集群中本来就已经存在一个Leader。（因为网络问题没有检测到Leader）

     对于第一种已经存在Leader的情况，机器试图去选举Leader时，会被告知当前服务器的Leader信息，对于该机器来说，仅仅需要和Leader机器简历连接，并进行状态同步即可。

   - **集群中确实不存在Leader**

     - 假设`Zookeeper`又5台服务器组成，SID分别为1，2，3，4，5，`ZXID`分别为8、8、8、7、7，并且此时SID为3的服务器是Leader。某一关键时刻，3和5服务器出现故障，因此开始进行Leader选举。`SID`为1，2，4的机器投票情况（`EPOCH`，`ZXID`，`SID`）:（1，8，1），（1，8，2），（1，7，4）。
     - **选举Leader规则：1. EPOCH大的直接胜出；2. EPOCH相同，事务id大的胜出；3. 事务id相同，服务器id大的胜出**

## 客户端命令行操作

### 命令行语法

| 命令                 | 说明                                                         |
| -------------------- | ------------------------------------------------------------ |
| **`czxid`**          | 创建节点的事务`zxid` 每次修改`Zookeeper`状态都会产生一个`Zookeeoer`事务ID.事务ID是`Zookeeper`中所有修改总的次序.每次修改都有唯一的`zxid`,如果`zxid`1小于`zxid`2,那么标识`zxid`1对应的修改在`zxid`2之前发生 |
| **`ctime`**          | `znode`被创建的毫秒数(从1970年开始)                          |
| **`mzxid`**          | `znode`最后更新的事务`zxid`                                  |
| **`mtime`**          | `znode`最后修改的毫秒数(从1970年开始)                        |
| **`pZxid`**          | `znode`最后更新的子节点`zxid`                                |
| **`cversion`**       | `znode`子节点变化号,`znode`子节点修改次数                    |
| **`dataversion`**    | `znode`数据变化号                                            |
| **`aclVersion`**     | `znode`访问控制列表的变化号                                  |
| **`ephemeralOwner`** | 如果是临时节点,这个是`znode`拥有者的`session id`.如果不是临时节点则是0. |
| **`dataLength`**     | `znode`的数据长度                                            |
| **`numChildren`**    | `znode`子节点数量                                            |

### 节点类型

节点类型分为以下两大类:

**持久节点(Persistent)**:客户端和服务器端断开连接后,创建的节点不删除

**短暂节点(Ephemeral)**:客户端和服务器端断开连接后,创建的节点自己删除

[![image-20220720151900500](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720151900500.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720151900500.png)

> 说明: 创建`znode`时设置顺序标识,znode名称后会附加一个值,顺序号是一个单调递增的计数器,由父节点维护
>
> 注意: 在分布式系统中,顺序号可以被用于为所有的事件进行全局排序,这样客户端可以通过顺序号推断事件发生的顺序

#### 持久节点

1. 持久化目录节点

   客户端与`Zookeeper`断开连接后,该节点依旧存在

2. 持久化顺序编号目录节点

   客户端与`Zookeeper`断开连接后,该节点依旧存在,只是`Zookeeper`给该节点名称进行顺序编号

#### 短暂节点

1. 临时目录节点

   客户端与`Zookeeper`断开连接后，该节点被删除

2. 临时顺序编号目录节点

   客户端与`Zookeeper`断开连接后，该节点被`Zookeeper`给该节点名称进行顺序编号。

#### 实际操作

1. 分别创建普通节点（永久节点+不带序号）

   ```BASH
   create 路径 [信息]（类似于创建文件一样需要输入完成的路径）
   [zk: 172.17.0.4:2181(CONNECTED) 3] create /sanguo "diaochan"
   [zk: 172.17.0.4:2181(CONNECTED) 5] create  /sanguo/shuguo "liubei"
   ```

   获取节点信息

   ```BASH
   get -s 路径
   [zk: 172.17.0.4:2181(CONNECTED) 12] get -s /sanguo
   # 获取得到的信息
   diaochan
   cZxid = 0x200000009
   ctime = Wed Jul 20 15:27:45 CST 2022
   mZxid = 0x200000009
   mtime = Wed Jul 20 15:27:45 CST 2022
   pZxid = 0x20000000a
   cversion = 1
   dataVersion = 0
   aclVersion = 0
   ephemeralOwner = 0x0
   dataLength = 8
   numChildren = 1
   ```

2. 分别创建普通节点（永久节点+带序号）

   ```BASH
   create -s 路径 [信息]
   [zk: 172.17.0.4:2181(CONNECTED) 4] create -s /sanguo
   Created /sanguo0000000001
   ```

3. 创建临时节点（临时节点+不带序号）

   ```BASH
   create -e 路径 [信息]
   [zk: 172.17.0.4:2181(CONNECTED) 21] create -e /sanguo/zhongguo
   Created /sanguo/zhongguo
   ```

   临时节点在客户端与服务器断开连接后被删除，永久节点则反不会。

4. 修改节点数据值

   ```BASH
   [zk: 172.17.0.4:2181(CONNECTED) 10] set /sanguo diaochan1
   ```

   这样就可以修改或者添加节点的值了。

### 监听器原理

#### 监听器原理详解

1. 首先有一个main()线程
2. 在main线程中创建`Zookeeper`客户端，这时就会创建两个线程，一个负责网络连接(connect)，一个负责监听(listener)。
3. 通过connect线程将注册的监听事件发送给`Zookeeper`。
4. 在`Zookeeper`的注册监听器列表中将注册的监听事件添加到列表中。
5. `Zookeeper`监听到有数据或路径发生变化，就会将这个消息发送给listener线程。
6. listener线程内部调用process()方法

[![image-20220720170334171](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720170334171.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720170334171.png)

#### 节点的值变化监听

1. 在`Zookeeper01`主机上注册监听`/sanguo`节点的数据变化。

   ```BASH
   zk: 172.17.0.4:2181(CONNECTED) 16] get -w /sanguo
   ```

2. 在`Zookeeper02`主机上修改`/sanguo`节点的数据

   ```BASH
   [zk: 172.17.0.4:2181(CONNECTED) 1] set /sanguo "xisi"
   ```

3. 观察`Zookeeper01`主机收到数据变化的监听

   [![image-20220720172417291](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720172417291.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720172417291.png)

   > 注意: 在`Zookeeper02`再多次修改`/sanguo`的值,`Zookeeper01`上不会再收到监听.因为注册一次,只能监听一次.像再次监听,需要再次监听.

#### 节点的子节点变化监听

1. 在`Zookeeper01`主机上注册监听`/sanguo`节点的子节点变化。

   ```BASH
   [zk: 172.17.0.4:2181(CONNECTED) 17] ls -w /sanguo
   ```

2. 在`Zookeeper02`主机上`/sanguo`节点上创建子节点

   ```BASH
   [zk: 172.17.0.4:2181(CONNECTED) 2] create /sanguo/jin "simayi"
   ```

3. 观察`Zookeeper01`主机收到子节点变化的监听

   [![image-20220720173200076](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720173200076.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720173200076.png)

   > 注意: 节点的路径变化,也是注册一次,生效一次,想多次生效,就需要多次注册.

### 节点的删除与查看

#### 删除节点

```BASH
[zk: 172.17.0.4:2181(CONNECTED) 18] delete /sanguo/jin
```

#### 递归删除节点

```BASH
[zk: 172.17.0.4:2181(CONNECTED) 19] deleteall /sanguo/shuguo
```

如果节点下面有许多子节点,就不能够直接通过`delete`删除该节点,而是应该使用`deleteall`

#### 查看节点状态

```BASH
[zk: 172.17.0.4:2181(CONNECTED) 20] stat /sanguo
# 节点状态信息
cZxid = 0x200000009
ctime = Wed Jul 20 15:27:45 CST 2022
mZxid = 0x200000020
mtime = Wed Jul 20 17:21:49 CST 2022
pZxid = 0x200000023
cversion = 8
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 2
```

## 客户端API操作

**前提:**保证`Zookeeper`集群服务器启动。

### 环境搭建

1. 创建工程`zookeeper01`

2. 添加`pom.xml`文件

   ```XML
   <dependencies>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.13.2</version>
       </dependency>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <version>1.18.24</version>
       </dependency>
       <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
       <dependency>
           <groupId>org.apache.zookeeper</groupId>
           <artifactId>zookeeper</artifactId>
           <version>3.8.0</version>
       </dependency>
       <dependency>
           <groupId>io.netty</groupId>
           <artifactId>netty-common</artifactId>
           <version>4.1.77.Final</version>
       </dependency>
   </dependencies>
   ```

3. 创建包`com.dyw.zookeeper`

4. 在包下创建类 名称为`zkClient`

#### 创建Zookeeper客户端

```JAVA
String connectString = "36.137.128.27:2182,36.137.128.27:2181,36.137.128.27:2183";
   //会话过期时间
   int sessionTimeout = 2000;

   private ZooKeeper zkClient;

   @Before
   public void init() throws IOException {
       //这里我们创建Zookeeper客户端时可以自己new 一个Watcher
       zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
           @Override
           public void process(WatchedEvent event) {

           }
       });
   }
```

#### 创建子节点

```JAVA
@Test
public void create() throws InterruptedException, KeeperException {
    String nodeCreated = zkClient.create("/devildyw", "ss.avi".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.PERSISTENT);
}
```

> 注意：一定要在`init()`方法上加上@Before注解 这样子才可以在调用create()方法前进行连接的初始化。

测试服务器端中`Zookeeper`节点的变化

[![image-20220720185155078](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720185155078.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720185155078.png)

#### 获取子节点并监听节点变化

```JAVA
@Test
public void getChildren() throws InterruptedException, KeeperException {
    //这里方法中的true代表着使用Watcher这个Watcher就是前面创建Zookeeper客户端时new的Watcher 也可以自定义
    List<String> children = zkClient.getChildren("/", true);
    for (String child : children) {
        System.out.println(child);
    }
}
```

服务器中`"/"`下的节点

[![image-20220720185435045](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720185435045.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720185435045.png)

通过客户端`API`获取得到的`"/"`下的节点

[![image-20220720185447846](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720185447846.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720185447846.png)

> 注意：这个方法中我们只创建了一次监听器，用于监听给定路径的节点或在节点下创建/删除子节点的成功操作，但是一次之后监听器就会失效，如果要持续监听就需要再次注册。
>
> 可以采用循环注册监听器来解决。

#### 判断`Znode`是否存在

```JAVA
@Test
public void exist() throws Exception {
    Stat exists = zkClient.exists("/devildyw", false);
    System.out.println(exists==null?"not exist":"exist");
}
```

[![image-20220720192241351](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720192241351.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720192241351.png)

## 客户端向服务端写数据流程

### 写流程之写入请求直接发送给Leader节点

[![image-20220720192619337](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720192619337.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720192619337.png)

上述展示的是`Zookeeper`客户端向`Zookeeper`集群集群`Leader`节点发送写请求，`Leader`写完后，然后`Leader`节点会发送写请求给`Follower`，当`Follower`完成写请求后返回一个ACK信息表示数据已接收。如果集群中有半数以上的节点完成了写请求就会响应给客户端一个`ACK`，表示写请求成功，后续Leader会继续向其他的Follower重复写请求和返回`ACK(Follower)`

> `ACK (Acknowledge character）`即是确认字符，在数据通信中，接收站发给发送站的一种传输类控制字符。表示发来的数据已确认接收无误。

### 写流程之写入请求发送给follower节点

[![image-20220720192630844](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720192630844.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220720192630844.png)

如果`Zookeeper`客户端向`Zookeeper`集群中的Follower节点发送写请求，那么该`Follower`节点会将写请求直接转发到`Leader`节点上，**再执行写入请求发送给`Leader`节点的流程**，如果超过半数了，`Leader`就会响应`ACK`到一开始接收到写入请求的`Follower`节点，再通过这个节点将`ACK`返回到客户端，剩余`Follower`节点继续接收执行`Leader`的写入请求。

## 服务器动态上下线

### 需求

某分布式系统中，主节点可以有多台，可以动态上下线，任意一台客户端都能实时感知到主节点服务器的上下线。

### 需求分析

[![image-20220721002217841](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721002217841.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721002217841.png)

通过`Zookeeper`集群去管理服务器的动态上下线，主要是各个服务器通过`Zookeeper`客户端去再`Zookeeper`集群中创建节点，当下线是将节点删除，而客户端则是通过`Zookeeper`客户端去监听服务器的上下线，执行响应的业务。

### 具体实现

1. 首先在集群上创建`/servers`节点

   ```BASH
   [zk: localhost:2181(CONNECTED) 14] create /servers "servers"
   ```

2. 创建包`com.dyw.case1`

3. 创建类`DistributeServer` 服务器端向 `Zookeeper `注册代码

   ```JAVA
   package com.devildyw.case1;
   
   import org.apache.zookeeper.*;
   
   import java.io.IOException;
   
   /**
    * @author Devil
    * @since 2022-07-21-13:00
    *
    * 分布式服务器动态上下线中的服务器角色
    */
   public class DistributeServer {
       //注意配置连接多台Zookeeper服务器 服务器之间不能留有空格
       String connectString = "36.137.128.27:2182,36.137.128.27:2181,36.137.128.27:2183";
       //会话过期时间
       int sessionTimeout = 2000;
   
       private ZooKeeper zkClient;
       public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
           DistributeServer server = new DistributeServer();
           //1. 获取zk连接
           server.getConnect();
           //2 注册服务器zk集群
           server.register(args[0]);
           //3.启动业户逻辑（这里为了演示 使用线程休眠代替）
           server.business();
       }
   
       private void business() throws InterruptedException {
           Thread.sleep(Long.MAX_VALUE);
       }
   
       private void register(String hostname) throws InterruptedException, KeeperException {
           String create = zkClient.create("/servers/"+hostname, hostname.getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
           System.out.println(hostname+"is online");
       }
   
       private void getConnect() throws IOException {
           zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
               @Override
               public void process(WatchedEvent event) {
   
               }
           });
       }
   }
   ```

4. 客户端代码`DistributeClient`

   ```JAVA
   package com.devildyw.case1;
   
   import org.apache.zookeeper.KeeperException;
   import org.apache.zookeeper.WatchedEvent;
   import org.apache.zookeeper.Watcher;
   import org.apache.zookeeper.ZooKeeper;
   
   import java.io.IOException;
   import java.util.ArrayList;
   import java.util.List;
   
   /**
    * @author Devil
    * @since 2022-07-21-13:10
    *
    * 分布式服务器动态上下线中的客户端角色
    */
   public class DistributeClient {
       //注意配置连接多台Zookeeper服务器 服务器之间不能留有空格
       String connectString = "36.137.128.27:2182,36.137.128.27:2181,36.137.128.27:2183";
       //会话过期时间
       int sessionTimeout = 2000;
   
       ZooKeeper zkClient;
       public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
           DistributeClient client = new DistributeClient();
           //1. 获取ZK连接
           client.getConnect();
           //2.监听/servers下面节点的增加或删除
           client.getServerList();
           //3.业务逻辑(线程休眠)
           client.business();
       }
   
       private void business() throws InterruptedException {
           Thread.sleep(Long.MAX_VALUE);
       }
   
       private void getServerList() throws InterruptedException, KeeperException {
           List<String> children = zkClient.getChildren("/servers", true); //这里监听器位置参数设置为true代表使用初始化Zookeeper客户端时的Watcher
           ArrayList<String> servers = new ArrayList<>();
           for (String child : children) {
               //取出节点中的数据信息
               byte[] data = zkClient.getData("/servers/" + child, false, null);
               //封装到集合中
               servers.add(new String(data));
           }
   
           //打印集合信息
           System.out.println(servers);
   
       }
   
       private void getConnect() throws IOException {
           zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
               @Override
               public void process(WatchedEvent event) {
                   try {
                       //循环注册监听 当触发了监听器后 在监听器业务逻辑中再次注册监听器
                       getServerList();
                   } catch (InterruptedException e) {
                       throw new RuntimeException(e);
                   } catch (KeeperException e) {
                       throw new RuntimeException(e);
                   }
               }
           });
       }
   }
   ```

5. 运行/测试

   先启动客户端监听`/servers`下的节点变化,然后启动服务器.

   服务器端的参数传入用到了命令行参数,我们这里可以`Idea`中配置

   [![image-20220721134945087](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721134945087.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721134945087.png)

[![image-20220721135008120](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721135008120.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721135008120.png)

启动服务器创建节点,客户端监测到打印服务器信息列表.

## **ZooKeeper** **分布式锁案例**

 什么叫分布式锁?

 比如说”进程1”在使用该资源的时候，会先去获得锁，”进程1”获得锁以后会对该资源保持独占，这样其他进程就无法访问该资源,”进程1”用完该资源以后就将锁释放掉，让其他进程来获得锁，那么通过这个锁机制，我们就能保证分布式系统中多个进程能够有序的访问该临界资源。那么我们把这个分布式环境下的这个锁叫做分布式锁。

### 分布式锁案例

[![image-20220721135958109](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721135958109.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721135958109.png)

多个客户端在`/locks`的路径下创建节点(临时有序的节点)，如果节点序号小优先获得锁处理业务，其他序号大的点，监听其序号前一个节点，如果前一个节点处理完业务后，锁会被释放且前一个节点会被删除，这时后一个节点因为是在监听前一个节点的所以此时它回去获得锁，处理业务，依次类推。

### **原生** **Zookeeper** 实现分布式锁案例

1. 创建工程`zookeeper-02-Distribute-Lock`

2. 导入依赖

   ```XML
   <dependencies>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
           <version>4.13.2</version>
       </dependency>
       <dependency>
           <groupId>org.projectlombok</groupId>
           <artifactId>lombok</artifactId>
           <version>1.18.24</version>
       </dependency>
       <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
       <dependency>
           <groupId>org.apache.zookeeper</groupId>
           <artifactId>zookeeper</artifactId>
           <version>3.8.0</version>
       </dependency>
       <dependency>
           <groupId>io.netty</groupId>
           <artifactId>netty-common</artifactId>
           <version>4.1.77.Final</version>
       </dependency>
   </dependencies>
   ```

3. 创建包`com.dyw.distributeLock`

4. 创建类`DistributeLock`

   ```JAVA
   package top.devildyw.distributeLock;
   
   import org.apache.zookeeper.*;
   import org.apache.zookeeper.data.Stat;
   
   import java.io.IOException;
   import java.util.Collections;
   import java.util.List;
   import java.util.concurrent.CountDownLatch;
   
   /**
    * @author Devil
    * @since 2022-07-21-14:13
    */
   public class DistributeLock {
   
       //注意配置连接多台Zookeeper服务器 服务器之间不能留有空格
       String connectString = "36.137.128.27:2182,36.137.128.27:2181,36.137.128.27:2183";
       //会话过期时间
       int sessionTimeout = 2000;
       ZooKeeper zkClient;
   
       private String waitPath;
   
       private CountDownLatch connectLatch = new CountDownLatch(1);
       private CountDownLatch waitLatch = new CountDownLatch(1);
       private String currentMode;
   
       public DistributeLock() throws IOException, InterruptedException, KeeperException {
           //获取连接
           zkClient = new ZooKeeper(connectString, sessionTimeout, new Watcher() {
               @Override
               public void process(WatchedEvent event) {
                   //connectLatch 如果连接上了zk 可以释放
                   if (event.getState()==Event.KeeperState.SyncConnected){
                       connectLatch.countDown();
                   }
                   //waitLatch 需要释放
                   if (event.getType()== Event.EventType.NodeDeleted && event.getPath().equals(waitPath)){
                       waitLatch.countDown();
                   }
               }
           });
           //等待zk正常连接后,才往下走程序 可以提高代码健壮性
           connectLatch.await();
           //判断根节点locks是否存在
           Stat exists = zkClient.exists("/locks", false);
           //如果不存在 需要创建
           if (exists==null){
               //创建一个根节点
               zkClient.create("/locks","locks".getBytes(), ZooDefs.Ids.OPEN_ACL_UNSAFE,CreateMode.PERSISTENT);
           }
   
       }
   
       //对zk加锁
       public void zkLock() {
           //创建对应的临时带序号节点
           try {
               currentMode = zkClient.create("/locks/" + "seq-", null, ZooDefs.Ids.OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL_SEQUENTIAL);
   
               //wait一小会,让结果更清晰
   //            Thread.sleep(10);
               //判断创建的节点是否是最小的序号节点，如果是获取到锁；如果不是，监听序号前一个节点
               //判断是否是最下序号节点
               List<String> children = zkClient.getChildren("/locks", false);
               //如果children只有一个值，那就直接获取锁；如果有多个节点，需要判断，谁最小
               if (children.size()==1){
                   return;
               }else{
                   Collections.sort(children);
   
                   //获取节点名称
                   String thisNode = currentMode.substring("/locks/".length());
                   //通过seq-0000000获取该节点咋children集合的位置
                   int index = children.indexOf(thisNode);
                   //判断
                   if (index==-1){
                       System.out.println("数据异常");
                   }else if (index==0){
                       //第一个节点可以获取锁了
                       return;
                   }else{
                       //需要监听它前一个节点的变化
                       waitPath = "/locks/"+children.get(index-1);
                       zkClient.getData(waitPath,true,null);
                       //等待监听 知道上一个节点被删除后才释放
                       waitLatch.await();
   
                       return;
                   }
               }
           } catch (KeeperException e) {
               throw new RuntimeException(e);
           } catch (InterruptedException e) {
               throw new RuntimeException(e);
           }
       }
   
       //对zk解锁
       public void zkUnlock() {
           //删除节点 后面的版本号根据实际要求给 这里随便给了个-1
   
           try {
               zkClient.delete(currentMode,-1);
           } catch (InterruptedException e) {
               throw new RuntimeException(e);
           } catch (KeeperException e) {
               throw new RuntimeException(e);
           }
       }
   }
   ```

5. 测试

   ```JAVA
   package top.devildyw.distributeLock;
   
   import org.apache.zookeeper.KeeperException;
   
   import java.io.IOException;
   
   /**
    * @author Devil
    * @since 2022-07-21-14:46
    */
   public class DistributeLockTest {
       public static void main(String[] args) throws IOException, InterruptedException, KeeperException {
           DistributeLock lock1 = new DistributeLock();
           DistributeLock lock2 = new DistributeLock();
           //开启多线程测试
           new Thread(new Runnable() {
               @Override
               public void run() {
                   try {
                       //获取锁
                       lock1.zkLock();
                       System.out.println("线程一获取到锁");
                       Thread.sleep(5 * 1000);
                       //释放锁
                       lock1.zkUnlock();
                       System.out.println("线程一释放锁");
                   } catch (InterruptedException e) {
                       throw new RuntimeException(e);
                   }
               }
           }).start();
   
           //开启多线程测试
           new Thread(new Runnable() {
               @Override
               public void run() {
                   try {
                       //获取锁
                       lock2.zkLock();
                       System.out.println("线程二获取到锁");
                       Thread.sleep(5 * 1000);
                       //释放锁
                       lock2.zkUnlock();
                       System.out.println("线程二释放锁");
                   } catch (InterruptedException e) {
                       throw new RuntimeException(e);
                   }
               }
           }).start();
       }
   }
   ```

   控制台

   ```TEX
   线程二获取到锁
   
   线程二释放锁
   
   线程一获取到锁
   
   线程一释放锁
   ```

### **Curator** 框架实现分布式锁案例

#### 原生的Java API开发存在的问题

1. 会话连接是异步的,需要自己去处理。比如使用`CountDownLatch`
2. Watch需要重复注册，不然就不能生效
3. 开发的复杂性还是比较高的
4. 不支持多节点删除和创建。需要自己去递归。

#### **Curator** **是一个专门解决分布式锁的框架，解决了原生** **JavaAPI** 开发分布式遇到的问题。

> Apache Curator is a Java/JVM client library for [Apache ZooKeeper](https://zookeeper.apache.org/), a distributed coordination service. It includes a highlevel API framework and utilities to make using Apache ZooKeeper much easier and more reliable. It also includes recipes for common use cases and extensions such as service discovery and a Java 8 asynchronous DSL.
>
> 翻译：Apache Curator 是 Apache ZooKeeper 的 Java/JVM 客户端库，Apache ZooKeeper 是一种分布式协调服务。它包括一个高级 API 框架和实用程序，使 Apache ZooKeeper 的使用更加轻松和可靠。它还包括常见用例和扩展的秘诀，例如服务发现和 Java 8 异步 DSL。

> 详情请查看官方文档：[Apache Curator](https://curator.apache.org/)

#### 案例实操

1. 创建工程`zookeeper-04-Distrubuted-Lock-Curator`

2. 添加依赖

   ```XML
   <dependencies>
          <dependency>
              <groupId>junit</groupId>
              <artifactId>junit</artifactId>
              <version>4.13.2</version>
          </dependency>
          <dependency>
              <groupId>org.projectlombok</groupId>
              <artifactId>lombok</artifactId>
              <version>1.18.24</version>
          </dependency>
          <!-- https://mvnrepository.com/artifact/org.apache.zookeeper/zookeeper -->
          <dependency>
              <groupId>org.apache.zookeeper</groupId>
              <artifactId>zookeeper</artifactId>
              <version>3.8.0</version>
          </dependency>
          <dependency>
              <groupId>io.netty</groupId>
              <artifactId>netty-common</artifactId>
              <version>4.1.77.Final</version>
          </dependency>
          <dependency>
              <groupId>com.google.guava</groupId>
              <artifactId>guava</artifactId>
              <version>31.1-jre</version>
          </dependency>
          <dependency>
              <groupId>org.apache.curator</groupId>
              <artifactId>curator-client</artifactId>
              <version>5.2.1</version>
          </dependency>
          <dependency>
              <groupId>org.apache.curator</groupId>
              <artifactId>curator-framework</artifactId>
              <version>5.2.1</version>
          </dependency>
          <dependency>
              <groupId>org.apache.curator</groupId>
              <artifactId>curator-recipes</artifactId>
              <version>5.2.1</version>
          </dependency>
      </dependencies>
   ```

3. 创建包`com.dyw.DistributedLockCurator`

4. 创建测试类 测试框架`API` `CuratorLockTest`

   ```JAVA
   package com.dyw.DistributedLockCurator;
   
   import org.apache.curator.framework.CuratorFramework;
   import org.apache.curator.framework.CuratorFrameworkFactory;
   import org.apache.curator.framework.recipes.locks.InterProcessMutex;
   import org.apache.curator.retry.ExponentialBackoffRetry;
   
   /**
    * @author Devil
    * @since 2022-07-21-15:20
    */
   public class CuratorLockTest {
       public static void main(String[] args) {
           //创建分布式锁1
           InterProcessMutex lock1 = new InterProcessMutex(getCuratorFramework(), "/locks");
           //创建分布式锁2
           InterProcessMutex lock2 = new InterProcessMutex(getCuratorFramework(), "/locks");
   
           new Thread(new Runnable() {
               @Override
               public void run() {
                   try {
                       //获取锁
                       lock1.acquire();
                       System.out.println("线程一 获取到锁");
                       //这里再次获取锁是为了验证该框架的锁是可重入锁
                       lock1.acquire();
                       System.out.println("线程一 再次获取到锁");
   
                       Thread.sleep(5*1000);
                       //释放锁
                       lock1.release();
                       System.out.println("线程一 释放锁");
                       //再次释放锁
                       lock1.release();
                       System.out.println("线程一 再次释放锁");
                   } catch (Exception e) {
                       throw new RuntimeException(e);
                   }
               }
           }).start();
   
           new Thread(new Runnable() {
               @Override
               public void run() {
                   try {
                       //获取锁
                       lock2.acquire();
                       System.out.println("线程二 获取到锁");
                       //这里再次获取锁是为了验证该框架的锁是可重入锁
                       lock2.acquire();
                       System.out.println("线程二 再次获取到锁");
   
                       Thread.sleep(5*1000);
                       //释放锁
                       lock2.release();
                       System.out.println("线程二 释放锁");
                       //再次释放锁
                       lock2.release();
                       System.out.println("线程二 再次释放锁");
                   } catch (Exception e) {
                       throw new RuntimeException(e);
                   }
               }
           }).start();
       }
   
       /**
        * 初始化分布式锁
        * @return 分布式锁
        */
       private static CuratorFramework getCuratorFramework() {
   
           //重试策略，初试时间3秒,重试3次
           ExponentialBackoffRetry retry = new ExponentialBackoffRetry(3000, 3);
   
           //通过创建者模式创建Curator
           CuratorFramework client = CuratorFrameworkFactory.builder().connectString("36.137.128.27:2182,36.137.128.27:2181,36.137.128.27:2183")
                   .connectionTimeoutMs(2000)
                   .sessionTimeoutMs(2000)
                   .retryPolicy(retry)
                   .build();
           //开启连接 启动客户端
           client.start();
           System.out.println("zookeeper 客户端启动成功");
           return client;
       }
   }
   ```

   控制台

   ```TEX
   线程一 获取锁
   线程一 再次获取锁
   线程一 释放锁
   线程一 再次释放锁
   线程二 获取锁
   线程二 再次获取锁
   线程二 释放锁
   线程二 再次释放锁
   ```

   > 通常企业级项目都会使用成熟的框架，原生`API`的开发是非常少见的

## 企业面试重点真题

### 选举机制

半数机制，超过半数的投票通过，即通过。

1. 第一次启动选举规则：

   投票超过半数时，服务器id大的胜出

2. 第二次启动选举规则：

   1. EPOCH大的直接胜出
   2. EPOCH相同，事务id大的胜出
   3. 事务id相同，服务器id大的胜出

### 生产集群安装多少台`Zookeeper`合适

安装奇数台

**生产经验：**

> 10台服务器：3台Zookeeper
>
> 20台服务器：5台Zookeeper
>
> 100台服务器： 11台Zookeeper
>
> 200台服务器： 11台Zookeeper

**服务器台数多：好处，提高可靠性；坏处：提供通信延迟**

### 常用命令

```
ls`、`get`、`create`、`delete
```

> `create -e`: 表示创建临时节点
>
> ```
> ls -s`： 表示显示数据节点的状态信息 类似于`get -s
> ```
>
> `-w`: 表示监听
>
> `deleteall`：递归删除节点（将该节点下的所有节点递归删除后再删除该节点）
>
> `create -s`：表示创建有序号的节点