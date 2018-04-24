hadoop的核心思想是MapReduce，但shuffle又是MapReduce的核心。shuffle的主要工作是从Map结束到Reduce开始之间的过程。首先看下这张图，就能了解shuffle所处的位置。图中的partitions、copy phase、sort phase所代表的就是shuffle的不同阶段。

![](https://images0.cnblogs.com/blog/676214/201409/281109297177557.jpg)



**参考资料**
http://www.cnblogs.com/gwgyk/p/3997849.html

