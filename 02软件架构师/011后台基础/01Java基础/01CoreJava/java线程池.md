debug模式下，任务执行完后，显示的效果如下：

也就是说，任务已经执行完毕了，但是线程池中的线程并没有被回收。但是我在ThreadPoolExecutor的参数里面设置了超时时间的，好像没起作用，原因如下：
工作线程回收需要满足三个条件：
1)  参数allowCoreThreadTimeOut为true

2)  该线程在keepAliveTime时间内获取不到任务，即空闲这么长时间

3)  当前线程池大小 > 核心线程池大小corePoolSize。





       BlockingQueue queue = new LinkedBlockingQueue();
          ThreadPoolExecutor executor = new ThreadPoolExecutor(3, 6, 1, TimeUnit.SECONDS, queue);  
          executor.allowCoreThreadTimeOut(true);


http://www.importnew.com/23984.html



4.在线程池中，有核心线程，对于核心线程超时也回收，所以，需要执行下边这个方法，确保核心线程超时之后也被回收。

　　　解决办法：threadPoolExecutor.allowCoreThreadTimeOut(true);



<script>alert("xxx");</script>


