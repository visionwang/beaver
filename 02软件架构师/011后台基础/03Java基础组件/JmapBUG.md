2.5 观察heapdump
jvisualvm按类实例大小排序，int数组容量有2G，然后查看实例，一个数组长度为4亿，容量为1.6G，查看引用，确定是由图片相关的类导致的，但是visualvm无法知道确切原因。


google搜索之后，基本确定是jmap的bug，参见https://bugs.openjdk.java.net/browse/JDK-8073605 （10行代码重现低版本jmapbug）

我建议SRE升级java版本以解决dump失败的问题，SRE目前没有标准流程应对此情况，建议我自己手动往服务器增加新java，以测试问题。


这个版本的jdk在执行jmap时居然将旧生代的百分比算成了负数。
从数据库中拉取当时的html类型凭证。
[jihongming@overseas-openplatform-service06.beta ~]$ java -version
    java version "1.8.0_45"
    Java(TM) SE Runtime Environment (build 1.8.0_45-b14)
    Java HotSpot(TM) 64-Bit Server VM (build 25.45-b02, mixed mode)
[jihongming@overseas-openplatform-service06.beta ~]$ javac Test.java
[jihongming@overseas-openplatform-service06.beta ~]$ java Test &
[1] 174343
[jihongming@overseas-openplatform-service06.beta ~]$ 2
[jihongming@overseas-openplatform-service06.beta ~]$ jmap -F -dump:format=b,file=./jvmdump2 174343


不加-F拉不下来dump文件

直到某一刻你终于绷不住了：“就算我能承受，孩子一直被当成吊车尾，他愿不愿意承受？”我认识不少朋友，从前都是很酷的青年人，声称：“我要有了娃，一定不逼他上清华北大。北大老子都上过了，没什么意思。”现在他们还是咬着牙给孩子报兴趣班。他们照样还是很酷的人，但是，谁敢拿孩子冒险呢？

到处鼓吹阶层固化，鼓吹学习改变命运。
所谓的军备竞赛，实质就是一场囚徒博弈。
每个人身在局中，都无力脱身。
你不加码，就被别人踩在脚下。
你一旦加码，就为恐慌添薪加柴。

一线城市发酵，慢慢辐射全国。
撤出了，恐慌还在。
就像有的人卖了北京的房搬到大理，但他们逃不脱房价上涨带来的失落。除非这些人从一开始就承认：“我退出”，从根上斩断烦恼。




