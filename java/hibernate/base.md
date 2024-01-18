

## 主方法

```java
//Hibernate 加载核心配置文件（有数据库连接信息）
Configuration configuration = new Configuration().configure();
//创建一个 SessionFactory 用来获取 Session 连接对象
SessionFactory sessionFactory = configuration.buildSessionFactory();
//获取session 连接对象
Session session = sessionFactory.openSession();
//开始事务
Transaction transaction = session.beginTransaction();
//根据主键查询 user 表中的记录
User user = session.get(User.class, 1);
System.out.println(user);
//提交事务
transaction.commit();
//释放资源
session.close();
sessionFactory.close();
```

## hibernate.cfg.xml 

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!--使用 Hibernate 自带的连接池配置-->
        <property name="connection.url">jdbc:mysql://localhost:3306/demo</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">root</property>
        <property name="connection.driver_class">com.mysql.jdbc.Driver</property>

        <!--hibernate 方言-->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>

        <!--打印sql语句-->
        <property name="hibernate.show_sql">true</property>
        <!--格式化sql-->
        <property name="hibernate.format_sql">true</property>

        <!-- 加载映射文件 -->
        <mapping resource="com/tbp/entity/mapping/User.hbm.xml"/>
        <!-- 或 -->
      	<mapping resource="resources/User.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```

## User.hbm.xml

> [实体类名].hbm.xml

```xml
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>
    <!-- name:类的全路径:-->
    <!-- table:表的名称:(可以省略的.使用类的名称作为表名.)-->
    <class name="net.biancheng.www.po.User" table="user" schema="bianchengbang_jdbc">
        <!-- 主键-->
        <id name="id" column="id">
            <!--主键生成策略-->
            <generator class="native"></generator>
        </id>
        <!--type:三种写法-->
        <!--Java类型 :java.lang.String-->
        <!--Hibernate类型:string-->
        <!--SQL类型 :不能直接使用type属性,需要子标签<column>-->
        <!--<column name="name" sql-type="varchar(20)"/>-->
        <property name="userId" column="user_id" type="java.lang.String"/>
        <property name="userName" column="user_name"/>
        <property name="password" column="password"/>
        <property name="email" column="email"/>
    </class>
</hibernate-mapping>
```

