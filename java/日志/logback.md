[toc]

`Spring Boot`工程中建议将`logback`文件命名为`logback-spring.xml`



# 概述

日志记录器：Logger，控制要输出哪些日志记录，对日志信息进行级别限制

输出端：Appender，指定日志打印的地方（控制台还是文件），日志打印策略

日志格式化器：Layout，控制日志信息的显示格式



# 组件



## Logger



## Appender

常用Appender

- ConsoleAppender：打印日志信息到控制台，相当于System.out或者System.err
- FileAppender：打印日志信息到文件中
- RollingFillAppender：根据RollingPolicy和TriggeringPolicy将日志打到相应的文件中
  - RollingPolicy，负责滚动
  - TriggeringPolicy，决定是否以及何时进行滚动
  - TimeBasedRollingPolicy比较特殊，它同时继承了RollingPolicy和TriggerPolicy



Appender中还使用了Encoder组件。

Encoder负责

- 把事件转换为字节数组
- 把字节数组写入输出流

`patternLayoutEncoder`是最常使用encoder。





## Layout

仅将事件转成字符串。

写入到输出流，依赖writer。

推荐使用encoder。



# 参数



## 日志格式

```
%d{yyyy-MM-dd HH:mm:ss.SSS}|%-5level|%X{tid}|%thread|%logger{36}.%M:%L-%msg%n
```

- `·%d{yyyy-MM-dd HH:mm:ss.SSS}`：时间，`d{pattern}/date{pattern}`，打印时间

- `level` ： 日志级别，p/le/level，都是代表日志级别
- `%X{tid}`： %X用于输出和当前线程相关联的MDC，在代码中给MDC添加key-value，即可增加新值
  - %X：输出所有值
  - %X{key}：输出key对应的value，没有默认值
  - %X{key:-aaa}：输出key对应的value，默认值为aaa
- `%thread`： 线程，t/thread
- `%logger{36}` ：c{length}/lo{length}/logger{length}，输出logger的名字，一般是class的全名，花括号代表最长字符数，比如36代表最多36个字符
- `%M` ：方法名，M/method
- `%L` ：行号，L/line
- `%msg` ： 输出信息，m/msg/message
- `%n` ： 换行



```
%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(%5p) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%10.10t]){faint} %clr(%-40.40logger{39}[%L]){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}
```

- `%clr()`，配置不同的颜色输出，支持的颜色有：blue，cyan青色，faint，green，magenta洋红，red，yellow
- `highlight()`，颜色，info为蓝色，warn为浅红，error为加粗红，debug为黑色
- `%d{yyyy-MM-dd HH:mm:ss.SSS}`，时间
- `%5p`，日志级别
- `${PID:- }`，进程ID号
- `%clr(---){faint}`，用"---"分隔符分开
- `[%10.10t]`，线程名称，括在方括号中
- `%-40.40logger{39}`，日志的名字，一般是类名，39代表最长字符
- `[%L]`，行号
- `%m` ： 输出信息，m/msg/message
- `%n` ： 换行
- `-%wEx`：记录异常时使用的转换字



格式参数说明

### 格式说明

| Format modifier | Left justify | Minimum width | Maximum width | Comment                                                      |
| --------------- | ------------ | ------------- | ------------- | ------------------------------------------------------------ |
| %20logger       | false        | 20            | none          | Left pad with spaces if the logger name is less than 20 characters long. |
| %-20logger      | true         | 20            | none          | Right pad with spaces if the logger name is less than 20 characters long. |
| %.30logger      | NA           | none          | 30            | Truncate from the beginning if the logger name is longer than 30 characters. |
| %20.30logger    | false        | 20            | 30            | Left pad with spaces if the logger name is shorter than 20 characters. However, if logger name is longer than 30 characters, then truncate from the beginning. |
| %-20.30logger   | true         | 20            | 30            | Right pad with spaces if the logger name is shorter than 20 characters. However, if logger name is longer than 30 characters, then truncate from the *beginning*. |
| %.-30logger     | NA           | none          | 30            | Truncate from the *end* if the logger name is longer than 30 characters. |





## configuration

- scan：属性值为true时，配置文件如果发生改变，将会被重新加载，默认值为false
- scanPeriod：检测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒，默认时间间隔是1min，scan为true时生效
- debug：属性值为true时，打印出logback内部日志信息，实时查看logback运行状态，默认值为false

```xml
<configuration scan="true" scanPeriod="60 second" debug="false">
```



## contextName

上下文名称

默认上下文名称为"default

用于区分不同应用程序的记录

```xml
<!--
上下文名称
每个logger都关联到logger上下文，默认上下文名称为"default"
用于区分不同应用程序的记录
一旦设置，不能修改
-->
<contextName>${APP_NAME}</contextName>
```



## property

设置变量/属性

有两个属性，name和value

- name的值是变量的名称
- value的值是变量定义的值

通过`<property>`定义的值会被插入到logger上下文中

定义变量后，可以使`${}`来使用变量

```xml
<!--
    设置变量/属性
    <property> 有两个属性，name和value
    name的值是变量的名称，value的值是变量定义的值
    通过<property>定义的值会被插入到logger上下文中。定义变量后，可以使"${}"来使用变量
-->
<property name="APP_NAME" value="jsamuel-study-logback"></property>
<property name="CONSOLE_LOG_PATTERN" value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(%5p) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%10.10t]){faint} %clr(%-40.40logger{39}[%L]){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"></property>

<!-- 设置默认值 -->
<property name="FILE_MAX_SIZE" value="${LOG_FILE_MAX_SIZE:-100MB}" />
```



## timestamp

获取时间戳字符串

- key：标识此`<timestamp> `的名字
- datePattern：设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式

```xml
<!--
获取时间戳字符串
key，标识此<timestamp> 的名字
datePattern：设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式
-->
<timestamp key="bySecond" datePattern="yyyy-MM-dd'T'HH:mm:ss"></timestamp>
```



## springProfiler

`<springProfile>`，读取spring.profiles.active设置的值，用于设置不同环境的不同逻辑

- 固定值: `<springProfile name="dev">`，spring.profiles.active是dev时生效

- 或: `<springProfile name="dev | test">`，spring.profiles.active是dev或者test时生效
- 非: `<springProfile name="!dev">`，spring.profiles.active不是dev时生效



## include

引入其他日志配置文件

```xml
<include resource="logback-file-appender.xml"/>	
```



## root

root与logger是父子关系，没有特别定义的类，则默认属于root。

任何一个类只会和一个logger对应，要么是logger，要么是root。

匹配顺序是，先找到logger，再找这个logger的appender和level。



参数

- `level="INFO`，日志级别指定为INFO
- `appender-ref`，支持配置多个appender，ref是appender的名字
  - CONSOLE，输出到控制台

```xml
<!--root日志输出-->
<!--root与logger是父子关系，没有特别定义的类，则默认属于root-->
<!--任何一个类只会和一个logger对应，要么是logger，要么是root-->
<!--先找到logger，再找这个logger的appender和level-->
<!--日志级别指定为INFO-->
<root level="INFO">
  	<!--支持配置多个appender-->
  	<!--具体是否能将日志appender，取决level是否匹配-->
  	<appender-ref ref="APP-DEBUG-APPENDER"/>
  	<appender-ref ref="APP-INFO-APPENDER"/>
  	<appender-ref ref="APP-WARN-APPENDER"/>
  	<appender-ref ref="APP-ERROR-APPENDER"/>
</root>

<include resource="org/springframework/boot/logging/logback/console-appender.xml" />
<root level="INFO">
  	<!-- 打印到控制台 -->
  	<appender-ref ref="CONSOLE"/>
</root>
```



## logger

logger，存放日志对象，可以定义日志类型，级别，会先执行。



参数

- name，表示匹配的logger类型前缀，可以是包名，也可以是某个logger的名称，private static final Logger accLogger = LoggerFactory.getLogger("acc");
- level，要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR
  - 没有设置level，继承上级root的level
- additivity，children-logger是否使用root-logger配置的appender进行输出，即是否将打印信息向上级传递
  - additivity，false，表示只用当前logger的appender-ref，logger的打印信息不再向上级传递，只会打印一次
  - additivity，true，默认值，表示既使用当前logger的appender-ref，也使用root的，即logger的打印信息向上级传递，会打印多次
- appender-ref，指定使用的appender
  - 没有设置appender，则此logger不会打印任何信息

```xml
<!--ACC日志打印-->
<!--logger，存放日志对象，可以定义日志类型，级别，会先执行-->
<!--name，表示匹配的logger类型前缀，可以是包名，也可以是某个logger的名称，private static final Logger accLogger = LoggerFactory.getLogger("acc"); -->
<!--level，要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR-->
<!--additivity，children-logger是否使用root-logger配置的appender进行输出，即是否将打印信息向上级传递-->
<!--additivity，false，表示只用当前logger的appender-ref，logger的打印信息不再向上级传递，只会打印一次-->
<!--additivity，true，默认值，表示既使用当前logger的appender-ref，也使用root的，即logger的打印信息向上级传递，会打印多次-->
<logger name="acc" level="INFO" additivity="false">
  	<appender-ref ref="ACC-APPENDER"/>
</logger>

<!--没有设置level，继承上级root的level-->
<!--没有设置appender，则此logger不会打印任何信息-->
<!--additivity默认值是true，表示logger的打印信息向上级传递，直接传递到root，使用root logger的appender-ref-->
<logger name="com.tencent.polaris"/>
```



## appender



参数

- `name`：appender名称
- `class`：appender类型
- `file`：指定输出文件路径&文件名
- `filter`：日志过滤器，支持配置一个或多个，当满足过滤器指定条件时，才记录日志
  - class：指定filter类型
    - LevelFilter：对指定level的日志进行记录(或不记录)，对不等于指定level的日志不记录(或进行记录)
    - ThresholdFilter：对大于或等于指定level的日志进行记录(或不记录)，对小于指定level的日志不记录(或进行记录)
    - EvaluatorFilter：对满足指定表达式的日志进行记录(或不记录)，对不满足指定表达式的日志不作记录(或进行记录)
    - MDCFilter：若MDC域中存在指定的key-value，则进行记录，否者不作记录
    - DuplicateMessageFilter：根据配置不记录多余的重复的日志
    - DynamicThresholdFilter：动态版的ThresholdFilter，根据MDC域中是否存在某个键，该键对应的值是否相等，可实现日志级别动态切换
    - MarkerFilter：针对带有指定标记的日志，进行记录(或不作记录)
  - level：日志级别
  - OnMismatch：日志级别不匹配
  - OnMatch：日志级别匹配
  - FilterReply有三种枚举值
    - DENY：表示不用看后面的过滤器了，这里就给拒绝了，不作记录
    - NEUTRAL：表示需不需要记录，还需要看后面的过滤器。若所有过滤器返回的全部都是NEUTRAL，那么需要记录日志
    - ACCEPT：表示不用看后面的过滤器了，这里就给直接同意了，需要记录
- `rollingPolicy`：发生滚动时，决定RollingFileAppender的行为，包括文件移动和文件重命名
  - `class`：rolling类型
    - TimeBasedRollingPolicy，根据时间制定滚动策略，负责滚动&发出滚动
    - SizeBasedRollingPolicy，根据大小制定滚动策略
    - SizeAndTimeBasedRollingPolicy，以上两者结合体
  - `fileNamePattern`：滚动时产生的文件的存放位置及文件名称
    - %d{yyyy-MM-dd-HH}，按天进行日志滚动
    - %i，按大小进行日志滚动，文件大小超过maxFileSize时，按照i进行文件滚动
  - `cleanHistoryOnStart`：是否在启动的时候清理历史日志，默认false
  - `maxFileSize`：控制保留的单个归档文件的最大容量
  - `maxHistory`：控制保留的归档文件的最大数量，超出则删除旧文件
  - `totalSizeCap`：限制总的日志文件的大小
- `encoder`：格式化输出
  - pattern：格式化
  - charset：编码格式
- `layout`：日志输出格式，通过pattern指定



```xml
<appender name="ACC-INFO-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
  	<!--指定日志文件的名称-->
  	<file>${LOG_HOME}/acc.log</file>
  	
  	<!-- 日志过滤器，支持配置一个或多个，当满足过滤器指定条件时，才记录日志 -->
    <!--
        FilterReply有三种枚举值：
        1. DENY：表示不用看后面的过滤器了，这里就给拒绝了，不作记录
        2. NEUTRAL：表示需不需要记录，还需要看后面的过滤器。若所有过滤器返回的全部都是NEUTRAL，那么需要记录日志
        3. ACCEPT：表示不用看后面的过滤器了，这里就给直接同意了，需要记录
    -->
  	<filter class="ch.qos.logback.classic.filter.LevelFilter">
      		<!--日志级别-->		
      		<level>INFO</level>
      		<!--日志级别不匹配，拒绝-->
      		<OnMismatch>DENY</OnMismatch>
      		<!--日志级别匹配，接收-->
      		<OnMatch>ACCEPT</OnMatch>
  	</filter>
  	<!--发生滚动时，决定RollingFileAppender的行为，包括文件移动和文件重命名
        	TimeBasedRollingPolicy，根据时间制定滚动策略，负责滚动&发出滚动
        	SizeBasedRollingPolicy，根据大小制定滚动策略
        	SizeAndTimeBasedRollingPolicy，以上两者结合体-->
  	<rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
    		<!--滚动时产生的文件的存放位置及文件名称
            		%d{yyyy-MM-dd-HH}，按天进行日志滚动
            		%i，按大小进行日志滚动，文件大小超过maxFileSize时，按照i进行文件滚动-->
    		<fileNamePattern>${LOG_HOME}/acc.%d{yyyy-MM-dd-HH}.%i.log</fileNamePattern>
    		<!--是否在启动的时候清理历史日志，默认false-->
    		<cleanHistoryOnStart>${CLEAN_HISTORY_ON_START}</cleanHistoryOnStart>
        <!--当日志文件超过maxFileSize指定的大小是，根据上面提到的%i进行日志文件滚动-->
    		<maxFileSize>${FILE_MAX_SIZE}</maxFileSize>
    		<!--控制保留的归档文件的最大数量，超出则删除旧文件-->
    		<maxHistory>${FILE_MAX_HISTORY}</maxHistory>
    		<!--限制总的日志文件的大小-->
    		<totalSizeCap>${FILE_TOTAL_SIZE_CAP}</totalSizeCap>
  	</rollingPolicy>
  	<encoder>
    		<pattern>%msg %n</pattern>
    		<charset>UTF-8</charset>
  	</encoder>
  	<!--日志输出格式-->
  	<layout class="ch.qos.logback.classic.PatternLayout">
   		 <pattern>${ACC_LOG_PATTERN}</pattern>
  	</layout>
</appender>
```



## AsyncAppender



参数

- name：appender名称
- class：appender类型
- discardingThreshold：抛弃日志阈值
  - 默认情况下，当队列剩余容量小于总容量的20%时，日志级别<=INFO的内容将不会打印，只会打印WARN、ERROR级别的内容
  - discardingThreshold =0时，表示剩余容量在小于0%的时候，日志级别<=INFO的内容才不会打印；但剩余容量最少也只会等于0%，不会小于0%，所以在此情况下所有日志级别都会打印。
  - discardingThreshold =256时，表示剩余容量在小于100%的时候，日志级别<=INFO的内容不会打印；所以此情况下只会打印WARN、ERROR级别的内容。
- queueSize：队列容量配置
  - Logback使用BlockingQueue存放业务线程的打印内容，AsyncAppender会从此队列中取内容
  - 队列默认大小为256
- appender-ref：
  - AsyncAppender中只能配置一个appender-ref，否则无效

```xml
<!-- 日志异步输出 -->
<appender name="ASYNC-ACC" class="ch.qos.logback.classic.AsyncAppender">
  	<!--抛弃日志阈值
        默认情况下，当队列剩余容量小于总容量的20%时，日志级别<=INFO的内容将不会打印，只会打印WARN、ERROR级别的内容-->
  	<discardingThreshold>0</discardingThreshold>
  	<!--队列容量配置，默认值256-->
  	<queueSize>256</queueSize>
  	<!--AsyncAppender中只能配置一个appender-ref，否则无效-->
  	<appender-ref ref="ACC-APPENDER"/>
</appender>
```





总配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
scan：属性值为true时，配置文件如果发生改变，将会被重新加载，默认值为false
scanPeriod：检测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒，默认时间间隔是1min，scan为true时生效
debug：属性值为true时，打印出logback内部日志信息，实时查看logback运行状态，默认值为false
-->
<configuration scan="true" scanPeriod="60 second" debug="false">
    <!--
    上下文名称
    每个logger都关联到logger上下文，默认上下文名称为"default"
    用于区分不同应用程序的记录
    一旦设置，不能修改
    -->
    <contextName>${APP_NAME}</contextName>

    <!--
    设置变量/属性
    <property> 有两个属性，name和value
    name的值是变量的名称，value的值是变量定义的值
    通过<property>定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量
    -->
    <property name="APP_NAME" value="jsamuel-study-logback"></property>
    <property name="CONSOLE_LOG_PATTERN" value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(%5p) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%10.10t]){faint} %clr(%-40.40logger{39}[%L]){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"></property>
     <!-- 设置默认值 -->
     <property name="FILE_MAX_SIZE" value="${LOG_FILE_MAX_SIZE:-100MB}" />

    <!--
    获取时间戳字符串
    key，标识此<timestamp> 的名字
    datePattern：设置将当前时间（解析配置文件的时间）转换为字符串的模式，遵循java.txt.SimpleDateFormat的格式
    -->
    <timestamp key="bySecond" datePattern="yyyy-MM-dd'T'HH:mm:ss"></timestamp>

    <!-- run application from command line: @see assembly/bin/start.sh -->
    <!-- 脚本执行命令有 exec=$(getExec "&#45;&#45;spring.profiles.active=cli" ${@}) -->

    <!--springProfile，读取spring.profiles.active设置的值，用于设置不同环境的不同逻辑-->
    <!--固定值: <springProfile name="dev">，spring.profiles.active是dev时生效-->
    <!--或: <springProfile name="dev | test">，spring.profiles.active是dev或者test时生效-->
    <!--非: <springProfile name="!dev">，spring.profiles.active不是dev时生效-->
    <springProfile name="cli">
        <!--include，引入其他日志配置文件-->
        <include resource="logback-file-appender.xml"/>

        <!--root日志输出-->
        <!--root与logger是父子关系，没有特别定义的类，则默认属于root-->
        <!--任何一个类只会和一个logger对应，要么是logger，要么是root-->
        <!--先找到logger，再找这个logger的appender和level-->
        <!--日志级别指定为INFO-->
        <root level="INFO">
            <!--支持配置多个appender-->
            <!--具体是否能将日志appender，取决level是否匹配-->
            <appender-ref ref="APP-DEBUG-APPENDER"/>
            <appender-ref ref="APP-INFO-APPENDER"/>
            <appender-ref ref="APP-WARN-APPENDER"/>
            <appender-ref ref="APP-ERROR-APPENDER"/>
        </root>

        <!--ACC日志打印-->
        <!--logger，存放日志对象，可以定义日志类型，级别，会先执行-->
        <!--name，表示匹配的logger类型前缀，可以是包名，也可以是某个logger的名称，private static final Logger accLogger = LoggerFactory.getLogger("acc"); -->
        <!--level，要记录的日志级别，包括 TRACE < DEBUG < INFO < WARN < ERROR-->
        <!--additivity，children-logger是否使用root-logger配置的appender进行输出，即是否将打印信息向上级传递-->
        <!--additivity，false，表示只用当前logger的appender-ref，logger的打印信息不再向上级传递，只会打印一次-->
        <!--additivity，true，默认值，表示既使用当前logger的appender-ref，也使用root的，即logger的打印信息向上级传递，会打印多次-->
        <logger name="acc" level="INFO" additivity="false">
            <appender-ref ref="ACC-APPENDER"/>
        </logger>

        <!--APP日志打印-->
        <logger name="com.jsamuel.study.logback" level="DEBUG" additivity="false">
            <appender-ref ref="APP-DEBUG-APPENDER"/>
            <appender-ref ref="APP-INFO-APPENDER"/>
            <appender-ref ref="APP-WARN-APPENDER"/>
            <appender-ref ref="APP-ERROR-APPENDER"/>
        </logger>
    </springProfile>

    <springProfile name="!cli">
        <root level="INFO">
            <!-- 打印到控制台 <appender-ref ref="STDOUT"/> -->
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>

    <!-- APP日志打印 -->

    <!--没有设置appender，则此logger不会打印任何信息-->
    <!--additivity默认值是true，表示logger的打印信息向上级传递，直接传递到root，使用root logger的appender-ref-->
    <logger name="${root-package-name}" level="DEBUG" />

    <!--没有设置level，继承上级root的level-->
    <!--没有设置appender，则此logger不会打印任何信息-->
    <!--additivity默认值是true，表示logger的打印信息向上级传递，直接传递到root，使用root logger的appender-ref-->
    <logger name="com.tencent.polaris"/>

</configuration>
```





Logback-file-appender.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<included>
    <property name="LOG_HOME" value="${APP_LOGS_PATH}"/>
    <property name="LOG_PATTERN" value="${FILE_LOG_PATTERN}" />
    <property name="CLEAN_HISTORY_ON_START" value="${LOG_FILE_CLEAN_HISTORY_ON_START:-false}" />
    <property name="FILE_MAX_SIZE" value="${LOG_FILE_MAX_SIZE:-100MB}" />
    <property name="FILE_MAX_HISTORY" value="${LOG_FILE_MAX_HISTORY:-14}" />
    <property name="FILE_TOTAL_SIZE_CAP" value="${LOG_FILE_TOTAL_SIZE_CAP:-0}" />

    <property name="ACC_LOG_PATTERN" value="%d{yyyy-MM-dd HH:mm:ss.SSS}  %-5level  [%-15.15(%thread)]  %-50.50(%logger{50}) :  %msg%n"/>

    <!-- ACC 日志-->

    <appender name="ACC-INFO-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--指定日志文件的名称-->
        <file>${LOG_HOME}/acc.log</file>
        <!-- 日志过滤器，支持配置一个或多个，当满足过滤器指定条件时，才记录日志 -->
        <!--
        FilterReply有三种枚举值：
        1. DENY：表示不用看后面的过滤器了，这里就给拒绝了，不作记录
        2. NEUTRAL：表示需不需要记录，还需要看后面的过滤器。若所有过滤器返回的全部都是NEUTRAL，那么需要记录日志
        3. ACCEPT：表示不用看后面的过滤器了，这里就给直接同意了，需要记录
        -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!--日志级别-->
            <level>INFO</level>
            <!--日志级别不匹配，拒绝-->
            <OnMismatch>DENY</OnMismatch>
            <!--日志级别匹配，接收-->
            <OnMatch>ACCEPT</OnMatch>
        </filter>
        <!--发生滚动时，决定RollingFileAppender的行为，包括文件移动和文件重命名
        TimeBasedRollingPolicy，根据时间制定滚动策略，负责滚动&发出滚动
        SizeBasedRollingPolicy，根据大小制定滚动策略
        SizeAndTimeBasedRollingPolicy，以上两者结合体-->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!--滚动时产生的文件的存放位置及文件名称
            %d{yyyy-MM-dd-HH}，按天进行日志滚动
            %i，按大小进行日志滚动，文件大小超过maxFileSize时，按照i进行文件滚动-->
            <fileNamePattern>${LOG_HOME}/acc.%d{yyyy-MM-dd-HH}.%i.log</fileNamePattern>
            <!--是否在启动的时候清理历史日志，默认false-->
            <cleanHistoryOnStart>${CLEAN_HISTORY_ON_START}</cleanHistoryOnStart>
            <!--当日志文件超过maxFileSize指定的大小是，根据上面提到的%i进行日志文件滚动-->
            <maxFileSize>${FILE_MAX_SIZE}</maxFileSize>
            <!--控制保留的归档文件的最大数量，超出则删除旧文件-->
            <maxHistory>${FILE_MAX_HISTORY}</maxHistory>
            <!--限制总的日志文件的大小-->
            <totalSizeCap>${FILE_TOTAL_SIZE_CAP}</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%msg %n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!--日志输出格式-->
        <layout class="ch.qos.logback.classic.PatternLayout">
            <pattern>${ACC_LOG_PATTERN}</pattern>
        </layout>
    </appender>

    <!-- 日志异步输出 -->
    <appender name="ASYNC-ACC" class="ch.qos.logback.classic.AsyncAppender">
        <!--抛弃日志阈值
        默认情况下，当队列剩余容量小于总容量的20%时，日志级别<=INFO的内容将不会打印，只会打印WARN、ERROR级别的内容-->
        <discardingThreshold>0</discardingThreshold>
        <!--队列容量配置，默认值256-->
        <queueSize>256</queueSize>
        <!--AsyncAppender中只能配置一个appender-ref，否则无效-->
        <appender-ref ref="ACC-APPENDER"/>
    </appender>

</included>
```



# 使用

`Spring Boot`工程默认使用resources目录下的`logback-spring.xml`

也可以通过application.yaml配置文件指定

```yaml
logging:
	config: classpath:config/logback.xml
```





# 参考文档

https://www.cnblogs.com/chrischennx/p/6781574.html