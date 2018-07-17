



#Redis 优势
1. 性能极高：Redis能读的速度是110000次/s,写的速度是81000次/s 。(单机性能)
2. 丰富的数据类型：Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
3. 原子：Redis的所有操作都是原子性的，意思就是要么成功执行要么失败完全不执行。单个操作是原子性的。多个操作也支持事务，即原子性，通过MULTI和EXEC指令包起来。
4. 丰富的特性：Redis还支持 publish/subscribe, 通知, key 过期等等特性。

#Redis Key
1	DEL key
该命令用于在 key 存在时删除 key。
2	DUMP key 
序列化给定 key ，并返回被序列化的值。
3	EXISTS key 
检查给定 key 是否存在。
4	EXPIRE key seconds
为给定 key 设置过期时间。
5	EXPIREAT key timestamp 
EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 EXPIREAT 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。
6	PEXPIRE key milliseconds 
设置 key 的过期时间以毫秒计。
7	PEXPIREAT key milliseconds-timestamp 
设置 key 过期时间的时间戳(unix timestamp) 以毫秒计
8	KEYS pattern 
查找所有符合给定模式( pattern)的 key 。
9	MOVE key db 
将当前数据库的 key 移动到给定的数据库 db 当中。
10	PERSIST key 
移除 key 的过期时间，key 将持久保持。
11	PTTL key 
以毫秒为单位返回 key 的剩余的过期时间。
12	TTL key 
以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。
13	RANDOMKEY 
从当前数据库中随机返回一个 key 。
14	RENAME key newkey 
修改 key 的名称
15	RENAMENX key newkey 
仅当 newkey 不存在时，将 key 改名为 newkey 。
16	TYPE key 
返回 key 所储存的值的类型。


#Redis 发布订阅
Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。
Redis 客户端可以订阅任意数量的频道。
下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

#Redis 事务(类似数据库的batch而不是原子性事务)
Redis 事务可以一次执行多个命令， 并且带有以下两个重要的保证：
批量操作在发送 EXEC 命令前被放入队列缓存。
收到 EXEC 命令后进入事务执行，事务中任意命令执行失败，其余的命令依然被执行。
在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。
一个事务从开始到执行会经历以下三个阶段：
开始事务。
命令入队。
执行事务。
**单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的。**
事务可以理解为一个打包的批量执行脚本，但批量指令并非原子化的操作，中间某条指令的失败不会导致前面已做指令的回滚，也不会造成后续的指令不做。
#Redis 管道技术
从处于局域网中的Mac OS X系统上执行上面这个简单脚本的数据表明，开启了管道操作后，往返时延已经被改善得相当低了。

保存过期时间

redisDB结构的expires字典保存了数据库中所有键的过期时间，我们称这个字典为过期字典:
过期字典的键是一个指针，这个指针指向键空间中的某个键对象( 也即是某个数据库键)。
过期字典的值是一个long long 类型的整数，这个整数保存了键所指向的数据库键的过期时间:一个毫秒精度的UNIX 时间戳。
不会被删除，只是从内存移到磁盘上。
redis不止是内存数据库
不要被误导了，如果实际内存超过你设置的最大内存，就会使用LRU删除机制。
楼主，我前面的回答感觉不妥，查了下文档资料。现在重新回答
Redis无论有没有设置expire，他都会遵循redis的配置好的删除机制，在配置文件里设置：
redis最大内存不足"时,数据清除策略,默认为"volatile-lru"。
volatile-lru  ->对"过期集合"中的数据采取LRU(近期最少使用)算法.如果对key使用"expire"指令指定了过期时间,那么此key将会被添加到"过期集合"中。将已经过期/LRU的数据优先移除.如果"过期集合"中全部移除仍不能满足内存需求,将OOM.
allkeys-lru ->对所有的数据,采用LRU算法
volatile-random ->对"过期集合"中的数据采取"随即选取"算法,并移除选中的K-V,直到"内存足够"为止. 如果如果"过期集合"中全部移除全部移除仍不能满足,将OOM
allkeys-random ->对所有的数据,采取"随机选取"算法,并移除选中的K-V,直到"内存足够"为止
volatile-ttl ->对"过期集合"中的数据采取TTL算法(最小存活时间),移除即将过期的数据.
noeviction ->不做任何干扰操作,直接返回OOM异常
另外，如果数据的过期不会对"应用系统"带来异常,且系统中write操作比较密集,建议采取"allkeys-lru"。
由以上可以看出，对没设置expire的数据，产生影响的是allkeys-lru机制，allkeys-random。
所以redis没设置expire的数据是否会删除，是由你自己选择的删除机制决定的。

#本机环境
cd /usr/local/Cellar/redis/4.0.9/bin

###参考资料
[Redis入门](http://www.runoob.com/redis/redis-intro.html)
[MAC环境安装Redis](https://www.cnblogs.com/maoxiaolv/p/5765640.html)
[SDS 与 C 字符串的区别](http://redisbook.com/preview/sds/different_between_sds_and_c_string.html)