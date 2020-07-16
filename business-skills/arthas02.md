## arthas 线上修改代码热更新
在开发过程中时常有这样的痛点：
* 线上排查问题，找到修复思路了，但应用重启之后环境现场就变了，问题难以复现。  
* 本地开发时，怀疑某个框架有bug，要去修改验证。如果自己编译开源框架再发布，耗时会非常长，还不一定能编译成功。 

**面对这样的痛点借助 arthas 的 `jad/mc/redefine` 命令，不用重启就直接可以实现代码热更新，非常方便。**

### 一、修改代码 
* 直接在服务器上修改：可以将要修改的类的.class文件通过 `jad`命令反编译生成.java文件，再用文本编辑器进行修改。  
命令如下：  
`jad --source-only com.xxx.BasGoodsTypeServiceImpl > /xxx/redefine-test/BasGoodsTypeServiceImpl.java`
* 也可以在本地 IEDA 直接修改代码，这样也更加稳妥。  

比如现在要修改 BasGoodsTypeServiceImpl 这个类的代码，在代码中加一行日志如下：  
```java 
LOGGER.keyword("findGoodsTypeOnMain").info("arthas test success");
``` 
修改完将.java文件上传至服务器，用于之后的编译和热替换。也可将本地 IEDA target 下编译好的.class文件上传服务器。

### 二、编译代码
1. 先使用`sc`命令查找出修改类所对应的 classLoader 的 hash 值，命令与结果如下：  
```
sc -d *BasGoodsTypeServiceImpl | grep classLoaderHash  
classLoaderHash   6f2b958e  
 ```  
2.使用`mc`命令 指定 classloader 去内存编译代码，将修改后的 BasGoodsTypeServiceImpl.java 编译为 BasGoodsTypeServiceImpl.class ，  
命令如下：  
```
mc -c 6f2b958e /home/work/spring-boot/arthas-output/redefine-test/BasGoodsTypeServiceImpl.java -d /home/work/spring-boot/starter  

```  

### 三、热更新代码  
使用`redefine`命令重新加载新编译好的 BasGoodsTypeServiceImpl.class，命令如下：  
```
redefine /home/work/spring-boot/starter/com/pagoda/basedata/service/goods/BasGoodsTypeServiceImpl.class
```
可以看到已经更新成功了。  
![image](https://github.com/islongfei/Blog/blob/master/images/arthas01.png)     

由于项目用的的日志系统为 elk ,在调用对应接口后，直接在 kibina 上查找日志，看加上的日志是否热加载成功。如图，日志成功打印出来，在没有重启服务的情况下实现代码修改热加载。  
![image](https://github.com/islongfei/Blog/blob/master/images/arthas02.png)   

### 四、注意 
* `redefine`热更新的类中不允许新增加 field/method。另外正在跑的函数如果没有退出，热更新是不会生效的。
* Arthas 里 `jad/mc/redefine` 一条龙来线上热更新代码，非常强大，但也很危险，需要做好权限管理，操作不规范，同事两行泪 :sob: 。

