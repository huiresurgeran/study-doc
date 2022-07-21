

`Spring Boot`工程中建议将`logback`文件命名为`logback-spring.xml`



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--
scan：属性值为true时，配置文件如果发生改变，将会被重新加载，默认值为false
scanPeriod：检测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒，默认时间间隔是1min，scan为true时生效
debug：属性值为true时，打印出logback内部日志信息，实时查看logback运行状态，默认值为false
-->
<configuration scan="true" scanPeriod="60 second" debug="false">
    <!--属性-->
    <property name="CONSOLE_LOG_PATTERN" value="%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(%5p) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%10.10t]){faint} %clr(%-40.40logger{39}[%L]){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}"></property>

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

    <appender name="ACC-APPENDER" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!--指定日志文件的名称-->
        <file>${LOG_HOME}/acc.log</file>
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

</included>
```