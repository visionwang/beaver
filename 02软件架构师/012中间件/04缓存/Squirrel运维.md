

#简介
* 是什么？
概念：基于redis-cluster的**纯内存**的KV存储系统
发展：单节点->主从版->集群版
* 适用场景？
性能要求低，数据量大：Mysql
性能高，数据量小：Squirrel
性能较高，数据量较大：Cellar
* 适用状况
响应时间: 0.7ms
可用性: 5个9
200亿调用->1.3万亿调用
* 单机
value大小对性能影响很大，由于网络的开心，qps很难压得很高
跨机房调用，优化点，选择同节点调用
pipeline方式？是什么意思
10万->6万（单节点的合适值）->1万
单线程模型
* zk同步配置，直连的架构

#技术内幕
不存在中心节点或代理节点
redis-cluster，hash槽16384 hash slot,CRC16(key)%16384
* 集群选择方案
集群总大小：单节点大小*数量
规模选择：1G，2G，5G
每个可以部署多个从库，从库可读


问题
主库内存使用率高于从库
mysql，Binlog同步
redis过去需要全量缓存

6个：
真实数据，DICT数据字典，连接的开销(产生堆积时，会变得很大)
<remark>主从复制缓冲区(128mb,master有)</remark>，AOF缓冲区，showlog记录区(1000记录)

##<remark>大Key的风险</remark>，999线提高？128K->528K？
大key不当操作会堵死redis?
sso问题？
40-50万元素，200ms以上平均效应时间
大key的删除，几个G的大key，业务大量报错，阻塞一定时间failoveer，引起连锁反应，echo?

1.集合类型Hash,List,Set,Zset，打撒到N个key红，保证每个不超过50000？没太明白
2.禁止对元素做全量操作
3.key的清除

大key运营，不要使用Category删除

###路由策略
1.set key value
2.(error)MOVED 15495: 127.0.0.1:7001
3.set key value
本地路由表？client

业务：只读master，主从，读写严格分离
不同机房，不同地域

读策略：routerType
master-only:强一致，分布式锁
master-slave:默认
slave-only:主节点只写，从节点只读

规则的优先级，routeType的master-only规则有限级最高，即使跨地域也只读master

**最佳实践**
1.绝大多数，推荐routeType=master-slave，充分利用从库分担读流量
2.对于高qps集群建议redis的各机房比例和应用比例一致

1.每个分片从节点数量最多不超过3个(30%)，failover会承担所有之和
2.对于写多读少，且平均value大小超过10KB，从库数量不超过1个（常规）或2个（3机房容灾）

##热key风险
热数据，50k，千兆网卡？万兆，qps2000
zcard msg.call_0(大key就是大value?)
1.小容量热key，做本地缓存
2.大容量热key，拆成多个key，流量打散到不同节点
3.热key的写,DBA限流(category)，运营及早发现
4.master-slave，做好降级

#缓存过期和淘汰
evicted_keys（淘汰）
1.noevicetion，业务报错
2.allkey
increase,increaseby(自增)?pv,uv
<remark>缓存穿透，给默认值，避免Key同时过期，满容时，大量key淘汰对性能产生影响，抖动（比如GC垃圾回收的处理，这段时间就不行，其他时候正常）</remark>
a.集合设置合理时间
b.主动更新缓存，不要同时过期大量key，避免缓存穿透和雪崩


expired_keys
a.String类型，管理平台catefory默认过期时间
b.集合类型，不支持细粒度控制


每多一个slave，性能消耗，连接堆积
抖动？专线容易打满，

#Redis持久化
RDB:主从
AOF:binlog

开启AOF的问题
刷盘策略，always,everysec:0-2s,no（操作系统？）:0-30s
aof缓冲区，rewrite,比如加1，1万次，1->100001，合并log

最佳实践
1.持久化和非持久化分离，aof持久化
2.非持久化->三级房容灾
3.大容量Cellar

#高效运维
wiki, squirrel客户机器人
Ca是端到端的处理
自查：
超时，抖动

cat报表->部分vm上->负载过高->full GC->thread block
falcon监控
这边的兄弟们也都在考虑6了，所以这部分其实没那么大的所谓的，哈哈

上海和北京双中心，数据比对，
scan命令？随机性？采用？不太明白，

问题
zk中，category配置，全局唯一，包含集群信息
北京: staging环境
现在：category找到集群名，层次zk的树，需要好好试试，一直木有机会啊。

protobuf方式？
key:category

mset,mget
pipeline?(自由组合命令？)
key的映射

集合类型，rua脚本？
increaseby




