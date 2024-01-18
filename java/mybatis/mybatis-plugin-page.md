https://github.com/pagehelper/Mybatis-PageHelper



```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>最新版本</version>
</dependency>

<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>最新版本</version>
</dependency>
```



配置spring mvc

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!-- 注意其他配置 -->
  <property name="plugins">
    <array>
      <bean class="com.github.pagehelper.PageInterceptor">
          <property name="properties">
              <!--使用下面的方式配置参数，一行配置一个 -->
              <value>
                  params=value1
                	<!-- oracle,mysql,mariadb,sqlite,hsqldb,postgresql,
                    db2,sqlserver,informix,h2,sqlserver2012,derby -->
                	helperDialect=mysql
                  helperDialect=mysql
                  reasonable=true
                  supportMethodsArguments=true
                  autoRuntimeDialect=true

                	<!-- 自定义实现方言
									dialectAlias=oracle=com.github.pagehelper.dialect.helper.OracleDialect -->
                	<!-- useSqlserver2012=true -->
									<!-- useSqlserver2012=true -->
<!-- params：为了支持startPage(Object params)方法，增加了该参数来配置参数映射，用于从对象中根据属性名取值， 可以配置 pageNum,pageSize,count,pageSizeZero,reasonable，不配置映射的用默认值， 默认值为pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero -->
              </value>
          </property>
      </bean>
    </array>
  </property>
</bean>

<!-- mybatis.xml 文件中配置插件 -->
<!--
    plugins在配置文件中的位置必须符合要求，否则会报错，顺序如下:
      properties?, settings?,
      typeAliases?, typeHandlers?,
      objectFactory?,objectWrapperFactory?,
      plugins?,
      environments?, databaseIdProvider?, mappers?
-->
<plugins>
    <!-- com.github.pagehelper为PageHelper类所在包名 -->
    <plugin interceptor="com.github.pagehelper.PageInterceptor">
        <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
        <property name="supportMethodsArguments" value="true"/>
	</plugin>
</plugins>
```

配置spring boot

```yaml
#如果不想在启动时输出 banner，可以通过系统变量或环境变量关闭。
	#系统变量：-Dpagehelper.banner=false
	#环境变量：PAGEHELPER_BANNER=false
pagehelper:
  propertyName: propertyValue
  reasonable: false
  defaultCount: true # 分页插件默认参数支持 default-count 形式，自定义扩展的参数，必须大小写一致
```



使用分页查询

```java
//第一种，RowBounds方式的调用
List<User> list = sqlSession.selectList("x.y.selectIf", null, new RowBounds(0, 10));

//第二种，Mapper接口方式的调用，推荐这种使用方式。
PageHelper.startPage(1, 10);
List<User> list = userMapper.selectIf(1);
//用PageInfo对结果进行包装
PageInfo page = new PageInfo(list);

//第三种，Mapper接口方式的调用，推荐这种使用方式。
PageHelper.offsetPage(1, 10);
List<User> list = userMapper.selectIf(1);

//第四种，参数方法调用
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(
            @Param("user") User user,
            @Param("pageNum") int pageNum,
            @Param("pageSize") int pageSize);
}
//配置supportMethodsArguments=true
//在代码中直接调用：
List<User> list = userMapper.selectByPageNumSize(user, 1, 10);

//第五种，参数对象
//如果 pageNum 和 pageSize 存在于 User 对象中，只要参数有值，也会被分页
//有如下 User 对象
public class User {
    //其他fields
    //下面两个参数名和 params 配置的名字一致
    private Integer pageNum;
    private Integer pageSize;
}
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(User user);
}
//当 user 中的 pageNum!= null && pageSize!= null 时，会自动分页
List<User> list = userMapper.selectByPageNumSize(user);

//第六种，ISelect 接口方式
//jdk8 lambda用法
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(()-> userMapper.selectGroupBy());

```

