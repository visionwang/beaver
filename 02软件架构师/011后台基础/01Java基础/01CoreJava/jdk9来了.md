    2009年4月20日，甲骨文以74亿美元的价格收购SUN公司，取得java的版权，业界传闻说这对Java程序员是个坏消息（其实恰恰相反）；

    2010年11月，由于甲骨文对Java社区的不友善，因此Apache扬言将退出JCP；

    2011年7月28日，甲骨文发布Java SE 7；

    2014年3月18日，甲骨文发表Java SE 8；

    2017年7月，甲骨文发表Java SE 9。

##modularity System 模块系统
Java 9中主要的变化是已经实现的模块化系统。
Modularity提供了类似于OSGI框架的功能，模块之间存在相互的依赖关系，可以导出一个公共的API，并且隐藏实现的细节，Java提供该功能的主要的动机在于，减少内存的开销，在JVM启动的时候，至少会有30～60MB的内存加载，主要原因是JVM需要加载rt.jar，不管其中的类是否被classloader加载，第一步整个jar都会被JVM加载到内存当中去，模块化可以根据模块的需要加载程序运行需要的class。
在引入了模块系统之后，JDK 被重新组织成 94 个模块。Java 应用可以通过新增的 jlink 工具，创建出只包含所依赖的 JDK 模块的自定义运行时镜像。这样可以极大的减少 Java 运行时环境的大小。使得JDK可以在更小的设备中使用。采用模块化系统的应用程序只需要这些应用程序所需的那部分JDK模块，而非是整个JDK框架了。

##HTTP/2
JDK9之前提供HttpURLConnection API来实现Http访问功能，但是这个类基本很少使用，一般都会选择Apache的Http Client，此次在Java 9的版本中引入了一个新的package:java.net.http，里面提供了对Http访问很好的支持，不仅支持Http1.1而且还支持HTTP2（什么是HTTP2？请参见HTTP2的时代来了...），以及WebSocket，据说性能特别好。

![](https://upload-images.jianshu.io/upload_images/9481801-d4f9b908249d0c55?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

##JShell
用过Python的童鞋都知道，Python 中的读取-求值-打印循环（ Read-Evaluation-Print Loop ）很方便。它的目的在于以即时结果和反馈的形式。
java9引入了jshell这个交互性工具，让Java也可以像脚本语言一样来运行，可以从控制台启动 jshell ，在 jshell 中直接输入表达式并查看其执行结果。当需要测试一个方法的运行效果，或是快速的对表达式进行求值时，jshell 都非常实用。

##不可变集合工厂方法
Java 9增加了List.of()、Set.of()、Map.of()和Map.ofEntries()等工厂方法来创建不可变集合。

    List strs = List.of("Hello", "World");
    List strs List.of(1, 2, 3);
    Set strs = Set.of("Hello", "World");
    Set ints = Set.of(1, 2, 3);
    Map maps = Map.of("Hello", 1, "World", 2);

##私有接口方法
Java 8 为我们提供了接口的默认方法和静态方法，接口也可以包含行为，而不仅仅是方法定义。
默认方法和静态方法可以共享接口中的私有方法，因此避免了代码冗余，这也使代码更加清晰。如果私有方法是静态的，那这个方法就属于这个接口的。并且没有静态的私有方法只能被在接口中的实例调用。

interface InterfaceWithPrivateMethods {
    private static String staticPrivate() {
        return "static private";
    }
    private String instancePrivate() {
        return "instance private";
    }
    default void check() {    
        String result = staticPrivate();
        InterfaceWithPrivateMethods pvt = new InterfaceWithPrivateMethods() {
            // anonymous class 匿名类
        };
        result = pvt.instancePrivate();
    }
}

##HTML5风格的Java帮助文档
Java 8之前的版本生成的Java帮助文档是在HTML 4中。在Java 9中，Javadoc 的输出现在符合兼容 HTML5 标准。现在HTML 4是默认的输出标记语言，但是在之后发布的JDK中，HTML 5将会是默认的输出标记语言。

##多版本兼容 JAR
当一个新版本的 Java 出现的时候，你的库用户要花费很长时间才会切换到这个新的版本。这就意味着库要去向后兼容你想要支持的最老的 Java 版本 (许多情况下就是 Java 6 或者 7)。这实际上意味着未来的很长一段时间，你都不能在库中运用 Java 9 所提供的新特性。幸运的是，多版本兼容 JAR 功能能让你创建仅在特定版本的 Java 环境中运行库程序时选择使用的 class 版本：
<mark>感觉这个不错</mark>
multirelease.jar

├── META-INF
│   └── versions
│       └── 9
│           └── multirelease
│               └── Helper.class
├── multirelease
├── Helper.class
└── Main.class
在上述场景中， multirelease.jar 可以在 Java 9 中使用, 不过 Helper 这个类使用的不是顶层的 multirelease.Helper 这个 class, 而是处在“META-INF/versions/9”下面的这个。这是特别为 Java 9 准备的 class 版本，可以运用 Java 9 所提供的特性和库。同时，在早期的 Java 诸版本中使用这个 JAR 也是能运行的，因为较老版本的 Java 只会看到顶层的这个 Helper 类。

##统一 JVM 日志
Java 9 中 ，JVM 有了统一的日志记录系统，可以使用新的命令行选项-Xlog 来控制 JVM 上 所有组件的日志记录。该日志记录系统可以设置输出的日志消息的标签、级别、修饰符和输出目标等。

##java9的垃圾收集机制
Java 9 移除了在 Java 8 中 被废弃的垃圾回收器配置组合，同时把G1设为默认的垃圾回收器实现。替代了之前默认使用的Parallel GC，对于这个改变，evens的评论是酱紫的：这项变更是很重要的，因为相对于Parallel来说，G1会在应用线程上做更多的事情，而Parallel几乎没有在应用线程上做任何事情，它基本上完全依赖GC线程完成所有的内存管理。这意味着切换到G1将会为应用线程带来额外的工作，从而直接影响到应用的性能

##I/O 流新特性
java.io.InputStream 中增加了新的方法来读取和复制 InputStream 中包含的数据。
    readAllBytes：读取 InputStream 中的所有剩余字节。
    readNBytes： 从 InputStream 中读取指定数量的字节到数组中。
    transferTo：读取 InputStream 中的全部字节并写入到指定的 OutputStream 中 。

参考资料
1.https://blog.csdn.net/mxw2552261/article/details/79080678
