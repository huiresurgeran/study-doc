Stream流，抽象。



这种风格将要处理的元素集合看作一种流， 流在管道中传输， 并且可以在管道的节点上进行处理， 比如筛选， 排序，聚合等。



# 什么是Stream

Stream（流）是一个来自数据源的元素队列并支持聚合操作。

- **元素**是特定类型的对象，形成一个队列
- **数据源** 流的来源。 可以是集合，数组，I/O channel， 产生器generator 等。
- **聚合操作** 类似SQL语句一样的操作， 比如filter, map, reduce, find, match, sorted等



和Collection的不同

- **Pipelining**：中间操作都会返回流对象本身。
  - 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 
  - 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
- **内部迭代**：以前对集合遍历都是通过Iterator或者For-Each的方式, 显式的在集合外部进行迭代， 这叫做外部迭代。 Stream提供了内部迭代的方式， 通过访问者模式(Visitor)实现。





# 参考文档

https://www.runoob.com/java/java8-streams.html