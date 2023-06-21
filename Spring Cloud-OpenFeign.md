# Spring Cloud-OpenFeign

## 简介

> [Feign](https://github.com/OpenFeign/feign) is a declarative web service client. It makes writing web service clients easier. To use Feign create an interface and annotate it. It has pluggable annotation support including Feign annotations and `JAX-RS` annotations. Feign also supports pluggable encoders and decoders. Spring Cloud adds support for Spring `MVC` annotations and for using the same `HttpMessageConverters` used by default in Spring Web. Spring Cloud integrates Eureka, Spring Cloud `CircuitBreaker`, as well as Spring Cloud `LoadBalancer` to provide a load-balanced `http` client when using Feign.
>
> 翻译：
>
> Feign是一个声明式的`WebService`客户端。使用Feign能让编写Web Service客户端更加简单。
>
> 它的使用方法是定义一个服务接口然后在上面添加注解。Feign也支持可插拔式的编码器和解码器。Spring Cloud对Feign进行了封装，使其支持了Spring `MVC`标准注解和`HttpMessageConverters`。Spring Cloud 集成了 Eureka、Spring Cloud `CircuitBreaker` 以及 Spring Cloud `LoadBalancer`，在使用 Feign 时提供负载均衡的 `http` 客户端。

> 官网地址：[Spring Cloud-`OpenFeign`](https://docs.spring.io/spring-cloud-openfeign/docs/current/reference/html/)
>
> `github`源码地址：[spring-cloud/spring-cloud-`openfeign`](https://github.com/spring-cloud/spring-cloud-openfeign)

### Feign能干什么？

Feign旨在使编写Java `Http`客户端变得更加容易。

前面在使用Ribbon+`RestTemplate`时，例用`RestTemplate`对`Http`请求的封装处理，形成了一套模板化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装，我们只需要创建一个接口并使用注解的方式来配置它（以前时`DAO`接口上面标注Mapper注解，现在是一个微服务接口上面标注一个Feign注解即可），即可完成对服务提供方的接口绑定，简化了使用Spring Cloud Ribbon时，自动封装服务调用调用客户端的开发量。

传统 RestTemplate 远程调用存在的问题：

- 代码可读性差，编程体验不统一
- 参数复杂URL难以维护

------

**Feign集成了Spring Cloud中客户端负载均衡的组件**

例用`LoadBalancer`维护了服务的服务列表信息，并且通过轮询实现了客户端的负载均衡。**而与`LoadBalancer`不同的时，通过Feign需要定义服务绑定接口并且声明式的方法**，简单而优雅的实现了服务调用。

> `RestTemplate`和`OpenFeign`都是针对restful接口的远程调用 `OpenFeign`则相当于将远程调用的restful接口做了一个封装。

## `OpenFeign`使用

**宗旨：接口+注解** —> 微服务调用接口+@FeignClient

### 创建父工程

1. 创建工程`Cloud-05-OpenFeign`

2. 添加依赖

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.dyw</groupId>
       <artifactId>Cloud-05-OpenFeign</artifactId>
       <version>1.0-SNAPSHOT</version>
       
   
       <packaging>pom</packaging>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
           <junit.version>4.12</junit.version>
           <lombok.version>1.18.10</lombok.version>
           <log4j.version>1.2.17</log4j.version>
           <mysql.version>8.0.28</mysql.version>
           <druid.version>1.2.11</druid.version>
           <mybatis.spring.boot.version>2.1.1</mybatis.spring.boot.version>
           <mybatis-plus>3.5.2</mybatis-plus>
       </properties>
   
       <!--子模块继承之后，提供作用：锁定版本+子module不用谢groupId和version-->
       <dependencyManagement>
           <dependencies>
               <dependency>
                   <groupId>org.apache.maven.plugins</groupId>
                   <artifactId>maven-project-info-reports-plugin</artifactId>
                   <version>3.2.2</version>
               </dependency>
               <!--spring boot 2.2.2-->
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-dependencies</artifactId>
                   <version>2.6.8</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <!--spring cloud Hoxton.SR1-->
               <dependency>
                   <groupId>org.springframework.cloud</groupId>
                   <artifactId>spring-cloud-dependencies</artifactId>
                   <version>2021.0.3</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <!--spring cloud 阿里巴巴-->
               <dependency>
                   <groupId>com.alibaba.cloud</groupId>
                   <artifactId>spring-cloud-alibaba-dependencies</artifactId>
                   <version>2021.1</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <!--mysql-->
               <dependency>
                   <groupId>mysql</groupId>
                   <artifactId>mysql-connector-java</artifactId>
                   <version>${mysql.version}</version>
                   <scope>runtime</scope>
               </dependency>
               <!-- druid-->
               <dependency>
                   <groupId>com.alibaba</groupId>
                   <artifactId>druid-spring-boot-starter</artifactId>
                   <version>${druid.version}</version>
               </dependency>
               <!--mybatis-->
               <dependency>
                   <groupId>org.mybatis.spring.boot</groupId>
                   <artifactId>mybatis-spring-boot-starter</artifactId>
                   <version>${mybatis.spring.boot.version}</version>
               </dependency>
               <dependency>
                   <groupId>com.baomidou</groupId>
                   <artifactId>mybatis-plus-boot-starter</artifactId>
                   <version>${mybatis-plus}</version>
               </dependency>
               <!--junit-->
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-starter-test</artifactId>
                   <version>2.6.8</version>
               </dependency>
               <!--log4j-->
               <dependency>
                   <groupId>log4j</groupId>
                   <artifactId>log4j</artifactId>
                   <version>${log4j.version}</version>
               </dependency>
           </dependencies>
   
       </dependencyManagement>
   
       <build>
           <finalName>SpringCloud-Hello-01</finalName>
           <plugins>
               <plugin>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-maven-plugin</artifactId>
                   <configuration>
                       <fork>true</fork>
                       <addResources>true</addResources>
                   </configuration>
               </plugin>
           </plugins>
       </build>
   </project>
   ```

### 创建子工程

1. 创建工程`Cloud-consumer-openFeign-order80`

2. 添加依赖

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>Cloud-05-OpenFeign</artifactId>
           <groupId>com.dyw</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>Cloud-consumer-openFeign-order80</artifactId>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <!--openfeign-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-openfeign</artifactId>
           </dependency>
           <!--eureka-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
               <optional>true</optional>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
           </dependency>
           <dependency>
               <groupId>com.dyw</groupId>
               <artifactId>Cloud-api-commons</artifactId>
               <version>${project.version}</version>
           </dependency>
       </dependencies>
   
   </project>
   ```

3. `application.yml`配置

   ```YAML
   server:
     port: 81
   spring:
     application:
       name: cloud-order-service
     # 注册Eureka服务
   eureka:
     client:
       # Eureka服务注册中心会将自己作为客户端来尝试注册它自己
       register-with-eureka: false
       # 我们要访问注册中心的服务所以这里必须为true 获取注册中心的服务列表信息
       fetch-registry: true
       service-url:
         #设置与Eureka Server交互的地址查询服务和注册服务都需要依赖这个地址
         defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
   ```

   只作为访问服务，所以就不将服务注册进去了。

4. 主启动类

   ```JAVA
   @SpringBootApplication
   @EnableFeignClients //开启OpenFeign远程调用功能
   @EnableEurekaClient
   public class OderOpenFigenMain80 {
       public static void main(String[] args) {
           SpringApplication.run(OderOpenFigenMain80.class, args);
       }
   }
   ```

5. 业务类

   业务逻辑接口+@`FeignClient`配置调用provider服务

   新建`PaymentFeignService`接口并新增注解@`FeignClient`

   ```JAVA
   @FeignClient(value = "CLOUD-PAYMENT-SERVICE")
   public interface PaymentFeignService {
       @GetMapping("/payment/get/{id}")
       CommonResult getPaymentById(@PathVariable("id") Long id) ;
   }
   ```

   主要是基于SpringMVC的注解来声明远程调用的信息，比如：

   - 服务名称：userservice
   - 请求方式：GET
   - 请求路径：/user/{id}
   - 请求参数：Long id
   - 返回值类型：User

6. 控制器类

   `OpenFeignController`

   ```JAVA
   @RestController
   @Slf4j
   @RequestMapping("consumer")
   public class OpenFeignController {
       @Resource
       private PaymentFeignService paymentFeignService;
   
       @GetMapping("/payment/get/{id}")
       public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
           return paymentFeignService.getPaymentById(id);
   
       }
   }
   ```

7. 启动测试 （Eureka服务器，支付服务集群，该订单服务）`http://localhost:80/consumer/payment/get/1547118279208656901`

   ```JSON
   {
       "code": 200,
       "msg": "查询成功,serverPort:8001",
       "data": {
           "id": 1547118279208656901,
           "serial": "100"
       }
   }
   ```

   ```JSON
   {
       "code": 200,
       "msg": "查询成功,serverPort:8002",
       "data": {
           "id": 1547118279208656901,
           "serial": "100"
       }
   }
   ```

   测试访问成功。

8. 调用流程: **消费者Controller–>接口–>负载均衡–>生产者Controller–>生产者服务接口**

## `OpenFeign`超时控制

首先在`application.yml`中配置feign的超时

```YML
feign:
  client:
    config:
      default:
        connectTimeout: 5000 #连接超时的最大时限
        readTimeout: 1000 #访问服务的最大时限 访问服务超过这个时间就会报错 这里将其设置成1秒 测试超时
```

`Cloud-Payment-Service`集群服务中Controller新增restful接口

```JAVA
   @GetMapping("/feign/timeout")
   public String PaymentFeignTimeout(){
       try {
           TimeUnit.SECONDS.sleep(3);
       } catch (InterruptedException e) {
           throw new RuntimeException(e);
       }
       return serverPort;
   }
```

1. 将上述新增接口的方法签名加入到我们`OpenFeign`的订单服务的Feign接口中。

   ```JAVA
   @FeignClient(value = "CLOUD-PAYMENT-SERVICE")
   public interface PaymentFeignService {
       
       ......
   
       @GetMapping("/payment/feign/timeout")
       public String PaymentFeignTimeout();
   }
   ```

2. 控制器类新增restful接口

   ```JAVA
   @RestController
   @Slf4j
   @RequestMapping("consumer")
   public class OpenFeignController {
       @Resource
       private PaymentFeignService paymentFeignService;
   
       ......
           
       @GetMapping("/payment/feign/timeout")
       public String paymentFeignTimeout(){
           //openfeign-loadbalancer，客户端一般默认等待1秒钟
           return paymentFeignService.PaymentFeignTimeout();
       }
   }
   ```

3. 测试访问`http://localhost:80/consumer/payment/feign/timeout`

   [![image-20220726184850298](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220726184850298.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220726184850298.png)

 执行超时。

------

我们可以在默认客户端和命名客户端上配置超时。`OpenFeign` 使用两个超时参数：

- `connectTimeout`防止由于服务器处理时间长而阻塞调用者。
- `readTimeout`从连接建立时开始应用，在返回响应时间过长时触发。

通过该配置可以进行`OpenFeign`的超时时间

```YML
feign:
  client:
    config:
      default:
        connectTimeout: 5000 #连接超时的最大时限
        readTimeout: 5000 #访问服务的最大时限 访问服务超过这个时间就会报错
```

## Feign的配置

[![image-20220819170221388](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191702491.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191702491.png)

## `OpenFeign`日志级别

`OpenFeign`内置日志，可以打印一些在远程调用接口时的详细信息，方便开发者调试。

> `OpenFeign`提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解Feign中的`Http`请求细节。**即对Feign接口的调用情况的监控和输出。**

`OpenFeign`日志级别

> NONE：默认的，不显示任何日志
>
> BASIC：仅请求方法、URL、状态码及执行时间；
>
> HEADERS：除了BASIC中定义的信息，还有请求和响应头信息；
>
> FULL：除了HEADERS中定义的信息外，还有请求和响应正文及元数据。

配置`OpenFeign`日志级别有两种方式

方式一：

1. 全局配置

 通过配置类来配置 创建配置类`FeignConfig`

```JAVA
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel() {
        return Logger.Level.FULL;
    }
}
```

也可以在主启动的@EnableFeignClients中配置该类

```JAVA
@EnableFeignClients(defaultConfiguration = FeignClientConfiguration.class) 
```

1. 局部配置

   如果是局部配置，则把它放到@FeignClient这个注解中：

   ```JAVA
   @FeignClient(value = "userservice", configuration = FeignClientConfiguration.class) 
   ```

`application.yml`中配置日志显示级别

```YML
logging:
  level:
    # feign日志以声明级别监控那个接口 监控如下接口 并且控制台打印日志以debug级别打印 这样可以将所有日志信息都打印出
    com.dyw.springcloud.service.PaymentFeignService: debug
```

重启服务调用接口查看控制台日志

```json
http://localhost:81/consumer/payment/feign/timeout
```

控制台：

[![image-20220726190509735](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220726190509735.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220726190509735.png)

方式二：`application.yml`配置文件的方式

1. 全局生效：

   ```YML
   feign:
     client:
       config:
         default: # 这里用default就是全局配置,如果是写服务名称,则是针对某个微服务的配置
           loggerLevel: FULL #日志级别
   ```

2. 局部生效：

   ```YML
   feign:
     client:
       config:
         cloud-payment-service: # 这里用default就是全局配置,如果是写服务名称,则是针对某个微服务的配置
           loggerLevel: FULL #日志级别
   ```

## Feign的性能优化

Feign底层客户端实现：

- URLConnection：默认实现，不支持连接池
- Apache `HttpClient`：支持连接池
- `OKHttp`：支持连接池

因此优化Feign的性能主要包括：

1. 使用连接池代替默认的URLConnection
2. 日志级别，最好用basic或者none（日志越详细越影响性能）

### Feign的性能优化-连接池配置

Feign添加HttpClient的支持：

引入依赖：

```XML
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-httpclient</artifactId>
</dependency>
```

配置连接池：

```YML
feign:
  client:
    config:
      default:
        connectTimeout: 5000 #连接超时的最大时限
        readTimeout: 5000 #访问服务的最大时限 访问服务超过这个时间就会报错
  httpclient:
    enabled: true #开启feign对httpClient的支持
    max-connections: 200 #最大的连接数
    max-connections-per-route: 50 #每个路由的最大连接数 
```

`OkHttp`整合方式同理

## Feign的最佳实践

方式一（继承）：给消费者的FeignClient和提供者的Controller定义统一的父接口作为标准。

[![image-20220819173342471](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191733541.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191733541.png)

存在的问题：

- 服务紧耦合
- 父接口参数列表中的映射不会被继承

方式二（抽取）：将`FeignClient`抽取为独立模块，并且把接口有关的`POJO`、默认的Feign配置都放到这个模块中，提供给所有消费者使用

[![image-20220819173453427](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191734490.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191734490.png)

与 Dubbo 接口的方式相似。

**常用第二种**

### 抽取FeignClient

实现最佳实践方式二的步骤如下：

- 首先创建一个module，命名为feign-api，然后引入feign的starter依赖
- 将order-service中编写的`UserClient`、User、`DefaultFeignConfiguration`都复制到`feign-api`项目中
- 在order-service中引入feign-api的依赖
- 修改order-service中的所有与上述三个组件有关的import部分，改成导入feign-api中的包
- 重启测试

当定义的`FeignClient`不在`SpringBootApplication`的扫描包范围时，这些`FeignClient`无法使用。有两种方式解决：

方式一：指定`FeignClient`所在包

```JAVA
@EnableFeignClients(basePackages = "cn.itcast.feign.clients")
```

方式二：指定`FeignClient`字节码

```JAVA
@EnableFeignClients(clients = {UserClient.class})
```