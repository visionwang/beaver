

Jetty和Tomcat为目前全球范围内最著名的两款开源的webserver/servlet容器。由于它们的实现都遵循Java Servlet规范，一个Java Web应用部署于两款容器的任意一个皆可。但选择哪个更优？也许这得看场景。 

近期很多人关注Jetty，也许主要是因为GAE放弃了Tomcat而选择了Jetty。于是，以很直接的想法，Jetty更符合GAE的需求、即云环境的需求，亦分布式环境的需求。

那Jetty与Tomcat比较，有哪差异呢？ 自己简单做了些调研，也请救了熟悉Tomcat和Jetty的朋友和师兄，得出以下结论：
 
    1）Jetty更轻量级。这是相对Tomcat而言的。

    由于Tomcat除了遵循Java Servlet规范之外，自身还扩展了大量JEE特性以满足企业级应用的需求，所以Tomcat是较重量级的，而且配置较Jetty亦复杂许多。但对于大量普通互联网应用而言，并不需要用到Tomcat其他高级特性，所以在这种情况下，使用Tomcat是很浪费资源的。这种劣势放在分布式环境下，更是明显。换成Jetty，每个应用服务器省下那几兆内存，对于大的分布式环境则是节省大量资源。而且，Jetty的轻量级也使其在处理高并发细粒度请求的场景下显得更快速高效。

 

    2）Jetty更灵活，体现在其可插拔性和可扩展性，更易于开发者对Jetty本身进行二次开发，定制一个适合自身需求的Web Server。

    相比之下，重量级的Tomcat原本便支持过多特性，要对其瘦身的成本远大于丰富Jetty的成本。用自己的理解，即增肥容易减肥难。

 

    3）然而，当支持大规模企业级应用时，Jetty也许便需要扩展，在这场景下Tomcat便是更优的。



    总结：Jetty更满足公有云的分布式环境的需求，而Tomcat更符合企业级环境。

 
    GAE放弃了Tomcat，选择了Jetty，正是因为Jetty的体积和灵活性，Google可以更好地定制一个足够小的Java Web Server为其GAE服务。
     而Tomcat为满足更多的企业级需求，增加了JEE特性，在服务企业级应用时，它的支持优于Jetty。然而，即使Tomcat性能略优于Jetty，但对于大多非企业级应用而言，配置复杂体积庞大的Tomcat显得过于重量级。
 
    正因为这个，实验室的云平台实现便是把云平台本身的门户网站放在Tomcat内，而云台托管的Java Web应该是部署在Jetty内的。
 

Tip:
GAE 英文全称为 Google App Engine。它是 Google 管理的数据中心中用于 WEB 应用程序的开发和托管的平台。2008 年 4月 发布第一个测试版本。目前支持python、java和php开发。全球已有数十万的开发者在其上开发了众多的应用。



**参考资料**
[Google 选择 Jetty, 放弃 Tomcat](http://www.iteye.com/news/9918)


