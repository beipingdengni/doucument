# 基础使用

## 官方网址

https://www.h2database.com/html/features.html#auto_reconnect

## pom  xml

```xml

<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <version>2.2.220</version>
</dependency>
<dependency>
  <groupId>com.zaxxer</groupId>
  <artifactId>HikariCP</artifactId>
  <version>4.0.3</version>
</dependency>
```



## java 连接使用

FEXISTS=TRUE //只有存在才打开

INIT=runscript from '~/create.sql'\\;runscript from '~/init.sql'  // 执行sql脚本

​	【MySQL、MariaDB、Oracle、PostgreSQL、MSSQLServer、HSQLDB、Derby、DB2】

MODE=MYSQL;DATABASE_TO_LOWER=TRUE  //模式mysql   //;CASE_INSENSITIVE_IDENTIFIERS=TRUE 大小写敏感

;AUTO_RECONNECT=TRUE 自定重连接

```properties
url = "jdbc:h2:mem:testdb;MODE=MYSQL";  #内存模式
url = "jdbc:h2:file:./h2/testdb;MODE=MYSQL";  #文件模式 ，一样效果：jdbc:h2:./h2/testdb;MODE=MYSQL
username = "sa";
password = "";
driverClass="org.h2.Driver"

classpath:h2/h2-schema.sql
classpath:h2/h2-data.sql
```

Code

```java
private static final String url = "jdbc:h2:mem:testdb";
    private static final String username = "sa";
    private static final String password = "";
    private static final HikariConfig config = new HikariConfig();
    private static final HikariDataSource dataSource;
    static {
        config.setJdbcUrl(url);
        config.setUsername(username);
        config.setPassword(password);
        // 设置连接池大小，默认为10
        config.setMaximumPoolSize(10);
        dataSource = new HikariDataSource(config);
    }
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
    public static void close() {
        dataSource.close();
    }

```

## 基本配置

Jpa 模式

```yaml
spring:
  datasource:
    platform: h2
    driverClassName: org.h2.Driver
    password: sa
    username: sa
    url: jdbc:h2:mem:dbtest
    schema: classpath:db/schema.sql
    data: classpath:db/data.sql
  h2: # 本地web页面 host/h2
    console:
      enabled: true
      path: /h2 #path 路径
      settings:
        web-allow-others: true
  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true
```

第一种：spring xml

```xml
 <!--当然是配置datasource了-->
 <jdbc:embedded-database id="dataSource" type="H2">
   <!--一定要是先DDL，即数据库定义语言-->
   <jdbc:script location="classpath:sql/h2-schema.sql"/>
   <!--然后才是DML，数据库操作语言-->
   <jdbc:script location="classpath:sql/h2-data.sql" encoding="UTF-8"/>
 </jdbc:embedded-database>
 
 <!--定义springjdbctemplate-->
 <bean class="org.springframework.jdbc.core.JdbcTemplate" id="jdbcTemplate">
	 <property name="dataSource" ref="dataSource"/>
 </bean>
```

第二种：spring beans

```java
@Configuration
public class BeanConfig {

    @Bean
    public DataSource dataSource() {
        try {
            EmbeddedDatabaseBuilder dbBuilder = new EmbeddedDatabaseBuilder();
            return dbBuilder.setType(EmbeddedDatabaseType.H2)
                    .addScripts("classpath:sql/h2-schema.sql", "classpath:sql/h2-data.sql")
                    .build();
        } catch (Exception e) {
            System.out.println("创建数据库连接失败");
            return null;
        }
    }

    @Bean
    public JdbcTemplate jdbcTemplate(){
        JdbcTemplate jdbcTemplate = new JdbcTemplate();
        jdbcTemplate.setDataSource(dataSource());
        return jdbcTemplate;
    }
}

```

