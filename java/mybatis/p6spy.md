## p6spy

> https://github.com/p6spy/p6spy

## maven

```xml
<dependency>
    <groupId>p6spy</groupId>
    <artifactId>p6spy</artifactId>
    <version>3.9.1</version>
</dependency>
```



## 配置

```yaml
spring:
  datasource:
    driver-class-name: com.p6spy.engine.spy.P6SpyDriver
    host: jdbc:p6spy:mysql://xxxxxx:3306
    url: ${spring.datasource.host}/xxxx?useUnicode=true&characterEncoding=utf8&tinyInt1isBit=false&autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai
    username: xxxxxx
    password: xxxxxx
    initialSize: 5
    minIdle: 5
    maxActive: 20
```



spy.properties

> 官方配置：https://p6spy.readthedocs.io/en/latest/configandusage.html

```properties
#3.2.1以上使用
modulelist=com.baomidou.mybatisplus.extension.p6spy.MybatisPlusLogFactory,com.p6spy.engine.outage.P6OutageFactory
#3.2.1以下使用或者不配置
#modulelist=com.p6spy.engine.logging.P6LogFactory,com.p6spy.engine.outage.P6OutageFactory
# 自定义日志打印
logMessageFormat=com.baomidou.mybatisplus.extension.p6spy.P6SpyLogger
#customLogMessageFormat=%(currentTime)|%(executionTime)|%(category)|connection%(connectionId)|%(sqlSingleLine)
#logMessageFormat=com.tbp.config.P6SpyLogger
#logMessageFormat=com.p6spy.engine.spy.appender.SingleLineFormat


#日志输出到控制台
appender=com.baomidou.mybatisplus.extension.p6spy.StdoutLogger
# 使用日志系统记录 sql
# (default is com.p6spy.engine.spy.appender.FileLogger)
#appender=com.p6spy.engine.spy.appender.Slf4JLogger
#appender=com.p6spy.engine.spy.appender.StdoutLogger
#appender=com.p6spy.engine.spy.appender.FileLogger

# (default is spy.log)
#logfile=spy.log

# 设置 p6spy driver 代理
deregisterdrivers=true
# 取消JDBC URL前缀
useprefix=true
# 配置记录 Log 例外,可去掉的结果集有error,info,batch,debug,statement,commit,rollback,result,resultset.
excludecategories=info,debug,result,commit,resultset
# 日期格式
dateformat=yyyy-MM-dd HH:mm:ss
# 实际驱动可多个
driverlist=com.mysql.cj.jdbc.Driver
#driverlist=org.h2.Driver
# 是否开启慢SQL记录
outagedetection=true
# 慢SQL记录标准 2 秒
outagedetectioninterval=2
```



### 自定义输出日志格式

```java
public class P6SpyLogger implements MessageFormattingStrategy {

    private SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SSS");

    private static final Formatter formatter;

    static {
        formatter = new BasicFormatterImpl();
    }

    @Override
    public String formatMessage(int connectionId, String now, long elapsed, String category, String prepared, String sql, String url) {
        StringBuilder sb = new StringBuilder();
        return !"".equals(sql.trim()) ? sb.append(this.format.format(new Date())).append(" | took ").append(elapsed).append("ms | ").append(category).append(" | connection ").append(connectionId).append(formatter.format(sql)).append(";").toString()  : "";
    }
}
```

