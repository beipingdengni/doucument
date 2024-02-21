## JPA数据处理

### 1、创建查询的顺序

```
Spring Data JPA 在为接口创建代理对象时，如果发现同时存在多种上述情况可用，它该优先采用哪种策略呢？为此，<jpa:repositories> 提供了 query-lookup-strategy 属性，用以指定查找的顺序。它有如下三个取值：

create --- 通过解析方法名字来创建查询。即使有符合的命名查询，或者方法通过 @Query 指定的查询语句，都将会被忽略。
create-if-not-found --- 如果方法通过 @Query 指定了查询语句，则使用该语句实现查询；如果没有，则查找是否定义了符合条件的命名查询，如果找到，则使用该命名查询；如果两者都没有找到，则通过解析方法名字来创建查询。这是 query-lookup-strategy 【属性的默认值】。
use-declared-query --- 如果方法通过 @Query 指定了查询语句，则使用该语句实现查询；如果没有，则查找是否定义了符合条件的命名查询，如果找到，则使用该命名查询；如果两者都没有找到，则抛出异常。

```

### 2、事务

```
默认情况下，Spring Data JPA 实现的方法都是使用事务的。针对查询类型的方法，其等价于 @Transactional(readOnly=true)；增删改类型的方法，等价于 @Transactional。可以看出，除了将查询的方法设为只读事务外，其他事务属性均采用默认值。

```

### 3、接口中的部分方法提供自定义实现

```
为了享受 Spring Data JPA 带给我们的便利，同时又能够为部分方法提供自定义实现，我们可以采用如下的方法

将需要开发者手动实现的方法从持久层接口（假设为 AccountDao ）中抽取出来，独立成一个新的接口（假设为 AccountDaoPlus ），并让 AccountDao 继承 AccountDaoPlus；
为 AccountDaoPlus 提供自定义实现（假设为 AccountDaoPlusImpl ）；
将 AccountDaoPlusImpl 配置为 Spring Bean；
在 <jpa:repositories> 中按清单 19 的方式进行配置。

1、指定自定义实现类
  <jpa:repositories base-package="footmark.springdata.jpa.dao"> 
  <jpa:repository id="accountDao" repository-impl-ref=" accountDaoPlus " /> 
  </jpa:repositories>
  <bean id="accountDaoPlus" class="......."/>

2、设置自动查找时默认的自定义实现类命名规则
<jpa:repositories > 提供了一个 repository-impl-postfix 属性，用以指定实现类的后缀。假设做了如下配置：
<jpa:repositories base-package="footmark.springdata.jpa.dao"
repository-impl-postfix="Impl"/>
框架扫描到 AccountDao 接口时，它将尝试在相同的包目录下查找 AccountDaoImpl.java，如果找到，便将其中的实现方法作为最终生成的代理类中相应方法的实现
```

## spring boot 处理 

参考博客：

https://www.cnblogs.com/coderjinjian/p/9686729.html 【多数据源】

https://blog.csdn.net/chukun123/article/details/82861361 【多数据源、读写分离】

```
@Bean("secondaryDataSource")
@ConfigurationProperties("spring.datasource.secondary")
public DataSource secondaryDataSource() {
	return DataSourceBuilder.create().type(HikariDataSource.class).build();
}
```

生成entity.     JPA参考配置：https://blog.csdn.net/lwladzhj/article/details/81671598

```xml
persistence.xml

<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <persistence-unit name="SimplePU" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.ejb.HibernatePersistence</provider>
        <class>footmark.springdata.jpa.domain.UserInfo</class>
        <class>footmark.springdata.jpa.domain.AccountInfo</class>
        <properties>
            <property name="hibernate.connection.driver_class" value="com.mysql.jdbc.Driver"/>
            <property name="hibernate.connection.url" value="jdbc:mysql://10.40.74.197:3306/zhangjp"/>
            <property name="hibernate.connection.username" value="root"/>
            <property name="hibernate.connection.password" value="root"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="false"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>
</persistence>
```



### spring xml 配置

```xml
<beans>
    <context:component-scan base-package="footmark.springdata.jpa"/>
    <tx:annotation-driven transaction-manager="transactionManager"/>
    <bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>
    <bean id="entityManagerFactory" class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
    </bean>
</beans>

<-- 需要在 <beans> 标签中增加对 jpa 命名空间的引用 --> 
<jpa:repositories base-package="footmark.springdata.jpa.dao" 
				entity-manager-factory-ref="entityManagerFactory" 
				transaction-manager-ref="transactionManager"/>
```

### Dao 业务层处理 

```java
public interface UserDao extends Repository<AccountInfo, Long> { 
  
	@Query("select a from AccountInfo a where a.accountId = ?1") 
	public AccountInfo findByAccountId(Long accountId); 
 
  @Query("from AccountInfo a where a.balance > :balance") 
  public Page<AccountInfo> findByBalanceGreaterThan( 
    @Param("balance")Integer balance,
    Pageable pageable); 
  
  @Modifying 
	@Query("update AccountInfo a set a.salary = ?1 where a.salary < ?2") 
	public int increaseSalary(int after, int before);
  
}
```

### 关键字处理，解析生产SQL

```
Spring Data JPA 为此提供了一些表达条件查询的关键字，大致如下：

And --- 等价于 SQL 中的 and 关键字，比如 findByUsernameAndPassword(String user, Striang pwd)；
Or --- 等价于 SQL 中的 or 关键字，比如 findByUsernameOrAddress(String user, String addr)；
Between --- 等价于 SQL 中的 between 关键字，比如 findBySalaryBetween(int max, int min)；
LessThan --- 等价于 SQL 中的 "<"，比如 findBySalaryLessThan(int max)；
GreaterThan --- 等价于 SQL 中的">"，比如 findBySalaryGreaterThan(int min)；
IsNull --- 等价于 SQL 中的 "is null"，比如 findByUsernameIsNull()；
IsNotNull --- 等价于 SQL 中的 "is not null"，比如 findByUsernameIsNotNull()；
NotNull --- 与 IsNotNull 等价；
Like --- 等价于 SQL 中的 "like"，比如 findByUsernameLike(String user)；
NotLike --- 等价于 SQL 中的 "not like"，比如 findByUsernameNotLike(String user)；
OrderBy --- 等价于 SQL 中的 "order by"，比如 findByUsernameOrderBySalaryAsc(String user)；
Not --- 等价于 SQL 中的 "！ ="，比如 findByUsernameNot(String user)；
In --- 等价于 SQL 中的 "in"，比如 findByUsernameIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；
NotIn --- 等价于 SQL 中的 "not in"，比如 findByUsernameNotIn(Collection<String> userList) ，方法的参数可以是 Collection 类型，也可以是数组或者不定长参数；

```

### 自定义启动类【HIbernate】

```java
@Id
@GeneratedValue(generator="increment")
@GenericGenerator(name="increment", strategy = "increment")
public Long getId() {
    return id;
}

@Temporal(TemporalType.TIMESTAMP)
@Column(name = "EVENT_DATE")
public Date getDate() {
    return date;
}

// A SessionFactory is set up once for an application!
final StandardServiceRegistry registry = new StandardServiceRegistryBuilder()
	.configure() // configures settings from hibernate.cfg.xml
	.build();
try {
	sessionFactory = new MetadataSources( registry ).buildMetadata().buildSessionFactory();
}
catch (Exception e) {
	StandardServiceRegistryBuilder.destroy(registry );
}
// 执行Hql 语言
session = sessionFactory.openSession();
session.beginTransaction();
List result = session.createQuery( "from Event" ).list();
for ( Event event : (List<Event>) result ) {
    System.out.println( "Event (" + event.getDate() + ") : " + event.getTitle() );
}
session.getTransaction().commit();
session.close();
```

