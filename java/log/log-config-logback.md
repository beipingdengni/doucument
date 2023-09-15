## 配置文件





```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <property name="PROJECT" value="app-demo"/>
    <property name="FILESIZE" value="500MB"/>
    <property name="MAXHISTORY" value="100"/>
    <property name="LOG_HOME" value="./logs" />
    <timestamp key="DATETIME" datePattern="yyyy-MM-dd HH:mm:ss"/>
    <!-- 控制台打印 -->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %logger{36} %line - %m%n</pattern>
        </encoder>
    </appender>
    <!-- ERROR 输入到文件，按日期和文件大小 -->
    <appender name="ERROR-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %logger{36} %line - %m%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/%d/error.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <maxFileSize>${FILESIZE}</maxFileSize>
            <!--<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"/>-->
        </rollingPolicy>
    </appender>
    <!-- WARN 输入到文件，按日期和文件大小 -->
    <appender name="WARN-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %logger{36} %line - %m%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>WARN</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/%d/warn.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <maxFileSize>${FILESIZE}</maxFileSize>
            <!--<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"/>-->
        </rollingPolicy>
    </appender>
    <!-- INFO 输入到文件，按日期和文件大小 -->
    <appender name="INFO-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %logger{36} %line - %m%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>INFO</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/%d/info.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <maxFileSize>${FILESIZE}</maxFileSize>
            <!--<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"/>-->
        </rollingPolicy>
    </appender>
    <!-- DEBUG 输入到文件，按日期和文件大小 -->
    <appender name="DEBUG-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %logger{36} %line - %m%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>DEBUG</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/%d/debug.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <maxFileSize>${FILESIZE}</maxFileSize>
            <!--<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"/>-->
        </rollingPolicy>
    </appender>
    <!-- TRACE 输入到文件，按日期和文件大小 -->
    <appender name="TRACE-OUT" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <encoder>
            <pattern>[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %logger{36} %line - %m%n</pattern>
        </encoder>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>TRACE</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${LOG_HOME}/%d/trace.%i.log</fileNamePattern>
            <maxHistory>${MAXHISTORY}</maxHistory>
            <maxFileSize>${FILESIZE}</maxFileSize>
            <!--<timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy"/>-->
        </rollingPolicy>
    </appender>

		<!-- Logger 业务配置 -->
    <logger name="com.keruyun.bigdata.*">
        <level value="INFO"/>
    </logger>

    <!-- Logger 根目录 -->
    <root level="INFO">
        <appender-ref ref="STDOUT"/>
        <appender-ref ref="ERROR-OUT"/>
        <appender-ref ref="WARN-OUT"/>
        <appender-ref ref="INFO-OUT"/>
        <appender-ref ref="DEBUG-OUT"/>
        <appender-ref ref="TRACE-OUT"/>
    </root>
</configuration>
```