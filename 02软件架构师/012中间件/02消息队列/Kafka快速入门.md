Kafka算是近几年比较热的一个高吞吐量的分布式发布订阅消息系统，本文通过它作为示例，将对整个消息系统进行一个简要的分析。
![](http://i.imgur.com/k4Cxjvm.png)

# 消息队列相关概念 #
MQ框架非常的多，包括[RibbitMQ](http://www.rabbitmq.com/), [ActiveMQ](http://activemq.apache.org/), ZeroMQ, [Kafka](http://kafka.apache.org/)等，一般来说，业务操作比较推荐RibbitMQ（基于高并发高可用erlang语言，因此性能有一定优势），而日志类操作推荐Kafka。消息系统的特点包括解耦；冗余，遵循“插入-获取-删除”，保证数据安全；扩展性；灵活性，可以很好的消除请求峰值；可恢复性；顺序保证；缓冲；异步通信。


# Kafka #
Kafka的基础概念如下所示：
**Broker**:Kafka集群包含一个或多个服务器，这种服务器称为broker.
**Topic**:每条发布到Kafka集群的消息都有一个类别，被称为Topic，物理上不同Topic的消息分开存储.
**Partition**:物理上的概念，一个Topic包含一个或多个Partition.
Producer&Consumer：消息生产者和消费者。
Consumer Group: 每个Consumer属于一个特定的Consume Group，便于广播消息。
leader&follower:replica中的一个角色，producer和consumer只能和leader交互，而follower只作为leader(partition)的副本。
controller:集群中的服务器，用来进行leader election和failover。
zookeeper:存储集群的meta信息。

Tip:
producer发送消息时的路由机制为：指定了 patition，则直接使用；未指定 patition 但指定 key，通过对 key 的 value 进行hash 选出一个 patition；patition 和 key 都未指定，使用轮询选出一个 patition。

# 消息队列的实践 #
**QMQ**（未开源）
特点：完全可靠，高容错性，高扩展性，消息支持事务，支持延迟发送消息。
![](http://i.imgur.com/ASSPjyJ.png)
消息生产者按发送消息可靠性分为：事务持久型消息生产者，如支付、出票、航变等交易操作；持久型消息生产者，如事件触发，通知更新等；非持久型消息生产者，抓包，统计数据；非可靠型消息生产者，促销打折活动推广信息。需要注意的是，消息的吞吐量随着可靠型的要求而下降。
![](http://i.imgur.com/UNyUhq5.png)
这部分屡清楚思路就好，其实可以看到消息实际上是写入到Mysql数据库中了，通过这种方式来保证消息的可靠性。

**Hermes**（未开源）
![](http://i.imgur.com/SZy26X9.png)
消息管理：Portal是用户管理信息的站点，用户可以在Portal申请Topic、消费者分组，查看消息的整个流转情况，进行消息的自助管理。
三种存储：MySql用户消息量中等及以下，对消息治理有较高要求的场景；Kafka适用于消息量大的场景；Broker分布式文件存储。
策略：保证至少收到一次消息，Broker或Consumer重启时，可能导致部分消息无法判断是否处理完毕，Broker会重新发送这些消息。

Tip:
百度云推荐产品Kafka的介绍，支持的业务场景包括：从网站、设备采集海量的用户操作数据；汇总分布式应用的遥感数据；对接Spark Streaming等服务以进行实时流数据分析。其特点是：主题可以通过分区Partition来实现水平扩展；分区分布在多个节点上以达到高数据可用性；通过消费者组Consumer Group来支持单个消费者以队列或者Pub/Sub形式的消息消费，或者多个消费者集群顺序消息。


**参考资料**
[Kafka官网](http://kafka.apache.org/)
[腾讯VS阿里VS携程消息中间件设计方案与思路](http://blog.csdn.net/lizhitao/article/details/51718156)
[最全最给力的kafka博客](http://blog.csdn.net/lizhitao/article/category/2194509)
[Kafka学习笔记](http://www.cnblogs.com/cyfonly/p/5954614.html)
