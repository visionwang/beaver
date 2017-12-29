在入职新公司后，发现BU有一套非常棒的RD培养体系，自己也果断学习一波，为未来做好准备。在最初的一个月（校招3个月），需要在Confluence中记录自己的学习情况，还需要每天将学习情况以邮件形式发送小组并抄送leader，学习期结束还需要做一个技术和业务学习的串讲，整体感觉非常规范而有体系。

# 基础篇
###开发工具
**IDE**  要求精通，[Intellij IDEA 使用教程](http://wiki.jikexueyuan.com/project/intellij-idea-tutorial/)
* 熟练使用IDEA进行开发
* 熟练使用快捷键
* 会本地和远程DEBUG
**Git**  要求精通，[Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)
* 理解git分布式
* 理解git暂存区、对象
* 熟练使用push, fetch, rebase, reset等命令
* 会使用一种git可视化工具，如sourceTree, TortoiseGit
**Maven**  要求精通，[Maven教程](http://www.yiibai.com/maven/)
* 熟练配置及使用
* 理解Maven依赖机制，会解决jar冲突问题
* 掌握mvn clean, compile, test, install, deploy等命令
###CS基础
**Linux操作系统**  要求掌握，[Linux入门基础](http://study.163.com/course/introduction/232007.htm#/courseDetail)
* 中断/异常，地址空间，系统调用，进程调度，文件系统
* 系统管理：文件，权限，vim使用，shell脚本，启动流程，crontab, syslog等。
* 常见命令：less, grep, cut, sed; top, netstat, tcpdump, vmstat, iostat等 
**数据结构与算法**  要求精通，[大话数据结构](http://product.dangdang.com/21088369.html)
* 链表，散列表，栈，队列，二叉树，红黑树，B树的原理
* 常见排序算法实现及其时间&空间复杂度计算
**计算机网络**  要求掌握，[计算机网络](http://product.dangdang.com/24165174.html)，[图解HTTP](http://product.dangdang.com/23462067.html)
* TCP/IP基础：IP, 子网, 路由, TCP
* HTTP, HTTPs协议基本原理
* 熟练掌握cookie, session原理及跨域等问题的由来和处理方法。
* 了解前端页面加载过程及常用网络优化技术
* 熟悉抓包工具Fiddler或Charles
**数据库**  要求精通，[MySQL技术内幕](http://product.dangdang.com/23727515.html), [高性能MySQL](http://product.dangdang.com/23214590.html)
* 了解Mysql
* 基本操作：Sql，数据导入/导出，备份/恢复
* 常用MySQL存储引擎及索引方式
* 查看sql执行计划，掌握常用SQL调优方法
* 功能基本原理：复制、文件、日志、事务和锁
* Mongo的原理和使用
###Java项目
**Java基础**  要求精通

| 内容 | 参考资料 |
|:--|:--|
| 基本语法: 集成、异常、引用、泛型等 | [Java核心技术 卷1](http://product.dangdang.com/24035306.html), [Java编程思想](http://product.dangdang.com/9317290.html) |
| 类库: 常见类、集合、序列化 | JDK源码：String, Integer, Long, Enum, BigDecimal,ArrayList, LinkedList, HashMap, LinkedHashMap, TreeMap, ConcurrentHashMap, HashSet, LinkedHashSet, TreeSet, AtomicInteger, ThreadLocal |
|  IO(文件IO, 网络IO), NIO| [Java网络编程](http://product.dangdang.com/23560594.html) |
| 并发编程 | [Java并发编程实战](http://product.dangdang.com/22606835.html) |
| JVM: Class Code, 反射, ClassLoader, GC; 常用java监控命令: jstack, jmap, jstat; 常用java启动参数: gc相关, 常用环境变量 | [深入理解Java虚拟机](http://product.dangdang.com/23259731.html) |
| Java安全 |  |
| 反射、动态代理、jmx、jms |  |
**JavaWeb** 要求掌握
* Servlet/JSP [Servlet官方文档](http://download.oracle.com/otndocs/jcp/servlet-3.0-fr-oth-JSpec/)
* Struts/Spring MVC
* Spring  [Spring技术内幕](http://product.dangdang.com/22606836.html)
* JDBC, c3p0, MyBatis [Mybatis官方文档](http://www.mybatis.org/mybatis-3/zh/)
* Jetty
* UnitTest
* HttpClient
**第三方类库** 要求掌握
* apache.commons3
* fastjson, jackson, gson
* guava  [Guava官方文档](https://github.com/google/guava/wiki)
**系统部署** 要求了解
* 负载均衡： F5, SLB, LVS, Nginx
* 反向代理： Nginx, Tengine(阿里)
* Web容器: Tomcat, Jetty
* 容灾: 数据库冷/热备，异地多活，服务降级
###工程素质
**APIj接口设计**  要求掌握
* Restful/MVC
**编码**  要求掌握  [大话设计模式](http://product.dangdang.com/20079096.html), [重构 改善既有代码的设计](http://product.dangdang.com/23734636.html), [EffectiveJava中文版](http://product.dangdang.com/20459091.html)
* OO，业务建模，如何把产品需求转化为合理的软件模型
**开发规范**  要求掌握
* 编码规范 [Alibaba规范], [GoogleJava编码规则](http://www.hawstein.com/posts/google-java-style.html)
* 数据库设计规范
* 日志规范
**安全规范**  要求掌握
* 遵循XSS、CSRF、SQL注入等常见安全问题的相关规范
**测试**  要求掌握
* 基本概念：黑盒/白盒、冒烟、回归、单元测试、集成测试、性能测试

# 进阶篇
###SOA架构
**RPC** 要求了解  
* 了解RPC框架的基本原理和使用
* 了解RPC框架的核心模块：服务注册与发现，序列化，网络通信，服务治理
**MQ**  要求了解
* 消息队列原理和模型
* activemq, rabbitmq, kafka
* RocketMQ(阿里)
**DAL**  要求了解
**缓存**  要求掌握
* 了解本地缓存ehcache和guava的区别
* 了解本地缓存和分布式缓存的区别和使用场景
* 了解memcache和redis区别
* 了解redis基本原理和操作
**协调ZooKeeper**  要求了解
* 理解分布式一致性原理
* Paxos算法
* ZK原理与使用
**搜索引擎**  要求了解
* lucene, solr, ES

###成为应用Owner
重点关注沟通能力(态度、理解、表达)和工程能力，详情如下。
**实现**
* 理解原始需求：判断可行性和代价
* 业务建模：完整准确的将业务需求转化为领域模型
* 自身管理： 多分支开发、进度控制、代码/资源/文档的输出。
**质量**
**运维**

# 高级篇










