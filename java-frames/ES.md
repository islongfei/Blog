`总结内容方向：`
* Lucene 
* 倒排索引
* 全文检索
* es 分布式架构原理
* es 写入和查询的工作原理
* es 在数据量很大情况下（亿级）如何提高性能
* es 生产集群部署架构，每个索引数据量大概多少，每个索引有多少个分片？  

### 一、es 分布式架构原理
es是基于lucene的，已成为大部分互联网系统的标配
分布式：在每台机器启动es进程，多个进程组成一个集群。
es存储数据的基本单位是索引，一个索引相当于mysql的一张表，

**index ->type->mapping->document->field**   
* index: mysql一张表。  
* type: 一个index一般有多个type。每个type字段都是差不多的，只有少量的区别。  
比如 index是来放订单的，那么可以建两个type，虚拟商品和实物商品。
为什么一类type放在同一个index，为了性能 具体？
* mapping：相当于type的表结构定义。
* 一个document类似mysql表中的一行。 
* field相当于一行数据某个字段的值。  

假设一个索引有三个shard，每个shard放索引的一部分数据，每个shard放在不同机器，以这样的方式构成分布式存储。
每个shard有primary shard 和 副本（replica）shard，来保证高可用，同一个shard primary 和replica放在了不同机器上。
如果一个es客户端写数据到primary shard中，primary shard会把数据同步到几个 replica shard 中。

es会在所有的进程中选择一个作为master节点，复制维护索引元数据，负责 primary replica 身份，如果一个非master节点宕机，
那么master节点会把宕机节点上的primary shard 身份转移到其他机器上的 peimary shard ，宕机的机器修复之后，master
节点会控制将缺失的 replica shard 分配过去，同步后续修改之类的，让集群恢复正常。当master 节点宕机了，那么会在剩下的节点中重新选举一个新的节点作为maseter。
会把宕机的 对于的replica 变为 permary shard。之前宕机的master 重启后会取消自己的master身份，会把其primary 变为 replica.

读，可以从 primary或者 replica中去读数据。

### 二、es 写入原理
es 写数据的核心为 **refresh、flush、translog、merge** 。   
写数据随便挑一个机器的 进程节点去写，这个节点被叫做协调节点。协调节点会对这条数据进行hash，hash完了之后它把这条数据路由到对应的primary shard中，primary shard 将数据同步到其他 replica shard 上去。 如果primary 和 replica 都写完了，就会告诉客户端数据写入完成。  

**1、refresh**     
primary shard 拿到数据后，将数据写入 内存 buffer 中，同时会写入（refresh）到 translog 日志文件中，此时客户端要查这条数据是查不到的。  
内存buffer是会写满的，es会定时将内存buffer中的数据写入一个新的 segment file(磁盘文件)，数据写入到segment file 中后，同时会建立好倒排索引，默认1s写一次，但是buffer没数据的话，就不会执行 refresh 操作。  
操作系统里面，磁盘文件会有 操作系统级别的缓存（os cache）,数据写入磁盘文件之前，会先进入 os cache 中，数据被写入os cache中，buffer就会被清空了，因为不需要保留buffer了，数据在translog里面已经持久化到磁盘去一份了。只要buffer中的数据被 refresh 到 os cache 中，客户端就可以搜索到刚写入的数据了，这也为什么es是称作准实时的，默认是每隔1秒refresh一次的，所以es是**准实时**的，因为写入的数据1秒之后才能被看到。

**2、flush** 

随着 translog 变的越来越大，当translog 达到一定程度时，就会出发 commit 操作。
commit 执行的两个条件：tanslog大到阈值、定期执行commit（默认30min）,这个commit全过程叫做flush操作。也以以通过es api 手动执行 flush 操作。  
1. commit操作发生第一步，就是将buffer中现有数据refresh到os cache中去，清空buffer。  
2. 将一个commit point写入磁盘文件，里面标识着这个时间之前刷入 os cache 的所有数据。  
3. 强行将os cache中目前缓存所有的数据都fsync到磁盘文件 segment file 中去。  
4. 将现有的translog清空，然后再次重启启用一个translog。  


**3、merge**  
buffer每次refresh一次，就会产生一个segment file，所以默认情况下是1秒钟一个segment file，segment file会越来越多，此时会定期执行merge。
每次merge的时候，会将多个segment file合并成一个大的segment file，同时这里会将标识为deleted的document给物理删除掉，然后将新的segment file写入磁盘，这里会写一个commit point，标识所有新的segment file，然后打开segment file供搜索使用，同时删除旧的segment file。

**translog 日志文件作用**   
在执行commit操作之前，数据要么是停留在buffer中，要么是停留在os cache中，无论是buffer还是os cache都是内存，一旦这台机器死了，内存中的数据就全丢了。
所以需要将数据对应的操作写入一个专门的日志文件，translog日志文件中，一旦此时机器宕机，再次重启的时候，es会自动读取translog日志文件中的数据，恢复到内存buffer和os cache中去。

**es 可能会丢失数据的场景**  
translog其实也是先写入os cache的，默认每隔5秒刷一次到磁盘中去，所以默认情况下，可能有5秒的数据会仅仅停留在buffer或者translog文件的os cache中，如果此时机器挂了，会丢失5秒钟的数据。但是这样性能比较好，最多丢5秒的数据。也可以将translog设置成每次写操作必须是直接fsync到磁盘，但是性能和吞吐量会差很多。

**es 删除数据**    
如果是删除操作，commit的时候会在磁盘中生成一个.del文件，里面将某个document标识为deleted状态，那么搜索的时候根据.del文件就知道这个doc被删除了。

### 三、 es 读数据原理  
查询，GET某一条数据，写入了某个document，这个document会自动给你分配一个全局唯一的id，doc id，同时也是根据doc id进行hash路由到对应的primary shard上面去。也可以手动指定doc id，比如用订单id，用户id。也可以通过doc id来查询，会根据doc id进行hash，判断出来当时把doc id分配到了哪个shard上面去，从那个shard去查询。  
具体过程如下： 
1. 客户端发送请求到任意一个node（负载均衡），成为coordinate node（协调节点）。  
2. coordinate node对document进行路由，将请求转发到对应的node，此时会使用round-robin随机轮询算法，在primary shard以及其所有replica中随机选择一个，让读请求负载均衡。
3. 接收请求的node返回document给coordinate node。
4. coordinate node返回document给客户端。  

### 四、 es 搜索数据原理  

es最强大的是做全文检索，就是比如有三条数据：`java真好玩、java哈哈哈、j2ee嘿嘿嘿`。根据java关键词来搜索，将包含java的document给搜索出来，es就会给你返回：`java真好玩、java哈哈哈`。  

具体过程如下：  
1. 客户端发送请求到一个coordinate node。  
2. 协调节点将搜索请求转发到所有的shard对应的primary shard或replica shard也可以。  
3. query phase：每个shard将自己的搜索结果（其实就是一些doc id），返回给协调节点，由协调节点进行数据的合并、排序、分页等操作筛选出最匹配的哪些数据，产出最终结果。  
4. fetch phase：接着由协调节点，根据doc id去各个节点上拉取实际的document数据，最终返回给客户端。  

















