# 基础配置

## Mysql server 配置

```
1、修改my.cnf：
[mysqld]

wait_timeout=31536000        
interactive_timeout=31536000  将过期时间修改为1年 
配置解释：MySQL服务器默认的“wait_timeout”是28800秒即8小时(查看mysql实际wait_timeout可以用sql命令:show global variables like ‘wait_timeout’?，意味着如果一个连接的空闲时间超过8个小时，MySQL将自动断开该连接，而连接池却认为该连接还是有效的(因为并未校验连接的有效性)，当应用申请使用该连接时，就会导致上面的报错

```

## mysql 连接配置

> mysql_jdbc jar 包中，可以找配置【com.mysql.cj.conf.PropertyKey】
>
> user 数据库用户名（用于连接数据库）
>
> password 用户密码（用于连接数据库）
>
> useSSL=false 使用ssl加密协议
>
> useUnicode=true 是否使用Unicode字符集，如果参数characterEncoding设置为gb2312或gbk，本参数值必须设置为true
>
> characterEncoding=utf8 当useUnicode设置为true时，指定字符编码。比如可设置为gb2312或gbk
>
> autoReconnect=true 自动重新连接
>
> failOverReadOnly=false   自动重连成功后，连接是否设置为只读
>
> maxReconnects=3    autoReconnect设置为true时，重试连接的次数 3
>
> initialTimeout=2  autoReconnect设置为true时，两次重连之间的时间间隔，单位：秒 2
>
> socketTimeout=0  socket操作（读写）超时，单位：毫秒。 0表示永不超时
>
> connectTimeout=0  和数据库服务器建立socket连接时的超时，单位：毫秒。 0表示永不超时、
>
> serverTimezone=Asia/Shanghai
>
> allowMultiQueries=true
>
> rewriteBatchedStatements 
>
> useServerPrepStmts  和 emulateUnsupportedPstmts  控制是否使用服务端预编译语句
>
> cachePrepStmts  参数就可以控制是否启用缓存【预编译语句】 [可以参考文档](https://www.cnblogs.com/micrari/p/7112781.html)
>
> prepStmtCacheSqlLimit   缓存【预编译语句-长度】
>
> prepStmtCacheSize   缓存【预编译语句-长度】
>

```sql
-- 查询表列名称
show full columns from table_name;
```

### serverTimezone

> URL中添加&serverTimezone=<时区>来设置serverTimezone参数

| 时区             | 时区标识          |
| ---------------- | ----------------- |
| 中国标准时间     | Asia/Shanghai     |
| 美国东部标准时间 | America/New_York  |
| 澳大利亚悉尼时间 | Australia/Sydney  |
| 印度标准时间     | Asia/Kolkata      |
| 英国格林尼治时间 | Europe/London     |
| 加拿大多伦多时间 | America/Toronto   |
| 巴西圣保罗时间   | America/Sao_Paulo |
| 韩国标准时间     | Asia/Seoul        |
| 俄罗斯莫斯科时间 | Europe/Moscow     |

**插入和查询时间数据**：当插入或查询时间数据时，MySQL会自动将其转换为服务器时区的UTC格式。如果不正确设置时区，可能导致数据显示错误。

**时区转换函数**：MySQL提供了一系列函数用于时区转换，如CONVERT_TZ()、DATE_ADD()和DATE_SUB()等。这些函数可以帮助将时间数据从一个时区转换到另一个时区。

```sql
SELECT CONVERT_TZ('2022-01-01 12:00:00', 'UTC', 'America/New_York'); -- 输出：2022-01-01 07:00:00
```

**日期和时间计算**：在计算日期和时间时，时区设置会影响结果。MySQL会自动考虑时区的差异来进行计算。

**存储和显示时间数据**：MySQL存储时间数据时以UTC格式保存，但在显示时会根据时区进行转换。正确设置时区可以确保数据的正确显示



# java 连接

# python 连接

# golang 连接













