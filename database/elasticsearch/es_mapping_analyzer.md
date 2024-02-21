## ES分词器包含三部分：

### char_filter、tokenizer、fliter

#### char_filter：在分词之前对原字段字符进行过滤

1. html_strip，用于html标签的过滤，可通过escaped_tags属性排除需要过滤的标签；

2. mapping，字符映射filter

3. pattern_replace ，正则表达式替换

4. ```js
   {
     "char_filter": {
       "my_char_filter": {
         "type": "html_strip",
         "escaped_tags": ["b"]
       }
     }
   }
   
   {
     "char_filter": {
       "my_char_filter": {
         "type": "mapping",
         "mappings": [
           "٠ => 0",
           "١ => 1",
           "٢ => 2",
           "٣ => 3",
           "٤ => 4",
           "٥ => 5",
           "٦ => 6",
           "٧ => 7",
           "٨ => 8",
           "٩ => 9"
         ]
       }
     }
   }
   
   {
     "char_filter": {
       "my_char_filter": {
         "type": "pattern_replace",
         "pattern": "(\\d+)-(?=\\d)",
         "replacement": "$1_"
       }
   	}
   }
   ```


#### tokenizer，对输入文本进行处理，拆分成各个词元

1. standard 默认分析器，英文分词器，对中文不适用
2. letter ，字符分析器，分词结果只包含字符：
3. lowercase ，小写分析器，与letter分词结果一致，同时会将字符转成小写：
4. whitespace ，空白字符分析器，以空白字符来拆分输入文本：
5. uax_url_email，url、email分析器，与standard类似，但是会将url和email当成一个词元
5. ngram，将给定输入按照指定长度分成连续的词元，忽略语义，ngram分析器可用来代替模糊查询
5. edge_ngram ，将给定输入按照空格或者字符拆分后，从前向后分成词元，edge_ngram可用于处理搜索建议词问题；
5. keyword ，关键词分析器，即不做分词
5. pattern，正则分析器

#### fliter，后置处理器，tokenizer拆分词元之后，filter进行后续处理，可新增或者删除词元，如中文的拼音分词器、同义词 就是使用此方式

1. length 长度过滤filter，将长度过大或者过小的词元移除；

   1. ```jsx
      属性：
      min：最小长度 默认0
      max：最大长度 默认Integer.MAX_VALUE
      ```

2. lowercase 小写filter，将词元中的大写字符转成小写；

3. uppercase 大写filter，将词元中小写字符转成大写；

4. nGram，参考ngram tokenizer；

5. edgeNGram ，参考edge_ngram tokenizer；

6. stop 停用词 filter，移除词元中的停用词：

   1. ```jsx
      属性：
      stopwords：停用词集合，默认_english_
      stopwords_path：停用词文件路径
      ignore_case：忽略大小写
      remove_trailing：删除最后一个词元，false
      "filter": {
          "my_stop": {
              "type":       "stop",
              "stopwords": ["and", "is", "the"]
          }
      }
      ```

7. word_delimiter，单词分隔：

   1. ```jsx
      属性：
      generate_word_parts：单词拆分，"PowerShot" ⇒ "Power" "Shot"，默认true
      generate_number_parts：数字拆分，"500-42" ⇒ "500" "42"，默认true
      catenate_words：连词拆分，"wi-fi" ⇒ "wifi"，默认false
      catenate_numbers：连续数字拆分："500-42" ⇒ "50042"，默认false
      catenate_all："wi-fi-4000" ⇒ "wifi4000"
      split_on_case_change：大小写改变时拆分词元
      preserve_original：保留原始值，"500-42" ⇒ "500-42" "500" "42"，默认false
      split_on_numerics：数字拆分，"j2se"  ⇒  "j" "2" "se"，默认true
      ```

8. synonym ，同义词过滤器：

   1. ```jsx
      属性：
      synonyms_path：同义词文件路径
      synonyms：同义词配置
      "filter" : {
          "synonym" : {
              "type" : "synonym",
              "format" : "wordnet",
              "synonyms" : [
                  "s(100000001,1,'abstain',v,1,0).",
                  "s(100000001,2,'refrain',v,1,0).",
                  "s(100000001,3,'desist',v,1,0)."
              ]
          }
      }
      同义词文件配置规则：
      utf-8文件
      每行多个词以英文逗号分隔，表示双向同义词
      单向同义词以 =>分隔，如 i-pod, i pod => ipod
      ```

## 索引不存在添加

```http
PUT /my_index
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "tokenizer": "ik_max_word",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  }
}
```

## 索引已经存在添加

> 为现有索引添加自定义分析器： 如果索引已经存在，你需要先关闭索引，然后更新设置，并重新打开索引：

```http
hPOST /my_index/_close

PUT /my_index/_settings
{
  "analysis": {
    "analyzer": {
      "my_custom_analyzer": {
        "type": "custom",
        "tokenizer": "ik_max_word",
        "filter": [
          "lowercase"
        ]
      }
    }
  }
}

POST /my_index/_open
```



# 分词器验证(ik、ngram、pinyin)

```json
// ik分词器
POST /_analyze
{
  "text": ["关注我,间隔系统学编程"],
  "analyzer": "ik_smart" // ik_max_word
}

//  n-gram分词器
POST /_analyze
{
  "text": ["白云机场"],
  "tokenizer": "ngram"
}

POST /_analyze
{
  "text": ["据记者了,郑州文化馆、河南省大河文化艺术中心自2016年以来,通过组织第十一届、第十二届中国郑州国际少林武术节书画展,通过书画展文化艺术搭台,是认真贯彻习中央文艺工作座谈会重要讲话精神,响应文化部开展深入生活、扎根人民主题实践活动。以作品的真善美陶冶人类崇高之襟怀品格,树立中华民族的文化自信,用写意精神推动社会文学艺术的繁荣发展。"],
  "tokenizer": {
    "type": "ngram",
    "min_gram": 1,
    "max_gram": 3,
    "token_chars": [
      "letter",
      "digit"
    ]
  }
}
//min_gram	字符最小长度。默认是1
//max_gram	字符最大长度。默认是2
//token_chars // 过滤掉字符串中的特殊符号或作分割符号
	//letter—— a, b, ï, 京		 文本字符，如 a,b,c 京等；
	//digit—— 3, 7					  数字
	//whitespace——" ", "\n" 	空白字符
	//punctuation——!, " 		  标点符号
	//symbol——$, √					  符号

较复杂的配置参考文档：https://github.com/medcl/elasticsearch-analysis-pinyin
//  pinyin 拼音分词器
POST /_analyze
{
  "text": ["白云机场"],
  "tokenizer": "pinyin"
}

{
  "text": ["白云机场"],
  "tokenizer": {
    "type":"pinyin",
    "keep_none_chinese":false
  }
}


```



## 数据准备

```js
PUT /demo_index
{
  "settings": {
    "index": {
      "number_of_shards": 1,
      "number_of_replicas": 1
    },
    // 分词配置  
    "index.max_ngram_diff": 10,
    "analysis": {
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "my_tokenizer"
        }
      },
      "tokenizer": {
        "my_tokenizer": {
          "type": "ngram", //颗粒度很细分词
          "min_gram": 3,
          "max_gram": 10,
          "token_chars": [
            "letter",
            "digit"
          ]
        }
      }
    }
  },
  "mappings": {
    "_doc": {
      "dynamic": false,
      "properties": {
        "id": {
          "type": "integer"
        },
        "title": {
          "type": "text",
          "analyzer": "my_analyzer", // 使用自定义分词器
          "fields": {
            "keyword": {
              "type": "keyword"
            }
        },
        "content": {
          "type": "keyword",
          "fields": {
            "ik_max_analyzer": {
              "type": "text",
              "analyzer": "ik_max_word",   // 写入使用分词器 content.ik_max_analyzer
              "search_analyzer": "ik_max_word" // 搜索使用分词器
            },
            "ik_smart_analyzer": {
              "type": "text",
              "analyzer": "ik_smart"	// 写入分词器 content.ik_smart_analyzer
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
```

## multi_match、match、match_phrase、match_phrase_prefix

```js

// DSL 语句
GET /demo_index/_doc/_search
{
  "query":{
    "match":{
      "content.ik_smart_analyzer":"系统编程"
    }
  }
}

// 使用match_phrase查询，ik_smart分词
GET /demo_index/_doc/_search
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
GET demo_index/_doc/_search
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



## 动态模版设置分词器

```json
curl -XPUT http://localhost:9200/_template/rtf
-d'
{
  "template":   "*", 
  "settings": { "number_of_shards": 1 }, 
  "mappings": {
    "_default_": {
      "_all": { 
        "enabled": true
      },
      "dynamic_templates": [
        {
          "strings": { 
            "match_mapping_type": "string",
            "mapping": {
              "type": "text",
              "analyzer":"ik_max_word",
              "ignore_above": 256,
              "fields": {
                "keyword": {
                  "type":  "keyword"
                }
              }
            }
          }
        }
      ]
    }
  }
}
'
```

