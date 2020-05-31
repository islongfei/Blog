#### Lucene 
#### 倒排索引
#### 全文检索
#### es 分布式架构原理
#### es 写入和查询的工作原理
#### es 在数据量很大情况下（亿级）如何提高性能
#### es 生产集群部署架构，每个索引数据量大概多少，每个索引有多少个分片？  

#### -----

#### es 分布式架构原理
es是基于lucene的，已成为大部分互联网系统的标配
分布式：在每台机器启动es进程，多个进程组成一个集群。
es存储数据的基本单位是索引，一个索引相当于mysql的一张表，

index ->type->mapping->document->field
index: mysql一张表
type: 一个index一般有多个type。每个type字段都是差不多的，只有少量的区别。
比如 index是来放订单的，那么可以建两个type，虚拟商品和实物商品。
为什么一类type放在同一个index，为了性能 具体？
mapping：相当于type的表结构定义
一天document类似mysql表中的一行
field相当于一行数据某个字段的值

假设一个索引有三个shard，每个shard放索引的一部分数据，每个shard放在不同机器，以这样的方式构成分布式存储。
每个shard有primary shard 和 副本（replica）shard，来保证高可用，同一个shard primary 和replica放在了不同机器上。
如果一个es客户端写数据到primary shard中，primary shard会把数据同步到几个 replica shard 中。

es会在所有的进程中选择一个作为master节点，复制维护索引元数据，负责 primary replica 身份，如果一个非master节点宕机，
那么master节点会把宕机节点上的primary shard 身份转移到其他机器上的 peimary shard ，宕机的机器修复之后，master
节点会控制将缺失的 replica shard 分配过去，同步后续修改之类的，让集群恢复正常。当master 节点宕机了，那么会在剩下的节点中重新选举一个新的节点作为maseter。
会把宕机的 对于的replica 变为 permary shard。之前宕机的master 重启后会取消自己的master身份，会把其primary 变为 replica.

读，可以从 primary或者 replica中去读数据。
