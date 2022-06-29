# 什么是SPI机制

SPI（Service Provider Interface），JDK内置的一种服务提供发现的机制，可以用来启用框架扩展和替换组件，一般是框架的开发人员使用。

Java的SPI机制可以为某个接口寻找服务实现。



当服务的提供者提供了一种接口的实现之后，需要在classpath下的`META-INF/services/`目录中创建一个以服务接口命名的文件，文件中的内容就是这个接口的具体的实现类。

当其他程序需要这个服务时，通过查找这个jar包的META-INF/services/中的配置文件，得到配置文件中接口的具体实现类名，再根据这个类名进行加载实例化，就可以使用该服务了。

JDK中查找服务的实现的工具类是：`java.util.ServiceLoader`



# 实现



1. 定义接口Search

```java
public interface Search {

    public List<String> searchDoc(String keyword);
}
```





2. 定义实现类FileSearch和DbSearch

```java
public class FileSearch implements Search {

    @Override
    public List<String> searchDoc(String keyword) {
        System.out.println("search by doc, keyword: " + keyword);
        return null;
    }
}
```

```java
public class DbSearch implements Search {

    @Override
    public List<String> searchDoc(String keyword) {
        System.out.println("search by DB, keyword: " + keyword);
        return null;
    }
}
```



3. 配置文件

路径：META-INF/services/

文件名：接口类full name(package+class)，`com.jsamuel.study.api.Search`

内容：需要使用到的实现类

```java
com.jsamuel.study.spi.FileSearch
com.jsamuel.study.spi.DbSearch
```



4. 测试代码

```java
public class TestSpi {

    @Test
    public void testSpi () {
        ServiceLoader<Search> loader = ServiceLoader.load(Search.class);
        Iterator<Search> it = loader.iterator();
        while(it.hasNext()) {
            Search search = it.next();
            search.searchDoc("hello world");
        }
    }
}
```



5. 输出结果

```
search by doc, keyword: hello world
search by DB, keyword: hello world
```



注意：

若`com.jsamuel.study.api.Search`中的内容改为

```java
com.jsamuel.study.spi.FileSearch
```

则输出结果为

```
search by doc, keyword: hello world
```



6. 原理说明

ServiceLoader.load()在加载接口时，会去META-INF/services/目录下找接口的全限定名文件，再根据文件里面的内容加载相应的实现类。

若多个jar包中，有相同的接口全限定名文件，会出现文件覆盖的情况，导致加载不到指定类。



# SPI机制的应用



JDBC接口 + mysql实现/postgresql实现/...



Common-Logging：LogFactory.getLog()



插件体系：最具spi思想

Eclipse使用OSGi作为插件系统的基础，动态添加新插件和停止现有插件。

插件的文件结构必须在指定目录下包含以下三个文件：

- META-INF/MANIFEST.MF：项目基本配置信息，版本，名称，启动器等
- build.properties：项目的编译配置信息，包括源代码路径，输出路径
- plugin.xml：插件的操作配置信息，包含弹出菜单及点击菜单后对应的操作执行类等



Spring中的SPI机制？？

在springboot的自动装配过程中，最终会加载`META-INF/spring.factories`文件，而加载的过程是由`SpringFactoriesLoader`加载的。

从CLASSPATH下的每个Jar包中搜寻所有`META-INF/spring.factories`配置文件，然后将解析properties文件，找到指定名称的配置后返回。（注意，同名将会覆盖）



# 使用规范

1. 需要一个目录
   1. META/service
   2. 放到ClassPath下面
2. 目录下面放置一个配置文件
   1. 文件名：待扩展的接口全名（package+class）
   2. 文件内容：要实现的接口实现类
   3. 文件必须为UTF-8编码
3. 如何使用
   1. ServiceLoad.load(xx.class)
   2. ServiceLoad<HelloInterface> loader = ServiceLoad.load<HelloInterface.class>;



# SPI和API的区别

spi，接口位于调用方所在的包中

- 概念上更依赖调用方
- 组织上位于调用方所在的包中
- 实现位于独立的包中
- 常见例子：插件模式的插件



api，接口位于实现方所在的包中

- 概念上更接近实现方
- 组织上位于实现方所在的包中
- 实现和接口在一个包中





# SPI机制的缺陷

- 不能按需加载，需要遍历所有的实现，并实例化，然后在循环中才能找到我们需要的实现。如果不想用某些实现类，或者某些类实例化很耗时，它也被载入并实例化了，这就造成了浪费。
- 获取某个实现类的方式不够灵活，只能通过 Iterator 形式获取，不能根据某个参数来获取对应的实现类。
- 多个并发多线程使用 ServiceLoader 类的实例是不安全的。





# 参考文档

https://pdai.tech/md/java/advanced/java-advanced-spi.html