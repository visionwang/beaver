由于最近项目的合作方使用Jenkins作为部署工具，而我方需要将部分应用发布到他们的服务器上，因此打算好好的熟悉一下Jenkins系统。
![](http://i.imgur.com/BZ0qQ8e.png)

# 基本概念 #
Jenkins是基于Java开发的一种持续集成工具，用于监控持续重复的工作，功能包括持续的软件版本发布/测试项目，监控外部调用执行的工作。Jenkins的安装非常简单，既可以以jar包运行`java -jar jenkins.war`，也可以发布到tomcat等应用服务器运行，安装的向导页面也非常清晰，便于配置，接下来，根据官方文档step by step的开始设置。
1.创建第一个Pipeline任务:选择新建item，且选择Multibranch Pipeline，add source选择指定的代码仓库（还可以指定如gitlab的代码浏览器），Save运行。
2.系统管理，安装maven插件，之后设置maven,jdk,configureTools，之后建立maven项目，。
clean install -Dmaven.test,skip=true
Maven Integration plugin
3.构建后操作
Deploy to container Plugin
自动化脚本的概念

# 插件的选择 #
初始安装：按照最小化原则进行选取，之后通过系统设置可以再添加，个人选取build timeout plugin、Workspace Cleanup Plugin、Pipeline、 Git plugin、Email Extension Plugin。




http://www.cnblogs.com/Leo_wl/p/4314792.html
git
gitlab
maven
jenkins
email


http://www.jianshu.com/p/052a2401595a





参考资料
[Docker结合jenkins](https://my.oschina.net/donhui/blog/470372)
https://www.zhihu.com/question/25385945
http://blog.csdn.net/pretent/article/details/53932493



	docker pull jenkins
	docker run --name jenkins01 -p 9090:8080 -p 9091:50000 -v $PWD/jenkins01:/var/jenkins_home -d jenkins


**参考资料**
[官方文档](https://jenkins.io/doc/)
[Docker集成Jenkins进行持续部署](http://www.csdn.net/article/2015-07-21/2825266)