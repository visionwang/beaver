
一、单例模式：在spring中其实是scope(作用范围)参数的缺省设定值
每个bean定义只生成一个对象实例,每次getBean请求获得的都是此实例
单例模式分为饿汉模式和懒汉模式；
饿汉模式
spring singleton的缺省是饿汉模式:启动容器时,为所有spring配置文件中定义的bean都生成一个实例（且是线程安全）
懒汉模式
在第一个请求时才生成一个实例,以后的请求都调用这个实例
spring singleton设置为懒汉模式:
<beans default-lazy-init="true">
关于单例的饿汉，懒汉请参考：http://zliguo.iteye.com/blog/2258879
二、默认情况下为单例模式（饿汉），prototype多实例模式介绍
调用getBean时,就new一个新实例
默认单例（饿汉）：




Spring-context -> spring-beans


[Spring初始化Bean状态](http://zliguo.iteye.com/blog/2258897)
[Spring Bean的生命周期（非常详细）](http://www.cnblogs.com/zrtqsk/p/3735273.html)



