# CAP理论

CAP理论告诉我们，一个分布式系统不可能同时满足以下三种

⚫ 一致性（C:Consistency）

⚫ 可用性（A:Available）

⚫ 分区容错性（P:Partition Tolerance）

这三个基本需求，最多只能同时满足其中的两项，因为P是必须的，因此往往选择就在CP或者AP中。

1. **一致性**（**C:Consistency**）

   在分布式环境中，一致性是指数据在多个副本之间是否能够保持数据一致的特性。在一致性的需求下，当一个系统在数据一致的状态下执行更新操作后，应该保证系统的数据仍然处于一致的状态。

2. **可用性**（**A:Available**）

   可用性是指系统提供的服务必须一直处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果。

3. **分区容错性**（**P:Partition Tolerance**）

   分布式系统在遇到任何网络分区故障的时候，仍然需要能够保证对外提供满足一致性和可用性的服务，除非是整个网络环境都发生了故障。

**ZooKeeper保证的是CP**

1. **`ZooKeeper`不能保证每次服务请求的可用性。**（注：在极端环境下，ZooKeeper可能会丢弃一些请求，消费者程序需要重新请求才能获得结果）。所以说，ZooKeeper不能100%保证服务可用性。
2. **进行Leader选举时集群都是不可用。**

**Eureka保证的是AP**

## AP架构

当网络分区出现后，为了保证可用性，系统B可以返回旧值，保证系统的可用性

**结论：违背了一致性C的要求，只满足可用性和分区容错性，即AP**

[![image-20220724002343024](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220724002343024.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220724002343024.png)

## CP架构

当网络分区出现后，为了而保证一致性，就必须拒接请求，否则无法保证一致性

**结论：违背了可用性A的要求，只满足一致性和分区容错性，即CP**

[![image-20220724002423706](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220724002423706.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220724002423706.png)