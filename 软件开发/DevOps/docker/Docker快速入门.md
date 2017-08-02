Docker已经火了很长一段时间，最近打算在**阿里云**上好好熟悉一下Docker的相关部署发布，为今后做准备，本文的重点部分为**阿里云上实际环境的搭建和Dockerfile的配置**。
![](http://i.imgur.com/EGXyo48.png)

# 基本概念 #
Docker是基于Go语言实现的云开源项目，诞生于2013年初，最初发起者是dotCloud公司，其目标是“Build, Ship and Run Any App, Anywhere”，主要概念包括**镜像、容器、仓库**。Docker引擎的技术是Linux容器(**Linux Containers**, LXC)技术。容器有效地将由单个操作系统的资源划分到孤立的组中，以便更好地在孤立的组之间平衡有冲突的资源使用需求。
![](http://i.imgur.com/B2ue5LF.png)
- **镜像Image**:类似于虚拟机镜像，可以理解为面向Docker引擎的**只读模板**，包括文件系统。
获取镜像：`docker pull NAME[:TAG]`
查看镜像信息: 查看所有镜像`docker images`；查看某个镜像具体信息`docker inspect`
添加标签: `docker tag xxx ubuntu:first`
搜寻镜像: `docker search xxx`, `-s=0`指定星级
删除镜像: `docker rmi xxx`，一般情况下会删除镜像的标签，而不是文件，当删除最后一个TAG时则会删除文件，需要注意。
使用镜像ID删除镜像: `-f`删除可以强制删除镜像，推荐做法为先删除依赖该镜像的所有容器，之后删除镜像，`Qdocker rm e81`
创建镜像: 创建镜像包括3种方式，基于已有镜像的容器创建，首先启动一个镜像`docker run -ti ubuntu:14.04 /bin/bash`，任意创建一个test文件，之后创建镜像`docker commit -m "add file" -a "xionger" a9fdsfxx test`；基于本地模板创建，推荐使用OpenVZ提供的模板来创建；基于Dockerfile创建。
存出镜像和载入镜像（导出、导入）: 导出到本地文件`sudo docker save -o ubuntu_14.04.tar ubuntu:14.04`，导入镜像`docker load --input ubuntu_14.04.tar`
上传镜像: `docker push NAME[:TAG]`，默认上传镜像到DockerHub官方仓库，需要登录。
- **容器Container**：类似一个轻量级的沙箱，可以利用容器来运行和隔离应用，容器从镜像启动时会在镜像的**最上层创建一个可写层，镜像本身保持不变**。
创建容器：`docker create -it ubuntu:lastest`,通过`docker ps -a`查看容器，通过`docker start`启动容器
新建并启动容器：`docker run ubuntu /bin/bash`,`-d`参数守护态运行，通过Ctrl+d或者exit退出容器
终止容器：`docker stop xxx`，首先会发送SIGTERM信号，一段时候后发送SIGKILL,可以通过`docker kill`强行中止，`docker restart`可以关闭并重启容器，`docker ps -a -q`可以查看处于终止态的容器信息。
进入容器：`docker attach xxx`会被阻塞不推荐使用；`docker exec -ti xxx /bin/bash`可以直接在容器中运行命令；nsenter工具。
删除容器：`docker rm xxx`，需要注意区分，`rmi`是删除镜像，`rm`是删除容器
导入和导出容器：`docker export xxx`导出一个已经创建的容器到文件，不管是否在运行；`docker import`，需要理解的是`export`的是快照，信息少，而`save`的是镜像，信息多，包含元数据和历史信息。
- **仓库Repositor**y：类似于代码仓库，是Docker存放镜像的场所，而Registry注册服务器是存放仓库的地方，其上放着很多仓库，每个仓库集中存放某一类镜像的多个文件，可以通过tag标签来区分。目前最大的公有仓库是[Docker Hub](https://hub.docker.com/)，而国内是Docker Pool。
Docker Pub:本地用户目录.dockercfg中存储登录信息，在仓库中存在centos这类由Docker公司创建、验证、支持的根镜像，也有类似xionger/centos这类由个人提供的镜像，可以通过`-s N`来查看高星镜像。此外，Docker Hub还可以通过设置追踪类似GitHub的网站，然后根据其更行，自动执行创建。
创建和使用私有仓库：可以通过官方提供的`registry`镜像来简单搭建一套本地私有仓库环境。
	docker run -d -p 5000:5000 -v /opt/data/registry:/tmp/registry registry
	docker images
	docker tag ubuntu:14.04 139.196.96.27:5000/test
	docker push 139.196.xx.xx:5000/test
	curl http://139.196.xx.xx:5000/v1/search
	docker pull 139.196.xx.xx:5000/test

Tip:
CURL（CommandLine Uniform Resource Locator）:curl是利用URL语法在命令行方式下工作的开源文件传输工具。
**安装Docker**(Ubuntu16.04)

	sudo apt-get install apt-transport-https
	sudo apt-get update
	sudo apt-get install -y docker.io

Tip:
在用putty连接阿里云时，经常会断开，如何解决？
解决方法：在Connection里面有个Seconds between keepaliaves。这里就是每间隔指定的秒数，就给服务器发送一个空的数据包，来保持连接。以免登录的主机那边在长时间没接到数据后，会自动断开SSH的连接，设置为10。

# 进阶概念 #
**数据管理**:在使用docker过程中，会涉及查看容器内应用产生的数据，或者数据在多个容器间共享，此时需要管理数据的两种方式包括数据卷Data Volumes和数据卷容器Data Volume Containers. 
**数据卷**：是一个可供容器使用的特殊目录，绕过文件系统，具有的特性包括数据卷可以在容器之间共享和重用、对数据卷的修改会马上生效、对数据卷的更新不会影响镜像、卷会一致存在，知道没有容器使用，类似Linux下对目录或文件进行mount操作。
在容器内创建一个数据卷：使用`training/webapp`镜像创建一个web容器，并创建一个数据卷挂在到容器的/webapp目录，`docker run -d -P --name web -v /webapp python app.py`。
挂载一个主机目录作为数据卷：加载主机的`/src/webapp`目录到容器的`/opt/webapp`目录,`docker run -d -P --name web - v /src/webapp:/opt/webapp training/webapp python app.py`。
Tip:编辑工具包括vi或者sed --in-place，推荐挂载目录而不是文件，因为inode变化会造成docker容器启动失败。
**数据卷容器**：其实就是一个普通的容器，其中会挂载数据卷用户共享，创建数据库容器dbdata，之后其他容器将挂载可以挂载该数据卷容器中的数据卷。

	docker run -it -v /dbdata --name dbdata ubuntu
	ls
	docker run -it --volumes-from dbdata --name db1 ubuntu

**利用数据卷容器迁移数据**：可以通过数据卷容器对其中的数据卷进行备份、回复，以实现数据的迁移。接下来的示例利用ubuntu镜像创建一个容器worker，使用`--volumes-from dbdata`参数挂载dbdata容器的数据卷，
使用`-v ${pwd}:/backup`参数来挂载本地的当前目录到worker容器的`/backup`目录，
容器启动后，使用`tar cvf /backup/backup.tar /dbdata`来讲`/dbdata`下内容备份为容器内的`/backup/backup.tar`。

	docker run --volumes-from dbdata -v ${pwd}:/backup --name worker ubuntu tar cvf /backup/backup.tar /dbdata
	//恢复,首先创建一个带有数据卷的容器dbdata2，之后 创建另一个新的容器，挂载dbdata2容器，并使用untar解压备份文件到所挂载的容器卷中即可
	docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
	docker run --volumes-from dbdata2 -v ${pwd}:/backup busybox tar xvf /backup/backup.tar
在生产环境，推荐使用分布式文件系统Ceph、GPFS、HDFS定期对主机的本地数据进行备份。
**网络基础配置**:
端口映射实现访问容器：在启动容器时，如果不指定对应参数，在容器外部是无法通过网络来访问容器内的网络应用和服务的。可以使用`-p ip:hostPort:containerPort`映射端口,`docker logs`查看应用的信息，`docker port`查看端口配置。
	
	docker run -d -p 5000:5000 -p 3000:80 training/webapp python app.py
容器互联实现容器间通信：容器见的连接系统是除了端口映射外另一种可以与容器中应用进行交互的方式，它会在源和接受容器间创建一个隧道，接受容器可以看到源容器制定的信息，比如`--link`连接应用容器和数据库容器，这样可以保证db的接口不暴露到公网。

	docker run -d -P --name web training/webapp python app.py
	docker ps -l
	docker inspect -f xxx
	//容器互联
	docker run -d --name db training/postgres
	docker rm -f web
	docker run -d -P --name web --link db:db training/webapp python app.py
	docker ps
Docker通过两种方式为容器公开连接信息，包括环境变量`env`和/etc/hosts文件，通过`apt-get install -yqq inetutils-ping`安装ping。
扩展知识：Docker核心技术、Docker安全、高级网络配置、其他项目

# 使用Dockerfile创建镜像 #
**基本结构**：dockerfile由命令语句组成，支持#开头的注释，分为4个部分，包括基础镜像信息、维护者信息、镜像操作指令和容器启动执行指令，在**docker hub上有很多dockerfile的demo**，需要时可以直接使用。

	#基础镜像
	FROM ubuntu
	#维护者信息
	MAINTAINER xionger xiongere@email.com
	#镜像的操作指令
	RUN apt-get update && apt-get install -y nginx
	RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf
	#容器启动时执行指令
	CMD /usr/sbin/nginx

**指令**：一般格式为INSTRUCTION arguments，具体如下所示。
FROM `<image>:<tag>`默认的第一条指令
MAINTAINER `<name>`维护者信息
RUN `<command>`或者RUN `["executable", "param1", "param2"]`，前者将在shell终端中运行命令，即`/bin/sh -c`，后者则使用`exec`执行。
CMD  `["executable", "param1", "param2"]`使用`exec`执行，推荐方式。
EXPOSE `<port> [<port>..]`告诉Docker服务器容器暴露的端口号，供互联网系统使用。
ENV `<key> <value>`指定一个环境变量，会被后续的`RUN`指令使用
ADD `<src> <dest>`该命令将复制指定<src>到容器中的<dest>
COPY `<src> <dest>`复制本地主机`<src>`到容器中`<dest>`,推荐使用
ENTRYPOINT `["executable", "param1", "param2"]`配置容器启动后执行的命令，不能被`docker run`提供的参数覆盖
VOLUME `["/data"]`创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据。
USER `daemon`指定运行容器时的UID，后续的`RUN`也会使用指定用户，如`RUN group add -r postgres && useradd -r -g postgres postgres`，要获取管理员权限时可以使用`gosu`而不是`sudo`
WORKDIR `path/to/workdir`为后续的指令配置工作目录
ONBUILD `[INSTRUCTION]`配置当所创建的镜像作为其他新创建镜像的基础镜像时，所执行的操作指令。
![](http://i.imgur.com/6mKi3y5.png)
**创建镜像**：编写好dockerfile后，可以通过`docker build`命令来创建镜像，该命令将读取指定路径下（包括子目录）的dockerfile，并将该路径下所有内容发送给docker服务端，由服务端来创建镜像，此外可以通过`.dockerignore`文件来忽略目录或文件，还可以通过`-t`指定镜像的标签信息。示例`docker build -t build_repo/first_image /tmp/docker_builder/`

# 实践之道 #
**操作系统**:CentOS和Ubuntu都可以，个人喜好ubuntu（还可以选用debian:jessie， alpine），属于最基础的镜像。
tip: 当试图安装软件出现没有相关包信息时，需要`apt-get update`或编辑`/etc/apt/sources.list`文件（`deb, deb-src`,需要时在查询，比如163的镜像，阿里云的话无需设置）,可以通过`netstat -tunlp`查看当前网络情况。
**支持SSH**：当需要直接进入容器进行管理时安装，不必须。
**Web服务器与应用**(Nginx,可以使用淘宝优化的Tengine代替Nginx，Tomcat)：在`/root`下创建`tomcat`,`nginx`目录应用存放Dockerfile文件，最终还是选择通过pull拉去镜像的方式安装应用，dockerfile比较复杂。

	docker pull nginx
	docker ps -a
	//-p 80:80：将容器的80端口映射到主机的80端口
	//-name mynginx：将容器命名为mynginx
	//-v $PWD/www:/www：将主机中当前目录下的www挂载到容器的/www
	//-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf：将主机中当前目录下的nginx.conf挂载到容器的/etc/nginx/nginx.conf
	//-v $PWD/logs:/wwwlogs：将主机中当前目录下的logs挂载到容器的/wwwlogs
	docker run --name nginx01 -p 80:80 -v $PWD/www:/www -v $PWD/conf:/etc/nginx -v $PWD/logs:/wwwlogs -d nginx

	docker pull tomcat
	docker run --name tomcat01 -p 8080:8080 -v $PWD/test:/usr/local/tomcat/webapps/test -d tomcat
tip:有时可能需要重启docker服务, `service docker restart`，可以选择[tomcat7.0:jdk1.8]
[nginx配置详解](http://blog.csdn.net/tjcyjd/article/details/50695922)
[nginx官方文档](http://nginx.org/en/docs/)
**数据库应用**MySQL, MongoDB, Redis
	
	docker pull mysql
	docker run -p 3306:3306 --name mysql01 -v $PWD/conf:/etc/mysql -v $PWD/logs:/logs -v $PWD/data:/mysql_data -e MYSQL_ROOT_PASSWORD=xxxxx -d mysql
	//主从模式
	docker run -d -e REPLICATION_MASTER=true -P --name mysql01 mysql
	docker run -d -e REPLICATION_SLAVE=true -P --name mysql02 --link mysql01:mysql01 mysql
	//mongodb
	docker pull mongo
	docker run -p 27017:27017 --name mongodb01 -v $PWD/db:/data/db -d mongo
	//redis
	docker pull  redis
	docker run -p 6379:6379 --name redis01 -v $PWD/data:/data  -d redis redis-server --appendonly yes
tip:可以进入db的容器进行操作，`docker exec -ti mysql /bin/bash`
**构建Docker容器集群**：
**阿里云安装Docker**：

**逻辑上的环境搭建视图**

阿里云购买ECS， 操作系统版本Ubuntu 16.04（LTS）
**目前实践计划**
私有Docker仓库暂时不建立，先使用DockerHub；Git类似，先使用Github；Maven需要使用Nexus建立一个私有库；jenkins之间搭建就好。

**参考资料**
1. 杨保华. Docker技术入门与实践[M]. 北京:机械工业出版社, 2016.
2. [Docker常见安装指南](http://www.runoob.com/docker/ubuntu-docker-install.html)




