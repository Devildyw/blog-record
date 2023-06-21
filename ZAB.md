# `Zookeeper Atomic Broadcast`(`Zookeeper` 原子广播协议)

## 什么是`ZAB`算法

`Zab`借鉴了`Paxos`算法(`Multi Paxos`)，是特别为`Zookeeper`设计的支持崩溃回复的原子广播协议。基于该协议，`Zookeeper`设计了为只有一台客户端(Leader)负责处理外部的写事务请求，然后Leader客户端将数据同步到其他Follower节点。即`Zookeeper`只有一个Leader可以发起提案。

## `ZAB`协议内容

`ZAB`协议包括两种基本的模式：消息广播、崩溃恢复。

#### 消息广播

[![image-20220722122448654](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220722122448654.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220722122448654.png)

1. 客户端发起一个写操作请求。
2. Leader服务器将客户端的请求转化为事务Proposal提案，同时为每个Proposal分配一个全局ID，即`zxid`
3. Leader服务器**为每个Follower服务器分配一个单独的队列**，然后将需要广播的Proposal依次放到队列中去，并且根据FIFO策略进行消息发送。
4. Follower接受到Proposal后，会首先将起以事务日志的方式是写入本地磁盘中，写入成功后向Leader反馈一个`Ack`响应消息
5. Leader接收到**超过半数以上**Follower的`Ack`响应消息后，即认为消息发送成功，可以发送commit消息
6. Leader向**所有Follower**广播commit消息，同时自身也会完成事务提交。Follower接受到commit消息后，会将上一条事务提交
7. `Zookeeper`采用`Zab`协议的核心，就是只要有一台服务器提交Proposal，就要确保所有服务器最终都能正确提交Proposal。

> `ZAB`协议针对事务请求的处理过程类似于一个两阶段提交过程
>
> 1. 广播事务阶段
> 2. 广播提交阶段
>
> 这两个阶段提交模型如下，有可能因为Leader宕机带来数据不一致，比如
>
> 1. Leader发起一个事务`Proposal1`后就宕机，Follower都没有`Proposal1`
> 2. Leader收到半数`ACK`宕机，没来得及向Follower发送Commit
>
> **为了解决上述问题，`ZAB`引入了崩溃恢复模式。**

#### 崩溃恢复–异常假说

一旦Leader服务器出现崩溃或者由于网络原因导致Leader服务器失去了与过半Follower的联系，那么就会进入**崩溃恢复模式**。

[![image-20220722131218448](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220722131218448.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220722131218448.png)

1. 假设两种服务器异常情况
   1. 假设一个事务在Leader提出之后，Leader挂了。
   2. 一个事务在Leader上提交了，并且过半的Follower都响应`Ack`了，但是Leader在Commit消息发出之前挂了。
2. `Zab`协议崩溃恢复要求满足以下两个要求：
   1. 确保已经被Leader提交的提案Proposal，必须最终被所有的Follower服务器提交。(**已经产生的提案，Follower必须执行**)
   2. 确保丢弃已经被Leader提出的但是没有被提交的Proposal。(**丢弃胎死腹中的提案**)

#### 崩溃恢复–Leader选举

崩溃恢复主要包括两个部分：**Leader选举和数据恢复**

[![image-20220722131903208](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220722131903208.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220722131903208.png)

**Leader选举：**根据上述要求，`Zab`协议需要保证选举出来的Leader需要满足以下条件：

1. 新选举出来的Leader不能包括未提交的Proposal。**即新Leader必须都是已经提交了Proposal的Follower服务器节点。**
2. **新选举的Leader节点中含有最大的`zxid`。**这样做的好处是可以避免Leader服务器检查Proposal的提交和丢弃工作。(因为它自身的`zxid`最大所以它有着最新的操作记录，所以不需要去检查其他服务器的Proposal的提交和丢弃，他自己就可以知道)

#### 崩溃恢复–数据恢复

崩溃恢复主要包括两个部分：**Leader选举和数据恢复**

[![image-20220722132241185](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220722132241185.png)](https://ding-blog.oss-cn-chengdu.aliyuncs.com/images/image-20220722132241185.png)

**`Zab`如何数据同步：**

1. 完成Leader选举后，在正式开始工作之前（接收事务请求，然后提出新的Proposal），**Leader服务器会首先确认事务日志中的所有Proposal是否已经被集群中过半的服务器Commit。**（在正式工作之前需要将日志中的数据同步）
2. Leader服务器需要确保所有的Follower服务器能够接收到每一条事务的Proposal，并且能将所有已经提交的事务Proposal应用到内存数据中。**等到Follower将所有尚未同步的事务Proposal都从Leader服务器上同步过，并且应用到内容数据中以后，Leader次啊会把该Follower加入到真正可用的Follower列表中。**