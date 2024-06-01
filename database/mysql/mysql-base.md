## 查看权限

```sql
-- 查看
SHOW GRANTS FOR 'username'@'hostname';
-- 案例：SHOW GRANTS FOR 'root'@'localhost'; 或 SHOW GRANTS FOR 'root';

-- 查看用户级别权限
SELECT * FROM mysql.user WHERE User='username'; -- AND Host='hostname';
-- 查看数据库级别权限
SELECT * FROM mysql.db WHERE User='username'; -- AND Host='hostname';
-- 查看表级别权限
SELECT * FROM mysql.tables_priv WHERE User='username'; -- AND Host='hostname';
```

## 配置权限

### mysql5.7、mysql8.0

> mysql8.0创建用户、授权都需要分开执行（先创建用户、再授权用户）

```sql
===========================================查看用户=====================================================
-- 查看所有用户
select host,user from user;
===========================================创建用户=====================================================
-- 创建用户
-- create user '新用户名'@'%' identified by '密码';   -- 允许所有ip连接（用通配符%表示）
CREATE USER 'my_cust_user'@'%' IDENTIFIED BY 'password密码';
===========================================删除用户=====================================================
-- 删除用户
drop user 'my_cust_user'@'%';
===========================================密码修改=====================================================
-- 密码调整
-- 降低密码复杂度
set global validate_password_policy=LOW;
-- 查看初始化密码
grep password /var/log/mysql.log
-- 修改当前登录用户的密码；
alter user user() identified by '密码';
-- 或
ALTER USER 'my_cust_user'@'%' IDENTIFIED BY '密码'; -- (推荐使用)
	-- 提示因密码问题链接不上数据库时，执行如下: 
	-- Mysql8.0 默认采用 caching-sha2-password 加密,有可能旧的客户端不支持，可改为 mysql_native_password;
	-- alter user 'my_cust_user'@'%' identified with mysql_native_password by '密码';
-- 设置更新密码（不建议使用、mysql8.0）
update mysql.user set authentication_string=password('新密码') where User='my_cust_user' and Host='%';
===========================================授予权限=====================================================
-- 授予所有权限 
-- grant all privileges on 数据库名.表名 to '新用户名'@'指定ip' identified by '用户密码' ;
-- ALL PRIVILEGES 可以替换为 select,insert,delete,update,create,drop
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'xxx' WITH GRANT OPTION;
-- %是指所有的ip地址，也可以换成指定IP
-- xxx是密码
-- 或sql修改： update user set host = "%" where user = "root";
===========================================撤销权限=====================================================
-- 撤销权限 
REVOKE SELECT,INSERT ON testdb.* FROM 'my_cust_user'@'%';
-- 撤销所有权限：revoke all on testdb.* from 'my_cust_user'@'%'; 
-- 或使用  
delete FROM mysql.user Where User='my_cust_user';
===========================================刷新生效=====================================================
-- 刷新生效
flush privileges;
```

## 数据基本信息查询

```sql
-- 数据线程连接数
SHOW STATUS LIKE 'Threads_connected';
-- 查看具体的IP连接
SHOW PROCESSLIST;
SELECT * FROM information_schema.processlist;
-- mysql 查看cpu占用sql
SELECT 
    DIGEST_TEXT, COUNT_STAR, SUM_TIMER_WAIT, AVG_TIMER_WAIT 
FROM 
    performance_schema.events_statements_summary_by_digest 
ORDER BY SUM_TIMER_WAIT DESC  
LIMIT 10;
-- DIGEST_TEXT: SQL语句的文本的摘要。COUNT_STAR: 语句执行的次数。
-- SUM_TIMER_WAIT: 语句执行时消耗的总时间。AVG_TIMER_WAIT: 语句执行的平均时间
```



解除执行SQL【sql】

```sql
SELECT * FROM INFORMATION_SCHEMA.PROCESSLIST ORDER BY TIME DESC;
SHOW FULL PROCESSLIST;
KILL process_id;

-- 查看缓冲区：SHOW VARIABLES LIKE '%buffer%'
-- 查看InnoDB引擎：SHOW ENGINE INNODB STATUS
-- 使用ps命令查看MySQL进程的内存占用情况： ps -p <mysql_pid> -o rss,vsz

-- 查询全局内存使用情况：SHOW VARIABLES LIKE 'innodb_buffer_pool_size';   
								 --  show variables like 'innodb_buffer_pool%';
-- 查询会话内存使用情况： SHOW VARIABLES LIKE 'sort_buffer_size';

-- 判断MySQL使用内存会不会过高: 单位是MB
SELECT ROUND(SUM(index_length) + SUM(data_length), 2)/1024/1024 AS 'Total Memory Usage (MB)'FROM information_schema.TABLES WHERE table_schema NOT IN ('information_schema', 'mysql');
-- 如果这个数值接近你服务器的物理内存，那么可能需要考虑优化内存使用或增加更多的物理内存。
-- 如果你的服务器物理内存有限，并且MySQL的内存使用接近物理内存，那么可能需要调整MySQL的配置，减少innodb_buffer_pool_size、减少max_connections等，或者禁用查询缓存

```

#### 计算缓存命中

```sql
show global status like 'innodb%read%'
```

Innodb_buffer_pool_reads: 表示从物理磁盘读取页的次数

Innodb_buffer_pool_read_ahead: 预读的次数

Innodb_buffer_pool_read_ahead_evicted: 预读的页，但是没有读取就从缓冲池中被替换的页的数量，一般用来判断预读的效率

Innodb_buffer_pool_read_requests: 从缓冲池中读取页的次数

Innodb_data_read: 总共读入的字节数

Innodb_data_reads: 发起读取请求的次数，每次读取可能需要读取多个页

> Innodb缓冲池命中率计算=Innodb_buffer_pool_read_requests/(Innodb_buffer_pool_read_requests+Innodb_buffer_pool_read_ahead+Innodb_buffer_pool_reads)

#### mysql8.0 性能优化配置

> mysql8.0 性能优化配置： https://blog.csdn.net/haveqing/article/details/130358261

```sql
# my.cnf
# innodb缓冲池大小
innodb_buffer_pool_size=1G
# innodb缓冲池块大小
innodb_buffer_pool_chunk_size=128M
# innodb缓冲池实例数
innodb_buffer_pool_instances=8
注意：
	把innodb_buffer_pool_size设置为1G。
	专用服务器可以设为内存70%以上，个人建议innodb_buffer_pool_size设置为系统内存的50%。
	最好设置为：innodb_buffer_pool_size=innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances.
	否则，innodb_buffer_pool_size自动调整可能是innodb_buffer_pool_chunk_size * innodb_buffer_pool_instances的两倍
```



#### 创建临时表

```sql
show global status like '%tmp%';
show global variables like '%tmp_table%';
```

#### buffer刷新使用

```sql
-- 创建一个存储过程来计算客户订单总金额
CREATE PROCEDURE refresh_customer_order_total()
BEGIN
    -- 清除查询缓存中的结果
    FLUSH QUERY CACHE;
    
    -- 重新计算每个客户的订单总金额并缓存结果
    INSERT INTO customer_order_total (customer_id, total_amount)
    SELECT customer_id, SUM(order_amount) FROM orders GROUP BY customer_id;
    
    -- 刷新InnoDB缓冲池
    RESET QUERY CACHE;
END;
```



### MySQL的内存由innodb_buffer_pool和连接内存

MySQL的内存由innodb_buffer_pool和连接内存组成，内存高通常是其中一部分高或者两部分都高：

1.innodb_buffer_pool高：

京东云innodb_buffer_pool分配原则如下:

- 实例内存1G，buffer_pool占用25%
- 实例内存2G，buffer_pool占用50%
- 实例内存4G，buffer_pool占用60%
- 实例内存8G，buffer_pool占用60%
- 实例内存16G ，buffer_pool占用65%,
- 实例内存大于16G，实例内存buffer_pool占用75%。

如果用户实例中热点数据较多，buffer_pool使用就会较高，buffer_pool使用增长内存使用也会随之增长，达到buffer_pool分配限额后存使用会趋于稳定，**直至重启实例buffer_pool占用的内存才会释放**。从监控看，监控项“InnoDB缓存池读命中率、使用率、脏块率 (%)的使用率如果较高，则buffer_pool使用就会较高。

解决方案：优化业务访问模式、优化慢sql、重启实例或者升配规格（用户均可自行操作）

