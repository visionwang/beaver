

SpringBootServletInitializer是SpringBoot启动的核心，帅帅哒。




# 微服务基础概念 #
微服务虽然是最近一两年兴起的一个比较时髦的技术名词，但实际上其并非是什么新的理念，很多比较好的企业早都已经在使用和实施微服务了，其本质就是SOA概念的进一步发展。



# Spring与SpringBoot基础知识 #

[Spring Boot干货系列：（三）启动原理解析](http://www.cnblogs.com/zheting/p/6707035.html)

jar @SpringBootApplication main
war 
这个JAR包与传统JAR包的不同之处在于里面有一个名为lib的目录，在这个目录中包含了这个简单应用所依赖的其他JAR包，其中也包含内置的嵌 入式Tomcat，正是使用它，才能发布服务和访问Web资源。除了我们编写的源码所编译形成的CLASS以外，在org目录下还有许多Spring所提 供的CLASS，正是依赖这些CLASS，才能够加载位于lib目录下JAR中的类。这样的加载机制与在OSGi bundle中声明Bundle-Classpath很类似，不过在OSGi中会由容器来负责加载指定路径下的类。这大致阐述了这样一个JAR包能够发布 服务的原因。


自动配置幕后英雄：SpringFactoriesLoader详解

1.修改成

	<packaging>war</packaging>  
2.新增如下到pom.xml文件中

	<dependency>  
	    <groupId>org.springframework.boot</groupId>  
	    <artifactId>spring-boot-starter-tomcat</artifactId>  
	    <scope>provided</scope>  
	</dependency>  
3.新增ServletInitializer类

	import org.springframework.boot.builder.SpringApplicationBuilder;  
	import org.springframework.boot.context.web.SpringBootServletInitializer;  
	public class ServletInitializer extends SpringBootServletInitializer {  
	    @Override  
	    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {  
	        return application.sources(Application.class);  
	    }  
	}  


对于需要部署到传统servlet容器之中的应用，Boot提供了一种方式以编码的方式初始化Web配置。为了使用这一点，Boot提供了可选的WebApplicationInitializer，它会使用servlet容器来注册应用，这会通过Servlet 3.0 API以编码的方式注册servlet并且会用到ServletContext。通过提供SpringBootServletInitializer的子类，Boot应用能够使用嵌入的Spring上下文来注册配置，这个Spring上下文是在容器初始化的时候创建的。为了阐述这个功能，考虑程序清单1.27中的示例代码。


# SpringBoot微服务实践 #


#补充知识#
spring-boot-starter基础知识

Scala


**参考资料**
1. 王福强. SpringBoot揭秘[M]. 北京:机械工业出版社, 2017.



那么现在问题来了，这个web项目里面没有web.xml，也就是说，没有配置任何初始化、过滤器或者其他Servlet，为什么能启动spring容器呢？
首先要看看Servlet3.0中的一个新接口javax.servlet.ServletContainerInitializer
ServletContainerInitializer 是 Servlet 3.0 新增的一个接口，容器在启动时使用 JAR 服务 API(JAR Service API) 来发现 ServletContainerInitializer 的实现类，并且容器将 WEB-INF/lib 目录下 JAR 包中的类都交给该类的 onStartup() 方法处理，我们通常需要在该实现类上使用 @HandlesTypes 注解来指定希望被处理的类，过滤掉不希望给 onStartup() 处理的类。
也就是说，这里用到了Java的SPI机制。

SPI的全名为Service Provider Interface。这个是针对厂商或者插件的。在java.util.ServiceLoader的文档里有比较详细的介绍。简单的总结下java spi机制的思想。
我们系统里抽象的各个模块，往往有很多不同的实现方案，比如日志模块的方案，xml解析模块、jdbc模块的方案等。
面向的对象的设计里，我们一般推荐模块之间基于接口编程，模块之间不对实现类进行硬编码。一旦代码里涉及具体的实现类，就违反了可拔插的原则，如果需要替换一种实现，就需要修改代码。
**为了实现在模块装配的时候能不在程序里动态指明，这就需要一种服务发现机制**。 java spi就是提供这样的一个机制：为某个接口寻找服务实现的机制。
有点类似IOC的思想，就是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要。
java spi的具体约定为:**当服务的提供者，提供了服务接口的一种实现之后，在jar包的META-INF/services/目录里同时创建一个以服务接口命名的文件。该文件里就是实现该服务接口的具体实现类。**
而当外部程序装配这个模块的时候，就能通过该jar包META-INF/services/里的配置文件找到具体的实现类名，并装载实例化，完成模块的注入。 
基于这样一个约定就能很好的找到服务接口的实现类，而不需要再代码里制定。jdk提供服务实现查找的一个工具类：**java.util.ServiceLoader**
在spring-web-${version}.jar文件里面，可以看到

META-INF/services/javax.servlet.ServletContainerInitializer
文件里面的内容是：
org.springframework.web.SpringServletContainerInitializer
意思就是web容器在启动的时候，会去加载所有的javax.servlet.ServletContainerInitializer接口的实现类，然后调用onStartup方法做初始化工作。
所以，在没有web.xml的情况下，也能进行spring容器的启动