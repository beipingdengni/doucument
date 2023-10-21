

Java应该关闭日志打印

方法一

```bash
-Drocketmq.client.logLevel=\"ERROR\" \
```

方法二

```xml
<loggers>
  <root level="INFO">
    <appender-ref ref="RollingFile"/>
    <appender-ref ref="ErrorRollingFile"/>
  </root>
	<!-- 关键是配置 RocketmqClient 为warn -->
	<logger name="RocketmqClient" level="ERROR" additivity="false">
		<appender-ref ref="ErrorRollingFile"/>
	</logger>
</loggers>
```

