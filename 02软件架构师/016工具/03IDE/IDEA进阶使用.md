
安装mavenhelper插件，查看maven依赖关系
**手工安装方式**
http://plugins.jetbrains.com/plugin/7179-maven-helper
选择从disk安装即可



1905年一群以亨利·马蒂斯为首的年轻画家在巴黎秋季沙龙展出自己的形象简单，色彩鲜艳大胆的作品，震惊了画坛，人们惊呼“这简直是野兽！”从此画坛上出现了一个新的野兽派。



排除编译，引入编译

idea选择主题
https://www.jianshu.com/p/7ca5ee761c97
主题下载站
http://color-themes.com/?view=index


打开IDEA后，点击『Help』，选择『Edit Custom VM Options』，填入以下参数配置：
# custom IntelliJ IDEA VM options

##################JVM模式############################
# IDEA的JVM以Server模式启动（新生代默认使用ParNew）
-server
 
##################内存分配############################
# 堆初始值占用3G，意味着IDEA启动即分配3G内存
-Xms3g
# 堆最大值占用3G# 堆初始值占用3G，意味着IDEA启动即分配3G内存
-Xms3g
# 堆最大值占用3G
-Xmx3g
# 强制JVM在启动时申请到足够的堆内存（否则IDEA启动时堆初始大小不足3g）
-XX:+AlwaysPreTouch
# 年轻代与老年代比例为1:3（默认值是1:4），降低年轻代的回收频率
-XX:NewRatio=3
# 栈帧大小为16m
-Xss16m# 堆初始值占用3G，意味着IDEA启动即分配3G内存
-Xms3g
# 堆最大值占用3G# 堆初始值占用3G，意味着IDEA启动即分配3G内存
-Xms3g
# 堆最大值占用3G
-Xmx3g
# 强制JVM在启动时申请到足够的堆内存（否则IDEA启动时堆初始大小不足3g）
-XX:+AlwaysPreTouch
# 年轻代与老年代比例为1:3（默认值是1:4），降低年轻代的回收频率
-XX:NewRatio=3
# 栈帧大小为16m
-Xss16m
  
##################老年代回收器############################
# 使用CMS老年代回收器
-XX:+UseConcMarkSweepGC
# CMS的重新标记步骤：多线程一起执行
-XX:+CMSParallelRemarkEnabled
# CMS的并发标记步骤：启用4个线程并发标记（理论上越多越好，前提是CPU核心足够多）
-XX:ConcGCThreads=4
 
##################JIT编译器############################
# 代码缓存，用于存放Just In Time编译后的本地代码，如果塞满，JVM将只解释执行，不再编译native代码。
-XX:ReservedCodeCacheSize=512m
# 分层编译，JIT编译优化越来越好，IDEA运行时间越久越快
-XX:+TieredCompilation
# 节省64位指针占用的空间，代价是JVM额外开销
-XX:+UseCompressedOops
# 增大软引用在JVM中的存活时长（堆空闲空间越大越久）
-XX:SoftRefLRUPolicyMSPerMB=50
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djsse.enableSNIExtension=false


[IDEA中如果一个类被排除编译之后如何恢复编译](https://jingyan.baidu.com/article/9989c746eb4f15f648ecfe9d.html)
