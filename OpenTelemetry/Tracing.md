[toc]



分布式追踪是一组事件，由单个逻辑操作触发，跨应用程序中的各个组件合并。

一个分布式追踪包含跨进程，网络，安全边界的事件。



# Traces

Trace由Span隐式定义，Span的有向无环图。



# Spans

Span代表了事务中的操作，每个Span封装以下状态

- 操作名称
- 起止时间戳
- 属性Attributes：K-V键值对
- 0或多个事件的集合Event：每个都是一个元组（时间戳，名称，属性），名称必须是字符串
- 父Span标识
- 与0或多个具有因果关系的Span链接（Links），通过相关Span的SpanContext
- 引用Span所需的SpanContext信息



## SpanContext

标识 Trace 中的 Span 的所有信息，必须传播到子 Span 和跨进程边界。

一个 SpanContext 包含从父 Span 传播到子 Span 的跟踪标识符和选项。



- TraceId：trace 的标识符。全局唯一，随机生成 16 个字节。TraceId 用于将跨进程的特定 trace 的所有 span 分组在一起。
- SpanId：span 的标识符。全局唯一，随机生成 8 个字节。当传递给子 Span 时，该标识符将成为子 Span 的父 span id 。
- TraceFlags：trace 的选项。表示为一字节（位图 bitmap）
  - Sampling bit：表示 trace 是否被采样的比特（掩码 `0x1`）
- Tracestate：在一个键值对列表中携带特定于追踪系统的上下文。Tracestate 允许不同的供应商传播额外的信息，用它们的遗留的 Id 格式进行互操作。



## Span之间的链接（Links）

一个 Span 必须和 0 个或多个因果关联的其他 Span 链接（由 SpanContext 定义）。链接可以指向一个 Trace 内的 Span 或跨 Trace 。

当 Trace 进入服务的可信边界，并且服务策略要求生成新的 Trace 而不信任传入的 Trace 上下文时，可以用来表示原始 trace 和接下来的 trace 之间的关系。



## attributes

相当于tag, 可以给 span 添加 metadata 数据。

`SetAttributes`属性可以多次添加，不存在就添加，存在就覆盖。



## Events

类似于日志, 比如在 trace 中嵌入请求体跟响应体。

