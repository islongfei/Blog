## 异常堆栈丢失

### 问题描述
前几天发现项目个别账号在页面查询会报错，但是查看日志发现仅仅只有一行NPE，这就很尴尬了，正常情况下抛出异常都会打印异常堆栈信息，但这次没有异常堆栈信息，很难去定位到 NPE 的来源，如图所示：
  
<img src="https://github.com/islongfei/Blog/blob/master/images/FastThrow1%20.png" width="85%" hegiht="85%"  />

### 问题排查
  于是就顺藤摸瓜，先从日志打印的出处：全局异常处理去排查 NPE，
```Java         
        String errorStack = Throwables.getStackTraceAsString(e);
        Object url = request.getRequestURI();
        if (e instanceof BusinessException){
            LOGGER.keyword(("BusinessExceptionHandler url:"+url)).warn("访问服务出现异常，url:{} \n message:{} \n {}", url, errorMsg, errorStack);
        }else {
            LOGGER.keyword(url).error("访问服务出现异常，url:{} \n message:{} \n {}", url, errorMsg, errorStack);
        }
 ```
  发现在全局异常处理的之前，Exception 的 StackTrace 已经为空了，导致日志没有打印堆栈信息。接着继续根据接口的 url 向下定位到相关业务 Controller 代码，由于异常是在调用中台接口之前抛出的，很快定位到了 NPE 在 Controller 层的位置，是在调用获取用户权限接口时所抛出的。由于获取用户权限的接口实现代码调用层级嵌套太深，并且日志没有异常堆栈信息，线下无法重现问题，此时通过日志和代码已经很难判断到底是在哪一行报的空指针了。    
  
  于是使用线上排查利器 arthas ,进行线上 debug ,通过线上问题的不断复现， 根据抛出的异常通过 arthas 去一层一层向下跟踪方法调用链来定位异常来源。最终定位到是引入的权限系统 platform 包的某行代码没有做非空检验，在用户的权限配置有误的场景下，导致 map 为 null，在 get(key) 时抛出了空指针。
 <img src=" https://github.com/islongfei/Blog/blob/master/images/FastThrow2.png" width="85%" hegiht="85%"  /> 






