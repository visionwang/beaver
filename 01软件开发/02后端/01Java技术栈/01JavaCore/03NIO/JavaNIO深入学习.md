NIO是Jdk中非常重要的一个组成部分，基于它的Netty开源框架可以很方便的开**发高性能、高可靠性的网络服务器**和客户端程序。本文将就其核心基础类型**`Channel`, `Buffer`, `Selector`**进行详细介绍，之后将介绍内存映射文件，`Scatter/Gatter`等扩展知识，最后将对Linux的5种IO模型进行剖析。
![](http://i.imgur.com/bmG0Qrb.png)

# 基础概念 #
** Java NIO(non-blocking IO非阻塞IO)**是jdk1.4后提供的新IO API，为所有基础类型都提供**类缓存**支持。其基础类型**`Channel`**定义了一个新的I/O接口，**支持锁和访问内存映射文件**，提供多路非阻塞式的高伸缩性网络I/O，其与传统IO的区别如下所示。
1. **Channels and Buffers（通道和缓冲区）**：标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。
2. **Asynchronous IO（异步IO）**：Java NIO可以让你异步的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。
3. **Selectors（选择器）**：Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

**核心理解**
NIO中的`SocketChanne`l等网络`Channel`充分利用了操作系统内核对象的调度，**只使用少量的几个线程来处理和客户端的所有通信，消除了无谓的线程上下文切换**，最大限度的提高了网络通信的性能。可以看到，NIO主要关注文件IO和基于传输层的网络传输IO。对于.NET程序员来说，Windows的**完成端口模型**和NIO有一些相似之处，具体的IO模型分析请见本文最后部分的**五种IO模型**。
**使用场景**
对于比较时髦的物联网公司，需要收集设备的数据，通常需要自定义相关协议并基于传输层通信。

## Channel ##
**`Channel`通道**和传统IO中的Stream流类似，但它是双向的，而流式单向的，如`InputStream`，`OutputStream`等，Channel的常见实现如下所示。
**FileChannel**： 从文件中读写数据。
**DatagramChannel**： 能通过UDP读写网络中的数据。
**SocketChannel**： 能通过TCP读写网络中的数据。
**ServerSocketChannel**：可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。
接下里通过一个TCP客户端的NIO实现来熟悉`Channel`的应用。

	public class TCPClient {
		public void start() {
			ByteBuffer buffer = ByteBuffer.allocate(1024);
			try (SocketChannel channel = SocketChannel.open()) {
				channel.configureBlocking(false);// 在非阻塞式信道上调用一个方法总是会立即返回
				channel.connect(new InetSocketAddress("127.0.0.1", 8080));
	
				if (channel.finishConnect()) {
					int i = 0;
					while (true) {
						TimeUnit.SECONDS.sleep(1);
						String info = "I'm  " + (i++) + "-th information from client";
						buffer.clear();
						buffer.put(info.getBytes());
						buffer.flip();
						while (buffer.hasRemaining()) {
							// 问题，这样一个个直接的输出，速度快么？
							System.out.println(buffer);
							channel.write(buffer);
						}
					}
				}
			} catch (Throwable ex) {
				ex.printStackTrace();
			}
		}
	}
比对一下传统的IO

	try (RandomAccessFile aFile = new RandomAccessFile("src/test/resources/test.xml", "r")) {
		// 1.读数据
		byte[] buffer = new byte[1024];
		StringBuilder requestString = new StringBuilder();
		int bytesRead = 0;
		while (-1 != (bytesRead = aFile.read(buffer))) {
			requestString.append(new String(buffer), 0, bytesRead);
		}
	}

## Buffer ##
NIO中主要的**Buffer缓存**实现包括ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBuffer, LongBuffer, ShortBuffer等，分别对应相应的基础类型，Buffer的主要作用请见下图。
![](http://i.imgur.com/OmSdy0e.png)
Buffer的数据结构包括一个连续数组，表示缓冲区数组的总长度的capacity，记录下一个要操作的数据元素的位置position，以及limit、mark等字段，接下来请见一个最简单的NIO示例（**其中使用到了文件锁和CharBuffer**）。

	public static void readFile() {
		long timeBegin = System.currentTimeMillis();
		try (RandomAccessFile aFile = new RandomAccessFile("src/nio.txt", "rw")) {
			FileChannel fileChannel = aFile.getChannel();
			ByteBuffer buf = ByteBuffer.allocate(1024);
			Charset charset = Charset.forName("UTF-8");

			int position = 0;
			while (true) {
				// 文件锁，锁一段区域，shard为true表示共享锁
				FileLock fileLock = fileChannel.lock(position, 1024, true);
				int bytesRead = fileChannel.read(buf);
				fileLock.release();
				if (-1 == bytesRead)
					break;
				position = position + bytesRead;
				buf.flip();
				//将ByteBuffer转化为CharBuffer
				System.out.println(charset.decode(buf));
				buf.clear();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		long timeEnd = System.currentTimeMillis();
		System.out.println("Read time: " + (timeEnd - timeBegin) + "ms");
	}

## Selector ##
Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中，要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，如新连接进来，数据接收等。仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好，接下来通过TCP服务端的NIO实现来熟悉`Selector`的应用。

	public class TCPServer {
		private static final int BUF_SIZE = 1024;
		private static final int PORT = 8080;
		private static final int TIMEOUT = 3000;
		private TCPServer() {
		}
		public static final TCPServer instance = new TCPServer();
		public static TCPServer getInstance() {
			return instance;
		}
	
		/*
		 * 关键类
		 */
		public void selector() {
			try (Selector selector = Selector.open(); ServerSocketChannel ssChannel = ServerSocketChannel.open()) {
				ssChannel.bind(new InetSocketAddress(PORT));
				ssChannel.configureBlocking(false);
				ssChannel.register(selector, SelectionKey.OP_ACCEPT);
	
				while (true) {
					if (0 == selector.select(TIMEOUT)) {
						System.out.println("==");
						continue;
					}
					// 使用 for (SelectionKey key : selector.selectedKeys()) 方式无法移除对象
					// Iterator对象的remove方法是迭代过程中删除元素的唯一方法
					Iterator<SelectionKey> selectKeys = selector.selectedKeys().iterator();
					while (selectKeys.hasNext()) {
						SelectionKey key = selectKeys.next();
						//分别处理接受、连接、读、写4中状态
						if (key.isAcceptable()) {
							handleAccept(key);
						}
						if (key.isReadable()) {
							handleRead(key);
						}
						if (key.isWritable() && key.isValid()) {
							handleWrite(key);
						}
						if (key.isConnectable()) {
							System.out.println("isConnectable = true");
						}
						selectKeys.remove();
					}
				}
			} catch (Throwable ex) {
				ex.printStackTrace();
			}
		}
	
		public void handleAccept(SelectionKey key) throws IOException {
			ServerSocketChannel ssChannel = (ServerSocketChannel) key.channel();// 获取ServerSocket的Channel
			SocketChannel sc = ssChannel.accept();// 监听新进来的链接
			sc.configureBlocking(false);// 设置client链接为非阻塞
			sc.register(key.selector(), SelectionKey.OP_READ, ByteBuffer.allocate(BUF_SIZE));// 将channel注册Selector
		}
	
		public void handleRead(SelectionKey key) throws IOException {
			SocketChannel channel = (SocketChannel) key.channel();// 获取链接client的channel
			ByteBuffer buf = (ByteBuffer) key.attachment();// 获取附加在key上的数据
			long bytesRead = channel.read(buf);// 读channel上数据到buf?这部分表达的是否正确？
			while (bytesRead > 0) {
				buf.flip();
				while (buf.hasRemaining()) {
					System.out.println(buf.getChar());
				}
				System.out.println();
				buf.clear();
				bytesRead = channel.read(buf);
			}
			if (-1 == bytesRead) {
				channel.close();
			}
		}
	
		public void handleWrite(SelectionKey key) throws IOException {
			ByteBuffer buf = (ByteBuffer) key.attachment();// 获取buffer对象
			buf.flip();
			SocketChannel channel = (SocketChannel) key.channel();
			while (buf.hasRemaining()) {
				channel.write(buf);
			}
			buf.compact();
		}
	}


# 进阶概念 #
**Java IO与NIO的区别**主要包括如下的3个方面。
面向流和面向缓冲：Java IO是面向流的，其意味着在读取所有字节前，数据未被缓存到任何地方，其不能前后移动流中的数据。而NIO是面向缓存的，可以方便在缓存区的移动数据。
阻塞和非阻塞IO：java IO是完全阻塞的，在数据读取或写入完成前，该线程一直被占用。而NIO的非阻塞模式，当线程获取不到数据时，可以被释放用于其他工作。
选择器Selector：其允许一个单独的线程来监视多个出入通道，利于线程的高效利用。

### 内存映射文件 ###
对于内存映射文件，大家应该不会陌生，大学里学习windows网络编程时就终点介绍过该概念。在JAVA中，一般用`BufferedReader`，`BufferedInputStream`的来处理大文件。而当文件超大时推荐使用`MappedByteBuffer`，其是NIO引入的文件内存映射方案，读写性能极高。一般操作系统的内存包括物理内存和虚拟内存。其中虚拟内存一般使用的是页面映像文件，即硬盘中的某个特殊的文件。操作系统负责该页面文件内容的读写，这个过程叫**”页面中断/切换”**。`MappedByteBuffer`与其类似，可以看做一个超级大的`ByteBuffer`，示例如下所示。

	public class MappedByteBufferDemo {
		public static void commonReadBigFile() {
			try (RandomAccessFile aFile = new RandomAccessFile("D:/svn.rar", "rw");
					FileChannel fileChannel = aFile.getChannel()) {
	
				long timeBegin = System.currentTimeMillis();
				ByteBuffer buff = ByteBuffer.allocate((int) aFile.length());
				buff.clear();
				fileChannel.read(buff);
				long timeEnd = System.currentTimeMillis();
				System.out.println("Read time: " + (timeEnd - timeBegin) + "ms");
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	
		public static void highPerformanceReadBigFile() {
			try (RandomAccessFile aFile = new RandomAccessFile("D:/svn.rar", "rw");
					FileChannel fileChannel = aFile.getChannel()) {
	
				long timeBegin = System.currentTimeMillis();
				MappedByteBuffer mappedBuffer = fileChannel.map(FileChannel.MapMode.READ_ONLY, 0, aFile.length());
				long timeEnd = System.currentTimeMillis();
				System.out.println("Read time: " + (timeEnd - timeBegin) + "ms");
			} catch (IOException e) {
				e.printStackTrace();
			}
		}
	}

通过一个1G的文件做单元测试测试，普通的Buffer读需要1373ms，而内存映射文件只需要1ms，这个测试可能不太准确，但差异已经无比明显。当然使用内存映射文件也存在问题，就是它只要GC时才能被回收，会占用大量内存资源，我所了解的一个使用场景涉及复杂的**推荐算法因素和规则的加载**（国际机票组合选择），之后进行运算。

### 其他 ###
**Pipe**：之前介绍的`Channel`虽然既可以用作读，也可以用作写，但它每次只能支持一种方式的通讯，即单工/半双工的。而Pipe是全双工的，其内部包含两个Channel，一个是SinkChannel用于写入数据，一个Source通道用于读取数据。
**Scatter/Gatter**：前者在读操作时将数据从一个Channel中读入到多个Buffer，而后者用于将多个Buffer写入一个Channel，常见的使用场景为将消息头和消息体一起写入到Channel中。
**TransferFrom & TransferTo**：传输方法用于通道之间的通信，比如FileChannel的`transferFrom()`方法可以将数据从源通道传输到FileChannel中，而`transferTo()`方法将数据从FileChannel传输到其他的channel中。
Java的Path接口是Java NIO2 的一部分，是对Java6 和Java7的 NIO的更新。Java的Path接口在Java7 中被添加到Java NIO，位于java.nio.file包中， 其全路径是java.nio.file.Path。一个Path实例代表了一个文件系统中的路径。一个路径可以指向一个文件或者一个文件夹。一个路径可以是绝对路径或者是相对路径。绝对路径是从根路径开始的全路径，相对路径是一个相对其他路径的文件或文件夹路径。相对路径可能会造成一点混乱，但是不要担心，在本文章中，我会详细解释相对路径的。
**DatagramChannel**：用于基于UDP协议，用于发送和接受数据包，常用于QQ语音、视频等通讯质量要求不高的场景。

### NIO2 ###
Jdk7对NIO做了一些改进，包括对AIO的支持，以及增强型的文件操作类。过去的`java.io.File`访问文件系统时，无法利用特定文件系统的特性，且性能不高，因此引入了Path，Paths，Files等。对于AIO，其提供了`AsynchronousChannel`，`AsynchronousFileChannel`等与NIO响应的类型，大大简化了异步IO操作，比如在过去我们需要自己管理线程池来进行Callable的调用返回Future<?>，而现在省去了该步骤，请见下面的示例（该场景下直接使用NIO更合适，AIO适合更加复杂的场景）。

	public static void readFile() {
		Path filePath = Paths.get("src/nio.txt");

		long timeBegin = System.currentTimeMillis();
		try (AsynchronousFileChannel afc = AsynchronousFileChannel.open(filePath)) {
			ByteBuffer buf = ByteBuffer.allocate(1024);
			Charset charset = Charset.forName("UTF-8");
			
			int position = 0;
			for (;;) {
				Future<Integer> futureResult = afc.read(buf, position);
				int result = futureResult.get();

				if (-1 == result)
					break;

				position = position + result;
				buf.flip();
				System.out.println(charset.decode(buf));
				buf.clear();
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
		long timeEnd = System.currentTimeMillis();
		System.out.println("Read time: " + (timeEnd - timeBegin) + "ms");
	}

# Linux的五种IO模型 #
通常来说，网络IO模型大致包括如下2大类，其中同步模型包括4种。
**1. 同步模型**
a.阻塞IO：简称bio，linux中默认所有的socket都是blocking，其特点是进程会一直阻塞，直到数据拷贝完成。
b.非阻塞IO：简称nio，与本文的NIO(New IO)不是一个概念，其特点是进程会反复调用IO函数，并马上返回，但在数据拷贝的过程中，进行仍然是阻塞的。
c.多路复用IO：由于NIO需要大量的轮训会消耗大量CPU时间，而如果这件事有操作内核来通知就好了，这是就会用到Linux的`select`、`poll`、`epoll`函数。比如`select`，其可以对多个IO端口进行监控。
d.信号驱动IO：允许Socket进行信号驱动IO,并安装一个信号处理函数，进程继续运行并不阻塞。当数据准备好时，进程会收到一个SIGIO信号，可以在信号处理函数中调用I/O操作函数处理数据。
**2. 异步模型(aio)：**相对于同步IO，异步IO不是顺序执行。其特点是IO的两个阶段，进程都是非阻塞的。

**小结**
以上对Linux的IO模型进行了介绍，对应到java程序，**那么io包中的操作其实就是阻塞IO的方式，而nio包中的Channel的类型就是非阻塞IO的方式，`Selector`提供了多路复用的方式，而AsynchronousChannel提供了异步IO的方式。**此外，这几种IO模型并没有谁优谁劣的说法，需要结合具体场景进行分析，通常来说，高并发的程序会使用同步非阻塞的方式。如果感觉解释的不是很完善，请见博文[聊聊Linux 五种IO模型](http://www.jianshu.com/p/486b0965c296)，该博主通过和女友等餐的过程对5种IO的过程做了很好的诠释，大赞。
Tip:
之后的文章中将对**Netty，mina**框架进行介绍，其是NIO的封装库。

**参考资料**
[netty官网](http://netty.io/)
[Java NIO系列教程](http://ifeve.com/overview/)
[攻破JAVA NIO技术壁垒](http://www.importnew.com/19816.html)
[完成端口(Completion Port)详解](http://www.cnblogs.com/lancidie/archive/2011/12/19/2293773.html)
[java nio框架netty 与tomcat的关系 ](http://www.iteye.com/problems/92400)
[Tomcat7中NIO处理分析（一）](http://tyrion.iteye.com/blog/2256896)
[NIO文档](http://tutorials.jenkov.com/java-nio/index.html)
[高性能IO模型浅析](http://www.cnblogs.com/fanzhidongyzby/p/4098546.html)
[聊聊同步、异步、阻塞与非阻塞](https://my.oschina.net/xianggao/blog/661085)