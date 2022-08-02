

- [x] NettyChannel
- [x] NettyChannelBuffer
- [x] NettyChannelManager
- [x] NettyClientHandler
- [x] NettyClientTransportFactory
- [x] NettyCodecAdapter
- [ ] NettyFutureUtils
- [ ] NettyServerHandler
- [ ] NettyServerTransportFactory
- [x] NettyTcpClientTransport
- [ ] NettyTcpServerTransport



区别

1. netty支持tcp+udp，fitnetty只支持tcp
2. fitnetty区分busy，idle，destroy，用途？
3. fitnetty支持参数maxConns，connsNumber，保留channel-channelFuture map，用于软重启



1. 根据transporter配置，加载NettyClientTransportFactory
2. NettyClientTransportFactory，根据network配置，初始化NettyTcpClientTransport（tcp or udp）
   1. 初始化config，handler，codec，channels
   2. 初始化多个实例共享的NioEventLoopGroup：NioEventLoopGroup
3. 





```
NettyClientTransportFactory.create(ProtocolConfig config, ChannelHandler handler,ClientCodec clientCodec)
-> new NettyTcpClientTransport(config, handler, clientCodec)
	-> 
-> new NettyUdpClientTransport(config, handler, clientCodec)
```



NettyTcpClientTransport继承AbstractClientTransport

AbstractClientTransport中有

- 参数
  - String name：类名，用于日志打印
  - ProtocolConfig config：service实例配置信息
  - ClientCodec codec：编码插件
  - ChannelHandler handler：channel事件处理器
  - InetSocketAddress remoteAddress：远程地址
  - ReentrantLock connLock：conn lock
  - AtomicInteger channelIdx：轮询的channel索引
  - List<ChannelFutureItem> channels：channel列表（默认size2）
  - LifecycleObj lifecycleObj：内置生命周期控制
- 方法
  - open：初始化客户端transport
    - lifecycleBase.start()
      - LifecycleObj.startInternal()
        - NettyTcpClientTransport.doOpen()
  - isConnected：循环判断channels中的每个item，只要有一个建立连接就表示连接建立成功,或者初始化的状态是也认为是ok的
  - isClosed：判断 clientTransport 是否关闭
    - lifecycleObj.isFailed() || lifecycleObj.isStopping() || lifecycleObj.isStopped()
  - close：
    - lifecycleBase.stop()
      - LifecycleObj.stopInternal()
        - NettyTcpClientTransport.doClose()
  - send：异步化发送包
    - isClosed
    - getChannel0().thenCompose(f -> f.send(msg))
      - thenCompose将前一个Future的返回结果作为后一个操作的输入
      - f.send()，调用的是AbstractChannel.send
        - isClosed
        - isConnected
        - doSend()
  - getChannel：轮询获取channel
    - isClosed()
    - getChannel0()
  - getChannel0：
    - useChannelPool()
    - 获取channel index
    - 根据channel index，ensureChannelActive
    - channels.get(channel index).getChannelFuture()，返回的是一个channel
      - 
    - 如果没有，就createChannel()
      - make()，创建新的channel，类型为CompletableFuture<Channel>，异步创建
      - make.whenComplete()，获取到建立好的channel
        - 打印日志
        - 增加回调判断防止连接泄漏：连接建立成功时,发现close时,则直接关闭已建立的连接
          - isClosed()
          - channel.close()
  - ensureChannelActive()
    - 未发起连接建立/连接断掉 =》发起新的创建连接的动作createChannel()
      - 状态一，未发起连接：isNotYetConnect()
        - channelFuture == null
      - 状态二，正在建立连接：isConnecting()，
        - channelFuture != null && !channelFuture.isDone()
      - 判断channel是否可用：isAvailabel()
        - channelFuture == null，不可用
        - !channelFuture.isDone() =》 isConnecting状态表示不可用
  - 抽象了以下方法给子类实现
    -  doOpen：open相关
    - doClose：close相关
    - make：创建新的channel, future cancel时需要保证最底层的channel future也cancel掉
    - useChannelPool：是否使用channel连接池
  - 内部类
    - ChannelFutureItem
      - CompletableFuture<Channel> channelFuture
      - ProtocolConfig config
    - LifecycleObj extends LifecycleBase
      - initInternal()
      - startInternal()
        - doOpen()
        - useChannelPool()
          - 根据配置的连接数，提前建立好连接channels.add(new ChannelFutureItem)
      - stopInternal()
        - channels => each ChannelFutureItem => close()
        - doClose()
        - handler.destroy()
      - init
        - 修改life cycle 状态
        - initInternal()
      - start
        - preStart：
          - 没有init，先触发init
          - failed，先触发stop，再start
          - 初始化ok或者stop的状态可以迁移至start状态
        - 修改life cycle 状态
        - startInternal()
      - stop
        - 修改life cycle 状态
        - stopInternal()



NettyTcpClientTransport

|        | Trpc-netty                                                   | Fit-netty                                                    |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 参数   | 多个实例共享的NioEventLoopGroup：NioEventLoopGroup SHARE_EVENT_LOOP_GROUP | 多个实例共享的NioEventLoopGroup：NioEventLoopGroup SHARE_EVENT_LOOP_GROUP |
|        | Bootstrap bootstrap                                          | Bootstrap bootstrap                                          |
|        | ConcurrentHashSet<Channel> channelSet                        | ConcurrentHashSet<Channel> channelSet                        |
|        | 共享的NioEventLoopGroup使用数量：AtomicInteger SHARE_EVENT_LOOP_GROUP_USED_NUMS |                                                              |
|        |                                                              | 繁忙的channel：Queue<ChannelFutureItem> busyChannels         |
|        |                                                              | 空闲的channel：BlockingQueue<ChannelFutureItem> idleChannels |
|        |                                                              | 销毁的channel：Queue<ChannelFutureItem> destroyChannels      |
|        |                                                              | 软重启 - (连接-连接future)：Map<Channel, ChannelFutureItem> cfiMap |
|        |                                                              | 软重启 - 连接数：AtomicInteger channelCnt                    |
|        |                                                              | 软重启 - 最大连接数：int maxConns = 100                      |
| 方法   | 初始化<br>1. 初始化SHARE_EVENT_LOOP_GROUP                    | 初始化<br>1. 初始化SHARE_EVENT_LOOP_GROUP<br>2. 初始化lifecycleObj = new NettyLifecycleObj()，覆盖掉AbstractClientTransport中的lifecycleObj，覆盖掉LifecycleObj中的startInternal()<br/>3. 初始化busyChannels = new ConcurrentLinkedDeque<>()<br/>4. 初始化idleChannels = new LinkedBlockingDeque<>()<br/>5. 初始化destroyChannels = new ConcurrentLinkedQueue<>()<br/>6. 初始化channelCnt = new AtomicInteger()<br/>7. 初始化cfiMap = new ConcurrentHashMap<>()<br/>8. 初始化maxConns（配置>100，则使用配置的值） |
|        | doOpen                                                       | doOpen                                                       |
|        | make                                                         | make                                                         |
|        | doClose                                                      | doClose                                                      |
|        | getChannels                                                  | getChannels                                                  |
|        | useChannelPool                                               | useChannelPool                                               |
|        |                                                              | getNettyClientHandler                                        |
|        |                                                              | getChannel0                                                  |
|        |                                                              | send                                                         |
| 内部类 |                                                              | NettyLifecycleObj extends LifecycleObj                       |
|        |                                                              | 重写方法：startInternal()                                    |



|               | trpc-netty                                                   | Fit-netty                                                    | 不一致                                                       |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| doOpen        | 初始化bootstrap = New Bootstrap()                            | 初始化bootstrap = New Bootstrap()                            |                                                              |
|               | 初始化NioEventLoopGroup myEventLoopGroup = new NioEventLoopGroup/SHARE_EVENT_LOOP_GROUP | 初始化NioEventLoopGroup myEventLoopGroup = new NioEventLoopGroup/SHARE_EVENT_LOOP_GROUP |                                                              |
|               | SHARE_EVENT_LOOP_GROUP_USED_NUMS.incrementAndGet();          |                                                              | Fit-netty没有SHARE_EVENT_LOOP_GROUP_USED_NUMS                |
|               | bootstrap.group(myEventLoopGroup).channel(NioSocketChannel.class) | bootstrap.group(myEventLoopGroup).channel(NioSocketChannel.class) |                                                              |
|               | 初始化NettyClientHandler clientHandler =  new NettyClientHandler | 初始化NettyClientHandler clientHandler =  new NettyClientHandler | NettyClientHandler不是同一个类型                             |
|               | 初始化channelSet = clientHandler.getChannelSet()             | 初始化channelSet = clientHandler.getChannelSet()             |                                                              |
|               | bootstrap.handler<br>1. 空闲状态处理：IdleStateHandler clientIdleHandler =   new IdleStateHandler()<br>2. channel管道：ChannelPipeline p = ch.pipeline()<br>3. netty编解码适配器：NettyCodecAdapter nettyCodec = NettyCodecAdapter.createTcpCodecAdapter(codec, config);<br>4. p.add(nettyCodec.getEncoder)<br/>.add(nettyCodec.getDecoder)<br/>.add(clientIdleHandler)<br/>.add(clientHandler) | bootstrap.handler<br/>1. 空闲状态处理：IdleStateHandler clientIdleHandler =   new IdleStateHandler()<br/>2. channel管道：ChannelPipeline p = ch.pipeline()<br/>3. netty编解码适配器：NettyCodecAdapter nettyCodec = NettyCodecAdapter.createTcpCodecAdapter(codec, config);<br/>4. p<br/>.add(nettyCodec.getEncoder)<br/>.add(nettyCodec.getDecoder)<br/>.add(clientIdleHandler)<br/>.add(clientHandler) | NettyCodecAdapter不是同一个类型                              |
| make          | NettyFutureUtils.fromConnectingFuture()                      | NettyFutureUtils.fromConnectingFuture()                      | NettyFutureUtils不是同一个类型：内部走到NettyChannel，区别在于fit-netty，多了busy参数 |
| doClose       | bootstrap.config().group().shutdownGracefully()              | bootstrap.config().group().shutdownGracefully();             |                                                              |
|               | closeShareEventLoopGroup()<br>1. SHARE_EVENT_LOOP_GROUP.shutdownGracefully();<br>2. SHARE_EVENT_LOOP_GROUP = null; |                                                              |                                                              |
|               |                                                              | busyChannels.clear();                                        |                                                              |
|               |                                                              | idleChannels.clear();                                        |                                                              |
|               |                                                              | destroyChannels.clear();                                     |                                                              |
|               |                                                              | cfiMap.clear();                                              |                                                              |
| getChannels   | 返回状态是isConnected的channels                              | 返回状态是isConnected的channels                              |                                                              |
| getChannel0   | 有channel pool<br>1. 获取channel index<br>2. 确保这个channel index代表的channel存活，不存在就新建<br>2.1 channels.get(chIndex)<br>2.2 未发起连接建立 或者 连接断掉时, 则发起新的创建连接的动作<br>2.3 createChannel()<br>2.4 new ChannelFutureItem<br>2.5 channels.set<br>3. 从channels根据index获取channel | 有channel pool<br>1. 空闲队列不为空，从空闲队列获取连接<br>1.1 idle channels -> busy channels<br>2. 空闲队列为空，判断是否到达最大值<br>2.1 channelCnt > maxConns，抛异常<br>2.2 channelCnt < maxConns<br>2.2.1 新建连接 CompletionStage<Channel> cf = createChannel()<br>2.2.2 ChannelFutureItem cfi = new ChannelFutureItem<br>2.2.3 busyChannels.add(cfi);<br>2.2.4 channelCnt.incrementAndGet();<br>2.2.5 cf.whenComplete<br>2.2.5.1 有异常，busyChannels.remove(cfi)，channelCnt.decrementAndGet()<br>2.2.5.2 无异常，cfiMap.put(c, cfi) | Fit-netty：<br>1. 区分idle，busy<br>2. 记录channelCnt，判断maxConns，用于软重启<br>3. 记录cfiMap，channel - channelFuture，用于软重启 |
|               | 没有pool，直接新建channel                                    | 没有pool，直接新建channel                                    |                                                              |
| Send          | isClosed()                                                   | isClosed()                                                   |                                                              |
|               |                                                              | CompletionStage<Void> future = FutureUtils.newFuture();      |                                                              |
|               | getChannel0().thenCompose(f -> f.send(msg));                 | getChannel0().thenCompose(f -> f.send(msg)).whenComplete((r, t)<br>1. 有异常：重试一次<br>2. 没有异常：future.toCompletableFuture().complete(r); | fit-netty多了一次重试                                        |
| startInternal | super.startInternal();                                       |                                                              |                                                              |
|               | doOpen()                                                     | doOpen()                                                     |                                                              |
|               | channels.add(new ChannelFutureItem())                        |                                                              | Trpc-netty，支持提前建立连接<br>fit-netty，不支持提前建立连接 |



NettyClientHandler

|      | trpc-netty                            | Fit-netty                                |
| ---- | ------------------------------------- | ---------------------------------------- |
| 参数 | ConcurrentHashSet<Channel> channelSet | ConcurrentHashSet<Channel> channelSet    |
|      |                                       | NettyTcpClientTransport clientTransport; |
|      | ChannelHandler handler                | ChannelHandler handler                   |
|      | ProtocolConfig config                 | ProtocolConfig config                    |
|      | boolean isTcp                         | boolean isTcp                            |
| 方法 | 初始化                                | 初始化                                   |
|      | isValid                               | isValid                                  |
|      | channelActive                         | channelActive                            |
|      | channelRead                           | channelRead                              |
|      | exceptionCaught                       | exceptionCaught                          |
|      | channelInactive                       | channelInactive                          |
|      | Write                                 | Write                                    |
|      | userEventTriggered                    | userEventTriggered                       |



|                    | trpc-netty                                                   | Fit-netty                                                    | 不同点                                                       |
| ------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 初始化             | handler                                                      | handler                                                      |                                                              |
|                    | isTcp                                                        | isTcp                                                        |                                                              |
|                    | config                                                       | config                                                       |                                                              |
|                    |                                                              | clientTransport                                              | fit-netty传入NettyTcpClientTransport clientTransport         |
| channelActive      | NettyChannel channel = NettyChannelManager.getOrAddChannel() | NettyChannel channel = NettyChannelManager.getOrAddChannel   | fit-netty的NettyChannel，多一个参数，isBusy                  |
|                    | channelSet.add(new NettyChannel)                             | channelSet.add(new NettyChanne)                              |                                                              |
|                    | handler.connected(channel)                                   | handler.connected(channel)                                   |                                                              |
|                    | NettyChannelManager.removeChannelIfDisconnected(ctx.channel()) | NettyChannelManager.removeChannelIfDisconnected(ctx.channel()) |                                                              |
|                    |                                                              | Ctx.channel不是active：clientTransport.toDestroy(channel);   | fit-netty会调用transport的销毁，销毁连接<br>1. cfiMap.remove(channel)<br>2. busyChannels.remove(cf)<br>3. idleChannels.remove(cf)<br>4. destroyChannels.add(cf)<br>5. channelCnt.decrementAndGet(); |
| channelRead        | NettyChannel nettyChannel = NettyChannelManager.getOrAddChannel | NettyChannel nettyChannel = NettyChannelManager.getOrAddChannel |                                                              |
|                    | handler.received                                             | handler.received                                             |                                                              |
|                    | // 短连接场景下close                                         | // 短连接场景下close                                         |                                                              |
|                    |                                                              | clientTransport.busyToIdle(nettyChannel);                    | Fit-netty处理请求后，将channel 从繁忙队列移到 idle           |
|                    | NettyChannelManager.removeChannelIfDisconnected              | NettyChannelManager.removeChannelIfDisconnected              |                                                              |
| exceptionCaught    | NettyChannel channel = NettyChannelManager.getOrAddChannel   | NettyChannel channel = NettyChannelManager.getOrAddChannel   |                                                              |
|                    | handler.caught                                               | handler.caught                                               |                                                              |
|                    | // 只有tcp场景下才能close                                    | // 只有tcp场景下才能close                                    |                                                              |
|                    |                                                              | clientTransport.toDestroy                                    | fit-netty会调用transport的销毁，销毁连接                     |
|                    | NettyChannelManager.removeChannelIfDisconnected              | NettyChannelManager.removeChannelIfDisconnected              |                                                              |
| channelInactive    | NettyChannel channel = NettyChannelManager.getOrAddChanne    | NettyChannel channel = NettyChannelManager.getOrAddChanne    |                                                              |
|                    | channelSet.remove(new NettyChannel                           | channelSet.remove(new NettyChannel                           |                                                              |
|                    | handler.disconnected                                         | handler.disconnected                                         |                                                              |
|                    |                                                              | clientTransport.toDestroy                                    | fit-netty会调用transport的销毁，销毁连接                     |
|                    | NettyChannelManager.removeChannelIfDisconnected              | NettyChannelManager.removeChannelIfDisconnected              |                                                              |
|                    |                                                              |                                                              |                                                              |
| write              | super.write                                                  | super.write                                                  |                                                              |
|                    | NettyChannel channel = NettyChannelManager.getOrAddChanne    | NettyChannel channel = NettyChannelManager.getOrAddChanne    |                                                              |
|                    | handler.send                                                 | handler.send                                                 |                                                              |
|                    | NettyChannelManager.removeChannelIfDisconnected              | NettyChannelManager.removeChannelIfDisconnected              |                                                              |
|                    |                                                              | Ctx.channel不是active：clientTransport.toDestroy(channel);   | fit-netty会调用transport的销毁，销毁连接                     |
| userEventTriggered | NettyChannel channel = NettyChannelManager.getOrAddChanne    | NettyChannel channel = NettyChannelManager.getOrAddChanne    |                                                              |
|                    | 只有tcp场景下才close                                         | 只有tcp场景下才close                                         |                                                              |
|                    | channel.close();                                             | channel.close();                                             |                                                              |
|                    |                                                              | clientTransport.toDestroy                                    | fit-netty会调用transport的销毁，销毁连接                     |
|                    | NettyChannelManager.removeChannelIfDisconnected              | NettyChannelManager.removeChannelIfDisconnected              |                                                              |



NettyCodecAdapter

|        | trpc-netty             | Fit-netty              |
| ------ | ---------------------- | ---------------------- |
| 参数   | channelHandler encoder | channelHandler encoder |
|        | channelHandler decoder | channelHandler decoder |
|        | Codec codec            | Codec codec            |
|        | Protocol config        | Protocol config        |
| 方法   | 初始化                 | 初始化                 |
|        | createTcpCodecAdapter  | createTcpCodecAdapter  |
|        | createUdpCodecAdapter  | createUdpCodecAdapter  |
|        | decode                 | decode                 |
| 内部类 | TcpEncoder0<br>encode  | TcpEncoder0<br/>encode |
|        | UdpEncoder0            |                        |
|        | TcpDecoder0<br>decode  | TcpDecoder0<br/>decode |
|        | TcpDecoder1<br>decode  |                        |
|        | UdpDecoder()           |                        |
|        |                        |                        |
|        |                        |                        |



|                        | trpc-netty                                                   | Fit-netty                                              | 不同点                                              |
| ---------------------- | ------------------------------------------------------------ | ------------------------------------------------------ | --------------------------------------------------- |
| 初始化                 | 可选udp<br>可选tcp decoder 0（批量解码，默认） 或者 tcp decoder 1 | 无可选                                                 |                                                     |
| TcpEncoder0<br/>encode |                                                              |                                                        | 一致                                                |
| TcpEncoder0<br/>decode |                                                              |                                                        | trpc-netty，支持批量解码<br>fit-netty不支持批量解码 |
| decode                 | ChannelBuffer message = new NettyChannelBuffer(input);       | ChannelBuffer message = new NettyChannelBuffer(input); | NettyChannelBuffer不是一个类型                      |
|                        |                                                              |                                                        |                                                     |
|                        |                                                              |                                                        |                                                     |
|                        |                                                              |                                                        |                                                     |
|                        |                                                              |                                                        |                                                     |
|                        |                                                              |                                                        |                                                     |



- NettyChannel
  - fitnetty，多了busy参数
- 
  - 
  - 