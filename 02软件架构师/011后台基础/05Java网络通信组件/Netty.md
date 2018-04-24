对NIO的封装


Pigeon底层也是用的nio


[攻破JAVA NIO技术壁垒](http://blog.csdn.net/u013256816/article/details/51457215#comments)


debug模式下，任务执行完后，显示的效果如下：

也就是说，任务已经执行完毕了，但是线程池中的线程并没有被回收。但是我在ThreadPoolExecutor的参数里面设置了超时时间的，好像没起作用，原因如下：
工作线程回收需要满足三个条件：
1)  参数allowCoreThreadTimeOut为true

2)  该线程在keepAliveTime时间内获取不到任务，即空闲这么长时间

3)  当前线程池大小 > 核心线程池大小corePoolSize。



[Netty实现原理浅析](http://www.importnew.com/15656.html)






