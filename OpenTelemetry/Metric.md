[toc]



metric，度量指标。

OpenTelemetry 允许用预定义的聚合和标签集记录原始测量或度量。

- 使用 OpenTelemetry API 记录原始测量允许最终用户决定应该为这个度量用什么聚合算法，以及定义标签（维度）。
- 使用 OpenTelemetry API 记录预定义聚合的度量同样重要。它允许收集 CPU 和内存使用，或者是像队列长度这样的简单度量





metric会和tracing联系起来。



# 记录原始测量

用于记录原始测量的主要类是 `Measure` 和 `Measurement` 。



## Measure

`Measure` 描述库记录的单个值的类型。

`Measure` 由名称、描述和一个值的单位标识。

`Measure` 定义了公开测量方法的库和聚合独立测量到 `Metric` 中的应用之间的关系。



## Measurement

`Measurement` 描述为 `Measure` 收集的单一值，`Measuremrnt` 是一个空接口，在 SDK 中定义。



# 使用预定义聚合记录度量

所有类型的预定义聚合度量的基类称为 `Metric` ，它定义了基本的度量属性,比如名称和标签。

继承 `Metric` 的类定义自己的聚合类型和单个测量或点的结构。

API 定义了以下类型的预定义聚合度量：

- Counter metric 报告瞬时测量。Counter 值可以上升或保持不变，但不可能下降也不可能为负，Counter 度量值有两种类型：`double` 和 `long`。
- Gauge metric 报告数字值的瞬时测量。Gauge 值可以上升或下降，也可以为负值。Gauge 度量值有两种类型：`double` 和 `long` 。

API 允许构造选定类型的 `Metric` ，SDK 定义了要导出的 `Metric` 值的查询方式。

每种类型的 `Metric` 拥有自己的 API 记录将要聚合的值。

API 支持推拉模型（push and pull model）来设置 `Metric` 值。



## 度量数据模型和SDK

基于 [metrics.proto](https://github.com/open-telemetry/opentelemetry-proto/blob/master/opentelemetry/proto/metrics/v1/metrics.proto) 建立度量数据模型（[Metrics Data Model](https://github.com/open-telemetry/opentelemetry-specification/blob/main/specification/metrics/datamodel.md)），这个数据模型定义了三种语义：

- API 使用的事件模型（Event model）
- SDK 和 OTLP 使用的 in-flight 数据模型
- 表示导出工具如何解释 in-flight 数据模型时间序列模型（TimeSeries model）。

Metrics 对数据的约束最小（例如，键中允许哪些字符），处理 Metrics 的代码应该避免对其进行验证和清洗。应该将数据传递给后端，依赖后端执行验证，并从后端返回错误。





