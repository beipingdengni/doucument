## 调用接口

### 有密码

> http://elastic:password@host:port/

## 数据迁移

### 内部索引迁移

```json
// GET _tasks?detailed=true&actions=*reindex //查看任务
// GET /_tasks/r1A2WoRbTwKZ516z6NEs5A:36619 //查看任务详细
// POST _tasks/r1A2WoRbTwKZ516z6NEs5A:36619/_cancel // 取消任务

POST _reindex?slices=5&refresh&wait_for_completion=false  // 手动设置分片、自动设置分片
{
	"conflicts": "proceed",	//有冲突继续，默认是有冲突终止
	"size":1000,	//设定条数  
	"source": {    
	   "index": "twitter", 	//也可以为 ["twitter", "blog"]
		"type": "tweet", 	// 或["type1","type2"] 	//红字限制范围 ，非必须  限制文档
		"query": { "term": { "user": "kimchy" } }，	//添加查询来限制文档
		"sort": { "date": "desc" }, 	//排序
		"_source": ["user", "tweet"]，	//指定字段
		"size": 100,	//滚动批次1000更改批处理大小:
	 },  
  "dest": {    
		"index": "new_twitter",
		"op_type": "create",	//设置将导致_reindex只在目标索引中创建丢失的文档,create 只插入没有的数据
		"version_type": "external"，	//没有设置 version_type或设置为internal 将覆盖掉相同id的数据,设置为external 将更新相同ID文档当version比较后的时候
		"routing": "=cat",	//将路由设置为cat
		"pipeline": "some_ingest_pipeline",	//指定管道来使用Ingest节点特性
   },
	"script": { // 执行脚本 
	   "source": "if (ctx._source.foo == 'bar') {ctx._version++; ctx._source.remove('foo')} ",			 	
		  "lang": "painless" 
	}
}

// 重建索引，将原索引中的flag字段重命名为tag字段
// "source": "ctx._source.tag = ctx._source.remove(\"flag\")"
// 源：{"_source": {"text": "words words","flag": "foo"}}  目标：{"_source": {"text": "words words","tag": "foo"}}
```

> "version_type": "external"，external表示外部的，将 version_type 设置为 external 将导致 Elasticsearch 保留源中的版本，创建任何丢失的文档，并更新目标索引中版本比源索引中版本旧的任何文档。

### 远程数据源恢复

```json
POST _reindex
{
  "conflicts": "proceed",
  "source": {
    "remote": {
      "host": "http://xxx.xxx.com:9200",
      "username": "xxx",
      "password": "xxxxxxxxx",
      "socket_timeout": "1m",
      "connect_timeout": "30s"
    },
    "index": "case_cause",
    "size": 5000  //滚动批次1000更改批处理大小:
  },
  "dest": {
    "index": "case_cause",
    "op_type": "create",
    "routing": "=cat"
  }
}
```



## ES自带备份恢复

### 启动ES配置备份路径

```yaml
config/elasticsearch.yml

path.repo: ["/usr/local/elasticsearch/snapshot"]
```

### 建立仓库

```sh
curl  -H "Content-Type: application/json" \
	-XPUT \
	-u elastic:xxx \
	http://ES的IP:9200/_snapshot/backup \
	-d '{"type": "fs","settings": {"location": "/usr/local/elasticsearch/snapshot"}}'
# 使用kibana
PUT _snapshot/backup
{
  "type": "fs",
  "settings": {
    "location": "data_bk",
    "compress": true,    #compress   #是否压缩，默认为是。
    "max_snapshot_bytes_per_sec" : "50mb",
    "max_restore_bytes_per_sec" : "50mb"
  }
}

# 备注
# location 是指定备份配置
# compress   #是否压缩，默认为是。
# max_snapshot_bytes_per_sec   #每一个节点快照速率。默认40mb/s。  
# max_restore_bytes_per_sec    #节点恢复速率。默认40mb/s。
```

### 删除备份数据

```
curl -XDELETE http://ES的ip:端口/_snapshot/backup/bk_20190926
```

### 开始备份数据

```
curl -XPUT http://ES的ip:端口/_snapshot/backup/bk_20190926?wait_for_completion=true
```

### 查看备份数据

```
curl -XGET http://ES的ip:端口/_snapshot/backup/_all
```

### 恢复数据

#### 备份data文件夹

```
data文件夹其实就是当前ES的数据存储地，防止恢复数据出现异常，先把ES目录下面的data目录备份一下。
tar -cvf data-20190626.tar.gz data
```

#### 清空数据

```go
curl -XDELETE http://ES的ip:端口/_all
```

#### 恢复数据

```
curl -XPOST http://ES的ip:端口/_snapshot/backup/bk_20190926/_restore
```



## 使用工具elasticdump

### 安装

> npm install elasticdump -g

github地址：https://github.com/elasticsearch-dump/elasticsearch-dump

### 导出

> --limit 10000 限制条数据
>
> --overwrite 输出文件已存在覆盖
>
> --type  (default: data, options: [index, settings, analyzer, data, mapping, policy, alias, template, component_template, index_template])

```sh
elasticdump --input=http://localhost:9200/old_index --output=exported_data.json     --type=analyzer
elasticdump --input=http://localhost:9200/old_index --output=exported_settings.json --type=settings
elasticdump --input=http://localhost:9200/old_index --output=exported_mapping.json  --type=mapping
elasticdump --input=http://localhost:9200/old_index --output=exported_index.json    --type=index
elasticdump --input=http://localhost:9200/old_index --output=exported_data.json     --type=data



增加查询 【默认是数据导出】
elasticdump --input=http://localhost:9200/old_index --output=exported_data.json --limit 10000 --searchBody {\"query\":{\"bool\":{\"must\":[{\"range\":{\"JD\":{\"from\":116.388474,\"to\":116.67818}}}]}} 
```

### 导入

```sh
elasticdump --input=exported_settings.json --output=http://localhost:9200/new_index --type=analyzer
elasticdump --input=exported_settings.json --output=http://localhost:9200/new_index --type=settings
elasticdump --input=exported_mapping.json  --output=http://localhost:9200/new_index --type=mapping
elasticdump --input=exported_index.json    --output=http://localhost:9200/new_index --type=index 
elasticdump --input=exported_data.json     --output=http://localhost:9200/new_index --type=data
```

### 导入和出两ES

```
elasticdump --input=http://ip:9200/old_index --output=http://127.0.0.1:9200/new_index

elasticdump \
  --input=http://elastic:password@host:port/old_index \
  --output=http://elastic:password@host:port/new_index \
  --type=data --limit 1000 --support-big-int
```

