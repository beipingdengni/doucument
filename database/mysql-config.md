Mysql server 配置

```
1、修改my.cnf：
[mysqld]

wait_timeout=31536000        
interactive_timeout=31536000  将过期时间修改为1年 
配置解释：MySQL服务器默认的“wait_timeout”是28800秒即8小时(查看mysql实际wait_timeout可以用sql命令:show global variables like ‘wait_timeout’?，意味着如果一个连接的空闲时间超过8个小时，MySQL将自动断开该连接，而连接池却认为该连接还是有效的(因为并未校验连接的有效性)，当应用申请使用该连接时，就会导致上面的报错

```

mysql 连接配置

> user 数据库用户名（用于连接数据库）
>
> password 用户密码（用于连接数据库）
>
> useUnicode 是否使用Unicode字符集，如果参数characterEncoding设置为gb2312或gbk，本参数值必须设置为true
>
> characterEncoding 当useUnicode设置为true时，指定字符编码。比如可设置为gb2312或gbk
>
> autoReconnect=true 自动重新连接
>
> failOverReadOnly=false   自动重连成功后，连接是否设置为只读
>
> maxReconnects autoReconnect设置为true时，重试连接的次数 3
>
> initialTimeout autoReconnect设置为true时，两次重连之间的时间间隔，单位：秒 2
>
> socketTimeout socket操作（读写）超时，单位：毫秒。 0表示永不超时
>
> connectTimeout 和数据库服务器建立socket连接时的超时，单位：毫秒。 0表示永不超时

