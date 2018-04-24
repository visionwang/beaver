https://www.cnblogs.com/hafiz/p/6160298.html

1.依赖管理

　　　　1).删除项目中存在的Log4j1.x所必须的log4j和slf4j-log4j12等依赖。

　　　　　　可以到项目的根目录，执行：mvn dependency:tree > tree.log，之后使用 cat tree.log | grep log4j命令进行查找。

 1     <exclusions>  
 2         <exclusion>  
 3             <groupId>org.slf4j</groupId>  
 4           **  <artifactId>slf4j-log4j12</artifactId>  **
 5         </exclusion>  
 6         <exclusion>  
 7             <groupId>log4j</groupId>  
 8             <artifactId>log4j</artifactId>  
 9         </exclusion>  
10     </exclusions>  
　　　　2).添加以下slf4j和log4j2的依赖.
 <!-- slf4j核心包-->  
 2         <dependency>  
 3             <groupId>org.slf4j</groupId>  
 4             <artifactId>slf4j-api</artifactId>  
 5             <version>1.7.13</version>  
 6         </dependency>  
 7         <dependency>  
 8             <groupId>org.slf4j</groupId>  
 9             <artifactId>jcl-over-slf4j</artifactId>  
10             <version>1.7.13</version>  
11             <scope>runtime</scope>  
12         </dependency>  
13   
14         <!--核心log4j2jar包-->  
15         <dependency>  
16             <groupId>org.apache.logging.log4j</groupId>  
17             <artifactId>log4j-api</artifactId>  
18             <version>2.4.1</version>  
19         </dependency>  
20         <dependency>  
21             <groupId>org.apache.logging.log4j</groupId>  
22             <artifactId>log4j-core</artifactId>  
23             <version>2.4.1</version>  
24         </dependency>  
25         <!--用于与slf4j保持桥接-->  
26         <dependency>  
27             <groupId>org.apache.logging.log4j</groupId>  
28             <artifactId>log4j-slf4j-impl</artifactId>  
29             <version>2.4.1</version>  
30         </dependency>  
31         <!--web工程需要包含log4j-web，非web工程不需要-->  
32         <dependency>  
33             <groupId>org.apache.logging.log4j</groupId>  
34             <artifactId>log4j-web</artifactId>  
35             <version>2.4.1</version>  
36             <scope>runtime</scope>  
37         </dependency>  
38   
39         <!--需要使用log4j2的AsyncLogger需要包含disruptor-->  
40         <dependency>  
41             <groupId>com.lmax</groupId>  
42             <artifactId>disruptor</artifactId>  
43             <version>3.2.0</version>  
44         </dependency>  

