# Mysql自带数据库备份

## 第一种方式：load（导入、导出）

使用案例

```sql
// 导出
select * from mytable where status!=0 and name!='' into outfile '/home/db_bak2012' fields terminated by '|' enclosed by '"' lines terminated by '\r\n' ;

// 导入
load data infile  '/home/db_bak2012' into table mytable_bak fields terminated by '|' enclosed by '"' lines terminated by '\r\n' ;
```

使用注意：

1、关键词low_priority：如果你指定关键词low_priority， 那么MySQL将会等到没有其他人读这个表的时候，才把插入数据。可以使用如下的命令

```sql
load data low_priority infile "/home/mark/data sql" into table Orders;
```

2、如果指定local关键词，则表明从客户主机读文件。如果local没指定，文件必须位于服务器上

3、replace和ignore关键词控制对现有的唯一键记录的重复的处理。如果你指定replace，新行将代替有相同的唯一键值的现有行。如果你指定ignore，跳过有唯一键的现有行的重复行的输入。如果你不指定任何一个选项，当找到重复键时，出现一个错误，并且文本文件的余下部分被忽略。例如：

```sql
load data low_priority infile "/home/mark/data sql" replace into table Orders;
```

### 语法

```sql
load data  [low_priority] [local] infile 'file_name txt' [replace | ignore]
 into table tbl_name
 [fields]
 [terminated by 't']
 [OPTIONALLY] enclosed by '']
 [escaped by'\' ]]
 [lines terminated by'n']
 [ignore number lines]
 [(col_name,   )]
 
terminated by	分隔符：意思是以什么字符作为分隔符
enclosed by	字段括起字符
escaped by	转义字符 
terminated by 	描述字段的分隔符，默认情况下是tab字符（\t） 
enclosed by	描述的是字段的括起字符。
escaped by	描述的转义字符。默认的是反斜杠（backslash：\ ）   
 
例如：load data infile "/home/mark/Orders txt" replace into table Orders fields terminated by',' enclosed by '"';
```

## 第二种方式：source（导入）

>  source /root/pro_sql/test.sql （source后边为sql文件存放位置）



## 第三种方式：mysqldump（导出）

### 数据导出

> mysqldump -u <用户名> -p<密码> <数据库名> > <输出文件路径> 

使用案例

```shell
# demo数据库备份
mysqldump -h 127.0.0.1 -P3306 -u root -p123456 demo > /tmp/demo.sql
# mysqldump --user=username --password=password --host=host-ip --port=3306 --databases database-name > mysqldb.sql

# 全量备份
# mysqldump --single-transaction --all-databases > snapshot.sql
mysqldump -uroot -p123456 -F -B it --default-character-set=utf8 --single-transaction -e | gzip > /root/mysql_back_`date +%F`.sql.gz

出现告警：If you don't want to restore GTIDs, pass --set-gtid-purged=OFF. To make a complete dump, pass --all-databases --triggers --routines --events
脚本中加入 --set-gtid-purged=off 或者–gtid-mode=OFF
```

### 数据导入

```sql
mysql -uroot -p123456 -D cs_event < ./test.sql
# mysql cs_event < snapshot.sql
```

#### 导出到文件

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

## 快照备份和恢复

## 备份

```sql
-- 使用快照数据库
USE snapshot_db;

-- 复制快照到备份文件
FLUSH TABLES WITH READ LOCK; -- 再次锁定所有数据库的表，以确保在备份数据的过程中不会有其他写入操作。
SHOW MASTER STATUS; -- 再次显示当前的二进制日志文件和位置，以确保备份文件和快照点一致
system mysqldump --user=<username> --password=<password> snapshot_db > snapshot_backup.sql
UNLOCK TABLES; -- 解锁所有数据库的表，以恢复正常的写入操作

# 备份单个库
mysqldump -uroot -p -R -E --single-transactio --databases [database_one] > database_one.sql
# 备份部分表
mysqldump -uroot -p --single-transaction [database_one] [table_one] [table_two] > database_table12.sql
# 排除某些表
mysqldump -uroot -p [database_one] --ignore-table=[database_one.table_one] --ignore-table=[database_one.table_two] > database_one.sql
# 只备份结构
mysqldump -uroot -p [database_one] --no-data > [database_one.defs].sql
# 只备份数据
mysqldump -uroot -p [database_one] --no-create-info > [database_one.data].sql
```

### 恢复

```sql
# mysql -uroot -p123456 < /data/mysqlDump/mydb.sql
# mysql> source /data/mysqlDump/mydb.sql

-- 使用快照数据库
USE snapshot_db;

-- 清空快照表，以便恢复到快照点时可以重新插入数据
TRUNCATE TABLE snapshot;

-- 使用备份文件还原数据库
SOURCE snapshot_backup.sql;
```

