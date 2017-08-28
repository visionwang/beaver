

#Spring框架的本质#
Ioc(Inversion Of Control)：其包括DI和DL两种方式。
DL(Dependency Lookup)：依赖查找，当前软件实体主动去某个服务[注册地]查找器依赖的哪些服务
DI(Dependency Injection)：当前软件实体被动接受其依赖的其他组件被Ioc容器注入
常见Spring框架中都会有context.getBean()代码，实际上就是做DL的操作，而构建的任何一种Ioc容器则是一个DI的过程(BeanFactory, ApplicationContect)。

第一步：收集和注册
第二部：分析与组装

JavaConfig
表现形式层面
注册bean定义层面
表达依赖注入关系层面

@ComponentScan
@PropertySource
@PropertySources

@Import
@ImportResource 

# SpringBoot工作机制 #
@SpringBootApplication

@Configuration：Ioc容器配置类
@EnableAutoConfiguration：借助@Import的支持，收集和注册特定场景相关的bean的定义
@Import(EnableAutoConfigurationImportSelector.class)
通过SpringFactoriesLoader工具类智能的获取所有的配置
spring.factories: 一个key-value的配置文件 key接口，value实现

提供SPI扩展的场景，



@CompontScan

SpringApplication.run();


# 微服务的理解 #
