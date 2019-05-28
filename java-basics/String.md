### String被final修饰的好处

* 第一，保证 String 对象的安全性。假设 String 对象是可变的，那么 String 对象将可能被恶意修改。
* 第二，保证 hash 属性值不会频繁变更，确保了唯一性，使得类似 HashMap 容器才能实现相应的 key-value 缓存功能。
* 第三，可以实现字符串常量池。在 Java 中，通常有两种创建字符串对象的方式，一种是通过字符串常量的方式创建，
 如 String str=“abc”；另一种是字符串变量通过 new 形式的创建，如 String str = new String(“abc”)。
 
 当代码中使用第一种方式创建字符串对象时，JVM 首先会检查该对象是否在字符串常量池中，如果在，就返回该对象引用
 ，否则新的字符串将在常量池中被创建。这种方式可以减少同一个值的字符串对象的重复创建，节约内存。  
 
 
 
 ### String 对象的优化
 
 * 怎样有效拼接字符串  
 
 使用 + 号作为字符串的拼接，是被编译器优化成 StringBuilder 的方式去拼接的。
 每次循环都会生成一个新的 StringBuilder 实例，同样也会降低系统的性能。所以平时做字符串拼接的时候，建议还是要显示地使用 String Builder 来提升系统性能。
 多线程下，String 对象的拼接涉及到线程安全，可以使用 StringBuffer，但性能上会相对差些。
 
 
* 使用 String.intern 节省内存  

在字符串常量中，默认会将对象放入常量池（String str1= "abc"），
在字符串变量中，对象是会创建在堆内存中（String str2= new String("abc")），同时也会在常量池中创建一个字符串对象，
复制到堆内存对象中，并返回堆内存对象引用。
如果调用 intern 方法，会先去查看字符串常量池中是否有等于该对象的字符串，如果没有，就在常量池中新增该对象，并返回该对象引用；
如果有，就返回常量池中的字符串引用。堆内存中原有的对象由于没有引用指向它，将会通过垃圾回收器回收。  

注意：一定要结合实际场景。因为常量池的实现是类似于一个 HashTable 的实现方式，HashTable 存储的数据越大，遍历的时间复杂度就会增加。
如果数据过大，会增加整个字符串常量池的负担。 


* 怎样有效去切割字符串  

Split() 方法使用了正则表达式实现了其强大的分割功能，
split有两种情况不会使用正则表达式：  
第一种为传入的参数长度为1，且不包含“.$|()[{^?*+\\”regex元字符的情况下，不会使用正则表达式； 
第二种为传入的参数长度为2，第一个字符是反斜杠，并且第二个字符不是ASCII数字或ASCII字母的情况下，不会使用正则表达式。 

而正则表达式的性能是非常不稳定的，使用不恰当会引起回溯问题，很可能导致 CPU 居高不下。 
所以应该慎重使用 Split() 方法，可以用java.util.StringTokener与String.indexOf() 方法代替 Split() 方法完成字符串的分割。  

性能方面：Vector & indexOf() >java.util.StringTokener()>Split() [性能比较]（https://ben-sin.iteye.com/blog/659611）  

如果实在无法满足需求，你就在使用 Split() 方法时，对回溯问题加以重视就可以了。


`小测试`
```Java
String str1= "abc";
String str2= new String("abc");
String str3= str2.intern();
assertSame(str1==str2);
assertSame(str2==str3);
assertSame(str1==str3);
```

答案为：false false true ，原因：第一个和第二个为false是因为比的是堆和常量池两个不同的对象，第三个为true因为比的是常量池中同一个对象。

 
