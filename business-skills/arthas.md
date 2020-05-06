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
