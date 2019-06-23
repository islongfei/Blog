单例模式：一个类只能构建一个对象的设计模式。 

应用场景：有时候需要推迟一些高开销的对象初始化操作，并且只有在使用时才对这些对象进行初始化。
下面来说几个线程安全的单例模式： 

### volatile+双检锁的单例模式
```Java
/**
 * @Description: 单例模式
 * @Auther: longfei.wang
 * @Date: 2019-6-21
 */
public class Singleton {
    private Singleton() {
    }  //私有构造函数

    private volatile static Singleton instance = null;  //单例对象

    //静态工厂方法
    public static Singleton getInstance() {
        if (instance == null) {      //双重检测机制
            synchronized (Singleton.class) {  //同步锁
                if (instance == null) {     //双重检测机制
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

}

```
解释：
* 1.要想让一个类只能构建一个对象，自然不能让它随便去做new操作，因此Signleton的构造方法是私有的。   

* 2.instance是单例对象，初始值为null(懒汉模式),也可以初始为new Singleton()(饿汉模式)  

* 3.使用synchronized，为了避免多个线程下的线程安全问题（比如线程A执行if (instance == null)的同时 ，线程B已经执行了 instance = new Singleton()，就可能会出现线程A认为对象还没完成初始化，再次去new Singleton()），所以用同步锁锁住整个类，避免new Singleton()被执行多次。  

* 4.使用双检锁，如果用synchronized修饰整个getInstance代码块的话，这样也可以实现同步，但是synchronized存在着巨大的性能开销，所以要去减少同步锁所修饰的代码块。当第一次检查instance不为null时，就不需要执行加锁和初始话操作，这样可以降低syncronized带来的性能开销。  

* 5.使用volatile，原因：在第一次判断if (instance == null)时，读取到instance不为null,这时instance引用的对象可能还没完成初始化。在创建对象时可分解为3个动作：1：分配对象内存空间，2：初始化对象，3：将instance指向内存地址，但操作系统为了提高效率可能会进行重排序，先执行3再执行2。这样线程A可能会访问到未初始化的对象。为了避免这种情况，所以使用volatile避免指令重排。  

### 使用静态内部类实现单例模式
```Java
public class Singleton {
    private static class LazyHolder {
        private static final Singleton INSTANCE = new Singleton();
    }
    private Singleton (){}
    public static Singleton getInstance() {
        return LazyHolder.INSTANCE;
    }
}
```
解释：从外部无法访问静态内部类LazyHolder,只有调用getInstance才能得到单例对象。单例对象初始化并不是再类初始化的时候，而是在调用getInstance时候，这种模式利用了classloader的加载机制实现懒加载，从而保证线程安全。   

缺点：无法防止利用反射来重复构建对象。

### 用枚举实现单例模式
```Java
public enum SingletonEnum {
    INSTANCE;
}
```
解释：由于enum的语法糖，JVM会阻止反射获取枚举类的私有构造方法，解决了上个方法的缺点，并且可以保证线程安全。  而且可以在美剧类反序列化的时候，保证反序列返回的结果是同一对象。  

缺点：不是懒加载，单例对象时在枚举类被加载的时候初始化的。





