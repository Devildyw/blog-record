# Spring Cloud

## 微服务架构理论入门

**什么是微服务**

> In short, the microservice architectural style is an approach to developing a single application as a suite of small services, each running in its own process and communicating with lightweight mechanisms, often an HTTP resource API. These services are built around business capabilities and independently deployable by fully automated deployment machinery. There is a bare minimum of centralized management of these services, which may be written in different programming languages and use different data storage technologies.——James Lewis and Martin Fowler (2014)
>
> 微服务架构是一种架构模式，它提倡将单一应用程序划分为成一组小的服务，服务之间互相协调，互相配合，为用户提供最终价值，每个服务运行在其独立的进程中，服务与服务间采用轻量级的通信机制互相协作（通常是基于HTTP协议的`RESTful API`）。每个服务都围绕着本业务进行构建，并且能够独立的部署到生产环境，类生产环境等，另外应当避免统一的，集中式的服务管理机制，对具体的一个服务而言，根据业务上下文，选择合适的语言，工具对其进行构建。

单体的架构不利于现在互联网的发展，举个栗子，假如你在某宝买了一件衣服，要去下订单，调用库存，支付，调用仓储和物流，收货成功了，给用户增加积分等等模块，一个一个的模块就是我们利用`springboot`开发的微服务，以前是一个单体应用，现在有很多模块，服务，就需要一种机制将多个服务管理起来，所以说`spingboot`就是一个一个的提供功能的微服务。

- 微服务是一种架构风格
- 一个应用拆分为一组小型服务
- 每个服务运行在自己的进程内，也就是可独立部署和升级
- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署
- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

### 服务架构的演变

#### 单体架构

[![image-20220811220445575](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112208750.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112208750.png)

**单体架构**：将业务的所有功能集中在一个项目中开发，打包成一个包部署。

**优点**：

- 架构简单
- 部署成本低

**缺点**：

- 耦合度高

#### 分布式架构

[![image-20220811220519757](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112208755.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112208755.png)

**分布式架构**：根据业务能力对系统进行拆分，每个业务模块作为独立项目开发，称为一个服务。

**优点**：

- 降低服务耦合
- 有利于服务升级拓展

#### 服务治理

[![image-20220811220836306](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112208373.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112208373.png)

分布式架构的要考虑的问题：

- 服务拆分粒度如何？
- 服务集群地址如何维护？
- 服务之间如何实现远程调用？
- 服务健康状态如何感知？

上述问题有着许多的解决方案，例如Dubbo等，但是最好的解决方案还是微服务方案。

#### 微服务

微服务是一种经过良好架构设计的分布式架构方案，微服务架构特征：

- 单一职责：微服务拆分力度更小，每一个服务都对应唯一的业务能力，做到单一职责，避免重复业务开发
- 面向服务：微服务对外暴露业务接口
- 自治：团队独立、技术独立、数据独立、部署独立
- 隔离性强：服务调用做好隔离、容错、降级，避免出现级联问题

[![image-20220811221641573](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112216628.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112216628.png)

#### 总结

单体架构特点？

- 简单方便，高度耦合，拓展性查，适合小型项目。例如学生管理系统

分布式架构特点？

- 松耦合，拓展性好，但架构复杂，难度大。适合大型互联网项目，例如：京东，淘宝

微服务：一种良好的分布式架构解决方案

### 微服务技术对比

#### 微服务结构

微服务这种方案需要技术框架来落地，全球的互联网公司都在积极尝试自己的微服务落地技术。在国内知名就是Spring Cloud 和 阿里巴巴的`Dubbo`。

[![image-20220811222613716](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112226780.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112226780.png)

#### 微服务技术对比

[![image-20220811223219706](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112232777.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112232777.png)

#### 企业需求

[![image-20220811223411028](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112234095.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112234095.png)

- `SpringCloud`+`Feign`

  - 使用Spring Cloud技术栈
  - 服务接口采用RestFul风格
  - 服务调用采用Feign方式

- `SpringCloudAlibaba`+`Feign`

  - 使用Spring Cloud Alibaba 技术栈
  - 服务接口采用RestFul风格
  - 服务调用采用Feign方式

- `SpringCloudAlibaba`+`Dubbo`

  - 使用 Spring Cloud Alibaba 技术栈
  - 服务接口采用 `Dubbo` 标准协议
  - 服务调用采用 `Dubbo` 方式

- `Dubbo`

  原始模式

  - 基于 `Dubbo` 老技术体系
  - 服务接口采用 `Dubbo` 协议标准
  - 服务调用采用 `Dubbo` 方式

------

## 简介

`Spring Cloud` 是分布式微服务架构的一站式解决方案，它提供了一套简单易用的编程模型，使我们能在 `Spring Boot` 的基础上轻松地实现微服务系统的构建。

`Spring Cloud` 被称为构建分布式微服务系统的“全家桶”，它并不是某一门技术，而是一系列微服务解决方案或框架的有序集合。它将市面上成熟的、经过验证的微服务框架整合起来，并通过 `Spring Boot` 的思想进行再封装，屏蔽调其中复杂的配置和实现原理，最终为开发人员提供了一套简单易懂、易部署和易维护的分布式系统开发工具包。

`Spring Cloud` 中包含了 `spring-cloud-config`、`spring-cloud-bus` 等近 20 个子项目，提供了服务治理、服务网关、智能路由、负载均衡、断路器、监控跟踪、分布式消息队列、配置管理等领域的解决方案。

简单的说`Spring Cloud`提供了一套完美的一站式分布式微服务解决方案。

`Spring Cloud` 本身并不是一个拿来即可用的框架，它是一套微服务规范，共有两代实现。

- `Spring Cloud Netflix` 是 `Spring Cloud` 的第一代实现，主要由 `Eureka`、`Ribbon`、`Feign`、`Hystrix` 等组件组成。
- `Spring Cloud Alibaba` 是 `Spring Cloud` 的第二代实现，主要由 `Nacos`、`Sentinel`、`Seata` 等组件组成。

------

- **Spring Cloud是国内使用最广泛的微服务框架。**
- **Spring Cloud 集成了各种微服务功能组件，并基于Spring Boot实现了这些组件的自动装配，从而提供了良好的开箱即用体验**

> 官网地址：https://spring.io/projects/spring-cloud

**但是微服务并不等同于Spring Cloud 还包含着`JenKins`、`Docker`、`K8s`等对微服务进行打包部署，才能够称为完成的微服务技术栈。**

### 基于分布式的微服务架构

满足那些维度？

支持起这些维度的具体技术？

SpringCloud官网给出了一张图

![Diagram](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/cloud-3.svg)

总结可以得到，一个完整的基于分布式的微服务的架构需要满足以下要求。

[![image-20220811225914725](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112259795.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112259795.png)

- 服务调用
- 服务降级
- 服务注册与发先
- 服务熔断
- 负载均衡
- 服务消息队列
- 服务网关
- 配置中心管理
- 自动化构建部署
- 服务监控
- 全链路追踪
- 服务定时任务
- 调度操作

由前面我们也得知`SpringCloud` = 分布式微服务架构的一站式解决方案，是多种微服务架构落地技术的集合体，俗称微服务全家桶。

------

## 服务拆分及远程调用

**服务拆分注意事项**

1. 不同微服务，不要重复开发相同业务
2. 微服务数据独立，不要访问其他微服务的数据库
3. 微服务可以将自己的业务暴露为接口，供其他微服务调用

[![image-20220811232752553](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112327615.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208112327615.png)

### 远程调用

远程调用方式分析

[![image-20220812120520298](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121205446.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121205446.png)

步骤：

1. 注册RestTemplate

   ```JAVA
    @Bean
   public RestTemplate restTemplate(){
       return new RestTemplate;
   }
   ```

2. 服务远程调用RestTemplate

   ```JAVA
   restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
   ```

#### 总结

1. 微服务调用方式
   1. 基于RestTemplate发起的http请求实现远程调用
   2. http请求左远程调用是与语言无关的调用，只要实现对方的ip、端口、接口路径、请求参数即可。

------

### 提供者与消费者

- 服务消费者：一次业务中，被其他微服务调用的服务。（提供接口给其他微服务）
- 服务消费者：一次业务中，调用其他微服务的五福。（调用其他微服务提供的接口）

[![image-20220812121337980](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121213019.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/202208121213019.png)

**思考：**

**服务 A 调用 服务 B，服务 B 调用服务 C，那么服务B是什么角色？**

其实消费者与提供者的角色是相对的，一个服务可以同时是服务的提供者和服务的消费者。

------

## 入门

### `SpringCloud`的版本选型

`SpringCloud`版本不能盲目选择，SpringCloud的选择与SpringBoot版本选择有关，但是我们不用烦恼如何去选择版本，因为官方已经帮我们列了一张表了。

[Spring Cloud官方文档](https://spring.io/projects/spring-cloud)

[![image-20220713233202322](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220713233202322.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220713233202322.png)

通过上面这张图我们可以很清楚地看到，`SpringCloud`版本与`Spring Boot`版本之间的对应关系。

不仅如此，当你选择某个版本的`SpringCloud`版本的文档查看时，Spring官方还会为你选择`SpringCloud`版本推荐最最合适的`SpringBoot`版本。

[![image-20220713233517123](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220713233517123.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220713233517123.png)

本篇博客所编写的Demo将用到如下组件版本

> - `SpringCloud - 2021.0.3`
> - `SpringBoot - 2.6.8`
> - `SpringCloud Alibaba - 2021.1`
> - `Java - Jdk1.8`
> - `Maven - 3.5+`
> - `Mysql - 5.7`

------

### `SpringCloud`组件停更说明

- 停更引发的“升级惨案”
  - 停更不停用
  - 被动修复bugs
  - 不再接受合并请求
  - 不再发布新版本
- `SpringCloud`升级
  - 服务注册中心
    - × `Eureka`
    - ✔ `Zookeeper`(`Dubbo`官方推荐注册中心)
    - ✔ `Consul`(`golang`语言编写)
    - ✔ `Nacos`(`SpringCloud Alibaba` `Spring`官方推荐)
  - 服务调用
    - ✔ `Ribbon`
    - ✔ `LoadBalancer`
  - 服务调用2
    - × `Feign`
    - ✔ `OpenFeign`
  - 服务降级
    - × `Hystrix`
    - ✔ `resilience4j`
    - ✔ `sentienl`
  - 服务网关
    - × `Zuul`
    - ! `Zuul2`(`SpringCloud Netflix` 还未发布 疑似胎死腹中)
    - ✔ `gateway`
  - 服务配置
    - × `Config`
    - ✔ `Nacos`
  - 服务总线
    - × `Bus`
    - ✔ `Nacos`

[Spring Cloud 文档 | 中文文档](https://docs.gitcode.net/spring/guide/spring-cloud/documentation-overview.html)

------

### Quick Start

一定要记住 **约定 > 配置 > 编码**

**工程结构**

[![image-20220713235303555](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220713235303555.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220713235303555.png)

1. 创建父工程管理子工程模块之间的版本依赖。

   `Cloud-01-HelloSpringCloud`

   `pom.xml`

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns="http://maven.apache.org/POM/4.0.0"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.dyw</groupId>
       <artifactId>Cloud-01-HelloSpringCloud</artifactId>
       <version>1.0-SNAPSHOT</version>
       <modules>
           <module>Cloud-provide-payment-8001</module>
           <module>Cloud-consumer-order80</module>
           <module>Cloud-api-commons</module>
       </modules>
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
               <!--spring boot 2.6.8-->
               <dependency>
                   <groupId>org.springframework.boot</groupId>
                   <artifactId>spring-boot-dependencies</artifactId>
                   <version>2.6.8</version>
                   <type>pom</type>
                   <scope>import</scope>
               </dependency>
               <!--spring cloud 2021.0.3-->
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

2. 创建子工程

   `Cloud-provide-payment-8001`服务生产者

   1. `pom.xml`

      ```XML
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns="http://maven.apache.org/POM/4.0.0"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <parent>
              <artifactId>Cloud-01-HelloSpringCloud</artifactId>
              <groupId>com.dyw</groupId>
              <version>1.0-SNAPSHOT</version>
          </parent>
          <modelVersion>4.0.0</modelVersion>
      
          <artifactId>Cloud-provide-payment-8001</artifactId>
      
          <properties>
              <maven.compiler.source>8</maven.compiler.source>
              <maven.compiler.target>8</maven.compiler.target>
              <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          </properties>
      
          <dependencies>
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
                  <groupId>com.baomidou</groupId>
                  <artifactId>mybatis-plus-boot-starter</artifactId>
              </dependency>
              <dependency>
                  <groupId>mysql</groupId>
                  <artifactId>mysql-connector-java</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.mybatis.spring.boot</groupId>
                  <artifactId>mybatis-spring-boot-starter</artifactId>
              </dependency>
              <dependency>
                  <groupId>com.alibaba</groupId>
                  <artifactId>druid-spring-boot-starter</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.projectlombok</groupId>
                  <artifactId>lombok</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-devtools</artifactId>
                  <scope>runtime</scope>
                  <optional>true</optional>
              </dependency>
              <dependency>
                  <groupId>com.dyw</groupId>
                  <artifactId>Cloud-api-commons</artifactId>
                  <version>${project.version}</version>
              </dependency>
          </dependencies>
      
      </project> 
      ```

   2. `application.yml`

      ```YML
      server:
        port: 8001
      
      spring:
        application:
          name: cloud-payment-service
        datasource:
          type: com.alibaba.druid.pool.DruidDataSource
          driver-class-name: com.mysql.jdbc.Driver
          username: username
          password: password
          url: jdbc:mysql://localhost:3306/springcloud?useSSL=false&characterEncoding=utf-8&userUnicode=true
          druid:
            initial-size: 5       # 初始线程数
            max-active: 20        # 最大线程数
            max-wait: 60000       # 最大等待时间
            time-between-eviction-runs-millis: 60000      # 最大空闲实践
            min-idle: 5
      
      mybatis-plus:
        mapper-locations:
          - classpath:mapper/*.xml
        type-aliases-package: com.dyw.springcloud.entity
        configuration:
          log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
      ```

   3. 建表语句

      ```SQL
      SET NAMES utf8mb4;
      SET FOREIGN_KEY_CHECKS = 0;
      
      -- ----------------------------
      -- Table structure for payment
      -- ----------------------------
      DROP TABLE IF EXISTS `payment`;
      CREATE TABLE `payment`  (
        `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
        `serial` varchar(200) CHARACTER SET utf8 COLLATE utf8_general_ci NULL DEFAULT NULL,
        PRIMARY KEY (`id`) USING BTREE
      ) ENGINE = InnoDB AUTO_INCREMENT = 1547166640708214786 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
      
      SET FOREIGN_KEY_CHECKS = 1;
      ```

   4. 编写实体类

      ```JAVA
      @Data
      public class Payment {
          private Long id;
      
          private String serial;
      }
      ```

   5. 统一返回结果类

      ```JAVA
      @Data
      @NoArgsConstructor
      @AllArgsConstructor
      public class CommonResult<T> {
          private Integer code;
      
          private String msg;
      
          private T data;
      
          public CommonResult(Integer code, String msg) {
              this(code, msg, null);
          }
      }
      ```

   6. `Mapper`

      ```JAVA
      @Mapper
      public interface PaymentMapper extends BaseMapper<Payment> {
      
      }
      ```

   7. 服务类

      `PaymentService`

      ```JAVA
      public interface PaymentService {
      
          int create(Payment payment);
      
          Payment getPaymentById(Long id);
      }
      ```

      `PayMentServiceImpl`

      ```JAVA
      @Service
      public class PaymentServiceImpl implements PaymentService {
          @Resource
          PaymentMapper paymentMapper;
      
          @Override
          public int create(Payment payment) {
              return paymentMapper.insert(payment);
          }
      
          @Override
          public Payment getPaymentById(Long id) {
              return paymentMapper.selectById(id);
          }
      }
      ```

   8. 控制器类

      `PaymentController`

      ```JAVA
      @RestController
      @Slf4j
      @RequestMapping("payment")
      public class PaymentController {
          @Resource
          private PaymentService paymentService;
      
          @PostMapping("create")
          public CommonResult create(@RequestBody Payment payment) {
              int result = paymentService.create(payment);
              log.info("*****插入结果:{}", result);
              if (result > 0) {
                  return new CommonResult(200, "插入数据库成功", result);
              } else {
                  return new CommonResult(444, "插入数据失败", null);
              }
          }
      
          @GetMapping("/get/{id}")
          public CommonResult getPaymentById(@PathVariable("id") Long id) {
              Payment payment = paymentService.getPaymentById(id);
              log.info("*****插入结果:{}", payment);
              if (payment != null) {
                  return new CommonResult(200, "查询成功", payment);
              } else {
                  return new CommonResult(444, "没有对应记录,查询ID:" + id, null);
              }
          }
      }
      ```

3. 创建子工程

   `Cloud-consumer-order80`

   1. `pom.xml`

      ```XML
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns="http://maven.apache.org/POM/4.0.0"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <parent>
              <artifactId>Cloud-01-HelloSpringCloud</artifactId>
              <groupId>com.dyw</groupId>
              <version>1.0-SNAPSHOT</version>
          </parent>
          <modelVersion>4.0.0</modelVersion>
      
          <artifactId>Cloud-consumer-order80</artifactId>
      
          <properties>
              <maven.compiler.source>8</maven.compiler.source>
              <maven.compiler.target>8</maven.compiler.target>
              <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
          </properties>
      
          <dependencies>
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

   2. `application.yml`

      ```YML
      server:
        port: 80 #没有太多的配置 因为这里的子模块无需其他功能 只要能够调用生产者提供的服务即可
      ```

   3. 实体类，和统一结构返回类与`Cloud-provide-payment-8001`相同

   4. `RestTemplate`配置类

      在`SpringCloud`中服务之间的调用是通过`RestTemplate`(底层是`http`)访问的。这个类无需导入，导入的`spring-boot-starter-web`包中包含了，只需要通过配置类，将其注入`Spring`容器即可使用。

      ```JAVA
      @Configuration
      public class RestTemplateConfig {
          @Bean
          public RestTemplate restTemplate() {
              return new RestTemplate();
          }
      }
      ```

   5. 控制器类

      `OrderController`

      ```JAVA
      @RestController
      @RequestMapping("consumer")
      public class OrderController {
          private static final String PAYMENT_URL = "http://localhost:8001";
      
          @Resource
          RestTemplate restTemplate;
      
          @RequestMapping("payment/create")
          public CommonResult<Payment> createPayment(@RequestBody Payment payment) {
              return restTemplate.postForObject(PAYMENT_URL + "/payment/create", payment, CommonResult.class);
          }
      
          @RequestMapping("payment/get/{id}")
          public CommonResult<Payment> getPayment(@PathVariable Long id) {
              return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
          }
      }
      ```

4. 启动测试

   当`Idea`检测到有多个服务运行时，会提供`Services`面板来帮助使用者管理服务。

   [![image-20220714001043018](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714001043018.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220714001043018.png)

   启动成功后，调用消费者的接口查看是否能够有预期返回值。

   `GET : http://localhost:80/consumer/payment/get/1547118279208656900`

   ```
   JSON
   {
       "code": 200,
       "msg": "查询成功",
       "data": {
           "id": 1547118279208656900,
           "serial": "10"
       }
   }
   ```

   `POST : http://localhost:80/consumer/payment/create`

   请求参数:

   ```
   JSON
   {
       "serial":100
   }
   ```

   结果：

   ```
   JSON
   {
       "code": 200,
       "msg": "插入数据库成功",
       "data": 1
   }
   ```

5. **优化重构工程**

   从前面的步骤来看我们可以知道，有一部分是两个子模块公用的，那就是实体类部分和统一结果返回类，可以复用的部分，我们不妨将他们抽离出两个子模块单独做成一个模块供其他子模块使用。除此之外还可以存放一些可以共用的工具类。

   1. `pom.xml`

      ```XML
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns="http://maven.apache.org/POM/4.0.0"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
          <parent>
              <artifactId>Cloud-01-HelloSpringCloud</artifactId>
              <groupId>com.dyw</groupId>
              <version>1.0-SNAPSHOT</version>
          </parent>
          <modelVersion>4.0.0</modelVersion>
      
          <artifactId>Cloud-api-commons</artifactId>
      
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
                  <groupId>org.projectlombok</groupId>
                  <artifactId>lombok</artifactId>
              </dependency>
              <dependency>
                  <groupId>cn.hutool</groupId>
                  <artifactId>hutool-all</artifactId>
                  <version>5.8.3</version>
              </dependency>
          </dependencies>
      
      </project>
      ```

      没有什么特别的配置，只需要满足提供共用的实体类和统一结果返回类即可。

   2. 剩下的便是将共用的类移入该模块中

   3. `maven : install` 将子模块上传到本地仓库 也可`maven : deploy`将其部署到远端仓库中

   4. 其他子模块导包

      ```XML
      <dependency>
                <groupId>com.dyw</groupId>
                <artifactId>Cloud-api-commons</artifactId>
                <version>${project.version}</version>
      </dependency>
      ```