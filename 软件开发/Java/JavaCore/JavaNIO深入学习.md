NIO是Jdk中非常重要的一个组成部分，基于它的Netty开源框架可以很方便的开发高性能、高可靠性的网络服务器和客户端程序。本文将就其核心基础类型`Channel`, `Buffer`, `Selector`进行详细介绍，此外将简要介绍Netty框架。
![](http://i.imgur.com/bmG0Qrb.png)

# 概念 #
** Java NIO(non-blocking IO非阻塞IO)**是jdk1.4后提供的新IO API，为所有基础类型都提供**类缓存**支持。其基础类型**`Channel`**定义了一个新的I/O接口，**支持锁和访问内存映射文件**，提供多路非阻塞式的高伸缩性网络I/O，其与传统IO的区别如下所示。
1. **Channels and Buffers（通道和缓冲区）**：标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。
2. **Asynchronous IO（异步IO）**：Java NIO可以让你异步的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。
3. **Selectors（选择器）**：Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

Tip:
对于.NET程序员来说，Windows的**完成端口模型**和这部分很类似。

# 详细分析 #
## Channel ##
`Channel`通道和传统IO中的Stream流类似，但它是双向的，而流式单向的，如`InputStream`，`OutputStream`等，Channel的常见实现如下所示。
**FileChannel**： 从文件中读写数据。
**DatagramChannel**： 能通过UDP读写网络中的数据。
**SocketChannel**： 能通过TCP读写网络中的数据。
**ServerSocketChannel**：可以监听新进来的TCP连接，像Web服务器那样。对每一个新进来的连接都会创建一个SocketChannel。
接下里通过一个TCP客户端的NIO实现来熟悉`Channel`的应用。



## Buffer ##
NIO中主要的Buffer实现包括ByteBuffer, CharBuffer, DoubleBuffer, FloatBuffer, IntBuffer, LongBuffer, ShortBuffer等，分别对应相应的基础类型，Buffer的主要作用请见下图。
![](http://i.imgur.com/OmSdy0e.png)
Buffer的数据结构包括一个连续数组，表示缓冲区数组的总长度的capacity，记录下一个要操作的数据元素的位置position，以及limit、mark等字段，接下来请见一个最简单的NIO示例。

	public static void readFile() {
		try (RandomAccessFile aFile = new RandomAccessFile("src/nio.txt", "rw")) {
			FileChannel fileChannel = aFile.getChannel();
			// 1.分配空间
			ByteBuffer buf = ByteBuffer.allocate(1024);
			// 2.写入数据到Buffer，还可以通过buf.put()的方式
			int bytesRead = fileChannel.read(buf);
			System.out.println(bytesRead);

			while (-1 != bytesRead) {
				// 3.flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。
				buf.flip();
				while (buf.hasRemaining()) {
					// 4.从Buffer中读取数据，也可以用channel.write(buf)将Buffer中数据读取到channel
					System.out.println((char) buf.get());
				}
				// 5.调用clear或compact方法。
				// 如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。
				// 如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。
				buf.compact();
				bytesRead = fileChannel.read(buf);
				//此外，通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position
			}
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

## Selector ##
Selector允许单线程处理多个 Channel。如果你的应用打开了多个连接（通道），但每个连接的流量都很低，使用Selector就会很方便。例如，在一个聊天服务器中，要使用Selector，得向Selector注册Channel，然后调用它的select()方法。这个方法会一直阻塞到某个注册的通道有事件就绪。一旦这个方法返回，线程就可以处理这些事件，如新连接进来，数据接收等。


Selector（选择器）是Java NIO中能够检测一到多个NIO通道，并能够知晓通道是否为诸如读写事件做好准备的组件。这样，一个单独的线程可以管理多个channel，从而管理多个网络连接。

其实就是windows的完成端口模型的形式IO

仅用单个线程来处理多个Channels的好处是，只需要更少的线程来处理通道。事实上，可以只用一个线程处理所有的通道。对于操作系统来说，线程之间上下文切换的开销很大，而且每个线程都要占用系统的一些资源（如内存）。因此，使用的线程越少越好。

但是，需要记住，现代的操作系统和CPU在多任务方面表现的越来越好，所以多线程的开销随着时间的推移，变得越来越小了。实际上，如果一个CPU有多个内核，不使用多任务可能是在浪费CPU能力。不管怎么说，关于那种设计的讨论应该放在另一篇不同的文章中。在这里，只要知道使用Selector能够处理多个通道就足够了。

通过调用Selector.open()方法创建一个Selector，如下：
Selector selector = Selector.open();

为了将Channel和Selector配合使用，必须将channel注册到selector上。通过SelectableChannel.register()方法来实现，如下：
channel.configureBlocking(false);
SelectionKey key = channel.register(selector,Selectionkey.OP_READ);
与Selector一起使用时，Channel必须处于非阻塞模式下。这意味着不能将FileChannel与Selector一起使用，因为FileChannel不能切换到非阻塞模式。而套接字通道都可以。
注意register()方法的第二个参数。这是一个“interest集合”，意思是在通过Selector监听Channel时对什么事件感兴趣。可以监听四种不同类型的事件：
Connect
Accept
Read
Write
这四种事件用SelectionKey的四个常量来表示：
SelectionKey.OP_CONNECT
SelectionKey.OP_ACCEPT
SelectionKey.OP_READ
SelectionKey.OP_WRITE



Scatter/Gather
通道之间的数据传输
FileChannel
SocketChannel
ServerSocketChannel
Java NIO DatagramChannel

int readySet = selectionKey.readyOps();
Channel  channel  = selectionKey.channel();
Selector selector = selectionKey.selector();


Pipe
 Java NIO与IO
Java NIO Path


# 示例 #




# Netty #




**参考资料**
[netty官网](http://netty.io/)
[Java NIO系列教程](http://ifeve.com/overview/)
[攻破JAVA NIO技术壁垒](http://www.importnew.com/19816.html)