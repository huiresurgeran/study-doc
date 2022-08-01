[toc]



LogModel日志数据模型

1. 系统日志:操作系统、硬件产生的日志，比如Syslog
2. 第三方应用日志：流行的第三方软件的日志，比如Mysql慢日志，Apache日志等
3. 应用日志：业务应用产生的日志



| 字段名         | 描述                   | 必选 | 说明                                                         |
| -------------- | ---------------------- | ---- | ------------------------------------------------------------ |
| imestamp       | 日志时间戳             | 是   | Uint64，纳秒                                                 |
| TraceId        | 关联请求的TraceId      | 否   | 字节数组                                                     |
| SpanId         | 关联请求的SpanId       | 否   | 字节数组                                                     |
| TraceFlags     | W3C trace flag         | 否   | 单字节                                                       |
| SeverityText   | 日志等级的可读描述     | 否   | 不设置，按照`SeverityNumber`的默认Mapping规则映射            |
| SeverityNumber | 日志等级               | 否   | 和Syslog中的日志等级比较像                                   |
| ShortName      | 用于标识日志类型的短语 | 否   | 一般用一个特定的词来标识，不要超过50个字节                   |
| Body           | 具体的日志内容         | 否   | 日志内容为Any类型<br>可以使int，string，bool，float，也可以是数组或者map |
| Resource       | Log关联的Resource      | 否   | K-V对列表，包括主机名，进程号，服务名等，可以用于关联Tracing和Metric |
| Attributes     | 额外关联属性           | 否   | K-V对列表，key是String，value是Any类型                       |
|                |                        |      |                                                              |
|                |                        |      |                                                              |
|                |                        |      |                                                              |
|                |                        |      |                                                              |



| SeverityNumber | 通用等级 | 含义                        | Short  Name    |
| -------------- | -------- | --------------------------- | -------------- |
| 1-4            | TRACE    |                             | TRACE - TRACE4 |
| 5-8            | DEBUG    |                             | DEBUG - DEBUG4 |
| 9-12           | INFO     |                             | INFO - INFO4   |
| 13-16          | WARN     |                             | WARN - WARN4   |
| 17-20          | ERROR    |                             | ERROR - ERROR4 |
| 21-24          | FATAL    | application or system crash | FATAL - FATAL4 |



W3C Trace Context: https://www.w3.org/TR/trace-context/?spm=a2c6h.12873639.article-detail.7.3cb21d65QoCb3C

