### ZooKeeper来源
起初雅虎项目都以动物来命名，整个分布式系统看上去就像一个大型的动物园一样，而Zookeeper正好要用来进行分布式环境的协调，所以ZooKeeper这个名字就诞生了。

### 相关概念
ZooKeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。  

ZooKeeper 是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、
命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。  

Zookeeper 一个最常用的使用场景就是用于担任服务生产者和服务消费者的注册中心(提供发布订阅服务)，Dubbo就是用ZooKeeper来作为服务发现与注册中心，
如下图所示：  

![image](https://github.com/islongfei/Blog/blob/master/images/ZooKeeper%E5%8E%9F%E7%90%86.jpg)

为什么最好使用奇数台服务器构成 ZooKeeper 集群？  

由于ZooKeeper半数以上节点存活ZooKeeper 就能正常服务的机制，所以使用奇数台服务器，更省资源。比如部署4台服务器，要正常服务就要半数以上存活，只允许挂掉1台，用3台也只允许挂掉1台，那么为什么不去节省资源使用3台服务器达到同样的效果呢。  


### 相关机制
* ZooKeeper 本身就是一个分布式程序，只要半数以上节点存活，ZooKeeper 就能正常服务。  

* ZooKeeper 将数据保存在内存中，这也就保证了 高吞吐量和低延迟。但是内存限制了能够存储的容量不太大，这也是znode中存储的数据量较小的原因。  

* ZooKeeper 是高性能的。 在读多写少的场景中性能最好，因为“写”会使所有的服务器去同步状态。  

* 临时节点：它的生命周期和客户端会话绑定，一旦客户端会话失效，那么这个客户端创建的所有临时节点都会被移除。  

* 持久节点：指一旦这个ZNode被创建了，除非主动进行ZNode的移除操作，否则这个节点将一直保存在Zookeeper上。  

* ZooKeeper 底层其实只提供了两个功能：①管理（存储、读取）用户程序提交的数据；②为用户程序提供数据节点监听服务。  

### 重要概念
#### Session
Session 指的是 ZooKeeper 服务器与客户端会话，客户端与服务端使用长连接。
通过这个连接，客户端能够通过心跳检测与服务器保持有效的会话，也能够向Zookeeper服务器发送请求并接受响应，同时还能够通过该连接接收来自服务器的Watch事件通知。  

可以用`sessionTimeout`设置会话超时时间，当客户端异常断开时（比如网络故障），只要在sessionTimeout规定的时间内能够重新连接上集群中任意一台服务器，那么之前创建的会话仍然有效。  

#### Znode
在Zookeeper中，“节点"分为两类，第一类同样是指构成集群的机器，我们称之为`机器节点`；第二类则是指数据模型中的`数据单元`，我们称之为数据节点ZNode。ZNode也分为两类：`持久节点`和`临时节点` 

Zookeeper将所有数据存储在内存中，数据模型是一棵树（Znode Tree)，由斜杠（/）的进行分割的路径，就是一个Znode，例如/app1/p_1。每个上都会保存自己的数据内容，同时还会保存一系列属性信息。Znode Tree结构模型如下所示：  
![image](https://github.com/islongfei/Blog/blob/master/images/ZooKeeper%E8%8A%82%E7%82%B9%E6%A8%A1%E5%9E%8B.jpg)

#### 版本
Zookeeper 的每个 ZNode 上都会存储数据，对应于每个ZNode，Zookeeper 都会为其维护一个叫作 Stat 的数据结构，Stat 中记录了这个 ZNode 的三个数据版本，分别是`version`（当前ZNode的版本）、`cversion`（当前ZNode子节点的版本）和 `aversion`（当前ZNode的ACL版本）。  

#### Watcher
Watcher（事件监听器），Zookeeper允许用户在指定节点上注册一些Watcher，并且在一些特定事件触发的时候，ZooKeeper服务端会将事件通知到感兴趣的客户端上去，该机制是Zookeeper实现分布式协调服务的重要特性。  

#### ACL
ACL（AccessControlLists）是Zookeeper的权限控制策略，其定义了5种权限： 

* CREATE：创建子节点权限； 
* READ: 获取节点数据和子节点列表的权限；
* WRITE: 更新节点的权限；
* DELETE: 删除子节点的权限；
* ADMIN: 设置节点的ACL权限； 

注意：CREATE和DELETE这两种权限都是针对子节点的权限控制。 

### 特点

* 顺序一致性：从同一客户端发起的事务请求，由于`zxid`的存在，最终将会严格地按照顺序被应用到 ZooKeeper 中去。  

对于来自客户端的每个更新请求，ZooKeeper 都会分配一个全局唯一的递增编号，这个编号反应了所有事务操作的先后顺序，应用程序可以使用 ZooKeeper 这个特性来实现更高层次的同步原语。 这个编号也叫做时间戳——`zxid`（Zookeeper Transaction Id）  

* 原子性：所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，也就是说，要么整个集群中所有的机器都成功应用了某一个事务，要么都没有应用。 

* 单一系统映像 ：无论客户端连到哪一个 ZooKeeper 服务器上，其看到的服务端数据模型都是一致的。  

* 可靠性：一旦一次更改请求被应用，更改的结果就会被持久化，直到被下一次更改覆盖。  

### 集群
与传统集群方式 Master/Slave 模式（主从模式）类似，ZooKeeper引入了`Leader`、`Follower` 和 `Observer` 三种角色的集群方式。如下图所示：
![image](https://github.com/islongfei/Blog/blob/master/images/ZooKeeper%E9%9B%86%E7%BE%A4%E6%A8%A1%E5%BC%8F.jpg)  

ZooKeeper 集群中的所有机器通过一个 Leader 选举过程来选定一台称为 “Leader” 的机器，Leader 既可以为客户端提供写服务又能提供读服务。  
Follower 和 Observer 都只能提供读服务。Follower 和 Observer 唯一的区别在于 Observer 机器不参与 Leader 的选举过程，也不参与写操作的“过半写成功”策略，因此 Observer 机器可以在不影响写性能的情况下提升集群的读性能。  

各角色职责如下图所示：
![image](https://github.com/islongfei/Blog/blob/master/images/ZooKeeper%E8%A7%92%E8%89%B2.jpg)  

### Leader选举
当 Leader 服务器出现网络中断、崩溃退出与重启等异常情况时，ZAB 协议就会进人恢复模式并选举产生新的Leader服务器。这个过程大致是这样的：  

1、`Leader election（选举阶段）`：节点在一开始都处于选举阶段，只要有一个节点得到超半数节点的票数，它就可以当选准 leader。
2、`Discovery（发现阶段）`：在这个阶段，followers 跟准 leader 进行通信，同步 followers 最近接收的事务提议。
3、`Synchronization（同步阶段）`:同步阶段主要是利用 leader 前一阶段获得的最新提议历史，同步集群中所有的副本。同步完成之后 准 leader 才会成为真正的 leader。
4、`Broadcast（广播阶段）`: 到了这个阶段，Zookeeper 集群才能正式对外提供事务服务，并且 leader 可以进行消息广播。同时如果有新的节点加入，还需要对新节点进行同步。








