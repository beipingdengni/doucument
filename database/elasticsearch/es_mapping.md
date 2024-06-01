

## 控制字段是否自动生成

- 通过dynamic参数来控制字段的新增：
  - true（默认）允许自动新增字段，但是mapping不显示，查询返回JSON有（开启 —— 遇到陌生字段时, 进行动态映射）
  - false 不允许自动新增字段，但是文档可以正常写入，但无法对新增字段进行查询等操作（关闭 —— 忽略遇到的陌生字段）
  - strict 文档不能写入，（遇到陌生字段时, 作报错处理）

### 参考使用类型

```json
// 日期格式如下：
{
  "date": {
    "type":   "date",
    "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
  }
}
//  ES的复杂类型有3个，Array、object、nested
// 内嵌对象数组
{
    "my_index": {
        "mappings": {
            "_doc": {
              	"dynamic": true, //开启 —— 遇到陌生字段时, 进行动态映射
                "properties": {
                    "lists": {
                        //"type": "nested", //独立存储
                        "properties": {
                            "description": {
                                "type": "keyword"
                            },
                            "name": {
                                "type": "keyword"
                            }
                        }
                    },
                    "message": {
                        "type": "keyword"
                    },
                    "tags": {    // 数组
                        "type": "keyword"
                    }
                }
            }
        }
    }
}
```

## 添加字段

#### 方法一

```json
PUT my-index-000001
{
  "mappings": {
  	// "_source": { "mode": "synthetic" },
    "properties": {
      "date": {
        "type":   "date",
        "format": "yyyy-MM-ddTHH:mm:ssZ||yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis"
      },
      "date_nanos": {
        "type":   "date_nanos"   #2015-01-01T12:10:30.000Z
      }
    }
  }
}
```

#### 方法二

```sh
PUT /my-index/_mapping
{
  "properties": {
    "email": {
      "type": "keyword"
    }
  }
}
```

## 更新setting

```sh
PUT /my-index-000001/_settings
{
  "index" : {
    "number_of_replicas" : 2
  }
}
```



## 创建从文件中

```sh
curl -X PUT -H "Content-Type:application/json" -d @index_mapping_source.json "http://localhost:9200/demo_index"

#mapping文件：index_mapping_source.json
{
    "aliases": [],
    "settings": {
      "index": {
        "number_of_shards": "5",
        "number_of_replicas": "1"
      }
    },
    "mappings": {
      "_doc": {
        "properties": [{}]
      }
    }
}
```

## 参考创建

```json
{
  "properties": {
    "birthday": {
      "format": "yyyy-MM-dd HH:mm:ss||yyyy-MM-dd||epoch_millis",
      "type": "date"
    },
    "address": {
      "type": "text"
    },
    "hight": {
      "type": "text"
    },
    "name": {
      "type": "keyword"
    },
    "age": {
      "type": "long"
    }
  }
}
```

## 参考创建的mapping(ES版本6.5.4)

### 包含分词器 analysis-ik、analysis-pinyin、analysis-dynamic-synonym

> index_mapping_source.json

```json
{
  "aliases": [],
  "settings": {
    "index": {
      "number_of_shards": "5",
      "number_of_replicas": "1",
      "analysis": {
        "analyzer": {
          "jrkf_academy_analyzer": {
            "type": "custom",
            "tokenizer": "jrkf_academy_tokenizer"
          },
          "pinyin_analyzer": {
            "tokenizer": "jrkf_academy_pinyin_tokenizer"
          }
        },
        "tokenizer": {
          "jrkf_academy_pinyin_tokenizer": {
            "type": "pinyin",
            "keep_first_letter": "false"
          },
          "jrkf_academy_tokenizer": {
            "token_chars": [
              "letter",
              "digit",
              "punctuation",
              "symbol"
            ],
            "min_gram": "1",
            "type": "ngram",
            "max_gram": "1"
          }
        }
      }
    }
  },
  "mappings": {
    "ikbs_knowledge": {
      "properties": {
        "summary": {
          "type": "keyword"
        },
        "knowledgeVersion": {
          "type": "integer"
        },
        "hiddenFlag": {
          "type": "boolean"
        },
        "created": {
          "type": "keyword"
        },
        "submitDate": {
          "type": "long"
        },
        "dataStatus": {
          "type": "integer"
        },
        "source": {
          "type": "keyword"
        },
        "title": {
          "search_analyzer": "ik_smart",
          "analyzer": "ik_max_word",
          "type": "text",
          "fields": {
            "pinyin": {
              "analyzer": "pinyin_analyzer",
              "type": "text"
            },
            "raw": {
              "analyzer": "jrkf_academy_analyzer",
              "type": "text"
            }
          }
        },
        "click": {
          "type": "long"
        },
        "periodDateEnd": {
          "type": "long"
        },
        "content": {
          "search_analyzer": "ik_smart",
          "analyzer": "ik_max_word",
          "term_vector": "with_positions_offsets",
          "type": "text",
          "fields": {
            "raw": {
              "analyzer": "jrkf_academy_analyzer",
              "term_vector": "with_positions_offsets",
              "type": "text"
            }
          }
        },
        "attaches": {
          "type": "long"
        },
        "labels": {
          "type": "long"
        },
        "tags": {
          "search_analyzer": "ik_smart",
          "analyzer": "ik_max_word",
          "type": "text",
          "fields": {
            "pinyin": {
              "analyzer": "pinyin_analyzer",
              "type": "text"
            },
            "raw": {
              "analyzer": "jrkf_academy_analyzer",
              "type": "text"
            }
          }
        },
        "periodClick": {
          "type": "long"
        },
        "periodDateBegin": {
          "type": "long"
        },
        "companyId": {
          "type": "long"
        },
        "createdDate": {
          "type": "long"
        },
        "submitEditor": {
          "type": "text"
        },
        "categories": {
          "type": "long"
        },
        "id": {
          "type": "long"
        }
      }
    }
  }
}
```

## 动态mapping创建

### 语法格式

```json
{
	"mappings": {
		"dynamic_templates": [
    		{
      			"my_template_name": {  	#1 自定义动态模板名称
        		...匹配规则...        	#2 使用match_mapping_type、match、unmatch等等定义规则 
       		 	"mapping": { ... } 		#3 设置符合该规则的mapping配置
        		}
    		}
  		]
	}
}
```

`match_mapping_type` 写入json类型

### 默认格式

| **JSON**数据类型****                                         | **`"dynamic":"true"`**  **ES数据类型** | **`"dynamic":"runtime"  ` ES数据类型** |
| ------------------------------------------------------------ | -------------------------------------- | -------------------------------------- |
| `null`                                                       | No field added（不添加字段）           | No field added（不添加字段）           |
| `true` or `false`                                            | `boolean`                              | `boolean`                              |
| `double`                                                     | `float`                                | `double`                               |
| `long`                                                       | `long`                                 | `long`                                 |
| `object`                                                     | `object`                               | No field added                         |
| `array`                                                      | 取决于数组中的第一个非`null`值         | 取决于数组中的第一个非`null`值         |
| `string` that passes [date detection](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/dynamic-field-mapping.html#date-detection)<br/>通过[日期检测](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/dynamic-field-mapping.html#date-detection)的string | `date`                                 | `date`                                 |
| `string` that passes [numeric detection](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/dynamic-field-mapping.html#numeric-detection)<br/>通过[数字检测](https://www.elastic.co/guide/en/elasticsearch/reference/7.17/dynamic-field-mapping.html#numeric-detection)的string | `float` or `long`                      | `double` or `long`                     |
| 没有通过上面2个检测的string                                  | 带`.keyword`子字段的text类型           |                                        |

Keyword 类型存储长度: 10966(char)，未设置（ignore_above:10966）



#### 匹配名字规则

- `match`：字段名称匹配某规则
- `unmatch `：字段名称不匹配某规则
- `match_pattern`：设置为`regex`，配合`match `和 `unmatch `使用正则表达式
- 注意：对于嵌套对象，`match `和 `unmatch `只作用于最后一级字段名

### 操作案例

```json
// 创建的模版
PUT pigg_test
{
  "mappings": {
    "dynamic_templates": [
      {
        "my_template_long": { // 名称
          "match_mapping_type": "long",
          "mapping": {
            "type": "integer"
          }
        }
      },
      {
        "my_template_string": {
          "match_mapping_type": "string",
          // "match_pattern": "regex","match": "^profit_\d+$"
          // "match": "text_*_ik"
          // "match": "keyword_*"
          "mapping": {
            "type": "keyword"
          }
        }
      },
      {
        "string_to_text_ik": {
          "match_mapping_type": "string",
          "match":   "text_*_ik*",
          "mapping": {
            "type": "text",
            "analyzer": "ik_max_word"
          }
        }
      }
    ]
  }
}

// 添加数据
PUT pigg_test/_doc/1
{
  "name": "亚瑟王",
  "age": 33
}

// 自动生成
GET pigg_test/_mapping
{
    "properties": {
        "age": {
            "type": "integer"
        },
        "name": {
            "type": "keyword"
        }
    }
}
```

