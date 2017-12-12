首先要解释一下什么是延迟加载，延迟加载就是等到真真使用的时候才去创建实例，不用时不要去创建。
 
**从速度和反应时间角度来讲，非延迟加载（又称饿汉式）好；从资源利用效率上说，延迟加载（又称懒汉式）好。**
 
下面看看几种常见的单例的设计方式：
 
第一种：**非延迟加载单例类**

	public class Singleton {  
	 private Singleton() {}  
	 private static final Singleton instance = new Singleton();  
	 public static Singleton getInstance() {  
	  return instance;  
	 }  
	}  
 
第二种：**同步延迟加载**

	public class Singleton {  
	 private static Singleton instance = null;  
	 private Singleton() {}  
	 public static synchronized Singleton getInstance() {  
	  if (instance == null) {  
	   instance = new Singleton();  
	  }  
	  return instance;  
	 }  
	}  

第三种：**双重检测同步延迟加载** 
为处理原版非延迟加载方式瓶颈问题，我们需要对 instance 进行第二次检查，目的是避开过多的同步（因为这里的同步只需在第一次创建实例时才同步，一旦创建成功，以后获取实例时就不需要同获取锁了），但在Java中行不通，因为同步块外面的if (instance == null)可能看到已存在，但不完整的实例。JDK5.0以后版本若instance为**volatile**则可行：


	public class Singleton {  
	 private volatile static Singleton instance = null;  
	 private Singleton() {}  
	 public static Singleton getInstance() {  
	  if (instance == null) {  
	   synchronized (Singleton.class) {// 1  
	    if (instance == null) {// 2  
	     instance = new Singleton();// 3  
	    }  
	   }  
	  }  
	  return instance;  
	 }  
	}  

双重检测锁定失败的问题并不归咎于 JVM 中的实现 bug，而是归咎于 Java 平台内存模型。**内存模型允许所谓的“无序写入”，这也是失败的一个主要原因。**


无序写入：
为解释该问题，需要重新考察上述清单中的 //3 行。此行代码创建了一个 Singleton 对象并初始化变量 instance 来引用此对象。这行代码的问题是：在 Singleton 构造函数体执行之前，变量 instance 可能成为非 null 的，即赋值语句在对象实例化之前调用，此时别的线程得到的是一个还会初始化的对象，这样会导致系统崩溃。
什么？这一说法可能让您始料未及，但事实确实如此。在解释这个现象如何发生前，请先暂时接受这一事实，我们先来考察一下双重检查锁定是如何被破坏的。假设代码执行以下事件序列：

1、线程 1 进入 getInstance() 方法。
2、由于 instance 为 null，线程 1 在 //1 处进入 synchronized 块。 
3、线程 1 前进到 //3 处，但在构造函数执行之前，使实例成为非 null。 
4、线程 1 被线程 2 预占。
5、线程 2 检查实例是否为 null。因为实例不为 null，线程 2 将 instance 引用返回给一个构造完整但部分初始化了的 Singleton 对象。 
6、线程 2 被线程 1 预占。
7、线程 1 通过运行 Singleton 对象的构造函数并将引用返回给它，来完成对该对象的初始化。
 
为展示此事件的发生情况，假设代码行 instance =new Singleton(); 执行了下列伪代码：
mem = allocate();             //为单例对象分配内存空间.
instance = mem;               //注意，instance 引用现在是非空，但还未初始化
ctorSingleton(instance);    //为单例对象通过instance调用构造函数

这段伪代码不仅是可能的，而且是一些 JIT 编译器上真实发生的。执行的顺序是颠倒的，但鉴于当前的内存模型，这也是允许发生的。JIT 编译器的这一行为使双重检查锁定的问题只不过是一次学术实践而已。
 
 
如果真像这篇文章：http://dev.csdn.net/author/axman/4c46d233b388419e9d8b025a3c507b17.html所说那样的话，1.2或以后的版本就不会有问题了，但这个规则是JMM的规范吗？谁能够确认一下。
确实,在JAVA2(以jdk1.2开始)以前对于实例字段是直接在主储区读写的.所以当一个线程对resource进行分配空间,
初始化和调用构造方法时,可能在其它线程中分配空间动作可见了,而初始化和调用构造方法还没有完成.
但是从JAVA2以后,JMM发生了根本的改变,分配空间,初始化,调用构造方法只会在线程的工作存储区完成,在没有
向主存储区复制赋值时,其它线程绝对不可能见到这个过程.而这个字段复制到主存区的过程,更不会有分配空间后
没有初始化或没有调用构造方法的可能.在JAVA中,一切都是按引用的值复制的.向主存储区同步其实就是把线程工作
存储区的这个已经构造好的对象有压缩堆地址值COPY给主存储区的那个变量.这个过程对于其它线程,要么是resource
为null,要么是完整的对象.绝对不会把一个已经分配空间却没有构造好的对象让其它线程可见.
 
另一篇详细分析文章：http://www.iteye.com/topic/260515
 
第四种：使用ThreadLocal修复双重检测
 
借助于ThreadLocal，将临界资源（需要同步的资源）线程局部化，具体到本例就是将双重检测的第一层检测条件 if (instance == null) 转换为了线程局部范围内来作。这里的ThreadLocal也只是用作标示而已，用来标示每个线程是否已访问过，如果访问过，则不再需要走同步块，这样就提高了一定的效率。但是ThreadLocal在1.4以前的版本都较慢，但这与volatile相比却是安全的。
 

	public class Singleton {  
	 private static final ThreadLocal perThreadInstance = new ThreadLocal();  
	 private static Singleton singleton ;  
	 private Singleton() {}  
	   
	 public static Singleton  getInstance() {  
	  if (perThreadInstance.get() == null){  
	   // 每个线程第一次都会调用  
	   createInstance();  
	  }  
	  return singleton;  
	 }  
	  
	 private static  final void createInstance() {  
	  synchronized (Singleton.class) {  
	   if (singleton == null){  
	    singleton = new Singleton();  
	   }  
	  }  
	  perThreadInstance.set(perThreadInstance);  
	 }  
	}  

**第五种：使用内部类实现延迟加载**(静态内部类)
为了做到真真的延迟加载，双重检测在Java中是行不通的，所以只能借助于另一类的类加载加延迟加载：

	public class Singleton {  
	 private Singleton() {}  
	 public static class Holder {  
	  // 这里的私有没有什么意义  
	  /* private */static Singleton instance = new Singleton();  
	 }  
	 public static Singleton getInstance() {  
	  // 外围类能直接访问内部类（不管是否是静态的）的私有变量  
	  return Holder.instance;  
	 }  
	}  
 
单例测试
下面是测试单例的框架，采用了类加载器与反射。
注，为了测试单便是否为真真的单例，我自己写了一个类加载器，且其父加载器设置为根加载器，这样确保Singleton由MyClassLoader加载，如果不设置为根加载器为父加载器，则默认为系统加载器，则Singleton会由系统加载器去加载，但这样我们无法卸载类加载器，如果加载Singleton的类加载器卸载不掉的话，那么第二次就不能重新加载Singleton的Class了，这样Class不能得加载则最终导致Singleton类中的静态变量重新初始化，这样就无法测试了。
下面测试类延迟加载的结果是可行的，同样也可用于其他单例的测试：


	public class Singleton {  
	 private Singleton() {}  
	  
	 public static class Holder {  
	  // 这里的私有没有什么意义  
	  /* private */static Singleton instance = new Singleton();  
	 }  
	  
	 public static Singleton getInstance() {  
	  // 外围类能直接访问内部类（不管是否是静态的）的私有变量  
	  return Holder.instance;  
	 }  
	}  
  
class CreateThread extends Thread {  
 Object singleton;  
 ClassLoader cl;  
  
 public CreateThread(ClassLoader cl) {  
  this.cl = cl;  
 }  
  
 public void run() {  
  Class c;  
  try {  
   c = cl.loadClass("Singleton");  
   // 当两个不同命名空间内的类相互不可见时，可采用反射机制来访问对方实例的属性和方法  
   Method m = c.getMethod("getInstance", new Class[] {});  
   // 调用静态方法时，传递的第一个参数为class对象  
   singleton = m.invoke(c, new Object[] {});  
   c = null;  
   cl = null;  
  } catch (Exception e) {  
   e.printStackTrace();  
  }  
 }  
}  
  
class MyClassLoader extends ClassLoader {  
 private String loadPath;  
 MyClassLoader(ClassLoader cl) {  
  super(cl);  
 }  
 public void setPath(String path) {  
  this.loadPath = path;  
 }  
 protected Class findClass(String className) throws ClassNotFoundException {  
  FileInputStream fis = null;  
  byte[] data = null;  
  ByteArrayOutputStream baos = null;  
  
  try {  
   fis = new FileInputStream(new File(loadPath  
     + className.replaceAll("\\.", "\\\\") + ".class"));  
   baos = new ByteArrayOutputStream();  
   int tmpByte = 0;  
   while ((tmpByte = fis.read()) != -1) {  
    baos.write(tmpByte);  
   }  
   data = baos.toByteArray();  
  } catch (IOException e) {  
   throw new ClassNotFoundException("class is not found:" + className,  
     e);  
  } finally {  
   try {  
    if (fis != null) {  
     fis.close();  
    }  
    if (fis != null) {  
     baos.close();  
    }  
  
   } catch (Exception e) {  
    e.printStackTrace();  
   }  
  }  
  return defineClass(className, data, 0, data.length);  
 }  
}  
  
class SingleTest {  
 public static void main(String[] args) throws Exception {  
  while (true) {  
   // 不能让系统加载器直接或间接的成为父加载器  
   MyClassLoader loader = new MyClassLoader(null);  
   loader  
     .setPath("D:\\HW\\XCALLC16B125SPC003_js\\uniportal\\service\\AAA\\bin\\");  
   CreateThread ct1 = new CreateThread(loader);  
   CreateThread ct2 = new CreateThread(loader);  
   ct1.start();  
   ct2.start();  
   ct1.join();  
   ct2.join();  
   if (ct1.singleton != ct2.singleton) {  
    System.out.println(ct1.singleton + " " + ct2.singleton);  
   }  
   // System.out.println(ct1.singleton + " " + ct2.singleton);  
   ct1.singleton = null;  
   ct2.singleton = null;  
   Thread.yield();  
  }  
 }  
}  