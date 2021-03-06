### 为什么要使用MQ
* **异步**: 减少接口调用的感知时间，加快网站响应速度。对于用户异步可以提高用户界面的体验，相比同步用户几乎无感知
* **解耦**：如果多个系统耦合在一起，并且维护的成本非常高，通过mq发布订阅模型（Pub/Sub 模型）进行解耦。如果实际有这种场景，可以用mq去解耦优化。
* **削峰填谷**: 减少高峰时期的数据库的并发量，减少服务器负载压力，保证数据库不会挂掉（一般的mysql瓶颈在2000个请求/s，具体视配置而定），高峰过后会有消息积压消费消息的速度还会和之前一样，直到消费完积压的消息。

### 使用MQ有什么优缺点
* 优点： 异步、解耦、削峰填谷。
* 缺点： 1. 系统可用性降低（需要考虑到MQ挂掉的场景）、2. 系统复杂度提高（需要考虑到消息重复消费、消息丢失、消息顺序性的问题）、3. 数据一致性问题（发完消息数据库写入失败）。

### 怎样保证MQ消息不丢失
 **1. 生产者弄丢数据：** 
 
生产者发送消息到MQ时，可能数据在网络传输中搞丢了，这时MQ收不到消息，消息就丢了。  
解决方式如下：
* 事务方式：在生产者发送消息之前开启一个事务，MQ收到了这个消息，再去提交事务，缺点：生产者的吞吐量和性能都会降低。
* confirm机制：生产者每次写消息的时候会分配一个唯一的id，然后RabbitMQ收到之后会回传一个ack，告诉生产者这个消息ok了，
如果MQ没有处理到这个消息，那么就回调接口，这个时候生产者就可以重发。  

事务机制和cnofirm机制最大的不同在于事务机制是同步的，提交一个事务之后会阻塞在那儿，但是confirm机制是异步的，
发送一个消息之后就可以发送下一个消息，然后那个消息MQ接收了之后会异步回调生产者一个接口通知上传者这个消息接收到了。一般都会去使用confirm机制。

 **2. MQ弄丢数据：**   
 
 如果MQ宕机了，这个时候消息就会丢失。   
 解决方式如下： 
 * 持久化机制：消息写入之后会持久化到磁盘。
 * 配合confirm：如果及持久化到磁盘就挂掉了，那么就要配合confirm机制一起来保证的，就是在消息持久化到磁盘之后才会给生产者发送ack消息。  
 
  **3. 消费端弄丢数据：**   
  
  在消费消息的时候，刚拿到消息，结果进程挂了，这个时候MQ就会认为你已经消费成功了，这条数据就丢了。  
  解决方式如下： 
  * 在消费者收到消息的时候，会发送一个ack给MQ，告诉MQ这条消息被消费到了，这样MQ就会把消息删除。
 但是默认情况下这个发送ack的操作是自动提交的，也就是说消费者一收到这个消息就会自动返回ack给MQ，所以会出现丢消息的问题。
所以针对这个问题的解决方案就是：在消费者处理完这条消息之后再手动提交ack。

### 大量消息在 MQ 积压怎么办  
当消费者挂掉了，mq积压了比如几千万数据，即使过段时间消费者正常了，mq也需要很长时间恢复（去消费完之前堆积的大量消息）。  

 **1. 方案一 想办法快速消费**  
 先修复好消费者的问题，确保恢复消费速度。新建一个topic，partition是原来的数倍，然后写一个临时的分发数据的consumer程序去消费积压的数据（不做耗时的数据库操作，只做转发处理，轮询写入临时建立好的数倍数量的queue）。然后临时加机器部署消费者，这样消费的速度可以提升数倍。比如临时将queue资源和consumer资源扩大10倍，以正常的10倍速度来消费数据。   
 
 **2. 方案二 如果出现消息过期情况，低峰期把消息补回去**   
比如rabbitMq是可以设置过期时间的（很坑一般没人设置），等过了高峰期以后，反查出丢失的那批数据，写个临时程序，一点一点的查出来，然后重新灌入mq里面去，把高峰期丢的数据给他补回来。  

 **3. 方案三 无法去快速消费积压的消息，先把消息给扔掉**   
 如果mq磁盘即将写满，而且也无法去快速消费积压的消息怎么办？可以临时写程序，只来消费数据什么耗时操作都不要做，消费一个丢弃一个，都不要了，快速消费掉所有的消息。然后走第二个方案，到了晚上业务低峰期再补数据。

 
