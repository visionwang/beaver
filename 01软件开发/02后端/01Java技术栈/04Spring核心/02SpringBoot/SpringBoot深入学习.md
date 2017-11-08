

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



