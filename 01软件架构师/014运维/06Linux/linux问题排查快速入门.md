top命令是linux非常基础的性能分析工具，可以查看系统中各个进程的资源占用情况，之前一直理解的比较粗浅，在一次面试中`load到底指什么？`给问懵逼了，因此过去一直简单的把Load认为对应到某个百分比，错的非常厉害！

#Top命令
load average: 1.15, 1.42, 1.44 — load average后面的三个数分别是1分钟、5分钟、15分钟的负载情况。
load average

	top - 16:07:37 up 241 days, 20:11,  1 user,  load average: 0.96, 1.13, 1.25
	Tasks: 231 total,   1 running, 230 sleeping,   0 stopped,   0 zombie
	Cpu(s): 12.7%us,  8.4%sy,  0.0%ni, 77.1%id,  0.0%wa,  0.0%hi,  1.8%si,  0.0%st
	Mem:  12196436k total, 12056552k used,   139884k free,    64564k buffers
	Swap:  2097144k total,   151016k used,  1946128k free,  3120236k cached
	
	PID     USER      PR    NI   VIRT    RES     SHR    S   %CPU    %MEM        TIME+   COMMAND
	18411   pplive    20     0  11.9g   7.8g    5372    S  220.2    67.1     16761:00   java
	 1875   pplive    20     0  3958m   127m    4564    S    4.6     1.1     12497:35   java
	    4   root      20     0      0      0       0    S    0.3     0.0    184:01.76   ksoftirqd/0
	   13   root      20     0      0      0       0    S    0.3     0.0    135:49.83   ksoftirqd/2
	   25   root      20     0      0      0       0    S    0.3     0.0    136:54.49   ksofti

top命令的结果分为两个部分：

* 统计信息：前五行是系统整体的统计信息；
* 进程信息：统计信息下方类似表格区域显示的是各个进程的详细信息，默认5秒刷新一次。

##统计信息说明
###第1行：Top 任务队列信息(系统运行状态及平均负载)，与uptime命令结果相同。
第1段：系统当前时间，例如：16:07:37
第2段：系统运行时间，未重启的时间，时间越长系统越稳定。 
格式：up xx days, HH:MM
例如：241 days, 20:11, 表示连续运行了241天20小时11分钟
第3段：当前登录用户数，例如：1 user，表示当前只有1个用户登录
第4段：<mark>系统负载，即任务队列的平均长度，3个数值分别统计最近1，5，15分钟的系统平均负载</mark>
系统平均负载：单核CPU情况下，0.00 表示没有任何负荷，1.00表示刚好满负荷，超过1侧表示超负荷，**理想值是0.7**；
多核CPU负载：CPU核数 * 理想值0.7 = 理想负荷，例如：4核CPU负载不超过2.8何表示没有出现高负载。
<mark>之前面试遇到过该问题，因此这儿的load仅仅指的是cpu的负载</mark>

###第2行：Tasks 进程相关信息 
第1段：进程总数，例如：Tasks: 231 total, 表示总共运行231个进程
第2段：正在运行的进程数，例如：1 running,
第3段：睡眠的进程数，例如：230 sleeping,
第4段：停止的进程数，例如：0 stopped,
第5段：僵尸进程数，例如：0 zombie

###第3行：Cpus CPU相关信息，如果是多核CPU，按数字1可显示各核CPU信息，此时1行将转为Cpu核数行，数字1可以来回切换。
第1段：us **用户空间**占用CPU百分比，例如：Cpu(s): 12.7%us,
第2段：sy **内核空间**占用CPU百分比，例如：8.4%sy,
第3段：ni 用户进程空间内改变过优先级的进程占用CPU百分比，例如：0.0%ni,
第4段：id 空闲CPU百分比，例如：77.1%id,
第5段：wa 等待输入输出的CPU时间百分比，例如：0.0%wa,
第6段：hi CPU服务于硬件中断所耗费的时间总额，例如：0.0%hi,
第7段：si CPU服务软中断所耗费的时间总额，例如：1.8%si,
第8段：st Steal time 虚拟机被hypervisor**偷去的CPU时间**（如果当前处于一个hypervisor下的vm，实际上hypervisor也是要消耗一部分CPU处理时间的）
<mark>关于用户空间和内核空间？</mark>

###第4行：Mem 内存相关信息（Mem: 12196436k total, 12056552k used, 139884k free, 64564k buffers） 
第1段：物理内存总量，例如：Mem: 12196436k total,
第2段：使用的物理内存总量，例如：12056552k used,
第3段：空闲内存总量，例如：Mem: 139884k free,
第4段：用作**内核缓存**的内存量，例如：64564k buffers
<mark>如何理解用于内核缓存的内存量？</mark>

###第5行：Swap 交换分区相关信息（Swap: 2097144k total, 151016k used, 1946128k free, 3120236k cached） 
第1段：交换区总量，例如：Swap: 2097144k total,
第2段：使用的交换区总量，例如：151016k used,
第3段：空闲交换区总量，例如：1946128k free,
第4段：缓冲的交换区总量，3120236k cached
<mark>如何理解缓冲的交换区总量？</mark>

##进程信息
在top命令中按f按可以查看显示的列信息，按对应字母来开启/关闭列，大写字母表示开启，小写字母表示关闭。带*号的是默认列。<mark>(未明白)</mark>
A: PID = (Process Id) 进程Id；
E: USER = (User Name) 进程所有者的用户名；
H: PR = (Priority) 优先级
I: NI = (Nice value) nice值。负值表示高优先级，正值表示低优先级
<remark>NI, VIRT, SHR完全没有理解<remark>
O: VIRT = (Virtual Image (kb)) 进程使用的虚拟内存总量，单位kb。VIRT=SWAP+RES
Q: RES = (Resident size (kb)) 进程使用的、未被换出的物理内存大小，单位kb。RES=CODE+DATA
T: SHR = (Shared Mem size (kb)) 共享内存大小，单位kb

W: S = (Process Status) 进程状态。D=不可中断的睡眠状态,R=运行,<mark>S=睡眠</mark>,T=跟踪/停止,Z=僵尸进程
K: %CPU = (CPU usage) 上次更新到现在的CPU时间占用百分比
N: %MEM = (Memory usage (RES)) 进程使用的物理内存百分比
M: TIME+ = (CPU Time, hundredths) 进程使用的CPU时间总计，单位1/100秒 
X: COMMAND = (Command name/line) 命令名/命令行

Tip:
PID（process ID）PID是程序被操作系统加载到内存成为进程后动态分配的资源。
PPID（parent process ID）：PPID是程序的父进程号。
**内核态与用户态**是指CPU的运行状态（即特权级别），每个进程的每种CPU状态都有其运行上下文，运行上下文就包括了当前状态所使用的空间，CPU访问的逻辑地址（即空间）通过地址映射表映射到相应的物理地址（即物理内存）。在Linux系统中，进程的用户空间是独立的，而内核空间是公用的，进程切换时，用户空间切换，内核空间不变。对于多数CPU而言，处于内核态时，可以访问所有地址空间，而处于用户态时，就只能访问用户空间了。
查看多核CPU命令`mpstat -P ALL`  和  `sar -P ALL`

#jStack命令
记得前段时间，同事说他们测试环境的服务器cpu使用率一直处于100%，本地又没有什么接口调用，为什么会这样？cpu使用率居高不下，自然是有某些线程一直占用着cpu资源，那又如何查看占用cpu较高的线程？

一次排错的全过程
1.jps查找java运行所属进程或使用tasklist找到javaw所属进程的pid
2.使用jstack `pid`查询进程信息
通过top -Hp 23344可以查看该进程下各个线程的cpu使用情况
这里使用的仍然是pid，为什么是线程？
然后可以看到第一条线程占用的cpu时间最多，有意思是，其他资源的占用是一样的，因为都属于一个进程，只有cpu占用情况不用。

通过top命令定位到cpu占用率较高的线程之后，继续使用jstack pid命令查看当前java进程的堆栈状态
jstack命令生成的thread dump信息包含了JVM中所有存活的线程，为了分析指定线程，必须找出对应线程的调用栈，应该如何找？

在top命令中，已经获取到了占用cpu资源较高的线程pid，将该pid转成16进制的值，在thread dump中每个线程都有一个nid，找到对应的nid即可；隔段时间再执行一次stack命令获取thread dump，区分两份dump是否有差别，在nid=0x246c的线程调用栈中，发现该线程一直在执行JstackCase类第33行的calculate方法，得到这个信息，就可以检查对应的代码是否有问题。







#通过thread dump分析线程状态
除了上述的分析，大多数情况下会基于thead dump分析当前各个线程的运行情况，如是否存在死锁、是否存在一个线程长时间持有锁不放等等。
在dump中，线程一般存在如下几种状态：
1、RUNNABLE，线程处于执行中
2、BLOCKED，线程被阻塞
3、WAITING，线程正在等待

实例1：多线程竞争synchronized锁
很明显：线程1获取到锁，处于RUNNABLE状态，线程2处于BLOCK状态
1、locked <0x000000076bf62208>说明线程1对地址为0x000000076bf62208对象进行了加锁；
2、waiting to lock <0x000000076bf62208> 说明线程2在等待地址为0x000000076bf62208对象上的锁；
3、waiting for monitor entry [0x000000001e21f000]说明线程1是通过synchronized关键字进入了监视器的临界区，并处于"Entry Set"队列，等待monitor，具体实现可以参考[深入分析synchronized的JVM实现](https://www.jianshu.com/p/c5058b6fe8e5)；

lock对象、线程A和线程B三者是一种什么关系？根据上面的结论，可以想象一个场景：
1、lock对象维护了一个等待队列list；
2、线程A中执行lock的wait方法，把线程A保存到list中；
3、线程B中执行lock的notify方法，从等待队列中取出线程A继续执行；
当然了，Hotspot实现不可能这么简单。

synchronized代码块通过javap生成的字节码中包含 ** monitorenter ** 和 ** monitorexit **指令。
表示线程执行lock.wait()方法时，必须持有该lock对象的monitor，如果wait方法在synchronized代码中执行，该线程很显然已经持有了monitor。
wait方法会将当前线程放入wait set，等待被唤醒，并放弃lock对象上的所有同步声明，意味着线程A释放了锁，线程B可以重新执行加锁操作，不过又有一个疑问：在线程A的wait方法释放锁，到线程B获取锁，这期间发生了什么？线程B是如何知道线程A已经释放了锁？好迷茫....
4、线程B执行加锁操作成功，对于notify方法，JDK注释：notify方法会选择wait set中**任意**一个线程进行唤醒；

在HotSpot虚拟机中，monitor采用ObjectMonitor实现。
每个线程都有两个ObjectMonitor对象列表，分别为free和used列表，如果当前free列表为空，线程将向全局global list请求分配ObjectMonitor。










**参考资料**
[Linux性能分析工具top命令详解](http://www.linuxidc.com/Linux/2016-08/133871.htm)
[Linux top命令的用法详细详解](https://www.cnblogs.com/edgedance/p/7044753.html)
[JVM调优之jstack找出最耗cpu的线程并定位代码
](https://www.cnblogs.com/chengJAVA/p/5821218.html)
[如何使用jstack分析线程状态](https://www.jianshu.com/p/6690f7e92f27)
