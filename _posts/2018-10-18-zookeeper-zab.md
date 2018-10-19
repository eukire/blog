---
layout: post
title:  "Zookeeper学习--ZAB协议"
categories: 学习笔记
tags:  ZooKeeper
excerpt: 读本文之前，您可能需要对分布式一致性协议有一定的了解。</br>有人或许会认为Zookeeper是Paxos算法的一个实现。事实上，Zookeeper并没有完全采用Paxos算法，而是采用了ZAB协议作为其数据一致性的核心算法...
---

* content
{:toc}

> 开源学习，自由转载，转载请注明出处！ 

## 简介

ZAB(Zookeeper Atomic Broadcast)协议是为分布式协调服务Zookeeper专门设计的一种支持崩溃恢复的院子广播协议。所以ZAB协议并不像Paxos算法那样，是一种通用的分布式一致性算法。ZAB协议的开发设计人员在协议设计之初，并没有要求其具有很好的拓展性，最初也只是为雅虎公司内部那些高吞吐量、低延迟、健壮、简单的分布式系统场景设计的。

ZAB协议的**核心**是定义了对于那些会改变Zookeeper服务器数据状态的事务请求的处理方式，如下图：

![2018/10/18/1.png](https://github.com/eukire/imgSrc/blob/master/2018/10/18/1.png?raw=true)

1. 客户端发送的所有事务请求必须由一个全局唯一的服务器来协调处理（Leader），余下的其他服务器成为Follower服务器。
2. Leader服务器负责将事务请求转换成一个事务Proposal（提议），并将Proposal分发给集群中的所有Follower服务器。
3. Follower服务器对Leader进行反馈。
4. 如果超过半数的Follower进行正确的反馈后，Leader就会再次向所有的Follower服务器分发Commit消息，要求其将前一个Proposal进行提交。

## 协议介绍

从上面的介绍中，我们已经了解了ZAB协议的核心，现在我们就可以来详细的了解下ZAB协议的具体内容。ZAB协议包括两种基本的模式，分别是崩溃恢复和消息广播。

### 消息广播

ZAB协议的消息广播过程使用的是一个院子广播协议，类似于一个二阶段提交过程。

![2018/10/18/2.png](https://github.com/eukire/imgSrc/blob/master/2018/10/18/2.png?raw=true)

这边主要说明一下ZAB协议和二阶段提交协议的区别。在ZAB协议的二阶段提交过程中。溢出了中断逻辑。这意味着我们可以在过半的Follower反馈ack后就开始提交事务Proposal，而二阶段提交要求所有的参与者要么全部成功要么全部失败。二阶段提交会产生严重阻塞问题。

换言之，只要有一个Follower提交了Proposal，就要确保所有的服务器最终都能正确提交proposal。这也是CAP/BASE最终实现一致性的一个体现。当然在这种简化了的二阶段提交模型下，是无法直接处理Leader奔溃退出而带来的数据不一致的问题。这个问题的解决依赖了ZAB协议的另一个模式崩溃恢复，这个后面会讲到。

其他关于消息广播的总结：

1. 消息广播协议是基于具有FIFO特性的TCP协议来进行网络通信的，因此能保证消息接收和发送的顺序性
2. 在Leader广播事务Proposal之前，会先为这个事务Proposal分配一个全局单调递增的唯一ID，就是我们熟知的事务ID(**ZXID**)。ZXID因此也保证了每一个消息严格的因果关系。

### 崩溃恢复

在上面消息广播的过程中，正常情况下运行非常良好，但是一旦Leader出现崩溃，或者因为网络原因导致Leader失去了与过半Follower的联系，那么此时就会进入崩溃恢复模式。

在ZAB协议中，为了保证程序正确的运行，整个恢复过程必须满足：

* 快速选举出新的Leader
* Leader自己知道自己成了Leader
* Follower知道新的Leader是谁

Leader服务器发生崩溃时分为如下场景： 

1. Leader在提出proposal时未提交之前崩溃，则经过崩溃恢复之后，新选举的Leader一定不能是刚才的Leader。因为这个Leader存在未提交的proposal。 
2. Leader在发送commit消息之后，崩溃。即消息已经发送到队列中。经过崩溃恢复之后，参与选举的follower服务器(刚才崩溃的leader有可能已经恢复运行，也属于follower节点范畴)中有的节点已经是消费了队列中所有的commit消息。即该follower节点将会被选举为最新的leader。剩下动作就是数据同步过程。

所以崩溃恢复的Leader选举算法最最基本的特性必须包含了两点：

1. ZAB协议需要确保那些已经在Leader服务器提交的事务最终被所有的服务器都提交
2. ZAB协议需要确保丢弃那些只在Leader服务器上被提出的事务

针对这两点，如果让Leader选举算法能够保证新选举出来的Leader服务器拥有集群中所有机器的最高编号（MAX ZXID）的事务Proposal，那么就可以保证这个新选举出来的Leader一定具有了所有已经提交的提案。同时，可以省去Leader服务器检查Proposal的提交和丢弃工作的这一步操作！

### 数据同步

1. 在zookeeper集群中新的leader选举成功之后，leader会将自身的提交的最大proposal的事物ZXID发送给其他的follower节点。follower节点会根据leader的消息进行回退或者是数据同步操作。最终目的要保证集群中所有节点的数据副本保持一致。
2. 数据同步完之后，zookeeper集群如何保证新选举的leader分配的ZXID是全局唯一呢？ZXID是一个长度64位的数字，其中低32位是按照数字递增，即每次客户端发起一个proposal,低32位的数字简单加1。高32位是leader周期的epoch编号，至于这个编号如何产生，每当选举出一个新的leader时，新的leader就从本地事物日志中取出ZXID,然后解析出高32位的epoch编号，进行加1，再将低32位的全部设置为0。这样就保证了每次新选举的leader后，保证了ZXID的唯一性而且是保证递增的。 

## ZAB协议内部原理

整个ZAB协议主要包括了消息广播和崩溃恢复两个过程，进一步可以细分为三个阶段。分别是发现（Discovery）、同步（Syschronization）和广播（Broadcast）阶段。组成ZAB协议的每一个分布式进程，会循环的执行这三个阶段，我们将这一个循环成为一个主进程周期。

1. **发现**：即要求zookeeper集群必须选择出一个leader进程，同时leader会维护一个follower可用列表。将来客户端可以这follower中的节点进行通信。
2. **同步**：leader要负责将本身的数据与follower完成同步，做到多副本存储。这样也是体现了CAP中高可用和分区容错。follower将队列中未处理完的请求消费完成后，写入本地事物日志中。
3. **广播**：leader可以接受客户端新的proposal请求，将新的proposal请求广播给所有的follower。

## 写在最后

深刻理解ZAB协议，才能更好的理解zookeeper对于分布式系统建设的重要性。以及为什么采用zookeeper就能保证分布式系统中数据最终一致性，服务的高可用性

### 参考文献

1. 从PAXOS到ZOOKEEPER分布式一致性原理与实践》-倪超
2. [《zookeeper中的ZAB协议理解》](https://blog.csdn.net/junchenbb0430/article/details/77583955)
3. [《Zookeeper基础》](https://www.w3cschool.cn/zookeeper/zookeeper_fundamentals.html)

