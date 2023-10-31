

## 添加字段

```sh
PUT my-index-000001
{
  "mappings": {
  	"_source": { "mode": "synthetic" },
  	
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

### 添加field字段

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

### 更新setting

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

