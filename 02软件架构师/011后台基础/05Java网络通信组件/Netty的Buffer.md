NIO Buffer部分学习

#Buffer的capacity,position和limit
缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。
###capacity
作为一个内存块，Buffer有一个固定的大小值，也叫“capacity”.你只能往里写capacity个byte、long，char等类型。一旦Buffer满了，需要将其清空（通过读数据或者清除数据）才能继续写数据往里写数据。
###position
当你写数据到Buffer中时，position表示当前的位置。初始的position值为0.当一个byte、long等数据写到Buffer后， position会向前移动到下一个可插入数据的Buffer单元。position最大可为capacity – 1.
当读取数据时，也是从某个特定位置读。当将Buffer从写模式切换到读模式，position会被重置为0. 当从Buffer的position处读取数据时，position向前移动到下一个可读的位置。
###limit
在写模式下，Buffer的limit表示你最多能往Buffer里写多少数据。 写模式下，limit等于Buffer的capacity。
当切换Buffer到读模式时， limit表示你最多能读到多少数据。因此，当切换Buffer到读模式时，limit会被设置成写模式下的position值。换句话说，你能读到之前写入的所有数据（limit被设置成已写数据的数量，这个值在写模式下就是position）
#Buffer的类型
Java NIO 有以下Buffer类型

- ByteBuffer、CharBuffer、DoubleBuffer、FloatBuffer、IntBuffer、LongBuffer、ShortBuffer
- MappedByteBuffer
#Buffer的分配
要想获得一个Buffer对象首先要进行分配。 每一个Buffer类都有一个allocate方法。下面是一个分配48字节capacity的ByteBuffer的例子。`ByteBuffer buf = ByteBuffer.allocate(48);`
#向Buffer中写数据
写数据到Buffer有两种方式：

- 从Channel写到Buffer。`int bytesRead = inChannel.read(buf); //read into buffer.`
- 通过Buffer的put()方法写到Buffer里。
flip()方法:flip方法将Buffer从写模式切换到读模式。调用flip()方法会将position设回0，并将limit设置成之前position的值。
#从Buffer中读取数据
从Buffer中读取数据有两种方式：

- 从Buffer读取数据到Channel。`int bytesWritten = inChannel.write(buf);`
- 使用get()方法从Buffer中读取数据。`byte aByte = buf.get();`
rewind()方法:Buffer.rewind()将position设回0，所以你可以重读Buffer中的所有数据。limit保持不变，仍然表示能从Buffer中读取多少个元素（byte、char等）。
clear()与compact()方法:一旦读完Buffer中的数据，需要让Buffer准备好再次被写入。可以通过clear()或compact()方法来完成。
如果调用的是clear()方法，position将被设回0，limit被设置成 capacity的值。换句话说，Buffer 被清空了。Buffer中的数据并未清除，只是这些标记告诉我们可以从哪里开始往Buffer里写数据。
如果Buffer中有一些未读的数据，调用clear()方法，数据将“被遗忘”，意味着不再有任何标记会告诉你哪些数据被读过，哪些还没有。
如果Buffer中仍有未读的数据，且后续还需要这些数据，但是此时想要先先写些数据，那么使用compact()方法。
**compact()方法将所有未读的数据拷贝到Buffer起始处。然后将position设到最后一个未读元素正后面。**limit属性依然像clear()方法一样，设置成capacity。现在Buffer准备好写数据了，但是不会覆盖未读的数据。

###mark()与reset()方法
通过调用Buffer.mark()方法，可以标记Buffer中的一个特定position。之后可以通过调用Buffer.reset()方法恢复到这个position。例如：

    buffer.mark();
    //call buffer.get() a couple of times, e.g. during parsing.
    buffer.reset();  //set position back to mark.

###equals()与compareTo()方法
可以使用equals()和compareTo()方法两个Buffer。
equals()

当满足下列条件时，表示两个Buffer相等：

有相同的类型（byte、char、int等）。
Buffer中剩余的byte、char等的个数相等。
Buffer中所有剩余的byte、char等都相同。
如你所见，equals只是比较Buffer的一部分，不是每一个在它里面的元素都比较。**实际上，它只比较Buffer中的剩余元素。**
就是说

compareTo()方法

compareTo()方法比较两个Buffer的剩余元素(byte、char等)， 如果满足下列条件，则认为一个Buffer“小于”另一个Buffer：

第一个不相等的元素小于另一个Buffer中对应的元素 。
所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。
（译注：剩余元素是从 position到limit之间的元素）


营业中 1
暂停营业 6
筹建中 7
已关闭 5

商户停止营业 1
暂停歇业 2
尚未营业 4