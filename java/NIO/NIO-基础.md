[toc]



# 流与块

NIO把I/O抽象成块，每次I/O操作的单位都是一个块，块被读入内存之后是一个bytes[]，NIO可以一次读或写多个字节。



I/O和NIO的最重要的区别：

- 数据打包
- 传输的方式



I/O：

- 流的方式处理数据
- 一次处理一个字节数据
- 创建过滤器容易
- 数据处理比较慢
- 阻塞的

NIO：

- 块的方式处理数据
- 一次处理一个数据块
- 比面向流的I/O复杂
- 数据处理快
- 非阻塞的



# 通道与缓冲区



## 通道

channel是对I/O包中的流的模拟，可以通过channel读取和写入数据。



channel和stream的不同点

- 流只能在一个方向上移动：InputStream或者是OutputStream
- 通道是双向的：可以用于读，写，同时读写



channel包括的类型

- FileChannel: 从文件中读写数据
- DatagramChannel: 通过 UDP 读写网络中数据
- SocketChannel: 通过 TCP 读写网络中数据
- ServerSocketChannel: 可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 SocketChannel



## 缓冲区



### 缓冲区主要用途

发送给channel的所有数据，必须先放到缓冲区。

从channel中读取的任何数据，都要先读到缓冲区。

也就是说，不会直接对channel进行读取数据，必须要先经过缓冲区。



缓冲区提供了对数据的结构化访问。

缓冲区可以跟踪系统的读/写进程。



### 缓冲区实现

缓冲区可以看做是一个数组，但不仅是一个数组。



### 缓冲区类型

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer



### 缓冲区状态变量

-  capacity：最大容量
- position：当前已经读写的字节数
- limit：还可以读写的字节数



# 选择器

NIO，常被叫做非阻塞IO，因为NIO在网络通信中的非阻塞特性被广泛使用。



NIO，实现了IO多路复用中的Reactor模型：一个线程thread，使用一个选择器selector，通过轮询的方式，监听多个通道channel上的事件，让一个线程可以处理多个事件。



监听的通道channel为非阻塞，所以当channel上的IO事件没有到达时，不会进入阻塞状态一直等待，而是继续轮询其他channel，找到IO事件已经到达的channel执行。



![image](https://pdai.tech/_images/pics/4d930e22-f493-49ae-8dff-ea21cd6895dc.png)



# 内存映射文件？？

内存映射文件I/O，是一种读和写文件数据的方法。

比常规的基于流或者基于通道的I/O快很多。



向内存映射文件写入可能是危险的，可能会直接修改磁盘上的文件，因为修改数据和将数据保存到磁盘是没有分开的。