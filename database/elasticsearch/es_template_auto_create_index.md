# 模版创建

## 查看模版

```
GET _template/{模版名称}
或
GET _template/{正则} #如：GET _template/shop*

# 查看是否存在
HEAD _template/shop_tem  # status code 返回200成功，404为失败
```

## 创建模版

###  ES 6.0之前的版本:

```json
PUT _template/shop_template
{
    "template": "shop*",       // 可以通过"shop*"来适配
    "order": 0,                // 模板的权重, 多个模板的时候优先匹配用, 值越大, 权重越高
    "settings": {
        "number_of_shards": 5,  // 分片数量, 可以定义其他配置项
        "number_of_replicas": 1 // 数据备份
    },
    "aliases": {
        "alias_1": {}          // 索引对应的别名
    },
    "mappings": {
        "_default": {          // 默认的配置, ES 6.0开始不再支持
            "_source": { "enabled": false },  // 是否保存字段的原始值
            "_all": { "enabled": false },     // 禁用_all字段
            "dynamic": "strict"               // 只用定义的字段, 关闭默认的自动类型推断
        },
        "type1": {             // 默认的文档类型设置为type1, ES 6.0开始只支持一种type, 所以这里不需要指出
            "_source": {"enabled": false},
            "properties": {        // 字段的映射
                "@timestamp": {    // 具体的字段映射
                    "type": "date",           
                    "format": "yyyy-MM-dd HH:mm:ss"
                },
                "@version": {
                    "doc_values": true,
                    "index": "not_analyzed",  // 不索引
                    "type": "string"          // string类型
                }
            }
        }
    }
}
```

### ES 6.0之后的版本:

```
mappings主要是一些说明信息，大致又分为_all、_source、prpperties这三部分：
     (1) _all：主要指的是AllField字段，我们可以将一个或多个都包含进来，在进行检索时无需指定字段的情况下检索多个字段。设置“_all" : {"enabled" : true}
     (2) _source：主要指的是SourceField字段，Source可以理解为ES除了将数据保存在索引文件中，另外还有一份源数据。_source字段在我们进行检索时相当重要，如果在{"enabled" : false}情况下默认检索只会返回ID， 你需要通过Fields字段去到索引中去取数据，效率不是很高。但是enabled设置为true时，索引会比较大，这时可以通过Compress进行压缩和inclueds、excludes来在字段级别上进行一些限制，自定义哪些字段允许存储。
     (3) properties：这是最重要的步骤，主要针对索引结构和字段级别上的一些设置。
```



```json
PUT _template/shop_template
{
    "index_patterns": ["shop*", "bar*"],       // 可以通过"shop*"和"bar*"来适配, template字段已过期
    "order": 0,                // 模板的权重, 多个模板的时候优先匹配用, 值越大, 权重越高
    "settings": {
        "number_of_shards": 5,  // 分片数量, 可以定义其他配置项
        "number_of_replicas": 1 // 数据备份
    },
    "aliases": {
        "alias_1": {}          // 索引对应的别名
    },
    "mappings": {
        // ES 6.0开始只支持一种type, 名称为“_doc”
        "_doc": {
            "_source": {            // 是否保存字段的原始值
                "enabled": false
            },
            "properties": {        // 字段的映射
                "@timestamp": {    // 具体的字段映射
                    "type": "date",           
                    "format": "yyyy-MM-dd HH:mm:ss"
                },
                "@version": {
                    "doc_values": true,
                    "index": "false",   // 设置为false, 不索引
                    "type": "text"      // text类型
                },
                "logLevel": {
                    "type": "long"
                },
                "logLevel": {
                    "type": "long"
                },
                "duration": {
                  "index": true,
                  "store": false,
                  "type": "long"
                },
                "lockStatus": {
                  "index": true,
                  "store": false,
                  "type": "integer"
                },
                "customerId": {
                  "index": true,
                  "store": false,
                  "type": "keyword"
                },
                "commentDetail": {
                  "search_analyzer": "ik_max_word",
                  "analyzer": "ik_max_word",
                  "index": true,
                  "store": false,
                  "type": "text"
                },
                "comment": {
                  "index": true,
                  "store": false,
                  "type": "byte"
                },
                "endTime": {
                  "index": true,
                  "store": false,
                  "type": "date"
                }
            }
        }
    }
}
```

## 修改模版

```
查询出原始内容，再增加字段，使用PUT方法
```

## 删除模版

```
DELETE _template/{模版名称}
```



# 自动创建索引配置

> 也可以增加配置到，在启动文件：action.auto_create_index=true

```
// PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "twitter,index10,-index1*,+ind*" 
    }
}

// 允许twitter, index10 自动创建
// 不允许-index1 开头的自动创建
// 注意是由先后顺序的.

// PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "false" 
    }
}
// 完全禁止自动创建

// PUT _cluster/settings
{
    "persistent": {
        "action.auto_create_index": "true" 
    }
}
// 完全允许自动创建
```

