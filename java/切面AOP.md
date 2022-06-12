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

- Aspect：切面，即一个横跨多个核心逻辑的功能，或者称之为系统关注点；
- Joinpoint：连接点，即定义在应用程序流程的何处插入切面的执行；
- Pointcut：切入点，即一组连接点的集合；
- Advice：增强，指特定连接点上执行的动作；
- Introduction：引介，指为一个已有的Java对象动态地增加新的接口；
- Weaving：织入，指将切面整合到程序的执行流程中；
- Interceptor：拦截器，是一种实现增强的方式；
- Target Object：目标对象，即真正执行业务的核心逻辑对象；
- AOP Proxy：AOP代理，是客户端持有的增强后的对象引用。



# 实现

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



目标类

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



# 原理



# 参考文档

https://blog.csdn.net/J080624/article/details/53695064

https://www.liaoxuefeng.com/wiki/1252599548343744/1310052352786466



AOP的源码中用到了两种动态代理来实现拦截切入功能：jdk动态代理和cglib动态代理。两种方法同时存在，各有优劣。

 jdk动态代理是由java内部的反射机制来实现的，cglib动态代理底层则是借助asm来实现的。

 总的来说，反射机制在生成类的过程中比较高效，执行时候通过反射调用委托类接口方法比较慢；而asm在生成类之后的相关代理类执行过程中比较高效（可以通过将asm生成的类进行缓存，这样解决asm生成类过程低效问题）。

 还有一点必须注意：jdk动态代理的应用前提，必须是委托类基于统一的接口。如果没有上述前提，jdk动态代理不能应用。

 由此可以看出，jdk动态代理有一定的局限性，cglib这种第三方类库实现的动态代理应用更加广泛，且在效率上更有优势。



实现AOP关键特点是定义好两个角色 切点 和 切面 。

 代理模式中被代理类 委托类处于切点角色，需要添加的其他比如 校验逻辑，事务，审计逻辑 属于非功能实现逻辑通过 切面类定义的方法插入进去。