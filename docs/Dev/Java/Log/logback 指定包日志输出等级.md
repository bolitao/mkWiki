# logback 指定包日志输出等级

``` xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/app.%d{yyyy-MM-dd}.log.zip</fileNamePattern>
            <maxHistory>10</maxHistory>
        </rollingPolicy>

        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger - %msg%n</pattern>
        </encoder>
    </appender>
    
    <appender name="QUEUE" class="ch.qos.logback.classic.AsyncAppender">  
        <discardingThreshold>0</discardingThreshold>  
          <queueSize>10000</queueSize>  
          <appender-ref ref="FILE" />  
    </appender>

    <root level="info">
        <appender-ref ref="QUEUE"/>
        <appender-ref ref="STDOUT"/>
    </root>

    <Logger name="com.xxx.xxx" level="debug" additivity="false">
        <appender-ref ref="QUEUE"/>
        <appender-ref ref="STDOUT"/>
    </Logger>
</configuration>
```

refer:

- [Log4j2指定包的日志级别 - 简书](https://www.jianshu.com/p/e6f6bd8665bc)
