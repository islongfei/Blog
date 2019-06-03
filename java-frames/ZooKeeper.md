### 名字来源
起初雅虎项目都以动物来命名，整个分布式系统看上去就像一个大型的动物园一样，而Zookeeper正好要用来进行分布式环境的协调，所以ZooKeeper这个名字就诞生了。

### ZooKeeper相关概念
ZooKeeper 的设计目标是将那些复杂且容易出错的分布式一致性服务封装起来，构成一个高效可靠的原语集，并以一系列简单易用的接口提供给用户使用。  

ZooKeeper 是一个典型的分布式数据一致性解决方案，分布式应用程序可以基于 ZooKeeper 实现诸如数据发布/订阅、负载均衡、
命名服务、分布式协调/通知、集群管理、Master 选举、分布式锁和分布式队列等功能。  

Zookeeper 一个最常用的使用场景就是用于担任服务生产者和服务消费者的注册中心(提供发布订阅服务)，Dubbo就是用ZooKeeper来作为服务发现与注册中心，
如下图所示：  

![image](https://github.com/islongfei/Blog/blob/master/images/ZooKeeper%E5%8E%9F%E7%90%86.jpg)
