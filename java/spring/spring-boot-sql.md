## spring-boot 中数据库使用和配置简单介绍


#### 多数据源

> application.properties

``` js
    spring.datasource.primary.url=jdbc:mysql://localhost:3306/test1
    spring.datasource.primary.username=root
    spring.datasource.primary.password=root
    spring.datasource.primary.driver-class-name=com.mysql.jdbc.Driver
    spring.datasource.secondary.url=jdbc:mysql://localhost:3306/test2
    spring.datasource.secondary.username=root
    spring.datasource.secondary.password=root
    spring.datasource.secondary.driver-class-name=com.mysql.jdbc.Driver

```

> Java 代码中的配置
>　可以参考　http://blog.didispace.com/springbootmultidatasource/

``` java

    @Configuration
public class DataSourceConfig {
    @Bean(name = "primaryDataSource")
    @Qualifier("primaryDataSource")
    @ConfigurationProperties(prefix="spring.datasource.primary")
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create().build();
    }
    @Bean(name = "secondaryDataSource")
    @Qualifier("secondaryDataSource")
    @Primary
    @ConfigurationProperties(prefix="spring.datasource.secondary")
    public DataSource secondaryDataSource() {
        return DataSourceBuilder.create().build();
    }
}

```

> 对JdbcTemplate的支持比较简单，只需要为其注入对应的datasource即可
``` java

    @Bean(name = "primaryJdbcTemplate")
public JdbcTemplate primaryJdbcTemplate(
        @Qualifier("primaryDataSource") DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}
@Bean(name = "secondaryJdbcTemplate")
public JdbcTemplate secondaryJdbcTemplate(
        @Qualifier("secondaryDataSource") DataSource dataSource) {
    return new JdbcTemplate(dataSource);
}

// 使用中 如下
    @Autowired
	@Qualifier("primaryJdbcTemplate")
	protected JdbcTemplate jdbcTemplate1;

    @Autowired
	@Qualifier("secondaryJdbcTemplate")
	protected JdbcTemplate jdbcTemplate2;


``` 