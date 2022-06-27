[toc]

OpenTelemetry：合并了openTracing和OpenCencus

常用开源项目：[Jaeger](https://www.jaegertracing.io/)和[Prometheus](https://prometheus.io/)，这两个是调用链追踪工具，用来后端可视化存储，查询，观测等



# OpenTelemetry的数据源和内容



## Traces



### 1. trace

![在这里插入图片描述](https://img-blog.csdnimg.cn/img_convert/cb4c68dba74eb3ab83e468aa38c8e733.png#pic_center)



上图是一个简单的trace示例。

比如我们可以把自己一天的行程，看成一个大trace。



有效的trace由有效的子trace组成。

我们可以把我们一天当做做的事情，当做是子trace。



一个trace是链接span的集合，这些span表示请求中的工作单元的命名和定时操作。



### 2. span

span是trace的构建块，是一个命名的定时操作（用一个name来表示，表示需要耗费一定时间的操作，即为span，比如一次命名的http请求或者rpc调用等），他表示分布式系统中的一部分工作流。多个span链接在一起行程一个trace。



trace和span的关系：

- trace通常被认为是span的树，用来反映每个span开始和完成的时间
- 一个trace包含了一个根span，这个根span包括了整一个请求从头到尾的延迟
- trace开始于一个代表请求开始的root span，这个root span可以有一个或多个子span，这些子span又可以有他们自己的子span



span的目的是向可观测性工具提供有关程序执行的一些信息，包括

- 操作名，span name
- 开始和结束时间戳
- spanContext
- 一个属性set
- 有序事件列表



### 3. trace如何跨域传播



#### （1）context

context上下文，是一种传播机制，它可以跨API边界和有逻辑关联的执行单元之间**传递执行范围内的值**。

`Cross-cutting concerns`(横切关注点)可以使用共享的相同context对象来访问其处理的数据。

`context`必须是不可变的，并且它的写操作必须得返创建一个新的context，这个新context包含原始值和指定更新值。



#### （2）propagation

`propagation`能让trace变成分布式trace的机制（跨域），能够促进`context`在服务和进程之间的流动。

`context`可以被注入inject到一个请求中，并且由接受服务来提取extract并生成新的span，这个服务会生成新的请求，再把context注入到请求中，并发送到其他的服务。

![在这里插入图片描述](https://img-blog.csdnimg.cn/img_convert/469c88cb6304ab088af180549e32ee21.png#pic_center)



`propagation`通常通过特定的请求拦截器(request interceptors)和传播器(propagator)来实现，`interceptors`用来检测传入和传出的请求，并分别使用`propagator`的注入和提取操作。



#### （3）propagator

`propagator`是用来实现`propagation`机制的，在应用程序之间交换`message`中读写`context`对象。



api：`Inject`和`Extract`

- 这两个操作是为了能够分别地从`carriers`中**写入**和**读取**数据
- 每一个`propagator`必须定义其特定的`carrier`类型，并且允许定义其他参数。



inject：

- 将值注入到carrier中。例如，注入到一个HTTP请求头中
- 必备参数：
  - context，一个propagator必须先从context中检索适当的值
  - carrier，这个carrier必须是能够承载propagation字段



extract

- 从传入的请求中提取出值。例如，从HTTP请求头中提取
- 必备参数：
  - context
  - carrier
- 返回数据
  - 返回一个新的context，这个返回的context是传入的context参数派生出来的



#### （4）carrier

`carrier`是`propagator`用来**读值和写值**的**介质**。

每个特定的propagator类型都定义了与其预期的carrier类型，比如`string map` 或 一个 `byte array`。

在trpc中就是一个request。



### 4. tracing api

包括

- TracerProvider
  - API的入口点，提供了tracers的访问权限
  - 创建tracer或获取一个已有tracer
- Tracer：
  - 负责创建，获取span的类
- span：
  - 用来跟踪操作的API
  - `Get Context`: 从指定span中得到对应的spanContext
  - `Set Attributes` : 给span设置一些属性，如键值对形式的元数据
    - Attributes是作为元数据应用于span的键值对，利用这些键值对属性能够方便聚集，筛选和分组trace
    - Attributes可以在创建span时就添加，也可以在span的结束前的任意时刻添加
  - `Add Event` : 添加一个可读的消息
    - Event是一个可读的信息，它描述了一个span在整个生命周期“正在发生的事情”
- context
  - 从`context`实例中提取`span`
  - 往`context`实例中注入`span`



# 参考文档

https://blog.csdn.net/dopamine_joker/article/details/120573857



