"学无长幼，达者为先"，作者陈康贤通过3年左右时间就能写出如此著作确实令人钦佩，加油，熊二，早日成为一个合格的后端程序员。

# 基础概念 #
**SOA（Service-Oriented Architecture）**：由于互联网场景下，应用越来越复杂，系统经历了三个阶段的变化`单一应用架构->垂直应用架构->分布式应用架构`
单一应用架构到垂直应用架构：通过业务划分，将流量分散到不同的子系统，但子系统见可能存在重叠业务，需要重复造轮子，且容易形成信息孤岛。
垂直应用架构到分布式应用架构：通过基于HTTP协议的RPC风格服务可以很好的解决应用间通信的问题。
**服务风格**：RPC(Remote Process Call)远程过程调用，通过方法名区分服务；Restful风格，通过GET,POST等Http请求方法区分服务。
**协议基础**：主要是TCP协议和Http协议，前者特点是由于处于网络协议的第四层，因此报文较小，网络开销小，性能高，但实现的代价高，不利于跨平台；而后者基于网络协议的第五层，可以支持JSON&XML格式数据传输，利于跨平台(不同终端)。
**序列化形式**：常见的包括Java内置序列化、(google)Protocal Buffers，Hessian, XML, Json等。此外，网上Json序列化性能比较非常多，最简明的请见[FastJson、Gson和Jackson性能比较](http://blog.csdn.net/accountwcx/article/details/50252657)，**结论推荐Jackson**。
**服务路由和负载均衡**：在SOA架构中，服务消费者需要通过服务名称，第一步需要通过**路由**在众多服务中获取到可供调用的服务列表，第二步通过**负载均衡**算法在所选服务的集群地址列表中选择一台服务器用于服务调用。接下来通过一张dubbo的架构图来加深理解。
![](http://i.imgur.com/Vi5MzbL.png)
Tip:
传统的负载均衡还包括基于硬件的F5，基于软件的LVS和Nginx。
**负载均衡算法**：包括[加权]轮询(Round Robin)法、[加权]随机(Random)法、源地址Hash法、最小连接数(Least Connections)法等，还可以动态的设置规则，比如动态解析存放在ZooKeeper中的规则文件。需要注意的是，轮询由于涉及到使用锁，因此效率上有一些问题，接下来通过一段随机轮询法的代码加深理解。

	public static void main(String[] args) throws Exception {
		List<ServerInfo> list = Lists.newArrayList();
		list.add(new ServerInfo("127.0.0.1", 1));
		list.add(new ServerInfo("127.0.0.2", 2));
		list.add(new ServerInfo("127.0.0.3", 3));
		List<String> finalServerList = Lists.newArrayList();
		for (ServerInfo item : list) {
			for (int i = 0; i < item.getWeight(); i++)
				finalServerList.add(item.getIp());
		}
		//int total = list.stream().mapToInt(i -> i.getWeight()).sum();
		// AtomicInteger curCount = new AtomicInteger(0);
		while (true) {
			Random rand = new Random();
			int opt = rand.nextInt(finalServerList.size());
			System.out.println(finalServerList.get(opt));
		}
	}

ZooKeeper相关知识请见[ZooKeeper入门学习](http://www.cnblogs.com/wanliwang01/p/zookeeper01.html)，接下来请见一个模拟的Zookeeper树。此外，针对不同地域地域可以再添加一个层次，或者在节点数据中表明地域。
![](http://i.imgur.com/PLZJlkA.png)

**Http服务网关**：之前提到通过SOA体系，各类类型的终端、第三方的ISV应用都可以发送服务请求，由于http协议所包含的信息基本都是未经加密的明文（请求参数、返回值、cookie、head等），外界可以很容易的进行监听并伪造变造请求，因此系统需要一个公共的网关统一处理这类问题。该gateway即可以很好的解决安全问题，也可以提供通过服务名称的路由和负载均衡服务。

**参考资料**
1. 陈康贤. 大型分布式网站架构[M]. 北京:电子工业出版社, 2014.