```xml
<dependency>
   <groupId>com.zaxxer</groupId>
   <artifactId>HikariCP</artifactId>
   <version>5.1.0</version>
</dependency>
```



```java
public class HikariCPExample {
    private static HikariDataSource dataSource;

    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydatabase");
        config.setUsername("username");
        config.setPassword("password");
        config.setMinimumIdle(5);
        config.setMaximumPoolSize(10);
        dataSource = new HikariDataSource(config);
    }

    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
}
```



```java
@Test
public void testHikariCP() throws SQLException {
    // 1、创建Hikari配置
    HikariConfig hikariConfig = new HikariConfig();
    // JDBC连接串
    hikariConfig.setJdbcUrl("jdbc:mysql://127.0.0.1:3306/iam?characterEncoding=utf8");
    // 数据库用户名
    hikariConfig.setUsername("root");
    // 数据库用户密码
    hikariConfig.setPassword("123456");
    // 连接池名称
    hikariConfig.setPoolName("testHikari");
    // 连接池中最小空闲连接数量
    hikariConfig.setMinimumIdle(4);
    // 连接池中最大空闲连接数量
    hikariConfig.setMaximumPoolSize(8);
    // 连接在池中的最大空闲时间
    hikariConfig.setIdleTimeout(600000L);
    // 数据库连接超时时间
    hikariConfig.setConnectionTimeout(10000L);

    // 2、创建数据源
    HikariDataSource dataSource = new HikariDataSource(hikariConfig);

    // 3、获取连接
    Connection connection = dataSource.getConnection();

    // 4、获取Statement
    Statement statement = connection.createStatement();

    // 5、执行Sql
    ResultSet resultSet = statement.executeQuery("SELECT COUNT(*) AS countNum tt_user");

    // 6、输出执行结果
    if (resultSet.next()) {
        System.out.println("countNum结果为：" + resultSet.getInt("countNum"));
    }

    // 7、释放链接
    resultSet.close();
    statement.close();
    connection.close();
    dataSource.close();
}
```

