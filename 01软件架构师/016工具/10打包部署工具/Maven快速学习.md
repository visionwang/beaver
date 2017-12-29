Maven是Java中最为普及的包管理工具，在实际项目中由于依赖的各类jar包非常多，因此概念清晰的处理好各类Jar依赖显得非常重要，接下来通过基础知识，Jar包主要分类方式和进阶知识来介绍。

# 基础知识 #
Maven使用起来非常方便，配置卸载setting.xml中，如果是自己本地开发，推荐直接使用如下配置。

    <?xml version="1.0" encoding="UTF-8"?>
    <settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>E:\javaAssist\maven\repository</localRepository>
    <pluginGroups>
    </pluginGroups>
    <proxies>
    </proxies>
    </servers>
    <mirrors>
    <!-- 阿里云仓库 -->
    <mirror>
    <id>alimaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>http://maven.aliyun.com/nexus/content/repositories/central/</url>
    </mirror>
    <!-- 中央仓库1 -->
    <mirror>
    <id>repo1</id>
    <mirrorOf>central</mirrorOf>
    <name>Human Readable Name for this Mirror.</name>
    <url>http://repo1.maven.org/maven2/</url>
    </mirror>
    <!-- 中央仓库2 -->
    <mirror>
    <id>repo2</id>
    <mirrorOf>central</mirrorOf>
    <name>Human Readable Name for this Mirror.</name>
    <url>http://repo2.maven.org/maven2/</url>
    </mirror>
    </mirrors>
    <profiles>
    </profiles>
    </settings>

**常见命令**

|命令|诠释|
|:--|:--|
| mvn package| 生成target目录，编译、测试代码，生成测试报告，生成jar/war文件 |
| mvn tomcat7:run| 运行项目于tomcat[jetty等server也OK]上|
| mvn compile | 编译|
|mvn test   |编译并测试 |
| mvn deploy -Dmaven.test.skip=true| 发布|
| mvn clean| 清空生成的文件|
| mvn install -X| 想要查看完整的依赖踪迹，包含那些因为冲突或者其它原因而被拒绝引入的构件，打开 Maven 的调试标记运行 |
Tip: 对于community版本的Idea就需要借助maven命令来启动tomcat容器。
此外生产环境，有时会指定特殊版本的jar，因此需要使用systemscope，如下所示

	<dependency>
	<groupId>com.xxxxx.security</groupId>
	<artifactId>encryption</artifactId>
	<version>1.0.0.2</version>
	<scope>system</scope>
	<systemPath>${project.basedir}/src/main/resources/lib/encryption-1.0.0.2.jar</systemPath>
	</dependency>

**Maven的Scope**

|名称|诠释|
|:--|:--|
| compile |编译范围，默认的范围 |
| provided | 已提供范围，比如有些web相关jar，tomcat中有，但本地没有时使用|
| runtime | 运行时范围，比如编译时只需要slf4j-api，运行时才需要具体的实现jar|
|test	 |测试范围，例如junit,spring-test |

#依赖库分类
在java项目中，通常我们将所有的库归类为三种：
**一方库**：指当前项目中各模块间的相互依赖，其中每个模块都可以定义为该工程的一方库。（工程范围内）
**二方库**：指当前公司内部jar包间的相互依赖，公司内部除了当前工程之外的jar及依赖都可以定义为该工程的二方库。（公司范围内）
**三方库**：指公司之外的开源库， 比如apache、google、spring等第三方提供的jar及依赖。

#进阶知识
项目中经常遇到jar包冲突的问题，通常的都是遇到冲突后再去检查，很被动。即使是通过 mvn dependency:tree -Dverbose 也只能是从jar层面自己分析是否有冲突。如何提前检测，如何准确定位到具体类在哪个jar包中存在冲突，这才是解决问题的根本。现推荐一款maven插件可以具体定位到哪些类有冲突，在哪些jar包中有冲突。
**使用方法**
在pom.xml文件build标签中添加如下代码（根据自己代码视情况添加）

	<pluginManagement>
	     <plugins>
	            <plugin>
	                   <groupId>org.apache.maven.plugins</groupId>
	                   <artifactId>maven-enforcer-plugin</artifactId>
	                   <version>1.1.1</version>
	                   <dependencies>
	                   <dependency>
	                          <groupId>org.codehaus.mojo</groupId>
	                          <artifactId>extra-enforcer-rules</artifactId>
	                          <version>1.0-alpha-4</version>
	                   </dependency>
	                   </dependencies>
	                   
	                   <configuration>
	                          <rules>
	                                 <bannedDependencies>
	                                        <searchTransitive>true</searchTransitive>
	                                 </bannedDependencies>
	                                 <evaluateBeanshell>
	                                        <condition>print("[INFO] [Alibaba Enforcer Rules] parent-pom ");1==1</condition>
	                                 </evaluateBeanshell>
	                                 <banDuplicateClasses>
	                                        <findAllDuplicates>true</findAllDuplicates>
	                                      <message>[ERROR] [Alibaba Enforcer Rules] find DuplicateClasses</message>
	                                 </banDuplicateClasses>
	                          </rules>
	                   </configuration>
	            </plugin>
	     </plugins>
	</pluginManagement>

之后在对应项目根目录执行一下maven命令：`mvn enforcer:enforce`。冲突较多单屏无法显示完整时可以重定向到某文件中，如下：`mvn enforcer:enforce > conflict.txt`。其中`Duplicate classes`显示的是重复的类，`Found in`显示的是重复的类所在的jar包。此外，可以通过`添加VM参数-verbose:class`可显示每个文件加载自哪个jar包。

附上一个基础的Maven文件Demo

	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	    <groupId>com.bjorktech.cayman</groupId>
	    <artifactId>cm-web</artifactId>
	    <packaging>war</packaging>
	
	    <parent>
	        <artifactId>cayman</artifactId>
	        <groupId>com.bjorktech</groupId>
	        <version>1.0.0-SNAPSHOT</version>
	    </parent>
	
	    <properties>
	        <!-- Web -->
	        <jsp.version>2.3.1</jsp.version>
	        <jstl.version>1.2</jstl.version>
	        <servlet.version>3.1.0</servlet.version>
	        <!-- <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding> -->
	    </properties>
	
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-webmvc</artifactId>
	        </dependency>
	        <!-- <dependency> <groupId>javax</groupId> <artifactId>javaee-web-api</artifactId> 
	            <version>7.0</version> <scope>provided</scope> </dependency> -->
	        <dependency>
	            <groupId>javax.servlet</groupId>
	            <artifactId>jstl</artifactId>
	            <version>${jstl.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>javax.servlet</groupId>
	            <artifactId>javax.servlet-api</artifactId>
	            <version>${servlet.version}</version>
	            <scope>provided</scope>
	        </dependency>
	        <dependency>
	            <groupId>javax.servlet.jsp</groupId>
	            <artifactId>javax.servlet.jsp-api</artifactId>
	            <version>${jsp.version}</version>
	            <scope>provided</scope>
	        </dependency>
	        <!-- tx -->
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-tx</artifactId>
	        </dependency>
	        <!-- aop -->
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-aop</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.aspectj</groupId>
	            <artifactId>aspectjweaver</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.aspectj</groupId>
	            <artifactId>aspectjrt</artifactId>
	            <version>1.8.5</version>
	        </dependency>
	        <!-- mybatis -->
	        <dependency>
	            <groupId>org.mybatis</groupId>
	            <artifactId>mybatis</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>com.google.guava</groupId>
	            <artifactId>guava</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>joda-time</groupId>
	            <artifactId>joda-time</artifactId>
	        </dependency>
	        <!-- log -->
	        <dependency>
	            <groupId>org.slf4j</groupId>
	            <artifactId>slf4j-api</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>ch.qos.logback</groupId>
	            <artifactId>logback-access</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>ch.qos.logback</groupId>
	            <artifactId>logback-classic</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>ch.qos.logback</groupId>
	            <artifactId>logback-core</artifactId>
	        </dependency>
	        <!-- <dependency> <groupId>org.apache.logging.log4j</groupId> <artifactId>log4j-api</artifactId> 
	            </dependency> <dependency> <groupId>org.apache.logging.log4j</groupId> <artifactId>log4j-core</artifactId> 
	            </dependency> <dependency> <groupId>org.apache.logging.log4j</groupId> <artifactId>log4j-slf4j-impl</artifactId> 
	            </dependency> -->
	        <!-- 本地tomcat -->
	        <!-- <dependency> <groupId>org.apache.tomcat</groupId> <artifactId>tomcat-servlet-api</artifactId> 
	            </dependency> -->
	        <!-- json -->
	        <dependency>
	            <groupId>com.fasterxml.jackson.dataformat</groupId>
	            <artifactId>jackson-dataformat-xml</artifactId>
	            <version>2.5.3</version>
	        </dependency>
	        <dependency>
	            <groupId>com.fasterxml.jackson.core</groupId>
	            <artifactId>jackson-databind</artifactId>
	            <version>2.5.3</version>
	        </dependency>
	        <!-- 文件上传 -->
	        <dependency>
	            <groupId>commons-fileupload</groupId>
	            <artifactId>commons-fileupload</artifactId>
	            <version>1.3.1</version>
	        </dependency>
	        <dependency>
	            <groupId>commons-io</groupId>
	            <artifactId>commons-io</artifactId>
	            <version>2.3</version>
	        </dependency>
	        <!-- 老式soap -->
	        <dependency>
	            <groupId>javax.xml.rpc</groupId>
	            <artifactId>javax.xml.rpc-api</artifactId>
	            <version>1.1.1</version>
	        </dependency>
	        <dependency>
	            <groupId>org.apache.axis</groupId>
	            <artifactId>axis</artifactId>
	            <version>1.4</version>
	        </dependency>
	        <!-- axis依赖包 -->
	        <dependency>
	            <groupId>commons-discovery</groupId>
	            <artifactId>commons-discovery</artifactId>
	            <version>0.2</version>
	            <exclusions>
	                <exclusion>
	                    <artifactId>commons-logging</artifactId>
	                    <groupId>commons-logging</groupId>
	                </exclusion>
	            </exclusions>
	        </dependency>
	        <dependency>
	            <groupId>wsdl4j</groupId>
	            <artifactId>wsdl4j</artifactId>
	            <version>1.6.3</version>
	        </dependency>
	        <!-- xml解析 -->
	        <dependency>
	            <groupId>dom4j</groupId>
	            <artifactId>dom4j</artifactId>
	            <version>1.6.1</version>
	        </dependency>
	        <!-- 测试 -->
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-test</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>junit</groupId>
	            <artifactId>junit</artifactId>
	        </dependency>
	    </dependencies>
	    <build>
	        <finalName>cm-web</finalName>
	        <plugins>
	            <plugin>
	                <groupId>org.apache.maven.plugins</groupId>
	                <artifactId>maven-compiler-plugin</artifactId>
	            </plugin>
	            <plugin>
	                <groupId>org.apache.maven.plugins</groupId>
	                <artifactId>maven-war-plugin</artifactId>
	                <configuration>
	                    <!-- 无web.xml不报错 -->
	                    <failOnMissingWebXml>false</failOnMissingWebXml>
	                </configuration>
	            </plugin>
	        </plugins>
	    </build>
	</project>
