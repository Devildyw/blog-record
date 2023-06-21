# `Paxos`算法详解

## 前言–拜占庭将军问题

在介绍共识算法之前，先介绍一个简化版拜占庭将军的例子来帮助理解共识算法。

> 假设多位拜占庭将军中没有叛军，信使的信息可靠但有可能被暗杀的情况下，将军们如何达成是否要进攻的一致性决定？

解决方案大致可以理解成：先在所有的将军中选出一个大将军，用来做出所有的决定。

举例如下：假如现在一共有 3 个将军 A，B 和 C，每个将军都有一个随机时间的倒计时器，倒计时一结束，这个将军就把自己当成大将军候选人，然后派信使传递选举投票的信息给将军 B 和 C，如果将军 B 和 C 还没有把自己当作候选人（自己的倒计时还没有结束），并且没有把选举票投给其他人，它们就会把票投给将军 A，信使回到将军 A 时，将军 A 知道自己收到了足够的票数，成为大将军。在有了大将军之后，是否需要进攻就由大将军 A 决定，然后再去派信使通知另外两个将军，自己已经成为了大将军。如果一段时间还没收到将军 B 和 C 的回复（信使可能会被暗杀），那就再重派一个信使，直到收到回复。

[![img](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/v2-24f50b80ff971e7a8ac6798b7fa5f726_720w.jpg)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/v2-24f50b80ff971e7a8ac6798b7fa5f726_720w.jpg)

## 背景

`Paxos`是什么? `Paxos`算法是基于**消息传递**且具有**高度容错特性**的**一致性算法**，是目前公认的解决**分布式一致性**问题**最有效**的算法之一。

`Paxos`算法是Lamport宗师提出的一种基于消息传递的分布式一致性算法,它使其获得了2013年图灵奖.

自`Paxos`问世以来就持续垄断了分布式一致性算法，`Paxos`这个名词几乎等同于分布式一致性。（Google Chubby的作者Mike Burrows说过这个世界上**只有一种**一致性算法，那就是`Paxos`，其它的算法都是**残次品**。）Google的很多大型分布式系统都采用了`Paxos`算法来解决分布式一致性问题，如Chubby、`Megastore`以及Spanner等。开源的`ZooKeeper`，以及`MySQL`5.7推出的用来取代传统的主从复制的`MySQL` Group Replication等纷纷采用`Paxos`算法解决分布式一致性问题。

然而，`Paxos`的最大特点**就是难，不仅难以理解，更难以实现。**

## `Paxos`解决的问题

在常见的分布式系统中，总会发生诸如**机器宕机**或**网络异常**（包括消息的延迟、丢失、重复、乱序，还有网络分区）等情况。`Paxos`算需要解决的问题就是如何在一个可能发生上述异常的分布式系统中，快速且正确地在集群内部对**某个数据的值**达成**一致**，并且保证不论发生以上任何异常，都不会破坏整个系统的一致性。

**注意：**这里指的**某个数据的值**并不一定只是狭义上的某个数，它可以使日志，也可以是一条命令（command）… 根据应用场景的不同，**某个数据的值**有着不同得含义。

[![问题产生的背景](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/1752522-d2136179b456e13e.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/1752522-d2136179b456e13e.png)

## `Paxos`介绍

### `Paxos`的三种角色

- **Proposer: 提议者**
- **Acceptor: 决策者**
- **Learners: 最终决策学习者**

 在具体的实现中，一个进程可能同时充当多种角色。比如一个进程可能及时**Proposer又是Acceptor又是Learner。**

 既然有提议者，那么一定有提议，这里还有个有很重的概念叫做**提案（Proposal）**。最终要达成一致的value就在提案里面。

Proposer可以提出（propose）提案；Acceptor可以接受（accept）提案；如果某个提案被选定（chosen），那么该提案里的value就被选定了。

### 算法描述

[![image-20220721192011652](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721192011652.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721192011652.png)

- 一个完整的`Paxos`算法流程分为三个阶段

- ```
  Prepare
  ```

  准备阶段

  - `Proposer`向多个`Acceptor`发出`Propose`请求`Promise`(承诺)
  - `Acceptor`针对收到的`Propose`请求进行`Promise`(承诺)

- ```
  Accept
  ```

  接收阶段

  - `Proposer`收到多数`Acceptor`承诺后,向`Acceptor`发出`Propose`请求
  - `Acceptor`针对收到的`Propose`请求进行`Accept`处理

- ```
  Learn
  ```

  学习阶段

  - `Proposer`将形成的决议发送给所有`Learners`

### 算法流程

[![image-20220721192655799](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721192655799.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721192655799.png)

1. `Prepare`：`Proposer`生成全局唯一且递增的`Proposal ID`,向所有`Acceptor`发送`Propose`请求,这里无序携带提案内容,只携带`Proposal ID`即可

2. ```
   Promise
   ```

   ：

   ```
   Accept
   ```

   收到

   ```
   Propose
   ```

   请求后，做出”两个承诺，一个答应”。

   - 不在接收`Proposal ID`小于等于（注意：这里时<=）当前请求的`Propose`请求。
   - 不在接收`Proposal ID`小于（注意：这里是<）当前请求的`Accept`请求。
   - 不违背以前做出的承诺下，回复已经Accept过的提案中`Proposal ID`最大的那个提案的`Value`和`Proposal ID`，没有则返回空值。

3. `Propose`：`Proposer`收到多数Acceptor的Promise答应后，从答应中选择Proposal ID最大的提案的Value，作为本次要发起的提案。如果所有应答的提案Value均为空值，则可以自己随意决定提案Value。然后携带当前Proposal ID，向所有Acceptor发送Propose请求。

4. `Accept`：`Acceptor`收到Propose请求后，在不违背自己之前做出的承诺下（prepare阶段投了一票那么accpt阶段也会投），接受并持久化当前Proposal ID和提案Value。

5. `Learn`：`Proposer`收到多数Acceptor的Accept后，决议形成，将形成的决议发送给所有Learner

**下面我们针对上述描述做三种情况的推演举例：为了简化流程，我们这里不设置Learner。**

1. 有A1,A2,A3,A4,A5 5为议员，就税率问题进行决议。

   [![image-20220721194726379](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721194726379.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721194726379.png)

   - A1发起1号Proposal的Propose，等待Promise承诺；
   - A2-A5回应Promise；
   - A1在收到两份回复时，就会发起税率10%的Proposal；
   - A2-A5回应Accept；
   - 通过Proposal，税率10%

2. 现在我们假设在A1提出提案的同时, A5决定将税率定为20%

   [![image-20220721195054681](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721195054681.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721195054681.png)

   - A1，A5同时发起Propose（序号分别为1，2）
   - A2承诺A1，A4承诺A5，A3行为成为关键
   - 情况1：A3先收到A1消息，承诺A1。
   - A1发起Proposal（1，10%），A2，A3接受。
   - 之后A3又收到A5消息，回复A1：（1，10%），并承诺A5。
   - A5发起Proposal（2，20%），A3，A4接受。之后A1，A5同时广播决议。

**Paxos 算法缺陷：在网络复杂的情况下，一个应用 Paxos 算法的分布式系统，可能很久无法收敛，甚至陷入活锁的情况。**

1. 现在我们假设在A1提出提案的同时, A5决定将税率定为20%

   [![image-20220721195627426](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721195627426.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220721195627426.png)

   - A1，A5同时发起Propose（序号分别为1，2）
   - A2承诺A1，A4承诺A5，A3行为成为关键
   - 情况2：A3先收到A1消息，承诺A1。之后立刻收到A5消息，承诺A5。
   - A1发起Proposal（1，10%），无足够响应，A1重新Propose （序号3），A3再次承诺A1。
   - A5发起Proposal（2，20%），无足够相应。A5重新Propose （序号4），A3再次承诺A5。
   - ……

 造成这种情况的原因是系统中有一个以上的 `Proposer`，多个 `Proposers` 相互争夺 `Acceptor`，

造成迟迟无法达成一致的情况。针对这种情况，一种改进的 `Paxos`算法被提出：从系统中选

出一个节点作为 `Leader`，只有 `Leader`能够发起提案。这样，一次 `Paxos` 流程中只有一个

`Proposer`，不会出现活锁的情况，此时只会出现例子中第一种情况。

> 详细文档：
>
> [paxos-wiki](https://github.com/BitNile/paxos-wiki)
>
> [JavaGuide](https://javaguide.cn/distributed-system/theorem&algorithm&protocol/paxos-algorithm.html#cap理论)