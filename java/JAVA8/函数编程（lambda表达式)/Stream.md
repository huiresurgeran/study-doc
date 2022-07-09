[toc]

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



# 模式

顺序执行&并行执行。



## 顺序流

```java
List<Person> people = list.getStream().collect(Collectors.toList());
```

使用顺序方法遍历，读完一个item再读下一个item





## 并行流

```java
List<Person> people = list.getStream().parallel().collect(Collectors.toList());
```

使用并行方式遍历，数组被分成多个段，每个在不同的线程处理，最后一起输出结果。



原理：

- 利用多核技术，将数据通过多核并行处理。

- 类似MapReduce，分布式，多机器



性能：多核机器，理论上并行比顺序快上一倍

```java
public static void performance() {
    long t0 = System.nanoTime();
    int a[] = IntStream.range(0, 1_000_000).filter(p -> p % 2 == 0).toArray();
    long t1 = System.nanoTime();
    int b[] = IntStream.range(0, 1_000_000).parallel().filter(p -> p % 2 == 0).toArray();
    long t2 = System.nanoTime();

    logger.info("serial time: {}", t1 - t0);
    logger.info("parallel time: {}", t2 - t1);
}
```



输出：

```
04:00:58.427 [main] INFO com.jsamuel.study.functional.interf.stream.SimpleStream - serial time: 81619798
04:00:58.432 [main] INFO com.jsamuel.study.functional.interf.stream.SimpleStream - parallel time: 45839563
```



# 常用方法

- `stream()`, `parallelStream()`
- 过滤：`filter()`
- 映射：`map()`
  - 比如从student list中获取student score，`map(Student::getScore)`
  - 将student score list每一个都减去10分，`map(i -> i - 10)`
- 计算： ` reduce()`
  - 比如计算student总分，`reduce(0,(a,b) -> a + b)`
  - 比如计算student最高分，`reduce(Integer::max)`
- 查找：`find()`, `findAny()`,  `findFirst()`
- 匹配：`match`()
- 排序：`sort()`
- 遍历：`forEach()`
- 将多个Stream连接成一个Stream：`flatMap()` 
- 集合：`collect(Collectors.toList())`
- 类数据库：`distinct`, `limit`, `count`
- `min`, `max`, `summaryStatistics`





# 参考文档

https://www.runoob.com/java/java8-streams.html