[toc]

应用场景：实现自定义starter

starter是SpringBoot的核心组成部分。



# 构建starter流程



## 添加依赖

创建starter项目，我们不需要创建SpringBoot项目，创建一个Maven项目即可。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-autoconfigure</artifactId>
</dependency>
```



## 配置映射参数实体

依赖`@ConfigurationProperties`注解。

该注解可以完成，将application.yml配置文件内的参数，映射到实体内的field中，不过需要提供setter方法，代码如下：

```java
@ConfigurationProperties(prefix = "movie")
@Setter
public class MovieProperties {

    /**
     * 类型
     */
    private String type = "";

    /**
     * 名称
     */
    private String name = "";

    /**
     * 上映年份
     */
    private int year = 0;

    /**
     * 是否喜欢
     */
    private boolean like = false;
}
```

`@ConfigurationProperties`注解

1. `prefix`属性，通过指定的前缀，绑定配置文件中对应的配置
2. 该注解可以放在类上，也可以放在方法上
3. 注解作用于方法上时，需要该方法有`@Bean`注解，并且所属的Class需要`@Configuration`注解
4. 配置文件不进行配置时，读取代码中的默认值



## 编写自定义业务Service

我们给starter提供一个Service，用于实现业务功能，比如下面的toString

```java
@Getter
@Setter
public class MovieService {

    /**
     * 类型
     */
    private String type = "";

    /**
     * 名称
     */
    private String name = "";

    /**
     * 上映年份
     */
    private int year = 0;

    /**
     * 是否喜欢
     */
    private boolean like = false;

    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("type: ").append(type)
                .append(", name: ").append(name)
                .append(", year: ").append(year)
                .append(", like: ").append(like);
        return sb.toString();
    }
}
```



## 实现自动化配置

在这里我们需要完成：实体bean的验证和初始化

auto configuration类

- application配置文件映射前缀实体对象：MovieProperties，可自动注入
- 创建业务Service的实体bean：public MovieService movieService()，带上@Bean注解

通过配置文件的映射参数实体对象，初始化业务Service的bean，实现自动化配置。

```java
// 开启配置：用于定义配置类，可替换xml配置文件
// 被注解的类的内部，包含一个或多个@Bean注解的方法，这些方法会被Context扫描，用于构建bean定义，初始化Spring容器
// 要求：不可以是final类型，不可以是匿名类，嵌套的configuration必须是静态类
@Configuration
// 开启使用映射实体对象
// 使标注了@ConfigurationProperties注解的类生效，即将标注了@ConfigurationProperties注解的类注入进spring容器
@EnableConfigurationProperties(MovieProperties.class)
// 当Spring Ioc容器内存在指定Class的条件
// 存在MovieService class时初始化该配置
@ConditionalOnClass(MovieService.class)
// 指定的属性是否有指定的值，存在对应配置信息时初始化该配置类
@ConditionalOnProperty(
        // 存在配置前缀movie
        prefix = "movie",
        // 开启，只要配置的值是非false都有效
        value = "enabled",
        // 缺失检查，如果该属性为true，表示name或者value没有配置对应属性时也能注入
        matchIfMissing = true
)
public class MovieAutoConfiguration {

    private static final Logger logger = LoggerFactory.getLogger(MovieAutoConfiguration.class);

    // application配置文件映射前缀实体对象
    @Autowired
    private MovieProperties movieProperties;

    // 创建MovieService实体bean
    @Bean
    // 当SpringIoc容器内不存在指定Bean的条件
    // 缺少MovieService实体bean时，初始化MovieService，并添加到Spring Ioc中
    @ConditionalOnMissingBean(MovieService.class)
    public MovieService movieService() {
        logger.info(">>> MovieService not found, create new MovieService bean.");
        MovieService movieService = new MovieService();
        movieService.setType(movieProperties.getType());
        movieService.setName(movieProperties.getName());
        movieService.setYear(movieProperties.getYear());
        movieService.setLike(movieProperties.isLike());
        return movieService;
    }
}
```



`@Configuration`：

- 该注解表示一个类声明了一个或多个拥有`@Bean`注解的方法，并且这些拥有`@Bean`的方法的返回对象，将会被Spring容器管理，在其他类中可以用`@Autowired`注解注入这些类
- 用于定义配置类，可替换xml配置文件，表示开启配置
- 要求：不可以是final类型，不可以是匿名类，嵌套的configuration必须是静态类



`@EnableConfigurationProperties(MovieProperties.class)`

- 开启使用配置参数
- 作用：使标注了@ConfigurationProperties注解的类生效，即将标注了@ConfigurationProperties注解的类注入进spring容器
- value值：配置实体参数映射的ClassType，会将配置实体作为配置的来源



`@ConditionOnXxx`

@Conditional注解用来约束自动配置生效的条件。通常自动配置类需要使用`@ConditionalOnClass`和`@ConditionalOnMissingBean`注解，主要是为了确保**只有**在相关的类被发现且没有声明自定义@Configuration时才应用自动配置。

- @ConditionalOnClass(MovieService.class)

  - 作用：当Spring Ioc容器内存在指定Class时，configuration进行注入
  - 即存在MovieService class时初始化该配置

- @ConditionalOnProperty()

  - 作用：通过配置文件中的属性值/是否存在，来判断configuration是否需要被注入

  - 支持参数: prefix, value, matchIfMissing

    - prefix：配置文件中key的前缀，可以和value/name属性组合使用
    - value/name作用一致：当value对应配置文件中的值为false时，注入不生效；不为false则注入生效
    - havingValue：和value/name组合使用，当value或者name对应的值，与havingValue的值相同时，注入生效
    - matchIfMissing：该属性为true时，配置文件中缺少对应的value/name的对应的属性值，也会注入成功，默认值false

  - ```java
    @ConditionalOnProperty(
            // 存在配置前缀movie
            prefix = "movie",
            // 只要配置的值是非false都有效
            value = "enabled",
            // 缺失检查，如果该属性为true，表示name或者value没有配置对应属性时也能注入
            matchIfMissing = true
    )
    ```

-  @ConditionalOnMissingBean(MovieService.class)

  - 当Spring Ioc容器内不存在指定Bean时，configuration进行注入
  - 当缺少MovieService实体bean时，初始化MovieService，并添加到Spring Ioc中

- @ConditionalOnBean

- @ConditionalOnMissingClass



## 自定义spring.factories

`@EnableAutoConfiguration`注解内，使用了`SpringFactoriesLoader.loadFactoryNames`方法，扫描有`META-INF/spring.factories`文件的jar包。



`spring.factories`文件配置格式：

- Key => Value
- 多个Value时，使用逗号`,`隔开
- 换行使用斜杠`\`
- Key: `org.springframework.boot.autoconfigure.EnableAutoConfiguration`
- value：auto configuration full class name（package + class）



文件路径：`src/main/resource/META-INF/`



示例

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.jsamuel.study.autoconfigure.MovieAutoConfiguration,\
  com.jsamuel.study.autoconfigure.SportAutoConfiguration
```



这样我们也就完成了一个starter的编写，接下来是创建项目进行测试



# 创建测试starter项目



## 依赖

包括两种类型

- 刚刚编写的starter包
- spring-boot相关包

```xml
<dependency>
    <groupId>com.jsamuel.study</groupId>
    <artifactId>study-spring-boot-starter</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```



## 配置文件application.yml

```yaml
debug: true

server:
  port: 9001

# movie配置，对应properties
movie:
  type: happy
  name: jsamuel
  year: 2022
  like: true
```



## 启动服务

日志中可以看到，starter已经生效，并且根据@ConditionalOnXxx条件，将MovieService实体bean注入到Spring Ioc容器中。

```
>>> MovieService not found, create new MovieService bean.
...
MovieAutoConfiguration matched:
      - @ConditionalOnClass found required class 'com.jsamuel.study.service.MovieService' (OnClassCondition)
      - @ConditionalOnProperty (movie.enabled) matched (OnPropertyCondition)

   MovieAutoConfiguration#movieService matched:
      - @ConditionalOnMissingBean (types: com.jsamuel.study.service.MovieService; SearchStrategy: all) did not find any beans (OnBeanCondition)
```



## 编写测试控制类

```java
@RestController
public class MovieController {
	
  	// 注入自定义starter
    @Autowired
    private MovieService movieService;

    @RequestMapping(value = "/movie")
    public String movie() {
        return movieService.toString();
    }
}
```



测试输出结果

```
"type: happy, name: jsamuel, year: 2022, like: true"
```





# 参考文档

https://segmentfault.com/a/1190000011433487

https://blog.csdn.net/u010142437/article/details/115732223

