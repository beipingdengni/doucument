## 官方文档

> https://mybatis.org/generator/configreference/xmlconfig.html

## 代码执行

```java
List<String> warnings = new ArrayList<String>();
boolean overwrite = true;
File configFile = new File("src/main/resources/generator/generatorConfig.xml");
ConfigurationParser cp = new ConfigurationParser(warnings);
Configuration config = cp.parseConfiguration(configFile);
DefaultShellCallback callback = new DefaultShellCallback(overwrite);
MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
myBatisGenerator.generate(null);
```



## Maven 插件配置

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
    </dependencies>
  </plugin>
</plugins>
```

## 生成文件配置

```xml
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<!--suppress MybatisGenerateCustomPluginInspection -->
<generatorConfiguration>
    <properties resource="db.properties"/>
    <!--    <classPathEntry location="./mysql-connector-java-5.1.29.jar"/>-->
    <context id="Mysql" targetRuntime="MyBatis3" defaultModelType="flat">
        <property name="javaFileEncoding" value="UTF-8"/>

<!--        <plugin type="tk.mybatis.mapper.generator.MapperPlugin">-->
<!--            <property name="mappers" value="tk.mybatis.mapper.common.Mapper"/>-->
<!--            <property name="caseSensitive" value="false"/>-->
<!--            <property name="forceAnnotation" value="true"/>-->
<!--        </plugin>-->

        <commentGenerator>
  		<!-- 静止生成备注，suppressAllComments为true时候，addRemarkComments才会生效  -->
            <property name="suppressAllComments" value="true"/>
            <property name="suppressDate" value="true"/>
  		<!-- <property name="addRemarkComments" value="true"/>-->
        </commentGenerator>

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
                         targetProject="src/main/resources">
          <property name="enableSubPackages" value="true" />
	      </sqlMapGenerator>
        <!--MyBatis 生成器只需要生成 mapper -->
        <javaClientGenerator targetPackage="com.tbp.mapper"
                             targetProject="src/main/java"
                             type="XMLMAPPER">
          <property name="enableSubPackages" value="true" />
	      </javaClientGenerator>
        <table tableName="%"
               enableCountByExample="false"
               enableUpdateByPrimaryKey="false"
               enableDeleteByExample="false"
               enableSelectByExample="false"
               enableUpdateByExample="false"
               enableInsert="true"
               enableDeleteByPrimaryKey="true"
               enableSelectByPrimaryKey="true"
        >
            <generatedKey column="id" sqlStatement="JDBC"/>
            <domainObjectRenamingRule searchString="^ia_" replaceString=""/>
      <!--  <columnRenamingRule searchString="^ia_" replaceString="" />-->
      <!--  <columnOverride column="LONG_VARCHAR_FIELD" javaType="java.lang.String" jdbcType="VARCHAR" />-->
      <!--  <ignoreColumn column="ignore_field" delimitedColumnName="false"/>-->
        </table>
    </context>
</generatorConfiguration>
```

## 自定义插件<plugin\>

```java
// 必须实现接口
org.mybatis.generator.api.Plugin
// 继承这个类更加容易
org.mybatis.generator.api.PluginAdapter
	// 参考学习：tk.mybatis.mapper.generator.MapperPlugin
```

