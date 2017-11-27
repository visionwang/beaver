



LinkedBlockingDeque的重要字段有如下几个：

//队列的头节点
 transient Node<E> first;
    //队列的尾节点
    transient Node<E> last;

    //队列中元素的个数
    private transient int count;

    //队列中元素的最大个数
    private final int capacity;

    //锁
    final ReentrantLock lock = new ReentrantLock();

    //队列为空时，阻塞take线程的条件队列
    private final Condition notEmpty = lock.newCondition();

    //队列满时，阻塞put线程的条件队列
    private final Condition notFull = lock.newCondition();


从上面的字段，可以得到LinkedBlockingDeque内部只有一把锁以及该锁上关联的两个条件，所以可以推断同一时刻只有一个线程可以在队头或者队尾执行入队或出队操作。可以发现这点和LinkedBlockingQueue不同，LinkedBlockingQueue可以同时有两个线程在两端执行操作。



[从LongAdder看更高效的无锁实现](http://developer.51cto.com/art/201404/436505.htm)

    public void add(long x) {
        Cell[] as; long b, v; int m; Cell a;
        if ((as = cells) != null || !casBase(b = base, b + x)) {
            boolean uncontended = true;
            if (as == null || (m = as.length - 1) < 0 ||
                (a = as[getProbe() & m]) == null ||
                !(uncontended = a.cas(v = a.value, v + x)))
                longAccumulate(x, null, uncontended);
        }
    }

什么LongAdder会比AtomicLong更高效了，没错，唯一会制约AtomicLong高效的原因是高并发，高并发意味着CAS的失败几率更高， 重试次数更多，越多线程重试，
减少并发，将单一value的更新压力分担到多个value中去，降低单个value的 “热度”，分段更新！！！


这样，线程数再多也会分担到多个value上去更新，只需要增加value就可以降低 value的 “热度”  AtomicLong中的 恶性循环不就解决了吗

当我需要总数时，把cells 中的value都累加一下不就可以了么！！

当我们的场景是为了统计技术，而不是为了更细粒度的同步控制时，并且是在多线程更新的场景时，LongAdder类比AtomicLong更好用。 在小并发的环境下，论更新的效率，两者都差不多。但是高并发的场景下，LongAdder有着明显更高的吞吐量，但是有着更高的空间复杂度。





**低并发时LongAdder和AtomicLong性能差不多，高并发时LongAdder更高效！**

上文中分析了AtomicLong以及Unsafe，本文将为大家带来LongAdder的分析.LongAdder之前在guava以及hystrix等中出现，但是目前已经出现在jdk8标准库中了，作者是著名的Doug lea大师。


LongAdder中会维护一个或多个变量，这些变量共同组成一个long型的“和”。当多个线程同时更新（特指“add”）值时，为了减少竞争，可能会动态地增加这组变量的数量。“sum”方法（等效于longValue方法）返回这组变量的“和”值。

从上面的java doc来看，LongAdder有两大方法，add和sum。其更适合使用在多线程统计计数的场景下，在这个限定的场景下比AtomicLong要高效一些，下面我们来分析下为啥在这种场景下LongAdder会更高效。


答案就在LongAdder的java doc中，从我们翻译的那段可以看出，LongAdder适合的场景**是统计求和计数的场景**，而且LongAdder基本只提供了add方法，而AtomicLong还具有cas方法(要使用cas，在不直接使用unsafe之外只能借助AtomicXXX了)





分布式领域CAP理论，
Consistency(一致性), 数据一致更新，所有数据变动都是同步的
Availability(可用性), 好的响应性能
Partition tolerance(分区容错性) 可靠性

定理：任何分布式系统只可同时满足二点，没法三者兼顾。
忠告：架构师不要将精力浪费在如何设计能满足三者的完美分布式系统，而是应该进行取舍。

关系数据库的ACID模型拥有 高一致性 + 可用性 很难进行分区：
Atomicity原子性：一个事务中所有操作都必须全部完成，要么全部不完成。
Consistency一致性. 在事务开始或结束时，数据库应该在一致状态。
Isolation隔离层. 事务将假定只有它自己在操作数据库，彼此不知晓。
Durability. 一旦事务完成，就不能返回。
跨数据库事务：2PC (two-phase commit)， 2PC is the anti-scalability pattern (Pat Helland) 是反可伸缩模式的，JavaEE中的JTA事务可以支持2PC。因为2PC是反模式，尽量不要使用2PC，使用BASE来回避。

BASE模型反ACID模型，完全不同ACID模型，牺牲高一致性，获得可用性或可靠性：
Basically Available基本可用。支持分区失败(e.g. sharding碎片划分数据库)
Soft state软状态 状态可以有一段时间不同步，异步。
Eventually consistent最终一致，最终数据是一致的就可以了，而不是时时高一致。

BASE思想的主要实现有
1.按功能划分数据库
2.sharding碎片 

BASE思想主要强调基本的可用性，如果你需要High 可用性，也就是纯粹的高性能，那么就要以一致性或容错性为牺牲，BASE思想的方案在性能上还是有潜力可挖的。

现在NOSQL运动丰富了拓展了BASE思想，可按照具体情况定制特别方案，比如忽视一致性，获得高可用性等等，NOSQL应该有下面两个流派：
1. Key-Value存储，如Amaze Dynamo等，可根据CAP三原则灵活选择不同倾向的数据库产品。
2. 领域模型 + 分布式缓存 + 存储 （Qi4j和NoSql运动），可根据CAP三原则结合自己项目定制灵活的分布式方案，难度高。

这两者共同点：都是关系数据库SQL以外的可选方案，逻辑随着数据分布，任何模型都可以自己持久化，将数据处理和数据存储分离，将读和写分离，存储可以是异步或同步，取决于对一致性的要求程度。

不同点：NOSQL之类的Key-Value存储产品是和关系数据库头碰头的产品BOX，可以适合非Java如PHP RUBY等领域，是一种可以拿来就用的产品，而领域模型 + 分布式缓存 + 存储是一种复杂的架构解决方案，不是产品，但这种方式更灵活，更应该是架构师必须掌握的。

不BB，Idea最终要用起来


我们知道，AtomicLong已经是非常好的解决方案了，涉及并发的地方都是使用CAS操作，在硬件层次上去做 compare and set操作。效率非常高。
因此，我决定研究下，为什么LongAdder比AtomicLong高效


正式开始前，强调下，我们知道，AtomicLong的实现方式是内部有个value 变量，当多线程并发自增，自减时，均通过CAS 指令从机器指令级别操作保证并发的原子性。