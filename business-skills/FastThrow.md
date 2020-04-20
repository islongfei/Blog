## 异常堆栈丢失

### 问题描述
前几天发现线上个别账号在页面查询会报错，重试多次后，每次日志的异常信息都仅仅只有一行NPE，这就很尴尬了，日志没有打印异常堆栈信息，很难去定位到 NPE 的来源。正常情况下当异常发生时，JVM会回溯调用栈，构建异常完整的堆栈信息，日志如图所示：
  
<img src="https://github.com/islongfei/Blog/blob/master/images/FastThrow1%20.png" width="85%" hegiht="85%"  />

### 问题排查
于是就顺藤摸瓜，先从最上游的日志打印的出处：全局异常处理去排查 NPE，
```Java         
        String errorStack = Throwables.getStackTraceAsString(e);
        Object url = request.getRequestURI();
        if (e instanceof BusinessException){
            LOGGER.keyword(("BusinessExceptionHandler url:"+url)).warn("访问服务出现异常，url:{} \n message:{} \n {}", url, errorMsg, errorStack);
        }else {
            LOGGER.keyword(url).error("访问服务出现异常，url:{} \n message:{} \n {}", url, errorMsg, errorStack);
        }
 ```
发现在全局异常处理的之前，Exception 的 StackTrace 已经为空了，导致日志没有打印堆栈信息。接着继续根据接口的 url 向下定位到相关业务 Controller 代码，由于异常是在调用中台接口之前抛出的，很快定位到了 NPE 在 Controller 层的位置，是在调用获取用户权限接口时所抛出的。由于获取用户权限的接口实现代码调用层级嵌套太深，并且日志没有异常堆栈信息，本地无法重现问题，此时通过日志和代码已经很难判断是在哪一行报的空指针了。    
  
于是使用线上排查利器 arthas ，进行线上 debug ，通过线上问题的复现，用 arthas 一层一层向下跟踪方法调用链来定位异常来源。最终定位到是引入的权限系统 platform 包的某行代码没有做非空检验，在用户的权限配置有误的场景下，导致 map 为 null，在 get(key) 时抛出了空指针。  
    
 <img src="https://github.com/islongfei/Blog/blob/master/images/FastThrow2.png" width="85%" hegiht="85%"  /> 
 
 ### 堆栈丢失分析
 空指针的问题已经定位到了，但异常的堆栈为什么会丢失呢？起初猜测是引用的 platform 包对异常做了特殊处理，导致没有了堆栈，经排查验证是没有做特殊处理的。
  并且发现了该 NPE 在之前某段时间的日志上又是有异常堆栈信息的。  
  
 真正的原因是 JVM 为了优化性能，如果检测到在代码里某个位置连续多次抛出同一类型异常的话，比如 NPE，JVM 会重新编译该方法（JIT编译器），用 Fast Throw  方式抛出事先创建好的异常（没有堆栈信息）。这种异常抛出速度非常快，因为不需要在堆里分配内存，也不需要构造完整的异常栈信息。这也与之前的 NPE 有异常堆栈但之后却没有了的现象吻合。  
 
 Fast Throw 机制在 jdk1.5 的相关发布说明有相关描述，通过 `-XX:-OmitStackTraceInFastThrow` 参数可以关闭 Fast Throw 。 

`The compiler in the server VM now provides correct stack backtraces for all "cold" built-in exceptions. For performance purposes,   when such an exception is thrown a few times, the method may be recompiled. After recompilation, the compiler may choose a faster tactic using preallocated exceptions that do not provide a stack trace. To disable completely the use of preallocated exceptions, use this new flag: -XX:-OmitStackTraceInFastThrow.`

通过查看 hotspot JVM 源码，全局搜索找到了 `OmitStackTraceInFastThrow` 相关处理的代码：  
```C++
  // If this throw happens frequently, an uncommon trap might cause
  // a performance pothole.  If there is a local exception handler,
  // and if this particular bytecode appears to be deoptimizing often,
  // let us handle the throw inline, with a preconstructed instance.
  // Note:   If the deopt count has blown up, the uncommon trap
  // runtime is going to flush this nmethod, not matter what.
  if (treat_throw_as_hot
      && (!StackTraceInThrowable || OmitStackTraceInFastThrow)) {
    // If the throw is local, we use a pre-existing instance and
    // punt on the backtrace.  This would lead to a missing backtrace
    // (a repeat of 4292742) if the backtrace object is ever asked
    // for its backtrace.
    // Fixing this remaining case of 4292742 requires some flavor of
    // escape analysis.  Leave that for the future.
    ciInstance* ex_obj = NULL;
    switch (reason) {
    case Deoptimization::Reason_null_check:
      ex_obj = env()->NullPointerException_instance();
      break;
    case Deoptimization::Reason_div0_check:
      ex_obj = env()->ArithmeticException_instance();
      break;
    case Deoptimization::Reason_range_check:
      ex_obj = env()->ArrayIndexOutOfBoundsException_instance();
      break;
    case Deoptimization::Reason_class_check:
      if (java_bc() == Bytecodes::_aastore) {
        ex_obj = env()->ArrayStoreException_instance();
      } else {
        ex_obj = env()->ClassCastException_instance();
      }
      break;
    }
```
阅读代码可以得知 JVM 对以下几种常见的异常做了 Fast Throw 优化，
* NullPointerException  
* ArithmeticException
* ArrayIndexOutOfBoundsException
* ArrayStoreException
* ClassCastException  

满足两个条件就会进行 Fast Throw 优化 ：
1. 检测出同一地方频繁抛出异常
2. OmitStackTraceInFastThrow 为 true，或 StackTraceInThrowable 为 false  

JVM 是默认开启了Fast Throw 的， OmitStackTraceInFastThrow 和 StackTraceInThrowable 都默认为true ，如果要关闭 Fast Throw ，加上 `-XX:-OmitStackTraceInFastThrow` 参数即可。

### 验证
* **本地验证**  
本地用多线程模拟在同一处代码频繁抛出 NPE ，看是否会出现 JVM Fast Throw 丢失异常的场景。
代码如下：
```Java 
package com.longfei.test;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class FastThrowMain {
    public static void main(String[] args) throws InterruptedException {
        NpeThread npeThread = new NpeThread();
        ExecutorService executorService = Executors.newFixedThreadPool(4);
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            executorService.execute(npeThread);
            Thread.sleep(2);
        }
    }
}

package com.longfei.test;

public class NpeThread extends Thread {
    private static int count = 0;
    @Override
    public void run() {
        try {
            System.out.println(this.getClass().getSimpleName() + "_" + (++count));
            //测试 NullPointerException
            String str = null;
            System.out.println(str.length());

//            //测试 ArrayIndexOutOfBoundsException
//            int[] array = new int[1];
//            System.out.println(array[2]);

//            //测试 ArithmeticException
//            System.out.println(1 / 0);

        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
}
```  

测试结果如下图，在 NPE 抛出了6000多次后，发生了 Fast Throw ，异常堆栈丢失。  

<img src="https://github.com/islongfei/Blog/blob/master/images/FastThrow3.png" width="85%" hegiht="85%"  /> 

本地加上 `-XX:-OmitStackTraceInFastThrow`之后测试结果， NPE 抛出了 2w 多次后， 没有发生Fast Throw，异常堆栈一直都在，说明参数生效。  

<img src="https://github.com/islongfei/Blog/blob/master/images/FastThrow4.png" width="85%" hegiht="85%"  />   

* **uat环境验证**  
uat环境加上`-XX:-OmitStackTraceInFastThrow`配置之后，用之前权限配置有误的账户请求同一报错接口，在压测了3w多次请求后，没有再次出现 Fast Throw 异常堆栈丢失，测试结果如下图所示：  

<img src="https://github.com/islongfei/Blog/blob/master/images/FastThrow5.png" width="85%" hegiht="85%"  /> 

### 总结
虽然 JVM 的 Fast Throw 机制会对性能有所优化，但在实际的大部分场景下，如果为了追求性能的极致而忽略异常堆栈信息的打印，这其实是得不偿失的，很多时候用异常堆栈信息来定位问题是十分有必要的。  
**建议在项目中加上`-XX:-OmitStackTraceInFastThrow` JVM 参数，关闭 Fast Throw 防止异常堆栈丢失，难以排查问题** 。













