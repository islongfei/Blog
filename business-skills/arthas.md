### 经常用到的命令  

* 下载  
`curl -O https://alibaba.github.io/arthas/arthas-boot.jar`

*  启动  
`java -jar arthas-boot.jar`

*  查看异常位置  
`watch  -e -x 2 com.pagoda.common.rbac.RbacResources findRbacResources "{params,throwExp}"`

*  查看crid  
`tail -100f prod.x-access.erp_basedata.log | grep 876a19e228c2cb01`

*  查看执行到了哪行代码  
`trace --skipJDKMethod false  com.pagoda.basedata.web.org.BasOrgController selectBasOrgDetailInfoList -n 1`

*  查看变量的值  
`watch com.pagoda.basedata.service.article.BasArticleServiceImpl getFunctionTree "{params,target}" -x 3`

*  在方法调用之前查看入参  
`watch com.pagoda.order.service.goods.BasOrgGoodsServiceImpl dealOrgGoodsInTwoDepot "{params,returnObj}" -x 3 -b`

*  查看方法出参和返回值  
`watch com.pagoda.order.service.goods.BasOrgGoodsServiceImpl dealOrgGoodsInTwoDepot "{params,returnObj}" -x 3`

*  查看方法调用前后参数  
`watch com.pagoda.order.service.goods.BasOrgGoodsServiceImpl dealOrgGoodsInTwoDepot "{params,target,returnObj}" -x 2 -b -s -n 2`

* 一键展示当前最忙的前N个线程并打印堆栈  
`thread -n 3`


* 找出当前阻塞其他线程的线程，发现死锁  
`thread -b`


* 查看当前jvm信息  
`jvm` 

* 查看类加载信息 (输出当前类的详细信息，包括这个类所加载的原始文件来源、类的声明、加载的ClassLoader等详细信息)  
`sc -d com.pagoda.basedata.api.goods.BasGoodsService`

* 查看类的字段信息  
`sc -f com.pagoda.basedata.api.goods.BasGoodsService`

* 反编译指定类,会有classLoader信息  
`jad java.lang.String`

* redefine 动态替换内存中class，热部署，限制条件：只能改方法实现（方法运行完成后才会生效），不能改方法名， 不能改属性  

* [热更新代码一条龙](https://github.com/islongfei/Blog/blob/master/business-skills/arthas02.md)

