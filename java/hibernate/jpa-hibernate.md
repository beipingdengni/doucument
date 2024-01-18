

## maven

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
 
 <!-- MySQL连接 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```

## 基础配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/demo?serverTimezone=Asia/Shanghai&characterEncoding=utf-8
    username: root
    password: root
  jpa:
    show-sql: true # 默认false，在日志里显示执行的sql语句
    database: mysql
    database-platform: org.hibernate.dialect.MySQL5Dialect
    hibernate:
      ddl-auto: update #指定为update，每次启动项目检测表结构有变化的时候会新增字段，表不存在时会 新建，如果指定create，则每次启动项目都会清空数据并删除表，再新建
      naming:
        #指定jpa的自动表生成策略，驼峰自动映射为下划线格式7
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyLegacyJpaImpl
        #physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

## java代码



```java
public interface UserDao extends JpaRepository<User,Integer>, Serializable {
}

```

