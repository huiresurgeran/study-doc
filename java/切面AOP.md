# 介绍

AOP: Aspect Oriented Programming, 面向切面编程

相对的有OOP: Object Oriented Programming,  面向对象编程



在Java平台，对于AOP的织入，有3种方式

- 编译期：编译时，由编译期把切面调用编译进字节码，这种方式需要定义新的关键字并扩展编译器，AspectJ就扩展了Java编译器，使用关键字aspect来实现织入
- 类加载器：在目标类被装载到JVM时，通过一个特殊的类加载器，对目标类的字节码重新“增强”
- 运行期：目标对象和切面都是普通Java类，通过JVM的动态代理功能或者第三方库实现运行期动态织入



Spring的AOP实现是基于JVM的动态代理，即上述第三种方式。

所以其实AOP本质上就是一个动态代理。



# 术语

- Aspect：切面，也称为Advisor，即一个横跨多个核心逻辑的功能，或者称之为系统关注点；可以认为是增强、引入和切入点的组合；在Spring中可以使用Schema和@AspectJ方式进行组织实现；在AOP中表示为“在哪干和干什么集合”；
- Joinpoint：连接点，表示需要在程序中插入横切关注点的扩展点，连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等，Spring只支持方法执行作为连接点，在AOP中表示为“在哪里干”；
- Pointcut：切入点，即一组连接点的集合；Spring支持perl5正则表达式和AspectJ切入点模式，Spring默认使用AspectJ语法，在AOP中表示为“在哪里干的集合”；
- Advice：增强，指特定连接点上执行的动作；增强表示在连接点上执行的行为，增强提供了在AOP中需要在切入点所选择的连接点处进行扩展现有行为的手段；包括前置增强（before advice）、后置增强(after advice)、环绕增强（around advice），在Spring中通过代理模式实现AOP，并通过拦截器模式以环绕连接点的拦截器链织入增强；在AOP中表示为“干什么”；
- Introduction：引介，指为一个已有的Java对象动态地增加新的接口；引介增强是一个比较特殊的增强，它不是在目标方法周围织入增强，而是为目标类创建新的方法或属性，所以引介增强的连接点是类级别的，而非方法级别的，Spring允许引入新的接口（必须对应一个实现）到所有被代理对象（目标对象）, 在AOP中表示为“干什么（引入什么）”；
- Weaving：织入，织入是一个过程，是将切面应用到目标对象从而创建出AOP代理对象的过程，织入可以在编译期、类装载期、运行期进行。
- Interceptor：拦截器，是一种实现增强的方式；
- Target Object：目标对象，即真正执行业务的核心逻辑对象；需要被织入横切关注点的对象，即该对象是切入点选择的对象，需要被增强的对象，从而也可称为“被增强对象”；由于Spring AOP 通过代理模式实现，从而这个对象永远是被代理对象，在AOP中表示为“对谁干”；
- AOP Proxy：AOP代理，是客户端持有的增强后的对象引用。AOP框架使用代理模式创建的对象，从而实现在连接点处插入增强（即应用切面），就是通过代理来对目标对象应用切面。在Spring中，AOP代理可以用JDK动态代理或CGLIB代理实现，而通过拦截器模型应用切面。



连接点和切入点的关系

- 连接点（Join point）：连接点是在应用执行过程中能够插入切面（Aspect）的一个点。这些点可以是调用方法时、甚至修改一个字段时。
- 切点（Pointcut）：切点是指通知（Advice）所要织入（Weaving）的具体位置。
- 每个方法都是**连接点**，但是我们使用的那个方法才是**切点**。每个应用有多个位置适合织入通知，这些位置都是**连接点**。但是只有我们选择的那个具体的位置才是**切点**。



# 实现

实现AOP关键特点是定义好两个角色：切点 和 切面 。



## 依赖

使用AspectJ实现AOP比较方便，因为它的定义比较简单

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
    <version>2.5.6</version>
    <scope>provided</scope>
</dependency>
```



### execution包名/类名匹配

切面类LoggingAspect

`@Aspect`注解：表示它的`@Before`标注的方法需要注入到`WriterService`的每个`public`方法执行前，`@Around`标注的方法需要注入到`WriterService`的每个`public`方法执行前后。

`@Component`注解：表示它本身也是一个Bean。

```
@Aspect
@Component
public class LoggingAspect {

    // 抽取公共的切入点表达式
    // 切入点表达式，指定在哪个地方进行切入
    @Pointcut("execution(public * com.jsamuel.study.WriterService.*(..))")
    public void pointCut() {
    }

    // @Before，前置通知，在目标方法之前切入，在方法执行之前执行的通知
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint) {
        // 获取方法名
        String methodName = joinPoint.getSignature().getName();
        // 获取方法的参数
        List<Object> args = Arrays.asList(joinPoint.getArgs());

        System.out.println("[Before] start log, method: " + methodName + ", args: " + args);
    }

    // @After，后置通知，在目标方法之后切入，在方法执行之后执行的通知，无论是否抛出异常
    // 后置通知中不能访问目标方法执行后返回的结果
    @After("pointCut()")
    public void logEnd(JoinPoint joinPoint) {
        System.out.println("[After] end log ");
    }

    // @Around，环绕通知，在目标方法的前后执行，可以控制是否执行连接点方法
    // 环绕通知，连接点的参数类型必须是ProceedingJoinPoint类型，它是JointPoint的子接口，允许控制什么时候执行，是否执行连接点
    // 环绕通知必须有返回值，返回值即为目标方法的返回值
    @Around("pointCut()")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        System.out.println("[Around] start log ");

        // 执行目标对象方法
        Object retValue = null;
        try {
            retValue = pjp.proceed();
        } catch (Exception e) {
            System.out.println("[Around] log exception:" + e);
            throw new RuntimeException(e);
        }

        System.out.println("[Around] end log ");
        
        return retValue;
    }

    // @AfterReturning，返回通知，在目标方法正常执行返回结果之后执行
    // 返回通知可以获取目标方法的返回值
    // returning属性，用于访问连接点的返回值，该属性的值即为用来传入返回值的参数名称
    @AfterReturning(value = "pointCut()", returning = "obj")
    public void logReturn(JoinPoint joinPoint, Object obj) {
        System.out.println("[After Returning] end log, return value: " + obj);
    }

    // @AfterThrowing，异常通知，在目标方法抛出异常之后执行
    // 异常通知可以访问到异常对象
    // throwing属性，用于访问连接点抛出的异常，也可以指定某种特定的异常类型
    @AfterThrowing(value = "pointCut()", throwing = "exception")
    public void logException(JoinPoint joinPoint, Exception exception) {
        System.out.println("[After Throwing] end log, exception: " + exception);
    }
}
```



目标类WriterService

```java
public class WriterService {

    public void write() {
        System.out.println("write book");
    }

    public void writeException() throws Exception {
        System.out.println("write book error");
        throw new Exception("write book error");
    }
}
```



配置类AopConfig

`@EnableAspectJAutoProxy`注解：开启基于注解的aop模式

Spring的IoC容器看到`@EnableAspectJAutoProxy`注解，就会自动查找带有`@Aspect`的Bean，然后根据每个方法的`@Before`、`@Around`等注解把AOP注入到特定的Bean中。

```java
@Configuration
@EnableAspectJAutoProxy
public class AopConfig {

    // 业务逻辑类加入容器
    @Bean
    public WriterService writerService () {
        return new WriterService();
    }

    // 切面类加入容器
    @Bean
    public LoggingAspect loggingAspect () {
        return new LoggingAspect();
    }
}
```



测试类TestAspect

不能自己创建对象，只有Spring容器中的bean，才能使用Sprint AOP提供的功能

```java
public class TestAspect {

    @Test
    public void testWriterAspect() {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AopConfig.class);

        // 注意：不能自己创建对象，只有Spring容器中的bean，才能使用Sprint AOP提供的功能
        // WriterService writerService = new WriterService();
        // writerService.write();
        WriterService writerService = applicationContext.getBean(WriterService.class);
        writerService.write();

        applicationContext.close();
    }

    @Test
    public void testWriterAspectException() throws Exception {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AopConfig.class);

        WriterService writerService = applicationContext.getBean(WriterService.class);
        writerService.writeException();

        applicationContext.close();
    }
}
```



测试结果

```
// testWriterAspect
[Around] start log 
[Before] start log, method: write, args: []
write book
[After Returning] end log, return value: null
[After] end log 
[Around] end log 

//  testWriterAspectException
[Around] start log 
[Before] start log, method: writeException, args: []
write book error
[After Throwing] end log, exception: java.lang.Exception: write book error
[After] end log 
[Around] log exception:java.lang.Exception: write book error
```



### 注解实现AOP装配

采用包/类名匹配的方式，是在目标类无感知的情况下进行了织入，容易误伤到其他类。

于是，我们可以采用注解的方式，让目标类清楚的知道自己被安排了。



注解MetricTime

```java
@Target(METHOD)
@Retention(RUNTIME)
public @interface MetricTime {

    String value();
}
```



切面类MetricAspect

`@Around("@annotation(metricTime)")`，它的意思是，符合条件的目标方法是带有`@MetricTime`注解的方法。

方法参数除了有ProceedingJoinPoint之外，还有是MetricTime，我们通过它获取注解的名称。

```java
@Aspect
@Component
public class MetricAspect {

    @Around("@annotation(metricTime)")
    public Object metricAround(ProceedingJoinPoint pjp, MetricTime metricTime) throws Throwable {
        String name = metricTime.value();
        long start = System.currentTimeMillis();
        try {
            return pjp.proceed();
        } finally {
            long end = System.currentTimeMillis();
            long time = end - start;
            System.out.println("[Metrics Around] " + name + ": " + time + "ms");
        }
    }
}
```



目标类WriterService

在需要织入的method上，标注该注解

```java
@Component
public class WriterService {
		
  	// 监控writeAnnotation()方法性能
    @MetricTime("write")
    public void writeAnnotation() {
        System.out.println("write book with annotation");
    }
}
```



配置类AopConfig

```java
@Configuration
@EnableAspectJAutoProxy
public class AopConfig {

    // 业务逻辑类加入容器
    @Bean
    public WriterService writerService() {
        return new WriterService();
    }

    // 切面类加入容器
    @Bean
    public MetricAspect metricAspect() {
        return new MetricAspect();
    }
}
```



测试类

```java
public class TestAspect {

    @Test
    public void testWriterAspectAnotation() throws Exception {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(AopConfig.class);

        WriterService writerService = applicationContext.getBean(WriterService.class);
        writerService.writeAnnotation();

        applicationContext.close();
    }
}
```



输出结果

```
write book with annotation
[Metrics Around] write: 6ms
```





# 原理

AOP的源码中用到了两种动态代理来实现拦截切入功能：JDK动态代理和CGLIB动态代理，两种方法同时存在，各有优劣。

- JDK动态代理
  - 是由java内部的反射机制来实现的，通过接口实现
  - 有一定的局限性，JDK动态代理的应用前提，必须是委托类基于统一的接口。如果没有上述前提，jdk动态代理不能应用。
- CGLIB动态代理
  - 底层则是借助asm来实现的，通过生成新的子类，继承目标类。



#### 使用场景

- Spring对接口类型使用JDK动态代理，对普通类使用CGLIB创建子类。
- 如果一个Bean的class是final，Spring将无法为其创建子类。



#### 性能

反射机制在生成类的过程中比较高效，执行时候通过反射调用委托类接口方法比较慢。

而asm在生成类之后的相关代理类执行过程中比较高效（可以通过将asm生成的类进行缓存，这样解决asm生成类过程低效问题）。



#### 结论

CGLIB这种第三方类库实现的动态代理应用更加广泛，且在效率上更有优势。



# 参考文档

https://blog.csdn.net/J080624/article/details/53695064

https://www.liaoxuefeng.com/wiki/1252599548343744/1310052352786466

https://www.liaoxuefeng.com/wiki/1252599548343744/1310052317134882