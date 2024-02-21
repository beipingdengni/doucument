## 官方文档

> https://mybatis.org/generator/configreference/xmlconfig.html

## 代码执行

```java
// MBG 执行过程中的警告信息
List<String> warnings = new ArrayList<String>();
// 当生成的代码重复时，覆盖原代码
boolean overwrite = true;
// 读取我们的 MBG 配置文件
File configFile = new File("src/main/resources/generator/generatorConfig.xml");
// 或使用 InputStream configFile = Generator.class.getResourceAsStream("/generator/generatorConfig.xml");
ConfigurationParser cp = new ConfigurationParser(warnings);
Configuration config = cp.parseConfiguration(configFile);
DefaultShellCallback callback = new DefaultShellCallback(overwrite);
//创建 MBG
MyBatisGenerator myBatisGenerator = new MyBatisGenerator(config, callback, warnings);
//执行生成代码
myBatisGenerator.generate(null);
//输出警告信息
for (String warning : warnings) {
  System.out.println(warning);
}
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

## 参考项目macrozheng/mall生产

> https://github.com/macrozheng/mall/blob/master/mall-mbg/src/main/resources/generatorConfig.xml
>
> generator.properties

```properties
jdbc.driverClass=com.mysql.cj.jdbc.Driver
jdbc.connectionURL=jdbc:mysql://localhost:3306/mall?useUnicode=true&characterEncoding=utf-8&serverTimezone=Asia/Shanghai
jdbc.userId=root
jdbc.password=root
```

> generatorConfig.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">

<generatorConfiguration>
    <properties resource="generator.properties"/>
    <context id="MySqlContext" targetRuntime="MyBatis3" defaultModelType="flat">
        <!-- 配置SQL语句中的前置分隔符 -->
        <property name="beginningDelimiter" value="`"/>
        <!-- 配置SQL语句中的后置分隔符 -->
        <property name="endingDelimiter" value="`"/>
        <!-- 配置生成Java文件的编码 -->
        <property name="javaFileEncoding" value="UTF-8"/>
        <!-- 为模型生成序列化方法 -->
        <plugin type="org.mybatis.generator.plugins.SerializablePlugin"/>
        <!-- 为生成的Java模型创建一个toString方法 -->
        <plugin type="org.mybatis.generator.plugins.ToStringPlugin"/>
        <!-- 生成mapper.xml时覆盖原文件 -->
        <plugin type="org.mybatis.generator.plugins.UnmergeableXmlMappersPlugin" />
        <commentGenerator type="com.macro.mall.CommentGenerator">
            <!-- 是否阻止生成的注释 -->
            <property name="suppressAllComments" value="true"/>
            <!-- 是否阻止生成的注释包含时间戳 -->
            <property name="suppressDate" value="true"/>
            <!-- 是否添加数据库表的备注信息 -->
            <property name="addRemarkComments" value="true"/>
        </commentGenerator>
        <!-- 配置MBG要连接的数据库信息 -->
        <jdbcConnection driverClass="${jdbc.driverClass}"
                        connectionURL="${jdbc.connectionURL}"
                        userId="${jdbc.userId}"
                        password="${jdbc.password}">
            <!-- 解决mysql驱动升级到8.0后不生成指定数据库代码的问题 -->
            <property name="nullCatalogMeansCurrent" value="true" />
        </jdbcConnection>
        <!-- 用于控制实体类的生成 -->
        <javaModelGenerator targetPackage="com.macro.mall.model" targetProject="mall-mbg\src\main\java"/>
        <!-- 用于控制Mapper.xml文件的生成 -->
        <sqlMapGenerator targetPackage="com.macro.mall.mapper" targetProject="mall-mbg\src\main\resources"/>
        <!-- 用于控制Mapper接口的生成 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.macro.mall.mapper"
                             targetProject="mall-mbg\src\main\java"/>
        <!-- 配置需要生成的表，生成全部表tableName设为% -->
        <table tableName="%">
            <!-- 用来指定主键生成策略 -->
            <generatedKey column="id" sqlStatement="MySql" identity="true"/>
        </table>
    </context>
</generatorConfiguration>
```

### com.macro.mall.CommentGenerator

> 只处理swagger返回的值注解
>
> 如下：@ApiModelProperty(value = "举报状态：0->未处理；1->已处理")

```java

/**
 * 自定义注释生成器
 */
public class CommentGenerator extends DefaultCommentGenerator {
    private boolean addRemarkComments = false;
    private static final String EXAMPLE_SUFFIX="Example";
    private static final String MAPPER_SUFFIX="Mapper";
    private static final String API_MODEL_PROPERTY_FULL_CLASS_NAME="io.swagger.annotations.ApiModelProperty";

    /**
     * 设置用户配置的参数
     */
    @Override
    public void addConfigurationProperties(Properties properties) {
        super.addConfigurationProperties(properties);
        this.addRemarkComments = StringUtility.isTrue(properties.getProperty("addRemarkComments"));
    }

    /**
     * 给字段添加注释
     */
    @Override
    public void addFieldComment(Field field, IntrospectedTable introspectedTable,
                                IntrospectedColumn introspectedColumn) {
        String remarks = introspectedColumn.getRemarks();
        //根据参数和备注信息判断是否添加swagger注解信息
        if(addRemarkComments&&StringUtility.stringHasValue(remarks)){
//            addFieldJavaDoc(field, remarks);
            //数据库中特殊字符需要转义
            if(remarks.contains("\"")){
                remarks = remarks.replace("\"","'");
            }
            //给model的字段添加swagger注解
            field.addJavaDocLine("@ApiModelProperty(value = \""+remarks+"\")");
        }
    }

    /**
     * 给model的字段添加注释
     */
    private void addFieldJavaDoc(Field field, String remarks) {
        //文档注释开始
        field.addJavaDocLine("/**");
        //获取数据库字段的备注信息
        String[] remarkLines = remarks.split(System.getProperty("line.separator"));
        for(String remarkLine:remarkLines){
            field.addJavaDocLine(" * "+remarkLine);
        }
        addJavadocTag(field, false);
        field.addJavaDocLine(" */");
    }

    @Override
    public void addJavaFileComment(CompilationUnit compilationUnit) {
        super.addJavaFileComment(compilationUnit);
        //只在model中添加swagger注解类的导入
        if(!compilationUnit.getType().getFullyQualifiedName().contains(MAPPER_SUFFIX)
           &&!compilationUnit.getType().getFullyQualifiedName().contains(EXAMPLE_SUFFIX)){
            compilationUnit.addImportedType(new FullyQualifiedJavaType(API_MODEL_PROPERTY_FULL_CLASS_NAME));
        }
    }
}
```

