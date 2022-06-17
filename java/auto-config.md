应用场景：实现自定义starter

starter是SpringBoot的核心组成部分。



# 使用方式



## 1. 按



# 如何自动配置bean

auto-configuration通过标准的`@Configuration`类实现。

@Conditional注解用来约束自动配置生效的条件。

通常自动配置类需要使用`@ConditionalOnClass`和`@ConditionalOnMissingBean`注解，主要是为了确保**只有**在相关的类被发现且没有声明自定义@Configuration时才应用自动配置。



# 定位自动配置候选者

Spring boot会检查jar中是否存在META-INF/spring.factories文件，该文件中以EnableAutoConfiguration为key的属性，需要列出相关的配置类



