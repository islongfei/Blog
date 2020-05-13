### 自己经常用的命令
* [JDK 1.8 支持的JVM参数官方文档](https://docs.oracle.com/javase/8/docs/technotes/tools/windows/java.html#BABHDABI)
* `top -H -p pid` 找到 CPU 使用率比较高的一些线程
* `java -XX:+PrintFlagsFinal` 查看所有jvm参数默认值
* `jinfo -flags pid` 查询运行的jvm参数
* `jinfo -flag PrintGCDetails pid` 查看某个JVM参数是否开启
* `jmap -heap pid` 查询jvm堆内存使用情况
* `jstat -gc pid 1000`  查询gc次数（包含full gc 次数，1000为1000ms刷新一次统计信息）
* `java -XX:+PrintFlagsFinal | grep manageable` 查看哪些参数可以动态修改
* `jinfo -flag +HeapDumpBeforeFullGC pid`  执行动态修改某些JVM参数

### 垃圾回收器选型
<img src="https://github.com/islongfei/Blog/blob/master/images/%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6%E5%99%A8.jpg" width="85%" hegiht="85%"  />  
在 JDK1.8 环境下，默认使用的是 Parallel Scavenge（年轻代）+Serial Old（老年代）垃圾回收器。   


看上去 Parallel Scavenge 与 ParNew都是并行处理，也都是用复制算法，**但是有什么区别呢**？  

Parallel Scavenge收集器有一个参数- XX：+UseAdaptiveSizePolicy当这个参数打开之后，就不需要手动指定新生代的大小，Eden和Survivor区的比例，晋升老年代对象等细节参数了，虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大吞吐量，这种调节方式成为GC自适应的调节策略。

### GC 的性能衡量指标
* **吞吐量**：GC 的吞吐量：系统总运行时间 = 应用程序耗时 +GC 耗时。如果系统运行了 100 分钟，
GC 耗时 1 分钟，则系统吞吐量为 99%。GC 的吞吐量一般不能低于 95%。
* **停顿时间**：指垃圾回收器正在运行时，应用程序的暂停时间。对于串行回收器而言，停顿时间可能会比较长；而使用并发回收器，
由于垃圾回收器和应用程序交替运行，程序的停顿时间就会变短，但其效率很可能不如独占垃圾回收器，系统的吞吐量也很可能会降低。
* **垃圾回收频率**：通常垃圾回收的频率越低越好，增大堆内存空间可以有效降低垃圾回收发生的频率，但同时也意味着堆积的回收对象越多，
最终也会增加回收时的停顿时间。所以要去适当地增大堆内存空间，保证正常的垃圾回收频率即可。 

### 查看 GC 日志  
* 打印 GC 日志  
`-XX:+PrintGC 输出 GC 日志`  
`-XX:+PrintGCDetails 输出 GC 的详细日志`  
`-XX:+PrintGCTimeStamps 输出 GC 的时间戳（以基准时间的形式）`  
`-XX:+PrintHeapAtGC 在进行 GC 的前后打印出堆的信息`  
`-Xloggc:../logs/gc.log 日志文件的输出路径`  

* 利用图形化界面工具查看GC: 如果是长时间的 GC 日志，就很难通过文本形式日志去查看整体的 GC 性能。图形化界面更容易直观排查,
可通过以下两款工具读取GC日志文件来值观显示吞吐量、停顿时间以及 GC 的频率，判断GC性能。
`GCView`  
`GCeasy`

### GC 调优策略
* **降低 Minor GC 频率**  
    通常情况下，由于新生代空间较小，Eden 区很快被填满，就会导致频繁 Minor GC，因此可以通过增大新生代空间，或调整新生代中的比例来降低 Minor GC 的频率。  
单次 Minor GC 时间是由两部分组成：T1（扫描新生代）和 T2（复制存活对象），扩容 Eden 区会使扫描时间增大，但减去了复制存活对象的时间。
通常在虚拟机中，复制对象的成本要远高于扫描成本，在特定情况下（长期存活的对象很多，每次需要复制移动时），扩容是相比比较高效的。  
如果堆中的短期对象很多，那么扩容新生代，单次 Minor GC 时间不会显著增加。

* **降低 Full GC 的频率**  
由于堆内存空间不足或老年代对象太多，会触发 Full GC，频繁的 Full GC 会带来上下文切换，增加系统的性能开销。 

什么时候会发生Full GC 呢？
1、当年轻代晋升到老年代的对象大小比目前老年代剩余的空间大小还要大时，此时会触发Full GC；  
2、当老年代的空间使用率超过某阈值时，此时会触发Full GC;  
3、当元空间不足时（JDK1.7永久代不足），也会触发Full GC;  
4、当调用System.gc()也会安排一次Full GC;  

由于大对象创建会直接进入老年代（可通过 -XX:PretenureSizeThreshold 指定进入老年代大对象的大小），
占用老年代空间，所以应合理减少大对象的创建，可以通过一下方式：  
`在业务上拆解大对象，拆解后进行分批查询或处理`  
`增大堆内存空间，可以设置初始化堆内存为最大堆内存`

一个web应用，多久一次Full GC才算正常呢?
根据具体的业务来分析，正常小对象且请求平缓的应用服务中，几天一次较为正常。如果有大量大对象创建或者承受高并发场景的服务，Full GC可能会更频繁。可以用
`jstat -gc pid 1000`来查看 full GC 的次数。

* **根据业务场景来选择 GC 回收器**
[各种 GC 回收器官方文档介绍](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/index.html)
假设业务要求系统相应时间要很快，可以选择响应速度较快的 CMS（Concurrent Mark Sweep）回收器和 G1 回收器，可参考 [CMS 与 G1 区别](https://github.com/islongfei/Blog/blob/master/java-basics/CMS%20%E5%92%8C%20G1%20%E7%9A%84%E5%8C%BA%E5%88%AB.md)。  
而当需求对系统吞吐量有要求时，就可以选择 Parallel Scavenge 回收器来提高系统的吞吐量。  

通常情况，JVM 是默认垃圾回收优化的，在没有性能衡量标准的前提下，尽量避免修改 GC 的一些性能配置参数。
如果一定要改，那就**必须基于大量的测试分析和线上的具体性能来进行调整**。





//Todo : jdk各版本选择的垃圾回收器对比及原因


