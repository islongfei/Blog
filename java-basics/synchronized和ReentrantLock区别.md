### 什么是线程安全？  
就是保证多线程环境下共享的、可修改的状态的正确性。  
如果线程不是共享的或是不可修改的，也就不存在线程安全问题。可以通过两个方法保证线程的安全：
* 封装：通过封装将内部状态隐藏起来。
* 不可变：使用final 和 immutable。  

线程安全三个特性：
* 原子性：简单说就是相关操作不会中途被其他线程干扰，一般通过同步机制实现。
* 可见性：是一个线程修改了某个共享变量，其状态能够立即被其他线程知晓，volatile 就是负责保证可见性的。
* 有序性：是保证线程内串行语义，避免指令重排。
 
 
 ### synchronized
 synchronized可以修饰方法和代码块，本质上 synchronized 方法等同于把方法全部语句用synchronized 块包起来。
 ### ReentrantLock 
ReentrantLock 再入锁，可直接调用lock()方法获取，ReentrantLock可以做到更多的细节控制和公平性，要明确使用unlock()方法释放。 
它是表示当一个线程试图获取一个它已经获取的锁时，这个获取动作就自动成功，这是对锁获取粒度的一个概念，也就是锁的持有是以线程为单位而不是基于调用次数。  
再入锁可以设置公平性，如下
```Java
ReentrantLock fairLock = new ReentrantLock(true);
```
当公平性为真时，会倾向于将锁赋予等待时间最久的线程。公平性是减少线程“饥饿”（个别线程长期等待锁，但始终无法获取）情况发生的一个办法.  

为保证锁释放，每一个+lock()+动作，建议都立即对应一个 try-catch-finally
```Java
ReentrantLock fairLock = new ReentrantLock(true);// 创建公平锁，一般情况不需要,因为影响性能。
fairLock.lock();
try {
	// do something
} finally {
 	fairLock.unlock();
}

```

ReentrantLock的细节控制：
* 带超时的获取锁尝试。
* 可以判断是否有线程，或者某个特定线程，在排队等待获取锁。
* 可以响应中断请求  



