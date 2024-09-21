

博客：

包含binlog查看，及利用binlog数据恢复： https://blog.csdn.net/J080624/article/details/128591262

### 启动bin_log

```ini
[mysqld]
# 启用二进制日志 可自定义文件名也可以加上路径 这个对log_bin_basename log_bin_index有影响
log-bin=mysql-bin
#指定了二进制日志的格式为行级别（row-based）格式。在行级别格式中，二进制日志记录了实际被修改的行的内容。这种格式相对于其他格式（如语句级别或混合级别）更为详细和安全，因为它记录了实际数据的变化，而不仅仅是执行的SQL语句。
binlog_format=row
binlog_expire_logs_seconds=60000
max_binlog_size=300M
#(可添加可不添加)
#添加binlog的多个库(用于区分是不是自己的库)
#binlog-do-db=库名
```

## 使用开源工具

canal2sql:  https://github.com/zhuchao941/canal2sql

```xml
<!-- mysql bin log parse -->
<dependency>
  <groupId>com.alibaba.otter</groupId>
  <artifactId>canal.parse</artifactId>
  <version>1.1.7</version>
</dependency>
```

shyiko: https://github.com/shyiko/mysql-binlog-connector-java

```xml
<dependency>
  <groupId>com.github.shyiko</groupId>
  <artifactId>mysql-binlog-connector-java</artifactId>
  <version>0.13.0</version>
</dependency>
```

maxwell:   https://github.com/zendesk/maxwell

```sql
# 登陆
mysql -uroot -pPassword123$
#权限密码安全策略配置
set global validate_password_length=4
set global validate_password_policy=0;
# 创建数据库
create database maxwell;
# 分配所有权限
grant all on maxwell.* to 'maxwell'@'%' identified by 'Password123$';
# 分配binlog权限
grant select ,replication slave ,replication client on *.* to maxwell@'%';
# 刷新权限
flush privileges;

# cd /user/local/lib/maxwell-1.29.2/
# bin/maxwell --user='maxwell' --password='Password123$' --host='master' --producer=stdout
# --producer 生产者模式(stdout：控制台 kafka：kafka 集群)
# cp config.properties.example config.properties
# vi config.properties
#启动命令
#bin/maxwell --config ./config.properties
#bin/maxwell --user='maxwell' --password='Password123$' --host='master' --producer=kafka --kafka.bootstrap.servers=master:9092 --kafka_topic=maxwell
# kafka查看数据
#./kafka-console-consumer.sh --bootstrap-server master:9092 --topic maxwell --from-beginning
#./kafka-topics.sh --bootstrap-server master:9092 --list
```



## 查看bin_log

```sql
show VARIABLES like '%log_bin%'
-- log_bin_basename : 是binlog日志的基本文件名，后面会追加标识来表示每一个文件
-- log_bin_index：是binlog文件的索引文件，这个文件管理了所有的binlog文件的目录
-- log_bin_trust_function_creators：限制存储过程。前面我们已经讲过了，这是因为二进制日志的一个中药功能是用于主从复制，而存储函数有可能导致主从的数据不一致。所以当开启了二进制日志后，需要限制存储函数的创建、修改、调用。
-- log_bin_use_v1_row_events：此只读系统变量已弃用。ON表示使用版本1二进制日志行，OFF表示使用版本2二进制日志行（MySQL5.6的默认值为2）。

-- 查看：
show binary logs;

-- 直接sql查询
show binlog events [in 'log_name'] [from pos] [limit [offset,] row_count];
-- in ‘log_name’ ：指定要查询的binlog文件名（不指定就是第一个binlog文件）
-- from pos：指定从哪个pos起始点开始查起（不指定就是从整个文件首个pos点开始算）
-- limit [offset]：偏移量（不指定就是0）
-- row_count：查询总条数（不指定就是所有行）

-- 从pos点开始查询100条， from 2000 limit 2,100; 从pos掉开始，偏移2行（即中间跳过2行）查询100条
show binlog events in 'binlog.000011' from 2000 limit 100;

-- 删除bin log
PURGE MASTER LOGS TO "binlog.000013"; # 删除创建时间比binlog.000013早的日志
PURGE MASTER LOGS before "20230105"; # 删除20230105前创建的所有日志文件
RESET MASTER; # 删除所有日志文件

```

### mysqlbinlog 使用

> 注意：使用mysqlbinlog命令进行恢复操作时，必须是编号小的先恢复。例如binlog.000011必须在binlog.000012之前恢复。
>
> 这里有个小细节需要注意的是，假设你要恢复的操作在binlog.000011文件中，那么建议先执行` flush logs;` 这样会新生成一个binlog.000012，对当前数据库的修改操作会记录到新的binlog日志文件中，不会影响binlog.000011。

```sql
-- 查看binlog 离线日志
mysqlbinlog "/var/lib/mysql/binlog.000011"
-- 在MySQL的配置 my.cnf 中将default-character-set=utf8mb4 修改为 character-set-server = utf8mb4
mysqlbinlog --no-defaults -v --base64-output=DECODE-ROWS "/var/lib/mysql/binlog.000011"
-- 查看最后100行
mysqlbinlog  --no-defaults -v --base64-output=DECODE-ROWS "/var/lib/mysql/binlog.000011"|tail -100
-- mysqlbinlog -v /var/lib/mysql/binlog.00001 --start-datetime="2022-05-29 00:00:00"


-- 携带数据库查看
mysqlbinlog [option] filename|mysql -uuser -ppass;
-- filename：是日志文件名
-- option：可选项，比较重要的两对option参数是 --start-datetime、–stop-datetime 和 --start-position、–stop-position。
-- –start-datetime和–stop-datetime：可以指定恢复数据库的起始时间点和结束时间点
-- –start-position和–stop-position：可以指定恢复数据的开始位置和结束位置


-- 假设我们误删除了student表的数据：
-- 此时先执行flush logs;命令新生成一个binlog文件，然后我们查看上一个binlog文件
-- 即，这里我们要读取恢复的是binlog.000015文件。如果我们想依赖于pos进行恢复，可以使用下面语句查看：show binlog events in 'binlog.000015';
-- 数据恢复指令：
-- 基于位置
-- mysqlbinlog --start-position=4 --stop-position=775 --database=testindex /var/lib/mysql/binlog.000015|mysql -uroot -p123456 -v testindex
-- 基于时间恢复
-- mysqlbinlog --start-datetime="2023-01-08 15:31:53" --stop-datetime="2023-01-08 15:32:26" --database=testindex /var/lib/mysql/binlog.000015|mysql -uroot -p123456 -v testindex


```

