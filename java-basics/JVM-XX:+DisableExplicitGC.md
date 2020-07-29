每次new DirectByteBuffer()都会执行 `Bits.reserveMemory(size, cap);`方法去申请内存

```java 
    static void reserveMemory(long size, int cap) {

        if (!memoryLimitSet && VM.isBooted()) {
            maxMemory = VM.maxDirectMemory();
            memoryLimitSet = true;
        }

        // 1、 cas方式去申请内存，申请失败则进入队列等待下次尝试（tryReserveMemory机制是通过乐观锁的方式申请内存）
        // 可以通过-XX:MaxDirectMemorySize参数设置 DirectMemory 的最大空间
        if (tryReserveMemory(size, cap)) {
            return;
        }

        final JavaLangRefAccess jlra = SharedSecrets.getJavaLangRefAccess();

        //2、排入队列进行重试
        while (jlra.tryHandlePendingReference()) {
            if (tryReserveMemory(size, cap)) {
                return;
            }
        }

        // 3、如果两次cas申请内存都失败了，这时就会手动 full gc释放内存
        System.gc();

        boolean interrupted = false;
        try {
            long sleepTime = 1;
            int sleeps = 0;
            while (true) {
            
                //4、System.gc();后再去cas申请内存，此时如果再次失败则会 throw new OutOfMemoryError("Direct buffer memory");
                if (tryReserveMemory(size, cap)) {
                    return;
                }
                if (sleeps >= MAX_SLEEPS) {
                    break;
                }
                if (!jlra.tryHandlePendingReference()) {
                    try {
                        Thread.sleep(sleepTime);
                        sleepTime <<= 1;
                        sleeps++;
                    } catch (InterruptedException e) {
                        interrupted = true;
                    }
                }
            }

            throw new OutOfMemoryError("Direct buffer memory");

        } finally {
            if (interrupted) {
                Thread.currentThread().interrupt();
            }
        }
    }  
```
