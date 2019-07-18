### ThreadLocal介绍
ThreadLocal，很多地方叫做线程本地变量，也有些地方叫做线程本地存储，ThreadLocal 的作用是提供线程的私有变量，这种变量可以在一个线程的整个生命周期中传递，可以减少一个线程在多个函数或类中创建公共变量来传递信息，避免了复杂度。
ThreadLocal为变量在每个线程中都创建了一个副本，每个线程可以访问自己内部的副本变量。  

ThreadLocal使用 get() 和 set() 方法来获取默认值或将其值更改为当前线程所存的副本的值，从而避免了线程安全问题，remove()用来移除当前线程中变量的副本。  

每个Thread中都具备一个ThreadLocalMap，而ThreadLocalMap可以存储以ThreadLocal为key的键值对。
这里就是为什么每个线程访问同一个ThreadLocal，得到的确是不同的数值。另外，ThreadLocal 是 map结构是为了让每个线程可以关联多个 ThreadLocal变量。

### 内存泄露问题
ThreadLocalMap 中使用的 key 为 ThreadLocal 的弱引用,而 value 是强引用。
所以，如果 ThreadLocal 没有被外部强引用的情况下，在垃圾回收的时候会 key 会被清理掉，而 value 不会被清理掉。  

这样一来，ThreadLocalMap 中就会出现key为null的Entry。假如我们不做任何措施的话，value 永远无法被GC 回收，这个时候就可能会产生内存泄露。
ThreadLocalMap实现中已经考虑了这种情况，在调用 set()、get()、remove() 方法的时候，会清理掉 key 为 null 的记录。
使用完 ThreadLocal方法后 最好手动调用remove()方法

