# 自动生成代码

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <!-- <version>最新版本</version> -->
	  <version>3.5.5</version>
</dependency>

<!-- freemarker模板 -->
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.32</version>
</dependency>
<!--数据库驱动-->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <!-- <scope>runtime</scope> -->
    <version>5.1.49</version>
</dependency>
```

### 执行生成

> 参考文章：https://blog.csdn.net/Serendipity_o/article/details/126584058
>
> 

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
                .packageConfig(builder -> { // mapper位置生成
                    builder.parent("com.tbp") // 设置父包名
                            //.moduleName("admin") // 设置父包模块名
                            .pathInfo(Collections.singletonMap(OutputFile.xml, projectPath + "/src/main/resources/mapper")); // 设置mapperXml生成路径
                })
                .strategyConfig(builder -> { // 表的过滤处理
                    builder.addInclude("t_simple") // 设置需要生成的表名
                            .addTablePrefix("t_", "c_") // 设置过滤表前缀
                            .addFieldPrefix("ia", "")
                    ;
                })
                .strategyConfig(builder -> {  //Entity 策略配置
                    builder // 设置需要生成的表名
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

