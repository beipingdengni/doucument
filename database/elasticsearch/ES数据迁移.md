## **logstash 方式**

logstash 方式迁移数据，注意：

- 若要全量迁移索引数据，源端索引需先停写

- 若只迁移部分索引数据，可通过 query 参数指定查询语句

### logstash 简介

Logstash 是 ELK（Elasticsearch、Logstash、Kibana）中的 L，ELK 的整个流程是先由 Logstash 从数据源中采集数据并进行处理，然后输出到 Elasticsearch 集群中，最后通过 Kibana 进行查看或展示。Logstash 相当于一个实时管道，支持对多种类型的数据源进行数据采集，如果数据源是 Elasticsearch 集群，那就是 **es 集群之间的数据迁移**。

![image-20231111131106353](imgs/ES数据迁移/image-20231111131106353.png)

Logstash 数据处理主要包括三个过程，分别是 Inputs、Filters、Outputs，其中：

- Inputs：用于数据采集，比如从源 es 集群查出数据

- Filters：用于数据处理，比如对数据进行字段过滤

- Outputs：用于数据输出，比如将处理好的数据写入到目的 es 集群中

### logstash 安装

官方下载地址：https://www.elastic.co/cn/downloads/logstash

logstash 的安装，原则上要保证与 es 集群版本一致，如果需要跨版本，尽量保证版本差异不要太大，以免产生兼容性问题。

```
# 安装logstash所在物理机，要保证与源端集群和目的端集群网络都相通 # 假如用户为root，workspace为根目录 / cd / # 下载安装包 wget https://artifacts.elastic.co/downloads/logstash/logstash-6.5.0.tar.gz # 解压 tar -zxvf logstash-6.5.0.tar.gz -C logstash-6.5.0
```

### logstash 配置

logstash 的配置文件放在 config 目录下，常用的有3个，分别是 logstash-sample.conf、logstash.yml，jvm.options，其中：

- logstash-sample.conf 文件为样例配置文件，可参考该文件自定义出一个业务配置文件，比如cp logstash-sample.conf es-migrate.conf，该 es-migrate.conf 作为业务配置文件非常重要。

```
vim es-migrate.conf
# 以下为自定义业务配置文件内容


input {
        elasticsearch {  # 定义数据源为 elasticsearch
            hosts => 源es集群地址
            user => 源es集群用户名
            password => 源es集群密码
            index => 源索引名称
            docinfo => 默认为false，设置为true，将会提取es文档的元信息，例如index、type、id等
            size => 设置每次scroll查询大小，默认为1000
            query => 指定查询语句
        }
}
# 对从源es集群中查询到的文档进行数据处理
filter {
        mutate {
            add_field => 添加字段
            remove_field => 删除字段
        }
}


output {
        elasticsearch {  # 定义目的源为 elasticsearch
            hosts => 目的es集群地址
            user => 目的es集群用户名
            password => 目的es集群密码
            document_type => "%{[@metadata][_type]}" # 保持源索引与目的索引type同步
            document_id => "%{[@metadata][_id]}" # 保持源索引文档与目的索引文档id同步
        }
```

- logstash.yml，该配置文件为 logstash 主配置文件。

```
vim logstash.yml
# 下面简单介绍一下常用到的一些参数配置


# 节点名称，具备唯一性，默认为logstash主机的主机名
node.name: logstast-node1
# logstash端口，默认为9600
http.port: 9600
# logstash及其插件所使用的数据路径，可自定义，默认路径为logstash家目录下的data目录
path.data: ../data
# logstash日志目录位置，可自定义，默认为logstash路径下的logs
path.logs: ../logs
# 日志级别，可配置 debug、info（默认）、warn、error、fatal等
log.level: info
# 在input阶段，单个工作线程将从输入中收集的最大事件数
pipeline.batch.size: 5000
# 在将一个较小的批发送到filter+output之前，轮询下一个事件时等待的时间（以毫秒为单位）
pipeline.batch.delay: 10
# logstash的工作线程数，默认为主机的cpu核数
pipeline.workers: 8
```

- jvm.options，该配置文件用于配置JVM相关参数，常修改 -Xms、-Xmx 这两个参数，默认为 1g，最大不要超过 32g。

```
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space


-Xms16g
-Xmx16g
```

### logstash 实战

- 背景：es 集群间数据迁移

```
源es集群：http://es-migrate-logstash-1.jdcloud.com:9200
用户名：admin
密码：xxxxxx
源索引：test-migrate


目的es集群：http://es-migrate-logstash-2.jdcloud.com:9200
用户名：admin
密码：xxxxxx
目的索引：test-migrate


注意：要保证源索引与目的索引的设置信息一致，否则可能导致源索引与目的索引字段类型等信息不一致
例如：1、查询出源索引的表结构信息，并根据此表结构提前在目的集群中创建出目的索引
     2、若源索引有对应的索引模版，可提前将该索引模版在目的集群中创建出
```

- vim es-migrate.conf

```
input {
        elasticsearch {
            hosts => ["http://es-migrate-logstash-1.jdcloud.com:9200"]
            user => "admin"
            password => "xxxxxx"
            index => "test-migrate"
            docinfo => true
            size => 5000
        }
}


filter {
        mutate {
            remove_field => ["@timestamp", "@version"]
        }
}


output {
        elasticsearch {
            hosts => ["http://es-migrate-logstash-2.jdcloud.com:9200"]
            user => "admin"
            password => "xxxxxx"
            index => "test-migrate"
            document_type => "%{[@metadata][_type]}"
            document_id => "%{[@metadata][_id]}"
        }
```

- vim logstash.yml

```
node.name: logstast-node1
http.port: 9600
path.data: /logstash-6.5.0/data
path.logs: /logstash-6.5.0/logs
log.level: info
pipeline.batch.size: 5000
pipeline.batch.delay: 10
pipeline.workers: 5
```

- vim jvm.options

```
# Xms represents the initial size of total heap space
# Xmx represents the maximum size of total heap space


-Xms16g
-Xmx16g
```

- 启动 logstash

```
nohup /logstash-6.5.0/bin/logstash -f /logstash-6.5.0/config/es_migrate.conf &
```

- 查看日志

```
...
[2023-03-23T14:31:06,522][INFO ][logstash.runner          ] Starting Logstash {"logstash.version"=>"6.5.0"}
[2023-03-23T14:31:11,149][INFO ][logstash.pipeline        ] Starting pipeline {:pipeline_id=>"main", "pipeline.workers"=>5, "pipeline.batch.size"=>5000, "pipeline.batch.delay"=>10}
[2023-03-23T14:31:11,906][INFO ][logstash.outputs.elasticsearch] Elasticsearch pool URLs updated {:changes=>{:removed=>[], :added=>[http://admin:xxxxxx@es-migrate-logstash-2.jdcloud.com:9200/]}}
[2023-03-23T14:31:11,939][INFO ][logstash.outputs.elasticsearch] Running health check to see if an Elasticsearch connection is working {:healthcheck_url=>http://admin:xxxxxx@es-migrate-logstash-2.jdcloud.com:9200/, :path=>"/"}
[2023-03-23T14:31:12,670][WARN ][logstash.outputs.elasticsearch] Restored connection to ES instance {:url=>"http://admin:xxxxxx@es-migrate-logstash-2.jdcloud.com:9200/"}
[2023-03-23T14:31:12,756][INFO ][logstash.outputs.elasticsearch] ES Output version determined {:es_version=>6}
[2023-03-23T14:31:12,761][WARN ][logstash.outputs.elasticsearch] Detected a 6.x and above cluster: the `type` event field won't be used to determine the document _type {:es_version=>6}
[2023-03-23T14:31:12,823][INFO ][logstash.outputs.elasticsearch] New Elasticsearch output {:class=>"LogStash::Outputs::ElasticSearch", :hosts=>["http://admin:xxxxxx@es-migrate-logstash-2.jdcloud.com:9200"]}
[2023-03-23T14:31:12,862][INFO ][logstash.outputs.elasticsearch] Using mapping template from {:path=>nil}
[2023-03-23T14:31:12,894][INFO ][logstash.outputs.elasticsearch] Attempting to install template {:manage_template=>{"template"=>"logstash-*", "version"=>60001, "settings"=>{"index.refresh_interval"=>"5s"}, "mappings"=>{"_default_"=>{"dynamic_templates"=>[{"message_field"=>{"path_match"=>"message", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false}}}, {"string_fields"=>{"match"=>"*", "match_mapping_type"=>"string", "mapping"=>{"type"=>"text", "norms"=>false, "fields"=>{"keyword"=>{"type"=>"keyword", "ignore_above"=>256}}}}}], "properties"=>{"@timestamp"=>{"type"=>"date"}, "@version"=>{"type"=>"keyword"}, "geoip"=>{"dynamic"=>true, "properties"=>{"ip"=>{"type"=>"ip"}, "location"=>{"type"=>"geo_point"}, "latitude"=>{"type"=>"half_float"}, "longitude"=>{"type"=>"half_float"}}}}}}}}
[2023-03-23T14:31:12,913][WARN ][logstash.pipeline        ] CAUTION: Recommended inflight events max exceeded! Logstash will run with up to 25000 events in memory in your current configuration. If your message sizes are large this may cause instability with the default heap size. Please consider setting a non-standard heap size, changing the batch size (currently 5000), or changing the number of pipeline workers (currently 5) {:pipeline_id=>"main", :thread=>"#<Thread:0x617612f6 run>"}
[2023-03-23T14:31:13,792][INFO ][logstash.pipeline        ] Pipeline started successfully {:pipeline_id=>"main", :thread=>"#<Thread:0x617612f6 run>"}
[2023-03-23T14:31:13,958][INFO ][logstash.agent           ] Pipelines running {:count=>1, :running_pipelines=>[:main], :non_running_pipelines=>[]}
[2023-03-23T14:31:14,587][INFO ][logstash.agent           ] Successfully started Logstash API endpoint {:port=>9600}
....
```

- logstash 进程结束后，可通过"GET _cat/indices/索引名称?v"查看目的索引的文档数是否与源索引文档数一致，来判断数据是否迁移完整。

## **reindex 方式**

reindex 方式迁移数据，注意：

- 若要全量迁移索引数据，源端索引需先停写
- 若只迁移部分索引数据，可通过 query 参数指定查询语句
- 2.1.2 版本es集群使用的是 reindexing plugin，使用方法不同于高版本

### 2.1.2 版本es集群 reindexing plugin 使用

https://github.com/codelibs/elasticsearch-reindexing

### reindex 简介

官方文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html

reindex 为 ES 5.X 版本之后提供的数据迁移功能，不需要额外安装，支持同集群索引迁移和跨集群索引迁移。

使用 reindex，要注意两点：

- 要求源端索引的元字段 _source 是打开的，默认就是打开的。

- reindex 过程并不会自动将源端索引的设置拷贝到目标索引，所以需要事先在目标集群（源集群和目标集群可以是同一个集群）中按照源端索引的表结构建立好目标索引。

reindex 适用于迁移数量量和索引数都较小的场景，迁移速度较慢，可在集群性能允许的情况下，通过调大 size 参数值来提升迁移速度，默认 size 大小为 1000。

reindex 配置

```
# 在目的集群中执行 reindex api 命令
# reindex一般执行过程耗时较长，加上wait_for_completion=false参数意为异步执行

POST _reindex?wait_for_completion=false
{
  "source": {
    "remote": {
      "host": 源集群域名信息,
      "username": 源集群用户名,
      "password": 源集群密码
    },
    "index": 源端索引,
    "size": 设置batch size大小，默认为1000，越大速度越快，但要注意集群性能影响,
    "query": 指定查询语句，只迁移查到的数据
  },
  "dest": {
    "index": 目的索引
  }
}
```

### reindex 实战

#### 同集群索引迁移

当需要更改已创建索引的主分片数、字段类型等信息时，es 不支持在现有的索引上直接修改，可以通过 reindex 重建索引并将数据迁移到新索引中。举例说明：

```
背景：
现有索引 test_source，分片设置为5主1副，因数据量越来越大，导致单分片数据量过大。
希望可以将 test_source 索引的分片设置调整为10主1副。


步骤：
1、先创建出目的索引 test_dest，分片设置为10主1副，并且 test_dest 索引的表结构要与
test_source 索引的表结构保持一致。


2、调用 reindex api 进行数据迁移
POST _reindex?wait_for_completion=false
{
  "source": {
    "index": "test_source",
    "size": 5000
  },
  "dest": {
    "index": "test_dest"
  }
} 


# reindex操作一般耗时较长，加上wait_for_completion=false参数意为异步执行
# reindex底层是scroll查询，可在集群性能满足的情况下，通过调大size参数的值，来加快迁移速度
# 可通过命令 GET _cat/tasks?v 查看集群中正在执行的任务
# 可通过命令 GET _tasks?actions=*reindex*&detailed 精确查看正在执行的reindex任务


3、等到 reindex 任务执行完成后，可通过查看 test_source 和 test_dest 索引的文档数是否一致来
判断数据迁移是否完整。


# 注意：这种方式改变了索引的名称，原来是test_source，现为test_dest，若业务侧希望不改变索引名称，
# 可使用别名的方式，比如创建别名test_source，来关联test_dest索引，但别名与索引名称不能重复，
# 若要创建别名test_source，需先删除索引test_source，请谨慎评估风险


POST /_aliases
{
    "actions" : [
        { "add" : { "index" : "test_dest", "alias" : "test_source" } }
  
```

#### 跨集群索引迁移

当需要跨集群进行索引迁移时，也可以使用 reindex 进行。举例说明：

- 背景：将 test-migrate 索引由源 es 集群迁移至目的 es 集群

```
源es集群：http://es-migrate-reindex-1.jdcloud.com:9200
用户名：admin
密码：xxxxxx
源索引：test-migrate


目的es集群：http://es-migrate-reindex-2.jdcloud.com:9200
用户名：admin
密码：xxxxxx
目的索引：test-migrate


注意：要保证源索引与目的索引的表结构信息一致，否则可能导致源索引与目的索引字段类型等信息不一致
例如：1、查询出源索引的表结构信息，并根据此表结构提前在目的集群中创建出目的索引
     2、若源索引有对应的索引模版，可提前将该索引模版在目的集群中创建出
```

- 在目的 es 集群中配置上源 es 集群的白名单信息

```
vim elasticsearch.yml
# 在目的集群的elasticsearch.yml文件中增加源es集群的白名单配置
reindex.remote.whitelist: “es-migrate-reindex-1.jdcloud.com:9200”
```

- 在目的 es 集群中调用 reindex api 命令，进行数据迁移

```
POST _reindex?wait_for_completion=false
{
  "source": {
    "remote": {
      "host": "http://es-migrate-reindex-1.jdcloud.com:9200",
      "username": "admin",
      "password": "xxxxxx",
    "size": 5000
    },
    "index": "test-migrate"
  },
  "dest": {
    "index": "test-migrate"
  }
}


# reindex操作一般耗时较长，加上wait_for_completion=false参数意为异步执行
# reindex底层是scroll查询，可在集群性能满足的情况下，通过调大size参数的值，来加快迁移速度
# 可通过命令 GET _cat/tasks?v 查看集群中正在执行的任务
# 可通过命令 GET _tasks?actions=*reindex*&detailed 精确查看正在执行的reindex任务
```

- 等到 reindex 任务执行完成后，可通过查看源 es 集群中 test-migrate 索引与目的 es 集群中 test-migrate 索引的文档数是否一致，来判断数据迁移是否完整。