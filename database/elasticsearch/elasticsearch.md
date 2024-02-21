## Elasticearch



中文文档：https://github.com/elasticsearch-cn/elasticsearch-definitive-guide

githup中文文档地址：https://github.com/elasticsearch-cn/elasticsearch-definitive-guide



## nginx配置（有密码）

```
proxy_set_header Authorization "Basic xxxxxxx"; // echo -n 'user:password' | base64
```

## ES优化

### 将terms修改为should 使用

https://blog.csdn.net/hellozhxy/article/details/132065874

```json
# 优化前
{
  "query": {
    "bool": {
      "filter": [{"terms": {"f_status": [1,2]}}]
    }
  }
}

# 优化后
{
  "query": {
    "bool": {
      "filter": [{
          "bool": {
            "should": [{"term": {"f_status": 1}},{"term": {"f_status": 2}}],
            "minimum_should_match": "1"
          }}]
    }
  }
}
```



## ES优化分段合并

指导文档： https://zhuanlan.zhihu.com/p/647279604

```shell
#1、force merge
POST /logs-000001/_forcemerge?max_num_segments=1

#2、查看集群索引segment信息
GET _cat/segments?v

#3、获取热点线程
GET _nodes/hot_threads?human=true

#4、修改索引参数
PUT /index_name/_settings
{
	“refresh_interval”: “1s”,
	“segments_per_tier”: 15,
	“max_merged_segment”: “2G”
}

# 线程池
GET /_cat/thread_pool

# 内存查看
GET _cat/nodes?v&h=id,ip,port,r,ramPercent,ramCurrent,heapMax,heapCurrent,fielddataMemory,queryCacheMemory,requestCacheMemory,segmentsMemory

# 查询任务
GET _tasks?pretty

# 取消任务
POST _tasks/oTUltX4IQMOUUVeiohTt8A:12345/_cancel
```

## 别名

### 添加

```
POST /_aliases
{
    "actions": [
        { "add": { "index": "index_1", "alias": "my_alias" }},
        { "add": { "index": "index_2", "alias": "my_alias" }}
    ]
}
```

### 修改

```http
POST /_aliases
{
    "actions": [
        { "remove": { "index": "index_1", "alias": "my_alias" }},
        { "add": { "index": "index_3", "alias": "my_alias" }}
    ]
}
```

### 删除

```http
POST /_aliases
{
    "actions": [
        { "remove": { "index": "index_name", "alias": "alias_name" }}
    ]
}

DELETE /index_name/_alias/alias_name
DELETE /_all/_alias/alias_name  # 如果别名指向多个索引，并且你想要删除这个别名与所有相关索引的关联，可以省略索引名：
DELETE /*/_alias/alias_name   # 或者使用通配符匹配多个索引：

```

