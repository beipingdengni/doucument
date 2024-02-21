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

### 
