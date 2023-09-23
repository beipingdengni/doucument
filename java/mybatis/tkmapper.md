## Tk.mapper

> 参考文档：https://gitee.com/free/Mapper/wikis/Home

pom jar 包

```
<dependency>
    <groupId>tk.mybatis</groupId>
    <artifactId>mapper</artifactId>
    <version>最新版本</version>
</dependency>
```

xml 配置

```
<bean class="tk.mybatis.spring.mapper.MapperScannerConfigurer">
		<!-- 扫描的包位置 -->
    <property name="basePackage" value="tk.mybatis.mapper.mapper"/>
    <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
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
```



##### 代码生成器

> 参看博客：https://blog.csdn.net/lyf_ldh/article/details/80956556

```
<!-- 代码生成器 -->
<dependency>
  <groupId>tk.mybatis</groupId>
  <artifactId>mapper-generator</artifactId>
  <version>${mapper-generator.version}</version>
</dependency>
```



使用自动生成代码：

```
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



```
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



配置xml 【 generatorConfig.xml 】

```
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
 
<!--suppress MybatisGenerateCustomPluginInspection -->
<generatorConfiguration>
    <context id="Mysql" targetRuntime="MyBatis3Simple" defaultModelType="flat">
    
        <property name="javaFileEncoding" value="UTF-8"/>
        <property name="useMapperCommentGenerator" value="false"/>
 
        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">
            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>
            <property name="caseSensitive" value="true"/>
            <property name="forceAnnotation" value="true"/>
            <property name="beginningDelimiter" value="`"/>
            <property name="endingDelimiter" value="`"/>
        </plugin>
 
<-- 连接数据库 -->
        <jdbcConnection driverClass="org.hsqldb.jdbcDriver"
                        connectionURL="jdbc:hsqldb:mem:generator"
                        userId="sa"
                        password="">
        </jdbcConnection>
 
        <!--MyBatis 生成器只需要生成 Model-->
        <javaModelGenerator targetPackage="test.model" 
                            targetProject="generator/src/test/java"/>
 
        <table tableName="user_info">
            <generatedKey column="id" sqlStatement="JDBC"/>
        </table>
        <table tableName="country">
            <generatedKey column="id" sqlStatement="JDBC"/>
        </table>
    </context>
```

