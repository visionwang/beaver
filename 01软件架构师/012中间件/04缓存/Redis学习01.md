


特点：基于内存的K-V存储，C语言编写
使用场景：数据库、缓存和消息中间件，Session


#数据结构
3.0->4.0
简单动态字符串
链表

#1.简单动态字符串
SDS  
数据结构：len, free, char buf[]
性能更优秀，二级制安全（C字符串，二进制可能存在'\0'）

#2.字典
rehash的过程比较有意思？
typedef struct dict{
    dictht ht[2];
    trehashidx;//默认-1
}

typedef struct dictht{
    dictEntry **table;
}

使用hashtable[0]

8,比如9个，达到阈值

rehash 阻塞式rehash
渐进件式rehash

惰性rehash，有意思

[0]->[1]
空间换时间，很棒的选择
对别java的rehash

**jdk中的rehash**

#进阶
**两种encoding形式：？**，sds实现，还有一种1-9999，string池，其他实现方式？
PTR
REFCOUNT：引用计数器
LRU：最后触达时间，淘汰策略，计算出空转时间

通用命令：DEL,EXPIRE,TYPE
字符串命令：SET,GET,APPEND,STRLEN

5大结构，string, set, list, hash, sortedset
Rpush, useerid, spuid,

redisServer->16个RedisDB,
键空间字典  key, value
**过期字典  key, expire**
key就是简单动态字符串

过期策略：
* 定时删除，创建定时器

* **惰性删除**，用的时候判断是否过期，如果过期删除
* 定期删除，每个一段时间，对数据进行检查，删除多少也根据实际情况

**被动和主动**
删除也是一个消耗资源的操作

get目录，递归？ range...

3个问题了

JVM操作，为什么？

满容淘汰策略  所需内存超过MAX
volatile-lru
allkeys-lru
xxx-random
xxx-ttl
打满，**随机 删除？**

淘汰，轮训，随机

计算最短路径


VM:冷热key的分离，很棒嘛。。。
满容，扩容

#持久化
RDB：文件，SAVE,BGSAVE(fork一个子进程，)
二进制格式

AOF：binlog
文本格式，记录所有写命令日志（增量命令日志）
-append>aof缓冲区-write>操作系统缓存区-sync>aof文件

策略：no(20-30秒), everysec(时间戳，写入数据时判断), alway（每一条写很帅）
磁盘IO，刷盘刷不上去？抛出异常
fsync, fdatasync,同步调用？

主进程 lua脚本，io传输

大对象，等等会说

AOF重写有一个区域，文件越来越大，恢复时间越长，根据数据
1.启动AOF重写缓冲区
2.fork子进程，后台生成AOF文件
3.子进程释放，通知文件生成
4.主进程调用信号处理程序，将AOF重写缓冲区写入新的AOF

写命令多了？机器内存爆了，

#问题
squirrel

25G  超过80%

Keys:17121906
QPS:22062

KV,Category管理
内存，CPU，磁盘，网络

CopyOnWrite?

  AOF文件 10G（策略）
  AOF缓冲区 1M
  内存中数据  25G   （各个缓冲区的大小）
  新AOF缓存区  10G
  新AOF文件 25G

大小比例

二进制
文本

#事件驱动（如何驱动？）
事件调度：阻塞socket
文件事件/时间事件

###文件时间
套接字
IO多路复用
事件分派
事件处理

###时间事件
ServerCron


可以考虑好好整整

#分布式
主从一致，RDB+写命令
全部同步，部分同步，offset字段，断点续传
需要通信，这个Offset，用在db,和内存

先进先出
复制缓冲去

哨兵：完成故障转移
特殊状态下的redis服务器，这个哨兵如何做的

1.主从
* 主观下线
当前哨兵和主服务断线一段时间
* 客观下线
超过足够，主管下线
* 选举领头哨兵，
Raft,选举算法   （先来先得，字段，一个字段被设置就不能被设置了）
* 故障1转移,
主从切换，他来做 

2.集群
slot，没有使用一致性hash，

crc16(key)&16383

每个节点负责任意的槽

重新分片，集群


1.扩容后，渐进式rehash  A->B（rehash） 指定，重新算？等等去想想
2.memorycache，一致性hash
3.


5节点， 

1主1从
5节点 * 5


pipeline(网络IO)，增加QPS
QPS：180万上限，OPS

QPS：180万上限

qps，网络IO，redis命令
hash,

QQ阅读，微信阅读，小说阅读

弱，cas?
原子操作，生产


