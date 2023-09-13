

# java 连接sqlite

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
