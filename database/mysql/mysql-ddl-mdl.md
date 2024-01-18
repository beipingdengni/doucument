



```sql
CREATE TABLE if not exists student `students` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `name` varchar(100) NOT NULL COMMENT '姓名',
  `create_pin` varchar(100) NOT NULL COMMENT '创建人',
  `create_time` datetime NOT NULL default CURRENT_TIMESTAMP COMMENT '创建时间',
  `modify_pin` varchar(100) DEFAULT NULL COMMENT '修改人',
  `modify_time` datetime DEFAULT default CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `yn` int DEFAULT '1' COMMENT '记录状态（1:有效，-1:无效）',
   PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci COMMENT='学生联系表'

/*
CHARSET=utf8
不设置的话，会是mysql默认的字符集编码~(不支持中文!)
MySQL的默认编码是Latin1，不支持中文
*/
```



## 索引操作

1、添加主键索引（PRIMARY KEY）

> ALTER TABLE table_name ADD PRIMARY KEY ( column)

2、添加普通索引（INDEX） 

> ALTER TABLE table_name ADD INDEX index_name ( column ) 

3、添加唯一索引（UNIQUE）

> ALTER TABLE table_name ADD UNIQUE (column) 

4、添加全文索引（FULLTEXT）

> ALTER TABLE table_name ADD FULLTEXT ( column) 

5、添加复合索引

> ALTER TABLE table_name ADD INDEX index_name ( column1, column2, column3 )

6、删除索引

> DROP INDEX index_name ON table



## 插入数据

### 检查如果存在key，则更新数据

> ON DUPLICATE KEY UPDATE 

```
insert into (id,name,age) values('1','tian',19)
ON DUPLICATE KEY UPDATE 
name='tian',age='age'
```

