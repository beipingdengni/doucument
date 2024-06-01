

### 指定索引更新字段

```
POST ${index_name}/${_doc}/${_id}/_update
{
  "doc" : {
        "field_name" : field_value
    }
}
```

#### 查询更新

```shell
curl -XPOST "localhost:9200/my_index/_update_by_query?conflicts=proceed" -H 'Content-Type: application/json' -d'
{
  "query": {
    "term": {
      "status": "old"
    }
  },
  "script": {
    "source": "ctx._source.status = \"new\""
  }
}
'
```

#### 更复杂的查询更新

```sh
# 查询条件中计算
curl -XPOST http://host:9200/index_name/_update_by_query -H 'Content-Type: application/json' -d '
{
  "query": {
    "bool": {
      "must": [
        {
          "script": {
            "script": {
              "source": "doc['a_time'].value - doc['b_time'].value == 100000"
            }
          }
        }
      ]
    }
  },
  "script": {
    "source": "ctx._source.name = params.name",
    "params": {
      "name": "wb"
    }
  }
}
'
# 查询字段不存在，赋值处理
curl -XPOST http://host:9200/index_name/_update_by_query -H 'Content-Type: application/json' -d '
{
  "query": {
    "bool": {
      "must_not": [
        {
          "exists": {
            "field": "sort_time"
          }
        }
      ]
    }
  },
  "script": {
  	"lang": "painless",
    "source": "ctx._source['sort_time'] = ctx._source['update_time']"
    // 根据条件删除字段
	  // "source":"ctx._source.remove(\"dept_name\")"
	  // 更新多个字段
	  // "source": "ctx._source.field1='1';ctx._source.2r='1';ctx._source.field3='1';"
    // 判断条件处理数据
    // "source": """
    //  if (ctx._source.your_field.contains('old_string')) {
    //    ctx._source.your_field = ctx._source.your_field.replace('old_string', 'new_string');
    //  }
    // """
  }
}
'


curl -XPOST http://host:9200/index_name/_doc/id/_update -H 'Content-Type: application/json' -d '
{
  "doc": {
    "字段": "value值"
}
'



```

