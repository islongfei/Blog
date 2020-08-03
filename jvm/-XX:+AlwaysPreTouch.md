官网文档说明：  
>Enables touching of every page on the Java heap during JVM initialization. This gets all pages into the memory before entering the main() method. 
The option can be used in testing to simulate a long-running system with all virtualmemory mapped to physical memory.
By default, this option is disabled and all pages are committed as JVM heap space fills.   

[参考文章](https://hllvm-group.iteye.com/group/topic/28839#post-209144)    
[csdn](https://blog.csdn.net/ma_ru_long/article/details/106687512)  

-Xms4096m -Xmx4096m 我的一个Java应用里这这样配置的，按一般资料里都说堆最大值和最小值设为一样是为了避免多次内存分配的开销，
但在我实际的应用中应用刚起来时 没占用了4096m内存？，而是随着系统运行逐渐增长到4g的，这是为什么呢？
