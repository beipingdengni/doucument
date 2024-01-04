

## 数据准备

```js
PUT /tehero_index
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    }
  },
  "mappings": {
    "_doc": {
      "dynamic": false,
      "properties": {
        "id": {
          "type": "integer"
        },
        "content": {
          "type": "keyword",
          "fields": {
            "ik_max_analyzer": {
              "type": "text",
              "analyzer": "ik_max_word",
              "search_analyzer": "ik_max_word"
            },
            "ik_smart_analyzer": {
              "type": "text",
              "analyzer": "ik_smart"
            }
          }
        },
        "name":{
          "type":"text"
        },
        "createAt": {
          "type": "date"
        }
      }
    }
  }
}
# 导入测试数据
POST _bulk
{ "index" : { "_index" : "tehero_index", "_type" : "_doc", "_id" : "1" } }
{ "id" : 1,"content":"关注我,系统学编程" }
{ "index" : { "_index" : "tehero_index", "_type" : "_doc", "_id" : "2" } }
{ "id" : 2,"content":"系统学编程,关注我" }
{ "index" : { "_index" : "tehero_index", "_type" : "_doc", "_id" : "3" } }
{ "id" : 3,"content":"系统编程,关注我" }
{ "index" : { "_index" : "tehero_index", "_type" : "_doc", "_id" : "4" } }
{ "id" : 4,"content":"关注我,间隔系统学编程" }

# 文档4 content 的分词
GET /_analyze
{
  "text": ["关注我,间隔系统学编程"],
  "analyzer": "ik_smart"
}
```

## multi_match、match、match_phrase、match_phrase_prefix

```js

// DSL 语句
GET /tehero_index/_doc/_search
{
  "query":{
    "match":{
      "content.ik_smart_analyzer":"系统编程"
    }
  }
}

// 使用match_phrase查询，ik_smart分词
GET /tehero_index/_doc/_search
{
    "query": {
        "match_phrase": {
            "content": {
            	"query": "关注我,系统学",
              "analyzer": "ik_smart_analyzer"
            }
        }
    }
}

// match_phrase_prefix
GET tehero_index/_doc/_search
{
  "query": {
    "match_phrase_prefix": {
      "content.ik_smart_analyzer": {
        "query": "系",
        "max_expansions": 1
      }
    }
  }
}
query：必需，字符串）您希望在提供的 中查找的文本<field>
	在执行搜索之前，查询会将任何提供的文本分析match_phrase_prefix为标记。该文本的最后一个术语被视为 前缀，匹配以该术语开头的任何单词。
analyzer：可选，字符串
  用于将值中的文本转换为标记的分析器query 。默认为映射到 的索引时间分析器<field>。如果没有映射分析器，则使用索引的默认分析器。
max_expansions：可选，整数
	最后提供的值项将扩展到的最大项数query。默认为50.
slop：可选，整数
	匹配标记之间允许的最大位置数。默认为0. 转置项的斜率为2.
zero_terms_query：可选，字符串
	analyzer指示如果删除所有标记（例如使用过滤器时）是否不返回任何文档stop。有效值为：
		none（默认）：如果删除所有标记，则不会返回任何文档analyzer。
		all：返回所有文档，类似于match_all 查询

```

