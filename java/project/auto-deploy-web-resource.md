

# 部署平台表创建

```sql
CREATE TABLE if not exists `sys_app_resource` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `name` varchar(100) NOT NULL COMMENT '应用名称',
  `code` varchar(100) NOT NULL COMMENT '应用code',
  `type` int NOT NULL default 0 COMMENT '类型：0 后台应用， 1 前台应用',
  `status` int NOT NULL default 0 COMMENT '状态：0 停用，1 启用',
  `create_pin` varchar(100) NOT NULL COMMENT '创建人',
  `create_time` datetime NOT NULL default CURRENT_TIMESTAMP COMMENT '创建时间',
  `modify_pin` varchar(100) DEFAULT NULL COMMENT '修改人',
  `modify_time` datetime not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `yn` int DEFAULT '1' COMMENT '记录状态（1:有效，-1:无效）',
   PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='应用表';


CREATE TABLE if not exists `sys_app_resource_rel_path` (
  `id` bigint NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `app_id` bigint NOT NULL COMMENT '应用ID',
  `app_code` varchar(100) NOT NULL COMMENT '应用code',
  `app_path` varchar(2048) DEFAULT NULL COMMENT '资源内容 json',
  `status` int NOT NULL default 0 COMMENT '状态：0 待使用，1 启用，2 归档',
  `create_pin` varchar(100) NOT NULL COMMENT '创建人',
  `create_time` datetime NOT NULL default CURRENT_TIMESTAMP COMMENT '创建时间',
  `modify_pin` varchar(100) DEFAULT NULL COMMENT '修改人',
  `modify_time` datetime not null DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  `yn` int DEFAULT '1' COMMENT '记录状态（1:有效，-1:无效）',
   PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COMMENT='应用关联资源表';


// 
ALTER TABLE sys_app_resource ADD COLUMN `profile` varchar(100) DEFAULT null COMMENT '扩展属性(JSON)';
ALTER TABLE sys_app_resource ADD COLUMN `remark` varchar(100) DEFAULT null COMMENT '备注';
// 
ALTER TABLE sys_app_resource_rel_path ADD COLUMN `profile` varchar(100) DEFAULT null COMMENT '扩展属性(JSON)';
ALTER TABLE sys_app_resource_rel_path ADD COLUMN `remark` varchar(100) DEFAULT null COMMENT '备注';
```





```java
@Log4j2
@Service("systemService")
public class SystemServiceImpl implements SystemService {

    @Resource
    private StorageService storageService;

    @Resource(name = "sqlTemplate")
    private SqlSessionTemplate sqlTemplate;

    @PostConstruct
    public void init() throws SQLException {
        // SELECT COUNT(*) FROM information_schema.tables WHERE table_name = 'your_table_name';
        Statement statement = sqlTemplate.getConnection().createStatement();
        boolean execute = statement.execute("");
    }

    public Boolean addAppResources(Map<String,Object> map){
        return Boolean.TRUE;
    }

  
}

```







```xml
<dependency>
  <groupId>org.springframework.data</groupId>
  <artifactId>spring-data-elasticsearch</artifactId>
  <version>4.1.15</version>
  <exclusions>
    <exclusion>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>transport</artifactId>
    </exclusion>
    <exclusion>
      <groupId>org.elasticsearch.plugin</groupId>
      <artifactId>transport-netty4-client</artifactId>
    </exclusion>
    <exclusion>
      <groupId>org.elasticsearch.client</groupId>
      <artifactId>elasticsearch-rest-high-level-client</artifactId>
    </exclusion>
  </exclusions>
</dependency>
```

