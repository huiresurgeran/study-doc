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



# 常用示例



## 匿名类

```java
public static void main(String[] args) {
    // (params) -> expression
    // (params) -> statement
    // (params) -> { statements }
    new Thread(() -> logger.info("lambda expression, anonymous class")).start();
}
```





## ForEach

可以用lambda表达式，也可以用方法引用

```java
public static void main(String[] args) {
    List features = Arrays.asList("piano", "wiolin", "guitar");
    features.forEach(n -> logger.info("feature: {}", n));

    // 方法引用，::
    features.forEach(System.out::println);
}
```



## 方法引用

```java
public static void main(String[] args) {
    // 构造引用
    Supplier<String> s = String::new;

    // 对象::实例方法 Lambda表达式的（形参列表）与实例方法（实参列表）类型，个数是对应的
    List list = Arrays.asList("piano", "wiolin", "guitar");
    // list.forEach(t -> System.out.print(t));
    list.forEach(System.out::println);

    // 类名::静态方法
    // Stream<Double> stream = Stream.generate(() -> Math.random());
    Stream<Double> stream = Stream.generate(Math::random);
    stream.forEach(System.out::println);

    // 类名::实例方法
    // TreeSet<String> set = new TreeSet<>((s1, s2) -> s1.compareTo(s2));
    // 上一行代码，lambda表达式，可以被替换成下面的方法引用
    TreeSet<String> set = new TreeSet<>(String::compareTo);
}
```



- `构造方法引用`：new 一个实例对象
- `对象::实例方法`：Lambda表达式的（形参列表）与实例方法（实参列表）类型，个数是对应的
- `类名::静态方法`：static方法
- `类名::实例方法`：



## Filter&Predicate

filter里面需要放Predicate类型参数

```java
public static void main(String[] args) {
    List<String> languages = Arrays.asList("Java", "C++", "GO", "Javascript");

    logger.info("Language starts with J: ");
    languages.stream()
            .filter((str) -> str.startsWith("J"))
            .forEach((name) -> logger.info("name: {}", name));

    logger.info("Language end with a: ");
    languages.stream()
            .filter((str) -> str.endsWith("a"))
            .forEach((name) -> logger.info("name: {}", name));

    logger.info("print all languages: ");
    languages.stream()
            .filter((str) -> true)
            .forEach((name) -> logger.info("name: {}", name));

    logger.info("print no language: ");
    languages.stream()
            .filter((str) -> false)
            .forEach((name) -> logger.info("name: {}", name));

    logger.info("print language whose length greater than 4: ");
    languages.stream()
            .filter((str) -> str.length() > 4)
            .forEach((name) -> logger.info("name: {}", name));

    logger.info("print language starts with J and length is 4: ");
    multiple(languages);

}
```





支持多个Predicate组合filter，支持逻辑操作and, or xor

```java
public static void multiple(List names) {
    Predicate<String> startsWithJ = (n) -> n.startsWith("J");
    Predicate<String> fourLetterLong = (n) -> n.length() == 4;
    // 多个Predicate组合，可以用and，or，xor
    // pre1.and(pre2)
    names.stream()
            .filter(startsWithJ.and(fourLetterLong))
            .forEach((name) -> logger.info("name: {}", name));
}
```





## 	Map&Reduce

map：将集合类元素进行转换

reduce：将所有值合并成一个

```java
public static void main(String[] args) {
    List<Integer> costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);

    double bill = costBeforeTax.stream()
            // 集合类元素转换
            .map((cost) -> cost + .12 * cost)
            // 所有值合并成一个
            .reduce((sum, cost) -> sum + cost)
            .get();
    logger.info("total expense: {}", bill);
}
```





## Collectors

```java
public static void main(String[] args) {
    List<String> G7 = Arrays.asList("USA", "Japan", "France", "Germany", "Italy", "U.K.","Canada");
    String G7Countries = G7.stream()
            .map(x -> x.toUpperCase())
            .collect(java.util.stream.Collectors.joining(", "));
    logger.info("g7 countries: {}", G7Countries);
}
```



- Collectors.joining(", ")，连接集合
- Collectors.toList()，生成list
- Collectors.toSet() ，生成set
- Collectors.toMap(MemberModel::getUid, Function.identity())，生成map



## flatMap - TODO

将多个stream连成一个stream

以下有两个例子

- Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4))
- stream().flatMap(list -> list.stream())

```java
public static void main(String[] args) {
    // 多个stream连成一个stream
    List<Integer> result = Stream.of(Arrays.asList(1, 2), Arrays.asList(3, 4))
            .flatMap(a -> a.stream())
            .collect(Collectors.toList());
    result.forEach(i -> logger.info("integer: {}", i));

    List<String> teamIndia = Arrays.asList("Virat", "Dhoni", "Jadeja");
    List<String> teamAustralia = Arrays.asList("Warner", "Watson", "Smith");
    List<List<String>> playersInWorldCup2016 = new ArrayList<>();
    playersInWorldCup2016.add(teamIndia);
    playersInWorldCup2016.add(teamAustralia);
    List<String> flatMapList = playersInWorldCup2016.stream()
            .flatMap(list -> list.stream())
            .collect(Collectors.toList());
    flatMapList.forEach(i -> logger.info("flatMapList: {}", i));

}
```



## distinct

去重

```java
public static void main(String[] args) {
    // 去重
    List<String> list = Arrays.asList("toto", "sherry", "toto", "yuki");
    List<String> result = list.stream().distinct().collect(Collectors.toList());
    result.forEach(str -> logger.info("name: {}", str));
}
```



## count

计数

```java
public static void main(String[] args) {
    // 计算总数
    long count = Arrays.asList(1, 2, 3, 4, 5).stream()
            .filter(p -> p > 3)
            .count();
    logger.info("count: {}", count);
}
```



## Match

```java
public static void main(String[] args) {
    List<String> list = Arrays.asList("asd", "bnm", "qwe");

    // 匹配任何一个
    boolean anyStartsWithA = list.stream().anyMatch(s -> s.startsWith("a"));
    logger.info("anyStartsWithA: {}", anyStartsWithA);

    // 全部匹配
    boolean allStartsWithA = list.stream().allMatch(s -> s.startsWith("a"));
    logger.info("allStartsWithA: {}", allStartsWithA);

    // 不匹配
    boolean noneStartsWithA = list.stream().noneMatch(s -> s.startsWith("z"));
    logger.info("noneStartsWithA: {}", noneStartsWithA);


}
```

- anyMatch，匹配任何一个
- allMatch，匹配所有
- noneMatch，不匹配



## min,max,summaryStatistics

min，最小值，里面使用Comparator接口

```java
public static void min(List<Person> lists) {
    Person min = lists.stream().min(Comparator.comparing(t -> t.getN())).get();
    logger.info("person min: {}", min.getN());
}
```



max，最大值，里面使用Comparator接口

```java
public static void max(List<Person> lists) {
    Person max = lists.stream().max(Comparator.comparing(t -> t.getN())).get();
    logger.info("person max: {}", max.getN());
}
```



多个条件，自己实现`new Comparator`

```java
public static void multiple(List<Person> lists) {
    Person max = lists.stream().min(new Comparator<Person>() {
        @Override
        public int compare(Person o1, Person o2) {
            if (o1.getN() > o2.getN()) {
                return -1;
            }
            if (o1.getN() < o2.getN()) {
                return 1;
            }
            return 0;
        }
    }).get();
    logger.info("person multiple min: {}", max.getN());
}
```





summaryStatistics，统计分析数据

支持获取数字的个数、最小值、最大值、总和以及平均值

```java
// summaryStatistics，需要转成IntStream
List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
IntSummaryStatistics statistics = primes.stream().mapToInt((x) -> x).summaryStatistics();
logger.info("max: {}", statistics.getMax());
logger.info("min: {}", statistics.getMin());
logger.info("sum: {}", statistics.getSum());
logger.info("average: {}", statistics.getAverage());
logger.info("count: {}", statistics.getCount());
```





## peek - TODO





# 参考文档

https://www.runoob.com/java/java8-streams.html