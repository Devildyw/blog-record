# Spring Cloud-Zookeeper

**`Spring Cloud`整合`Zookeeper`代替`Eureka`**

> 学习之前请先安装`Zookeeper`且开放端口 可参考:[`zookeeper`](https://devildyw.github.io/2022/07/14/Zookeeper/)

## 一. 创建父工程

1. 新建`module` `Cloud-03-Zookeeper`

2. 删除`src`文件目录

3. 添加`pom.xml`配置

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>com.dyw</groupId>
       <artifactId>Cloud-03-Zookeeper</artifactId>
       <packaging>pom</packaging>
       <version>1.0-SNAPSHOT</version>
       <modules>
           <module>Cloud-zookeeper-payment8004</module>
       </modules>
   
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

## 二. 服务提供者

1. 创建工程

2. 添加`pom.xml`依赖

   ```XML
   <dependency>
             <groupId>org.springframework.cloud</groupId>
             <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
             <version>3.1.2</version>
         </dependency>
   ```

   这里有个坑,一般`spring-cloud-starter-zookeeper-discovery`包都会自带`zookeepr`包的依赖,此时如果我们的云服务器中安装的`zookeeper`版本高于`spring-cloud-starter-zookeeper-discovery`自带的`zookeeper`版本,容易发生报错。

   [![image-20220723174411363](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220723174411363.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220723174411363.png)

   这里我们也能看出报错的`zookeeper`版本是3.6.0，而我们服务器中的版本是3.7.0，这里的报错是初始化连接`zookeeper`错误。

   **解决方案在`spring-cloud-starter-zookeeper-discovery`依赖中排除自带的`zookeeper`，手动添加与服务器中`zookeepr`版本相对应的版本。**

   ```XML
   
   		<dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
               <version>3.1.2</version>
               <exclusions>
                   <exclusion> <!--排除zookeeper-discovery自带的zookeeper 手动导入与服务器中zookeeper版本对应的zookeeper包-->
                       <groupId>org.apache.zookeeper</groupId>
                       <artifactId>zookeeper</artifactId>
                   </exclusion>
               </exclusions>
           </dependency>
   <!--        导入与服务器zookeeper版本对应的zookeeper-->
           <dependency>
               <groupId>org.apache.zookeeper</groupId>
               <artifactId>zookeeper</artifactId>
               <version>3.7.0</version>
           </dependency>
   ```

   完整的`pom.xml`文件

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>Cloud-03-Zookeeper</artifactId>
           <groupId>com.dyw</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>Cloud-zookeeper-payment8004</artifactId>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
   <!--        SpringBoot整合zookeeper客户端-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
               <version>3.1.2</version>
               <exclusions>
                   <exclusion> <!--排除zookeeper-discovery自带的zookeeper 手动导入与服务器中zookeeper版本对应的zookeeper包-->
                       <groupId>org.apache.zookeeper</groupId>
                       <artifactId>zookeeper</artifactId>
                   </exclusion>
               </exclusions>
           </dependency>
   <!--        导入与服务器zookeeper版本对应的zookeeper-->
           <dependency>
               <groupId>org.apache.zookeeper</groupId>
               <artifactId>zookeeper</artifactId>
               <version>3.7.0</version>
           </dependency>
           <dependency>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
               <version>1.2.17</version>
           </dependency>
           <dependency>
               <groupId>org.bouncycastle</groupId>
               <artifactId>bcprov-jdk15on</artifactId>
               <version>1.70</version>
           </dependency>
           <dependency>
               <groupId>com.google.guava</groupId>
               <artifactId>guava</artifactId>
               <version>31.1-jre</version>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
           <!--入门项目中用到的公用类包-->
           <dependency>
               <groupId>com.dyw</groupId>
               <artifactId>Cloud-api-commons</artifactId>
               <version>1.0-SNAPSHOT</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
       </dependencies>
   
   </project>
   ```

3. `application.yml`文件配置

   ```YML
   server:
     port: 8004
   spring:
     application:
       name: cloud-provider-payment
     cloud:
       zookeeper:
         connect-string: 36.137.128.27:2181 #在Zookeeper的学习中我们知道connectString是连接Zookeeper客户端的必要参数 它即zookeeper的ip:port地址
   ```

4. 启动类

   `PaymentMain8004`

   ```JAVA
   @SpringBootApplication
   @EnableDiscoveryClient //该注解用于向使用consul或者zookeeper作为注册中心时注册服务
   public class PaymentMain8004 {
       public static void main(String[] args) {
           SpringApplication.run(PaymentMain8004.class,args);
       }
   }
   ```

5. 控制器类`PaymentController`

   ```JAVA
   @RestController
   public class PaymentController {
       @Value("${server.port}")
       private String serverPort;
   
       @RequestMapping("payment/zk")
       public String paymentzk(){
           return "springcloud with zookeepr: "+serverPort+"\t"+ UUID.randomUUID().toString();
       }
   }
   ```

6. 启动查看是否服务注册进入了`Zookeeper`

   ```TEX
   [zk: localhost:2181(CONNECTED) 3] ls /
   [locks, servers, services, zookeeper]
   
   [zk: localhost:2181(CONNECTED) 0] ls /services
   [cloud-provider-payment]
   ```

   可用发现在`zookeeper`的根节点下出现了一个新的节点`services`并且在`/services`下出现了我们注册的服务的名字。说明我们注册成功。

7. 测试调用服务

   [![image-20220723174736745](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220723174736745.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220723174736745.png)

   [![image-20220723174721165](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220723174721165.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220723174721165.png)

   测试成功

8. 获取zookeeper节点信息

   我们的服务注册到了cloud-provider-payment下面但是一个服务下面可以有多个实例，我们注册到该服务名称的节点下会有一个长的序列编码对应我们的一个实例。

   ```TEX
   [zk: localhost:2181(CONNECTED) 2] ls -s  /services/cloud-provider-payment 
   [ece4ce6c-a457-4874-8b3c-09f465e57939]
   ```

   可以发现在`cloud-provider-payment`有一个长串的序列编码(**这个就是我们刚刚注册的服务实例**)

   通过`zookeeper`客户端命令`get 路径`获得节点信息。

   ```TEX
   [zk: localhost:2181(CONNECTED) 7] get  /services/cloud-provider-payment/ece4ce6c-a457-4874-8b3c-09f465e57939 
   ```

   ```JSON
   {"name":"cloud-provider-payment","id":"ece4ce6c-a457-4874-8b3c-09f465e57939","address":"192.168.101.9","port":8004,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"cloud-provider-payment","name":"cloud-provider-payment","metadata":{"instance_status":"UP"}},"registrationTimeUTC":1658569595889,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}
   ```

   可以发现信息是一个`Json`串。

   通过`Json`解析工具解析得到

   ```json
   {
   	"name": "cloud-provider-payment",
   	"id": "ece4ce6c-a457-4874-8b3c-09f465e57939",
   	"address": "192.168.101.9",
   	"port": 8004,
   	"sslPort": null,
   	"payload": {
   		"@class": "org.springframework.cloud.zookeeper.discovery.ZookeeperInstance",
   		"id": "cloud-provider-payment",
   		"name": "cloud-provider-payment",
   		"metadata": {
   			"instance_status": "UP"
   		}
   	},
   	"registrationTimeUTC": 1658569595889,
   	"serviceType": "DYNAMIC",
   	"uriSpec": {
   		"parts": [{
   			"value": "scheme",
   			"variable": true
   		}, {
   			"value": "://",
   			"variable": false
   		}, {
   			"value": "address",
   			"variable": true
   		}, {
   			"value": ":",
   			"variable": false
   		}, {
   			"value": "port",
   			"variable": true
   		}]
   	}
   }
   ```

   就可以得到该服务实例的信息(`ip`、端口、状态等信息)。

### 注册到`zookeeper`中的服务是什么类型的节点

1. 将我们注册的服务停掉过一小段时间后查看zookeeper中节点的情况

   ```TEX
   [zk: localhost:2181(CONNECTED) 7] ls /services/cloud-provider-payment/0535acc7-1da1-4dc7-bef2-d4a70538d145 
   []
   
   关掉服务后
   
   [zk: localhost:2181(CONNECTED) 8] ls /services/cloud-provider-payment/0535acc7-1da1-4dc7-bef2-d4a70538d145 
   Node does not exist: /services/cloud-provider-payment/0535acc7-1da1-4dc7-bef2-d4a70538d145
   ```

2. 所以注册到`zookeeper`中的服务实例是采用**临时节点**保存信息。

## 三. 服务消费者

1. 创建工程

2. `pom.xml`中添加依赖

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>Cloud-03-Zookeeper</artifactId>
           <groupId>com.dyw</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>Cloud-zookeeper-order80</artifactId>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-web</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-starter-actuator</artifactId>
           </dependency>
           <!--        SpringBoot整合zookeeper客户端-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
               <version>3.1.2</version>
               <exclusions>
                   <exclusion> <!--排除zookeeper-discovery自带的zookeeper 手动导入与服务器中zookeeper版本对应的zookeeper包-->
                       <groupId>org.apache.zookeeper</groupId>
                       <artifactId>zookeeper</artifactId>
                   </exclusion>
               </exclusions>
           </dependency>
           <!--        导入与服务器zookeeper版本对应的zookeeper-->
           <dependency>
               <groupId>org.apache.zookeeper</groupId>
               <artifactId>zookeeper</artifactId>
               <version>3.7.0</version>
           </dependency>
           <dependency>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
               <version>1.2.17</version>
           </dependency>
           <dependency>
               <groupId>org.bouncycastle</groupId>
               <artifactId>bcprov-jdk15on</artifactId>
               <version>1.70</version>
           </dependency>
           <dependency>
               <groupId>com.google.guava</groupId>
               <artifactId>guava</artifactId>
               <version>31.1-jre</version>
           </dependency>
           <dependency>
               <groupId>org.projectlombok</groupId>
               <artifactId>lombok</artifactId>
           </dependency>
           <!--入门项目中用到的公用类包-->
           <dependency>
               <groupId>com.dyw</groupId>
               <artifactId>Cloud-api-commons</artifactId>
               <version>1.0-SNAPSHOT</version>
           </dependency>
           <dependency>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-devtools</artifactId>
               <scope>runtime</scope>
               <optional>true</optional>
           </dependency>
       </dependencies>
   
   </project>
   ```

3. `application.yml`文件配置

   ```YML
   server:
     port: 80
   spring:
     application:
       name: cloud-consumer-order
     cloud:
       zookeeper:
         connect-string: 36.137.128.27:2181
   ```

4. 主启动类

   ```JAVA
   @SpringBootApplication
   @EnableDiscoveryClient
   public class OrderMain80 {
       public static void main(String[] args) {
           SpringApplication.run(OrderMain80.class, args);
       }
   }
   ```

5. 配置类，将`RestTemlate`类配置加入到Spring容器中

   ```JAVA
   @Configuration
   public class ContextConfig {
       @Bean
       public RestTemplate restTemplate(){
           return new RestTemplate();
       }
   }
   ```

6. 控制器类`OrderZKController`

   ```JAVA
   @RestController
   public class OrderZKController {
       public static final String INVOKE_URL = "http://cloud-provider-payment";
   
       @Resource
       private RestTemplate restTemplate;
   
       @GetMapping("/consumer/payment/zk")
       public String paymentInfo(){
           String result = restTemplate.getForObject(INVOKE_URL + "/payment/zk", String.class);
           return result;
       }
   }
   ```

7. 启动生产者和消费者

   ```TEX
   [zk: localhost:2181(CONNECTED) 9] ls /services
   [cloud-consumer-order, cloud-provider-payment]
   ```

   可以看到生产者和消费者都被注册到了`zookeeper`中。

   访问消费者`http://localhost:80/consumer/payment/zk`

   [![image-20220723193407609](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220723193407609.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220723193407609.png)

   访问成功

## 生产者集群

与Eureka类似，只需要多个提供相同的服务的生产者注册到同一个服务名称下即可，如果要使用消费者的`RestTemplate`访问记得加上`@LoadBalanced`注解即可（负载均衡 如果不加会报错）。