过去对GC一直都只是浮于字面的理解，这次好好的俊一波实践。


-XX:+PrintGC 输出GC日志
-XX:+PrintGCDetails 输出GC的详细日志
-XX:+PrintGCTimeStamps 输出GC的时间戳（以基准时间的形式）
-XX:+PrintGCDateStamps 输出GC的时间戳（以日期的形式，如 2013-05-04T21:53:59.234+0800）
-XX:+PrintHeapAtGC 在进行GC的前后打印出堆的信息
-Xloggc:../logs/gc.log 日志文件的输出路径



我们使用Jetty容器来部署应用，并开启Servlet3.0的异步特性，

ps -ef | grep java




mat, 推荐的dump分析工具，推工具ß


http://ifeve.com/category/guava-2/

http://ifeve.com/google-guava/




