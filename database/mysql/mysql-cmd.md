# Mysql 指令基础使用

## 数据库连接

```shell
mysql -h localhost -P3306 -uroot -p123456
```

### 执行sql文件

```
# 进入交互界面执行 source ./test.sql

mysql -uroot -p123456 -D cs_event < ./test.sql
```



### 导出到文件

1. INTO OUTFILE的参数及导出到 csv 文件

2. INTO OUTFILE：「导出文件信息」指定导出的目录、文件名及格式

3. FIELDS TERMINATED BY ：「字段间分隔符」用于定义字段间的分隔符

4. OPTIONALLY ENCLOSED BY： 「字段包围符」定义包围字段的字符

5. LINES TERMINATED BY： 「行间分隔符」定义每行的分隔符

   > 我们选择导出 *.csv 文件格式，然后分隔符用「 , 」字段包围符用「 " 」换行符为「 \n 」

```sql
SELECT id, first_name, last_name,email FROM kalacloud_users
INTO OUTFILE '/tmp/kalacloud_users_out_b.csv'
FIELDS TERMINATED BY ','
OPTIONALLY ENCLOSED BY '"'
LINES TERMINATED BY '\n';
```

### 查询innodb状态

```
SHOW ENGINE INNODB STATUS;
```

### 查看时区

```
show variables like '%time_zone%'
或
select @@system_time_zone,@@time_zone;

设置时区：
部署mysql：/etc/my.cnf 文件，在 [mysqld] 节下增加:  default-time-zone = '+08:00'
连接sql：useSSL=true&&characterEncoding=utf8&serverTimezone=Asia/Shanghai
			serverTimezone=Asia/Shanghai
			serverTimezone=GMT+8 / GMT%2B8
使用命令:(优点:不需要重启MySQLQ服务，缺点:一旦MySQL服务被重启，设置就会消失)
	set global time_zone ='+8:00'; 这个可以修改mysql全局时区为北京时间，也就是我们所在的东8区
	set time_zone = '+8:00'; 修改当前会话时区
	flush privileges;		使之立即生效


参考博客：https://blog.csdn.net/w8y56f/article/details/115473001
```

