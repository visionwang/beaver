公司最近项目需要用到Dubbo，因此进行一个简单的学习，目前来看，大的趋势是Spring Cloud可能会一统天下，但不管如何，现在有针对性的学习一下dubbo也是很有必要的。

# Dubbo #
Dubbo是一个分布式服务框架，提供高性能透明化的RPC远程服务调用方案，并提供**SOA服务治理**方案，其核心部分包含:
**远程通讯**: 提供对多种基于长连接的NIO框架抽象封装，包括多种线程模型，序列化，以及“请求-响应”模式的信息交换方式。
**集群容错**: 提供基于接口方法的透明远程过程调用，包括多协议支持，以及软负载均衡，失败容错，地址路由，动态配置等集群支持。
**自动发现**: 基于注册中心目录服务(ZooKeeper)，使服务消费方能动态的查找服务提供方，使地址透明，使服务提供方可以平滑增加或减少机器。


**Dubbo Admin**

参考资料
[dubbo admin安装实践](http://blog.csdn.net/evankaka/article/details/47858707)
[springboot整合dubbo](http://blog.csdn.net/hong0220/article/details/51072872)
[springboot+dubbo](http://www.cnblogs.com/cl2Blogs/p/5692203.html)

# CXF #
Apache CXF = Celtix + XFire，相比于Axis等框架，其是目前最好用的SOAP协议实现框架，其主要特点如下所示。
Web 服务标准支持：CXF 支持以下 Web 服务标准：
Java API for XML Web Services (JAX-WS)
SOAP
Web 服务描述语言（Web Services Description Language ，WSDL）
消息传输优化机制（Message Transmission Optimization Mechanism，MTOM）
WS-Basic Profile
WS-Addressing
WS-Policy
WS-ReliableMessaging
WS-Security
前端建模：CXF 提供了前端建模的概念，允许您使用不同的前端 API 来创建 Web 服务。API 允许您使用简单的工厂 Bean 并通过 JAX-WAS 实现来创建 Web 服务。它还允许您创建动态 Web 服务客户端。
工具支持：CXF 提供了用于在 Java Bean、Web 服务和 WSDL 之间进行转换的不同工具。它提供了对 Maven 和 Ant 集成的支持，并无缝地支持 Spring 集成。
RESTful 服务支持：CXF 支持代表性状态传输（Representational State Transfer，RESTful ）服务的概念，并支持 Java 平台的 JAX-RS 实现。（本系列的第 2 部分将提供有关 RESTful 服务的更多信息。）
对不同传输和绑定的支持：CXF 支持不同种类的传输，从 XML 到逗号分隔值 (CSV)。除了支持 SOAP 和 HTTP 协议绑定之外，它还支持 Java Architecture for XML Binding (JAXB) 和 AEGIS 数据绑定。
对非 XML 绑定的支持：CXF 支持非 XML 绑定，例如 JavaScript Object Notation (JSON) 和 Common Object Request Broker Architecture (CORBA)。它还支持 Java 业务集成（Java Business Integration，JBI）体系架构和服务组件体系架构（Service Component Architecture，SCA）。
code first 或者 xml first  ： 支持使用code first 或者 xml first 的方式来创建web服务。

Tip:
[dubbo官网](http://dubbo.io/)
[CXF官网](http://cxf.apache.org/)

