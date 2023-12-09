### match

```js
get /testcopy/_search
{
  "query":{
    "bool": {
      "must": [
        {"match": {"content": "like"}},
        {"match": {"content": "elasticsearch"}}
      ]
    }
  }
}
```

### match_phrase

```js
 #这种就是 查 like elasticsearch  之间最多移动 3次 匹配的结果
get /testcopy/_search
{
  "query":{
    "match_phrase": {
      "content":{
        "query": "like elasticsearch",
        "slop": 3  // 默认1
      }
    }
  }
}

```

### multi_match

```js
get /testcross/_search
{
  "query":{
    "multi_match": {
      "query": "湖北省 武汉市 江汉区",
      "fields": ["provice","city^2","area"],
      "type": "cross_fields", //  跨字段查询 
      "operator": "and" // 或者用 or 
    }
  }
}
// 如果 provice， city 及area 每个词的权重不同， 比如 想要把city权重放的更高点，让权重优先的更考前的返回，我们可以直接在fields中计入 权重计算, 可以看到 city 被我改成了 city ^ 2 就是权重扩大 2倍，默认都是1倍
```

### Rescore重积分

> 优化 近似搜索的查询性能，一般就是减少要进行proximity（近似） match搜索的document数量。
>
> 参考博客： https://blog.csdn.net/u010134642/article/details/125577419

1. 用match query先过滤出需要的数据
2. 然后再用proximity match来根据term距离提高doc的分数
3. 同时proximity match只针对每个shard的分数排名前n个doc起作用，来重新调整它们的分数，这个过程称之为rescorte，重计分，因为一般用户会分页查询，只会看到前几页的数据，所以不需要对所有结果进行proximity match操作
4. 重积分参数 window_size 默认值是from和size参数值之和，它指定了每个分片上参与二次评分的文档个数, 表示从shared分片数据中取每个分片的多少条数据， 假如window_size=20,你现在有3个shared分片，那么结果就是有20*3=60条数据
5. 重积分参数 query_weight 查询权重，默认值是1，原始查询得分与二次评分的得分相加之前 ，乘 该值 提高原始查询得分
6. 重积分参数 rescore_query_weight 二次评分查询的权重值，默认值是1，二次评分查询得分在与原始查询得分相加之前 ， 乘该值 提高二次评分查询得分
7. rescore_mode 二次评分模式，默认为total，可用的选项有total、max、min、avg和mutiply。

> 查询 like elasticsearch 重新计算分数， query_weight =10 ，表示 原分数 * 10 + 积分后的分数

```js
get /testcopy/_search
{
  "query":{
    "match": {
      "content": "like elasticsearch"
    }
  },
  //rescore重积分与query平级
 "rescore" : {
      "window_size" : 50,
      "query" : {
         "rescore_query" : {
            "match_phrase" : {
               "message" : {
                  "query" : "like elasticsearch",
                  "slop" : 4
               }
            }
         },
         "query_weight" : 10,
         "rescore_query_weight" : 1
      }
   }
}
```

