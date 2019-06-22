单例模式：一个类只能构建一个对象的设计模式。  
下面来说几个线程安全的单例模式： 

### volatile+双重检测+同步锁的单例模式
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
//todo
