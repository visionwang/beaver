由单例模式引发的血案

最近去平安系面试时，遇到了个人技术领域认定的一大偶像吴大师（Cat作者），他随口问了个单例的问题，要求基于Java技术栈，给出**几种单例的方案，并给出单元测试代码，最后要求谈谈单例模式最需要注意的问题时什么**？我想想挺简单的，就是一个恶汉，一个懒汉模式，单元测试就一个判断NULL和2个Instance的比较就好。结果被大师劈头盖脸一顿数落，比如我写的懒汉单例（双锁），为什么使用volatile？还有别的更好的方式么？单元测试你不起多个线程，简单的比较有任何意义么？最后被定位为写代码不懂脑筋，仅仅就是照抄别人的成熟方案，缺少对问题关键的把握，无法进行变通。
说实话当时还有些抵触情绪，即使是被自己的偶像批评，不过回家之后好好回顾了相关的问题，发现自己对Java技术栈的理解仍然很浅薄，痛定思痛，决定好好来学习下单例模式和内部类，本文就是基于这两个问题总结，此外，祝大家新的一周工作愉快。

#单例模式
在详细介绍单例模式前，首先来谈谈**单例模式的目的和问题**，尤其是问题部分，我们常常容易忽视（感谢身边同事的提醒）。单例模式的目的非常明确，就是在当前应用中只保存指定对象的一个实例，主要目的是减少资源的消耗，各种提供服务的类会选用该模式。单例模式主要有**资源消耗的取舍，线程安全和如何防止反射或序列化破坏单例等**三个问题。主要单例的实现模式包括**最简单有效的饿汉式、最多变的懒汉式、最优的静态内部类方式**、奇特的枚举类方式和综合的登记式模式，本文主要介绍前3种，也是个人认为比较最有价值的3种。

**饿汉式**
对于一般的业务开发来说饿汉式已经足够，而且Spring框架的单例默认就是饿汉模式，绝大部分的提供各类服务的类都不是很占有内存空间，以此在项目启动时进行预加载对于系统影响不大，即使始终不被使用也没有太大的关系，其代码如下（**解决了反射和序列化破坏单例的问题**）。

	public class HungrySingleton  implements Serializable
	{
	    private HungrySingleton(){
	        if(null != instance){//防止反射破坏单例，场景为二方库调用者强行破坏单例
	            throw new IllegalOperationException();
	        }
	    }
	    public  static final HungrySingleton instance = new HungrySingleton();
	    public static HungrySingleton getInstance(){
	        return instance;
	    }
		// 防止反序列化获取多个对象，Java这儿比较奇特，因为在Serializable
		private Object readResolve() throws ObjectStreamException {
		    return instance;
		}
	}
		
	public class HungrySingletonTest {
	    @Test
	    public void testGetInstance() throws Exception {
	        Class<HungrySingleton> clazz = HungrySingleton.class;
	        Constructor<?> ctor = clazz.getDeclaredConstructor(null);
	        ctor.setAccessible(true);
	        HungrySingleton first = HungrySingleton.getInstance();
	        HungrySingleton second = (HungrySingleton)ctor.newInstance(null);//破坏单例失败
	    }
	}

**懒汉式**
懒汉式单例是考点最多的一个，虽然现实中使用次数不是很多，但掌握它有利于了解Java并发编程，常见的实现方式包括**加同步锁的懒汉式和防止指令重排优化的懒汉式**。
	
	//加同步锁的懒汉式，比较简单，但每次获取实例时都需要获得锁，对性能有一定影响
	public static synchronized LazySingleton01 getInstance(){
	    if(instance == null){
	        instance = new LazySingleton01();
	    }
	    return instance;
	}
	
	//防止指令重排优化的懒汉式
	//对常见的双锁进行了优化，对instance使用volatile修饰
	//再JAVA中，同步块外的判空操作有可能看到已存在，但不完整的实例.
	//如果使用不完整的实例则会造成系统崩溃，造成该问题的原因是由于Java内存模型的重排序机制。
	public  static volatile LazySingleton02 instance = null;
	public static LazySingleton02 getInstance(){
	    if(null  == instance){
	        synchronized (LazySingleton02.class) {
	            if(null == instance){
	                instance  = new LazySingleton02();
	            }
	        }
	    }
	    return instance;
	}
	
	//单测时一定要注意，需要使用多个线程进行测试，不然就失去了意义
	public class LazySingleton02Test {
	    @Test
	    public void getInstance() throws  InterruptedException, ExecutionException{
	        ExecutorService threadPool = Executors.newFixedThreadPool(2);
	        Future<LazySingleton02> futureA = threadPool.submit(
	                new Callable<LazySingleton02>() {
	                    @Override
	                    public LazySingleton02 call() {
	                        return LazySingleton02.getInstance();
	                    }
	                }
	        );
	        Future<LazySingleton02> futureB = threadPool.submit(
	                new Callable<LazySingleton02>() {
	                    @Override
	                    public LazySingleton02 call() {
	                        return LazySingleton02.getInstance();
	                    }
	                }
	        );
	        Assert.assertNotNull(futureA.get());
	        Assert.assertNotNull(futureB.get());
	        Assert.assertTrue(futureA.get().equals(futureB.get()));
	    }
	}


**静态内部类方式**
静态内部类这种方式是个人最不熟悉的，之前又一次面试中还被问过一个**如何扩充类**的问题，即Java中不支持多继承，如果想要复用多个类的属性如何做到？相对于将属性提取到接口中或通过自合模式复用，内部类的方式会更加优雅。对于单例同样可以借助内部类的特性优雅的处理，代码如下所示，注意看关于内部类加载的注释。

	public class LazySingleton03 {
	    private LazySingleton03(){}
	    private  static class  LazyHolder{
	            private  static final LazySingleton03 INSTANCE = new LazySingleton03();
	        }
	        //借助了静态内部类的特性，其要被引用后才会装载到内存
	        //通常的理解是，只要是当前jar中的静态属性或方法都会被加载到内存，但静态内部类却不是，它只有在第一次调用getInstance方法，产生了LazyHolder的引用，才会被真正加载。
	        //实际上也是懒加载。
	        public static final LazySingleton03 getInstance(){
	            return LazyHolder.INSTANCE;
	    }
	}
	
此外，枚举类型的单例借助其特性默认就是线程安全和防止反射破坏等行为，但并不是很适合大范围的使用。而登记器的方式实际上就是做一个Mapper其中存放所有的单例对象，需要时去获取即可，因此最推荐的仍然是**内部类**方式。

**Tip**
**重排序**
对于`instance  = new LazySingleton02()`，其实际执行情况伪代码可能如下，可以看到Java内存模型分配内存并创建对象的方式和我们预想的不太一样，这部分会在Java并发编程系列文章中继续加强学习。

	memory = allocate();//分配内存空间，C语言中应该很熟悉
	instance = memory;//这时instance会变成非空，但还未初始化
	ctor(instance);//初始化对象
	
**`synchronized`关键字**
通常基于不同的维度有如下几种用法，锁定范围越来越小，尽可能选择更小粒度的锁定范围可以获得更好的性能。
给类加锁：`synchronized(XXXX.class)`
给对象加锁：`synchronized(this)`, `public synchronized void test(){}`
给代码块加锁：`synchronized(lock){ ... }`

**Tip**
**单例模式的类和提供方法的静态类有什么区别？**提供方法的静态类不是面向对象的思想的产物，相应的其没有封装、继承、多态等特性，简单来说你无法对提供方法的静态类进行扩展。

#内部类
之前通过静态内部类方式实现单例引入了内部类这一重要概念，接下来将详细介绍内部类的相关概念和用法。**如果在一个类的内部定义一个新的类型，就将这个新的类型称为内部类**，其名称无需和文件名一致。特别的，内部类是一个编译时概念，一旦编译成功，就会成为完全不同的两个类，比如`Outer.class`和`Outer$Inner.class`。通常来说，内部类包括**静态内部类、匿名内部类**、成员内部类和局部内部类，重要性依次递减。

**静态内部类与成员内部类**
个人认为**静态内部类**最重要的一种内部类，比如常见的`ReentrantLock`中的`Sync`，`NonfairSync`、`FairSync`等一系列静态内部类。其通过`static`修饰，可以包含static数据和属性，且其无需创建外部类和内部类即可被使用。
**成员内部类**是最基本的一种内部类类型，其可以访问外部类的所有成员和方法，但不能含有static的变量和方法，因为成员内部类需要先创建外部类，之后才能创建自己，特别的，其可以通过`外部类.this.属性`的方式访问外部类同名属性，示例如下所示。

	public class InnerClass {
	    private static String staticName = "xionger";
	    private String objectName = "xiongda";
	
	    public static final class InnerClass01 {//静态内部类
	        private statißßc String name = InnerClassInterface.class.getName();
	        public static String getName() {
	            return InnerClass.staticName + "--" + name;
	        }
	    }
	    public class InnerClass02 {//成员内部类
	        public String getName() {
	            return InnerClass.this.objectName;//注意这儿获取外部类属性的形式
	        }
	    }
	}

**匿名内部类**
匿名内部类经常会被使用，比如使用线程、事件等场景，示例代码如下所示。

	public class AnonymousInnerClass {
	    public void createThread(){
	        Thread thd = new Thread(new Runnable() {
	            @Override
	            public void run() {
	                System.out.println(AnonymousInnerClass.class.getName());
	            }
	        });
	    }
	    //注意方法参数上的final关键字，由于内部类编译时生成单独的.class文件，内部类与外部类不在同一文件。
	    //内部类是通过将传入的参数先通过构造器复制到自己内部再被使用的，因此为了保持数据的一致需要添加final。
		public InnerClassInterface getInnerClass(final String prefixName){
		        return new InnerClassInterface(){
		            @Override
		            public String getName() {
		                return prefixName + "--suffixName";
		            }
		        };
		    }
		}
		
		interface InnerClassInterface {
		   String getName();
		}

**局部内部类**的使用场景实在太少就不做介绍了。

#类加载器
**类加载器的分类**
类加载器包括以下4种，加载顺序按照序号从小到大。
1.Bootstrap ClassLoader启动类加载器，负责加载`jre/lib/rt.jar`。
2.Extension ClassLoader扩展类加载器，负责加载扩展功能Jar包，包括`jre/lib/*.jar`或`ext目录`下的jar包。
3.App ClassLoader应用类加载器,负责加载classpath中指定的jar包.
4.Custom ClassLoader自定义类加载器，如tomcat根据j2ee规范自行实现ClassLoader。

**类的加载过程**
Java加载类的过程主要包含如下3步。
a.**加载**：查找并加载类的二进制文件。
b.**链接**（包含3个子步骤）：**验证**，确保加载类的正确性，防止恶意代码；**准备**，为类的静态变量分配内存空间并赋默认值；**解析**，将类的符号引用转化为直接引用。
c.**初始化**：为类的静态变量赋予初始值。
**Tip**
**类初始化的条件**
1.创建类的实例，new对象或者反射创建对象。
2.访问类或接口的静态变量时或静态方法时。
3.初始化一个类的子类时会先初始化父类。
4.JVM启动时明确指定的启动类。
类加载器这部分的水很深，会在之后的文章专门用一篇文章进行解析，之后一段时间，将主要进行工作2年多来项目的回顾总结。

**参考资料**
推荐：海子大神的JAVA技术栈相关文章：http://www.cnblogs.com/dolphin0520/
[单例模式，你知道的和你所不一定知道的一切](http://blog.csdn.net/u011546655/article/details/50134057)
[如何防止JAVA反射对单例类的攻击？](https://www.cnblogs.com/lthIU/p/6240128.html)
[Java设计模式（一）：单例模式，防止反射和反序列化漏洞](http://blog.csdn.net/hardwin/article/details/51477359)
[java 单例模式通过内部静态类的方式？](https://www.zhihu.com/question/35454510)
[java 内部类如何访问外部类的同名属性](http://blog.csdn.net/u013655410/article/details/38040223)
[Java内部类的使用小结](http://blog.51cto.com/android/384844)
[Java类加载器总结](http://blog.csdn.net/gjanyanlig/article/details/6818655/)
[类加载原理分析&动态加载Jar/Dex](http://www.jianshu.com/p/0b1dba1a1e95)
[Java高新技术第一篇：类加载器详解](http://blog.csdn.net/jiangwei0910410003/article/details/17733153)


