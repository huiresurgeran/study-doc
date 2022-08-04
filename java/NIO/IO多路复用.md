[toc]



# 多路复用IO实现

著作权归https://pdai.tech所有。 链接：https://pdai.tech/md/java/io/java-io-nio-select-epoll.html

:

| IO模型 | 相对性能 | 关键思路         | 操作系统      | JAVA支持情况                                                 |
| ------ | -------- | ---------------- | ------------- | ------------------------------------------------------------ |
| select | 较高     | Reactor          | windows/Linux | 支持,Reactor模式(反应器设计模式)。Linux操作系统的 kernels 2.4内核版本之前，默认使用select；而目前windows下对同步IO的支持，都是select模型 |
| poll   | 较高     | Reactor          | Linux         | Linux下的JAVA NIO框架，Linux kernels 2.6内核版本之前使用poll进行支持。也是使用的Reactor模式 |
| epoll  | 高       | Reactor/Proactor | Linux         | Linux kernels 2.6内核版本及以后使用epoll进行支持；Linux kernels 2.6内核版本之前使用poll进行支持；另外一定注意，由于Linux下没有Windows下的IOCP技术提供真正的 异步IO 支持，所以Linux下使用epoll模拟异步IO |
| kqueue | 高       | Proactor         | Linux         | 目前JAVA的版本不支持                                         |



# 使用场景

多路复用IO技术，适用于高并发场景（1ms内同时有上千个连接请求准备好）。



# 模型



## 传统IO模型

![img](https://pdai.tech/_images/io/java-io-reactor-1.png)

一个Server对接N个客户端。

客户端连接之后，为每个客户端都分配一个执行线程。



### 特点

- 每个客户端连接，服务端会分配一个线程给该客户端，该线程会处理：读取数据，解码，业务计算，编码，发送数据
- 服务端的吞吐量与服务器提供的线程数呈线性关系



### 缺点

- 服务器的并发量对服务端能够创建的线程数有很大的依赖关系，但是服务器线程却是不能无限增长的
- 服务端每个线程不仅要进行IO读写操作，而且还需要进行业务计算
- 服务端在获取客户端连接，读取数据，以及写入数据的过程都是阻塞类型的，在网络状况不好的情况下，这将极大的降低服务器每个线程的利用率，从而降低服务器吞吐量



## Reactor事件驱动模型

![img](https://pdai.tech/_images/io/java-io-reactor-2.png)

非阻塞IO。



### 角色

- 客户端连接
- Reactor：分发从Acceptor收到的连接
- Acceptor：接收客户端的连接，然后交给Reactor
- Handler：处理读取数据，编解码等等



### 优点

- Reactor模型是以事件进行驱动的，其能够将接收客户端连接，网络读和网络写，以及业务计算进行拆分，从而极大的提升处理效率
- Reactor模型是异步非阻塞模型，工作线程在没有网络事件时可以处理其他的任务，而不用像传统IO那样必须阻塞等待



### 瓶颈

由于网络读写和业务操作在同一个线程，高并发情况下，依然有瓶颈：

- 高频率的网络读写事件处理
- 大量的业务操作处理



## Reactor模型----业务处理与IO分离

![img](https://pdai.tech/_images/io/java-io-reactor-3.png)



### 特点

- 使用一个线程进行客户端连接的接收以及网络读写事件的处理
- 在接收到客户端连接之后，将该连接交由线程池进行数据的编解码以及业务计算



### 优点

这种模式相较于前面的模式性能有了很大的提升，主要在于在进行网络读写的同时，也进行了业务计算，从而大大提升了系统的吞吐量。



### 缺点

- 网络读写是一个比较消耗CPU的操作，在高并发的情况下，将会有大量的客户端数据需要进行网络读写，此时一个线程将不足以处理这么多请求。



## Reactor模型----并发读写

![img](https://pdai.tech/_images/io/java-io-reactor-4.png)

使用线程池进行网络读写，而仅仅只使用一个线程专门接收客户端连接。



### 改造点

改进后的Reactor模型将Reactor拆分为了mainReactor和subReactor

- mainReactor主要进行客户端连接的处理，处理完成之后将该连接交由subReactor
- subReactor处理客户端的网络读写，subReactor则是使用一个线程池来支撑的，其读写能力将会随着线程数的增多而大大增加



### 性能

基本上可以支持百万连接