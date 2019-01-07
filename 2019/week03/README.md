## ARTS(week03)

### 算法题(Algorithm)

[旋转数组](https://github.com/geekwho11/learn.leetcode.xbcme/tree/master/php/src/189.rotate-array)

### 阅读点评(Review)

#### [Choosing MySQL High Availability Solutions](https://dzone.com/articles/choosing-mysql-high-availability-solutions)

#### 阅读笔记

高可用环境为持续可用的数据库提供了很大的好处。高可用数据库环境通过部署在多台服务器环境下，任一服务器都可以对外提供统一的服务。通过这种方式，数据库避免了单点故障。

##### 你想要解决的问题是什么？

1. 在发生灾难时需要多个数据副本
2. 需要增加读和/或写吞吐量
3. 需要尽量减少停机时间

##### CAP

你可能最多选择其中的2个。

1. 一致性(Consistency) 所有的节点看到一直的数据。
2. 可用性(Availability) 所有的请求都可以收到成功与否的回复。
3. 分区容错性(Partition Tolerance) 因网络原因造成系统任意的分区，系统依然可以继续运行。

##### 数据丢失

1. 是否可以接受数据的丢失？

##### 避免单点故障

##### 是否可以接受事务丢失？

##### 冲突检测和解决

##### 是需要故障转移还是分布式系统？

故障转移的缺点

1. 故障转移有一个监控会检测失败的节点，也有可能会移除正常的节点。
2. 故障转移需要花费时间

分布式系统

1. 分布式系统有很小的故障时间

故障转移是自动化还是手动？

手动的故障转移的优势

1. 最大的优势是人为去决定故障转移是否是必要的。
2. 系统运行尽管不完美，但接近可用。

自动故障转移的优势

1. 由于停机次数减少，可用性占比会提高
2. 不需要等到DBA手动去执行

故障转移的速度需要多快？

复制 MHA/MMM

1. 取决于在故障转移时，待处理事务完成同步的时间
2. 通常情况下30s

MHA(Master High Availability Manager and tools for MySQL ) 

MMM (Master-Master replication manager for MySQL)

DRBD Distributed Replicated Block Device

1. 通常15-30s

[XtraDB Cluster](https://www.percona.com/services/support/mysql-ha-cluster-support) / MySQL Cluster

1. 故障转移非常快，通常小于1s，取决与负载均衡

##### 你真正需要多少个9?

##### 是否需要可扩展的读和写？

##### The Rule of Threes

1. 你最少需要3个节点，避免中心化。
2. 避免51%概率事件

##### 你需要多少个数据中心？

##### 如何规划你的灾难恢复？

##### 你需要什么样的数据表引擎？

##### 负载均衡的选择

HAProxy 

1. 开源代码方案
2. 不能分离读写，如果是必备的话，就需要应用层面支持。

F5 BigIP

1. 硬件方案的代表

MaxScale

1. 无法分离读写

Elastic Load Balancer

1. 弹性负载均衡

##### 如果整个集群重新启动 会怎么样？

##### 数据写入后是否需要立即可读？

##### 如果要做大量数据加载 怎么办？

##### 你是否采取了措施预防"脑裂"的情况？

##### 你的应用是否需要高并发性？

##### RAM是否有限制

##### 网络有多稳定？

### 技术技巧(Tip)

####  查找指定提交者

```
git log --author="geek.who"  --after="2018-11-20 00:00:00" --before="2018-11-21 23:59:59"

参数使用
--since=<date>, --after=<date> 在指定的日期之后的提交。
--until=<date>, --before=<date> 在指定日期之前的提交。
```



### 分享(Share)

#### [MySQL High Availability at GitHub](https://githubengineering.com/mysql-high-availability-at-github/)

#### 思考总结

当考虑高可用性和服务发现时，一些问题可以指导你找到一个恰当的解决方案。这些问题包括但不限于：

- 你能容忍的宕机时间是多久？
- 崩溃检测的可靠性如何？你能容忍假阳性（过早进行故障恢复）吗？
- 故障恢复的可靠性如何？它在哪些情况下会失败？
- 解决方案跨数据中心能力如何？在低延迟和高延迟网络的能力如何？
- 解决方案能克服完整的数据中心故障或网络隔离的影响吗？
- 如果有的话，什么机制能够防止或减轻脑裂现象（两个服务器都宣称是指定集群的主节点，都独立地彼此无意识地接受写操作）？
- 你能够承受数据丢失吗？到什么程度？



基于 VIP 和 DNS 的服务发现，可能会产生“脑裂”的情况发生。



为了提升可用性，GitHub采用下面的方案：

1. [orchestrator](https://github.com/github/orchestrator)用来运行故障监听和故障恢复。我们使用了如下图所示的一个跨数据中心的[orchestrator/raft](https://github.com/github/orchestrator/blob/master/docs/raft.md)。
2. Hashicorp 公司的用于服务发现的[Consul](https://www.consul.io/)。
3. 作为客户端和写操作节点之间的代理层的[GLB/HAProxy](https://githubengineering.com/introducing-glb/)。
4. 用于网络路由的`anycast`。

在一个主节点宕机场景：

- `orchestrator`节点监测到故障。
- `orchestrator/raft`领导开始一次恢复措施，提升一个新的主节点。
- `orchestrator/raft`将主节点变更通告给所有`raft`集群节点
- 每个`orchestrator/raft`成员接收到一个领导变更通知。它们各自在本地 Consul 的 KV 存储器中更新新的主节点的身份。
- 每个 GLB/HAProxy 都运行了`consul-template`，监控 Consul 的 KV 存储中的变更，然后重新配置和加载 HAProxy。
- 客户端流量被重定向到新的主节点。

每个组件都职责清晰，而且整个设计既解耦又简单。`orchestrator`不需要知道负载均衡器。Consul 不需要知道信息来自哪里。代理只关心 Consul。客户端只关心代理。

此外：

- 无需传播 DNS 变更。
- 没有 TTL。
- 这个流程不需要挂掉的主节点的合作。它很大程度上被忽略了。

#### 参考链接
1. [GitHub 的 MySQL 高可用性实践](https://www.infoq.cn/article/mysql-high-availability-at-github)


