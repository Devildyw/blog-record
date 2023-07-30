# Xxl-Job 分布式任务调度

## 1.分布式任务调度

目前，我们的定时任务都是基于SpringTask来实现的。但是SpringTask存在一些问题：

- 当微服务多实例部署时，定时任务会被执行多次。而事实上我们只需要这个任务被执行一次即可。
- 我们除了要定时创建表，还要定时持久化Redis数据到数据库，我们希望这多个定时任务能够按照顺序依次执行，SpringTask无法控制任务顺序

不仅仅是SpringTask，其它单机使用的定时任务工具，都无法实现像这种任务执行者的调度、任务执行顺序的编排、任务监控等功能。这些功能必须要用到分布式任务调度组件。

### 1.1.分布式任务调度原理

那么分布式任务调度是如何实现任务调度和编排的呢？

我们先来看看普通定时任务的实现原理，一般定时任务中会有两个组件：

- 任务：要执行的代码
- 任务触发器：基于定义好的规则触发任务

因此在多实例部署的时候，每个启动的服务实例都会有自己的**任务触发器**，这样就会导致各个实例各自运行，无法统一控制：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NjY1MTQ4OTg0NzdhYTI4ZWYwZWQxNmU2MGRkMmYwZDNfZ1VSS0x6Z0Z3eWpaWjNNRkdtMEJpek9UWnRkZjByczBfVG9rZW46SUFXUWIyM29sb0JEaWx4UUo1Z2NVZXFtbmVoXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

那如果我们想要统一控制各个服务实例的任务执行和调度该怎么办？

大家应该能想到：就是要把任务触发器提取到各个服务实例之外，去做统一的触发、统一的调度。

事实上，大多数的分布式任务调度组件都是这样做的：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTkwNjZjYTM5YmEwMjE0MzYwNWI5MTZhMjYzMGM3ZTJfR3JNZDlva29ZNzFoc2V4ckg2d3hOOEpEcGZPY29XUE5fVG9rZW46TkozVmJaVnZ3b1RVckJ4bTF5WmNST0NZblBkXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

这样一来，具体哪个任务该执行，什么时候执行，交给哪个应用实例来执行，全部都有统一的任务调度服务来统一控制。并且执行过程中的任务结果还可以通过回调接口返回，让我们方便的查看任务执行状态、执行日志。这样的服务就是**分布式****调度服务**了。

### 1.2.分布式任务调度技术对比

能够实现分布式任务调度的技术有很多，常见的有：

|                  | **Quartz** |   **XXL-Job**    |       **SchedulerX**       |           **PowerJob**           |
| :--------------: | :--------: | :--------------: | :------------------------: | :------------------------------: |
|   **定时类型**   |    CRON    | 频率、间隔、CRON | 频率、间隔、CRON、OpenAPI  |    频率、间隔、CRON、OpenAPI     |
|   **任务类型**   |    Java    |    多语言脚本    |         多语言脚本         |            多语言脚本            |
| **任务调度方式** |    随机    |    单机、分片    | 单机、广播、Map、MapReduce | 单机、广播、分片、Map、MapReduce |
|  **管理控制台**  |     无     |       支持       |            支持            |               支持               |
|   **日志白屏**   |     无     |       支持       |            支持            |               支持               |
|   **报警监控**   |     无     |       支持       |            支持            |               支持               |
|    **工作流**    |     无     |       有限       |            支持            |               支持               |

其中：

- Quartz由于功能相对比较落后，现在已经很少被使用了。
- SchedulerX是阿里巴巴的云产品，收费。
- PowerJob是阿里员工自己开源的一个组件，功能非常强大，不过目前市值占比还不高，还需要等待市场检验。
- XXL-JOB：开源免费，功能虽然不如PowerJob，不过目前市场占比最高，稳定性有保证。

我们课堂中会选择XXL-JOB这个组件，如果你们企业具备探索精神，而且需要一些分布式运算功能，推荐使用PowerJob。

### 1.3.XXL-JOB介绍

官网地址：

https://www.xuxueli.com/xxl-job/

XXL-JOB的运行原理和架构如图：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NmNkNTY1OGNhYTA1ODc0OTQzYTFkMzVkMDBiZTBmNGZfcXkxdnlWM3pIekhJa1VQZ3BIbFFBNTA2YU1oeEFHOUNfVG9rZW46THJKdWJCT2tqb0U5WDB4ZUllOWNWOW5Bbm9NXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

XXL-JOB分为两部分：

- **执行器**：我们的服务引入一个XXL-JOB的依赖，就可以通过配置创建一个执行器。负责与XXL-JOB调度中心交互，执行本地任务。
- **调度中心**：一个独立服务，负责管理执行器、管理任务、任务执行的调度、任务结果和日志收集。

### 1.4.XXL-JOB定时创建榜单表

接下来，我们就来一个XXL-JOB的快速入门，顺便改造一下之前用SpringTask实现的定时创建榜单表的功能。

#### 1.4.1.部署调度中心

调度中心在我们提供的虚拟机开发环境中已经部署完成了。访问：[http://xxljob.tianji.com](http://xxl-job.tianji.com)即可查看调度中心控制台页面。默认的账号密码是：admin/123456

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=OTMwMmZjNWNkZWQ3M2FjNGEwN2U4Nzk2ZjM4MmJkZjFfZTVLVmg2dTR4aHZ3SUVDMHllMjVuRGY4QnRwTEtwT3NfVG9rZW46UGVwQmJqaTNrb1Rpaml4RDJhNGNKSXZvbmJmXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

如果要自己部署，分为两步：

- 运行初始化SQL，创建数据库表
- 利用Docker命令，创建并运行容器

课前资料已经给出了脚本：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=YWNkZjhhYTY4ZTRmODJlZmE3NDM4YjY1MjY5MTAxZDhfSWdqQllXZERqV1RCRk5tb3FHeG9McmtacWp0ZEF4QmZfVG9rZW46VDRKWmJFYTlrb3AxQTZ4ME1ocWNoZDRMbnplXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

最终XXL-JOB的表结构如下：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=YTBhMzkwYWZiYjVhMjc2M2JjYTQzM2RmM2E1N2UwZWNfQkRqSXdsSmNpTjl0WmVjMEd3cnR5Q0d3Yk1DOFBmRWNfVG9rZW46RHI2RWJPTnJxb0J6aVp4aGpCdmN6b3FGbjZlXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

说明：

- xxl_job_lock：任务调度锁表；
- xxl_job_group：执行器信息表，维护任务执行器信息；
- xxl_job_info：调度扩展信息表： 用于保存XXL-JOB调度任务的扩展信息，如任务分组、任务名、机器地址、执行器、执行入参和报警邮件等等；
- xxl_job_log：调度日志表： 用于保存XXL-JOB任务调度的历史信息，如调度结果、执行结果、调度入参、调度机器和执行器等等；
- xxl_job_log_report：调度日志报表：用户存储XXL-JOB任务调度日志的报表，调度中心报表功能页面会用到；
- xxl_job_logglue：任务GLUE日志：用于保存GLUE更新历史，用于支持GLUE的版本回溯功能；
- xxl_job_registry：执行器注册表，维护在线的执行器和调度中心机器地址信息；
- xxl_job_user：系统用户表；

#### 1.4.2.微服务集成执行器

首先需要在tj-learning服务引入依赖：

```XML
<!--xxl-job-->
<dependency>
    <groupId>com.xuxueli</groupId>
    <artifactId>xxl-job-core</artifactId>
</dependency>
```

然后还需要配置执行器，下面是一个配置执行器的示例：

```Java
@Bean
public XxlJobSpringExecutor xxlJobExecutor() {
    logger.info(">>>>>>>>>>> xxl-job config init.");
    XxlJobSpringExecutor xxlJobSpringExecutor = new XxlJobSpringExecutor();
    xxlJobSpringExecutor.setAdminAddresses(adminAddresses);
    xxlJobSpringExecutor.setAppname(appname);
    xxlJobSpringExecutor.setIp(ip);
    xxlJobSpringExecutor.setPort(port);
    xxlJobSpringExecutor.setAccessToken(accessToken);
    xxlJobSpringExecutor.setLogPath(logPath);
    xxlJobSpringExecutor.setLogRetentionDays(logRetentionDays);

    return xxlJobSpringExecutor;
}
```

参数说明：

- adminAddress：调度中心地址，天机学堂中就是填虚拟机地址
- appname：微服务名称
- ip和port：当前执行器的ip和端口，无需配置，自动获取
- accessToken：访问令牌，在调度中心中配置令牌，所有执行器访问时都必须携带该令牌，否则无法访问。咱们项目的令牌已经配好，就是`tianji`。如果要修改，可以到虚拟机的`/usr/local/src/xxl-job/application.properties`文件中，修改`xxl.job.accessToken`属性，然后重启XXL-JOB即可。
- logPath：任务运行日志的保存目录
- logRetentionDays：日志最长保留时长

但是呢，大家完全不需要自己配置调度器了，因为在天机学堂的tj-common模块已经实现了XXL-JOB的自动装配： 

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NjZlMDZhZDE5ZGFjYmVlODVhZjZhMjcwMDJhYTQyYmNfVFM5aDBnZXZjNmFwWmpqOW5vR1BjMFM0Q0ZxY1F6eUZfVG9rZW46RHBLVWJxYXFpb0ZyWXB4bjRnb2N1OFFabllTXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

配置中的关键属性都已经在Nacos中共享了：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=MzgzMzVhOTQyN2Y2N2MwYmZiYTU3OTc1ZGYwMWNlZjlfcHdFVVN5clBDbUEzbmdJVDhRZUdlb0hhM3p3aWp2bFFfVG9rZW46WjVKSWJqNWFMb09IMVh4SXYwVWNTaHBrblprXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

所以，我们项目的微服务模块只要引入`了tj-common`，并且引入了XXL-JOB的依赖，就可以直接使用了。

#### 1.4.3.定义任务

 接下来，把之前的SpringTask任务改成XXL-JOB的任务。

我们修改tj-learning模块下的`com.tianji.learning.handler.PointsBoardPersistentHandler`，将原本的`@Scheduled`注解替换为`@XXLJob`注解：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=Y2IxZGNhOGRhNTQ0NTcwZThlYzAxNmE4NGIzNGQwYTRfSW52U2NXd0JwWEhXZHRIWWM0UTJmWkhmNk5TdUh0QzZfVG9rZW46RHJmaGJOM2lmb214Nld4VUFKemM2dFE2blFlXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

其中，`@XxlJob`注解中定义的就是当前**任务的名称**。

#### 1.4.4.注册执行器

接下来，重启`tj-learning`服务，登录XXL-JOB控制台，注册执行器。

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZDM0NmUxMmYxODlkMzI1OTI2ZjYwMzBhOTU3MDFlZmNfV1BJQ1o4d29pUHlReU1idWl4WThnMzBzcUJrNkR0UUdfVG9rZW46SDA0QmI5VFVsb1JTdU54N1NyamNOOHIzbmp2XzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

在弹出的窗口中填写信息：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NDk5N2RlMGQ3YmU1NzJjMWViYTU2OTVhYTNjODMzNDFfMkx3NHd1R1FpTXpUQ2NhVFFNUlVNNE5CSFhMNUc1S0RfVG9rZW46V0w2cWJNTnRrb1RhS1d4UDlUZ2MwVGZObktlXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

等待一段时间，会发现`learning-service`已经成功注册了：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjA1NzQyNjYzMTg1Y2FiYTIyYTM4YjM5NWVlNWY3MjJfdGpSWXk3aVJobkVFSkRtaDdDNGI0aXQ5cHBqazRqbGFfVG9rZW46TGREOGJWQnhOb3dyNnl4RWpaWGNHdURoblJiXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

#### 1.4.5.配置任务调度

现在，执行器已经成功注册，任务也已经注册到调度中心。接下来，我们就可以来做任务调度了，也就是：

- 分配任务什么时候执行
- 如果有多个执行器，应该由哪个执行器执行（路由策略）

我们进入任务管理菜单，选中学习中心执行器，然后新增任务：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NTAwZjU4ZTU0MWQzZTQ2MGU1YjMzYzRjZjM1OTFjZDlfV3FYUU1uMG56a3JTMmFlS0ZRUzluTUZuZkJkZTBZc1dfVG9rZW46QTZMNGJRd043b3VFZm14Nkc1cGM3c1Rhbm9iXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

在弹出表单中，填写任务调度信息：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTljNWY4Zjc5Zjc1ZWIwMjc4OGUxMTY3MmQ3Y2I5MGNfRFZvYllhc2VKNGdDRG5XY0lUYnA5dGhnR2dEQ0dHblFfVG9rZW46UWlVdmJxcmh6b1luVUh4V2xld2NOMW1CbjhiXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

其中比较关键的几个配置：

- 调度配置：也就是什么时候执行，一般选择cron表达式
- 任务配置：采用BEAN模式，指定JobHandler，这里指定的就是在项目中`@XxlJob`注解中的任务名称
- 路由策略：就是指如果有多个任务执行器，该由谁执行？这里支持的策略非常多：
  - ![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=MGRjMmU3YTc1ZTE1ZjAxZTAzNDAzODMzZWM5MzIxYmFfVzZqdGl1djNYbExlSXdOT2VrdnlGS1h1b2oxbTdQWDlfVG9rZW46RlRmdGIzTmFvb0RuZzh4Y3F5QmNLb2Qxbm5jXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

路由策略说明：

- FIRST（第一个）：固定选择第一个执行器；
- LAST（最后一个）：固定选择最后一个执行器；
- ROUND（轮询）：在线的执行器按照轮询策略选择一个执行
- RANDOM（随机）：随机选择在线的执行器；
- CONSISTENT_HASH（一致性HASH）：每个任务按照Hash算法固定选择某一台执行器，且所有任务均匀散列在不同执行器上。
- LEAST_FREQUENTLY_USED（最不经常使用）：使用频率最低的执行器优先被选举；
- LEAST_RECENTLY_USED（最近最久未使用）：最久未使用的执行器优先被选举；
- FAILOVER（故障转移）：按照顺序依次进行心跳检测，第一个心跳检测成功的执行器选定为目标执行器并发起调度；
- BUSYOVER（忙碌转移）：按照顺序依次进行空闲检测，第一个空闲检测成功的执行器选定为目标执行器并发起调度；
- SHARDING_BROADCAST(分片广播)：广播触发对应集群中所有执行器执行一次任务，同时系统自动传递分片参数；可根据分片参数开发分片任务

#### 1.4.6.执行一次

当任务配置完成后，就会按照设置的调度策略，定期去执行了。不过，我们想要测试的话也可以手动执行一次任务。

在任务管理界面，点击要执行的任务后面的`操作`按钮，点击`执行一次`：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=MGE2MzYzYTBiNGNmZmE2Yzk4NzdjYzc5Y2VmMjRhOGVfV0cyRUpUVDZlRVZ2aENpa0VYc2VyMzRwQTk4dnZ1NG5fVG9rZW46UVZZemJSYm5ob1prZGR4WWVaVmNOQnlCbnhlXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

然后在弹出的窗口中，直接点保存即可执行：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ODBlNWEwNzg5ZjRhZTAzYmRiODBjNjU1NGMyYjNiZGVfYnBma3BKZnAyU0c0VkhOcFhjbkNXMWVBSzFIY3k0TG1fVG9rZW46VWl6YWJ5Y2tMb2RHRXV4U3lEWGNhb1hqbmRkXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)

注意，如果是分片广播模式， 这里还可以填写一些任务参数。

然后在调度日志中，可以看到执行成功的日志信息：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ODU1OThkOTllYTMzN2FhYTg4M2ZiOTAzNGYwMmQ1MTVfZ043djh4TFphSzVNTzN5bVJ2WUd2ZTBCSnF0bUhPUkNfVG9rZW46REM5ZGJva0hEb2JRcDd4SDQxVGNkeTJPbkFzXzE2OTAyNjI5MDg6MTY5MDI2NjUwOF9WNA)



### 1.5.XXL-JOB任务分片

刚才定义的定时持久化任务，通过while死循环，不停的查询数据，直到把所有数据都持久化为止。这样如果数据量达到数百万，交给一个任务执行器来处理会耗费非常多时间。

因此，将来肯定会将学习服务多实例部署，这样就会有多个执行器并行执行。**但是，**如果交给多个任务执行器，大家执行相同代码，都从第1页逐页处理数据，又会出现重复处理的情况。

怎么办？

这就要用到任务分片的方案了。

怎样才能确保任务不重复呢？我们可以参考扑克牌发牌的原理：

- 逐一给每个人发牌
- 发完一圈后，再回头给第一个人发
- 重复上述动作，直到牌发完为止

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZGI4ZDI3Nzc5ZGJhMzUxOWVjY2NhMzE4YmU5NmU3ZjRfN25ST1ByVWFDUkVHZHp3NjRTZW45TnJrdEtWUWdnVFpfVG9rZW46VlVRZ2JYbGZJbzFRbkV4ZHVIQmN1V05MbnRnXzE2OTAyNjMwNTM6MTY5MDI2NjY1M19WNA)

与此类似，比如我们启动了3个服务实例，就有3个执行器。我们可以把执行器当做打牌的人，然后把每一页数据作为一张牌：

- 把每页数据逐一分发给每个执行器，
- 发完一圈后，再回到第一个执行器。
- 直到所有页数据都发放完毕。

那么数据分发的过程如图：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NGNjMTdiOWQ2ODI1OTY4MTZlMGNiYTgyMTkzNzIzM2VfREhaTEJNNE1nSHdNekRESjUySnFIUHZzY05JSzFFQmJfVG9rZW46VURoamIxY1Bub25INmt4YWI3aWNhRjE3blJlXzE2OTAyNjMwNTM6MTY5MDI2NjY1M19WNA)

最终，每个执行器处理的数据页情况：

- 执行器1：处理第1、4、7、10、13、...页数据
- 执行器2：处理第2、5、8、11、14、...页数据
- 执行器3：处理第3、6、9、12、15、...页数据

要想知道每一个执行器执行哪些页数据，只要弄清楚两个关键参数即可：

- 起始页码：pageNo
- 下一页的跨度：step

而这两个参数是有规律的：

- 起始页码：执行器编号是多少，起始页码就是多少
- 页跨度：执行器有几个，跨度就是多少。也就是说你要跳过别人读取过的页码

因此，现在的关键就是获取两个数据：

- 执行器编号
- 执行器数量

这两个参数XXL-JOB作为任务调度中心，肯定是知道的，而且也提供了API帮助我们获取：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZWUyZmJlZDFiZDVkNzcyZmMxNzZlODFiYTZkNmYxZDRfcFB2N015WklvVXpuMDZxakFTVGtMdUswNzB1ZGVZUWFfVG9rZW46R1Rla2JBYkI0b2RFRmd4ZlhNZmNPMDNXbklkXzE2OTAyNjMwNTM6MTY5MDI2NjY1M19WNA)

这里的分片序号其实就是执行器序号，不过是从0开始，那我们只要对序号+1，就可以作为起始页码了。



### 1.6.任务链

现在，所有任务都已经定义完毕。接下来就给配置任务调度了。

我们最终期望的任务执行顺序是这样的：

![image-20230725133226710](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20230725133226710.png)

但问题来了，我们该如何控制三个任务的执行顺序呢？

这就要借助于XXL-JOB中的子任务功能了。

首先，我们把持久化榜单数据、清理Redis中历史榜单的任务也在XXL-JOB中定义出来。

首先是持久化榜单：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=YzY1Zjg0MDIzYTNiY2NkMzU0MWNlM2IwNzM0Y2Q4NWVfdnc2YmtrbWxWZGlOYzBmbW5FcVZWRU9HM0xBOG1tMGtfVG9rZW46R0x3WmIxQ0JOb05UanN4ZWg1WWNCbHpDbnBkXzE2OTAyNjI5ODU6MTY5MDI2NjU4NV9WNA)

然后是清理Redis的任务：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NzA2NjlkZGZkYjQ0NTUxYTc3ZDJmYWU0MDY1MjQ4NzBfWnd3aTJ3QkY2OTd2V0hvSFFpMW5nZXRCeW81Z3BQNW9fVG9rZW46UUM3WmJ4UFNPbzh1WkR4MDRYZGM3QWNFbkloXzE2OTAyNjI5ODU6MTY5MDI2NjU4NV9WNA)

接下来，回到任务管理页面，会看到3个任务都添加成功，并且每个任务都有自己的ID：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTM5NDliYTgzZDRiMzIxNmRjODczZmMzZjA3Y2YwYWJfWW1WUDZPWko1QkNDTkxrS2NETGtJZlRlQWZOYUs5QldfVG9rZW46RmdSbmJWOUV6bzVVZnV4aW5qYmNTQlVIbnRoXzE2OTAyNjI5ODU6MTY5MDI2NjU4NV9WNA)

要想让任务A、B依次执行，其实就是配置任务B作为任务A的子任务。因此，我们按照下面方式配置：

- 创建历史榜单表（10）的子任务是持久化榜单数据任务（12）
- 持久化榜单数据任务（12）的子任务是清理Redis中的历史榜单（13）

也就是说：10的子任务是12,  12的子任务是13

首先，点击创建历史绑定表后面的操作，然后编辑：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjUwNjAwMjhiOGNhZmExMmM1NjY3MDFjOGRjOWJhNWFfS2tzREtNd3NPc0F3YVhTam43czFzTHZkd05KcldiUE5fVG9rZW46VEJDdGJldE5yb0JFY1R4c2E1dmNjbWlvbkxkXzE2OTAyNjI5ODU6MTY5MDI2NjU4NV9WNA)

然后在子任务中，填写持久化榜单数据任务的id，本例中是12：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=MmM4NDRjNjYxNDc1MGUwYzY4ODEzNGU2YWFkMGE0OGNfZHhjNFJVS2x2YjZvdEdhdXZaQktNZWNoZmlSQ1FSZ2VfVG9rZW46U3BWR2Jrb1NWbzMyb3V4YjZVdGNlYWMxblZkXzE2OTAyNjI5ODU6MTY5MDI2NjU4NV9WNA)

保存。

然后点击持久化榜单数据任务后面的操作，编辑：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NjJkZDMxNDAwMmM3ZWRiZDJlODdiMjUwYjBiMzU2NzJfVVRXY2M5dzZhS2VvUDJkUkxSTURDR1gzWmJDMHRLVjBfVG9rZW46SmQ2cmJLdVJ5b2ZnY2x4RkNFRmNOT2xEblNoXzE2OTAyNjI5ODU6MTY5MDI2NjU4NV9WNA)

然后在子任务一栏，填写清理Redis中的历史榜单的任务id，本例中是13：

![img](https://b11et3un53m.feishu.cn/space/api/box/stream/download/asynccode/?code=NDYxMjg0NGVhMjM5NGIwNWU2MjFiNzU1YjM0YzkyZGVfTm9TUjg3cWwwb1ZPNmFudUIyY1FEeHdma0kyQklYcXBfVG9rZW46UndiR2JLNTFVb1UyOWN4aFRGeGNMWjlDblROXzE2OTAyNjI5ODU6MTY5MDI2NjU4NV9WNA)

好了，任务链形成了。

接下来，执行一次创建榜单表任务，就会发现后续的两个任务也依次执行了。

**注意：分片广播路由策略后设置了子任务id的话会在第一个分片执行完后就紧接着执行子任务了，如果有严格的先后顺序的话，分片广播后就不要添加子任务了，而是在分片广播任务执行完后，手动执行。**