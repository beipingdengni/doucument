## maven

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
   		  <!-- <version>最新版本</version> -->
	  		<version>3.5.5</version>
    </dependency>
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>

<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus</artifactId>
    <!-- <version>最新版本</version> -->
	  <version>3.5.5</version>
</dependency>
```



## 使用方式

Spring Boot:

```yaml
spring:
  datasource:
    driver-class-name: org.h2.Driver
    username: root
    password: test
  sql:
    init:
      schema-locations: classpath:db/schema-h2.sql
      data-locations: classpath:db/data-h2.sql

mybatis-plus:
  configuration:
  	configLocation: mybtatis原生的文件路径
  	mapperLocations: "classpath*:/mapper/**/*.xml" # 默认扫描
  	typeAliasesPackage: com.tbp.dao.entity
  	typeHandlersPackage: typeHandlersPackage扫描
  	configuration:
  		mapUnderscoreToCamelCase: true # default is true
  		defaultEnumTypeHandler: com.baomidou.mybatisplus.extension.handlers.MybatisEnumTypeHandler : 枚举类需要实现 IEnum 接口或字段标记@EnumValue 注解.(3.1.2 以下版本为 EnumTypeHandler)
  global-config:
  	banner: false # default is true
  	superMapperClass: com.baomidou.mybatisplus.core.mapper.Mapper  # default config
    db-config:
    	idType: ASSIGN_ID
    	keyGenerator: com.baomidou.mybatisplus.core.incrementer.IKeyGenerator #下支持@bean注入
    	logicDeleteField: deleted
    	logicDeleteValue: 0
    	logicNotDeleteValue: 1

@MapperScan("com.tbp.dao.mapper")
```

Spring MVC：

```xml
<bean id="sqlSessionFactory" class="com.baomidou.mybatisplus.extension.spring.MybatisSqlSessionFactoryBean">
    <property name="dataSource" ref="dataSource"/>
    <property name="configuration" ref="configuration"/> <!--  非必须  -->
    <property name="globalConfig" ref="globalConfig"/> <!--  非必须  -->
  	<property name="plugins">
        <array>
            <ref bean="mybatisPlusInterceptor"/>
        </array>
    </property>
    ......
</bean>

<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
    <property name="basePackage" value="com.tbp.dao.mapper"/>
</bean>

<bean id="configuration" class="com.baomidou.mybatisplus.core.MybatisConfiguration">
    ......
</bean>

<bean id="globalConfig" class="com.baomidou.mybatisplus.core.config.GlobalConfig">
    <property name="dbConfig" ref="dbConfig"/> <!--  非必须  -->
		<property name="identifierGenerator" ref="customIdGenerator"/> <!--  自定义ID生成器  -->
    ......
</bean>
<bean name="customIdGenerator" class="com.baomidou.samples.incrementer.CustomIdGenerator"/>


<bean id="dbConfig" class="com.baomidou.mybatisplus.core.config.GlobalConfig.DbConfig">
    ......
</bean>

<!-- 分页查询 -->
<bean id="mybatisPlusInterceptor" class="com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor">
    <property name="interceptors">
        <list>
            <ref bean="paginationInnerInterceptor"/>
        </list>
    </property>
</bean>

<bean id="paginationInnerInterceptor" class="com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor">
    <!-- 对于单一数据库类型来说,都建议配置该值,避免每次分页都去抓取数据库类型 -->
    <constructor-arg name="dbType" value="H2"/>
</bean>
```

java 代码

```java
public interface UserMapper extends BaseMapper<User> {
  // 涉及到分页查询
  List<User> selectPageVo(IPage<User> page, Integer state);
}

@Configuration
@MapperScan("scan.your.mapper.package")
public class MybatisPlusConfig {

    /**
     * 添加分页插件
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
				
      	PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
	  	  paginationInnerInterceptor.setDbType(DbType.MYSQL);//如果有多数据源可以不配具体类型 否则都建议配上具体的DbTyp
  	  	paginationInnerInterceptor.setOverflow(true);
	      //如果配置多个插件,切记分页最后添加
        interceptor.addInnerInterceptor(paginationInnerInterceptor);
        return interceptor;
    }
}
```

插件配置 mybatis-config.xml

```xml
<plugins>
  <plugin interceptor="com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor">
    <property name="@page" value="com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor"/>
    <property name="page:dbType" value="h2"/>
  </plugin>
</plugins>
```



# 自动生成代码

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <!-- <version>最新版本</version> -->
	  <version>3.5.5</version>
</dependency>
```

### 执行生成

> 参考文章：https://blog.csdn.net/Serendipity_o/article/details/126584058

```java
public class MainGenCode {

    public static void main(String[] args) {
        String projectPath = System.getProperty("user.dir");//得到当前项目的路径
        FastAutoGenerator.create("url", "username", "password")
                .globalConfig(builder -> {
                    builder.author("tianbeiping1") // 设置作者
                            .enableSwagger() // 开启 swagger 模式
//                            .fileOverride() // 覆盖已生成文件
                            .outputDir(projectPath); // 指定输出目录
                })
                .dataSourceConfig(builder -> builder.typeConvertHandler((globalConfig, typeRegistry, metaInfo) -> {
                    int typeCode = metaInfo.getJdbcType().TYPE_CODE;
                    if (typeCode == Types.SMALLINT) {
                        // 自定义类型转换
                        return DbColumnType.INTEGER;
                    }
                    return typeRegistry.getColumnType(metaInfo);
                }))
//                .injectionConfig(consumer -> {
//                    Map<String, String> customFile = new HashMap<>();
//                    // 自定义模版
//                    customFile.put("DTO.java", "/templates/entityDTO.java.ftl");
//                    consumer.customFile(customFile);
//                })
                .packageConfig(builder -> {
                    builder.parent("com.tbp") // 设置父包名
                            //.moduleName("admin") // 设置父包模块名
                            .pathInfo(Collections.singletonMap(OutputFile.xml, projectPath + "/src/main/resources/mapper")); // 设置mapperXml生成路径
                })
                .strategyConfig(builder -> {
                    builder.addInclude("t_simple") // 设置需要生成的表名
                            .addTablePrefix("t_", "c_") // 设置过滤表前缀
                            .addFieldPrefix("ia", "")
                    ;
                })
                //Entity 策略配置
                .strategyConfig(builder -> {
                    builder // 设置需要生成的表名
                            .addTablePrefix("t_", "c_") // 设置过滤表前缀
                            .addFieldPrefix("ia", "")
                            .entityBuilder()
                            .enableFileOverride()
                            .enableLombok()
                            .enableTableFieldAnnotation()
                            .versionColumnName("version") //乐观锁字段名(数据库)
                            .versionPropertyName("version") //乐观锁属性名(实体)
                            .logicDeleteColumnName("deleted") //逻辑删除字段名(数据库)
                            .logicDeletePropertyName("deleted") //逻辑删除字段名(数据库)
                            .superClass("com.tbp.common.BaseEntity") //或superClass(BaseEntity.class) 设置父类BaseEntity.class
                            //.naming(NamingStrategy.underline_to_camel) // 数据库表映射到实体的命名策略，默认下划线转驼峰命名
                            //.addSuperEntityColumns("id", "create_by", "create_time", "update_by", "update_time") //添加父类公共字段
                            //.addIgnoreColumns("yn");
                            .addTableFills(new Column("create_time", FieldFill.INSERT)) //添加表字段填充
                            .addTableFills(new Property("updateTime", FieldFill.INSERT_UPDATE)) //添加表字段填充
                            .idType(IdType.AUTO); //全局主键类型
                            //.formatFileName("%sEntity"); //格式化文件名称
                })
                /*模板引擎配置，默认 Velocity 可选模板引擎 Beetl 或 Freemarker 或 Enjoy
                  .templateEngine(new BeetlTemplateEngine())
                  .templateEngine(new FreemarkerTemplateEngine())
                  .templateEngine(new EnjoyTemplateEngine()) */
                .templateEngine(new FreemarkerTemplateEngine())
                .execute();
    }
}
```

