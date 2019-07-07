### CountDownLatch
CountDownLatch：允许一个或多个线程等待其他线程操作完成。  
CountDownLatch 实现类似 join 的功能，但比join的功能更多。  
当其他线程调用 .countDown()方法，创建 CountDownLatch 指定的计数器就会 -1 ，CountDownLatch 的 await 方法会阻塞当前线程，直到计数器为 0 。  
可以使用带超时时间的 await 方法，如果超时，就不再阻塞当前线程。

### CyclicBarrier
CyclicBarrier: 可循环使用的同步屏障，一组线程中的每个线程达到屏障时会被阻塞，直到最后一个线程达到屏障时，整组线程的屏障才会被取消，随后正常运行。  
每个线程调用 await 方法告诉 CyclicBarrier我已到达屏障，然后当前线程就会被阻塞。  
可以在 CyclicBarrier 初始化参数中指定 barrierAction(Runnable),用于当一组线程都到达屏障时，优先执行 barrierAction 。    

**应用场景**：统计用户的日均银行流水，一组中的每个线程分别统计每年的流水，等每个线程统计完之后，再用 barrierAction 去整合每年的流水，算出日均流水。   

**CyclicBarrier 与 CountDownLatch 的区别**：CountDownLatch 的计数器只能使用一次，而 CyclicBarrier 计数器可以使用 reset() 重置，重复使用。
CyclicBarrier 自带的 getNumberWaiting 方法可以获得阻塞线程的数量，isBroken 方法可以查看阻塞的线程是否被中断。  


### Semaphore
Semaphore 用于控制同时访问特定资源的线程数量，可以在初始化时指定具体数量。

**应用场景**：流量控制：比如控制数据库的连接数。  

### Exchanger
Exchanger 用于线程间的数据交换，它提供了一个同步点，在这个同步点两个线程可以交换数据。  
第一个线程执行 exchange() 方法，当第二个线程也执行了 exchange() 方法时，两个线程就到达了同步点，这时就可以交换数据。

**应用场景**：遗传算法（交换两个交配对象的数据，使用交叉规则得到交配结果）；校对工作（查看前后记录是否一致）。
