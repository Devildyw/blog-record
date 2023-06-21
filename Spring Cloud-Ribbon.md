# Spring Cloud-Ribbon

## 简介

Spring Cloud Ribbon是基于NetFlix Ribbon实现的一套**客户端 均衡负载工具**

Ribbon = 负载均衡 + RestTemplate

简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法和服务调用。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面的所有机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们很容易使用Ribbon实现自定义的负载均衡算法。

> [Netflix-Ribbon-wiki](https://github.com/Netflix/ribbon/wiki)
>
> 注意：Ribbon已经进入了维护阶段，Spring Cloud意向使用`LoadBalancer`替换

## LB负载均衡（Load Balance）是什么

简单的说就是将用户的请求平摊的分配到多个服务上，从而达到HA（高可用）。

常见的负载均衡右软件`Nginx`、`LVS`、硬件 F5等。

### 集中式LB

即在服务的消费方和提供方之间使用独立的LB设施（可以是硬件，如F5，也可以是软件，如Nginx），由该设施负责把访问请求通过某种策略转发至服务的提供方。

### 进程内LB

将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。

**Ribbon就属于进程内LB**，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。

## Ribbon本地负载均衡客户端 VS Nginx服务端负载均衡区别

Nginx是服务器负载均衡，客户端所有请求都会将给nginx，然后由nginx实现转发请求。即负载均衡是由服务端实现的。（被动 消费方被动）

Ribbon本地负载均衡，在调用微服务接口时候，会在注册中心上获取注册信息服务列表之后缓存到JVM本地，从而在本地实现RPC远程调用技术。（主动 消费方主动）

## 总结

Ribbon其实就是一个软负载均衡的客户端软件，它可以和其他所需请求的客户端结合使用，和Eureka结合只是其中的一个实例。

## 负载均衡流程

[![image-20220812124511348](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121245308.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121245308.png)

[![image-20220812125354619](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121253697.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121253697.png)

1. 发起请求，请求发送到了服务提供者的负载均衡拦截器上。
2. 通过Ribbon的负载均衡客户端通过服务名称向注册中心请求服务列表。
3. 获取到服务列表后，通过设定的负载均衡策略去选出一个服务，获取服务信息。
4. 将获取到的服务信息替换原来的请求中的服务名称。
5. 发送真正有意义的http请求。

[![image-20220812130247274](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121302320.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121302320.png)

## Pom依赖

```XML
<!-- https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-starter-netflix-ribbon -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
    <version>2.2.10.RELEASE</version>
</dependency>
```

在之前的入门项目Eureka案例中我们并没有显式的添加Ribbon依赖，但是依旧可以使用负载均衡的原因就是我们引入的`spring-cloud-starter-netflix-eureka-client`中自带了Ribbon，consul，zookeeper也是如此（新版本的Spring Cloud中所有原本自带Ribbon的包都被换为了Spring Cloud自己的Load Balancer）

## RestTemplate

> [restTemplate | Devil的个人博客 (devildyw.github.io)](https://devildyw.github.io/2022/04/16/restTemplate/#RestTemplate)

## Ribbon核心组件IRule

```JAVA
public interface IRule{
    /*
     根据键 @return 选择的服务器对象从 lb.allServers 或 lb.upServers 中选择一个活动服务器。如果没有可用的服务器，则返回 NULL
     */

    public Server choose(Object key);
    
    public void setLoadBalancer(ILoadBalancer lb);
    
    public ILoadBalancer getLoadBalancer();    
}
```

### Ribbon默认自带的负载策略

Ribbon的负载均衡鬼册是一个叫做IRule的接口来定义，每一个子接口都是一种规则。

[![image-20220812130331520](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121303598.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121303598.png)

IRule：根据特定算法中从服务列表中选取一个要访问的服务。

| 类名                                          | 策略                                                         |
| --------------------------------------------- | ------------------------------------------------------------ |
| **`com.netflix.loadbalancer.RoundRobinRule`** | 轮询                                                         |
| **`com.netflix.loadbalancer.RandomRule`**     | 随机                                                         |
| **`com.netflix.loadbalancer.RetryRule`**      | 先按照RoundRobinRule的策略获取服务，如果获取服务失败则在指定时间内会进行重试，获取可用的服务。 |
| **`WeightResponseTimeRule`**                  | 对RoundRobinRule的扩展，响应速度越快的实例选择权重越大，越容易被选择。 |
| **`BestAvailableRule`**                       | 会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。 |
| **`AvailabilityFilteringRule`**               | 先过滤掉故障实例，再选择并发较小的实例                       |
| **`ZoneAvoidanceRule`**                       | 默认规则，复合判断Server所在区域的性能和server的可用性选择服务器 |

### 如何替换负载均衡策略

修改工程`Cloud-eureka-consumer-order80`

修改Ribbon负载均衡策略有两种配置方式，配置类方式和YAML方式。

#### 方式一 创建配置类

编写配置类替换的负载均衡策略。配置负载均衡策略，就是将对应的IRule的Bean添加到 Spring 容器中。

```JAVA
MySelfRule

@Configuration
public class MyselfRule {
    @Bean
    public IRule myRule(){
        return new RandomRule(); //定义为随机策略 默认是轮询
    }
}
```

该方式会针对所有该消费者调用的服务进行策略修改。

也可以一通过在主启动类上添加@RibbonClient注解为服务指定负载均衡策略。

```JAVA
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MyselfRule.class) //指定那个服务使用那种配置类中的策略
```

修改主启动类

```JAVA
@EnableEurekaClient
@SpringBootApplication
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MyselfRule.class) //指定那个服务使用那种配置类中的策略
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class, args);
    }
}
```

#### 方式二 YAML

配置文件，为指定服务配置负载均衡策略。前缀为服务名称。

```YML
CLOUD-PAYMENT-SERVICE:
  ribbon:
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule #负载均衡规则
```

这里我们就为`cloud-payment-service`服务配置了随机的负载均衡规则。

#### 启动测试 `http://localhost:80/consumer/payment/get/1547134537463402497`

```JSON
{
    "code": 200,
    "msg": "查询成功,serverPort:8001",
    "data": {
        "id": 1547134537463402497,
        "serial": "100"
    }
}
{
    "code": 200,
    "msg": "查询成功,serverPort:8001",
    "data": {
        "id": 1547134537463402497,
        "serial": "100"
    }
}
```

连续访问两次都是8001服务响应说明已经不再是轮询机制 而是随机。

## Ribbon默认负载均衡算法–轮询原理

负载均衡算法：rest接口第几次请求数%服务器集群总数量 = 实际调用服务器位置下标，每次服务重启后rest接口计数从1开始。

### 源码

```JAVA
package com.netflix.loadbalancer;

import com.netflix.client.config.IClientConfig;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * 最广为人知的基本负载均衡策略，即 Round Robin Rule。
 *
 * @author stonse
 * @author Nikos Michalakis <nikos@netflix.com>
 *
 */
public class RoundRobinRule extends AbstractLoadBalancerRule {

    private AtomicInteger nextServerCyclicCounter; //JDK自带的原子计数器 用来记录rest接口访问的次数
    private static final boolean AVAILABLE_ONLY_SERVERS = true;
    private static final boolean ALL_SERVERS = false;

    private static Logger log = LoggerFactory.getLogger(RoundRobinRule.class);

    public RoundRobinRule() {
        nextServerCyclicCounter = new AtomicInteger(0); //初始化计数器为0
    }

    public RoundRobinRule(ILoadBalancer lb) { 
        this();
        setLoadBalancer(lb);
    }
	//choose核心算法 进行负载均衡获取服务器
    public Server choose(ILoadBalancer lb, Object key) {
        //如果没有负载均衡的实例 即认为没有指定负载均衡策略 因为Ribbon中的每一个负载均衡策略都对应一个实例 这里直接返回null 计算不出获取那个服务器的信息
        if (lb == null) {
            log.warn("no load balancer");
            return null;
        }

        Server server = null;
        int count = 0;
        //给予10次查询服务器的机会 是避免获取服务失败，例如获取服务时，刚好这个服务宕机了，那么这次请求就废了
        while (server == null && count++ < 10) {
            List<Server> reachableServers = lb.getReachableServers(); //获取那些已启动且可访问的服务器。
            List<Server> allServers = lb.getAllServers(); //获取所有已知的服务器，包括可访问和不可访问。
            int upCount = reachableServers.size(); //计算出可访问服务器的数量
            int serverCount = allServers.size(); //计算出所有已知服务器

            if ((upCount == 0) || (serverCount == 0)) { //如果两个数量任意一个为0则代表没有可用服务器 直接返回null
                log.warn("No up servers available from load balancer: " + lb);
                return null;
            }

            int nextServerIndex = incrementAndGetModulo(serverCount); //根据所有已知服务器数量在通过计数器找到下一个轮询的节点的索引
            server = allServers.get(nextServerIndex); //根据索引获取到服务器节点实例

            if (server == null) { //如果为空
                /* Transient. */
                Thread.yield(); //让出当前线程使其重新进入cpu时间片竞选阶段
                continue; //线程重新执行后 进入下一个循环
            }

            if (server.isAlive() && (server.isReadyToServe())) { //如果不为空 服务器还处于存活状态且服务器是可访问的那么直接返回当前服务器节点
                return (server);
            }

            // Next. 如果不是则继续寻找 将server置为null 使其进入下一个循环
            server = null;
        }

        if (count >= 10) { //如果轮询次数大于10了都没有找到可用的服务器节点则返回错误信息
            log.warn("No available alive servers after 10 tries from load balancer: "
                    + lb);
        }
        //返回null
        return server;
    }

    /**
     * Inspired by the implementation of {@link AtomicInteger#incrementAndGet()}.
     *
     * @param modulo The modulo to bound the value of the counter.
     * @return The next value.
     */
    private int incrementAndGetModulo(int modulo) {
        for (;;) { //这里是一个乐观锁 自旋锁
            int current = nextServerCyclicCounter.get(); //获取计数器当前计数值
            int next = (current + 1) % modulo; //将当前计数器值加1在与传入服务器数量modulo取模则可计算出下一台轮询的服务器节点的索引
            if (nextServerCyclicCounter.compareAndSet(current, next)) //AtomicInteger从内存偏移两种取出数据 如果取出的数据与current一样就将内存中的数据改为next 
                return next;
        }
    }
	//核心算法
    @Override
    public Server choose(Object key) {
        return choose(getLoadBalancer(), key);
    }

    @Override
    public void initWithNiwsConfig(IClientConfig clientConfig) {
    }
}
```

### 手写轮询算法

**前提：**注释掉主启动类上的`@RibbonClient`和RestTemplate配置类中的`@LoadBalancer` 旨在取消Ribbon的负载均衡。

1. 在`Cloud-eureka-provider-payment8001` `Cloud-eureka-provider-payment8002` 新增接口

   ```JAVA
      @GetMapping(value = "/payment/lb")
      public String getPaymentLB(){
          return serverPort;
      }
   ```

2. 新建接口`loadBalancer`

   ```JAVA
   public interface LoadBalancer {
       ServiceInstance instances(List<ServiceInstance> serviceInstances); //获取服务器实例 实现负载均衡算法
   }
   ```

3. 实现接口 实现自己的轮询算法`MyLB` （根据Ribbon轮询源码）

   ```JAVA
   package com.dyw.springcloud.lb;
   
   import org.springframework.cloud.client.ServiceInstance;
   import org.springframework.stereotype.Component;
   
   import java.util.List;
   import java.util.concurrent.atomic.AtomicInteger;
   
   /**
    * @author Devil
    * @since 2022-07-24-19:57
    */
   @Component
   public class MyLB implements LoadBalancer{
       //初始化计数器
       private AtomicInteger atomicInteger = new AtomicInteger(0);
   
       //负载均衡轮询策略计算出的节点索引
       public final int getAndIncrement(){ 
           //当前节点
           int current;
           //下一个节点
           int next;
           do { //自旋锁
               current = this.atomicInteger.get();
               //防止Integer类型变量越界
               next = current >= Integer.MAX_VALUE ? 0 : current+1;
           }while (!this.atomicInteger.compareAndSet(current,next));//AtomicInteger从内存偏移两种取出数据 如果取出的数据与current一样就将内存中的数据改为next并且返回true 是为了保障线程安全而使用一种乐观锁
           System.out.println("*****next"+next);
           return next;
       }
       @Override
       public ServiceInstance instances(List<ServiceInstance> serviceInstances) {
           //通过负载均衡算法计算出的索引取 服务器列表取出对应索引的服务器实例并返回
           int index = getAndIncrement() % serviceInstances.size();
           return serviceInstances.get(index);
       }
   }
   ```

4. 控制器类

   ```JAVA
   @Slf4j
   @RestController
   @RequestMapping("consumer")
   public class OrderController {
   //    private static final String PAYMENT_URL = "http://localhost:8001";
       private static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";
   
       @Resource
       RestTemplate restTemplate;
   
       //这个则是我们自定义的负载均衡的接口
       @Resource
       private LoadBalancer loadBalancer;
   
       //必须有它才能获取注册中心中的服务器实例列表
       @Resource
       private DiscoveryClient discoveryClient;
   
       @GetMapping("payment/lb")
       public String getPaymentLB(){
           //首先获取指定服务的服务器实例列表
           List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
           //判断服务器列表是否为空 如果是直接返回空
           if (instances==null||instances.size()<=0){
               return null;
           }
           //通过自定义的负载均衡算法获取服务实例
           ServiceInstance serviceInstance = loadBalancer.instances(instances);
           //再获取服务器的实例的uri 通过这个uri借助resttemplate去访问服务 这也是我们去掉RestTemplate配置类上@LoadBalancer的原因 加上这个注解会让RestTemplate通过服务名称去访问服务 而我们使用的是uri 是ip端口的格式 服务器列表中是没有这样的服务的所以会报错
           URI uri = serviceInstance.getUri();
           return restTemplate.getForObject(uri+"/payment/lb", String.class);
       }
   
   }
   ```

5. 启动测试

   ```TEX
   8001
   8002
   8001
   8002
   ```

   轮询成功，算法成功。

## 饥饿加载

Ribbon默认是采用懒加载，即第一次访问时才回去创建LoadBalanceClient，请求时间会很长。

而饥饿加载则会在项目启动时创建，降低第一次访问的好事，通过下面配置开启饥饿加载：

```YML
ribbon:
  eager-load:
    enabled: true #开启饥饿加载 默认为false
    clients:
      - CLOUD-PAYMENT-SERVICE # 指定对CLOUD-PAYMENT-SERVICE这个服务进行饥饿加载
```

# 总结

1. Ribbon负载均衡规则
   - 规则接口时IRule
   - 默认实现时ZoneAvoidanceRule，根据zone选择服务列表，然后轮询
2. 负载均衡配置方式
   - 代码方式：配置灵活，但修改时需要重新打包发布
   - 配置方式：直观，方便，无序重新打包发布，但无法做全局配置。
3. 饥饿加载
   - 开启饥饿加载
   - 指定饥饿加载的微服务名称