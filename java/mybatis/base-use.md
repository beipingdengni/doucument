

## jar使用

```xml
  <!-- MyBatis -->
  <dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.5.4</version>
  </dependency>
```



```java
String resource = "mybatis-config.xml";
// 加载MyBatis的主配置文件
InputStream inputStream = Resources.getResourceAsStream(resource);
// 通过构建器（SqlSessionFactoryBuilder）构建一个SqlSessionFactory工厂对象
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream)
SqlSession  sqlSession= factory.openSession()
  
  

// java 代码注入
// 数据源
DataSource dataSource = BaseDataTest.createBlogDataSource();
// 事务
TransactionFactory transactionFactory = new JdbcTransactionFactory();
// 环境和事物、数据源
Environment environment = new Environment("development", transactionFactory, dataSource);
// 核心配置
Configuration configuration = new Configuration(environment);
configuration.setLazyLoadingEnabled(true);
configuration.setEnhancementEnabled(true);
// 注入类的别名
configuration.getTypeAliasRegistry().registerAlias(Blog.class);
configuration.getTypeAliasRegistry().registerAlias(Post.class);
configuration.getTypeAliasRegistry().registerAlias(Author.class);
// 注册mapper
configuration.addMapper(BoundBlogMapper.class);
configuration.addMapper(BoundAuthorMapper.class);

// 构建会话
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(configuration);


// 执行事务
try (SqlSession session = sqlSessionFactory.openSession()) {
    // 假设下面三行代码是你的业务逻辑
    session.insert(...);
    session.update(...);
    session.delete(...);
    session.commit();
}


// 从工厂中获取会话的
SqlSession openSession()
SqlSession openSession(boolean autoCommit)
SqlSession openSession(Connection connection)
SqlSession openSession(TransactionIsolationLevel level)
SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level)
SqlSession openSession(ExecutorType execType)
SqlSession openSession(ExecutorType execType, boolean autoCommit)
SqlSession openSession(ExecutorType execType, Connection connection)
Configuration getConfiguration();
```



## mybatis-config.xml

```

```

### 分页

```
RowBounds rowBounds = new RowBounds(offset, limit);
```

### TypeHandler处理

### 配置

```xml
<!-- 全局配置 -->
<typeHandlers>
         <typeHandler handler="BlobToStringTypeHandler" javaType="java.util.string" jdbcType="VARCHAR"/>
</typeHandlers>


<!-- 独立配置 -->
<resultMap type="com.tbp.entity.Article" id="ArticleMap">
  <result property="id" column="ID" jdbcType="INTEGER"/>
  <result property="name" column="NAME" jdbcType="VARCHAR"/>
  <result property="content" column="CONTENT" jdbcType="BLOB" typeHandler="com.tbp.common.typeHandler.BlobToStringTypeHandler"/>
</resultMap>

<!-- 独立配置 -->
<insert id="insertNote" parameterType="com.tbp.pojo.Note">
   insert into note (id, date) values(
  #{id}, 
  #{content, typeHandler=com.tbp.common.typeHandler.BlobToStringTypeHandler}
  )
  <!--使用我们自定义的TypeHandler-->
</insert>
```



### 自定义实现

```java
/**
 * sql 查询数据的时候将 BLOB 字段转换成 String 类型接收
 */
public class BlobToStringTypeHandler extends BaseTypeHandler<String> {

  // 设置非空参数
  @Override
  public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
    ps.setString(i, parameter);
  }

  // 根据列名，获取可以为空的结果
  @Override
  public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
    Blob blob = rs.getBlob(columnName);
    return new String(blob.getBytes(1, (int)blob.length()));
  }

  // 根据列索引，获取可以为空的结果
  @Override
  public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
    Blob blob = rs.getBlob(columnIndex);
    return new String(blob.getBytes(1, (int)blob.length()));
  }

  // 根据列索引，获取可以为空的结果（执行器）
  @Override
  public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
    Blob blob = cs.getBlob(columnIndex);
    return new String(blob.getBytes(1, (int)blob.length()));
  }

}
```

