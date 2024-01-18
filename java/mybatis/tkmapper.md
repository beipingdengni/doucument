## Tk.mapper

> 参考文档：https://gitee.com/free/Mapper/wikis/Home
>
> https://github.com/abel533/Mapper

pom jar 包

```xml
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>最新版本</version>
</dependency>

<dependency>
  <groupId>tk.mybatis</groupId>
  <artifactId>mapper-spring-boot-starter</artifactId>
  <version>版本号</version>
</dependency>
```

xml 配置

```xml
<!-- 集成Mybatis -->
<bean id="sessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSource"/>
  <property name="configLocation" value="classpath:mybatis-config.xml"/>
</bean>

<!-- org.mybatis.spring.mapper.MapperScannerConfigurer 这个是spring-mybatis携带的 -->
<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 扫描的包位置 com.tbp.dao  tk.mybatis.mapper.mapper-->
    <property name="basePackage" value="com.tbp.dao"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
    <!-- 3.2.2版本新特性，markerInterface可以起到mappers配置的作用，详细情况需要看Marker接口类 -->
    <!-- 
			<property name="markerInterface" value="com.isea533.mybatis.util.MyMapper"/>
		-->
    <property name="properties">
        <value>
        		notEmpty=true
        		mappers=tk.mybatis.mapper.common.Mapper
            参数名=值
            参数名2=值2
            ...
        </value>
    </property>
</bean>

<!-- 配置事务管理器 -->
<bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
  <property name="dataSource" ref="dataSource"/>
</bean>

<!-- spring 模版管理 -->
<bean id="sqlTemplate" class="org.mybatis.spring.SqlSessionTemplate">
  <constructor-arg index="0" ref="sessionFactory"/>
</bean>
```

注解配置

```java
@Configuration
@MapperScan(value = "tk.mybatis.mapper.annotation",
            properties = {
                    "mappers=tk.mybatis.mapper.common.Mapper",
                    "notEmpty=true"
            })
public static class MyBatisConfigRef {

    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
        sessionFactory.setDataSource(dataSource());
        //tk.mybatis.mapper.session.Configuration
        Configuration configuration = new Configuration();
        //可以对 MapperHelper 进行配置后 set
        configuration.setMapperHelper(new MapperHelper());
        //设置为 tk 提供的 Configuration
        sessionFactory.setConfiguration(configuration);
        return sessionFactory.getObject();
    }
}

```

yaml配置

```yaml
mapper:
  mappers:
    - tk.mybatis.mapper.common.Mapper
    - tk.mybatis.mapper.common.Mapper2
  notEmpty: true
#mapper.mappers=tk.mybatis.mapper.common.Mapper,tk.mybatis.mapper.common.Mapper2
#mapper.notEmpty=true
```



### 创建mapper

```java
import tk.mybatis.mapper.common.Mapper;

@Mapper // 必须配合 @MapperScan
public interface CountryMapper extends Mapper<Country> {
}

// 查找生成sql
Example example = new Example(Country.class);
example.setForUpdate(true);
example.createCriteria().andGreaterThan("id", 100).andLessThan("id",151);
example.or().andLessThan("id", 41);
List<Country> countries = mapper.selectByExample(example);

// Example.builder 方式
Example example = Example.builder(Country.class)
        .select("countryname")
        .where(Sqls.custom().andGreaterThan("id", 100))
        .orderByAsc("countrycode")
        .forUpdate()
        .build();
List<Country> countries = mapper.selectByExample(example);
```



```java

/**
normal,                     //原值
camelhump,                  //驼峰转下划线
uppercase,                  //转换为大写
lowercase,                  //转换为小写
camelhumpAndUppercase,      //驼峰转下划线大写形式
camelhumpAndLowercase,      //驼峰转下划线小写形式
*/

@Table
@NameStyle(Style.camelhumpAndUppercase) // userName属性，对应表字段：USER_NAME
public class Country {
  
  // 主键生成策略：@KeySql(推荐)   @GeneratedValue
  @Id
  //@KeySql(dialect = IdentityDialect.MYSQL) // @GeneratedValue(strategy = GenerationType.IDENTITY)
  @KeySql(useGeneratedKeys = true) // JDBC 支持通过 getGeneratedKeys 方法取回主键的情况
  // @GeneratedValue(generator = "JDBC")
	private Integer id;
  /**
  // SqlServer 中使用时，需要设置 id 的 insertable=false
  <insert id="insert" useGeneratedKeys="true" keyProperty="id">
    insert into country (id, countryname, countrycode) values (#{id},#{countryname},#{countrycode})
	</insert>
  */
  
  @ColumnType(
        column = "countryname",
        jdbcType = JdbcType.VARCHAR,
        typeHandler = StringTypeHandler.class)
	private String  countryname;
  
  @Transient
	private String otherThings; //非数据库表中字段
  
  @Version
  private Integer version;
}


// 处理自定义注解，获取实体类的全部信息
column = EntityHelper.getColumns(entityClass);
//判断是否有某个注解
boolean hasVersion = column.getEntityField().isAnnotationPresent(Version.class)
//通过下面的代码可以获取注解信息
Version version = column.getEntityField().getAnnotation(Version.class);
```



 Spring Boot Devtools 兼容性分析

> 增加文件，添加内容如下：spring-devtools.properties
>
> restart.include.mapper=/mapper-[\\w-\\.]+jar



##### 代码生成器

> 参看博客：https://blog.csdn.net/lyf_ldh/article/details/80956556

```xml
<!-- 代码生成器 -->
<dependency>
  <groupId>tk.mybatis</groupId>
  <artifactId>mapper-generator</artifactId>
  <version>${mapper-generator.version}</version>
</dependency>
```



使用自动生成代码：

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis.generator/mybatis-generator-core -->
<dependency>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-core</artifactId>
    <version>1.3.6</version>
</dependency>
<!-- 通用 Mapper -->
<!-- https://mvnrepository.com/artifact/tk.mybatis/mapper -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>4.0.0</version>
</dependency>
<!-- 如果你只需要用到通用 Mapper 中的插件，可以只引入 mapper-generator -->
<!-- 注意，这个包不需要和上面的 mapper 同时引入，mapper 中包含 generator -->
<!-- https://mvnrepository.com/artifact/tk.mybatis/mapper-generator -->
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper-generator</artifactId>
    <version>1.0.0</version>
</dependency>
```



```java
public static void main(String[] args) throws Exception {
        List<String> warnings = new ArrayList<String>();
        boolean overwrite = true;
        ConfigurationParser cp = new ConfigurationParser(warnings);
        Configuration config = 
                      cp.parseConfiguration(getResourceAsStream("generatorConfig.xml"));
        DefaultShellCallback callback = new DefaultShellCallback(overwrite);
        MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
        myBatisGenerator.generate(null);
        for (String warning : warnings) {
            System.out.println(warning);
        }
    }
```



### 配置xml 【 generatorConfig.xml 】

> 配置文件博客： https://blog.csdn.net/isea533/article/details/42102297 
>
> db.properties [放在resources目录下]
>
> ```
> driver=com.mysql.jdbc.Driver
> url=jdbc:mysql://localhost:3306/demo
> user=root
> pwd=123456
> ```

```xml
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<!--suppress MybatisGenerateCustomPluginInspection -->
<generatorConfiguration>
    <properties resource="db.properties"/>
    <!--    <classPathEntry location="./mysql-connector-java-5.1.29.jar"/>-->
    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
        <property name="javaFileEncoding" value="UTF-8"/>
        <property name="autoDelimitKeywords" value="true"/>
        <property name="useMapperCommentGenerator" value="true"/>

        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <property name="caseSensitive" value="false"/>
            <property name="forceAnnotation" value="true"/>
<!--            <property name="beginningDelimiter" value="`"/>-->
<!--            <property name="endingDelimiter" value="`"/>-->
<!--            <property name="lombok" value="Getter,Setter,ToString,Accessors"/>-->
        </plugin>
        <jdbcConnection driverClass="${driver}"
                        connectionURL="${url}"
                        userId="${user}"
                        password="${pwd}">
        </jdbcConnection>

        <javaTypeResolver >
            <property name="forceBigDecimals" value="true" />
        </javaTypeResolver>
        <!--MyBatis 生成器只需要生成 Model-->
        <javaModelGenerator targetPackage="com.tbp.model"
                            targetProject="src/main/java">
            <!-- 在set方法中增加加去除 trim -->
            <property name="trimStrings" value="true"/>
            <!-- MBG会根据catalog和schema来生成子包。如果false就会直接用targetPackage属性。默认为false -->
            <property name="enableSubPackages" value="true"/>
        </javaModelGenerator>
        <!--MyBatis 生成器只需要生成 xml -->
        <sqlMapGenerator targetPackage="mapper"
                         targetProject="src/main/resources"/>
        <!--MyBatis 生成器只需要生成 mapper -->
        <javaClientGenerator targetPackage="com.tbp.mapper"
                             targetProject="src/main/java"
                             type="XMLMAPPER"/>
        <!--  <table tableName="users" domainObjectName="UsersEntity">-->    
        <table tableName="ia_%">
            <generatedKey column="id" sqlStatement="JDBC"/>
      <!--  <domainObjectRenamingRule searchString="^ia_" replaceString=""/>-->    
      <!--  <columnRenamingRule searchString="^ia_" replaceString="" />-->
      <!--  <columnOverride column="LONG_VARCHAR_FIELD" javaType="java.lang.String" jdbcType="VARCHAR" />-->
      <!--  <ignoreColumn column="ignore_field" delimitedColumnName="false"/>-->
        </table>
    </context>
</generatorConfiguration>
```

### Maven 中配置组件

```xml
<plugins>
  <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.8.1</version>
    <configuration>
      <source>1.8</source>
      <target>1.8</target>
      <encoding>UTF-8</encoding>
    </configuration>
  </plugin>
  <plugin>
    <groupId>org.mybatis.generator</groupId>
    <artifactId>mybatis-generator-maven-plugin</artifactId>
    <version>1.3.7</version>
    <configuration>
      <configurationFile>
        ${basedir}/src/main/resources/generator/generatorConfig.xml
      </configurationFile>
      <overwrite>true</overwrite>
      <verbose>true</verbose>
    </configuration>
    <dependencies>
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.29</version>
      </dependency>
      <dependency>
        <groupId>tk.mybatis</groupId>
        <artifactId>mapper</artifactId>
        <version>4.0.0</version>
      </dependency>
    </dependencies>
  </plugin>
</plugins>
```

