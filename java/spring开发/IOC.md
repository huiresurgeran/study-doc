[toc]

# 容器的概念

容器：为某种特定组件的运行提供必要支持的一个软件环境，同时还提供了许多底层服务

如：

- Tomcat是一个Servlet容器
- Docker是一个容器，提供了必要的Linux环境



Spring的核心，就是提供了一个IoC容器，可以管理所有轻量级的JavaBean组件，提供的底层服务包括组件的生命周期管理，配置和组装服务，AOP支持，以及建立在AOP基础上的声明式事务服务等。



# IoC原理

Spring提供的容器，又称为IoC容器。

IoC: Inversion of Control, 控制反转



## 为什么叫IoC

传统的应用程序中，控制权在程序本身，程序的控制流程完全由开发者控制：

- 创建MovieService，需要先创建DataSource组件
- 创建DataSource，需要先创建DataConfig组件
- 一个组件如果需要使用另一个组件，必须先要知道如何正确的创建它



在IOC模式下，控制权发生反转，从应用程序转移到了IOC容器

- 所有组件不再由应用程序自己创建和配置，而是由IOC容器负责

- 应用程序只需要直接使用已经创建好并且配置好的组件，无需再进行创建和配置

- 为了让组件在IOC容器中能够被装配出来，需要用`注入机制`

  - `BookService`自己并不会创建`DataSource`，而是等待外部通过`setDataSource()`方法来注入一个`DataSource`

  - ```
    public class BookService {
        private DataSource dataSource;
    
        public void setDataSource(DataSource dataSource) {
            this.dataSource = dataSource;
        }
    }
    ```



## IOC的作用

IOC又被称为依赖注入（DI: dependency injection）

- 将组件的创建和配置（创建并且装配Bean），与组建的使用分离开来
- 由IOC容器负责统一管理组件的生命周期





## 如何实现IOC

因为IOC容器需要负责实例化所有的组件，所以需要告诉容器，如何创建组件，以及组件之间的依赖关系。

在Spring的IoC容器中，我们把所有组件统称为JavaBean，即配置一个组件就是配置一个Bean。



### 1. XML文件

```xml
<beans>
    <bean id="dataSource" class="HikariDataSource" />
    <bean id="movieService" class="MovieService">
        <property name="dataSource" ref="dataSource" />
    </bean>
    <bean id="sportService" class="SportService">
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
```

- 指示IOC容器创建3个JavaBean
- 把id为`dataSource`的组件，通过属性`dataSource`注入到另外两个组件中（即调用SetDataSource()方法）



### 2. 通过注解自动注入

```java
@Component
public class UserService {
  	@Autowired
    MailService mailService;

    public UserService(@Autowired MailService mailService) {
        this.mailService = mailService;
    }
    ...
}
```

- `@Component`注解相当于定义了一个Bean，它有一个可选的名称，默认是`userService`，即小写开头的类名。
- 使用`@Autowired`相当于把指定类型的Bean注入到指定的字段中。和XML配置相比，`@Autowired`大幅简化了注入，因为它不但可以写在`set()`方法上，还可以直接写在字段上，甚至可以写在构造方法中：



## 依赖注入的方式

Spring的IOC容器，同时支持属性注入和构造方法注入两种模式，并且允许混合使用



### 1. 属性注入set()

```java
public class MovieService {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```





### 2. 构造方法注入

```java
public class MovieService {
		private DataSource dataSource;
	
		public void MovieService(DataSource dataSource) {
				this.dataSource = dataSource;
		}
}
```



# 装配Bean



## 依赖

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
</dependency>
```



## 创建service



### 1. MailService

通过setCount，注入变量int count

```java
public class MailService {

    private ZoneId zoneId = ZoneId.systemDefault();
    private int count = 0;

    public void setZoneId(ZoneId zoneId) {
        this.zoneId = zoneId;
    }

    public void setCount(int count) {
        this.count = count;
    }

    private static final Logger logger = LoggerFactory.getLogger(MailService.class);

    public void sendLoginMail() {
        logger.info(String.format("Hi, %s! You are logged in at %s, count is %d",
                "totoroyang", getTime(), count));
    }

    public void sendRegisterMail() {
        logger.info(String.format("Welcome, %s!, count is %d", "totoroyang", count));
    }

    public String getTime() {
        return ZonedDateTime.now(this.zoneId).format(DateTimeFormatter.ISO_ZONED_DATE_TIME);
    }
}
```



### 2. UserService

通过setMailService，注入一个MailService

```java
public class UserService {

    private static final Logger logger = LoggerFactory.getLogger(UserService.class);

    private MailService mailService;

    public void setMailService(MailService mailService) {
        this.mailService = mailService;
    }

    public void login() {
        logger.info(">>> login");
        mailService.sendLoginMail();
    }

    public void register() {
        logger.info(">>> register");
        mailService.sendRegisterMail();
    }

}
```



## 编写application.xml文件

application.xml配置文件，用于告诉Spring的IOC容器，应该如何创建并且组装Bean。

Spring容器通过读取XML文件后，使用反射完成Bean注入。

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="userService" class="com.jsamuel.study.ioc.service.UserService">
        <property name="mailService" ref="mailService">
        </property>
    </bean>

    <bean id="mailService" class="com.jsamuel.study.ioc.service.MailService">
        <property name="count" value="10"></property>
    </bean>
</beans>
```



### 1. bean配置

id：Bean的唯一ID

class：对应full class name（package + class）



每个bean中可以有多个property配置，用于注入另一个bean或者普通数据

每个property对应class中的一个参数



### 2. property配置

name：对应class代码中的setXxx()

ref：对应bean ID

value：常量值



可以支持注入bean和注入普通数据类型

- bean
  - name + ref
  - ref可以将对应的bean通过set方法注入
- 普通数据类型（boolean，int，String）
  - name + value
  - value可以将常量值通过set方法注入



## 自动装配&使用bean

```java
public static void main(String[] args) {
        // 1. 创建Spring的IOC容器实例
        // 2. 加载配置文件
        // 3. 让Spring容器创建并且装配好，配置文件中指定的所有的Bean
        ApplicationContext context = new ClassPathXmlApplicationContext("application.xml");

        // 从Spring容器中取出装配好的Bean
        // 根据Bean的类型
        UserService userService = context.getBean(UserService.class);
        // 根据Bean的ID
        UserService userService1 = (UserService) context.getBean("userService");

        userService.login();
        userService.register();

        userService1.login();
        userService1.register();
    }
```



### ApplicationContext

Spring容器就是`ApplicationContext`，它是一个接口，有很多实现类。

我们在上述代码中使用的是`ClassPathXmlApplicationContext`，表示他会自动从classpath中查找指定的XML配置文件，进行bean的装配。



### 从Spring容器中获取Bean

从`ApplicationContext`中可以获取Bean。

支持多种方式获取，包括根据Bean的ID，根据Bean的类型等等

```java
// 根据Bean的类型
UserService userService = context.getBean(UserService.class);

// 根据Bean的ID
UserService userService1 = (UserService) context.getBean("userService");
```



# 使用Annotation配置

使用XML配置

- 优点：所有Bean都能清除地列出来，并通过配置注入能直观地看到每个Bean的依赖
- 缺点：写起来非常繁琐，每增加一个组件，就必须把新的Bean配置到XML中



为了解决繁琐的问题，产生了更加简单的配置方式：Annotation配置。

不需要XML，通过Annotiaon，让Spring自动扫描bean并且组装他们。



使用Annotation配置自动扫描功能，能简化Spring的配置

- 每个Bean需要被标注为`@Component`
- 需要注入Bean的地方需要被标注为`@Autowired`
- 配置类需要被标注为`@Configuration`和`@ComponentScan`
- 所有Bean需要在指定的包或者子包内



## @Component

在Service类上添加`@Component`注解。

`@Component`注解相当于定义了一个Bean，它有一个可选参数value，默认是小写开头的类名。



在下面代码示例中，默认是`mailService`，即小写开头的类名。

```java
@Component
public class MailService {
  ...
}
```



## @Autowired

把Service类中需要做自动注入的字段，增加`@Autowired`注解。

```java
@Component
public class UserService {

    @Autowired
    private MailService mailService;
  
  	...
}
```

`@Autowired`相当于把指定类型的Bean注入到指定的字段中。



`@Autowired`大幅简化了注入，它可以写在

- `set()`方法上

  - ```java
    @Autowired
    private MailService mailService;
    ```

- 字段上

  - ```java
    @Autowired
    private MailService mailService;
    ```

- 构造方法中

  - ```java
    public UserService(@Autowired MailService mailService) {
      	this.mailService = mailService;
    }
    ```



通常我们把`@Autowired`注解写在字段上。



## AppConfig类启动容器

```java
@Configuration
@ComponentScan
public class AppConfig {

    public static void main(String[] args) {
      	// 独立的应用上下文
        // 创建Spring的IOC容器实例
        // 接收带注解的类作为输入
        // 让Spring容器创建并且装配好，配置文件中指定的所有的Bean
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
      
        UserService userService = context.getBean(UserService.class);
        userService.login();
        userService.register();
    }
}
```



### 1. @Configuration

`@Configuration`用于定义配置类，可以替换XML配置文件。



`使用位置`

@Configuration 作用于类上，相当于一个xml配置文件。



`要求`

- @Configuration不可以是final类型；
- @Configuration不可以是匿名类；
- 嵌套的configuration必须是静态类。
- 被注解的类内部，需要一个或多个被@Bean注解的方法（@Bean 作用于方法上，相当于xml配置中的<bean>）



### 2. @ComponentScan

@ComponentScan告诉容器，自动搜索当前类所在的包以及子包，把所有标注为`@Component`的Bean自动创建出来，并且根据`@Autowired`进行装配。



value参数，可以支持指定要扫描的包

```java
@ComponentScan(value = "io.mieux.controller")
public class BeanConfig {

}
```



excludeFilters 和 includeFilters参数，按规则排除/加入某些包的扫描

```java
@ComponentScan(value = "io.mieux",
        excludeFilters = {@Filter(type = FilterType.ANNOTATION,
        value = {Controller.class})})
public class BeanConfig {

}
```

excludeFilters 的参数是一个 Filter[] 数组

- type：可以指定FilterType的类型，在这里是用的ANNOTATION，也就是通过注解来进行过滤
- value：指定类名，在这里是Controller注解类

配置之后，spring扫描时，会跳过io.mieux包下，所有被@Controller注解标注的类。



 includeFilters ，按照规则只包含某些包的扫描

```java
@ComponentScan(value = "io.mieux", 
        includeFilters = {@Filter(type = FilterType.ANNOTATION, classes = {Controller.class})},
        useDefaultFilters = false)
public class BeanConfig {

}
```

用法和excludeFilters 一致。

多了一个参数`useDefaultFilters`，默认是true也就是spring默认会自动发现`@Component`，`@Repository`，`@Service`，`@Controller`标准的类，并且注册进容器中。



### 3. @AnnotationConfigApplicationContext



在main函数中使用spring，有两个常见的高级容器

- ClassPathXmlApplicationContext，基于classpath下的xml配置文件
- AnnotationConfigApplicationContext，基于java配置文件





`AnnotationConfigApplicationContext`是一个独立的应用上下文。

- 它接收带注解的类作为输入，比如`@Configuration`或者`@Component`。

- 他可以通过`scan`查找bean，也可以通过`register`注册bean。





# 参考文档

https://www.liaoxuefeng.com/wiki/1252599548343744/1282382145519649