### 为什么要使用 ConcurrentHashMap  
* **线程不安全的 HashMap** :多线程下使用 HashMap 的 put 操作会引起死循环，导致cpu利用率接近100%。因为多线程会使 HashMap 的 Entry 链表形成环形数据，
next节点永远不为空，就会产生死循环获取 Entry。  

* **HashMap效率低下**：所有的线程必须竞争同一把锁，当一个线程访问同步方法，其他线程只能阻塞或轮询。
当线程1使用put方法，线程2既不能使用 put 方法又不能使用 get 方法获取元素。

* **ConcurrentHashMap 锁分段技术能有效提高并发访问率**：将数据分为一段一段的存储，每段数据配一把锁，
当一个线程访问其中一段数据的时候，其他段的数据也能被其他线程访问。

### ConcurrentHashMap 结构

ConcurrentHashMap 包含一个 Segment 数组（一种可重入锁），Segment 结构与 HashMap 相似，是数据+链表结构，一个 Segment 元素包含一个 HashEntry 数组，每个HashEntry是一个链表结构的元素。当 HashEntry 数组数据需要被修改时，首先要获取他对应的 Segment 锁。

### get 操作
ConcurrentHashMap 的 get 操作如何做到不加锁了呢？因为它将需要使用的共享变量都用 volatile 修饰，
比如统计当前 Segment 的 count 字段，和存储值的 HashEntry 的 value。
使用 volatile 保证了可见性，使线程不会读到过期的值。 

### put 操作
put 操作共享变量时必须加锁，put 方法先会定义到 Segment，然后在 Segment 进行插入操作。  

* 判读是否需要扩容  

插入操作会先判断是否需要扩容，插入之前会判断数组是否超过了容量阈值，超过则进行扩容。 
扩容比HashMap更为恰当，HashMap 是在元素插入之后，才去判断元素是否达到容量的，达到了就去扩容，
如果扩容之后没有新元素插入，就相当于进行了一次无效扩容。  

* 如何扩容   

会创建一个容量大小2倍的数组，将原数组元素先散列再插入新数组中，为了高效，ConcurrentHashMap 不会对整个容器进行扩容，而是对某个 Segment 进行扩容。

### size操作

如果要统计整个 ConcurrentHashMap 的元素大小，就要把每个 Segment 的大小进行求和。  

由于 Segment 的 count 是用 volatile 修饰的,那么多线程下直接对各个 count 相加可不可以呢？虽然可以获取到 count 的最新值，但累加时 count 可能会发生变化，
这样就不准了。    

最安全的做法就是，统计size时将 put、remove、clean 方法全部锁住，显然这种方法是很低效的。   

ConcurrentHashMap 采用的是尝试两次不锁住 Segment 的方式来统计各个 Segment 的大小，因为累加过程中count变化的概率非常小，如果 count 发生了变化，则采用
加锁的方式来统计。怎么判断count发生了变化呢？ConcurrentHashMap 使用了 modCount 变量，每次 put、remove、clean 操作都会将 modCount +1,统计size 前后判断 modCount 是否发生了变化即可。

