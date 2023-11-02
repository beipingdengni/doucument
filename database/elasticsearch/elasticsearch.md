## Elasticearch



中文文档：https://github.com/elasticsearch-cn/elasticsearch-definitive-guide

githup中文文档地址：https://github.com/elasticsearch-cn/elasticsearch-definitive-guide



## nginx配置（有密码）

```
proxy_set_header Authorization "Basic xxxxxxx"; // echo -n 'user:password' | base64
```



## ES优化分段

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

