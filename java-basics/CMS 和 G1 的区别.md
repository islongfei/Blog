
### CMS
CMS 垃圾收集器是基于 `标记清除算法` 实现的，目前主要用于老年代垃圾回收，CMS只会回收老年代和永久代（元数据区）的对象。CMS 收集器的 GC 周期主要由 7 个阶段组成，其中有两个阶段会发生 STW (stop-the-world,暂停所有应用线程)，其它阶段都是并发执行的。   

### G1
G1 垃圾收集器是基于 `标记整理算法` 实现的，是一个分代垃圾收集器，既负责年轻代，也负责老年代的垃圾回收。在 jdk1.9 被设置为默认的收集器。
跟之前各个分代使用连续的虚拟内存地址不一样，G1 使用了一种 Region 方式对堆内存进行了划分，
但每一代使用的是 N 个不连续的 Region 内存块，每个 Region 占用一块连续的虚拟内存地址。  

同样也分 Eden、urvivor、Old，还多了一个 Humongous 区域，用于存储特别大的对象,G1 内部做了一个优化，一旦发现没有引用指向巨型对象，
则可直接在年轻代的 YoungGC 中被回收掉。  

G1 分为 Young GC、Mix GC 以及 Full GC。
Young GC 主要是在 Eden 区进行，当 Eden 区空间不足时，则会触发一次 Young GC，Young GC 的执行是并行的，期间会发生 STW。
当堆空间的占用率达到一定阈值后会触发 G1 Mix GC，阈值由命令参数 `-XX:InitiatingHeapOccupancyPercent `设定，默认值 45）

### CMS 和 G1 区别

* CMS 主要集中在老年代的回收，而 G1 集中在分代回收，包括了年轻代的 Young GC 以及老年代的 Mix GC；
* G1 使用了 Region 方式对堆内存进行了划分，且基于标记整理算法实现，整体减少了垃圾碎片的产生；
* 在初始化标记阶段，搜索可达对象使用到的 Card Table，其实现方式不一样；
* 解决并发标记（三色标记算法）时漏标的方式也不一样。 

### G1 相对于 CMS 的优势
* 并行与并发：G1能够更充分利用多CPU、多核环境运行
* 分代收集：G1虽然也用了分代概念，但相比其他收集器需要配合不同收集协同工作，但G1收集器能够独立管理整个堆
* 空间管理：与CMS的标记一清理算法不同，G1从整体上基于标记一整理算法，将整个Java堆划分为多个大小相等的独立区域（Region）,这种算法能够在运行过程中不产生内存碎片
* 可预测的停顿：降低停顿时间是G1和CMS共同目标，但是G1追求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定一个长度为M毫秒的时间片段内，消耗在垃圾收集器上的时间不得超过N毫秒。在规定的停顿时间内，如果回收不完，那就选择性的回收呗。所以这也就是g1为什么能够被1.9当成默认收集器的原因吧

**Card Table** ：  
在垃圾回收的时候都是从 Root 开始搜索，这会先经过年轻代再到老年代，也有可能老年代引用到年轻代对象，
这种属于跨代处理，非常消耗性能。为了避免在回收年轻代时跨代扫描整个老年代，CMS 和 G1 都用到了 Card Table 来记录这些引用关系。
只是 G1 在 Card Table 的基础上引入了 RSet，每个 Region 初始化时，都会初始化一个 RSet，RSet 记录了其它 Region 中的对象引用本 Region 对象的关系。

**三色标记算法**：  
* 黑色：根对象，或者对象和对象中的子对象都被扫描；
* 灰色：对象本身被扫描，但还没扫描对象中的子对象；
* 白色：不可达对象。  

**解决漏标**：  

三色标记法会有漏标的问题：当一个白色标记对象，在垃圾回收被清理掉时，
正好有一个对象引用了该白色标记对象，此时由于被回收掉了，就会出现对象丢失的问题。   

CMS 采用了 Incremental Update 算法，只要在写屏障（write barrier）里发现一个白对象的引用被赋值到一个黑对象的字段里，那就把这个白对象变成灰色的。
而在 G1 中，采用的是 SATB 算法，该算法认为开始时所有能遍历到的对象都是需要标记的，即认为都是活的
