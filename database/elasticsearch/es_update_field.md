

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
    "source": "ctx._source['sort_time'] = ctx._source['update_time']"
  }
}
'
```

