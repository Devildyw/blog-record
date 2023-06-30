# Spring Cloud-Config

## 前言

**分布式系统面临的配置问题**

微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设置是必不可少的。

**Spring Cloud提供了`ConfigServer`来解决这个问题。**

## 简介

[![image-20220729182943814](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729182943814.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729182943814.png)

### **是什么？**

Spring Cloud `Config`为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为**各个不同微服务应用**的所有环境提供了一个**中心化的外部配置**。

### **能干嘛？**

- 集中管理配置文件

- 不同环境不同配置，动态化的配置更新，分环境部署比如`dev/test/prod/beta/release`

- 运行期间动态调整配置，不再需要在每个服务部署的机器上配置文件，服务回向配置中心统一拉取配置自己的信息

- 当配置发生变动，服务不需要重启即可感知到配置的变化并应用新的配置。

- **将配置信息以REST接口的形式暴露**

  post、curl访问刷新即可

### **怎么使用？**

Spring Cloud `Config`分为**客户端和服务端两个部分**。

服务端也称为**分布式配置中心，他是一个独立的微服务应用**，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口。

客户端则是通过指定的配置信息来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息配置，服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端供来方便的管理和访问配置内容。

> 分布式配置中心，将微服务们共有的配置信息集中管理起来，微服务通过在配置中心中获取这部分配置信息，特有的部分配置信息则自己维护。

## Spring Cloud Config配置总控中心搭建

> 由于`SpringCloud Config`默认使用Git来存储配置文件（也有其它方式，比如支持`svn`和本地文件，但最推荐的还是Git，而且使用的是`http/https`访问的形式），所以我们选择于`Github`整合。

1. 使用自己的账号在`Github`上创建一个名为`springcloud-config`的新`Repository`

2. 由上一步获得刚建的git远程仓库地址

   `git@github.com:Devildyw/springcloud-config.git`

3. 本地硬盘目录上新建git仓库并clone

   - 本地地址：`E:\config\SpringCloud`

4. 此时在本地E盘符下`E:\config\SpringCloud、springcloud-config`

   - 创建表示多环境的配置文件

   - 保存格式必须为`UTF-8`

   - 如果需要修改，此处模拟运维人员操作`git`和`github`

   - `application-dev.yml`

     ```YML
     config:
       info: "master branch,springcloud-config/config-dev.yml version=1"
     ```

   - `application-prod.yml`

     ```YML
     config:
       info: "master branch,springcloud-config/config-prod.yml version=1"
     ```

   - `application-test.yml`

     ```YML
     config:
       info: "master branch,springcloud-config/config-test.yml version=1"
     ```

   - 新建后`git push`上传到远程仓库

5. 新建父工程模块`Cloud-08-Config`

6. `pom.xml`依赖

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <modelVersion>4.0.0</modelVersion>
   
       <groupId>top.devildyw</groupId>
       <artifactId>Cloud-08-Config</artifactId>
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

7. 创建子工程模块`Cloud-config-center3344`

8. `pom.xml`依赖

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>Cloud-08-Config</artifactId>
           <groupId>top.devildyw</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>Cloud-config-center334</artifactId>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-config-server</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <dependency>
               <groupId>com.dyw</groupId>
               <artifactId>Cloud-api-commons</artifactId>
               <version>${project.version}</version>
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
               <scope>test</scope>
           </dependency>
   
       </dependencies>
   
   </project>
   ```

9. `application.yml`配置

   ```YML
   server:
     port: 3344
   spring:
     application:
       name: cloud-config-center #注册进Eureka服务器的微服务名称
     cloud:
       config:
         server:
           git:
             uri: https://github.com/Devildyw/springcloud-config.git #Github上面git仓库的名字
             skipSslValidation: true # 跳过 SSL 证书验证
           #### 搜索目录
             search-paths:
               - springcloud-config
   
         ####读取分支
         label: master
   
   # 服务注册
   eureka:
     client:
       service-url:
         defaultZone: http://eureka7001.com:7001/eureka/
   ```

10. 主启动类

    `ConfigCenterMain3344`

    ```JAVA
    @SpringBootApplication
    @EnableEurekaClient
    @EnableConfigServer //关键
    public class ConfigCenterMain3344 {
        public static void main(String[] args) {
            SpringApplication.run(ConfigCenterMain3344.class,args);
        }
    }
    ```

11. 启动测试`Cloud-config-center3344` `Cloud-eureka-server7001`

    访问`http://localhost:3344/master/application-dev.yml`

    ```YML
    config:
      info: master branch,springcloud-config/config-dev.yml version=1
    ```

    获取成功

## 配置读取规则

Spring Cloud官网

```PLAINTEXT
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

**`/{label}/{application}-{profile}.yml`**

**该方式最为推荐**

```HTTP
示例
http://localhost:3344/master/application-dev.yml
http://localhost:3344/master/application-test.yml
http://localhost:3344/master/application-prod.yml
```

**`/{application}-{profile}.yml`**

```HTTP
示例
http://localhost:3344/application-test.yml
http://localhost:3344/application-dev.yml
http://localhost:3344/application-prod.yml
```

**`/{application}-{profile}[/{label}]`**

```HTTP
示例
http://localhost:3344/application/dev/master
http://localhost:3344/application/test/master
http://localhost:3344/application/prod/master
```

该方式获取到的配置的返回结果也不相同。

```JSON
{
    "name": "application",
    "profiles": [
        "test"
    ],
    "label": "master",
    "version": "5bba7057164c82bf3f4857d8e3992c8187faadad",
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/Devildyw/springcloud-config.git/file:C:\\Users\\Devil\\AppData\\Local\\Temp\\config-repo-4675055522948308019\\application-test.yml",
            "source": {
                "config.info": "master branch,springcloud-config/config-test.yml version=1"
            }
        }
    ]
}
```

------

**其余两种就是`properites`配置文件的版本只是在后缀上有所区别。**

## `Bootstrap.yml`

我们平时编写`SpringBoot`项目都会使用`application.yml`配置文件来配置项目中所需要的配置。

### 加载顺序

若`application.yml` 和`bootstrap.yml` 在**同一目录下**：**`bootstrap.yml` 先加载 `application.yml`后加载**

`bootstrap.yml` 用于应用程序上下文的引导阶段。`bootstrap.yml`由父Spring `ApplicationContext`加载。

> `application.yml`是用户级的资源配置项
>
> `bootstrap.yml`是系统级的，**优先级更加高**

### 配置区别

**bootstrap是spring cloud的配置上下文加载。由spring-cloud-content包加载。**

**application是spring boot的配置加载。**

`bootstrap.yml` 和 `application.yml` 都可以用来配置参数。

`bootstrap.yml` **用来程序引导时执行，应用于更加早期配置信息读取。可以理解成系统级别的一些参数配置，这些参数一般是不会变动的。一旦`bootStrap.yml` 被加载，则内容不会被覆盖。**

`application.yml` **可以用来定义应用级别的， 应用程序特有配置信息，可以用来配置后续各个模块中需使用的公共参数等。**

### 属性覆盖问题

- 启动上下文时，Spring Cloud 会创建一个 Bootstrap Context，作为 Spring 应用的 Application Context 的父上下文。
- 初始化的时候，Bootstrap Context 负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的 Environment。Bootstrap 属性有高优先级，默认情况下，**它们不会被本地配置覆盖**。
- 也就是说如果加载的 `application.yml` 的内容标签与 bootstrap 的标签一致，**application 也不会覆盖 bootstrap**，而 `application.yml` 里面的内容可以动态替换。

### 为什么Spring Cloud `Config` Client使用`bootstrap.yml`来编写配置

**当使用Spring Cloud的时候，配置信息一般是从配置中心获取加载的，为了取得配置信息（比如密码等），你需要一些提早的或引导配置。因此把配置中心信息放在`bootstrap.yml`中，用来加载真正需要的配置信息。同时也不容易让这些配置信息被新的配置信息所覆盖。**

比如：

客户端的服务需要数据库的配置信息，而数据库的配置信息是统一由配置中心管理的，我们必须要现在`bootstrap.yml`配置了配置中心中的信息，才能在服务启动之前的程序引导阶段将类似数据库配置之类的信息加载。否则程序会因为缺少配置而报错。

## `Config`客户端配置与测试

1. 创建子工程`Cloud-config-client3355`

2. `pom.xml`依赖

   ```XML
   <?xml version="1.0" encoding="UTF-8"?>
   <project xmlns="http://maven.apache.org/POM/4.0.0"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
       <parent>
           <artifactId>Cloud-08-Config</artifactId>
           <groupId>top.devildyw</groupId>
           <version>1.0-SNAPSHOT</version>
       </parent>
       <modelVersion>4.0.0</modelVersion>
   
       <artifactId>Cloud-config-client3355</artifactId>
   
       <properties>
           <maven.compiler.source>8</maven.compiler.source>
           <maven.compiler.target>8</maven.compiler.target>
           <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
       </properties>
   
       <dependencies>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-config</artifactId>
           </dependency>
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
           </dependency>
           <!--bootstrap-->
           <dependency>
               <groupId>org.springframework.cloud</groupId>
               <artifactId>spring-cloud-starter-bootstrap</artifactId>
               <version>3.1.3</version>
           </dependency>
           <dependency>
               <groupId>com.dyw</groupId>
               <artifactId>Cloud-api-commons</artifactId>
               <version>1.0-SNAPSHOT</version>
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
               <scope>test</scope>
           </dependency>
   
       </dependencies>
   </project>
   ```

3. `bootstrap.yml`配置

   **要将Client模块下的`application.yml`文件改为`bootstrap.yml`，这是很关键的**

   因为`bootstrap.yml`是比`application.yml`先加载的。`bootstrap.yml`优先级高于`application.yml`

   ```YML
   server:
     port: 3355
   spring:
     application:
       name: config-client
     cloud:
       #config客户端配置
       config:
         label: master #分支名称
         name: application #配置文件名称
         profile: dev #读取后缀名称  上述三个综合：master分支上application-dev.yml的配置文件被
         uri: http://localhost:3344 #配置中心地址k 上述综合 == http://localhost:3344/master/application-dev.yml
   # 服务注册到eureka地址
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
         
   ```

4. 主启动类`ConfigClientMain3355`

   ```JAVA
   @EnableEurekaClient
   @SpringBootApplication
   public class ConfigClientMain3355 {
       public static void main(String[] args) {
           SpringApplication.run(ConfigClientMain3355.class,args);
       }
   }
   ```

5. 业务类 `ConfigClientController`

   ```JAVA
   package top.devildyw.springcloud.controller;
   
   import org.springframework.beans.factory.annotation.Value;
   import org.springframework.web.bind.annotation.GetMapping;
   import org.springframework.web.bind.annotation.RestController;
   
   /**
    * @author Devil
    * @since 2022-07-29-21:23
    */
   @RestController
   public class ConfigClientController {
       @Value("${config.info}")
       private String configInfo;
   
       @GetMapping("/configInfo")
       public String getConfigInfo(){
           return configInfo;
       }
   }
   ```

6. 启动测试

   测试接口 `http://localhost:3355/configInfo` 通过该接口获取配置信息`config.info`并将其返回。

   ```JSON
   master branch,springcloud-config/config-dev.yml version=1
   ```

   成功获取了配置文件中的`config.info`信息

### 分布式配置的动态刷新问题

我们的运维人员对`github`上的配置文件进行了修改，访问我们的配置中心能够时刻获取到最新的数据，但是当我们再次访问客户端时却还是修改前的数据。**问题随时而来，分布式配置的动态刷新。**

1. Linux运维修改`Github`上的配置文件内容做调整
2. 刷新3344，发现`ConfigServer`配置中心立刻响应
3. 刷新3355，发现`ConfigClient`客户端没有任何响应（指获取的配置文件信息并没有改变）
4. 3355没有变化除非自己重启或者重新加载
5. 难道每次运维修改配置文件，客户端都要重启？？

## `Config`客户端之动态刷新

为了避免每次修改配置文件都要重新启动服务，我们配置`Config`客户端动态刷新

修改`Cloud-config-client3355`

1. 检查`pom.xml`是否加入`actuator`依赖

   ```XML
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. 修改`bootstrap.yml`暴露监控端口

   ```YML
   # 暴露监控端点
   management:
     endpoints:
       web:
         exposure:
           include: "*"
   ```

   完整的配置文件

   ```YML
   server:
     port: 3355
   spring:
     application:
       name: config-client
     cloud:
       #config客户端配置
       config:
         label: master #分支名称
         name: application #配置文件名称
         profile: dev #读取后缀名称  上述三个综合：master分支上application-dev.yml的配置文件被
         uri: http://localhost:3344 #配置中心地址k 上述综合 == http://localhost:3344/master/application-dev.yml
   
   # 服务注册到eureka地址
   eureka:
     client:
       service-url:
         defaultZone: http://localhost:7001/eureka
   
   # 暴露监控端点
   management:
     endpoints:
       web:
         exposure:
           include: "*"
   ```

3. 在业务类`ConfigClentController`类头上添加`@RefreshScope`用以带有该注解的Bean可以在运行时刷新。

4. **但是光加这些还没有用，启动过后它还是不会自动地更新配置，此时想起我们刚刚提到的`actuator`。为了动态更新配置，运维人员需要以`post`请求调用`http://localhost:3355/actuator/refresh`手动刷新客户端获取新的配置信息，客户端再通过`@RefreshScope`的功能在客户端执行/refresh的时候就会更新此类下面的变量值。**

5. 此时我们修改`github`中的数据将`application-dev.yml`版本号改为4

   [![image-20220729221744155](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729221744155.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220729221744155.png)

   在不刷新3355的情况下调用`http://localhost:3355/configInfo`

   ```JSON
   master branch,springcloud-config/config-dev.yml version=3
   ```

   可以发现并没有获得最新的配置信息

6. 调用`http://localhost:3355/actuator/refresh`刷新3355

   ```JSON
   [
   
     "config.client.version",
   
     "config.info"
   
   ]
   ```

7. 再次访问`http://localhost:3355/configInfo`

   ```JSON
   master branch,springcloud-config/config-dev.yml version=4
   ```

   配置信息更新了。

### 手动动态刷新有什么问题

1. 假如有多个微服务客户端，每个微服务更新配置都要手动执行一次`post`请求来刷新，这样还是会十分麻烦。
2. 能否广播，一次通知，处处生效？
3. 如何大范围的自动刷新？