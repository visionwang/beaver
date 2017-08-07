由于最近项目的合作方使用Jenkins作为部署工具，而我方需要将部分应用发布到他们的服务器上，因此打算好好的熟悉一下Jenkins系统。

# 概念 #
Jenkins是基于Java开发的一种持续集成工具，用于监控持续重复的工作，功能包括持续的软件版本发布/测试项目，监控外部调用执行的工作。Jenkins的安装非常简单，既可以以jar包运行`java -jar jenkins.war`，也可以发布到tomcat等应用服务器运行，安装的向导页面也非常清晰，便于配置，接下来，根据官方文档step by step的开始设置。
1.创建第一个Pipeline任务:选择新建item，且选择Multibranch Pipeline，add source选择指定的代码仓库（还可以指定如gitlab的代码浏览器），Save运行。









**参考资料**
[官方文档](https://jenkins.io/doc/)
[Docker集成Jenkins进行持续部署](http://www.csdn.net/article/2015-07-21/2825266)