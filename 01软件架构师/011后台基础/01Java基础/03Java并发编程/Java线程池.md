Java通过Executors提供四种线程池，分别为：
newCachedThreadPool创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程。
newFixedThreadPool 创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待。
newScheduledThreadPool 创建一个定长线程池，支持定时及周期性任务执行。
newSingleThreadExecutor 创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行。



###newCachedThreadPool

    Exception in thread "main" java.lang.OutOfMemoryError: unable to create new native thread  
        at java.lang.Thread.start0(Native Method)  
        at java.lang.Thread.start(Thread.java:714)  
        at java.util.concurrent.ThreadPoolExecutor.addWorker(ThreadPoolExecutor.java:950)  
        at java.util.concurrent.ThreadPoolExecutor.execute(ThreadPoolExecutor.java:1368)  
        at java.util.concurrent.AbstractExecutorService.submit(AbstractExecutorService.java:112)  
        at com.wenniuwuren.concurrent.newCachedThreadPoolTest.main(newCachedThreadPoolTest.java:15)  
        at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)  
        at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)  
        at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)  
        at java.lang.reflect.Method.invoke(Method.java:497)  
        at com.intellij.rt.execution.application.AppMain.main(AppMain.java:140)  


可以看出来是**堆外内存**溢出，因为我们新建的线程都在工作（代码中用sleep表示在工作中），newCachedThreadPool 只会重用空闲并且可用的线程，所以上述代码只能不停地创建新线程，在 64-bit JDK 1.7 中 -Xss 默认是 1024k，也就是 1M，那就是需要 10000*1M = 10G 的堆外内存空间来给线程使用，但是我的机器总共就 8G 内存，不够创建新的线程，所以就 OOM 了。


总结一下：所以这个 newCachedThreadPool 大家一般不用就是这样的原因，因为它的最大值是在初始化的时候设置为 Integer.MAX_VALUE，一般来说机器都没那么大内存给它不断使用。当然知道可能出问题的点，就可以去重写一个方法限制一下这个最大值，但是出于后期维护原因，一般来说用 newFixedThreadPool 也就足够了。 


HeapByteBuffer与DirectByteBuffer，在原理上，前者可以看出分配的buffer是在heap区域的，其实真正flush到远程的时候会先拷贝得到直接内存，再做下一步操作（考虑细节还会到OS级别的内核区直接内存），其实发送静态文件最快速的方法是通过OS级别的send_file，只会经过OS一个内核拷贝，而不会来回拷贝；在NIO的框架下，很多框架会采用DirectByteBuffer来操作，这样分配的内存不再是在java heap上，而是在C heap上，经过性能测试，可以得到非常快速的网络交互，在大量的网络交互下，一般速度会比HeapByteBuffer要快速好几倍。

直接内存（Direct Memory）并不是虚拟机运行时数据区的一部分，也不是Java虚拟机规范中定义的内存区域，但是这部分内存也被频繁地使用，而且也可能导致OutOfMemoryError 异常出现，所以我们放到这里一起讲解。
在JDK 1.4 中新加入了NIO（New Input/Output）类，引入了一种基于通道（Channel）与缓冲区（Buffer）的I/O 方式，它可以使用Native 函数库直接分配堆外内存，然后通过一个存储在Java 堆里面的DirectByteBuffer 对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在Java 堆和Native 堆中来回复制数据。

import sun.nio.ch.DirectBuffer;

import java.nio.ByteBuffer;

public class Main {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("Hello World!");
        ByteBuffer bb = ByteBuffer.allocateDirect(1024 * 1024 * 128);
        Thread.sleep(10000);
        ((DirectBuffer)bb).cleaner().clean();
        Thread.sleep(10000);
    }
}

可以在任务管理器那观察变化

堆外内存的优点和缺点

堆外内存，其实就是不受JVM控制的内存。相比于堆内内存有几个优势：
　　1 减少了垃圾回收的工作，因为垃圾回收会暂停其他的工作（可能使用多线程或者时间片的方式，根本感觉不到）
　　2 加快了复制的速度。因为堆内在flush到远程时，会先复制到直接内存（非堆内存），然后在发送；而堆外内存相当于省略掉了这个工作。
　　而福之祸所依，自然也有不好的一面：
　　1 堆外内存难以控制，如果内存泄漏，那么很难排查
　　2 堆外内存相对来说，不适合存储很复杂的对象。一般简单的对象或者扁平化的比较适合。

它是一个可以无限扩大的线程池；
它比较适合处理执行时间比较小的任务；corePoolSize为0，maximumPoolSize为无限大，意味着线程数量可以无限大；
keepAliveTime为60S，意味着线程空闲时间超过60S就会被杀死；
采用SynchronousQueue装等待的任务，这个阻塞队列没有存储空间，这意味着只要有请求到来，就必须要找到一条工作线程处理他，如果当前没有空闲的线程，那么就会再创建一条新的线程。







