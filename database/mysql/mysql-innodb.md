# 数据库结构

![mysql_struct](/Users/tianbeiping1/Documents/study/doucument/database/mysql/imgs/mysql_struct.jpg)





### 并发控制和锁的概念

```
解决并发问题最有效的方案是引入了锁的机制，锁在功能上分为共享锁(shared lock)和排它锁(exclusive lock)即通常说的读锁和写锁。

当一个select语句在执行时可以施加读锁，这样就可以允许其它的select操作进行，因为在这个过程中数据信息是不会被改变的这样就能够提高数据库的运行效率。当需要对数据更新时，就需要施加写锁了，不在允许其它的操作进行，以免产生数据的脏读和幻读。锁同样有粒度大小，有表级锁(table lock)和行级锁(row lock)，分别在数据操作的过程中完成行的锁定和表的锁定

```



#### 事务（ACID）特性

1.  原子性(atomicity):事务中的所有操作要么全部提交成功，要么全部失败回滚。
2. 一致性(consistency):数据库总是从一个一致性状态转换到另一个一致性状态。
3. 隔离性(isolation):一个事务所做的修改在提交之前对其它事务是不可见的。
4.  持久性(durability):一旦事务提交，其所做的修改便会永久保存在数据库中。

#### 事务隔离级别

>  默认MySQL中自动提交是开启的

​    READ UNCOMMITTED(读未提交)：事务中的修改即使未提交也是对其它事务可见
​    READ COMMITTED(读提交)：事务提交后所做的修改才会被另一个事务看见，可能产生一个事务中两次查询的结果不同。
​    REPEATABLE READ(可重读)：只有当前事务提交才能看见另一个事务的修改结果。解决了一个事务中两次查询的结果不同的问题。
​    SERIALIZABLE(串行化)：只有一个事务提交之后才会执行另一个事务

#### 事务基础操作

##### 查看

show variables like 'autocommit'

查询事务隔离级别：show variables like 'tx_isolation'

临时修改、隔离级别：set tx_isolation ='REDA-COMMITTED'

```
set tx_isolation ='REDA-COMMITTED'
start transaction;
	select .....
	update .....
	insert .....
commit;
```

#### MYSQL 存储引擎

##### 存储引擎的介绍：

```
InnoDB 引擎：
    1、讲数据存储在表空间中，表空间由一系列的数据文件组成，由InnoDB 管理
    2、支持每个表的数据和索引存在单独的文件中(innodb_file_per_table)
    3、支持事务,采用mvcc来控制并发，并实现的4个事务隔离级别，支持外键
    4、索引基于聚簇索引建立，对于主键查询有较高性能
    5、数据文件的平台无关性，支持数据在不同的架构平台移植
    6、能够过一些工具支持正真的热备。如XtraBackup等
    7、内部进行自身优化如采取可预测性预读，能够自动在内存中创建hash索引等。

MyISAM引擎：
    1.MySQL5.1中默认，不支持事务和行级锁；
    2.提供大量特性如全文索引、空间函数、压缩、延迟更新等；
    3.数据库故障后，安全恢复性差；
    4.对于只读数据可以忍受故障恢复，MyISAM依然非常适用；
    5.日志服务器的场景也比较适用，只需插入和数据读取操作；
    6.不支持单表一个文件，会将所有的数据和索引内容分别存在两个文件中；
    7.MyISAM对整张表加锁而不是对行，所以不适用写操作比较多的场景；
    8.支持索引缓存不支持数据缓存。

```

>  查看表存储引擎：show table status like '表名称'  \G;
>
> 修改引擎方法:      alter table '表名称' engine = InnoDB

查看innodb状态

```
SHOW ENGINE INNODB STATUS;
```

