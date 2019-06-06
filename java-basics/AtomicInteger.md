部分源码如下所示：
```Java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
```
，这里value使用了volatile关键字，volatile在这里可以做到的作用是使得多个线程可以共享变量，
但是问题在于使用volatile将使得VM优化失去作用，导致效率较低，所以要在必要的时候使用，因此AtomicInteger类不要随意使用，要在使用场景下使用。  

CAS算法是指先读取要修改的变量值，对它进行计算，然后执行检查并更新这个步骤（更新前判断那个值是否是之前那个读到的值），检查并更新是个原子操作
，它由硬件来保证可靠性，CAS算法的最后一步的方法就是Unsafe类的方法。因此，Unsafe类的方法可以说是Java高并发的各种扩展类的基础，
他们的底层都是调用Unsafe类的方法，Unsafe类为各种扩展类提供底层的原子操作。
