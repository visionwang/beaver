HashMap和双向链表合二为一即是LinkedHashMap。所谓LinkedHashMap，其落脚点在HashMap，因此更准确地说，它是一个将所有Entry节点链入一个双向链表的HashMap。由于LinkedHashMap是HashMap的子类，所以LinkedHashMap自然会拥有HashMap的所有特性。比如，LinkedHashMap的元素存取过程基本与HashMap基本类似，只是在细节实现上稍有不同。当然，这是由LinkedHashMap本身的特性所决定的，因为它额外维护了一个双向链表用于保持迭代顺序。此外，LinkedHashMap可以很好的支持LRU算法，笔者在第七节便在LinkedHashMap的基础上实现了一个能够很好支持LRU的结构。


transient
2、LinkedHashMap 的扩容操作 : resize()
　　在HashMap中，我们知道随着HashMap中元素的数量越来越多，发生碰撞的概率将越来越大，所产生的子链长度就会越来越长，这样势必会影响HashMap的存取速度。为了保证HashMap的效率，系统必须要在某个临界点进行扩容处理，该临界点就是HashMap中元素的数量在数值上等于threshold（table数组长度*加载因子）。但是，不得不说，扩容是一个非常耗时的过程，因为它需要重新计算这些元素在新table数组中的位置并进行复制处理。所以，如果我们能够提前预知HashMap 中元素的个数，那么在构造HashMap时预设元素的个数能够有效的提高HashMap的性能。 

到此为止，我们已经分析完了LinkedHashMap的存取实现，这与HashMap大体相同。LinkedHashMap区别于HashMap最大的一个不同点是，前者是有序的，而后者是无序的。为此，LinkedHashMap增加了两个属性用于保证顺序，分别是双向链表头结点header和标志位accessOrder。我们知道，header是LinkedHashMap所维护的双向链表的头结点，而accessOrder用于决定具体的迭代顺序。实际上，accessOrder标志位的作用可不像我们描述的这样简单，我们接下来仔细分析一波~ 
　　 

一、      关于LRU
LRU 即 Least  Rencetly  Used（最近最少使用）缓存替换策略。在任何LRU算法中，它必定有以下两个策略组成：
1、  退化 策略。根据访问情况，对节点按热度进行排序（hot->cold），以便决定哪些节点是热节点（hot）的，哪些节点是冷节点（cold）的。这个退化的策略，一般按以下两种方式去处理：
l  非集中式。即每命中一次就进行退化操作。
非集中式的退化操作，往往由双向链表的方式去实现。每次命中之后就移动命中节点在链表中的位置。（位置靠前的就是hot的数据）。当然，复杂的策略中，有用queue数组进行hot分级等。
l  集中式。定期去进行退化操作。
在集中式的退化操作，常用的策略是：每次命中之后，记录一个时间戳、定时器时间点等等参数。由一个线程去扫描，定期清除老数据。
2、  清除 策略。即去掉那些cold的数据。
l  替换。这个在操作系统缓存中应该是一个常用的做法。
l  删除。删除掉数据，以腾出空间放新的数据。（因为内存是有限的）

四、      ConcurrentLRUHashMap实现方式二：直接改造ConcurrentHashMap
该方案主要是重写ConcurrentHashMap。
1、  给每个Entry加一个timestamp。
2、  每次get命中的话，修改时间戳。
但是，由于时间片不可控，最终将导致内存爆炸的情况出现。

https://blog.csdn.net/a921122/article/details/51992713
https://blog.csdn.net/njchenyi/article/details/8046914


    今天我们介绍的是本地缓存缓存，我们这边采用java.util.concurrent.ConcurrentHashMap来保存，ConcurrentHashMap是一个线程安全的HashTable，并提供了一组和HashTable功能相同但是线程安全的方法，ConcurrentHashMap可以做到读取数据不加锁，提高了并发能力。我们先不考虑内存元素回收或者在保存数据会出现内存溢出的情况，我们用ConcurrentHashMap模拟本地缓存，当在高并发环境一下，会出现一些什么问题？
   CacheBuilder.newBuilder()后面能带一些设置回收的方法：
     （1）maximumSize(long)：设置容量大小，超过就开始回收。
     （2）expireAfterAccess(long, TimeUnit)：在这个时间段内没有被读/写访问，就会被回收。
     （3）expireAfterWrite(long, TimeUnit)：在这个时间段内没有被写访问，就会被回收 。
        (4)removalListener(RemovalListener)：监听事件，在元素被删除时，进行监听。


设计原则是这样的：
ConsumeStatus.CONSUME_FAILURE：失败后无限制重试，直到消费成功，在消费逻辑与消息内容不匹配的场景，保证消息不丢失，待修复消费逻辑后，可以继续消费
ConsumeStatus.RECONSUME_LATER：失败后有限制重试，默认3次，之后会将消息丢弃，继续消费后续消息，一般应用在积压敏感型场景，实在不能消费时不会影响后续消息的消费
业务同学需要根据自己业务类型，选择具体的返回值

afterNodeInsertion


