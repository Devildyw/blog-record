# Spring Cloud-Sleuth

## 引言

在微服务框架中，一个由客户端发起的请求在后端系统中会经果多个不同的服务节点调用来协同产生最后的请求结果，每一个前端请求都会形成一条复杂的分布式服务调用链路，链路中的任何一环出现高延时或者错误都会引起整个请求最后的失败。因此，就需要一些能够帮助理解系统行为、分析系统性能问题的工具， 以便在系统发生故障的时候，快速定位和解决问题。这些工具就是`APM`（Application Performance Management）

[![image-20220811180900496](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111809578.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111809578.png)

复杂的分布式服务调用链路

[![image-20220811180918384](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111809444.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111809444.png)

## 简介

Spring Cloud Sleuth 能够跟踪你的请求和消息，以便你可以将该通信与相应的日志条目相关联。 你还可以将跟踪信息导出到外部系统以可视化延迟。

Spring Cloud Sleuth提供了一套完整的服务跟踪的解决方案

在分布式系统中提供追踪解决方案并且兼容支持了`zipkin`。

通过Seuth产生的调用链监控信息，可以得知微服务之间的调用链路，但监控信息只输出到控制台不方便查看。我们需要一个**图形化的工具zipkin**。Zipkin是Twitter开源的分布式跟踪系统，**主要用来收集系统的时许数据，从而追踪系统的调用问题。**

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111822397.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111822397.png)

> 官方`Github`地址：https://github.com/spring-cloud/spring-cloud-sleuth

## Zipkin搭建安装

**docker安装部署`Zipkin`**。

Spring Cloud 从F版起已不需要自己构建`Zipkin` server了，只需要调用jar包即可，所以用Docker部署更为简单，只需要一个命令即可安装

```
SH
docker run -d -p 9411:9411 openzipkin/zipkin
```

[![image-20220811183218576](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111832618.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111832618.png)

开启云服务器端口后，就算是部署完成了。

访问`http://ip:端口/zipkin/`即可看到`zipkin`图形化控制台。

## 术语

Spring Cloud Sleuth 借用了 Dapper 的术语。

1. **Span**

> 基本工作单元，表示调用链路来源。 span用一个64位的id唯一标识。除ID外，span还包含其他数据，例如描述、时间戳事件、键值对的注解（标签）， spanID、span父 ID等。 span被启动和停止时，记录了时间信息。初始化 span被为”rootspan”，该 span的 id和 trace的 ID相等。**通俗的理解span就是一次请求信息**

1. **Trace**

> 一组共享”rootspan”的 span组成的树状结构称为 trace， trace也用一个64位的 ID唯一标识， trace中的所有 span都共享该 trace的 ID

1. **Annotation/Event**

> 用于及时记录某个事件的存在

> 1. CS（ Client sent客户端发送）：客户端发起一个请求，该 annotation描述了span的开始。
> 2. SR（ server Received服务器端接收）：服务器端获得请求并准备处理它。如果用 SR减去 CS时间戳，就能得到网络延迟。
> 3. SS（ server sent服务器端发送）：该 annotation表明完成请求处理（当响应发回客户端时）。如果用 SS减去 SR时间戳，就能得到服务器端处理请求所需的时间。
> 4. CR（ Client Received客户端接收）： span结束的标识。客户端成功接收到服务器端的响应。如果 CR减去 CS时间戳，就能得到从客户端发送请求到服务器响应的所需的时间。

下图显示了Span和Trace在系统中的流转

[![跟踪信息传播](https://raw.githubusercontent.com/spring-cloud/spring-cloud-sleuth/main/docs/src/main/asciidoc/images/trace-id.jpg)](https://raw.githubusercontent.com/spring-cloud/spring-cloud-sleuth/main/docs/src/main/asciidoc/images/trace-id.jpg)

每个note色块代表着一个span，（有七个span - 从A到G），思考一下下面的note

```JAVA
Trace Id = X
Span Id = D
Client Sent
```

此note表明当前span将Trace Id设置为X并将span Id设置为D。此外，从 `RPC` 的角度来看，发生了 Client Sent 事件。

再思考一下下面note，

```JAVA
Trace Id = X
Span Id = A
(no custom span)

Trace Id = X
Span Id = C
(custom span)
```

您可以继续使用创建的span（带有no custom span指示的示例），也可以手动创建子span（带有custom span指示的示例）。

下图显示了span的父子关系的流转：

[![Parent child relationship](https://raw.githubusercontent.com/spring-cloud/spring-cloud-sleuth/main/docs/src/main/asciidoc/images/parents.jpg)](https://raw.githubusercontent.com/spring-cloud/spring-cloud-sleuth/main/docs/src/main/asciidoc/images/parents.jpg)

## 搭建链路监控

Spring Cloud Sleuth 的maven依赖

```XML
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-sleuth-zipkin</artifactId>
  <version>3.1.3</version>
</dependency>
```

服务消费者和服务生产者都新添加该依赖。

### 服务提供者

`Cloud-eureka-provider-payment8001`

修改`application.yml`

```YML
spring:
  zipkin:
    base-url: http://36.137.128.27:9411/
  sleuth:
    sampler:
      #采样率介于0到1之间，1则表示全部采集
      probability: 1
  ......     
```

修改业务类

```JAVA
@RestController
@Slf4j
@RequestMapping("payment")
public class PaymentController {
    
	......   

    @GetMapping("/zipkin")
    public String paymentZipkin()
    {
        return "hi ,i'am paymentzipkin server fall back，welcome to atguigu，O(∩_∩)O哈哈~";
    }

}
```

### 服务消费者

`Cloud-eureka-consumer-order80`

修改`application.yml`

```YML
spring:
  zipkin:
    base-url: http://36.137.128.27:9411/
  sleuth:
    sampler:
      #采样率介于0到1之间，1则表示全部采集
      probability: 1
  ......     
```

修改业务类

```JAVA
@Slf4j
@RestController
@RequestMapping("consumer")
public class OrderController {
//    private static final String PAYMENT_URL = "http://localhost:8001";
    private static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";

    @Resource
    RestTemplate restTemplate;

	......

    @GetMapping("/payment/zipkin")
    public String paymentZipkin()
    {
        String result = restTemplate.getForObject("http://localhost:8001"+"/payment/zipkin/", String.class);
        return result;
    }

}
```

启动eureka注册中心，生产者，消费者

调用接口`get: http://localhost:80/consumer/payment/zipkin`调用成功后，观察zipkin图形控制台信息。

[![image-20220811193308993](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111933098.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111933098.png)

我们选择根据服务名称查询，选择好对应的服务名后，点击`RUN QUERY` 即可获得每次请求的链路跟踪信息。

[![image-20220811193429168](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111934252.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111934252.png)

图上显示了每次调用的链路基本信息

点击SHOW还可以看到更多详细的信息。

[![image-20220811193542614](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111935689.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208111935689.png)

包括请求持续时间，链路上的服务数量，TraceID等等。