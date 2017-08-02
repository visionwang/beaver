
# Hermes Info #
Hermes是什么
一个高可用、高吞吐、适合各种应用场景的消息队列。
Why another消息队列
简单、统一One API解决各种业务场景
完善的监控消息如何在系统中流转，用户可以一目了然
高可用、高吞吐、高性能

消息发送
消息由Producer(生产者)创建，然后发送到Broker(消息服务器)，Broker根据消息所属Topic的配置信息确定消息的存储，将消息保存到相应的存储。（问题是）
消息接收
Consumer(消费者)向Broker订阅Topic，然后Broker将Topic对应的消息投递到Consumer进行处理。
消息管理
Portal是用户管理消息的站点，用户可以在Portal申请Topic、消费者分组，查看消息的整个流转情况，进行消息的自助管理等。
三种存储
MySQL适用于消息量中等及以下，对消息治理有较高要求的场景
Kafka适用于消息量大的场景
Broker分布式文件存储用于替换Kafka，开发中

为什么要用分布式文件存储替换Kafka，Kafka的存储是如何的？
![](http://i.imgur.com/SJWodT7.png)
API
Producer
	Producer p = Producer.getInstance();
	Order order = new Order(...);
	p.message("hotel.order", order.getId(), order).send();//分别是Topic, key , value
Consumer
	Consumer c = Consumer.getInstance();
	String groupId = 'group1';
	 
	// start a non-batch consumer
	c.start('hotel.order', groupId, new BaseMessageListener(groupId) {
	    @Override
	    public void onMessage(ConsumerMessage msg) {
	        Order order = msg.getBody();
	        // process the order
	    }
	});
	 
	// start a batch consumer
	c.start('hotel.order', 'groupId', new MessageListener() {
	    @Override
	    public void onMessage(List<ConsumerMessage> msgs) {
	        // batch process orders
	    }
	});

Topic申请
用户可以在Portal申请新的Topic，提供Topic相关的信息。
![](http://i.imgur.com/POksTTq.png)
**消息分区**
消息分区用于保证消息的消费顺序，同一个分区内的消息会按发送顺序依次投递给消费者，不同分区内的消息不保证按发送顺序投递。因此，如果消息之间需要确保消费顺序，请确保这些消息的分区键(partition key)相同。

	p.message("hotel.order", "partition key1", msg1).send();
	p.message("hotel.order", "partition key1", msg2).send();

同时，在有序消费模式下，消息分区的数量决定了同一消费者分组内消费者数量的上限，对于一个消费者分组，一个分区只允许一个消费者消费，因为只有这样才能确保一个分区按顺序消费，因此：**消费者数量(一个消费者分组内) <= 分区数量**

**消费者分组**
![](http://i.imgur.com/je2Dlx4.png)
消费者分组用于控制消息分发的语义，一个消费者分组由若干设置了相同groupId的消费者组成。**Broker在分发消息时，会将每一条消息分发到所有的消费者分组**，**然后在每一个消费者分组内选择一个消费者进行投递**。**即每一个消费者分组都能消费到完整的消息流，互不影响。而同一消费者分组内的消费者“竞争”消费消息，每一条消息只会被消费一次。**
？问题来了，如果共用用户中心的消息，那么就会出现消费者的相关问题。
ConsumeGroup如何申请?

**消息重新投递**
消费者在处理消息时如果调用了消息的nack()方法，则Broker会在设定的时间后重新投递该消息给消费者进行再一次处理，当达到设定的重新投递次数上限后将不再重新投递，而保存到系统的死信(dead letter)存储中，用户可以在Portal查看或者设置将死信消息重新投递。

	new BaseMessageListener<Order>(groupId) {
	    @Override
	    public void onMessage(ConsumerMessage<Order> msg) {
	        msg.nack();
	    }
	}
如果消费者在处理消息(onMessage)时抛出异常，不会重新投递该消息，因此，在处理消息时请使用try-catch-finally捕获异常，并根据具体情况决定是否调用nack()来重新投递消息。
队列，死信队列？

**消息优先级**
Producer在发送消息时可以设定该消息是否优先处理，Broker在投递消息给Consumer时，会优先投递设定为优先处理的消息。
	Producer p = Producer.getInstance();
	p.message(...).withPriority().send();

**批量订阅**
如果需要订阅多个Topic的消息进行统一处理，可以通过批量订阅的方式。

	Consumer c = Consumer.getInstance();
	String groupId = 'group1';
	 
	// start a non-batch consumer
	c.start('hotel.*', groupId, new BaseMessageListener(groupId) {
	    @Override
	    public void onMessage(ConsumerMessage msg) {
	        Order order = msg.getBody();
	        // process the order
	    }
	});

**消息重新消费/放弃消费**
默认情况下，Broker会记录每个消费者分组的offset(消费偏移)，当消费者分组重启后，会从上次消费中断的消息开始投递消息，保证消息的连续性。
Hermes允许用户在Portal上修改消费者分组的offset，通过**调整offset**可以达到重新消费或者放弃消费的效果。
![](http://i.imgur.com/TrPEI3I.png)

**消息过滤**
后续会支持Broker端的消息过滤，和Consumer端过滤相比，Broker端的过滤更加轻量，也可以减少不必要的网络传输。

**消息发送模式**
同步
异步
基于内存
基于文件

Topic命名规范
只允许小写字母、数字和点号(.)，即a-z,0-9,.。推荐的格式为: 产线.a.b.c
如何编译Avro Schema
.Net用户请下载 Avro Codegen(C#)
avrogen.exe -s <schemafile> <destination>
Java用户请下载 avro-tools(Java)
java -jar /path/to/avro-tools-1.7.7.jar compile schema <schema file> <destination>
Portal上传schema之后，也会自动生成jar包，可以从portal下载。
FAQ
会不会丢消息?
消息会收到几次?
– 保证收到至少一次，Broker或者Consumer重启时可能导致部分消息无法判断是否已经处理完毕，Broker会再次重发这些消息。

# Hermes Pull模式消费者使用方法 #
适用场景
希望自己控制接收消息的时机、频率和批量的Job类型消费者。

**Poll方法**
方法参数：
maxMessageCount — 消息最大拉取条数
timeoutMillis — 超时(单位：毫秒)
方法结束条件：
超时
有消息到达(不一定达到maxMessageCount)

	string topic = "topic.test";
	string consumerGroup = "test.group";
	  
	PullConsumerConfig config = new PullConsumerConfig();
	IPullConsumerHolder holder = Consumer.GetInstance().OpenPullConsumer(topic, consumerGroup, typeof(string), config);
	try
	{
	    while (!stop)
	    {
	        IPulledBatch batch = holder.Poll(100, 1000);// 最多拉取100条消息，1秒超时(只要有1条消息到达，Poll方法立刻返回，否则最多Block 1秒)
	        List<IConsumerMessage> msgs = batch.Messages;
	        if (msgs != null && msgs.Count > 0)
	        {
	            foreach (IConsumerMessage msg in msgs)
	            {
	                try
	                {
	                    Console.WriteLine(msg.GetBody<string>());// TODO 你的业务逻辑
	                }
	                catch
	                {
	                    msg.Nack();// 对于有异常的消息或者需要后续重新处理的消息可以Nack
	                }
	            }
	            batch.CommitAsync();
	        }
	    }
	}
	finally
	{
	    holder.Close();// 结束后必须Close
	}


**Collect方法**
这儿有缓冲池，队列的概念
方法参数：
expectedMessageCount — 期望收集到的消息条数
timeoutMillis — 超时(单位：毫秒)
方法结束条件：
超时
收集到的消息数达到expectedMessageCount

	string topic = "topic.test";
	string consumerGroup = "test.group";
	  
	PullConsumerConfig config = new PullConsumerConfig();
	IPullConsumerHolder holder = Consumer.GetInstance().OpenPullConsumer(topic, consumerGroup, typeof(string), config);
	try
	{
	    while (!stop)
	    {
	        IPulledBatch batch = holder.Collect(100, 1000);// 期望1秒内尽量收集100条消息
	        List<IConsumerMessage> msgs = batch.Messages;
	        if (msgs != null && msgs.Count > 0)
	        {
	            foreach (IConsumerMessage msg in msgs)
	            {
	                try
	                {
	                    Console.WriteLine(msg.GetBody<string>());// TODO 你的业务逻辑
	                }
	                catch
	                {
	                    msg.Nack();// 对于有异常的消息或者需要后续重新处理的消息可以Nack
	                }
	            }
	            batch.CommitAsync();
	        }
	    }
	}
	finally
	{
	    holder.Close();// 结束后必须Close
	}

**Commit模式**
Pull模式必须主动commit，否则重启后会从上次commit的最后一条消息继续往后发送。
 提供三种Commit方式
CommitSync() — 直到Commit成功才返回
CommitAsync() — 异步Commit
CommitAsync(IOffsetCommitCallback callback) — 异步Commit，完成则回调callback

IOffsetCommitCallback接口如下：

	public interface IOffsetCommitCallback
	{
	    // offsets里面只包含本次Commit成功的Topic，Partition对应的offset
	    // exception如果非null，则表示Commit失败
	    void OnComplete(Dictionary<TopicPartition, OffsetAndMetadata> offsets, Exception exception);
	}

TopicPartition如下：

	public class TopicPartition
	{
	    public string Topic{ get; set;}
	    public int Partition{get; set;}
	}

OffsetAndMetadata如下：

	public class OffsetAndMetadata
	{
	    public long? PriorityOffset{ get; set; } // 如果本次Commit不包含优先级消息，则返回null
	    public long? NonPriorityOffset{ get; set; } // 如果本次Commit不包含非优先级消息，则返回null
	    public long? ResendOffset{ get; set; } // 如果本次Commit不包含重发消息，则返回null
	}

# Hermes的各种使用方式 #
**消息发送**

	string topic = "topic.test";
	string msg = "some message";
	Producer.GetInstance().Message(topic, null, msg).Send();
**优先消息**
设置为高优先级的消息会先于普通消息被消费者消费。

	string topic = "topic.test";
	string msg = "some message";
	Producer.GetInstance().Message(topic, null, msg).WithPriority().Send();
**如何让消息可以被找到**
为消息设置Reference Key(以下简称RefKey)，RefKey类似一条消息的身份证号，后续可以用RefKey查到这条消息的各种相关信息。
如果不设置RefKey，则无法从Hermes查询某条具体消息的相关信息。

	string topic = "topic.test";
	string msg = "some message";
	string refKey = "order-create-3415234598";
	Producer.GetInstance().Message(topic, null, msg).WithRefKey(refKey).Send();
**如何控制消息的消费顺序**
如何控制消息的消费顺序
为消息设置Partition Key，可以把Partition想象成Topic的多个虚拟通道，发送到Topic的消息会根据Partition Key进入到对应的通道，同一个通道内的消息消费者会依次收到，不同通道内的消息可以并行收到。
因此，如果希望某些消息按顺序处理，就为这些消息设置相同的Partition Key。如果希望某些消息并行处理，就为这些消息设置不同的Partition Key。

	string topic = "topic.test";
	string msg = "some message";
	string partitionKey = "hotelId-1234567";  //同一hotelId的消息顺序处理，不同hotelId的消息可以并行处理
	Producer.GetInstance().Message(topic, partitionKey, msg).Send();

**消息接收**
如何知道消息时何时发送的
	class MyMessageListener : BaseMessageListener
	{
	    //...  
	    protected override void OnMessage(IConsumerMessage msg)
	    {
	        Console.WriteLine(msg.BornTimeUtc);  //获取消息的发送时间，DateTime类型，注意是UTC时间
	    }
	}

**消息接收的顺序**
默认情况下，消费者按序收到每个Partition内的消息，即每个Partition会依次消费。只有重发会产生"消息乱序接收"的情况。
如，收到消息1后消费者NACK或者，此时，消息1会在设定的时间(假设10s)后重新让消费者处理，而后续的消息2会立即让消费者处理，这样，实际的消费顺序是：1(NACK，即处理失败)、2、1(再次处理)。
如果希望在消息1NACK之后还是优先处理消息1，而不是立即处理消息2，可以在消费者设定"严格有序"策略。
设定严格有序后，在消息1处理完成(处理成功或重试次数耗尽)前，不会处理消息2。
**感觉可以通过业务上的处理，解决这个问题，不然这样的阻塞很尴尬**

	MessageListenerConfig config = new MessageListenerConfig();
	config.StrictlyOrderingRetryPolicy = StrictlyOrderingRetryPolicy.EvenRetry(3000, 3);    // 优先重试消费当前消息3次，每次间隔3000ms。在消息1处理完成(处理成功或重试满3次)前，不会处理消息2.
	Consumer.GetInstance().Start(topic, groupId, new MyMessageListener(), config);

**订阅多个Topic**
使用通配符。支持两种通配符，`*`和`#`。Topic名字由点号(.)分隔的多段组成，`*`匹配一段，`#`号匹配一至多段。

	string topicPattern = "a.b.*";  //匹配a.b.c，不匹配a.b.c.d
	string topicPattern = "a.b.#";  //匹配a.b.c，匹配a.b.c.d
	Consumer.GetInstance().Start(topicPattern, groupId, new MyMessageListener());

用同一个Listener，start多次Consumer

	MyMessageListener listener = new MyMessageListener();
	foreach (topic in topics)
	{
	    Consumer.GetInstance().Start(topic, groupId, listener, config);
	}

存储类型的区别：Mysql vs Kafka
Mysql
Kafka
支持nack重发策略	Y	N
支持Pull模式	Y	N
支持RefKey消息跟踪	Y	N
支持avro/json	Y	Y
支持业务监控	Y	N
发送者	Java/.Net	Java/.Net
消费者	Java/.Net	Java
数据量(条/天)	< 10亿	>=10亿


# 常见问题 #

**Hermes如何判断一条消息已经处理完毕**
当OnMessage(IConsumerMessage msg)方法返回或抛出异常时，Hermes即认为msg处理完毕。
以下两种情况，Hermes认为msg处理失败，会重发消息：
用户调用了msg.Nack()
OnMessage()抛出了异常
其它情况下，Hermes都认为msg处理成功。
同时，OnMessage()可以使用线程池来多线程处理，线程数量可配置。用户不应该在OnMessage()中再进行异步化处理，否则OnMessage()一旦返回Hermes就会认为消息已经处理完毕，而事实上消息正常异步处理中。

**为消息设置RefKey**
为每个消息设置"合适"的RefKey，即Producer.GetInstance().Message(...).WithRefKey(refKey)。
string refKey = "order-create-3415234598";
Producer.GetInstance().Message(topic, null, msg).WithRefKey(refKey).Send();
RefKey可以用来唯一标识一条消息，可以用来在Hermes Portal查询这条消息的发送和处理的详细信息。RefKey是消息的"身份证"，如果没有设置RefKey，就没法告诉Hermes需要查询哪一条消息。通过消息内容来全文检索消息是非常低效和慢的操作，设置RefKey以后可以很迅速的定位消息的处理情况。
所谓“合适”的RefKey指两点：
唯一性，RefKey在某个Topic的消息中最好是唯一的，这样可以唯一定位这条消息，没有歧义
业务相关性，RefKey的值最好具有业务含义，比如是订单处理的流水号，这样在排查问题时可以从业务场景推断出消息的RefKey，进而排查出消息的处理情况。


**为消息设置PartitionKey**
Topic在内部分成多个Partition，消息在发送到Topic时会根据PartitionKey进入到不同的Partition。
string partitionKey = "hotelId-1234567";  //同一hotelId的消息顺序处理，不同hotelId的消息可以并行处理
Producer.GetInstance().Message(topic, partitionKey, msg).Send();
Partition有两个重要作用：
提升Topic的吞吐量。不同的Partition可以由不同的Hermes服务器存储和处理，因此，增加Partition的数量可以提高Topic的吞吐能力。
确保消息顺序。一个Partition内的消息，同一时间只能由一个消费者(同一ConsumerGroup内)处理，，因此消费顺序和发送顺序是确保一致的。比如依次发送msg1和msg2到某个Partition，则消费者接受到的消息顺序一定是先msg1再msg2。不同Partition的消息会由不同消费者并行处理，不确保消息顺序。
因此，设置合适的PartitionKey是用好Hermes的关键之一。
选取ParitionKey有两个考虑因素：
PartitionKey的值越多越好。如果Partition只有2个值，则最多将消息发送到2个Partition，会限制Topic的吞吐能力。
为需要确保消费顺序的消息设置同样的PartitionKey。如果依次发送msg1和msg2，需要确保先处理msg1，再处理msg2，则为msg1和msg2设置相同的PartitionKey。否则，可以设置不同的PartitionKey，以增加PartitionKey的值数量。

如果不关心消息的处理顺序，可以将PartitionKey设置成:
C#:  System.Guid.NewGuid().ToString()
Java:  UUID.randomUUID().toString()




1.那我们现在服务时间不同步，
由于Hermes内部使用Lease机制来做多个消费机器的分布式协调，所以会依赖服务器和客户端的时钟同步，现在租约有限期是20秒，所以如果时钟不同步超过20秒就会导致消费者无法拿到有效的租约进行消费。

那就是LRU的方式？会用到时间戳，大体懂了

2.cheetah异常场景？
大家好：
 
测试场景如下:（做了分库分表）
1 Hermes网络连接超时，切换到本地线程，本次请求重试本地线程
2 Hermes topic不存在，切换到本地线程，本次请求重试本地线程
3 切换到本地线程后5分钟，再次请求Hermes，如果Hermes ok走Hermes，否则继续走本地线程,5分钟后再继续重试（重试的基础是有新的请求）
 
Hermes异常的依据是：一个线程连续3次执行一个请求都异常，那么认为集群与Hermes之间异常
 
数据丢失严重，镇平看下原因~
 
 
压测30分钟：26402笔 （18：35：32-19：05：34）
数据落地：oney_move_08   pay_ledger_08    76396 /4   19099   笔  少了7000笔左右
SELECT count(1) FROM fnccheetahshard01db.money_move_08 where datachange_lasttime>='2016-07-05 18:35:30' ;
SELECT count(1) FROM fnccheetahshard01db.pay_ledger_08 where datachange_lasttime>='2016-07-05 18:35:30' ;


几个问题：
 
0).切hermes，要统一，谁先判断，异常了，就先给出标志：要切本地线程；其他人统一来切，不要再重试了；

1).切local thread 、切Hermes的log要区分；

2).回切线程，散列；要统一回切；由一个线程，去拿回切标志；谁拿到，谁去执行，其他人不能起哄，都去执行；

3).数据丢失 严重，本地线程太少，才30笔；系统间补偿的话，能解决最终问题，但是还是要起足够线程来保证，异常处理需要；
      分析lpt的DB数据：          看哪个时间段的数据丢的多；
      (1)hermes异常丢：
      (2)切本地线程丢：
      (3)本地线程丢：
      (4)回切丢：


Step1.构造服务访问Hermes失败（修改hosts）；
Step2.重启cheetah服务（初始化），开始压测；
Step3.压测开始后5分钟，hermes恢复；
Step4..等脚本执行完成；



**参考资料**
