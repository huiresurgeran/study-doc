

`Spring Boot`工程中建议将`logback`文件命名为`logback-spring.xml`



```xml
<?xml version="1.0" encoding="UTF-8" ?>
<configuration>
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
        <!--name，可以是包名，也可以是某个logger的名称，private static final Logger accLogger = LoggerFactory.getLogger("acc"); -->
        <!--additivity，false，表示loger的打印信息不再向上级传递，只会打印一次-->
        <!--additivity，true，默认值，表示loger的打印信息向上级传递，会打印多次-->
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
    <!--additivity默认值是true，表示loger的打印信息向上级传递，直接传递到root-->
    <logger name="${root-package-name}" level="DEBUG" />

    <!-- 没有设置level，继承上级root的level -->
    <logger name="com.tencent.polaris"/>

</configuration>
```