

# java 连接sqlite

## 官方网站

https://www.sqlite.org/index.html

## java jar包

```xml
<dependency>
  <groupId>org.xerial</groupId>
  <artifactId>sqlite-jdbc</artifactId>
  <version>3.30.1</version>
</dependency>
```



## java连接

```java
url=jdbc:sqlite:./demo.db

SQLiteDataSource dataSource = new SQLiteDataSource();
dataSource.setUrl(url);

spring.datasource.driver-class-name=org.sqlite.JDBC # 数据驱动
spring.datasource.url=jdbc:sqlite:./demo.db  # 数据库地址
spring.datasource.username=
spring.datasource.password=
```



## java 内存

```
SQLiteConnection source = new SQLiteConnection("Data Source=c:\\test.db");
source.Open();
 
using (SQLiteConnection destination = new SQLiteConnection("Data Source=:memory:"))
{
  destination.Open();               
 
  // copy db file to memory
  source.BackupDatabase(destination, "main", "main",-1, null, 0);
  source.Close();
 
  // insert, select ,...        
  using (SQLiteCommand command = new SQLiteCommand())
  {
    command.CommandText = "INSERT INTO t1 (x) VALUES('some new value');";
 
    command.Connection = destination;
    command.ExecuteNonQuery();
  }             
 
  source = new SQLiteConnection("Data Source=c:\\test.db");
  source.Open();
 
  // save memory db to file
  destination.BackupDatabase(source, "main", "main",-1, null, 0);
  source.Close();               
}
```

