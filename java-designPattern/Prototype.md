### 为什么要使用原型模式？
在有些场景下，需要重复创建多个实例，例如在循环体中赋值一个对象，此时就可以采用原型模式来优化对象的创建过程。
原型模式性能比new 实例化更快，因为 Object 类的 clone 方法是一个本地方法，它可以直接操作内存中的二进制流。


### 原型模式实现
```java
   // 实现 Cloneable 接口的原型抽象类 Prototype 
   class Prototype implements Cloneable {
        // 重写 clone 方法
        public Prototype clone(){
            Prototype prototype = null;
            try{
                prototype = (Prototype)super.clone();
            }catch(CloneNotSupportedException e){
                e.printStackTrace();
            }
            return prototype;
        }
    }
    // 实现原型类
    class ConcretePrototype extends Prototype{
        public void show(){
            System.out.println(" 原型模式实现类 ");
        }
    }

    public class Client {
        public static void main(String[] args){
            ConcretePrototype cp = new ConcretePrototype();
            for(int i=0; i< 10; i++){
                ConcretePrototype clonecp = (ConcretePrototype)cp.clone();
                clonecp.show();
            }
        }
    }
```  
* 实现 Cloneable 接口：在 JVM 中，只有实现了 Cloneable 接口的类才可以被拷贝，否则会抛出 CloneNotSupportedException 异常。
* 重写 Object 类中的 clone 方法：在 Java 中，所有类的父类都是 Object 类，而 Object 类中有一个 clone 方法，作用是返回对象的一个拷贝。
* 在重写的 clone 方法中调用 super.clone()：默认情况下，类不具备复制对象的能力，需要调用 super.clone() 来实现。  



### 问题及解决方式
对于一些属性（其它对象的引用以及 List 等类型的成员属性），则只能复制这些对象的引用了。所以简单调用 super.clone() 这种克隆对象方式，就是一种浅拷贝。
可以通过深拷贝来解决这种问题，基于浅拷贝来递归实现具体的每个对象的深拷贝。


### 应用场景
* 在循环体内创建对象，使用原型模式提高性能。
* Spring 中，@Service 默认都是单例的，用了私有全局变量，若不想影响下次请求，就需要用到原型模式，就可以可以通过以下注解来实现，@Scope(“prototype”)。

