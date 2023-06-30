# Eureka

> 注：Eureka中各个节点都是平等的，没有`ZK`中角色的概念，即使N-1个节点挂掉也不会影响其他节点的正常运行。

虽然Eureka已经停止维护了，但是并不代表我们不去学习它，理解它的思想也是后来为学习其他注册中心打下基础。

> 以下所有Demo都是基于上述入门案例改编。

## 服务调用出现的问题

- 服务消费者该如何获取服务提供者的地址信息？
- 如果有多个服务提供者，消费者该如何选择？
- 消费者如何得知服务提供者的健康状态？

[![image-20220812121848429](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121219787.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121219787.png)

### 服务治理

`Spring Cloud`封装`NetFlix`公司开发的`Eureka`模块来实现服务治理

那么什么事服务治理呢？

当服务较少的时候，可能我们根本不需要什么所谓的服务治理，会觉得这不就是中间商赚差价吗，为什么我能直接调用还要再中间加个服务治理呢？其实啊在传统的`RPC`框架中当服务多到一定程度时，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用服务治理，管理服务与服务之间的依赖关系，可以实现服务调用，负载均衡，熔断等，实现服务的注册与发现。

### 服务注册/发现

`Eureka`采用了`C/S`的设计架构，`Eureka Server`作为服务注册功能的服务器，他是服务注册中心。而系统中的其他微服务，使用`Eureka`的客户端连接到`Eureka Server`并维持心跳连接。这样系统的维护人员就可以通过`Eureka Server`来监控系统中各个微服务是否可以正常运行。

在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己的信息比如服务地址通讯地址等以别名的方式注册到注册中心上。另一方（消费者服务的提供者），以该别名的方式去注册中心上获取到实际的服务通讯地址（`ip`地址），然后在实现本地`RPC`调用`RPC`远程调用框架核心设计思想：在于注册中心，因为使用注册中心管理每个服务与每个服务之间的依赖关系（服务治理概念）。在任何`RPC`远程调用框架中，都会有一个注册中心存放服务地址相关信息（接口地址）。

注册中心会维护所有注册到注册中心上的健康的服务的信息，当有消费者消费对应服务时，注册中心会返回服务的`ip`地址等信息，消费者在通过这些信息去远程调用服务。

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/3956561052b9dc3909f16f1ff26d01bb.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/3956561052b9dc3909f16f1ff26d01bb.png)

[![image-20220812122041515](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121220609.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121220609.png)

8001、8003宕机后，注册中心通过心跳检测，会将这些服务信息剔除。

[![image-20220812122114480](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121221531.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121221531.png)

### Eureka组件

前面我们了解到，`Eureka`是`C/S`结构的，它有两个组件 **`Eureka Server` 和 `EurekaClient`**

#### Eureka Server

提供服务注册服务，就是我们所说的`Eureka`的服务端，提供服务治理的相关功能。

各个微服务节点通过配置启动后，会在`Eureka Server`中进行注册，这样`Eureka Server`中的服务注册表中将会存储所有可用的服务节点的信息，服务节点的信息可用在界面中直观的看到。

#### Eureka Client

通过注册中心进行访问，`Eureka`的客户端，提供了与`Eureka`服务端交互的功能。

他是一个Java客户端，用于简化与`Eureka Server`的交互，客户端同时也具备一个内置的，使用轮询（`round-robin`）负载均衡算法的负载均衡器（用于同一个服务下多个提供者的情况），在应用启动后，将会

向`Eureka Server`发送心跳（默认周期30秒）。如果`Eureka Server`在多个心跳周期内没有接收到某个节点的心跳，`Eureka Server`将会从服务注册表中把这个服务节点移除（默认90秒）。

### Eureka常用配置

#### Eureka Server

`Eureka Server`配置参数的格式：`eureka.server.xxx`。

- `enable-self-preservation`：

  - 表示注册中心是否开启服务的自我保护能力（后面会介绍）。

- `renewal-percent-threshold`：

  - 表示 Eureka Server 开启自我保护的系数，默认：0.85。

- `eviction-interval-timer-in-ms`：

  - 表示 `Eureka Server` 清理无效节点的频率，默认 60000 毫秒（60 秒）。

#### Eureka Instance

`Eureka Instance` 的配置参数格式：`eureka.instance.xxx`。

- `instance-id`：

  - 表示实例在注册中心注册的唯一ID。

- `prefer-ip-address`：

  - `true`：实例以 `IP` 的形式注册
  - `false`：实例以机器 `HOSTNAME` 形式注册

- `lease-expiration-duration-in-seconds`：

  - 表示 `Eureka Server` 在接收到上一个心跳之后等待下一个心跳的秒数（默认 90 秒），若不能在指定时间内收到心跳，则移除此实例，并禁止此实例的流量。
  - 此值设置太长，即使实例不存在，流量也能路由到该实例
  - 此值设置太小，由于网络故障，实例会被取消流量
  - **需要设置为至少高于 `lease-renewal-interval-in-seconds` 的值，不然会被误移除了。**

- `lease-renewal-interval-in-seconds`：

  - 表示 `Eureka Client` 向 `Eureka Server` 发送心跳的频率（默认 30 秒），如果在 `lease-expiration-duration-in-seconds` 指定的时间内未收到心跳，则移除该实例。

#### Eureka Client

Eureka Client 的配置参数格式：`eureka.client.xxx`。

- `register-with-eureka`：

  - 表示此实例是否注册到 Eureka Server 以供其他实例发现。在某些情况下，如果你不想自己的实例被发现，而只想发现其他实例，配置为 false 即可。

- `fetch-registry`：

  - 表示客户端是否从 Eureka Server 获取实例注册信息。

- `serviceUrl.defaultZone`：

  - 表示客户端需要注册的 Eureka Server 的地址。

#### 用到的其他参数

- `spring.application.name`：

  - 表示应用名称，在注册中心中显示的服务注册名称。

- `spring.cloud.client.ip-address`：

  - 获取客户端的 `IP` 地址。

**上面讲的 Eureka 某些参数都可以可以在 Eureka 控制台上面找到**

[![image-20220714140318475](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714140318475.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714140318475.png)

Eureka 控制台上面的其他参数都可以定制。

> Eureka详细配置：[Eureka详细配置](https://devildyw.github.io/2022/07/14/Eureka详细配置/)

### 搭建一个Eureka 服务端

根据前文我们可以知道要搭建一个`Eureka`服务器，需要用到`Eureka Server`

父工程的话`pom.xml`与上述入门案例一致。

创建Eureka服务端工程`Cloud-eureka-server7001`

项目结构：

[![image-20220714141129475](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714141129475.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714141129475.png)

1. `pom.xml`

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>Cloud-02-Eureka</artifactId>
           <groupId>com.dyw</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>Cloud-eureka-server7001</artifactId>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
           </dependency>
           <dependency>
               <groupId>com.dyw</groupId>
               <artifactId>Cloud-api-commons</artifactId>
               <version>1.0-SNAPSHOT</version>
           </dependency>
       </dependencies>
   </project>
   ```

2. `application.yml`

   ```YML
   server:
     port: 7001
   
   eureka:
     instance:
       hostname: localhost #eureka服务端的实例名称
     client:
       #false表示不向注册中心注册自己
       register-with-eureka: false
       #false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要去检索服务
       fetch-registry: false
       service-url:
         #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
         defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
   ```

3. 创建启动类

   ```JAVA
   @SpringBootApplication
   @EnableEurekaServer //表示该模块为Eureka的注册中心
   public class EurekaMain7001 {
       public static void main(String[] args) {
           SpringApplication.run(EurekaMain7001.class, args);
       }
   }
   ```

   需要在启动类的头上加上`@EnableEurekaServer`注解，表示该模块为Eureka服务端。

4. 启动

   启动后访问yml中的`http://${eureka.instance.hostname}:${server.port}/`地址看到如下界面表示成功。

   [![image-20220714141438302](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714141438302.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714141438302.png)

   **上述便是单机版Eureka注册中心的搭建了**

   ### 支付微服务8001注册到Eureka Server中

   将入门案例中的`Cloud-provide-payment-8001`复制粘贴到Eureka工程中并改名为`Cloud-eureka-provider-payment8001`业务部分不用更改 主要是`pom.xml`和`application.yml`的修改

   1. 改`pom.xml`新增依赖`spring-cloud-starter-netflix-eureka-client`

      ```XML
      <dependency>
          <groupId>org.springframework.cloud</groupId>
          <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
      </dependency>
      ```

   2. 改`application.yml` 新增Eureka配置

      ```YML
      eureka:
        client:
          #表示是否将自己注册进EurekaServer 默认为true
          register-with-eureka: true
          #是否从EurekaServer抓取已有的注册信息,默认为true.单节点无所谓,集群必须设置为true才能配合ribbon使用负载均衡
          fetch-registry: true
          service-url:
            defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka #要注册到的注册中心的地址
        instance:
          instance-id: payment8001 #指定实例名称
          prefer-ip-address: true #是否显示ip
      ```

   3. 启动类修改

      ```JAVA
      @SpringBootApplication
      @EnableEurekaClient //添加该注解提供与客户端的交互 这里是将服务注册到注册中心
      public class PaymentMain8002 {
          public static void main(String[] args) {
              SpringApplication.run(PaymentMain8002.class,args);
          }
      
      }
      ```

   4. 启动测试

      [![image-20220714154834760](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714154834760.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714154834760.png)

      可以通过`Eureka DashBoard`发现服务已经注册进了注册中心，实例的名称也是我们yml中指定的，服务名则是我们`spring.application.name`。

### 订单微服务80入驻进Eureka Server中

将入门案例中的`Cloud-consumer-order80`复制到Eureka工程中，改名为`Cloud-eureka-consumer-order80`我们要做的也是修改`pom.xml`和`application.yml`

1. 修改`pom.xml` 添加`spring-cloud-starter-netflix-eureka-client`依赖

   ```XML
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
   ```

2. 修改`application.yml` 添加Eureka相关配置

   ```YML
   eureka:
     client:
       #表示是否将自己注册进EurekaServer 默认为true
       register-with-eureka: false
       #是否从EurekaServer抓取已有的注册信息,默认为true.单节点无所谓,集群必须设置为true才能配合ribbon使用负载均衡
       fetch-registry: true
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
     instance:
       instance-id: order80 #指定实例id 不指定注册中心中显示的就是ip的格式
   ```

3. 启动类

   ```JAVA
   @EnableEurekaClient //添加该注解提供与客户端的交互 这里是消费注册中心中的服务
   @SpringBootApplication
   public class OrderMain80 {
       public static void main(String[] args) {
           SpringApplication.run(OrderMain80.class, args);
       }
   }
   ```

4. 修改业务类`Controller`

   注册到注册中心上的服务，需要使用服务提供者注册到注册中心的服务名称代替`ip地址:端口`的方式调用。单个提供者时，使用真实`ip地址:端口`与使用服务名称是没有区别的，但是当服务提供者是以集群的方式提供服务，那么这是想要使用负载均衡功能时，就必须使用这种方式了。均衡算法会返回一个正确`ip:端口`。

   `OrderController`

   ```JAVA
   //    private static final String PAYMENT_URL = "http://localhost:8001";
       private static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
   ```

   这里配置成服务名称其实也是一种规范，从一开始我们就提到了 **约定 > 配置 > 编码**

5. 启动测试

   `GET : http://localhost:80/consumer/payment/get/1547118279208656900`

   ```JSON
   {
       "code": 200,
       "msg": "查询成功",
       "data": {
           "id": 1547118279208656900,
           "serial": "10"
       }
   }
   ```

### Eureka集群原理说明

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/14570c4b7c4dd8653be6211da2675e45.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/14570c4b7c4dd8653be6211da2675e45.png)

**Eureka 集群，实际上就是启动多个 Eureka 实例，多个 Eureka 实例之间，互相注册，互相同步数据，共同组成一个 Eureka 集群。**

得到8个字**互相注册，相互守望**。

问题:微服务`RPC`远程服务调用最核心的是什么？
高可用，试想你的注册中心只有一个only one，万一它出故障了，会导致整个为服务环境不可用。

解决办法：搭建Eureka注册中心集群，实现负载均衡+故障容错。

### 搭建Eureka集群

本地机为了演示Eureka集群 需要修改电脑的hosts文件

[![image-20220714161642589](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714161642589.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714161642589.png)

原理我们已经直到，就是需要再搭建一个Eureka注册中心`Cloud-eureka-server7002`，让我们原来搭建的Eureka注册中心和现在这个相互注册，使其相互同步数据。

搭建步骤这里不再演示只说这里需要更改的地方

`Cloud-eureka-server7001`
`application.yml`
```YML
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: false
    #false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址 注册到eureka7002.com上
      defaultZone: http://eureka7002.com:7002/eureka/
```

------

`Cloud-eureka-server7002`
`application.yml`

```YML
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: false
    #false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址 注册到eureka7001.com上
      defaultZone: http://eureka7001.com:7001/eureka/
```

启动后观看Eureka Dashboard

```
http://eureka7001.com:7001/
```

[![image-20220714162757151](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714162757151.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714162757151.png)

```
http://eureka7002.com:7002/
```

[![image-20220714162843103](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714162843103.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714162843103.png)

可用发现两个注册中心的`DS Replicas`出现了对方的注册的实例名称，说明集群搭建成功。

这里我们启动`Cloud-eureka-provider-payment8001`使其`eureka7001.com`对应的注册中心上。

```
http://eureka7002.com:7002/
```

[![image-20220714163426520](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714163426520.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714163426520.png)

```
http://eureka7001.com:7001/
```

[![image-20220714163501059](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714163501059.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714163501059.png)

可用发现`http://eureka7002.com:7002/`上同样出现了我们注册的服务。这就是集群，集群的各个节点之间相互同步信息，防止单一节点宕机的问题。

------

### 支付微服务集群配置

#### 方式一

在实际开发中，不仅需要防止注册中心的单一节点宕机问题，服务提供者同样需要，不仅仅是为了防止宕机，同样也是为了提升服务的性能，服务提供者的集群不需要相互同步之间的信息，而是需要避免单一节点承受不住大量请求，导致反应慢或是请求失败等情况，同一服务新增节点集群 搭配上负载均衡，可以提升性能提高用户体验。

与Eureka搭建集群相似，没有什么特殊的改变，就是简单创建一个与之前服务相同工程`Cloud-eureka-provider-payment8002`

修改部分`application.yml`

```YML
server:
 port: 8002
 ... #其他部分相同这里省略
eureka:
  client:
    #表示是否将自己注册进EurekaServer 默认为true
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息,默认为true.单节点无所谓,集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: payment8002 #实例名称 如果不配置 到时Eureka注册中心中显示的就是IP的格式
    prefer-ip-address: true
```

#### 方式二

另外，我们可以将**`Cloud-eureka-provider-payment8001`**多次启动， 模拟多实例部署，但为了避免端口冲突，需要修改端口设置：

[![image-20220812123718765](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121237831.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121237831.png)

[![image-20220812123838314](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121238375.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121238375.png)

启动`Cloud-eureka-provider-payment8002`和`Cloud-eureka-provider-payment8001`以及Eureka注册中心。

[![image-20220714164450602](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714164450602.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714164450602.png)

可以发现同一个服务名称下面出现了两个服务实例，这也正是我们所配置的名称。

测试

**注意： 当服务以集群出现时如果采用了用服务名称代替`ip`+端口的格式的话 需要在`RestTemplate`配置类下配置`@LoadBalanced` 实现负载均衡 否则会出现访问报错**

`RestTemplateConfig`

```JAVA
@Configuration
public class RestTemplateConfig {
    @Bean
    @LoadBalanced //开启RestTemplate的负载均衡功能
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

并且为了方便查看负载均衡，我们在Controller的返回结果中加上了他们各自的端口

```JAVA
private final Integer serverPort = 8002; //Cloud-eureka-provider-payment8002
private final Integer serverPort = 8001; //Cloud-eureka-provider-payment8001
```

访问接口 `POST : http://localhost:80/consumer/payment/create`

```JSON
{
    "code": 200,
    "msg": "插入数据库成功,serverPort:8002",
    "data": 1
}
```

```JSON
{
    "code": 200,
    "msg": "插入数据库成功,serverPort:8001",
    "data": 1
}
```

### `DiscoveryClient`

对于注册进eureka里面的微服务，可以通过`DiscoveryClient`来获得该服务的信息

```
DiscoveryClient`提供了获取注册中心中注册服务信息的`API
```

**使用**

修改`Cloud-eureka-provider-payment8002`的`Controller` 新增

```JAVA
   @Resource
   private DiscoveryClient discoveryClient;

//新增接口 查看注册中心服务信息
   @GetMapping("/discovery")
   public Object discovery() {
       //获取注册中心里面的所有暴露的服务
       List<String> services = discoveryClient.getServices();
       for (String service : services) {
           log.info("*****element:{}",service);
       }
       //获取指定服务实例名称对应的实例信息
       List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
       for (ServiceInstance instance : instances) {
           log.info(instance.getServiceId()+"\t"+instance.getInstanceId()+"\t"+instance.getHost()+"\t"+instance.getUri()+"\t"+instance.getPort());
       }
       return this.discoveryClient;
   }
```

启动类新增注解`@EnableDiscoveryClient`

```JAVA
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient //添加该注解 使其该微服务支持Discovery
public class PaymentMain8002 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8002.class,args);
    }

}
```

启动测试

`GET : http://localhost:8002/payment/discovery`
```JSON
{
    "services": [
        "cloud-payment-service",
        "cloud-order-service"
    ],
    "order": 0
}
```

控制台：

[![image-20220714170357151](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714170357151.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714170357151.png)

### Eureka自我保护机制

**概述**

保护模式主要用于一组客户端和Eureka Server之间存在网络分区（由于网络波动等原因引起）场景下的保护。一旦进入保护模式，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是不会注销任何微服务。

**官方对于自我保护机制的定义**

> 自我保护模式正是一种针对网络异常波动的安全保护措施，使用自我保护模式能使Eureka集群更加的健壮、稳定的运行。

**自我保护机制的工作机制**

自我保护机制的工作机制是：**如果在15分钟内超过85%的客户端节点都没有正常的心跳，那么Eureka就认为客户端与注册中心出现了网络故障，Eureka Server自动进入自我保护机制**，此时会出现以下几种情况：

1. Eureka Server不再从注册列表中移除因为长时间没收到心跳而应该过期的服务。
2. Eureka Server仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上，保证当前节点依然可用。
3. 当网络稳定时，当前Eureka Server新的注册信息会被同步到其它节点中。

因此`Eureka Server`可以很好的应对因网络故障导致部分节点失联的情况，而不会像`ZK`那样如果有一半不可用的情况会导致整个集群不可用而变成瘫痪。

**为什么会产生Eureka自我保护机制?**

该功能防止节点因为网络波动导致心跳检测信息不能及时发送到注册中心，但节点本身没有问题的情况。如果关闭了自我保护机制，一旦检测到某个节点没有在指定时间内发送心跳包，就会将该节点剔除。

如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式:

[![image-20220714170747792](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714170747792.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714170747792.png)

从这个机制可以看出Eureka满足了CAP理论中的AP分支。即达到了100%可用性和100%分区容错性。

**什么是自我保护机制?**

默认情况下，如果`EurekaServer`在一定时间内没有接收到某个微服务实例的心跳，`EurekaServer`将会注销该实例(默认90秒)。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与`EurekaServer`之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。`Eureka`通过“自我保护模式”来解决这个问题——当`EurekaServer`节点在短时间内丢失过多客户端时(可能发生了网络分区故障)，那么这个节点就会进入自我保护模式。

自我保护机制∶默认情况下`EurekaClient`定时向`EurekaServer`端发送心跳包，如果Eureka在server端在一定时间内(默认90秒)没有收到`EurekaClient`发送心跳包，便会直接从服务注册列表中剔除该服务，但是在短时间( 90秒中)内丢失了大量的服务实例心跳，这时候`Eurekaserver`会开启自我保护机制，不会剔除该服务（该现象可能出现在如果网络不通但是`EurekaClient`为出现宕机，此时如果换做别的注册中心如果一定时间内没有收到心跳会将剔除该服务，这样就出现了严重失误，因为客户端还能正常发送心跳，只是网络延迟问题，而保护机制是为了解决此问题而产生的)。也正如官方所说的使用自我保护模式能使Eureka集群更加的健壮、稳定的运行。

------

### 自我保护开关

Eureka自我保护机制，通过配置 `eureka.server.enable-self-preservation` 来`true`打开/`false`禁用自我保护机制，默认打开状态，建议生产环境打开此配置。

如果要实现服务失效自动移除，只需要修改以下配置

##### 1、 注册中心关闭自我保护机制，修改检查失效服务的时间。

```YML
eureka:
  server:
     enable-self-preservation: false
     eviction-interval-timer-in-ms: 3000
```

##### 2、 微服务修改减短服务心跳的时间。

```BASH
# 默认90秒
lease-expiration-duration-in-seconds:  10

# 默认30秒
lease-renewal-interval-in-seconds:  3
```

> 以上配置建议在生产环境使用默认的时间配置。

根据如上我们可以根据需求关闭注册中心的自我保护机制。

`Cloud-eureka-server7001`
`application.yml`
```YML
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己
    register-with-eureka: false
    #false表示自己端就是注册中心,我的职责就是维护服务实例,并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
      defaultZone: http://eureka7002.com:7002/eureka/
  server:
    #关闭节点自我保护机制 默认是开启 关闭后如果在有限的心跳检测时间范围内节点没有及时发送心跳包 就将该服务节点从服务列表中踢出
    enable-self-preservation: false
    #设置心跳检测时间
    eviction-interval-timer-in-ms: 3000
```

`Cloud-eureka-provider-payment8002`
`application.yml`
```YML
server:
  port: 8002
	.... # 省略配置
eureka:
  client:
    #表示是否将自己注册进EurekaServer 默认为true
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息,默认为true.单节点无所谓,集群必须设置为true才能配合ribbon使用负载均衡
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka
  instance:
    instance-id: payment8002
    prefer-ip-address: true
    # Eureka客户端向服务端发送心跳的时间间隔,单位为秒(默认为30秒)
    lease-renewal-interval-in-seconds: 3
    #Eureka服务端在收到最后一次心跳后等待时间上限,单位为秒(默认是90秒),超时将剔除服务
    lease-expiration-duration-in-seconds: 10
	... #省略配置
```

启动`Cloud-eureka-server7001` 和`Cloud-eureka-provider-payment8002`

界面中会出现这么一句话

[![image-20220714173719274](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714173719274.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714173719274.png)

表示自我保护机制以及关闭

此时关闭`Cloud-eureka-provider-payment8002`Eureka会立刻剔除该服务。

### Eureka停更说明

[Netflix-eureka](https://github.com/Netflix/eureka/wiki)

> Eureka 2.0 (Discontinued)
>
> The existing open source work on eureka 2.0 is discontinued. The code base and artifacts that were released as part of the existing repository of work on the 2.x branch is considered use at your own risk.
>
> Eureka 1.x is a core part of Netflix’s service discovery system and is still an active project.

虽然Eureka停更了，但是`Spring Cloud`也有着许多其他功能更为丰富更为优秀的服务治理组件。