```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>druid</artifactId>
    <version>1.2.21</version>
</dependency>
```



```java
public class DruidDataSourceExample {
    private static DruidDataSource dataSource;

    static {
        dataSource = new DruidDataSource();
        dataSource.setUrl("jdbc:mysql://localhost:3306/mydatabase");
        dataSource.setUsername("username");
        dataSource.setPassword("password");
        dataSource.setInitialSize(5);
        dataSource.setMaxActive(10);
    }

    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
}
```

