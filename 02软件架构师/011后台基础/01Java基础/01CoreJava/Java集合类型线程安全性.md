Java中各种集合（字符串类）的线程安全性！！！

 

一、概念：

线程安全：就是当多线程访问时，采用了加锁的机制；即当一个线程访问该类的某个数据时，会对这个数据进行保护，其他线程不能对其访问，直到该线程读取完之后，其他线程才可以使用。防止出现数据不一致或者数据被污染的情况。
线程不安全：就是不提供数据访问时的数据保护，多个线程能够同时操作某个数据，从而出现数据不一致或者数据污染的情况。
对于线程不安全的问题，一般会使用synchronized关键字加锁同步控制。
线程安全 工作原理： jvm中有一个main memory对象，每一个线程也有自己的working memory，一个线程对于一个变量variable进行操作的时候， 都需要在自己的working memory里创建一个copy,操作完之后再写入main memory。 
当多个线程操作同一个变量variable，就可能出现不可预知的结果。 
而用synchronized的关键是建立一个监控monitor，这个monitor可以是要修改的变量，也可以是其他自己认为合适的对象(方法)，然后通过给这个monitor加锁来实现线程安全，每个线程在获得这个锁之后，要执行完加载load到working memory 到 use && 指派assign 到 存储store 再到 main memory的过程。才会释放它得到的锁。这样就实现了所谓的线程安全。
 

二、线程安全(Thread-safe)的集合对象：

Vector 
HashTable
StringBuffer
 

三、非线程安全的集合对象：

ArrayList ：
LinkedList：
HashMap：
HashSet：
TreeMap：
TreeSet：
StringBulider：
 

四、相关集合对象比较：

Vector、ArrayList、LinkedList： 
1、Vector： 
Vector与ArrayList一样，也是通过数组实现的，不同的是它支持线程的同步，即某一时刻只有一个线程能够写Vector，避免多线程同时写而引起的不一致性，但实现同步需要很高的花费，因此，访问它比访问ArrayList慢。 
2、ArrayList： 
a. 当操作是在一列数据的后面添加数据而不是在前面或者中间，并需要随机地访问其中的元素时，使用ArrayList性能比较好。 
b. ArrayList是最常用的List实现类，内部是通过数组实现的，它允许对元素进行快速随机访问。数组的缺点是每个元素之间不能有间隔，当数组大小不满足时需要增加存储能力，就要讲已经有数组的数据复制到新的存储空间中。当从ArrayList的中间位置插入或者删除元素时，需要对数组进行复制、移动、代价比较高。因此，它适合随机查找和遍历，不适合插入和删除。 
3、LinkedList： 
a. 当对一列数据的前面或者中间执行添加或者删除操作时，并且按照顺序访问其中的元素时，要使用LinkedList。 
b. LinkedList是用链表结构存储数据的，很适合数据的动态插入和删除，随机访问和遍历速度比较慢。另外，他还提供了List接口中没有定义的方法，专门用于操作表头和表尾元素，可以当作堆栈、队列和双向队列使用。
　　  Vector和ArrayList在使用上非常相似，都可以用来表示一组数量可变的对象应用的集合，并且可以随机的访问其中的元素。

 

HashTable、HashMap、HashSet： 
HashTable和HashMap采用的存储机制是一样的，不同的是： 
1、HashMap： 
a. 采用数组方式存储key-value构成的Entry对象，无容量限制； 
b. 基于key hash查找Entry对象存放到数组的位置，对于hash冲突采用链表的方式去解决； 
c. 在插入元素时，可能会扩大数组的容量，在扩大容量时须要重新计算hash，并复制对象到新的数组中； 
d. 是非线程安全的； 
e. 遍历使用的是Iterator迭代器；

Entry 包含四个属性：key, value, hash 值和用于单向链表的 next。
capacity：当前数组容量，始终保持 2^n，可以扩容，扩容后数组大小为当前的 2 倍。
loadFactor：负载因子，默认为 0.75。
threshold：扩容的阈值，等于 capacity * loadFactor
public V put(K key, V value) {
    // 当插入第一个元素的时候，需要先初始化数组大小
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    // 如果 key 为 null，感兴趣的可以往里看，最终会将这个 entry 放到 table[0] 中
    if (key == null)
        return putForNullKey(value);
    // 1. 求 key 的 hash 值
    int hash = hash(key);
    // 2. 找到对应的数组下标
    int i = indexFor(hash, table.length);
    // 3. 遍历一下对应下标处的链表，看是否有重复的 key 已经存在，
    //    如果有，直接覆盖，put 方法返回旧值就结束了
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }
 
    modCount++;
    // 4. 不存在重复的 key，将此 entry 添加到链表中，细节后面说
    addEntry(hash, key, value, i);
    return null;
}

2、HashTable： 
a. 是线程安全的； 
b. 无论是key还是value都不允许有null值的存在；在HashTable中调用Put方法时，如果key为null，直接抛出NullPointerException异常； 
c. 遍历使用的是Enumeration列举；

3、HashSet： 
a. 基于HashMap实现，无容量限制； 
b. 是非线程安全的； 
c. 不保证数据的有序；

 

TreeSet、TreeMap： 
TreeSet和TreeMap都是完全基于Map来实现的，并且都不支持get(index)来获取指定位置的元素，需要遍历来获取。另外，TreeSet还提供了一些排序方面的支持，例如传入Comparator实现、descendingSet以及descendingIterator等。 
1、TreeSet： 
a. 基于TreeMap实现的，支持排序； 
b. 是非线程安全的；

2、TreeMap： 
a. 典型的基于红黑树的Map实现，因此它要求一定要有key比较的方法，要么传入Comparator比较器实现，要么key对象实现Comparator接口； 
b. 是非线程安全的；

 

StringBuffer和StringBulider： 
StringBuilder与StringBuffer都继承自AbstractStringBuilder类，在AbstractStringBuilder中也是使用字符数组保存字符串。

　　  1、在执行速度方面的比较：StringBuilder > StringBuffer ； 
　　  2、他们都是字符串变量，是可改变的对象，每当我们用它们对字符串做操作时，实际上是在一个对象上操作的，不像String一样创建一些对象进行操作，所以速度快； 
　  　3、 StringBuilder：线程非安全的； 
　　  4、StringBuffer：线程安全的； 


　 
　　对于String、StringBuffer和StringBulider三者使用的总结： 
　　 1.如果要操作少量的数据用 = String 
　 　2.单线程操作字符串缓冲区 下操作大量数据 = StringBuilder 
　　 3.多线程操作字符串缓冲区 下操作大量数据 = StringBuffer

ConcurrentHashMap的历遍是从后向前历遍的，因为如果有另一个线程B在执行clear操作时，会把table中的所有slot都置为null，这个操作是从前向后执行
如果线程A在历遍Map时也是从前向后，则有可能会出现追赶现象。
concurrencyLevel：并行级别、并发数、Segment 数，怎么翻译不重要，理解它。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。

**public V get(Object key)**
先得到 key所在的table，再像HashMap一样get
中间并不加锁

**public V put(K key, V value)**
先得到所属的table，加锁
判断table是否要扩容
如果table要扩容，则产生newTable
hash值相同的slot整体移到newTable
hash值不同的slot，把oldTable中的所有Entry都复制到newTable中
因为有可能其它线程在历遍这个table，所以不能把原来的链表拆断

**public V remove(Object key)**
remove操作，要删除Entry3，则先复制Entry1为Entry1*，Entry1*指向Entry4，再复制Entry2为Entry2*，Entry2*指向Entry1*，最终形成一个两叉的链表。原本的Entry1，Entry2，Entry3会被GC自动回收。

迭代操作：

ConcurrentHashMap的历遍是从后向前历遍的，因为如果有另一个线程B在执行clear操作时，会把table中的所有slot都置为null，这个操作是从前向后执行
如果线程A在历遍Map时也是从前向后，则有可能会出现追赶现象。(胡扯)



添加节点的操作 put 和删除节点的操作 remove 都是要加 segment 上的独占锁的，所以它们之间自然不会有问题，我们需要考虑的问题就是 get 的时候在同一个 segment 中发生了 put 或 remove 操作。


我们根据数组元素中，第一个节点数据类型是 Node 还是 TreeNode 来判断该位置下是链表还是红黑树的。





[Java7/8 中的 HashMap 和 ConcurrentHashMap 全解析](http://www.importnew.com/28263.html)

[为什么说ArrayList的线程不安全？](https://www.cnblogs.com/efforts-will-be-lucky/p/7052672.html)
[ArrayList线程不安全分析](https://blog.csdn.net/u013769320/article/details/46300533)
