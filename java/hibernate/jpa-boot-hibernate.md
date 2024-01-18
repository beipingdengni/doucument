

## maven

```xml
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-entitymanager</artifactId>
    <version>5.6.15.Final</version>
</dependency>
```

## config

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence xmlns="http://java.sun.com/xml/ns/persistence" version="2.0">
    <!-- name：持久化单元名称，transaction-type：持久化单元事务类型(JTA:分布式事务管理,RESOURCE_LOCAL:本地事务管理) -->
    <persistence-unit name="myJpa" transaction-type="RESOURCE_LOCAL">
        <!--jpa的实现方式,配置JPA服务提供商 -->
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <!-- 可配可不配，如果配置了顺序不能错，必须在provider之后-->
        <!--<class>com.caochenlei.hibernate.jpa.Customer</class>-->
        <!--可选配置：配置jpa实现方的配置信息-->
        <properties>
            <!-- 数据库信息配置：数据库驱动、数据库地址、数据库账户、数据库密码 -->
            <property name="hibernate.connection.driver_class" value="com.mysql.jdbc.Driver"/>
            <property name="hibernate.connection.url" value="jdbc:mysql://127.0.0.1:3306/hibernate_jpa"/>
            <property name="hibernate.connection.username" value="root"/>
            <property name="hibernate.connection.password" value="password"/>
            <!-- 配置JPA服务提供商可选参数 -->
            <property name="hibernate.show_sql" value="true" /><!-- 自动显示sql -->
            <property name="hibernate.format_sql" value="true"/><!-- 格式化sql -->
            <!-- 自动创建数据库表：
                none        ：不会创建表
                create      : 程序运行时创建数据库表（如果有表，先删除表再创建）
                update      ：程序运行时创建表（如果有表，不会创建表）
                create-drop : 每次加载hibernate时根据model类生成表，但是sessionFactory一关闭,表就自动删除。
                validate    : 每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。
            -->
            <property name="hibernate.hbm2ddl.auto" value="update" />
        </properties>
    </persistence-unit>
</persistence>
```

## Test

```java
   /**
     * 测试jpa的保存
     * 案例：保存一个客户到数据库中
     * Jpa的操作步骤
     * 1.加载配置文件创建工厂（实体管理器工厂）对象
     * 2.通过实体管理器工厂获取实体管理器
     * 3.获取事务对象，开启事务
     * 4.完成增删改查操作
     * 5.提交事务（回滚事务）
     * 6.释放资源
     */
    @Test
    public void testSave() {
        //1.加载配置文件创建工厂（实体管理器工厂）对象
        EntityManagerFactory factory = Persistence.createEntityManagerFactory("myJpa");
        //2.通过实体管理器工厂获取实体管理器
        EntityManager em = factory.createEntityManager();
        //3.获取事务对象，开启事务
        EntityTransaction tx = em.getTransaction();
        tx.begin();
        //4.完成增删改查操作：保存一个客户到数据库中
        Customer customer = new Customer();
        customer.setName("Sam");
        customer.setAddress("Beijing");
        //保存操作
        em.persist(customer);
        //5.提交事务
        tx.commit();
        //6.释放资源
        em.close();
        factory.close();
    }

```

### Entity

```java
/**
 * 客户的实体类
 *      配置映射关系
 *   1.实体类和表的映射关系
 *      @Entity：声明实体类
 *      @Table：配置实体类和表的映射关系
 *          name : 配置数据库表的名称
 *   2.实体类中属性和表中字段的映射关系
 */
@Data
@Entity
@Table(name = "tb_customer")
public class Customer {
    /**
     * @Id：声明主键的配置
     * @GeneratedValue:配置主键的生成策略
     *      strategy:
     *          GenerationType.IDENTITY ：自增，mysql
     *                  底层数据库必须支持自动增长（底层数据库支持的自动增长方式，对id自增）
     *          GenerationType.SEQUENCE : 序列，oracle
     *                  底层数据库必须支持序列
     *          GenerationType.TABLE : jpa提供的一种机制，通过一张数据库表的形式帮助我们完成主键自增
     *          GenerationType.AUTO ： 由程序自动的帮助我们选择主键生成策略
     * @Column：配置属性和字段的映射关系
     *      name：数据库表中字段的名称
     */
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "customer_id")
    private Long Id; //客户的主键
    @Column(name = "customer_name")
    private String name;//客户名称
    @Column(name="customer_age")
    private int age;//客户年龄
    @Column(name="customer_sex")
    private boolean sex;//客户性别
    @Column(name="customer_phone")
    private String phone;//客户的联系方式
    @Column(name="customer_address")
    private String address;//客户地址
}
```

