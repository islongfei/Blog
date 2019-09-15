### 为什么HashMap不用LinkedList,而选用数组?  
```Java
Entry[] table = new Entry[capacity];//数组  
List<Entry> table = new LinkedList<Entry>();//LinkedList
```  
因为用数组效率最高！
在HashMap中，定位桶的位置是利用元素的key的哈希值对数组长度取模得到。此时，我们已得到桶的位置。显然数组的查找效率比LinkedList大。

### 那ArrayList，底层也是数组，查找也快啊，为啥不用ArrayList?  
因为采用基本数组结构，扩容机制可以自己定义，HashMap中数组扩容刚好是2的次幂，在做取模运算的效率高。
而ArrayList的扩容机制是1.5倍扩容。
