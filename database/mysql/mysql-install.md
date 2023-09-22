

#### 查看数据库基本信息

    show variables like '%version%';

#### 日志设置查看：

    show variables like '%general%';

#### 日志设置 文件样式：

    show variables like '%log_output%';  ##【table or file】

#### 设置环境系统

```
查询 show variables like '%slow_query%'
开启 set global slow_query_log=1

开启通用日志查询： set global general_log=on;

关闭通用日志查询： set globalgeneral_log=off;

设置通用日志输出为表方式： set globallog_output=’TABLE’;

设置通用日志输出为文件方式： set globallog_output=’FILE’;

设置通用日志输出为表和文件方式：set global log_output=’FILE,TABLE’;
```

#### 注意：上述命令只对当前生效，当MySQL重启失效，如果要永久生效，需要配置my.cnf）

select * from mysql.general_log  [慢日志查询]

是否开启慢查询：show variables like '%query%';

查询当前慢查询的语句的个数: show global status like '%slow%';

补充：

```
    perlmysqldumpslow –s c –t 10 slow-query.log

    -s 表示按何种方式排序，c、t、l、r分别是按照记录次数、时间、查询时间、返回的记录数来排序，ac、at、al、ar，表示相应的倒叙；

    -t 表示top的意思，后面跟着的数据表示返回前面多少条；

    -g 后面可以写正则表达式匹配，大小写不敏感

```
