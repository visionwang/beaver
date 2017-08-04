NIO是Jdk中非常重要的一个组成部分，基于它的Netty开源框架可以很方便的开发高性能、高可靠性的网络服务器和客户端程序。本文将就其核心基础类型`Channel`, `Buffer`, `Selector`进行详细介绍，此外将简要介绍Netty框架。
![](http://i.imgur.com/bmG0Qrb.png)

# Netty #



# 基础概念 #
** Java NIO(non-blocking IO非阻塞IO)**是jdk1.4后提供的新IO API，为所有基础类型都提供**类缓存**支持。其基础类型**`Channel`**定义了一个新的I/O接口，**支持锁和访问内存映射文件**，提供多路非阻塞式的高伸缩性网络I/O，其与传统IO的区别如下所示。
1. **Channels and Buffers（通道和缓冲区）**：标准的IO基于字节流和字符流进行操作的，而NIO是基于通道（Channel）和缓冲区（Buffer）进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。
2. **Asynchronous IO（异步IO）**：Java NIO可以让你异步的使用IO，例如：当线程从通道读取数据到缓冲区时，线程还是可以进行其他事情。当数据被写入到缓冲区时，线程可以继续处理它。从缓冲区写入通道也类似。
3. **Selectors（选择器）**：Java NIO引入了选择器的概念，选择器用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个的线程可以监听多个数据通道。

Tip:
对于.NET程序员来说，Windows的**完成端口模型**和这部分很类似。

## Channel ##
`Channel`通道和传统IO中的Stream流类似，但它是双向的，而流式单向的，如`InputStream`，`OutputStream`等，Channel的常见实现如下所示。
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

	try (RandomAccessFile aFile = new RandomAccessFile("src/test/resources/test.xml", "r");
			CloseableHttpClient client = HttpClients.createDefault();) {
		// 1.读数据
		byte[] buffer = new byte[1024];
		StringBuilder requestString = new StringBuilder();
		int bytesRead = 0;
		while (-1 != (bytesRead = aFile.read(buffer))) {
			requestString.append(new String(buffer), 0, bytesRead);
		}
	}

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
### 内存映射文件 ###
JAVA处理大文件，一般用BufferedReader,BufferedInputStream这类带缓冲的IO类，不过如果文件超大的话，更快的方式是采用MappedByteBuffer。MappedByteBuffer是NIO引入的文件内存映射方案，读写性能极高。

SocketChannel的读写是通过一个类叫ByteBuffer来操作的.这个类本身的设计是不错的,比直接操作byte[]方便多了. ByteBuffer有两种模式:直接/间接.间接模式最典型(也只有这么一种)的就是HeapByteBuffer,即操作堆内存 (byte[]).但是内存毕竟有限,如果我要发送一个1G的文件怎么办?不可能真的去分配1G的内存.这时就必须使用”直接”模式,即 MappedByteBuffer,文件映射.

先中断一下,谈谈操作系统的内存管理.一般操作系统的内存分两部分:物理内存;虚拟内存.虚拟内存一般使用的是页面映像文件,即硬盘中的某个(某些)特殊的文件.操作系统负责页面文件内容的读写,这个过程叫”页面中断/切换”. MappedByteBuffer也是类似的,你可以把整个文件(不管文件有多大)看成是一个ByteBuffer.MappedByteBuffer 只是一种特殊的ByteBuffer，即是ByteBuffer的子类。 MappedByteBuffer 将文件直接映射到内存（这里的内存指的是虚拟内存，并不是物理内存）。通常，可以映射整个文件，如果文件比较大的话可以分段进行映射，只要指定文件的那个部分就可以。

概念
FileChannel提供了map方法来把文件影射为内存映像文件： MappedByteBuffer map(int mode,long position,long size); 可以把文件的从position开始的size大小的区域映射为内存映像文件，mode指出了 可访问该内存映像文件的方式：

READ_ONLY,（只读）： 试图修改得到的缓冲区将导致抛出 ReadOnlyBufferException.(MapMode.READ_ONLY)
READ_WRITE（读/写）： 对得到的缓冲区的更改最终将传播到文件；该更改对映射到同一文件的其他程序不一定是可见的。 (MapMode.READ_WRITE)
PRIVATE（专用）： 对得到的缓冲区的更改不会传播到文件，并且该更改对映射到同一文件的其他程序也不是可见的；相反，会创建缓冲区已修改部分的专用副本。 (MapMode.PRIVATE)
MappedByteBuffer是ByteBuffer的子类，其扩充了三个方法：

force()：缓冲区是READ_WRITE模式下，此方法对缓冲区内容的修改强行写入文件；
load()：将缓冲区的内容载入内存，并返回该缓冲区的引用；
isLoaded()：如果缓冲区的内容在物理内存中，则返回真，否则返回假；

	public static void method4(){
	      RandomAccessFile aFile = null;
	      FileChannel fc = null;
	      try{
	          aFile = new RandomAccessFile("src/1.ppt","rw");
	          fc = aFile.getChannel();
	 
	          long timeBegin = System.currentTimeMillis();
	          ByteBuffer buff = ByteBuffer.allocate((int) aFile.length());
	          buff.clear();
	          fc.read(buff);
	          //System.out.println((char)buff.get((int)(aFile.length()/2-1)));
	          //System.out.println((char)buff.get((int)(aFile.length()/2)));
	          //System.out.println((char)buff.get((int)(aFile.length()/2)+1));
	          long timeEnd = System.currentTimeMillis();
	          System.out.println("Read time: "+(timeEnd-timeBegin)+"ms");
	 
	      }catch(IOException e){
	          e.printStackTrace();
	      }finally{
	          try{
	              if(aFile!=null){
	                  aFile.close();
	              }
	              if(fc!=null){
	                  fc.close();
	              }
	          }catch(IOException e){
	              e.printStackTrace();
	          }
	      }
	  }
	 
	  public static void method3(){
	      RandomAccessFile aFile = null;
	      FileChannel fc = null;
	      try{
	          aFile = new RandomAccessFile("src/1.ppt","rw");
	          fc = aFile.getChannel();
	          long timeBegin = System.currentTimeMillis();
	          MappedByteBuffer mbb = fc.map(FileChannel.MapMode.READ_ONLY, 0, aFile.length());
	          // System.out.println((char)mbb.get((int)(aFile.length()/2-1)));
	          // System.out.println((char)mbb.get((int)(aFile.length()/2)));
	          //System.out.println((char)mbb.get((int)(aFile.length()/2)+1));
	          long timeEnd = System.currentTimeMillis();
	          System.out.println("Read time: "+(timeEnd-timeBegin)+"ms");
	      }catch(IOException e){
	          e.printStackTrace();
	      }finally{
	          try{
	              if(aFile!=null){
	                  aFile.close();
	              }
	              if(fc!=null){
	                  fc.close();
	              }
	          }catch(IOException e){
	              e.printStackTrace();
	          }
	      }
	  }

可以看到差距拉大。
注：MappedByteBuffer有资源释放的问题：被MappedByteBuffer打开的文件只有在垃圾收集时才会被关闭，而这个点是不确定的。在Javadoc中这里描述：A mapped byte buffer and the file mapping that it represents remian valid until the buffer itself is garbage-collected。详细可以翻阅参考资料5和6.

### Scatter/Gatter ###
分散（scatter）从Channel中读取是指在读操作时将读取的数据写入多个buffer中。因此，Channel将从Channel中读取的数据“分散（scatter）”到多个Buffer中。

聚集（gather）写入Channel是指在写操作时将多个buffer的数据写入同一个Channel，因此，Channel 将多个Buffer中的数据“聚集（gather）”后发送到Channel。

scatter / gather经常用于需要将传输的数据分开处理的场合，例如传输一个由消息头和消息体组成的消息，你可能会将消息体和消息头分散到不同的buffer中，这样你可以方便的处理消息头和消息体。

	public class ScattingAndGather
	{
	    public static void main(String args[]){
	        gather();
	    }
	 
	    public static void gather()
	    {
	        ByteBuffer header = ByteBuffer.allocate(10);
	        ByteBuffer body = ByteBuffer.allocate(10);
	 
	        byte [] b1 = {'0', '1'};
	        byte [] b2 = {'2', '3'};
	        header.put(b1);
	        body.put(b2);
	 
	        ByteBuffer [] buffs = {header, body};
	 
	        try
	        {
	            FileOutputStream os = new FileOutputStream("src/scattingAndGather.txt");
	            FileChannel channel = os.getChannel();
	            channel.write(buffs);
	        }
	        catch (IOException e)
	        {
	            e.printStackTrace();
	        }
	    }
	}

### transferFrom & transferTo ###
FileChannel的transferFrom()方法可以将数据从源通道传输到FileChannel中。
	public static void method1(){
	        RandomAccessFile fromFile = null;
	        RandomAccessFile toFile = null;
	        try
	        {
	            fromFile = new RandomAccessFile("src/fromFile.xml","rw");
	            FileChannel fromChannel = fromFile.getChannel();
	            toFile = new RandomAccessFile("src/toFile.txt","rw");
	            FileChannel toChannel = toFile.getChannel();
	 
	            long position = 0;
	            long count = fromChannel.size();
	            System.out.println(count);
	            toChannel.transferFrom(fromChannel, position, count);
	 
	        }
	        catch (IOException e)
	        {
	            e.printStackTrace();
	        }
	        finally{
	            try{
	                if(fromFile != null){
	                    fromFile.close();
	                }
	                if(toFile != null){
	                    toFile.close();
	                }
	            }
	            catch(IOException e){
	                e.printStackTrace();
	            }
	        }
	    }

方法的输入参数position表示从position处开始向目标文件写入数据，count表示最多传输的字节数。如果源通道的剩余空间小于 count 个字节，则所传输的字节数要小于请求的字节数。此外要注意，在SoketChannel的实现中，SocketChannel只会传输此刻准备好的数据（可能不足count字节）。因此，SocketChannel可能不会将请求的所有数据(count个字节)全部传输到FileChannel中。

transferTo()方法将数据从FileChannel传输到其他的channel中。
上面所说的关于SocketChannel的问题在transferTo()方法中同样存在。SocketChannel会一直传输数据直到目标buffer被填满。

### Pipe ###

public static void method1(){
        Pipe pipe = null;
        ExecutorService exec = Executors.newFixedThreadPool(2);
        try{
            pipe = Pipe.open();
            final Pipe pipeTemp = pipe;
 
            exec.submit(new Callable<Object>(){
                @Override
                public Object call() throws Exception
                {
                    Pipe.SinkChannel sinkChannel = pipeTemp.sink();//向通道中写数据
                    while(true){
                        TimeUnit.SECONDS.sleep(1);
                        String newData = "Pipe Test At Time "+System.currentTimeMillis();
                        ByteBuffer buf = ByteBuffer.allocate(1024);
                        buf.clear();
                        buf.put(newData.getBytes());
                        buf.flip();
 
                        while(buf.hasRemaining()){
                            System.out.println(buf);
                            sinkChannel.write(buf);
                        }
                    }
                }
            });
 
            exec.submit(new Callable<Object>(){
                @Override
                public Object call() throws Exception
                {
                    Pipe.SourceChannel sourceChannel = pipeTemp.source();//向通道中读数据
                    while(true){
                        TimeUnit.SECONDS.sleep(1);
                        ByteBuffer buf = ByteBuffer.allocate(1024);
                        buf.clear();
                        int bytesRead = sourceChannel.read(buf);
                        System.out.println("bytesRead="+bytesRead);
                        while(bytesRead >0 ){
                            buf.flip();
                            byte b[] = new byte[bytesRead];
                            int i=0;
                            while(buf.hasRemaining()){
                                b[i]=buf.get();
                                System.out.printf("%X",b[i]);
                                i++;
                            }
                            String s = new String(b);
                            System.out.println("=================||"+s);
                            bytesRead = sourceChannel.read(buf);
                        }
                    }
                }
            });
        }catch(IOException e){
            e.printStackTrace();
        }finally{
            exec.shutdown();
        }
    }

DatagramChannel
Java NIO中的DatagramChannel是一个能收发UDP包的通道。因为UDP是无连接的网络协议，所以不能像其它通道那样读取和写入。它发送和接收的是数据包。

public static void  reveive(){
       DatagramChannel channel = null;
       try{
           channel = DatagramChannel.open();
           channel.socket().bind(new InetSocketAddress(8888));
           ByteBuffer buf = ByteBuffer.allocate(1024);
           buf.clear();
           channel.receive(buf);
 
           buf.flip();
           while(buf.hasRemaining()){
               System.out.print((char)buf.get());
           }
           System.out.println();
 
       }catch(IOException e){
           e.printStackTrace();
       }finally{
           try{
               if(channel!=null){
                   channel.close();
               }
           }catch(IOException e){
               e.printStackTrace();
           }
       }
   }
 
   public static void send(){
       DatagramChannel channel = null;
       try{
           channel = DatagramChannel.open();
           String info = "I'm the Sender!";
           ByteBuffer buf = ByteBuffer.allocate(1024);
           buf.clear();
           buf.put(info.getBytes());
           buf.flip();
 
           int bytesSent = channel.send(buf, new InetSocketAddress("10.10.195.115",8888));
           System.out.println(bytesSent);
       }catch(IOException e){
           e.printStackTrace();
       }finally{
           try{
               if(channel!=null){
                   channel.close();
               }
           }catch(IOException e){
               e.printStackTrace();
           }
       }
   }


Java NIO Path
Java的Path接口是Java NIO2 的一部分，是对Java6 和Java7的 NIO的更新。Java的Path接口在Java7 中被添加到Java NIO，位于java.nio.file包中， 其全路径是java.nio.file.Path。

一个Path实例代表了一个文件系统中的路径。一个路径可以指向一个文件或者一个文件夹。一个路径可以是绝对路径或者是相对路径。绝对路径是从根路径开始的全路径，相对路径是一个相对其他路径的文件或文件夹路径。相对路径可能会造成一点混乱，但是不要担心，在本文章中，我会详细解释相对路径的。


不要混淆了文件系统中的路径 和 操作系统中环境变量的path路径。java.nio.file.Path实例与环境变量中的path没有任何关系。

在很多地方java.nio.file.Path接口和java.io.File类是相似的，但是它们有几个主要的不同。 在很多类中，你可以使用Path 接口替换 file 类使用。

**参考资料**
[netty官网](http://netty.io/)
[Java NIO系列教程](http://ifeve.com/overview/)
[攻破JAVA NIO技术壁垒](http://www.importnew.com/19816.html)
[Netty是什么？](http://lippeng.iteye.com/blog/1907279)
[Netty源码解读](http://ifeve.com/netty1/)
[使用JAVA操作netty框架](http://flychao88.iteye.com/blog/1553058)
[对于Netty的十一个疑问](https://news.cnblogs.com/n/205413/)