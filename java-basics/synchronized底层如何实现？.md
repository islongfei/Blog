

### 用户态与内核态的切换  
在操作系统中，与系统相关的一些特别关键性的操作必须由高级别的程序来完成，这样可以做到集中管理，减少有限资源的访问和使用冲突。
Linux操作系统中则主要采用了0和3两个特权级，也就是常说的内核态和用户态。
运行于用户态的进程可以执行的操作和访问的资源都受到极大的限制，而运行于内核态的进程则可以执行任何操作并且在资源的使用上也没有限制。  

很多程序开始时运行于用户态，但在执行的过程中，一些操作需要在内核权限下才能执行，用户态就会切换到切换到内核态。  
用户态切换到内核态的三种情况：
* 发生系统调用时
* 产生异常时
* 外设产生中断时
### synchronized原理
synchronized代码块是由一对monitorenter和monitorexit指令实现的，Monitor对象是同步的基本实现单元。  
在Java6之前，Monitor的实现完全是依靠操作系统内部的互斥锁，因为需要进行用户态到内核态的切换，所以同步操作是一个无差别的重量级操作。  

之后，Java 供了三种不同的Monitor实现，也就是常说的三种不同的锁：
* 偏斜锁（BiasedLocking）
* 轻量级锁
* 重量级锁

#### 锁的降级、升级
这是一种JVM优化synchronized运行的机制，当JVM检测到不同的竞争状况时，会自动切换到适合的锁实现，这种切换就是锁的升级、降级。  

1. 锁升级：偏斜锁->轻量级锁->重量级锁  

**偏斜锁**：当没有竞争出现时，默认会使用偏斜锁。JVM会利用CAS操作（compare and swap），
在对象头上设置线程ID，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。
这样做的假设是基于在很多应用场景中，大部分对象生命周期中最多会被一个线程锁定，使用偏斜锁可以降低无竞争开销。  

**轻量级锁**：如果有另外的线程试图锁定某个已经被偏斜过的对象，JVM+就需要撤销（revoke）偏斜锁，并切换到轻量级锁实现。轻量级锁依赖CAS操作对象头来试图获取锁。如果重试成功，就使用普通的轻量级锁；

**重量级锁**：如果上述重试不成功，就会进一步升级为重量级锁。

2. 锁降级：当JVM进入安全点（SafePoint）的时候，会检查是否有闲置的Monitor，然后试图进行降级。  

### 代码实现分析 
在 jvm底层的sharedRuntime.cpp 中，体现了synchronized的主要逻辑。
```C++
Handle h_obj(THREAD, obj);
  if (UseBiasedLocking) {
    // Retry fast entry if bias is revoked to avoid unnecessary inflation
    ObjectSynchronizer::fast_enter(h_obj, lock, true, CHECK);
  } else {
    ObjectSynchronizer::slow_enter(h_obj, lock, CHECK);
  }
```
首先jvm会先检查，是否启用了偏斜锁。fast_enter是完整锁获取路径，slow_enter则是绕过偏斜锁，直接进入轻量级锁获取逻辑。
那么通过slow_enter是如何实现有偏斜锁到轻量级锁呢？cpp代码如下：
```C++
void ObjectSynchronizer::slow_enter(Handle obj, BasicLock* lock, TRAPS) {
  markOop mark = obj->mark();
 if (mark->is_neutral()) {
       // 将目前的 Mark Word 复制到 Displaced Header 上
	lock->set_displaced_header(mark);
	// 利用 CAS 设置对象的 Mark Word
    if (mark == obj()->cas_set_mark((markOop) lock, mark)) {
      TEVENT(slow_enter: release stacklock);
      return;
    }
    // 检查存在竞争
  } else if (mark->has_locker() &&
             THREAD->is_lock_owned((address)mark->locker())) {
	// 清除
    lock->set_displaced_header(NULL);
    return;
  }
 
  // 重置 Displaced Header
  lock->set_displaced_header(markOopDesc::unused_mark());
  ObjectSynchronizer::inflate(THREAD,
                          	obj(),
                              inflate_cause_monitor_enter)->enter(THREAD);
}

```
根据代码总结为下：
1. 设置Displaced Header，然后利用CAS设置对象Mark Word，如果成功就成功获取轻量级锁。
2. 否则就重置Displaced Header，然后进入锁膨胀阶段。

流程： 
![image](https://github.com/islongfei/Blog/blob/master/images/Sync%E9%94%81%E6%B5%81%E7%A8%8B.png)
