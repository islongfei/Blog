## arthas 线上修改热更新
我们在开发过程中时常有这样的痛点：
* 线上排查问题，找到修复思路了，但应用重启之后环境现场就变了，问题难以复现。  
* 本地开发时，怀疑某个框架有bug，要去修改验证。如果自己编译开源框架再发布，耗时会非常长，还不一定能编译成功。 

面对这样的痛点借助 arthas 的 `jad/mc/redefine` 命令，不用重启就直接可以实现代码热更新。

### 一、修改代码 
* 直接在服务器上修改：可以将要修改的类的.class文件通过 `jad`命令反编译生成.java文件，再用文本编辑器进行修改。
命令如下  `jad --source-only com.xxx.BasGoodsTypeServiceImpl > /xxx/redefine-test/BasGoodsTypeServiceImpl.java`
* 也可以在本地编辑器直接修改代码，这样也更加稳妥。

### 二、编译代码
1. 先使用`sc`命令查找出修改类所对应的classLoader，命令如下：  
``` sc -d *BasGoodsTypeServiceImpl | grep classLoaderHash  
 classLoaderHash   6f2b958e
 ```

