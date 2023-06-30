# RabbitMQ

## 基本概念

**`MQ`**全称 **Message Queue**（消息队列）,是在消息传输过程中保存消息的容器。多用于分布式系统之间进行通信。

- 分布式系统通信两种方式：直接远程调用 和 借助第三方完成间接通信
- 发送方称为生产者，接收方称为消费者

[![image-20220731133612671](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311844564.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311844564.png)

[![image-20220731134217286](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311844215.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311844215.png)

## `MQ`的优势与劣势

**优势**

- 应用解耦
- 异步提速
- 削峰填谷

**劣势**

- 系统可用性降低
- 系统复杂度提高
- 一致性问题

### `MQ`的优势

#### 应用解耦

[![image-20220731190749880](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311907908.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311907908.png)

**系统的耦合性越高，容错性就越低，可维护性就越低。**

[![image-20220731185909084](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311859119.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311859119.png)

有了`MQ`服务订单服务不需要集成库存服务、支付系统、物流系统或者其他系统，而是将系统全部解耦，拆分成不同的分布式微服务。微服务们通过监听`MQ`的信息，获取到符合的消息，然后消费。解耦也避免了某一个服务无法使用导致的整个系统崩溃问题。 同时多个服务耦合在一起也比解耦成单个服务的维护好做的多。

**使用`MQ`使得应用间解耦，提升容错性和可维护性。**

#### 异步提速

[![image-20220731191009885](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311910913.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311910913.png)

如果不适用`MQ`服务就必须等待服务远程调用到返回结果在响应。但是使用了`MQ`我们只需要将消息放入`MQ`中即可返回响应，分布式的系统只需要监听`MQ`，消费其中的消息即可。

用户点击完下单按钮后，只需等待`25ms`就能得到下单响应 (20 + 5 = `25ms`)。

**提升用户体验和系统吞吐量（单位时间内处理请求的数目）。**

#### 削峰填谷

**未使用`MQ`**

[![image-20220731191521273](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311915309.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311915309.png)

**使用`MQ`**

[![image-20220731191602635](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311916674.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311916674.png)

[![image-20220731191729405](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311917433.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311917433.png)

使用了 `MQ` 之后，限制消费消息的速度为1000，这样一来，高峰期产生的数据势必会被积压在 `MQ` 中，高峰

就被“削”掉了，但是因为消息积压，在高峰期过后的一段时间内，消费消息的速度还是会维持在1000，直

到消费完积压的消息，这就叫做“填谷”。

**使用`MQ`后，可以提高系统稳定性。**

### `MQ`的劣势

[![image-20220731191959687](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311919717.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311919717.png)

- **系统可用性降低**

  系统引入的外部依赖越多，系统稳定性越差。一旦 `MQ` 宕机，就会对业务造成影响。如何保证`MQ`的高可用？

- **系统复杂度提高**

  `MQ` 的加入大大增加了系统的复杂度，以前系统间是同步的远程调用，现在是通过 `MQ` 进行异步调用。如何保证消息没有被重复消费？怎么处理消息丢失情况？那么保证消息传递的顺序性？

- **一致性问题**

  A 系统处理完业务，通过 `MQ` 给B、C、D三个系统发消息数据，如果 B 系统、C 系统处理成功，D 系统处理失败。如何保证消息数据处理的一致性？

### 小结

既然 `MQ` 有优势也有劣势，那么使用 `MQ` 需要满足什么条件呢？

1. 生产者不需要从消费者处获得反馈。引入消息队列之前的直接调用，其接口的返回值应该为空，这才让明明下层的动作还没做，上层却当成动作做完了继续往后走，即所谓异步成为了可能。
2. 容许短暂的不一致性。
3. 确实是用了有效果。即解耦、提速、削峰这些方面的收益，超过加入`MQ`，管理`MQ`这些成本。

## 常见的`MQ`产品

目前业界有很多的 `MQ` 产品，例如 `RabbitMQ`、`RocketMQ`、`ActiveMQ`、`Kafka`、`ZeroMQ`、`MetaMq`等，也有直接使用 `Redis` 充当消息队列的案例，而这些消息队列产品，各有侧重，在实际选型时，需要结合自身需求及 `MQ` 产品特征，综合考虑。

[![image-20220731193822769](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311938809.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311938809.png)

## 简介

**`RabbitMQ` 是基于 `AMQP` 协议使用 Erlang 语言开发的一款消息队列产品。**

`AMQP`，即 Advanced Message Queuing Protocol（高级消息队列协议），是一个网络协议，是应用层协议的一个开放标准，为面向消息的中间件设计。基于此协议的客户端与消息中间件可传递消息，并不受客户端/中间件不同产品，不同的开发语言等条件的限制。2006年，**`AMQP` 规范发布。类比HTTP。**

[![image-20220731194023059](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311940089.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311940089.png)

2007年，Rabbit 技术公司基于 `AMQP` 标准开发的 `RabbitMQ` 1.0 发布。`RabbitMQ` 采用 Erlang 语言开发。Erlang 语言由 `Ericson` 设计，专门为开发高并发和分布式系统的一种语言，在电信领域使用广泛。

`RabbitMQ` 基础架构如下图：

[![image-20220731194224348](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311942381.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311942381.png)

### **`RabbitMQ` 中的相关概念：**

- **Broker**：接收和分发消息的应用，`RabbitMQ` Server就是Message Broker
- **Virtual host**：出于多租户和安全因素设计的，把`AMQP` 的基本组件划分到一个虚拟的分组中，类似于网络中的`namespace` 概念。当多个不同的用户使用同一个`RabbitMQ` server 提供的服务时，可以划分出多个`vhost`，每个用户在自己的`vhost` 创建exchange／queue 等
- **`Connection`**：publisher／consumer 和broker 之间的TCP 连接
- **Channel**：如果每一次访问`RabbitMQ` 都建立一个Connection，在消息量大的时候建立TCP Connection的开销将是巨大的，效率也较低。Channel 是在connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个thread创建单独的channel 进行通讯，`AMQP` method 包含了channel id 帮助客户端和message broker 识别channel，所以channel 之间是完全隔离的。Channel 作为轻量级的Connection 极大减少了操作系统建立TCP connection 的开销
- **`Exchange`**：message 到达broker 的第一站，根据分发规则，匹配查询表中的routing key，分发消息到queue 中去。常用的类型有：direct (point-to-point), topic (publish-subscribe) and fanout (`multicast`)
- **Queue**：消息最终被送到这里等待consumer 取走
- **`Binding`**：exchange 和queue 之间的虚拟连接，binding 中可以包含routing key。Binding 信息被保存到exchange 中的查询表中，用于message 的分发依据

### 工作模式

**`RabbitMQ`提供了6种工作模式**：简单模式、`workqueues`、`Publish/Subscribe`发布与订阅模式、Routing 路由模式、Topics主题模式、`RPC`远程调用模式（远程调用，不太算`MQ`；暂不作介绍）。官网对应模式介绍：[`RabbitMQ`模式介绍](https://www.rabbitmq.com/getstarted.html)

[![image-20220731195754182](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311957220.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207311957220.png)

### `JMS`

- `JMS` 即 Java 消息服务（`JavaMessage` Service）应用程序接口，是一个 Java 平台中关于面向消息中间件

  的`API`

- **`JMS` 是 `JavaEE` 规范中的一种，类比`JDBC`**

- 很多消息中间件都实现了`JMS`规范，例如：`ActiveMQ`。`RabbitMQ` 官方没有提供 `JMS `的实现包，但是开源社区有。

## 安装

**使用Docker安装**

**不指定账号密码的启动方式**

```
BASH
docker run -d -p 5672:5672 -p 15672:15672 --name rabbitmq rabbitmq:management
```

- **-d**：容器后台运行
- **-p**：映射端口 5672 `RabbitMQ`服务器端口号，15672是图形界面端口号
- **–name**：指定`RabbitMQ`名称

> 调用docker run后如果没有该镜像会自动拉取 不指定版本号默认拉取最新版`lastest`

**指定账户密码的启动方式**

```
BASH
docker run -d -p 15672:15672  -p  5672:5672  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin --name rabbitmq --hostname=rabbitmqhostone  rabbitmq:management
```

- **-d**： 后台运行
- **-p**： 隐射端口
- **–name**： 指定`rabbitMQ`名称
- **`RABBITMQ_DEFAULT_USER`**： 指定用户账号
- **`RABBITMQ_DEFAULT_PASS`**： 指定账号密码

安装好后，访问`http://ip:15672`如果安装无误可以看到以下界面

[![image-20220731203834936](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207312038049.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207312038049.png)

输入我们启动容器时设置的账号密码，如果没有指定默认`guest/guest`

[![image-20220731203915173](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207312039266.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202207312039266.png)

**得到以上界面表示你的安装无误。**

## 快速开始

1. 创建工程（生产者，消费者）

   `rabbitMQ-01-HellWord`、`rabbitMQ-01-comsumer`、`rabbitMQ-01-producer`

2. 分别添加依赖

   1. `rabbitMQ-01-HellWord` `pom.xml`

      ```XML
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns="http://maven.apache.org/POM/4.0.0"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
      
          <groupId>top.devildyw</groupId>
          <artifactId>rabbitMQ-01-HelloWord</artifactId>
          <packaging>pom</packaging>
          <version>1.0-SNAPSHOT</version>
          <modules>
              <module>rabbitMQ-01-consuemr</module>
              <module>rabbitMQ-01-producer</module>
          </modules>
      
          <properties>
              <maven.compiler.source>8</maven.compiler.source>
              <maven.compiler.target>8</maven.compiler.target>
              <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
              <amqp.client.version>5.13.1</amqp.client.version>
          </properties>
      
          <dependencyManagement>
              <dependencies>
                  <dependency>
                      <groupId>com.rabbitmq</groupId>
                      <artifactId>amqp-client</artifactId>
                      <version>${amqp.client.version}</version>
                  </dependency>
              </dependencies>
          </dependencyManagement>
      
          <build>
              <plugins>
                  <plugin>
                      <groupId>org.apache.maven.plugins</groupId>
                      <artifactId>maven-compiler-plugin</artifactId>
                      <version>3.10.1</version>
                  </plugin>
              </plugins>
          </build>
      
      </project>
      ```

      `rabbitMQ-01-comsumer` `pom.xml`

      ```XML
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns="http://maven.apache.org/POM/4.0.0"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <parent>
              <artifactId>rabbitMQ-01-HelloWord</artifactId>
              <groupId>top.devildyw</groupId>
              <version>1.0-SNAPSHOT</version>
          </parent>
          <modelVersion>4.0.0</modelVersion>
      
          <artifactId>rabbitMQ-01-consuemr</artifactId>
      
          <properties>
              <maven.compiler.source>8</maven.compiler.source>
              <maven.compiler.target>8</maven.compiler.target>
              <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          </properties>
      
          <dependencies>
              <dependency>
                  <groupId>com.rabbitmq</groupId>
                  <artifactId>amqp-client</artifactId>
              </dependency>
          </dependencies>
      
          <build>
              <plugins>
                  <plugin>
                      <groupId>org.apache.maven.plugins</groupId>
                      <artifactId>maven-compiler-plugin</artifactId>
                      <version>3.10.1</version>
                  </plugin>
              </plugins>
          </build>
      </project>
      ```

      `rabbitMQ-01-producer` `pom.xml`

      ```XML
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns="http://maven.apache.org/POM/4.0.0"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <parent>
              <artifactId>rabbitMQ-01-HelloWord</artifactId>
              <groupId>top.devildyw</groupId>
              <version>1.0-SNAPSHOT</version>
          </parent>
          <modelVersion>4.0.0</modelVersion>
      
          <artifactId>rabbitMQ-01-producer</artifactId>
      
          <properties>
              <maven.compiler.source>8</maven.compiler.source>
              <maven.compiler.target>8</maven.compiler.target>
              <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          </properties>
      
          <dependencies>
              <!--rabbitmq java客户端-->
              <dependency>
                  <groupId>com.rabbitmq</groupId>
                  <artifactId>amqp-client</artifactId>
              </dependency>
          </dependencies>
      
          <build>
              <plugins>
                  <plugin>
                      <groupId>org.apache.maven.plugins</groupId>
                      <artifactId>maven-compiler-plugin</artifactId>
                      <version>3.10.1</version>
                  </plugin>
              </plugins>
          </build>
      
      </project>
      ```

3. 编写生产者发送消息

   `Producer_HelloWorld`

   ```JAVA
   package top.dvildyw.producer;
   
   import com.rabbitmq.client.Channel;
   import com.rabbitmq.client.Connection;
   import com.rabbitmq.client.ConnectionFactory;
   
   import java.io.IOException;
   import java.util.concurrent.TimeoutException;
   
   /**
    * @author Devil
    * @since 2022-08-01-12:18
    */
   public class Producer_HelloWord {
       public static void main(String[] args) throws IOException, TimeoutException {
           //1. 创建连接工厂
           ConnectionFactory connectionFactory = new ConnectionFactory();
           //2. 设置参数
           connectionFactory.setHost("36.137.128.27"); //端口默认值 5672
           connectionFactory.setPort(5672);
           connectionFactory.setVirtualHost("/"); //虚拟机 默认值/
           connectionFactory.setUsername("admin"); //用户名 默认guest
           connectionFactory.setPassword("admin"); //用户名 默认guest
           //3. 获取对应连接 Connection
           Connection connection = connectionFactory.newConnection();
           //4. 创建Channel
           Channel channel = connection.createChannel();
           //5. 创建队列Queue
   
           /*
           Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                                    Map<String, Object> arguments) throws IOException;
            参数：
            1. queue: 队列名称
            2. durable: 是否持久化，当mq重启之后,还在
            3. exclusive:
               * 是否独占。只能能有一个消费者监听这个队列(仅限于此连接 如果该链接关闭队列也会删除)
            4. autoDelete: 是否自动删除。当没有Consumer时，自动删除掉
            5. argument: 参数
            */
           //如果没有一个名字叫作“hello_world”的队列,则会创建该队列,如果有则不会创建
           channel.queueDeclare("hello_world",true,false,false,null);
           //6.发送消息
           /*
           void basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties props, byte[] body)
               throws IOException;
   
           参数:
           1. exchange: 交换机名称。简单模式下交换机会使用默认的 “”
           2. routingKey: 路由配置
           3. props: 配置信息
           4. body: 发送消息数据
            */
           String body = "hello rabbitmq~~~";
   
           channel.basicPublish("","hello_word",null,body.getBytes());
   
   
           //7. 释放资源
           channel.close();
           connection.close();
       }
   }
   ```

   **生产者的编写大致可以分为7个步骤**

   1. 创建连接工厂
   2. 设置连接参数
   3. 获取对应连接
   4. 创建Channel
   5. 声明队列Queue
   6. 发送消息
   7. 关闭连接

   ------

   其中我们创建队列使用的方法是

   `Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments) throws IOException;`

   > 参数：
   > \1. `queue`: 队列名称
   > \2. ·durable`: 是否持久化，当mq重启之后,还在`
   > \3. exclusive`:`
   > \* 是否独占。只能能有一个消费者监听这个队列(仅限于此连接 如果该链接关闭队列也会删除)
   > \4. `autoDelete`: 是否自动删除。当没有Consumer时，自动删除掉
   > \5. `argument`: 参数

   ------

   发送消息则使用的是

   `void basicPublish(String exchange, String routingKey, boolean mandatory, BasicProperties props, byte[] body) throws IOException;`

   > 参数:
   > \1. `exchange`: 交换机名称。简单模式下交换机会使用默认的 “”
   > \2. `routingKey`: 路由配置
   > \3. `props`: 配置信息
   > \4. `body`: 发送消息数据

   启动查看图形控制界面 发现新增了队列`Hello_world`

   [![image-20220801124706439](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011247536.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011247536.png)

4. 编写消费者接收消息

   ```JAVA
   package top.devildyw.consumer;
   
   import com.rabbitmq.client.*;
   
   import java.io.IOException;
   import java.util.concurrent.TimeoutException;
   
   /**
    * @author Devil
    * @since 2022-08-01-12:47
    */
   public class Consumer_HelloWorld {
       public static void main(String[] args) throws IOException, TimeoutException {
           //1. 创建连接工厂
           ConnectionFactory connectionFactory = new ConnectionFactory();
           //2. 设置参数
           connectionFactory.setHost("36.137.128.27"); //端口默认值 5672
           connectionFactory.setPort(5672);
           connectionFactory.setVirtualHost("/"); //虚拟机 默认值/
           connectionFactory.setUsername("admin"); //用户名 默认guest
           connectionFactory.setPassword("admin"); //用户名 默认guest
           //3. 获取对应连接 Connection
           Connection connection = connectionFactory.newConnection();
           //4. 创建Channel
           Channel channel = connection.createChannel();
           //5. 声明队列Queue
   
           /*
           Queue.DeclareOk queueDeclare(String queue, boolean durable, boolean exclusive, boolean autoDelete,
                                    Map<String, Object> arguments) throws IOException;
            参数：
            1. queue: 队列名称
            2. durable: 是否持久化，当mq重启之后,还在
            3. exclusive:
               * 是否独占。只能能有一个消费者监听这个队列(仅限于此连接 如果该链接关闭队列也会删除)
            4. autoDelete: 是否自动删除。当没有Consumer时，自动删除掉
            5. argument: 参数
            */
           //如果没有一个名字叫作“hello_world”的队列,则会创建该队列,如果有则不会创建
           channel.queueDeclare("hello_world",true,false,false,null);
   
           //6.接收消息
           /*
           String basicConsume(String queue, boolean autoAck, Consumer callback) throws IOException;
           参数:
           1. queue: 队列名称
           2. autoAck: 是否自动确认
           3. callback: 回调对象
            */
           DefaultConsumer consumer = new DefaultConsumer(channel){
               /*
               回调方法,当收到消息后,会自动执行该方法
               1. consumerTag: 标识
               2. envelope: 获取一些信息,交换机,路由key
               3. properties: 配置信息
               4. body: 数据
                */
               @Override
               public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                   System.out.println("consumerTag: "+consumerTag);
                   System.out.println("Exchange: "+envelope.getExchange());
                   System.out.println("RoutingKey: "+envelope.getRoutingKey());
                   System.out.println("Properties: "+properties);
                   System.out.println("body: "+new String(body));
               }
           };
           channel.basicConsume("hello_world",true,consumer);
       }
   
       //消费者会去一直监听队列中的信息,不能够关闭资源
   }
   ```

   **消费者的编写大致可以分为6个步骤**

   1. 创建连接工厂
   2. 设置连接参数
   3. 获取对应连接
   4. 创建Channel
   5. 声明队列Queue
   6. 接收消息

   **这里之所以还要声明队列是为了防止该队列还未声明导致消费者监听报错。**

   **之所以不在最后关闭连接，是因为消费者需要一直监听队列中的信息。**

   **接收消息这里使用的方法是**

   `String basicConsume(String queue, boolean autoAck, Consumer callback) throws IOException;`

   > 参数:
   > \1. `queue`: 队列名称
   > \2. `autoAck`: 是否自动确认
   > \3. `callback`: 回调对象

   回调对象则是使用的`DefaultConsumer`

   ```JAVA
   DefaultConsumer consumer = new DefaultConsumer(channel){
              /*
              回调方法,当收到消息后,会自动执行该方法
              1. consumerTag: 标识
              2. envelope: 获取一些信息,交换机,路由key
              3. properties: 配置信息
              4. body: 数据
               */
              @Override
              public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                  System.out.println("consumerTag: "+consumerTag);
                  System.out.println("Exchange: "+envelope.getExchange());
                  System.out.println("RoutingKey: "+envelope.getRoutingKey());
                  System.out.println("Properties: "+properties);
                  System.out.println("body: "+new String(body));
              }
          };
   ```

   实现其中的回调方法`public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException;`

   > 回调方法,当收到消息后,会自动执行该方法
   > 1. `consumerTag`: 标识
   > 2. `envelope`: 获取一些信息,交换机,路由key
   > 3. `properties`: 配置信息
   > 4. `body`: 数据

------

上述的入门案例中其实使用的是如下的简单模模式：

[![image-20220801121453365](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011216044.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011216044.png)

在上图的模型中，有以下概念：

- P：生产者，也就是要发送消息的程序
- C：消费者：消息的接收者，会一直等待消息的到来
- queue：消息队列，途中红色部分。类似一个邮箱，可以缓存消息；生产者向其中投递消息，消费者从其中取出消息

## `RabbitMQ`工作模式

在**快速开始**中我们已经演示第一种工作模式`HelloWorld`模式了，下面我们会介绍其他几种模式。

### Work Queues

**模式说明**

[![image-20220801131354336](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011313383.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011313383.png)

- Work Queues：与入门程序的简单模式相比，**多了一个或一些消费者，多个消费端共同消费同一个队列中的消息（竞争关系）**。
- 应用场景：对于任务过重或任务较多情况使用工作队列可以**提高任务处理的速率**。

**Work Queues** 与入门程序的简单模式的代码几乎是一样的。可以完全复制，并多复制一个消费者进行多

个消费者同时对消费消息的测试。

**为了区分将队列名称修改为work_queues**

为了方便测试我们对生产者做了些许修改，使其可以一次发送大量的消息

```JAVA
for (int i = 0; i < 10; i++) {
    String body = i+" hello rabbitmq~~~"; //数字编号 1~10
    //发送
    channel.basicPublish("","hello_word",null,body.getBytes());
}
```

对于消费者我们对其进行了多个复制，来演示他们竞争的关系。

[![image-20220801141331431](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011413495.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011413495.png)

方便展示注释掉了回调方法中其他无关参数的打印。

```JAVA
	DefaultConsumer consumer = new DefaultConsumer(channel){
            /*
            回调方法,当收到消息后,会自动执行该方法
            1. consumerTag: 标识
            2. envelope: 获取一些信息,交换机,路由key
            3. properties: 配置信息
            4. body: 数据
             */
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
//                System.out.println("consumerTag: "+consumerTag);
//                System.out.println("Exchange: "+envelope.getExchange());
//                System.out.println("RoutingKey: "+envelope.getRoutingKey());
//                System.out.println("Properties: "+properties);
                System.out.println("body: "+new String(body));
            }
        };
        channel.basicConsume("hello_world",true,consumer);
    }
```

启动测试(先启动两个消费者监听队列)

`consumer1`

[![image-20220801141945564](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011419621.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011419621.png)

`consumer2`

[![image-20220801142010458](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011420513.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011420513.png)

**小结**

1. 在一个队列中如果有多个消费者，那么消费者之间对于同一个消息的消费关系是**竞争**的关系。
2. **Work Queues** 对于任务过重或人物较多情况使用工作队列可以提高人物处理的速度。例如：短信服务部署多个，只需要有一个节点发送成功即可。

### Pub/Sub 订阅模式

**模式说明**

[![image-20220801142920660](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011429714.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011429714.png)

在订阅模型中，多了一个 Exchange 角色，而且过程略有变化

- P：生产者，也就是要发送消息的程序，但不再发送到队列中，而是发给X（交换机）

- C：消费者，消息的接收者，会一直等待消息到来

- Queue：消息队列，接收消息，缓存消息

- Exchange：交换机（X）。一方面接受生产者放的消息，另一方面，直到如何处理消息，例如递交给某个特别队列、递交给所有队列、或是将消息丢弃。到底如何操作，取决于 Exchange 的类型。Exchange有常见以下三种类型：

  - Fanout：广播，将信息交给所有绑定到交换机的队列
  - Direct：定向，把消息交给符合所有指定routing key的队列
  - Topic：通配符，把消息交给符合routing pattern（路由模式）的队列

  Exchange（交换机）只负责转发消息，不具备存储消息的能力，因此如果没有任何队列与Exchange绑定，或者没有符合路由规则的队列，那么消息会丢失！

**代码实现**

**生产者`Producer_PubSub`**

```JAVA
package top.devildyw.producer;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author Devil
 * @since 2022-08-01-14:06
 */
public class Producer_PubSub {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1. 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //2. 设置参数
        connectionFactory.setHost("36.137.128.27"); //端口默认值 5672
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/"); //虚拟机 默认值/
        connectionFactory.setUsername("admin"); //用户名 默认guest
        connectionFactory.setPassword("admin"); //用户名 默认guest
        //3. 获取对应连接 Connection
        Connection connection = connectionFactory.newConnection();
        //4. 创建Channel
        Channel channel = connection.createChannel();

        //5. 声明交换机
        /*
        Exchange.DeclareOk exchangeDeclare(String exchange,
        BuiltinExchangeType type,
        boolean durable,
        boolean autoDelete,
        boolean internal,
        Map<String, Object> arguments) throws IOException;

        参数:
        1. exchange: 交换机名称
        2. type: 交换机类型 枚举类型
            DIRECT("direct"): 定向
            FANOUT("fanout"): 扇形(广播),发送消息到每一个与之绑定的队列
            TOPIC("topic"): 通配符的方式
            HEADERS("headers"): 参数匹配
        3. durable: 是否持久化
        4. autoDelete: 自动删除
        5. internal: 内部使用.-一般为false
        6. arguments: 参数
         */
        //测试广播模式
        String exchangeName = "test_fanout";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.FANOUT,true,false,false,null);
        //6. 创建队列
        String queue1Name = "test_fanout_queue1";
        String queue2Name = "test_fanout_queue2";
        channel.queueDeclare(queue1Name,true,false,false,null);
        channel.queueDeclare(queue2Name,true,false,false,null);
        //7. 绑定队列和交换机
        /*
        Queue.BindOk queueBind(String queue, String exchange, String routingKey) throws IOException;
        参数
        1. queue: 队列名称
        2. exchange: 交换机名称
        3. routingKey: 路由键,绑定规则
            如果交换机类型为fanout, routingKey设置为""
         */
        channel.queueBind(queue1Name,exchangeName,"");
        channel.queueBind(queue2Name,exchangeName,"");
        //8. 发送消息
        String body = "日志信息: 这是一条日志";
        channel.basicPublish(exchangeName,"",null,body.getBytes());
        //9. 释放资源
        channel.close();
        connection.close();
    }
}
```

**如上所述编写pub/sub模式的生产者需要9步**

1. 创建连接工厂
2. 设置参数
3. 获取对应连接 Connection
4. 创建Channel
5. 声明交换机
6. 创建队列
7. 绑定队列和交换机
8. 发送消息
9. 释放资源

**声明交换机**

```
Exchange.DeclareOk exchangeDeclare(String exchange, BuiltinExchangeType type, boolean durable, boolean autoDelete, boolean internal, Map<String, Object> arguments) throws IOException;
```

> 1. `exchange`: 交换机名称
>
> 2. `type`: 交换机类型 枚举类型
>
>    1. `DIRECT`(“direct”): 定向
>    2. `FANOUT`(“fanout”): 扇形(广播),发送消息到每一个与之绑定的队列
>    3. `TOPIC`(“topic”): 通配符的方式
>    4. `HEADERS`(“headers”): 参数匹配
>
> 3. `durable`: 是否持久化
>
> 4. `autoDelete`: 自动删除
>
> 5. `internal`: 内部使用.-一般为false
>
> 6. `arguments`: 参数

**绑定队列和交换机**

```
Queue.BindOk queueBind(String queue, String exchange, String routingKey) throws IOException;
```

> 1. `queue`: 队列名称
> 2. `exchange`: 交换机名称
> 3. `routingKey`: 路由键,绑定规则 如果交换机类型为fanout, `routingKey`设置为””

**`FANOUT`类型的交换机绑定`queue`是不需要设置`routingKey`的**

**消费者`Consuemr_PubSub1` `Consuemr_PubSub2`**

消费者没有太大的变换 只是分别绑定上述生产者创建的两个队列的队列名即可。

```JAVA
channel.basicConsume(queue1Name,true,consumer);
```
```JAVA
channel.basicConsume(queue2Name,true,consumer);
```

**启动测试**

`Consumer_PubSub1`

[![image-20220801152237353](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011522423.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011522423.png)

`Consumer_PubSub2`

[![image-20220801152301842](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011523900.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011523900.png)

**小结**

1. 交换机需要与队列进行绑定，绑定之后；一个消息可以被多个消费者都接收到（俗称广播）
2. 发布订阅模式和工作队列模式的区别：
   - 工作队列模式不用定义交换机，而发布/订阅模式需要定义交换机
   - **发布/订阅模式的生产方是面向交换机发送消息，工作队列模式的生方式面向队列发送消息（底层使用默认交换机）**
   - **发布/订阅模式需要设置队列和交换机的绑定**，工作队列模式不需要设置，实际上工作队列模式会将队列绑定到默认的交换机。

------

### Routing 路由模式

**模式说明**

- 队列与交换机的绑定，不能是任意绑定了，而是要指定一个`RoutingKey`（路由key）
- 消息的发送方在向`Exchagne`发送消息时，也必须指定消息的`RoutingKey`
- Exchange 不再把消息发送给每一个绑定的队列，而是根据消息的Routing Key进行判断，只有队列的`RoutingKey` 与消息的`RoutingKey` 完全一致，才会接收到消息。

[![image-20220801153109158](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011535573.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011535573.png)

图解：

- `P`：生产者，向Exchange发送消息，发送消息时，会指定一个`RoutingKey`
- `X`：Exchange（交换机），接收生产者的消息，然后把消息递交给`RoutingKey`完全匹配的队列
- `C1`：消费者，其所在队列指定了需要routing key 为error的消息
- `C2`：消费者，其所在队列指定了需要routing key为info、error、warning 的消息

> 在Direct类型下交换机下。交换机与队列绑定需要Routing Key，当生产者向交换机发送消息时也需要指定Routing Key，只有指定了Routing Key 交换机才可以确定将消息存入那个绑定的队列中。**相当于Routing Key只是生产者与队列之间的关系（生产者通过这种关系将消息存入指定的队列中），而消费者只需要去对应队列名中的队列中获取消息消费即可。**

**代码编写**

一般业务中要存到数据库中保存的日志 一般日志级别都是**`error`**

**设置Exchange类型为`Direct`**

生产者绑定两个队列，队列一绑定了Routing key 为 `error`， 队列二绑定了3个Routing Key ，分别为 `error`、`info`、`warning`

**生产者`Producer_Routing`**

```JAVA
package top.devildyw.producer;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author Devil
 * @since 2022-08-01-14:06
 */
public class Producer_Routing {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1. 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //2. 设置参数
        connectionFactory.setHost("36.137.128.27"); //端口默认值 5672
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/"); //虚拟机 默认值/
        connectionFactory.setUsername("admin"); //用户名 默认guest
        connectionFactory.setPassword("admin"); //用户名 默认guest
        //3. 获取对应连接 Connection
        Connection connection = connectionFactory.newConnection();
        //4. 创建Channel
        Channel channel = connection.createChannel();

        //5. 声明交换机
        /*
        Exchange.DeclareOk exchangeDeclare(String exchange,
        BuiltinExchangeType type,
        boolean durable,
        boolean autoDelete,
        boolean internal,
        Map<String, Object> arguments) throws IOException;

        参数:
        1. exchange: 交换机名称
        2. type: 交换机类型 枚举类型
            DIRECT("direct"): 定向
            FANOUT("fanout"): 扇形(广播),发送消息到每一个与之绑定的队列
            TOPIC("topic"): 通配符的方式
            HEADERS("headers"): 参数匹配
        3. durable: 是否持久化
        4. autoDelete: 自动删除
        5. internal: 内部使用.-一般为false
        6. arguments: 参数
         */
        //测试广播模式
        String exchangeName = "test_direct";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.DIRECT,true,false,false,null);
        //6. 创建队列
        String queue1Name = "test_direct_queue1";
        String queue2Name = "test_direct_queue2";
        channel.queueDeclare(queue1Name,true,false,false,null);
        channel.queueDeclare(queue2Name,true,false,false,null);
        //7. 绑定队列和交换机
        /*
        Queue.BindOk queueBind(String queue, String exchange, String routingKey) throws IOException;
        参数
        1. queue: 队列名称
        2. exchange: 交换机名称
        3. routingKey: 路由键,绑定规则
            如果交换机类型为fanout, routingKey设置为""
         */
        //队列1的绑定
        channel.queueBind(queue1Name,exchangeName,"error");
        //队列2的绑定
        channel.queueBind(queue2Name,exchangeName,"info");
        channel.queueBind(queue2Name,exchangeName,"error");
        channel.queueBind(queue2Name,exchangeName,"warning");
        //8. 发送消息
        String body = "日志信息: 这是一条日志";
        channel.basicPublish(exchangeName,"info",null,body.getBytes());
        //9. 释放资源
        channel.close();
        connection.close();
    }
}
```

**消费者`Consumer_Routing1`、`Consumer_Routing2`**

两个消费者去监听不同名称的队列即可

`Consumer_Routing1`用于存储日志级别为error的日志，`Consumer_Routing2`用来将个级别日志打印在控制台上。

启动测试

**生产者发送 routing key 为 `info`的消息**

`Consumer_Routing1`

[![image-20220801161056145](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011610208.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011610208.png)

`Consumer_Routing2`

[![image-20220801161048325](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011610389.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011610389.png)

**生产者发送 routing key 为`error`的消息**

`Consumer_Routing1`

[![image-20220801161615050](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011616105.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011616105.png)

`Consumer_Routing1`

[![image-20220801161628521](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011616571.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011616571.png)

**小结**

**`Routing`** 模式要求队列在绑定交换机时要指定 **routing key**，消息会转发到符合 routing key 的队列

### Topics 通配符模式

**模式说明**

- Topic 类型与 Direct 相比，都是可以根据 Routing Key把消息路由到不同的队列。只不过 Topic 类型 Exchange 可以让队列在绑定 Routing key的 时候使用**通配符**！
- `RoutingKey` 一般都是由一个或多个单词组成，多个单词之间以 ”.“ 分割，例如：item.insert
- 通配符规则：# 匹配一个或多个词，* 匹配不多不少恰好一个词，例如：item.# 能够匹配 item.insert.abc 或者 item.insert，item.* 只能匹配 item.insert

[![image-20220801162829423](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011628500.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011628500.png)

图解：

- 红色 Queue：绑定的是 usa.# ，因此凡是以 usa.开头的 routing key 都会被匹配到
- 黄色 Queue：绑定的是 #.news ,因此凡是以 .news 结尾的 routing key 都会被匹配到

**代码编写**

**需求: 所有error级别的日志存入数据库，所有order系统的日志存入数据库**

生产者`Producer_Topic`

```JAVA
package top.devildyw.producer;

import com.rabbitmq.client.BuiltinExchangeType;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

/**
 * @author Devil
 * @since 2022-08-01-14:06
 */
public class Producer_Topic {
    public static void main(String[] args) throws IOException, TimeoutException {
        //1. 创建连接工厂
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //2. 设置参数
        connectionFactory.setHost("36.137.128.27"); //端口默认值 5672
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/"); //虚拟机 默认值/
        connectionFactory.setUsername("admin"); //用户名 默认guest
        connectionFactory.setPassword("admin"); //用户名 默认guest
        //3. 获取对应连接 Connection
        Connection connection = connectionFactory.newConnection();
        //4. 创建Channel
        Channel channel = connection.createChannel();

        //5. 声明交换机
        /*
        Exchange.DeclareOk exchangeDeclare(String exchange,
        BuiltinExchangeType type,
        boolean durable,
        boolean autoDelete,
        boolean internal,
        Map<String, Object> arguments) throws IOException;

        参数:
        1. exchange: 交换机名称
        2. type: 交换机类型 枚举类型
            DIRECT("direct"): 定向
            FANOUT("fanout"): 扇形(广播),发送消息到每一个与之绑定的队列
            TOPIC("topic"): 通配符的方式
            HEADERS("headers"): 参数匹配
        3. durable: 是否持久化
        4. autoDelete: 自动删除
        5. internal: 内部使用.-一般为false
        6. arguments: 参数
         */
        //测试广播模式
        String exchangeName = "test_topic";
        channel.exchangeDeclare(exchangeName, BuiltinExchangeType.TOPIC,true,false,false,null);
        //6. 创建队列
        String queue1Name = "test_topic_queue1";
        String queue2Name = "test_topic_queue2";
        channel.queueDeclare(queue1Name,true,false,false,null);
        channel.queueDeclare(queue2Name,true,false,false,null);
        //7. 绑定队列和交换机
        /*
        Queue.BindOk queueBind(String queue, String exchange, String routingKey) throws IOException;
        参数
        1. queue: 队列名称
        2. exchange: 交换机名称
        3. routingKey: 路由键,绑定规则
            如果交换机类型为fanout, routingKey设置为""
         */
        
        
        //routing key 格式 系统的名称.日志的级别
        //需求: 所有error级别的日志存入数据库，所有order系统的日志存入数据库
        //队列1的绑定
        channel.queueBind(queue1Name,exchangeName,"#.error");
        channel.queueBind(queue1Name,exchangeName,"order.*");
        //队列2的绑定
        channel.queueBind(queue2Name,exchangeName,"*.*");
        //8. 发送消息
        String body = "日志信息: 这是一条日志 日志级别:error";
        channel.basicPublish(exchangeName,"order.info",null,body.getBytes());
        //9. 释放资源
        channel.close();
        connection.close();
    }
}
```

> `RoutingKey` 一般都是由一个或多个单词组成，多个单词之间以 ”.“ 分割，例如：item.insert

交换机与队列的绑定（通配符的配置）

```JAVA
channel.queueBind(queue1Name,exchangeName,"#.error");
      channel.queueBind(queue1Name,exchangeName,"order.*");
      //队列2的绑定
      channel.queueBind(queue2Name,exchangeName,"*.*");
```

**消费者 `Consumer_Topic1`、`Consumer_Topic2`**

修改部分：修改两个消费者监听的队列名称

启动测试

生产者发送 `routingKey` 为 `order.info` 的消息

`Consumer_Topic1`

[![image-20220801165834676](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011658734.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011658734.png)

`Consumer_Topic2`

[![image-20220801165818326](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011658384.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011658384.png)

生产者发送 `routingKey` 为 `goods.info` 的消息

`Consumer_Topic1`

[![image-20220801170213533](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011702595.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011702595.png)

`Consumer_Topic2`

[![image-20220801170152429](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011701487.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208011701487.png)

根据发送的routing key`goods.info`无法匹配队列一 通配符，但可以匹配队列二得通配符。

**小结**

Topic 主题模式可以实现 Pub/Sub 发布于订阅模式和 Routing 路由模式的功能， 只能是 Topic 在配置 routing key 的时候可以使用通配符，显得更加灵活。

### 工作模式总结

1. 简单模式 Hello World
   - 一个生产者、一个消费者，不需要设置交换机（使用默认的交换机）。
2. 工作队列模式 Work Queue
   - 一个生产者、多个消费者（竞争关系），不需要设置交换机（使用默认的交换机）。
3. 发布订阅模式 Publish/subscribe
   - 需要设置类型为 fanout 的交换机，并且交换机和队列进行绑定，当发送消息到交换机后，交换机会将消息发送到绑定的队列。
4. 路由模式 Routing
   - 需要设置类型为 direct 的交换机，交换机和队列进行绑定，并且指定 routing key，当发送消息到交换机后，交换机会根据 routing key 将消息发送到对应的队列。
5. 通配符模式 Topic
   - 需要设置类型为 topic 的交换机，交换机和队列进行绑定，并且指定通配符方式的 routing key，当发送消息到交换机后，交换机会根据 routing key 将消息发送到对应的队列。

------

## Spring Boot 整合 `RabbitMQ`

```JAVA
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
  <version>2.7.2</version>
</dependency>
```

### 生产者

1. 创建生产者工程

2. 导入`pom.xml`依赖

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>rabbitMQ-06-SpringBoot</artifactId>
           <groupId>top.devildyw</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>rabbitMQ-06-Consumer</artifactId>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-amqp</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
           </dependency>
       </dependencies>
   
   </project>
   ```

3. `application.yml`

   ```YML
   spring:
     rabbitmq:
       host: 36.137.128.27
       port: 5672
       username: admin
       password: admin
       virtual-host: /
   ```

4. 主启动类 （常规）

5. `RabbitMQ`配置类

   主要来配置交换机，队列，交换机以及队列之间的绑定关系

   ```JAVA
   package top.devildyw.consumer.config;
   
   import com.rabbitmq.client.AMQP;
   import org.springframework.amqp.core.*;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   /**
    * @author Devil
    * @since 2022-08-01-19:33
    */
   @Configuration
   public class RabbitMQConfig {
   
       public static final String EXCHANGE_NAME = "boot_topic_exchange";
       public static final String QUEUE_NAME = "boot_topic_queue";
   
       //配置交换机
       @Bean("bootExchange")
       public Exchange bootExchange(){
           //声明一个topic类型的交换机
           Exchange exchange = ExchangeBuilder.topicExchange(EXCHANGE_NAME)
                   .durable(true) //是否持久化
                   .build();
           return exchange;
       }
   
       //配置队列
       @Bean("bootQueue")
       public Queue bootQueue(){
           Queue queue = QueueBuilder.durable(QUEUE_NAME).build();
           return queue;
       }
   
   
       //队列和交换机绑定关系 Binding
   
       /**
        * 1. 队列
        * 2. 交换机
        * 3. routing key
        * @param queue 队列
        * @param exchange 交换机
        * @return binding
        */
       @Bean("bootQueueExchange")
       public Binding bootQueueExchange(@Qualifier("bootQueue") Queue queue, @Qualifier("bootExchange") Exchange exchange){
           Binding binding = BindingBuilder.bind(queue).to(exchange).with("boot.#").noargs();
           return binding;
       }
   }
   ```

6. 编写测试类

   ```JAVA
   package top.devildyw.consumer;
   
   import org.junit.jupiter.api.Test;
   import org.springframework.amqp.rabbit.core.RabbitTemplate;
   import org.springframework.boot.test.context.SpringBootTest;
   import top.devildyw.consumer.config.RabbitMQConfig;
   
   import javax.annotation.Resource;
   
   /**
    * @author Devil
    * @since 2022-08-01-19:43
    */
   @SpringBootTest
   public class ProducerTest {
       //注入RabbitTemplate
       @Resource
       private RabbitTemplate rabbitTemplate;
       @Test
       public void testSend(){
           rabbitTemplate.setExchange(RabbitMQConfig.EXCHANGE_NAME);//设置交换机
           rabbitTemplate.convertAndSend("boot.haha","boot mq hello~~"); //发送到指定交换机上的指定routingkey 队列
       }
   }
   ```

   > 流程：导入依赖–>配置->编写配置类->注入`RabbitTemplate`发送消息
   >
   > `RabbitMQ`发送消息的流程: 生产者指定交换机,Routing key –> 消息被发送到交换机 –> 交换机转发到与之绑定却routing key相匹配的队列

### 消费者

1. 创建工程

2. 导入`pom.xml`依赖 依赖于生产者相同

3. `application.yml` 也与生产者相同

4. 消费者没有过多配置

5. 主启动类

6. 创建`RabbitMQListener`类

7. 在`RabbitMQListener`新建一个方法 叫做`ListenerQueue` 带上`@RabbitListener`

   ```JAVA
   //指定queue的名称
      @RabbitListener(queues = "boot_topic_queue")
      public void listenerQueue(Message message){
          System.out.println(message);
      }
   ```

   监听queue中的消息，获取到消息后，会以Message对象的方式注入到该方法中。

> 流程：导入依赖–>配置->在受到Spring容器管理的类中 –> 编写方法来接受消息（带上注解，指定队列名称）
>
> `RabbitMQ`接收消息的流程: 消费者 –> 监听队列 –> 交换机将消息发送到队列中 –> 消费者接收队列中的消息

### 启动测试

生产者使用 `Topic`类型交换机发送消息`boot mq hello~~`

[![image-20220801201014349](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208012010429.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208012010429.png)

消费者监听`boot_topic_queue`队列

[![image-20220801201113895](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208012011953.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208012011953.png)

### 小结

Spring Boot整合 `RabbitMQ` 减少了代码量、提供了配置类工厂供开发人员使用，配置完成后，只需要注入其提供了`RabbitTemplate`，即可轻松地发送消息；

消费端直接使用`@RabbitListener`完成消息接收

### 消息转化器

Spring AMQP发送方法中，接收消息的类型是Object，也就是说我们可以发送任意对象类型的消息，是因为Spring AMQP默认会帮助我们序列化为字节后发送

当然我们也可以自定义序列化的方式，比如JSON格式

生产者消费者都引入JSON依赖

```XML
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-json</artifactId>
</dependency>
```

配置新的消息转化器

```JAVA
@Configuration
public class RabbitMQConfig {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```

生产者发送

```JAVA
@Test
public void testMessageConverter(){
    HashMap<String, String> map = new HashMap<>();
    map.put("ding","丁杨维");
    rabbitTemplate.convertAndSend("message.queue",map);
}
```

消费者接收

```JAVA
@RabbitListener(queuesToDeclare = @Queue(
        name = "message.queue",
        durable = "true"
))
public void listenMessageJsonConverter(Map<String,Object> map){
    System.out.println(map);
}
```

# 高级特性

[![image-20220807185257378](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208071853550.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208071853550.png)

## 消息的可靠性

### 消息的可靠性问题

消费者从生产者发送到exchange，再到queue，再到消费者，有那些倒置消息丢失的可能性？

- 发送时丢失：
  - 生产者发送的消息未送达exchange
  - 消息到达exchange后未到达queue
- `MQ`宕机，queue将消息丢失
- consumer接收到消息后未消费就宕机

[![image-20220807185729375](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208071857452.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208071857452.png)

### 生产者确认机制

`RabbitMQ`提供了`publisher confirm`机制来避免消息发送到`MQ`过程中丢失。消息发送到`MQ`以后，会返回一个结果给发送者，表示消息是否处理成功。结果有两种请求：

- `publisher-confirm`，发送者确认

  - 消息成功投递到交换机，返回`ack`
  - 消息未投递到交换机，返回`nack`

- `publisher-return`，发送者回执

  - 消息投递到交换机了，但是没有路由到队列。返回`ack`，及路由失败原因。

    [![image-20220807190521930](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208071905004.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208071905004.png)

> 注意：确认机制发送消息时，需要每个消息设置一个全局唯一ID，以区分不同消息，避免`ack`冲突。

#### 编码-工程基础配置

1. 创建生产者、消费者工程
2. pom.xml依赖

```XML
<dependencies>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-amqp</artifactId>
           <version>2.6.4</version>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <version>2.7.1</version>
       </dependency>
   </dependencies>
```

1. `application.yml`配置`rabbitMQ`的连接配置

生产者

```YML
spring:
  rabbitmq:
    host: 36.137.128.27
    port: 5672
    username: admin
    password: admin
    virtual-host: /
```

消费者

```YML
spring:
  rabbitmq:
    host: 36.137.128.27
    port: 5672
    username: admin
    password: admin
    virtual-host: /
    listener:
      simple:
        prefetch: 1
```

1. 启动类编写

2. 消费者编写监听类

   ```JAVA
   @Component
   public class SpringRabbitListener {
       @RabbitListener(queues = "test_queue_confirm")
       public void listenSimpleQueue(String msg){
           System.out.println("消费者接收到simple.queue的消息:"+msg);
       }
   }
   ```

#### 编码-SpringAMQP实现生产者确认

1. publisher这个微服务的application.yml中添加配置：

   ```YML
   spring:
     rabbitmq:
     	.....
       publisher-confirm-type: CORRELATED
       publisher-returns: true
       template:
         mandatory: true
   ```

   配置说明：

   - `publisher-confirm-type`：开启`publisher-confirm`，这里支持两种类型：

     - `simple`：同步等待`confirm`结果，直到超时（类似同步调用，等待消息发送到交换机中返回确认消息才继续执行）
     - `correlated`：异步回调，定义`ConfirmCallback`，`MQ`返回时会回调这个`ConfirmCallback`（异步调用，发送后继续后续操作，当交换机中接收到并返回结果时会通知。）

   - `publisher-returns`：开启`publish-return`功能，同样是基于`callback`机制，不过是定义`ReturnCallback`

   - `template.mandatory`：定义消息路由失败时的策略。`true`，则调用`ReturnCallback`；`false`：则直接丢弃消息

2. 每个`RabbitTemplate`只能配置一个`ReturnCallback`，因此需要在项目启动过程中配置全局`ReturnCallback`：

   ```JAVA
   package top.devildyw.producer.config;
   
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.amqp.core.ReturnedMessage;
   import org.springframework.amqp.rabbit.core.RabbitTemplate;
   import org.springframework.beans.BeansException;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.ApplicationContextAware;
   import org.springframework.context.annotation.Configuration;
   
   /**
    * @author Devil
    * @since 2022-08-07-19:39
    */
   @Slf4j
   @Configuration
   public class CommonConfig implements ApplicationContextAware {
       //实现ApplicationContextAware接口
       //在这里配置RabbitTemplate的全局ReturnCallBack
       @Override
       public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
           //获取RabbitTemplate对象
           RabbitTemplate rabbitTemplate = applicationContext.getBean(RabbitTemplate.class);
           rabbitTemplate.setReturnsCallback(new RabbitTemplate.ReturnsCallback() {
               @Override
               public void returnedMessage(ReturnedMessage returned) {
                   //记录日志
                   log.error("消息发送到队列失败，响应码：{},失败原因：{},交换机：{},路由key：{},消息msg：{}",
                           returned.getReplyCode(),returned.getReplyText(),returned.getExchange(),returned.getRoutingKey(),returned.getMessage());
                   //更具需求可以配置消息重发
               }
           });
       }
   }
   ```

3. `RabbitMQ`配置：交换机、队列、绑定关系

   ```JAVA
   package top.devildyw.producer.config;
   
   import org.springframework.amqp.core.*;
   import org.springframework.beans.factory.annotation.Qualifier;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   
   /**
    * @author Devil
    * @since 2022-08-04-19:52
    *
    * 配置类
    */
   @Configuration
   public class RabbitMQConfig {
       @Bean("getExchange")
       public Exchange exchange(){
           Exchange exchange = ExchangeBuilder.topicExchange("amp.topic").durable(true).build();
           return exchange;
       }
   
       @Bean("getQueue")
       public Queue queue(){
           Queue queue = QueueBuilder.durable("test_queue_confirm").build();
           return queue;
       }
   
       @Bean
       public Binding binding(@Qualifier("getExchange") Exchange exchange, @Qualifier("getQueue") Queue queue){
           Binding binding = BindingBuilder.bind(queue).to(exchange).with("simple.test").noargs();
           return binding;
       }
   
   }
   ```

4. 使用测试类完成消息的发送

   ```JAVA
   @Test
      public void TestConfirm(){
          //1.准备消息
          String message = "hello, spring amqp!";
      
          //2. 准备CorrelationData
          //2.1 准备ConfirmCallback
          CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
          correlationData.getFuture().addCallback(result -> {
              //判断结果
              if (result.isAck()){
                  //ACK
                  System.out.println("消息投递到交换机成功！消息ID:"+correlationData.getId());
                  log.info("消息投递到交换机成功！消息ID：{}",correlationData.getId());
              }else{
                  //NACK
                  System.out.println("消息投递到交换机失败！消息ID:"+correlationData.getId());
                  log.error("消息投递到交换机失败！消息ID：{}",correlationData.getId());
              }
          },ex -> { //出现异常导致消息发送失败
              //发送消息失败
              //记录日志
              log.error("消息发送失败！",ex);
              //根据需求可以重发消息
          });
          //发送消息
          rabbitTemplate.convertAndSend("amp.topic","simple.test",message,correlationData);
          while (true){
      
          }
      }
   ```

   想要完成**异步回调**，就需要在调用发送消息的方法中添加一个参数`correlationData`，在该参数中定义消息投递情况的回调方法以及发送消息失败的回调方法。当交换机接收到生产者的确认时，`CorrelateionData` 于 `ack/nack` 一起返回。

5. 启动消费者监听，启动生产者生成消息，观察控制台日志情况。

#### 小结

SpringAMQP中处理消息确认的集中情况：

- publisher-confirm：
  - 消息发送成功到exchange，返回ack
  - 消息发送失败，没有到达交换机，返回nack
  - 消息发送过程中出现异常，没有收到回执
- 消息成功发送到exchange，但没有路由到queue，
  - 调用`ReturnCallback`

------

### 消息持久化

`MQ`默认时内存存储消息，开启持久化功能可以确保缓存在`MQ`中的消息不丢失。

消息持久化是指将消息刷到磁盘以达到持久化保存的目的。

根据 [官方博文](http://www.rabbitmq.com/blog/2011/01/20/rabbitmq-backing-stores-databases-and-disks/) 的介绍，`RabbitMQ`在两种情况下会将消息写入磁盘：

1. 消息本身在publish的时候就要求消息写入磁盘；（后续惰性队列讲述）
2. 内存紧张，需要将部分内存中的消息转移到磁盘；

这里演示的就是内存中的消息到达一定阈值后，将消息转移到磁盘的情况。

将 exchange、queue 和 message 都进行持久化操作后，也不能保证消息一定不会丢失，消息存入`RabbitMQ` 之后，还需要一段时间才能存入硬盘。`RabbitMQ` 并不会为每条消息都进行同步存盘，如果在这段时间，服务器宕机或者重启，消息还没来得及保存到磁盘当中，就会丢失。

创建**交换机**或者**队列**时调用**durable方法**

> 注意：如果 exchange 和 queue 两者之间有一个持久化，一个非持久化，就不允许建立绑定。

1. 交换机持久化

   ```JAVA
   @Bean("getExchange")
      public Exchange exchange(){
          Exchange exchange = ExchangeBuilder.topicExchange("amp.topic").durable(true).build();
          return exchange;
      }
   ```

2. 队列持久化

   ```JAVA
   @Bean("getQueue")
      public Queue queue(){
          Queue queue = QueueBuilder.durable("test_queue_confirm").build();
          return queue;
      }
   ```

3. 消息持久化，`Spring AMQP`中的消息**默认是持久的**，可以通过`MessageProperties`中的`DeliveryMode`来指定（指定持久或是不持久）。

   ```JAVA
   MessageBuilder.withBody("hello".getBytes())
                   .setDeliveryMode(MessageDeliveryMode.PERSISTENT) //持久化消息
                   .build();
   ```

在图形控制界面中**`Features`**为D表示该组件持久化

[![image-20220807210954806](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072109883.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072109883.png)

通过观察创建队列，交换机等组件的构造方法可以看出 **`RabbitMQ`中的各个组件都是默认持久化的**

[![image-20220807211656649](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072116707.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072116707.png)

------

### 消费者确认

`RabbitMQ`支持消费者确认机制，即：消费者处理消息后可以向`MQ`发送`ack`回执，`MQ`收到`ack`回执后才会删除此消息。而`Spring AMQP`则允许三种确认模式：

- manual：手动`ack`，需要在业务代码结束后，调用`api`发送`ack`。
- auto：自动`ack`，由`spring`监控`listener`代码是否出现异常，没有异常则返回`ack`；抛出异常则返回`nack`
- none：关闭`ack`，`MQ`假定消费者获取消息后会成功处理，因此消息投递后立即被删除。

配置方式是修改**消费者**`application.yml`文件，添加下面配置：

```YML
spring:
  rabbitmq:
  	.....
    listener:
      simple:
        prefetch: 1 #每个消费者可以处理的未确认消息的最大数量。
        acknowledge-mode: NONE #none：关闭ack; manual：手动ack; auto: 自动ack
```

当消费者因报错或网络波动导致消息发送给了消费者，却没有返回`ack`，该消息就会被`Rabbitmq`标为`unacked`，队列会重新向消费者发送。

[![image-20220807213122791](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072131847.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072131847.png)

### 消费者失败重试

当消费者出现异常后，消息会不断requeue（重新入队）到队列，再重新发送给消费者，然后再次异常，再次erqueue，无限循环，导致mq的消息处理飙升，带来不必要的压力：

[![image-20220807213640249](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072136310.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072136310.png)

我们可以利用Spring的retry机制，在消费者出现异常时例用本地重试，而不是无限制的`requeue`到`mq`的队列。

```YML
spring:
  rabbitmq:
    host: 36.137.128.27
    port: 5672
    username: admin
    password: admin
    virtual-host: /
    listener:
      simple:
        prefetch: 1
        retry:
          enabled: true # 开启消费者失败重试
          initial-interval: 1000 # 初始的失败等待时长为1秒
          multiplier: 1 # 下次失败的等待时长倍数,下次等待时长 = multiplier * last-interval
          max-attempts: 3 # 最大重试次数
          stateless: true #true无状态;false有状态.如果业务中包含事务,这里改为false 决定重试是否是有状态
```

这种方式，重试次数耗尽，如果消息依然失败，则消息会被抛弃。

### 消费者失败消息处理策略

在开启重试模式后，重试次数耗尽，如果消息依然失败，则需要有`MessageRecoverer`接口处理，它包含三种不同的实现：

- `RejectAndDontRequeueRecoverer`：重试耗尽后，直接`reject`，丢弃消息。默认就是这种方式

- `ImmediateRequeueMessageRecoverer`：重试耗尽后，直接`nack`，消息重新入队

- `RepublishMessageRecoverer`：重试耗尽后，将失败消息投递到指定的交换机

  [![image-20220807220509233](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072205329.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072205329.png)

测试`RepublishMessageRecoverer`处理模式

- 首先定义接收失败消息的交换机、队列和其绑定关系：

  ```JAVA
  @Bean
      public DirectExchange directExchange(){
          return new DirectExchange("error.direct"); //创建一个交换机 用于专门处理(重发)消费失败的消息
      }
  
      @Bean
      public Queue errorQueue(){
          return new Queue("error.queue",true); //与上面专门处理消费失败的交换机相绑定的缓存消息的队列
      }
  
      /**
       * 定义队列与交换机绑定关系
       * @return
       */
      @Bean
      public Binding errorBinding(){
          return BindingBuilder.bind(errorQueue()).to(directExchange()).with("error");
      }
  ```

- 然后，定义`RepublishMessageRecoverer`：

  ```JAVA
  @Resource
      RabbitTemplate rabbitTemplate;
  
      /**
       * 配置消息重发模式
       * @return
       */
      @Bean
      public MessageRecoverer republishMessageRecoverer(){
          return new RepublishMessageRecoverer(rabbitTemplate,"error.direct","error");
      }
  ```

重新启动消费者，查看图形控制界面中观察error队列和交换机中的信息。

[![image-20220807222317143](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072223276.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208072223276.png)

可以看到，重试次数耗尽后的消息会进入到error交换机，发送到error队列中，其中信息内容会包含报错信息。

### 小结

任何保证RabbitMQ消息的可靠新？

- 开启生产者确认机制，确保生产者的消息能够到达队列
- 开启持久功能，确保消息未消费前在队列中不会丢失
- 开启消费者确认机制为auto，由spring确认消息处理成功后完成ack
- 开启消费者失败重试机制，并设置`MessageRecoverer`，多次重试失败后将消息投递到异常交换机，交由人工处理。

## 死信交换机

### 初始死信交换机

当一个队列中的消息满足下列情况之一时，可以成为**死信（dead letter）**：

- 消费者使用 `basic.reject` 或 `basic.nack` 声明消费失败，并且消息的 `requeue` 参数设置为 false
- 消息是一个过期消息，超时无人消费
- 要投递的队列消息堆积满了，最早的消息可能成为死信

如果该队列配置了 dead-letter-exchange 属性，指定了一个交换机，那么队列中的死信就会投递到这个交换机中，而这个交换机称为**死信交换机** （Dead Letter Exchange，简称 `DLX`）。

[![image-20220808130328413](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081303775.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081303775.png)

> 死信交换机与error交换机最大的区别就是，error交换机是由消费者去投递消息，而死信交换机则是通过队列投递。初次之外死信交换机还有着其他功能。

#### 小结

什么样的消息会成为死信？

- 消息被消费者reject或者返回nack
- 消息超时未消费
- 队列满了

如何给队列绑定死信交换机？

- 给队列设置 dead-letter-exchange 属性，指定一个交换机
- 给队列设置 dead-letter-routing-key 属性，设置死信交换机与死信队里额的 `RoutingKey`

------

### TTL

TTL，也就是Time-To-Live。如果一个队列中的消息TTL结束仍未消费，则会变为死信，ttl超时分为两种情况：

- 消息存在的队列设置了存活时间
- 消息本身设置了存活时间

[![image-20220808130910216](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081309319.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081309319.png)

#### 编码-实现延迟消息

**思路：消息可以设置超时存活时间，不设置消费者监听该队列，一但消息超出存活时间，就会被队列投递到我们事先配置好的死信交换机中，此时监听死信队列的消费者就可以接收到消息并完成消费，就实现了消息的延迟消费。**

- 消费者监听死信队列消息

```JAVA
@Component
@Slf4j
public class SpringRabbitListener {
    //使用注解声明队列、交换机、以及绑定关系
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(name = "dl.queue",durable = "true"),
            exchange = @Exchange(name = "dl.direct"),
            key = "dl"
    ))
    public void listenDlQueue(String msg){
        log.info("消费者接收到了dl.queue的延迟消息");
    }
}
```

- TTL交换机、队列、绑定关系配置

```JAVA
@Configuration
public class TTLMessageConfig {
    @Bean
    public DirectExchange ttlDirectExchange(){
        return new DirectExchange("ttl.direct");
    }

    @Bean
    public Queue ttlQueue(){
        return QueueBuilder.durable("dl.queue")
                .ttl(10000) //设置消息的生存时间，在此之后它将被丢弃或路由到死信交换（如果已配置）。
                .deadLetterExchange("dl.direct") //指定死信交换机 这里超过存活时间队列就会将消息投递到死信交换机中
                .deadLetterRoutingKey("dl") //指定死信交换机与死信队列之间的routingkey 到时投递的消息都会发送到死信交换机绑定的routingkey对应的队列中
                .build();
    }

    @Bean
    public Binding ttlBinding(){
        return BindingBuilder.bind(ttlQueue()).to(ttlDirectExchange()).with("ttl");
    }
}
```

- 启动消费者监听

- 生产者发送消息

  ```JAVA
  @Test
     public void testTTLMessage(){
         //1. 消息准备
         String message = "ttl message";
         //2. 发送消息
         rabbitTemplate.convertAndSend("ttl.direct","ttl",message);
         //记录日志
         log.info("消息已经成功发送！");
     }
  ```

  **这里消息可以不设置超时存活时间，因为队列中已经设置，如果消息也设置，则取两者最小值。**

  控制台结果

  [![image-20220808133702626](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081337679.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081337679.png)

  [![image-20220808133712476](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081337895.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081337895.png)

  **延时成功**

- 发送消息时，给消息本省设置超时时间

  ```JAVA
  @Test
     public void testTTLMessage(){
         //1. 消息准备
         Message message = MessageBuilder.withBody("ttl message".getBytes(StandardCharsets.UTF_8))
                 .setExpiration("5000") //设置5秒超时时间
                 .build();
         //2. 发送消息
         rabbitTemplate.convertAndSend("ttl.direct","ttl",message);
         //记录日志
         log.info("消息已经成功发送！");
     }
  ```

  此时延时时间由10秒变为了5秒。证实了当队列和消息都设置了超时时间取之间最小值。

#### 小结

消息超时的两种方式

- 给队列设置 `ttl` 属性，进入队列后超过 `ttl` 时间的消息变为死信
- 给消息设置 `ttl` 属性，队列接收到消息超过`ttl`时间后变为死信
- 两者共存时，以时间短的 `ttl` 为准

### 延迟队列

例用TTL结合死信交换机，我们实现了消息发出后，消费者延迟收到消息的效果。这种消息模式就称为**延迟队列（Delay Queue）**模式。

延迟队列的使用场景包括：

- 延迟发送短信
- 用户下单，如果用户在15分钟内未支付，则自动取消
- 预约工作会议，20秒后自动通知所有参会人员

#### 延迟队列插件

因为延迟队列的需求非常多，所以RabbitMQ的官方也推出了一个插件，原生支持延迟队列效果。

#### 安装延迟队列插件 DelayExchange

1. 下载插件

   `RabbitMQ`有一个官方的插件社区，地址为：[`Community Plugins — RabbitMQ`](https://www.rabbitmq.com/community-plugins.html)

   [![image-20220808135611458](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081356517.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081356517.png)

2. 上传插件

   因为我们是基于Docker安装，所以需要先查看RabbitMQ的插件对应的数据卷。如果没有数据卷，可以先创建一个。

   ```BASH
   $ docker volume create mq-plugins
   ```

   删除原有容器，创建新容器挂载数据卷

   ```BASH
   $ docker run -d -p 15672:15672  -p  5672:5672  -e RABBITMQ_DEFAULT_USER=admin -v mq-plugins:/plugins -e RABBITMQ_DEFAULT_PASS=admin --name rabbitmq --hostname=rabbitmqhostone  rabbitmq:management
   ```

   查看数据卷信息查找数据卷目录

   ```BASH
   $ docker volume inspect mq-plugins
   ```

   将我们刚刚下载的插件上传到该目录

3. 安装插件

   最后就是安装了，需要进入`MQ`容器内部来执行安装。

   进入容器内部后，执行下面命令开启插件：

   ```BASH
   bash rabbitmq-plugins enable rabbitmq_delayed_message_exchange
   ```

   [![image-20220808171500479](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081718986.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081718986.png)

   出现如下信息代表安装成功。

#### 使用插件

`DelayExchange`插件原理是对官方原生的`Exchange`做了功能的升级：

- 将`DelayExchange`接受到的消息暂存在内存中（官方的`Exchange`是无法存储消息的）
- 在`DelayExchange`中计时，超时后才投递消息到队列中

#### 手动指定

在 `RabbitMQ` 的管理平台声明一个 `DelayExchagne` ：

[![image-20220808173004955](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081730019.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081730019.png)

#### SpringAMQP使用延迟队列插件

`DelayExchange`的本质还是官方的三种交换机，只是添加了延迟功能。因此使用时只需要声明一个交换机，交换机的类型可以是任意类型，然后设定**`delayed`**属性为**`true`**即可。

基于注解的方式：

```JAVA
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(name = "delay.queue",durable = "true"),
        exchange = @Exchange(name = "delay.direct",delayed = "true"),
        key = "delay"
))
public void listenDelayedQueue(String msg){
    log.info("接收到 delay.queue的延迟消息：{}",msg);
}
```

基于`Java`代码的方式

```JAVA
@Configuration
public class DelayExchangeConfig {
    @Bean
    public DirectExchange delayedExchange(){
        return ExchangeBuilder.directExchange("delay.direct")
                .delayed() //指定为有延迟功能的交换机
                .durable(true)
                .build();
    }

    @Bean
    public Queue delayedQueue(){
        return new Queue("delay.queue");
    }

    @Bean
    public Binding delayBinding(){
        return BindingBuilder.bind(delayedQueue()).to(delayedExchange()).with("delay");
    }
}
```

然后我们相这个delay为true的交换机中发送消息，一定要给消息添加一个`header: x-delay`，值为延迟的时间，单位为毫秒

```JAVA
@Test
public void testDelayedMsg() {
    //创建消息
    Message message = MessageBuilder.withBody("hello,delayed message".getBytes(StandardCharsets.UTF_8))
            .setHeader("x-delay", 10000) //设置head 延迟属性 延迟10秒
            .build();
    //消息ID,需要封装到CorrelationData中
    CorrelationData correlationData = new CorrelationData(UUID.randomUUID().toString());
    //发送消息
    rabbitTemplate.convertAndSend("delay.direct","delay",message,correlationData);
    log.debug("发送消息成功");
}
```

> 这里的消息发送生产者端会报错，原因是我们的消息是发送到了交换机上暂存然后再发送到队列中，因为暂存所以消息没有一开始就被发送到队列，所以会报`NO_ROUTE`的错误。

可以在全局`ReturnCallback`中添加判断是否是延时消息来避免。

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081800548.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081800548.png)

#### 小结

延迟队列插件的使用步骤包括那些？

- 声明一个交换机，添加delayed属性为true
- 发送消息时，添加x-delay头，值为超时时间

## 惰性队列

### 消息堆积问题

当生产者发送个消息的速度超过了消费者处理消息的速度，就会导致队列中的消息堆积，直到队列存储消息到达上限。最早接收到的消息，就可能会称为死信，会被丢弃，这就是**消息堆积**问题

解决消息堆积有三种思路：

- 增加更多消费者，提高消费速度
- 在消费者内开启线程池加快消息处理速度（适合消息消费时间长的消息）
- 扩大队列容积，提高堆积上限

### 惰性队列

从RabbitMQ的3.6.0版本开始，就增加了Lazy Queues的概念，也就是惰性队列。惰性队列的特征如下：

- **接收到消息后直接存入磁盘而非内存**（消息默认存储到内存）
- **消费者要消费消息时才会从磁盘中读取并加载到内存**
- **支持数百万条的消息存储**

#### 声明惰性队列的方式

- **命令行方式**

而要设置一个队列为惰性队列，只需要在声明队列时，指定x-queue-mode属性为lazy即可。可以通过命令行将一个运行中的队列修改为惰性队列：

```BASH
$ rabbitmqctl set——policy Lazy "^lazy-queue$" '{"queue-mode":"lazy"}' --apply-to queues
```

- **用`SpringAMQP`声明惰性队列**

  - @Bean的方式

    ```
    JAVA
    @Configuration
    public class LazyConfig {
        @Bean
        public Queue lazyQueue(){
            return QueueBuilder.durable("lazy.queue")
                    .lazy() //开启x-queue-mode为lazy
                    .build();
        }
    }
    ```

  - 注解方式

    ```
    JAVA
    @RabbitListener(queuesToDeclare = @Queue(
               name = "lazy.queue",
               durable = "true",
               arguments = @Argument(name = "x-queue-mode",value = "lazy")
       ))
       public void listenLazyQueue(String msg){
           log.info("接收到 lazy,queue的消息：{}",msg);
       }
    ```

分别向将配置好的惰性队列和常规队列发送一百万条消息，**可以观察到惰性队列的消息接收更平稳，而常规队列波动很大。**

> 原因：惰性队列一接收到消息就会将消息写到磁盘，而不是内存；而常规队列是写到内存，一旦内存中的消息超过了`RabbitMQ`的一定阈值，就会暂停接收然后将消息写入磁盘（**page-out**）。

### 小结

消息堆积问题的解决方案？

- 队列上绑定多个消费者，提高消费速度
- 给消费者开启线程池，提高消费速度
- 使用惰性队列，可以在MQ中保存更多的消息

惰性队列的优点有哪些？

- 基于磁盘存储，消息上限高
- 没有间歇性的page-out，性能比较稳定

惰性队列的缺点有哪些?

- 基于磁盘存储，消息时效性会降低
- 性能受限于磁盘的IO

# `MQ`集群

## 集群分类

RabbitMQ是基于Erlang语言编写，而Erlang又是一个面向并发的语言，天然支持集群模式。RabbitMQ的集群有两种模式：

- 普通集群：是一种分布式集群，将队列分散到集群的各个结点，从而提高整个集群的并发能力。
- 镜像集群：是一种主从集群，普通集群的基础上，添加了主从备份功能，提高集群的数据可用性。不过，这种方式增加了数据同步的带宽消耗。

镜像集群虽然支持主从，但主从同步并不是强一致性的，某些请款下可能有数据丢失的风险。因此在`RabbitMQ`的3.8版本以后，退出了新的功能：**仲裁队列**来代替镜像集群，底层采取Raft协议确保主从的数据一致性。

## 普通集群

普通集群，或者叫做标准集群（classic cluster），具备下列特征：

- 会在集群的各个节点间共享部分数据，包括：交换机、队列元信息。不包括队列中的消息。

- 当访问集群某个节点时，如果队列不在该节点，会从数据所在节点传递到当前节点并返回

  [![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081852019.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081852019.png)

- 队列所在的节点宕机，队列中的消息就会丢失



### 搭建普通集群

我们的计划部署3节点的`mq`集群：

| 主机名 | 控制台端口    | amqp通信端口 |
| ------ | ------------- | ------------ |
| mq1    | 8081 —> 15672 | 8071 —> 5672 |
| mq2    | 8082 —> 15672 | 8072 —> 5672 |
| mq3    | 8083 —> 15672 | 8073 —> 5672 |

集群中的节点标示默认都是：`rabbit@[hostname]`，因此以上三个节点的名称分别为：

- rabbit@mq1
- rabbit@mq2
- rabbit@mq3

#### 获取cookie

RabbitMQ底层依赖于Erlang，而Erlang虚拟机就是一个面向分布式的语言，默认就支持集群模式。集群模式中的每个RabbitMQ 节点使用 cookie 来确定它们是否被允许相互通信。

要使两个节点能够通信，它们必须具有相同的共享秘密，称为**Erlang cookie**。cookie 只是一串最多 255 个字符的字母数字字符。

每个集群节点必须具有**相同的 cookie**。实例之间也需要它来相互通信。

我们先在之前启动的`mq`容器中获取一个cookie值，作为集群的cookie。执行下面的命令：

```SH
docker exec -it MQ容器id cat /var/lib/rabbitmq/.erlang.cookie
```

可以看到cookie值如下

```SH
CSKYABVGIEGXEZLHYGMR
```

接下来，停止并删除当前的`MQ`容器，我们重新搭建集群。

```SH
docker rm -f MQ容器id
```

清理下docker的数据卷

```SH
docker volume prune
```

#### 准备集群配置

在`/tmp`目录新建一个配置文件 `rabbitmq.conf`：

```SH
cd /tmp
# 创建文件
touch rabbitmq.conf
```

文件内容如下：

```NGINX
loopback_users.guest = false #禁用默认的guest用户 防止不法之人访问
listeners.tcp.default = 5672  #mq消息通信端口
cluster_formation.peer_discovery_backend = rabbit_peer_discovery_classic_config
cluster_formation.classic_config.nodes.1 = rabbit@mq1 #节点名称
cluster_formation.classic_config.nodes.2 = rabbit@mq2
cluster_formation.classic_config.nodes.3 = rabbit@mq3
```

再创建一个文件，记录cookie

```SH
cd /tmp
# 创建cookie文件
touch .erlang.cookie
# 写入cookie
echo "CSKYABVGIEGXEZLHYGMR" > .erlang.cookie
# 修改cookie文件的权限
chmod 600 .erlang.cookie
```

准备三个目录,mq1、mq2、mq3：

```SH
cd /tmp
# 创建目录
mkdir mq1 mq2 mq3
```

然后拷贝rabbitmq.conf、cookie文件到mq1、mq2、mq3：

```SH
# 进入/tmp
cd /tmp
# 拷贝
cp rabbitmq.conf mq1
cp rabbitmq.conf mq2
cp rabbitmq.conf mq3
cp .erlang.cookie mq1
cp .erlang.cookie mq2
cp .erlang.cookie mq3
```

#### 启动集群

创建一个网络：

```SH
docker network create mq-net
```

运行命令

```SH
docker run -d --net mq-net \
-v ${PWD}/mq1/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf \
-v ${PWD}/.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
--name mq1 \
--hostname mq1 \
-p 8071:5672 \
-p 8081:15672 \
rabbitmq:3.10-management
```
```SH
docker run -d --net mq-net \
-v ${PWD}/mq2/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf \
-v ${PWD}/.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
--name mq2 \
--hostname mq2 \
-p 8072:5672 \
-p 8082:15672 \
rabbitmq:3.10-management
```
```SH
docker run -d --net mq-net \
-v ${PWD}/mq3/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf \
-v ${PWD}/.erlang.cookie:/var/lib/rabbitmq/.erlang.cookie \
-e RABBITMQ_DEFAULT_USER=admin \
-e RABBITMQ_DEFAULT_PASS=admin \
--name mq3 \
--hostname mq3 \
-p 8073:5672 \
-p 8083:15672 \
rabbitmq:3.10-management
```

> `--net`将容器添加进指定的网络

[![image-20220808193233523](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081932654.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081932654.png)

#### 测试

在`mq1`这个节点上添加一个队列：

[![image-20220808193359818](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081933893.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081933893.png)

如图，在`mq2`和`mq3`两个控制台也都能看到：

[![image-20220808193516554](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081935633.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081935633.png)

#### 数据共享测试

点击这个队列，进入管理页面：

[![image-20220808193700143](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081937202.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081937202.png)

然后利用控制台发送一条消息到这个队列：

[![image-20220808193730351](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081937482.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081937482.png)

结果在`mq2`、`mq3`上都能看到这条消息：

[![image-20220808193834711](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081938791.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081938791.png)

#### 可用性测试

我们让其中一台节点mq1宕机：

```SH
docker stop mq1
```

然后登录mq2或mq3的控制台，发现simple.queue也不可用了：

[![image-20220808194107726](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081941214.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081941214.png)

说明队列是没有实现共享的。

## 镜像集群

镜像集群：本质是主从模式，具备下面 的特性

- 交换机、队列、队列中的消息会在各个mq的镜像节点之间同步备份。

- 创建队列的节点被称为该队列的主节点，备份到的其他节点叫做该队列的镜像节点。

- 一个队列的主节点可能是另一个队列的镜像节点

- 所有操作都是主节点完成，然后同步给镜像节点

- 主节点宕机后，镜像节点会代替称为新的主节点

  [![image-20220808194817144](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081948314.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208081948314.png)

> 官方文档地址：https://www.rabbitmq.com/ha.html

### 搭建镜像集群

镜像集群不需要重新搭建集群，只需要在原来普通集群节点上进行策略配置即可。**镜像集群更形象地可以成为集群的镜像模式。**

#### 镜像集群的配置

镜像模式的配置有3种模式：

| ha-mode         | ha-params         | 效果                                                         |
| --------------- | ----------------- | ------------------------------------------------------------ |
| 准确模式exactly | 队列的副本量count | 集群中队列副本（主服务器和镜像服务器之和）的数量。count如果为1意味着单个副本：即队列主节点。count值为2表示2个副本：1个队列主和1个队列镜像。换句话说：count = 镜像数量 + 1。如果群集中的节点数少于count，则该队列将镜像到所有节点。如果有集群总数大于count+1，并且包含镜像的节点出现故障，则将在另一个节点上创建一个新的镜像。 |
| all             | (none)            | 队列在群集中的所有节点之间进行镜像。队列将镜像到任何新加入的节点。镜像到所有节点将对所有群集节点施加额外的压力，包括网络I / O，磁盘I / O和磁盘空间使用情况。推荐使用exactly，设置副本数为（N / 2 +1）。 |
| nodes           | *node names*      | 指定队列创建到哪些节点，如果指定的节点全部不存在，则会出现异常。如果指定的节点在集群中存在，但是暂时不可用，会创建节点到当前客户端连接到的节点。 |

##### exactly模式

```PLAINTEXT
rabbitmqctl set_policy ha-two "^two\." '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'
```

- `rabbitmqctl set_policy`：固定写法

- `ha-two`：策略名称，自定义

- `"^two\."`：匹配队列的正则表达式，符合命名规则的队列才生效，这里是任何以`two.`开头的队列名称

- ```
  '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'
  ```

  : 策略内容

  - `"ha-mode":"exactly"`：策略模式，此处是exactly模式，指定副本数量
  - `"ha-params":2`：策略参数，这里是2，就是副本数量为2，1主1镜像
  - `"ha-sync-mode":"automatic"`：同步策略，默认是manual，即新加入的镜像节点不会同步旧的消息。如果设置为automatic，则新加入的镜像节点会把主节点中所有消息都同步，会带来额外的网络开销

------

##### all模式

```PLAINTEXT
rabbitmqctl set_policy ha-all "^all\." '{"ha-mode":"all"}'
```

- `ha-all`：策略名称，自定义

- `"^all\."`：匹配所有以`all.`开头的队列名

- ```
  '{"ha-mode":"all"}'
  ```

  ：策略内容

  - `"ha-mode":"all"`：策略模式，此处是all模式，即所有节点都会称为镜像节点

------

##### nodes模式

```PLAINTEXT
rabbitmqctl set_policy ha-nodes "^nodes\." '{"ha-mode":"nodes","ha-params":["rabbit@nodeA", "rabbit@nodeB"]}'
```

- `rabbitmqctl set_policy`：固定写法

- `ha-nodes`：策略名称，自定义

- `"^nodes\."`：匹配队列的正则表达式，符合命名规则的队列才生效，这里是任何以`nodes.`开头的队列名称

- `'{"ha-mode":"nodes","ha-params":["rabbit@nodeA", "rabbit@nodeB"]}'`:策略内容

  - `"ha-mode":"nodes"`：策略模式，此处是nodes模式
  - `"ha-params":["rabbit@mq1", "rabbit@mq2"]`：策略参数，这里指定副本所在节点名称

------

#### 测试

我们使用镜像集群的exactly模式，因为集群节点数量为3，因此镜像数量就设置为2.

运行下面的指令：

```SH
docker exec -it mq1 rabbitmqctl set_policy ha-two "^two\." '{"ha-mode":"exactly","ha-params":2,"ha-sync-mode":"automatic"}'
```

可以在控制台上`admin`中的Policies中看到我们配置的镜像集群策略

[![image-20220808215158257](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082151354.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082151354.png)

下面，我们创建一个新的队列：

[![image-20220808214913263](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082149347.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082149347.png)

在任意一个`mq`控制台查看队列：

将光标放在`+1`上可以看到镜像节点。也可点击队列进入队列详细信息中查看

[![image-20220808215441819](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082154886.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082154886.png)

[![image-20220808221121927](C:/Users/Devil/AppData/Roaming/Typora/typora-user-images/image-20220808221121927.png)](file:///C:/Users/Devil/AppData/Roaming/Typora/typora-user-images/image-20220808221121927.png)

------

##### 测试数据共享

给`two.queue`发送一条消息：

[![image-20220808215620484](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082156571.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082156571.png)

然后在`mq1`、`mq2`、`mq3`的任意控制台查看消息：

[![image-20220808215717049](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082157161.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082157161.png)

##### 测试高可用

现在，我们让two.queue的主节点mq1宕机：

```SH
docker stop mq1
```

查看集群状态：

[![image-20220808215810110](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082158235.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082158235.png)

查看队列状态：

[![image-20220808215843071](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082158135.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082158135.png)

发现`two.queue`依然是健康的！并且其主节点切换到了`rabbit@mq2`上，而且`mq3`成为了新的镜像节点。

## 仲裁队列

从RabbitMQ 3.8版本开始，引入了新的仲裁队列，他具备与镜像队里类似的功能，但使用更加方便。他是用来替代镜像模式的（因为镜像模式并非强一致性，可能会发生数据丢失即使概率不大）。

仲裁队列具有以下特征：

- 与镜像队列一样，都是主从模式，支持主从数据同步
- 使用非常简单，没有复杂的配置
- 主从同步基于Raft协议，强一致性

### 添加仲裁队列

在任意控制台添加一个队列，一定要选择队列类型为Quorum类型。

[![image-20220808220717424](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082207516.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082207516.png)

在任意控制台查看队列：

[![image-20220808221155857](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082211923.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082211923.png)

可以看到，仲裁队列的 + 2字样。代表这个队列有2个镜像节点。

因为仲裁队列默认的镜像数为5。如果你的集群有7个节点，那么镜像数肯定是5；而我们集群只有3个节点，因此镜像数量就是3.

查看队列详细信息

[![image-20220808221222923](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082212023.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208082212023.png)

可以看到主节点和所有成员，除去主节点其余的都是从节点（镜像）。

### 测试

测试情况参考镜像集群的测试，效果相同。

### SpringAMQP创建仲裁队列

在创建仲裁队列之前，首先需要配置连接集群。

```YML
spring:
  rabbitmq:
	.....
    addresses: 36.137.128.27:8071,36.137.128.27:8072,36.137.128.27:8073
    username: admin
    password: admin
    virtual-host: /
    ......
```

创建仲裁队列

```JAVA
@Configuration
public class QuorumQueueConfig {
    @Bean
    public Queue quorumQueue(){
        return QueueBuilder.durable("quorum.queue2")
                .quorum() //设置为仲裁队列
                .build();
    }
}
```

**发送消息与正常向队列发送消息无异**

# 代码地址

> `github`示例代码地址：https://github.com/Devildyw/RabbitMQ-Study