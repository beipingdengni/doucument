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

