![](http://i.imgur.com/jE6MDGi.png)


Credis简介

Credis是一个key-value存储系统，和Memcached类似，但它支持存储的value类型相对更多，包括string(字符串)、list(链表)、set(集合)和zset(有序集合)。
这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，并且这些操作都是原子性的。在此基础上，redis还支持各种不同方式的排序。
 与memcached一样，为了保证效率，数据都是缓存在内存中。不同的是redis可以周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并在此基础上实现了master-slave(主从)同步。
 
 目标：
l  发布一个高可用性的版本的Client Library
l  优化Redis Server的使用
l  提供一套用户友好的UI用于管理和监控Redis集群以及Redis Agent的配置信息
l  提供完善的文档和技术支持


![](http://i.imgur.com/1uwt810.png)

l  应用程序发起请求Redis客户端发起请求获取对应的cluster
l  获取流程：通过ConfigLoad发起web request，通过Json序列化后得到RedisCluster，以后的操作都由Cluster来执行。


具体场景设计：
用户使用类库调用工厂得到RedisCacheProvider，在使用数据操作之前要进行初始化以获取集群信息
![](http://i.imgur.com/AFSgLNP.png)


集群Sharding：
通过一致性哈希算法，将用户请求的key定位到不同的server
[一致性HASH算法详解](http://www.jianshu.com/p/e8fb89bb3a61)


这个真心很帅气
[对一致性Hash算法，Java代码实现的深入研究](http://www.cnblogs.com/xrq730/p/5186728.html)


通过哈希一致性算法自动方式添加节点， 该算法将造成一定程度的数据命中率下降，因此该方案要求：应用程序有数据补偿机制，应用程序能忍受一定程度的命中率下降，必须要估计穿透问题
 
Redis数据同步：
 
采用Redis本身自带的同步机制，同步的时候是Master向Slave定期进行同步，同步的时间间隔可配置，目前支持RDB的同步模式，必须要考虑到数据不一致的情况，应用程序需要忍受一定的数据不一致性。
另外，在读取slave中数据的时候如果读取当前slave失败，则会访问另一个slave，当所有的slave都读取失败才报错。
 

Redis配置管理系统：
 
管理功能：
l  管理Redis Cluster
l  管理Redis Group
l  管理Redis Server
l  管理Agent的基本配置
 
测试工具
l  查看各个Redis Server状态和信息
l  查看各个Agent状态和信息
l  Get/Set/Delete Redis Server状态
 
接口配置：
l  客户端读取配置
 