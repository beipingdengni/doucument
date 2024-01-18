## Spring JPA

> Spring-data-jpa
>
> Spring-data-template
>
> Spring-data-mongodb
>
> Spring-data-redis
>
> 原生的**Hibernate**中叫做Session，在JPA中叫做**EntityManager**，在MyBatis中叫做**SqlSession**，通过这个对象来操作数据库

##  **1.核心方法**

- 查询所有数据 findAll()

- 修改 添加数据 S save(S entity)

- 分页查询 Page<S> findAll(Example<S> example, Pageable pageable)

- 根据id查询 findOne()

- 根据实体类属性查询： findByProperty (type Property); 例如：findByAge(int age)

- 删除 void delete(T entity)

- 计数 查询 long count() 或者 根据某个属性的值查询总数 countByAge(int age)

- 是否存在  boolean existsById(ID primaryKey

#### 使用案例

```java
User findFirstByOrderByLastnameAsc();
User findTopByOrderByAgeDesc();
Page<User> queryFirst10ByLastname(String lastname, Pageable pageable);
Slice<User> findTop3ByLastname(String lastname, Pageable pageable);
List<User> findFirst10ByLastname(String lastname, Sort sort);
List<User> findTop10ByLastname(String lastname, Pageable pageable);
```

#### 分页+排序实现

```java
DemoBean demoBean = new DemoBean();
demoBean.setAppId(appId); //查询条件
//创建查询参数
Example<DemoBean> example = Example.of(demoBean);
//获取排序对象
Sort sort = new Sort(Sort.Direction.DESC, "id");
//创建分页对象
PageRequest pageRequest = new PageRequest(index, num, sort);
//分页查询
datas = demoRepository.findAll(example, pageRequest).getContent();
```

#### Example_实例查询

```java
Person person = new Person();                          
person.setFirstname("Dave");  //Firstname = 'Dave'                          
 
ExampleMatcher matcher = ExampleMatcher.matching()                     
                        .withMatcher("name", GenericPropertyMatchers.startsWith()) //姓名采用“开始匹配”的方式查询
                      	.withIgnorePaths("int");  //忽略属性：是否关注。因为是基本类型，需要忽略掉
 
Example<Person> example = Example.of(person, matcher);  //Example根据域对象和配置创建一个新的ExampleMatcher  
```

#### 详细查询语法

| **关键词**          | 示例                                                         | 对应的sql片段                                                |
| ------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| `And`               | `findByLastnameAndFirstname`                                 | `… where x.lastname = ?1 and x.firstname = ?2`               |
| `Or`                | `findByLastnameOrFirstname`                                  | `… where x.lastname = ?1 or x.firstname = ?2`                |
| `Is,Equals`         | `findByFirstname`,`findByFirstnameIs`,`findByFirstnameEquals` | `… where x.firstname = ?1`                                   |
| `Between`           | `findByStartDateBetween`                                     | `… where x.startDate between ?1 and ?2`                      |
| `LessThan`          | `findByAgeLessThan`                                          | `… where x.age < ?1`                                         |
| `LessThanEqual`     | `findByAgeLessThanEqual`                                     | `… where x.age <= ?1`                                        |
| `GreaterThan`       | `findByAgeGreaterThan`                                       | `… where x.age > ?1`                                         |
| `GreaterThanEqual`  | `findByAgeGreaterThanEqual`                                  | `… where x.age >= ?1`                                        |
| `After`             | `findByStartDateAfter`                                       | `… where x.startDate > ?1`                                   |
| `Before`            | `findByStartDateBefore`                                      | `… where x.startDate < ?1`                                   |
| `IsNull`            | `findByAgeIsNull`                                            | `… where x.age is null`                                      |
| `IsNotNull,NotNull` | `findByAge(Is)NotNull`                                       | `… where x.age not null`                                     |
| `Like`              | `findByFirstnameLike`                                        | `… where x.firstname like ?1`                                |
| `NotLike`           | `findByFirstnameNotLike`                                     | `… where x.firstname not like ?1`                            |
| `StartingWith`      | `findByFirstnameStartingWith`                                | `… where x.firstname like ?1` (parameter bound with appended `%`) |
| `EndingWith`        | `findByFirstnameEndingWith`                                  | `… where x.firstname like ?1` (parameter bound with prepended `%`) |
| `Containing`        | `findByFirstnameContaining`                                  | `… where x.firstname like ?1` (parameter bound wrapped in `%`) |
| `OrderBy`           | `findByAgeOrderByLastnameDesc`                               | `… where x.age = ?1 order by x.lastname desc`                |
| `Not`               | `findByLastnameNot`                                          | `… where x.lastname <> ?1`                                   |
| `In`                | `findByAgeIn(Collection<Age> ages)`                          | `… where x.age in ?1`                                        |
| `NotIn`             | `findByAgeNotIn(Collection<Age> ages)`                       | `… where x.age not in ?1`                                    |
| `True`              | `findByActiveTrue()`                                         | `… where x.active = true`                                    |
| `False`             | `findByActiveFalse()`                                        | `… where x.active = false`                                   |
| `IgnoreCase`        | `findByFirstnameIgnoreCase`                                  | `… where UPPER(x.firstame) = UPPER(?1)`                      |

## 自定义查询和复杂查询

#### 查询注解

```java
@Modifying //做update操作时需要添加
@Query // 自定义Sql

@Query(value = "SELECT * FROM USERS WHERE X = ?1", nativeQuery = true)
User findByEmailAddress(String X);

@Query("select u from User u where u.firstname = :firstname") //不加nativeQuery应使用HQL
User findByLastnameOrFirstname(@Param("lastname") String lastname);
```

#### 接口类

```java
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> 
```

#### 复杂查询

```java
// 复杂查询
public interface PersonDao extends JpaSpecificationExecutor<Person>{
}

// 接口下默认查询如下
public interface JpaSpecificationExecutor<T> {
    T findOne(Specification<T> spec);
    List<T> findAll(Specification<T> spec);
    Page<T> findAll(Specification<T> spec, Pageable pageable);
    List<T> findAll(Specification<T> spec, Sort sort);
    long count(Specification<T> spec);
}

// 自定义查询
public interface Specification<T> {
    Predicate toPredicate(Root<T> root, CriteriaQuery<?> query, CriteriaBuilder cb);
}
// 实现方法如下
public class PersonSpec {
    public static Specification<Person> method1(){
        return new Specification<Person>(){
            @Override
            public Predicate toPredicate(Root<Person> root, CriteriaQuery<?> query, CriteriaBuilder cb) {
              // 2.多表查询
              Join<Task,Project> join = root.join("project", JoinType.INNER);
                  Path<String> exp4 = join.get("projectName");
                  return cb.like(exp4, "%projectName%");
/* Hibernate: 
	select count(person0_.id) as col_0_0_  from tb_person person0_ inner join tb_project project1_ on person0_.project_id=project1_.id  where project1_.project_name like ?*/ 
                return null;
            }
 
        };
    }
```



## Entity注解

### 案例

```java
@Data
@Entity //不写@Table默认为user
@Table(name="t_user") //自定义表名
public class User {
 
    @Id //主键
    @GeneratedValue(strategy = GenerationType.AUTO)//采用数据库自增方式生成主键
    //JPA提供的四种标准用法为TABLE,SEQUENCE,IDENTITY,AUTO.
    //TABLE：使用一个特定的数据库表格来保存主键。
    //SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。
    //IDENTITY：主键由数据库自动生成（主要是自动增长型）
    //AUTO：主键由程序控制。
 
    @Transient //此字段不与数据库关联
    @Version//此字段加上乐观锁
    //字段为name，不允许为空，用户名唯一
    @Column(name = "name", unique = true, nullable = false)
    private String name;
 
    @Temporal(TemporalType.DATE)//生成yyyy-MM-dd类型的日期
    //出参时间格式化
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    //入参时，请求报文只需要传入yyyymmddhhmmss字符串进来，则自动转换为Date类型数据
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm")
    private Date createTime;
  
  	//@ManyToOne(fetch = FetchType.LAZY)
    //@JoinColumn(name = "project_id")
    //private Project project;
}
```



## Spring-boot-jpa 配置

```properties
#项目端口的常用配置
server.port=8081

# 数据库连接的配置
spring.datasource.url=jdbc:mysql:///jpa?useSSL=false
spring.datasource.username=root
spring.datasource.password=zempty123
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

#数据库连接池的配置，hikari 连接池的配置
spring.datasource.hikari.idle-timeout=30000
spring.datasource.hikari.connection-timeout=10000
spring.datasource.hikari.maximum-pool-size=15
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.auto-commit=true

#通过 jpa 自动生成数据库中的表
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```

