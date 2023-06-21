# Spring Cloud-Gateway

> Spring Cloud全家桶中有一个很重要的组件就是网关，在1.x版本中都是采用Zuul网关; 但在2.x版本中，zuul的升级一直跳票，Spring Cloud最后自己研发了一个网关替代Zuul。Gateway是原Zuul1.x版的替代。

[![Spring Cloud diagram](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/cloud-diagram-1a4cad7294b4452864b5ff57175dd983.svg)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/cloud-diagram-1a4cad7294b4452864b5ff57175dd983.svg)

> zuul官网：[zuul-wiki](https://github.com/Netflix/zuul/wiki)
>
> Gateway官网：[Spring Cloud Gateway](https://docs.spring.io/spring-cloud-gateway/docs/current/reference/html/)

------

## 简介

Gateway是在Spring生态系统之上构建的`API`网关服务，基于Spring 5，`Spring Boot2`和 Project Reactor等技术。

Gateway旨在提供一种简单而有效的方式来对`API`进行路由，以及提供一些强大的过滤器功能，例如：熔断、限流、重试等。

> This project provides an `API `Gateway built on top of the Spring Ecosystem, including: Spring 5, Spring Boot 2 and Project Reactor. Spring Cloud Gateway aims to provide a simple, yet effective way to route to `APIs` and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.
>
> 翻译：
>
> Spring Cloud Gateway 是Spring Cloud的一个全新项目，基于`Spring5.0`+`Spring Boot 2.0`和`Project Reactor `等技术开发的网关，它旨在为微服务提供一种简单有效的统一的`API`路由管理方式。例如：安全性、监控/指标和弹性。

Spring Cloud Gateway 作为Spring Cloud生态系统中的网关，目标是替代`Zuul`，在Spring Cloud 2.0以上的版本中，没有对新版本的`Zuul` 2.0以上最新高性能版本进行集成，仍然还是使用的`Zuul 1.x`非Reactor模式的老版本。而为了提升网关的性能，Spring Cloud Gateway是基于`WebFlux`框架实现的，而`WebFlux`框架底层则使用了高性能的Reactor模式通信框架Netty。

### 什么是网关

[![image-20220819180827895](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191810214.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191810214.png)

在微服务架构中，一个系统往往由多个微服务组成，而这些服务可能部署在不同机房、不同地区、不同域名下。这种情况下，客户端（例如浏览器、手机、软件工具等）想要直接请求这些服务，就需要知道它们具体的地址信息，例如 IP 地址、端口号等。

这种客户端直接请求服务的方式存在以下问题：

- 当服务数量众多时，客户端需要维护大量的服务地址，这对于客户端来说，是非常繁琐复杂的。
- 并不是所有服务都是公开的。
- 在某些场景下可能会存在跨域请求的问题。
- 身份认证的难度大，每个微服务需要独立认证。

**我们可以通过 API 网关来解决这些问题**

------

### API网关

API 网关是一个搭建在客户端和微服务之间的服务，我们可以在 API 网关中处理一些非业务功能的逻辑，例如权限验证、监控、缓存、请求路由等。

API 网关就像整个微服务系统的门面一样，是系统对外的唯一入口。有了它，客户端会先将请求发送到 API 网关，然后由 API 网关根据请求的标识信息将请求转发到微服务实例。

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/101P46212-0.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/101P46212-0.png)

------

### 网关能干嘛

1. 反向代理
2. 鉴权
3. 流量控制
4. 熔断
5. 日志监控
6. 负载均衡

 …….

**网关在服务中所处的位置**

[![image-20220728202107925](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220728202107925.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220728202107925.png)

### 为什么要选择Gateway

1. `netflix`不太靠谱，`zuul2.0`一直跳票，迟迟不发布

   - 一方面因为`Zuul1.0`已经进入维护阶段，而且Gateway是Spring Cloud团队研发的，是重点关注产品，而且很多功能`Zuul`都没有，用起来也非常的简单便捷。
   - Gateway是基于异步非阻塞模型上进行开发的，性能十分优秀。虽然`Netflix`早就发布了最新的`Zuul 2.x`，但Spring Cloud貌似没有整合计划。而且`Netflix`相关组件都宣布进入维护期；不知前景如何？
   - 多方面综合考虑Gateway是很理想的网关选择。

2. Spring Cloud Gateway具有如下特性

   - **基于Spring Framework 5，Project Reactor和Spring Boot 2.0+进行构建。**天生对Spring框架的支持好；
   - 动态路由：能够匹配任何请求属性；
   - 可以对路由执行Predicate（断言）和Filter（过滤器）；
   - 集成`Hystrix`的断路器功能；
   - 集成Spring Cloud服务发现功能；
   - 易于编写的Predicate（断言）和Filter（过滤器）；
   - 请求限流功能；
   - 支持路径重写；

3. Spring Cloud Gateway与`Zuul`的区别

   在Spring Cloud `Finchley`正式版之前,Spring Cloud的推荐网关是`Netflix`提供的`Zuul`：

   1. `Zuul1.x`，是一个基于阻塞I/O的`API` Gateway
   2. `Zuul1.x`基于`Servlet2.5`使用阻塞架构它不支持任何长连接(如 `WebSocket`)`Zuul`的设计模式和`Nginx`较像，每次I/O操作都是从工作线程中选择一个执行，请求线程被阻塞到工作线程完成，但是差别是`Nginx`用C++实现，`Zuul`用Java实现，而`JVM`本身会有第一次加载较慢的情况，是的`Zuul`的性能相对较差。
   3. `Zuul2.x`理念更先进，想基于Netty非阻塞和支持长连接，但Spring Cloud目前还没有整合。`Zuul2.x`的性能较`Zuul1.x`有较大提升。在性能方面，根据官方提供的基准测试，Spring Cloud Gateway的RPS（每秒请求数）是`Zuul`的1.6倍。
   4. Spring Cloud Gateway建立在Spring Framework 5、Project Reactor和Spring Boot 2 之上，使用非阻塞`API`。
   5. Spring Cloud Gateway 还支持 `WebSocket`，并且与Spring紧密集成拥有更好的开发体验。

### `Zuul1.x`模型

Spring Cloud中集成的`Zuul`版本，采用的是Tomcat容器，使用的是传统的`Servlet` IO处理模型

**`Servlet`的生命周期**

`Servlet`由`Servlet` container进行生命周期管理。

[![image-20220728213928259](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220728213928259.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220728213928259.png)

container启动时创建`servlet`对象并调用`servlet` `init()`进行初始化；

container运行时接受请求，并为每个请求分配一个线程（一般从线程池中获取空闲线程）然后调用`service()`。

container关闭时调用`servlet` `destory()`销毁`servlet`；

**上述模式的缺点：**

`servlet`是一个简单的网络IO模型，当请求进入`servlet` container时，`servlet` container就会为其绑定一个线程，并在**并发不高的场景下**这种模型是适用的。但是一旦高并发（比如使用`jmeter`进行高并发测压）,线程数量就会上涨，而线程资源代价是昂贵的（上线文切换，内存消耗大）严重影响请求的处理时间。在一些简单业务场景下，不希望为每个request分配一个线程（servlet模型），只需要一个或者几个线程就能应对极大并发的请求，这种业务场景下`servlet`模型没有优势。

所以`Zuul1.x`是基于`servlet`之上德一个阻塞式处理模型，即Spring实现了处理所有request请求的一个`servlet`（`DispatchServlet`）并由该`servlet`阻塞式处理请求。所以Spring Cloud `Zuul`无法摆脱`servlet`模型的弊端。

### Gateway模型

Spring Cloud Gateway是基于`WebFlux`框架实现的，而`WebFlux`框架底层则使用了高性能的Reactor模式通信框架Netty。

#### Spring `WebFlux`

> The original web framework included in the Spring Framework, Spring Web `MVC`, was purpose-built for the `Servlet API `and `Servlet` containers. The reactive-stack web framework, Spring` WebFlux`, was added later in version 5.0. It is fully non-blocking, supports [Reactive Streams](https://www.reactive-streams.org/) back pressure, and runs on such servers as Netty, Undertow, and `Servlet` 3.1+ containers.
>
> Both web frameworks mirror the names of their source modules ([`spring-webmvc`](https://github.com/spring-projects/spring-framework/tree/main/spring-webmvc) and [`spring-webflux`](https://github.com/spring-projects/spring-framework/tree/main/spring-webflux)) and co-exist side by side in the Spring Framework. Each module is optional. Applications can use one or the other module or, in some cases, both — for example, Spring `MVC` controllers with the reactive `WebClient`.
>
> 翻译：
>
> Spring Framework 中包含的原始 Web 框架 Spring Web `MVC` 是专门为 `Servlet API` 和 `Servlet` 容器构建的。反应式堆栈 Web 框架 Spring `WebFlux` 是在 5.0 版中添加的。它是完全无阻塞的，支持 [`Reactive Streams`](https://www.reactive-streams.org/)背压，并且可以在 Netty、Undertow 和 `Servlet` 3.1+ 容器等服务器上运行。
>
> 两个 Web 框架都反映了它们的源模块（[`spring-webmvc`](https://github.com/spring-projects/spring-framework/tree/main/spring-webmvc)和 [`spring-webflux`](https://github.com/spring-projects/spring-framework/tree/main/spring-webflux)）的名称，并在 Spring 框架中并存。每个模块都是可选的。应用程序可以使用一个或另一个模块，或者在某些情况下，两者都使用——例如，带有响应式的 Spring `MVC` 控制器`WebClient`。

传统的Web框架，比如说：`struts2`，`springmvc`等都是基于`Servlet API`与`Servlet`容器基础之上运行的。

在`Servlet3.1`之后有了异步非阻塞的支持。而`WebFlux`是一个典型非阻塞异步的框架，它的核心式基于Reactor的相关`API`实现的。相对于传统的web框架来说，它可以运行咋i注入Netty，Undertow和`Servlet3.1`的容器上。非阻塞式+函数式编程（`Spring5`必须让你使用`jdk1.8`）

Spring `WebFlux`是Spring 5.0引入的新的响应式框架，区别于Spring `MVC`，它不需要依赖`Servlet API`，他是完全异步非阻塞的，并且基于Reactor来实现响应式流规范。

## Gateway三大核心概念

1. **Route（路由）**

   路由是构建网关的基本模块，它由ID，目标URI，一系列的断言和过滤器组成，如果断言为true则匹配该路由

2. **Predicate（断言**）

   参考的是`java8`的`java.util.function.Predicate`开发人员可以匹配HTTP请求中的所有内容（例如请求头或请求参数），如果请求与断言相匹配则进行路由

3. **Filter（过滤）**

   指的是Spring框架中`GatewayFilter`的实例，使用过滤器，可以在请求被路由前或者之后对请求进行修改。

即Route（路由）包含了Predicate（断言）和Filter（过滤器），如果断言为真才会将请求转发到指定的服务上进行处理，Filter则作为路由成功请求前后对请求的一些列操作。

[![image-20220728222810287](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220728222810287.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220728222810287.png)

web请求，通过一些匹配条件，定位到真正的服务节点。并在这个转发过程的前后，进行一些精细化控制。

predicate就是我们的匹配条件

而filter，就可以理解为一个无所不能的拦截器。有了这个两个元素，再加上目标`uri`，就可以实现一个具体的路由了。

## Gateway工作流程

官网流程图

[![Spring Cloud Gateway Diagram](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/spring_cloud_gateway_diagram.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/spring_cloud_gateway_diagram.png)

> Clients make requests to Spring Cloud Gateway. If the Gateway Handler Mapping determines that a request matches a route, it is sent to the Gateway Web Handler. This handler runs the request through a filter chain that is specific to the request. The reason the filters are divided by the dotted line is that filters can run logic both before and after the proxy request is sent. All “pre” filter logic is executed. Then the proxy request is made. After the proxy request is made, the “post” filter logic is run.
>
> 翻译：
>
> 客户端向Spring Cloud Gateway 发出请求。然后在Gateway Handler Mapping中找到与请求相匹配的路由，将其发送到Gateway Web Handler。
>
> Handler再通过指定的过滤器链来将请求发送到我们实际的服务执行业务逻辑，然后返回。
>
> 过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。
>
> Filter在“pre”类型的过滤器可以做参数校验、权限校验、流量监控、日志输出、协议转化等，在“post”类型的过滤器中可以做响应内容、响应头的修改，日志的输出，流量监控等有着非常重要的作用。

总而言之，客户端发送到 Spring Cloud Gateway 的请求需要通过一定的匹配条件，才能定位到真正的服务节点。在将请求转发到服务进行处理的过程前后（pre 和 post），我们还可以对请求和响应进行一些精细化控制。

Predicate 就是路由的匹配条件，而 Filter 就是对请求和响应进行精细化控制的工具。有了这两个元素，再加上目标 URI，就可以实现一个具体的路由了。

------

## 搭建网关

1. 创建父工程`Cloud-07-Gateway`

   `pom.xml`依赖

   ```
   XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.dyw</groupId>
       <artifactId>Cloud-07-Gateway</artifactId>
       <version>1.0-SNAPSHOT</version>
   
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

2. 创建子工程`Cloud-gateway-gateway9527`

   引入SpringCloudGateway的依赖和nacos的服务发现依赖：

   ```
   XML
   <!--网关依赖-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-gateway</artifactId>
   </dependency>
   <!--nacos服务发现依赖-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId> 
   </dependency>
   ```

   `pom.xml`依赖

   ```
   XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>Cloud-07-Gateway</artifactId>
           <groupId>com.dyw</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>Cloud-gateway-gateway9527</artifactId>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <!--gateway-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-gateway</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <dependency>
               <groupId>com.dyw</groupId>
               <artifactId>Cloud-api-commons</artifactId>
               <version>1.0-SNAPSHOT</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-test</artifactId>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
       </dependencies>
   
   
   </project>
   ```

3. `application.yml`

   ```
   YML
   server:
     port: 9527 #网关端口
   spring:
     application:
       name: cloud-gateway #服务名称
     cloud:
       nacos:
         server-addr: 121.37.27.207:8848 #nacos地址
         username: nacos
         password: nacos
   ```

4. 主启动类`GatewayMain9527`

   ```
   JAVA
   @SpringBootApplication
   @EnableEurekaClient
   public class GatewayMain9527 {
       public static void main(String[] args) {
           SpringApplication.run(GatewayMain9527.class,args);
       }
   }
   ```

5. 为微服务配置网关

   为`cloud-provider-payment8001`配置网关，我们目前不想暴露8001端口，希望在8001外面套一层9527，**反向代理**；

   将请求发送到网关，网关根据路由断言再过滤器到指定的服务接口上。

   配置如下，`application.yml`中添加如下配置。

   ```
   YML
   cloud:
     gateway:
       routes:
         - id: payment_routh #payment_route   #路由的ID,没有固定规则但要求唯一,建议配合服务名
           uri: http://localhost:8001         #匹配后提供服务的路由地址
           predicates:
             - Path=/payment/get/**          #断言, 路径相匹配的进行路由
      
         - id: payment_routh2 #payment_route2   #路由的ID,没有固定规则但要求唯一,建议配合服务名
           uri: http://localhost:8001         #匹配后提供服务的路由地址
           predicates:
             - Path=/payment/lb/**          #断言, 路径相匹配的进行路由
   ```

   完整配置

   ```
   YML
   server:
     port: 9527 #网关端口
   spring:
     application:
       name: cloud-gateway #服务名称
     cloud:
       nacos:
         server-addr: 121.37.27.207:8848 #nacos地址
         username: nacos
         password: nacos
       gateway:
         discovery:
           locator:
             enabled: true
         routes:
           - id: payment_routh #payment_route   #路由的ID,没有固定规则但要求唯一,建议配合服务名
             uri: lb://CLOUD-PAYMENT-SERVICE        #匹配后提供服务的路由地址 lb就是负载均衡，后面是服务名称
             predicates:
               - Path=/payment/get/**          #断言, 路径相匹配的进行路由
   
           - id: payment_routh2 #payment_route2   #路由的ID,没有固定规则但要求唯一,建议配合服务名
             uri: lb://CLOUD-PAYMENT-SERVICE         #匹配后提供服务的路由地址
             predicates:
               - Path=/payment/lb/**          #断言, 路径相匹配的进行路由
   ```

   启动`Cloud-nacos-provider-payment8001` 、`Cloud-gateway-gateway9527`

   [![image-20220819183443749](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191834851.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191834851.png)

   测试反向代理

   `http://localhost:9527/payment/lb`

   ```
   JSON
   8001
   ```

   `http://localhost:9527/payment/get/1547504748557492225`

   ```
   JSON
   {
       "code": 200,
       "msg": "查询成功,serverPort:8001",
       "data": {
           "id": 1547504748557492225,
           "serial": "100"
       }
   }
   ```

**请求流程：**

[![image-20220819183754530](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191837604.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191837604.png)

### 编程式配置

通过配置类中配置`RouteLocator`来配置路由

```
JAVA
@Configuration
public class GateWatConfig {

    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        RouteLocator build = routes.route("path_routh_dyw",
                r -> r.path("/guonei")
                        .uri("http://news.baidu.com/guonei")).build();
        return build;
    }
}
```

这样配置之后 只要我们访问`localhost:9527/guonei`就会将请求转发到`http://news.baidu.com/guonei`上了。

编程式配置路由的核心代码

```
JAVA
routes.route("path_routh_dyw", //id
              r -> r.path("/guonei") //路径
                      .uri("http://news.baidu.com/guonei")) //转发路径
  
  	//函数签名
public Builder route(String id, Function<PredicateSpec, Buildable<Route>> fn) {
	Buildable<Route> routeBuilder = fn.apply(new RouteSpec(this).id(id));
	add(routeBuilder);
	return this;
}    
```

## 动态路由

默认情况下Gateway会根据注册中心的服务列表，以注册中心上微服务名为路径创建动态路由进行转发，从而实现动态路由的功能

> 需要注意的是uri的协议为lb，表示启用Gateway的负载均衡功能。
>
> lb://serviceName是spring cloud gateway在微服务中自动为我们创建的负载均衡uri

开启动态路由

修改`Cloud-gateway-gateway9527` `application.yml`文件

```
YML
gateway:
  discovery:
    locator:
      enabled: true
```

修改原路由uri为lb://serviceName的格式

```
YML
uri: lb://CLOUD-PAYMENT-SERVICE
```

完整配置

```
YML
server:
  port: 9527
spring:
  application:
    name: cloud-gateway
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: payment_routh #payment_route   #路由的ID,没有固定规则但要求唯一,建议配合服务名
          uri: lb://CLOUD-PAYMENT-SERVICE        #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/get/**          #断言, 路径相匹配的进行路由

        - id: payment_routh2 #payment_route2   #路由的ID,没有固定规则但要求唯一,建议配合服务名
          uri: lb://CLOUD-PAYMENT-SERVICE         #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**          #断言, 路径相匹配的进行路由
eureka:
  instance:
    instance-id: cloud-gateway-service
    prefer-ip-address: true
  client: #服务提供者provider注册进eureka服务列表内
    service-url:
      register-with-eureka: true
      fetch-registry: true
      defaultZone: http://eureka7001.com:7001/eureka
```

重新启动服务`Cloud-eureka-server7001`、`Cloud-eureka-provider-payment8001` 、`Cloud-gateway-gateway9527`、`Cloud-eureka-provider-payment8002`

测试 `http://localhost:9527/payment/lb`

```
JSON
8001
8002
8001
8002
......
```

测试成功。

------

## Predicate 断言

Spring Cloud Gateway 通过 Predicate 断言来实现 Route 路由的匹配规则。简单点说，Predicate 是路由转发的判断条件，请求只有满足了 Predicate 的条件，才会被转发到指定的服务上进行处理。

使用 Predicate 断言需要注意以下 3 点：

- Route 路由与 Predicate 断言的对应关系为“一对多”，一个路由可以包含多个不同断言。
- 一个请求想要转发到指定的路由上，就必须同时匹配路由上的所有断言。
- 当一个请求同时满足多个路由的断言条件时，请求只会被首个成功匹配的路由转发。

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/101P42B6-2.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/101P42B6-2.png)

Spring Cloud Gateway将路由匹配作为Spring WebFlux HandlerMapping基础架构的一部分。

### 断言工程

Spring Cloud Gateway包括许多内置的Route Predicate工厂。所有这些Predicate都与HTTP请求得不同属性匹配。多个Route Predicate工厂可以进行组合。

- **我们在配置文件中写的断言规则只是字符串，这些字符串会被Predicate Factroy读取并处理，转变为路由判断条件**
- 例如Path=/user/**是按照路径匹配，这个规则是由org.springframework.cloud.gateway.handler.predicate.PathRoutePredicateFactory类来处理的
- 像这样的断言工厂在SpringCloudGateway还有十几个

Spring Cloud Gateway创建Route对象时，使用RoutePredicateFactory创建Predicate对象，Predicate对象可以赋值给Route。Spring Cloud Gateway包含了许多内置得Route Predicate Factories。

[![image-20220729152720990](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729152720990.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729152720990.png)

> 所有这些为此都匹配HTTP请求得不同属性。多种谓词工厂可以组合，并通过逻辑and。

**常见的 Predicate 断言如下表**（假设转发的 URI 为 `http://localhost:8001`）。

[![image-20220819184349277](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191843358.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191843358.png)

| 断言       | 示例                                                         | 说明                                                         | 参数                                                  |
| ---------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| Path       | - Path=/dept/list/**                                         | 当请求路径与 /dept/list/** 匹配时，该请求才能被转发到 `http://localhost:8001` 上。 | 路径匹配字段                                          |
| Before     | - Before=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]        | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之前的请求，才会被转发到 `http://localhost:8001` 上。 | 一个参数datetime                                      |
| After      | - After=2021-10-20T11:47:34.255+08:00[Asia/Shanghai]         | 在 2021 年 10 月 20 日 11 时 47 分 34.255 秒之后的请求，才会被转发到 `http://localhost:8001` 上。 | 一个参数datetime                                      |
| Between    | - Between=2021-10-20T15:18:33.226+08:00[Asia/Shanghai],2021-10-20T15:23:33.226+08:00[Asia/Shanghai] | 在 2021 年 10 月 20 日 15 时 18 分 33.226 秒 到 2021 年 10 月 20 日 15 时 23 分 33.226 秒之间的请求，才会被转发到 `http://localhost:8001` 服务器上。 | 两个参数datetime                                      |
| Cookie     | - Cookie=name,c.biancheng.net                                | 携带 Cookie 且 Cookie 的内容为 name=c.biancheng.net 的请求，才会被转发到 `http://localhost:8001 `上。 | 两个参数，cookie的name和一个`Java`正则表达式          |
| Header     | - Header=X-Request-Id,\d+                                    | 请求头上携带属性 X-Request-Id 且属性值为整数的请求，才会被转发到 `http://localhost:8001` 上。 | 两个参数，标头和`Java`正则表达式                      |
| Method     | - Method=GET                                                 | 只有 GET 请求才会被转发到 `http://localhost:8001` 上。       | 一个参数，请求方法                                    |
| Host       | - Host=** .somehost.org, **.anotherhost.org                  | 只有请求的`Host`满足匹配规则才会被转发到`http://localhost:8001` 上。 | 一个参数，模式匹配的Host字符串                        |
| RemoteAddr | - RemoteAddr=192.168.1.1/24 （192.168.1.1是ip地址，24是子网掩码） | 只有请求的IP地址段在所设置的地址段中才会被转发`http://localhost:8001` 上。 | 一个参数，ip地址段                                    |
| Query      | - Query=green                                                | 只有请求的参数具有给定的名称且值与正则表达式匹配才会被转发`http://localhost:8001` 上。 | 两个参数，一个必须的param和一个可选的`Java`正则表达式 |
| Weight     |                                                              |                                                              | 权重处理                                              |

------

## Filter 过滤器

路由过滤器可用于修改进入得HTTP请求和返回HTTP响应，路由过滤器只能指定路由进行使用。

Spring Cloud Gateway内置了多种路由过滤器，他们都由GatewayFilter的工厂类来产生

**实际业务编写常常使用自定义过滤器。**

[![image-20220819184855424](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191848502.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191848502.png)

### 生命周期

- pre：在业务逻辑之前
- post：在业务逻辑之后

### 种类

- GatewayFilter：单一/全局
- GlobalFilter：全局

### 常用的GatewayFilter

Spring Cloud 自带的过滤器

#### 过滤器工厂

Spring提供了31种不同的路由过滤器工厂。例如：

[![image-20220819185058708](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191850785.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191850785.png)

配置过滤器

```
YML
spring:
  application:
    name: cloud-gateway #服务名称
  cloud:
    nacos:
      server-addr: 121.37.27.207:8848 #nacos地址
      username: nacos
      password: nacos
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: payment_routh #payment_route   #路由的ID,没有固定规则但要求唯一,建议配合服务名
          uri: lb://CLOUD-PAYMENT-SERVICE        #匹配后提供服务的路由地址 lb就是负载均衡，后面是服务名称
          predicates:
            - Path=/payment/get/**          #断言, 路径相匹配的进行路由
          filters:
            - AddRequestHeader=Truth, devildyw is freaking awesome! # 给请求添加请求头
```

#### 默认过滤器

如果要对所有的路由都生效，则可以将过滤器工厂写到default下。格式如下：

```
YML
server:
  port: 9527 #网关端口
spring:
  application:
    name: cloud-gateway #服务名称
  cloud:
    nacos:
      server-addr: 121.37.27.207:8848 #nacos地址
      username: nacos
      password: nacos
    gateway:
      discovery:
        locator:
          enabled: true
      routes:
        - id: payment_routh #payment_route   #路由的ID,没有固定规则但要求唯一,建议配合服务名
          uri: lb://CLOUD-PAYMENT-SERVICE        #匹配后提供服务的路由地址 lb就是负载均衡，后面是服务名称
          predicates:
            - Path=/payment/get/**          #断言, 路径相匹配的进行路由    
        - id: payment_routh2 #payment_route2   #路由的ID,没有固定规则但要求唯一,建议配合服务名
          uri: lb://CLOUD-PAYMENT-SERVICE         #匹配后提供服务的路由地址
          predicates:
            - Path=/payment/lb/**          #断言, 路径相匹配的进行路由
      default-filters:
        - AddRequestHeader=Truth, devildyw is freaking awesome! # 给请求添加请求头  
```

### 全局过滤器

全局过滤器的作用也是处理一切进入网关的请求和微服务响应，与GatewayFilter的作用一样。区别在于GatewayFilter通过配置定义，处理逻辑是固定的。而GlobalFilter的逻辑需要自己写代码实现。定义方式是实现GlobalFilter接口。

能干嘛？

- 全局日志记录
- 统一网关鉴权

#### 自定义GlobalFilter

修改`Cloud-gateway-gateway9527`**自定义过滤器实现接口`GlobalFilter，Ordered`**

**自定义过滤器的编写类似于Servlet的过滤器编写。**

1. 新建包`filter`

2. 新建类`MyLogGatewayFilter`实现GlobalFilter，Ordered接口

   ```
   JAVA
   /**
    * @author Devil
    * @since 2022-07-29-17:47
    *
    * 自定义全局过滤器演示类
    */
   @Slf4j
   @Component
   public class MyLogGatewayFilter implements GlobalFilter, Ordered {
       @Override
       public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
           log.info("***************com in MyLogGatewayFilter:"+new Date());
           //获取请求头中“uname”对应值
           String uname = exchange.getRequest().getQueryParams().getFirst("uname");
           if (uname==null){
               //如果为空 则返回响应 状态码为请求未被接受对应的状态码
               log.info("************用户名为null,非法用户  ┭┮﹏┭┮");
               exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
               //返回
               return exchange.getResponse().setComplete();
           }
           //如果不为空 放行请求 请求进入后续过滤器链
           return chain.filter(exchange);
       }
   
       @Override
       public int getOrder() {
           return 0;
       }
   }
   ```

   实现Ordered接口可以指定过滤器的执行顺序，也可通过@Order注解指定。

3. 启动测试

   测试接口`http://localhost:9527/payment/lb?uname=张三`

   ```
   JSON
   8001
   8002
   8001
   8002
   ```

   [![image-20220729180656549](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729180656549.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729180656549.png)

   测试接口`http://localhost:9527/payment/lb`

   [![image-20220729180854125](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729180854125.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729180854125.png)

   [![image-20220729180912942](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729180912942.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729180912942.png)

### 过滤器执行顺序

请求进入网关会碰到三类过滤器：当前路由的过滤器、DefaultFilter、GlobalFilter请求路由后，会将当前路由过滤器和DefaultFilter、GlobalFilter，合并到一个过滤器链（集合）中，排序后依次执行每个过滤器

**路由过滤器和DefaultFilter、GlobalFilter三者最终都会被转化为GatewayFilter**

[![image-20220819191225562](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191912636.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208191912636.png)

- 每一个过滤器都必须指定一个int类型的order值，**order值越小，优先级越高，执行顺序越靠前。**
- GlobalFilter通过实现Ordered接口，或者添加@Order注解来指定order值，由我们自己指定
- 路由过滤器和defaultFilter的order由Spring指定，默认是按照声明顺序从1递增。
- 当过滤器的order值一样时，会按照 defaultFilter > 路由过滤器 > GlobalFilter的顺序执行。

可以参考下面几个类的源码来查看：

```
JAVA
org.springframework.cloud.gateway.route.RouteDefinitionRouteLocator#getFilters()方法是先加载defaultFilters，然后再加载某个route的filters，然后合并。
JAVA
org.springframework.cloud.gateway.handler.FilteringWebHandler#handle()方法会加载全局过滤器，与前面的过滤器合并后根据order排序，组织过滤器链
```

## 跨域问题处理

跨域：域名不一致就是跨域，主要包括：

- 域名不同： [www.taobao.com](http://www.taobao.com/) 和 [www.taobao.org](http://www.taobao.org/) 和 [www.jd.com](http://www.jd.com/) 和 miaosha.jd.com
- 域名相同，端口不同：localhost:8080和localhost8081

跨域问题：浏览器禁止请求的发起者与服务端发生跨域ajax请求，请求被浏览器拦截的问题

解决方案：CORS

CORS跨域要配置的参数包括哪几个？

- 允许哪些域名跨域？
- 允许哪些请求头？
- 允许哪些请求方式？
- 是否允许使用cookie？
- 有效期是多久？

```
YML
spring:
  application:
    name: cloud-gateway #服务名称
  cloud:
    nacos:
      server-addr: 121.37.27.207:8848 #nacos地址
      username: nacos
      password: nacos
    gateway:
      globalcors: #全局跨域处理
        add-to-simple-url-handler-mapping: true #解决options请求被拦截问题
        cors-configurations:
          '[/**]':  #拦截一切请求
            allowedOrigins: # 允许那些网址的跨域请求
              - "http://localhost:8090"
              - "http://www.leyou.com"
            allowedMethods: # 允许的跨域ajax的请求方式
              - "GET"
              - "POST"
              - "DELETE"
              - "PUT"
              - "OPTIONS"
            allowedHeaders:
              - "*" # 允许在请求中携带的头信息
            allowCredentials: true # 是否允许携带cookie
            maxAge: 360000 # 这次跨域检测的有效期
```