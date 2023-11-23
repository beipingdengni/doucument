





### Spring Boot自定义日志配置

#### 加载文件顺序

>  logback.xml—>application.properties—>logback-spring.xml
>
> logging.config=classpath:logback-spring.xml
>
> 或
>
> logging.config= file:./config/logback-spring.xml

| Spring 环境变量                     | 系统变量<br/>（优先级高级Spring环境变量) | 说明                                     | 默认                                                         |
| ----------------------------------- | ---------------------------------------- | ---------------------------------------- | ------------------------------------------------------------ |
| logging.exception-conversion-word   | LOG_EXCEPTION_CONVERSION_WORD            | 异常日志转换字                           | %wEx                                                         |
| logging.file.clean-history-on-start | LOG_FILE_CLEAN_HISTORY_ON_START          | 启动时清理归档日志                       | FALSE                                                        |
| logging.file.name                   | LOG_FILE                                 | 日志文件名称，可以是相对路径或者绝对路径 | ${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/spring.log} |
| logging.file.path                   | LOG_PATH                                 | 日志文件路径                             |                                                              |
| logging.file.max-size               | LOG_FILE_MAX_SIZE                        | 日志文件最大容量（仅对logback有效）      | 10MB                                                         |
| logging.file.max-history            | 归档日志最大保留时间                     | 启动时清理归档日志                       | 7                                                            |
| logging.file.total-size-cap         | LOG_FILE_TOTAL_SIZE_CAP                  | 归档日志总计最大容量                     | 0                                                            |
| logging.pattern.console             | CONSOLE_LOG_PATTERN                      | 控制台日志格式                           | ${CONSOLE_LOG_PATTERN:-%clr(%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd  HH:mm:ss.SSS}}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:-  }){magenta} %clr(---){faint} %clr([%15.15t]){faint}  %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}} |
| logging.pattern.dateformat          | LOG_DATEFORMAT_PATTERN                   | Appender日期格式                         | yyyy-MM-dd HH:mm:ss.SSS                                      |
| logging.pattern.file                | FILE_LOG_PATTERN                         | 归档日志正则                             | ${FILE_LOG_PATTERN:-%d{${LOG_DATEFORMAT_PATTERN:-yyyy-MM-dd  HH:mm:ss.SSS}} ${LOG_LEVEL_PATTERN:-%5p} ${PID:- } --- [%t] %-40.40logger{39}  : %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}} |
| logging.pattern.level               | LOG_LEVEL_PATTERN                        | 日志级别                                 | %5p                                                          |
| logging.pattern.rolling-file-name   | ROLLING_FILE_NAME_PATTERN                | 日志文件分片名称规则                     | ${LOG_FILE}.%d{yyyy-MM-dd}.%i.gz                             |
| PID                                 | PID                                      | 当前进程                                 |                                                              |



### 注意

> xml中变量默认值按logback的语法： ${name`:-`value}
> yml或者properties中使用默认值要按[spel表达式](https://so.csdn.net/so/search?q=spel表达式&spm=1001.2101.3001.7020) ${name`:`value}，没有`-`



### 参考配置

#### 演示1

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- 日志级别从低到高分为TRACE < DEBUG < INFO < WARN < ERROR < FATAL，如果设置为WARN，则低于WARN的信息都不会输出 -->
<!-- scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true -->
<!-- scanPeriod:设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。 -->
<!-- debug:当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false。 -->
<configuration scan="true" scanPeriod="60 seconds" debug="false">

    <contextName>logback</contextName>
    <!-- 
			name的值是变量的名称，value的值时变量定义的值。
			通过定义的值会被插入到logger上下文中。定义变量后，可以使“${}”来使用变量
			LOG_PATH是来自，spring 的配置文件中
		-->
    <property name="log.path" value="${LOG_PATH:-.}"></property>
    <property name="log.path" value="./log" />

    <!-- 彩色日志插件 -->
    <conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule conversionWord="wEx" converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>

    <!-- 控制台彩色日志格式，注意：%L打印行号对性能有影响，因此不建议在生产环境使用。 -->
    <property name="CONSOLE_LOG_PATTERN" value="%clr([%d{HH:mm:ss}){faint} %clr(%p) %clr(%.10t]){faint} %clr(%C{39}){cyan} %clr(%M:%L){magenta}: %m%n%wEx"/>
    <!-- 输出到文件日志格式 -->
    <property name="FILE_LOG_PATTERN" value="[%d{MM-dd HH:mm:ss} %p %.10t] %C{39}\.%M\\(\\): %m%n"/>

    <!--输出到控制台-->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!-- 时间滚动输出 level为 INFO 日志 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/log_info.log</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>50GB</totalSizeCap>
          	<!-- 默认：false -->
          	<!--<cleanHistoryOnStart>true</cleanHistoryOnStart>-->
        </rollingPolicy>
        <!--
            此日志文件只记录debug级别的
            onMatch和onMismatch都有三个属性值，分别为Accept、DENY和NEUTRAL
            onMatch="ACCEPT" 表示匹配该级别及以上
            onMatch="DENY" 表示不匹配该级别及以上
            onMatch="NEUTRAL" 表示该级别及以上的，由下一个filter处理，如果当前是最后一个，则表示匹配该级别及以上
            onMismatch="ACCEPT" 表示匹配该级别以下
            onMismatch="NEUTRAL" 表示该级别及以下的，由下一个filter处理，如果当前是最后一个，则不匹配该级别以下的
            onMismatch="DENY" 表示不匹配该级别以下的
        -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 WARN 日志 -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/log_warn.log</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/warn/log-warn-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>20GB</totalSizeCap>
          	<!-- 默认：false -->
          	<!--<cleanHistoryOnStart>true</cleanHistoryOnStart>-->
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>


    <!-- 时间滚动输出 level为 ERROR 日志 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/log_error.log</file>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/error/log-error-%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>50GB</totalSizeCap>
          	<!-- 默认：false -->
          	<!--<cleanHistoryOnStart>true</cleanHistoryOnStart>-->
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!--root配置必须在appender下边，root日志级别比appender高，如果：root是ERROR，appender是INFO，那么就是ERROR-->
    <!-- spring.profiles.active选择那个环境 -->
    <!-- 开发环境 -->
    <springProfile name="dev">
        <root level="INFO">
            <appender-ref ref="STDOUT" />
        </root>
    </springProfile>

    <!-- 生产环境 -->
    <springProfile name="prod">
        <root level="ERROR">
            <appender-ref ref="ERROR_FILE" />
        </root>
    </springProfile>

    <!-- 测试环境 -->
    <springProfile name="test">
        <root level="INFO">
            <appender-ref ref="STDOUT" />
            <appender-ref ref="INFO_FILE" />
            <appender-ref ref="ERROR_FILE" />
            <appender-ref ref="WARN_FILE" />
        </root>
    </springProfile>

</configuration>
```

#### 演示2

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <!-- 应用名称 -->
    <springProperty scope="context" name="springApplicationName" source="spring.application.name" defaultValue="spring"/>
    <!-- 环境 -->
    <springProperty scope="context" name="springProfilesActive" source="spring.profiles.active" defaultValue="prod"/>
    <!-- 重启后是否删除日志 -->
    <springProperty scope="context" name="cleanHistoryOnStart" source="logging.logback.rollingpolicy.clean-history-on-start" defaultValue="false"/>
    <!-- 单个日志文件的大小 -->
    <springProperty scope="context" name="maxFileSize" source="logging.logback.rollingpolicy.max-file-size" defaultValue="10MB"/>
    <!-- 日志总文件最大值 -->
    <springProperty scope="context" name="totalSizeCap" source="logging.logback.rollingpolicy.total-size-cap" defaultValue="50GB"/>
    <!-- 日志保留时长 (天) -->
    <springProperty scope="context" name="maxHistory" source="logging.logback.rollingpolicy.max-history" defaultValue="30"/>
    <!-- 日志文件路径 -->
    <springProperty scope="context" name="path" source="logging.file.path" defaultValue="./log/${springApplicationName}/${springProfilesActive}"/>

    <contextName>${springApplicationName}</contextName>

    <property name="PATTERN_FILE" value="[%d{MM-dd HH:mm:ss} %p %.10t] %C{39}\.%M\\(\\): %m%n"/>

    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${path}/%d{yyyy-MM-dd}/%i.log</fileNamePattern>
            <totalSizeCap>${totalSizeCap}</totalSizeCap>
            <maxHistory>${maxHistory}</maxHistory>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>${maxFileSize}</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!-- 重启项目后删除日志 -->
            <cleanHistoryOnStart>${cleanHistoryOnStart}</cleanHistoryOnStart>
        </rollingPolicy>
        <encoder>
            <pattern>${PATTERN_FILE}</pattern>
        </encoder>
    </appender>

    <root level="ERROR">
        <appender-ref ref="FILE" />
    </root>

</configuration>
```

