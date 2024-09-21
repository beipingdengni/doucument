##  文章来源博客

​	https://blog.csdn.net/weixin_50902636/article/details/134216838

github：https://github.com/elasticsearch-dump/elasticsearch-dump

系列文章目录
------

第一章 [es集群搭建](https://blog.csdn.net/weixin_50902636/article/details/134210770?spm=1001.2014.3001.5501)  
第二章 [es集群基本操作命令](https://blog.csdn.net/weixin_50902636/article/details/134212006?spm=1001.2014.3001.5501)  
第三章 [es基于search-guard插件实现加密认证](https://blog.csdn.net/weixin_50902636/article/details/134212478?spm=1001.2014.3001.5501)  
第四章 [es常用插件](https://blog.csdn.net/weixin_50902636/article/details/134213194?spm=1001.2014.3001.5502)

* * *

#### 文章目录

*   [系列文章目录](#_0)
*   [前言](#_11)
*   [一、elasticdump是什么？](#elasticdump_21)
*   [二、安装elasticdump工具](#elasticdump_39)
*   *   [1.离线安装](#1_40)
    *   [2.在线安装](#2_70)
*   [三、elasticdump相关参数](#elasticdump_98)
*   [四、使用elasticdump进行数据备份](#elasticdump_577)
*   [五、使用elasticdump进行数据恢复](#elasticdump_617)


前言
--

在企业实际生产环境中,避免不了要对[es集群](https://so.csdn.net/so/search?q=es%E9%9B%86%E7%BE%A4&spm=1001.2101.3001.7020)进行迁移、数据备份与恢复，以此来确保数据的可用性及完整性。因此，就涉及到了数据备份与恢复。本章主要以elasticdump工具为主，讲解备份操作及恢复操作。其余备份和恢复方法暂不涉及，总结来自于生产实战。

* * *

| 迁移方式 | 使用场景 |
| --- | --- |
| logstash | 迁移全量​​​或​​增量数据​​，且对实时性要求不高的场景需要对迁移的数据通过 es query 进行简单的过滤的场景需要对迁移的数据进行复杂的过滤或处理的场景版本跨度较大的数据迁移场景，如 5.x 版本迁移到 6.x 版本或 7.x 版本 |
| elasticdump | 数据量较小​​的场景 |

一、elasticdump是什么？
-----------------

```txt
Elasticdump 是一个开源免费且用于导入和导出 Elasticsearch 数据的命令行工具。它提供了一种方便的方式来在不同的 Elasticsearch 实例之间传输数据，或者进行数据备份和恢复。

使用 Elasticdump，可以将 Elasticsearch 索引中的数据导出为 JSON 文件，或者将 JSON 文件中的数据导入到 Elasticsearch 索引中。它支持各种选项和过滤器，用于指定源和目标，包括索引模式、文档类型、查询过滤器等等。

主要特征包括：
	支持在Elasticsearch实例或者集群之间传输和备份数据。可以将数据从一个集群复制到另一个集群。
	支持不同格式的数据传输,包括JSON、NDJSON、CSV、备份文件等。
	可以通过命令行或者程序化的方式使用。命令行方式提供了便捷的操作接口。
	支持增量式同步,只复制目标集群中不存在的文档。
	支持各种认证方式连接Elasticsearch,如basic auth、Amazon IAM等。
	支持多线程操作,可以加快数据迁移的速度。
	开源免费,代码托管在GitHub上。

不足:
	elasticdump工具适用于备份单个索引或者整个es集群索引不太大的备份,如果es集群数据量较大，则需要使用logstash方法进行迁移恢复。
```

二、安装elasticdump工具
-----------------

### 1.离线安装

```yml
背景:
elasticdump工具依赖node环境
	1、离线安装node方式 (服务器为纯内网环境)
	 下载node安装包
		https://nodejs.org/dist/v16.14.0/node-v16.14.0-linux-x64.tar.xz
	上传至服务器
		rz node-v16.14.0-linux-x64.tar.xz
	解压并安装
		tar xf node-v16.14.0-linux-x64.tar.xz -C /usr/local/
		mv /usr/local/node-v16.14.0-linux-x64  /usr/local/node
    配置环境变量并重新加载
    	vim /etc/profile
		export NODE_HOME=/usr/local/node
		export PATH=$NODE_HOME/bin:$PATH
	source /etc/profile
    验证安装是否成功
    	node -v #安装成功此命令会返回node对应的版本信息
    	npm -v
    2、离线安装elasticdump
    本地下载对应的工具包
    	https://github.com/elasticsearch-dump/elasticsearch-dump/archive/v6.19.0.tar.gz
    上传至服务器
    	rz v6.19.0.tar.gz
    解压安装
    	tar xf v6.19.0.tar.gz -C /export/server/
    检查是否安装成功
    	elasticdump --version
```

### 2.在线安装

```yml
背景:
elasticdump工具依赖node环境
	1、在线安装node方式 (服务器可通外网)
	 #下载node安装包
		wget https://nodejs.org/dist/v16.14.0/node-v16.14.0-linux-x64.tar.xz
	 #解压并安装
		tar xf node-v16.14.0-linux-x64.tar.xz -C /usr/local/
		mv /usr/local/node-v16.14.0-linux-x64  /usr/local/node
     #配置环境变量并重新加载
    	vim /etc/profile
		export NODE_HOME=/usr/local/node
		export PATH=$NODE_HOME/bin:$PATH
	source /etc/profile
     #验证安装是否成功
    	node -v #安装成功此命令会返回node对应的版本信息
    	npm -v
     #或者直接采用yum命令安装node环境
     	yum -y install  nodejs npm
    2、在线安装elasticdump
    #设置npm源 否则安装会很慢
	npm config set registry https://registry.npm.taobao.org/
	#全局安装 
    npm install elasticdump -g
    #检查是否安装成功
    elasticdump --version
```

三、elasticdump相关参数
-----------------

```shell
	#相关参数 可以使用elasticdump --help查看
	--input: 指定输入的源位置 Elasticsearch 实例或JSON文件;
 	--output: 指定输出的目标位置 Elasticsearch实例或JSON文件;
	--type:指定要操作的数据类型，包括index、alias、template、data analyzers等;
	--searchBody:对于输入为Elasticsearch实例时，指定一个JSON对象作为查询参数;
	--limit:限制导出的文档数。通的导入导出是100条数据一次，如果是大批量数据的话就很耗时间。limit是一个限制大小的参数，可以根据需求来进行调整其大小。
	--inputIndex:指定输入源 Elasticsearch实例的索引名称;
	--ignore-errors:忽略出错的文档，继续导出其余文档。
	--scrollTime:设置scroll时间，以毫秒为单位。默认为10分钟。
	--timeout:设置请求超时时间，以毫秒为单位。默认为30秒。
	--support-big-int 支持大数类型
	--big-int-fields 指定支持的字段，默认是''(default '')
	--bulk-1imit:设置批量操作中的文档数量限制默认为1000
```

```yaml
官网参考
	https://github.com/elasticsearch-dump/elasticsearch-dump
	查看Options相关参数设置

#AWS specific options
--s3AccessKeyId AWS access key ID
--s3SecretAccessKey AWS secret access key
--s3Region AWS region
--s3Bucket Name of the bucket to which the data will be uploaded
--s3RecordKey Object key (filename) for the data to be uploaded
--s3Compress gzip data before sending to s3

将数据从S3(AWS)导入ES (使用 s3urls)
  elasticdump \
    --s3AccessKeyId "${access_key_id}" \
    --s3SecretAccessKey "${access_key_secret}" \
    --input "s3://${bucket_name}/${file_name}.json" \
    --output=http://production.es.com:9200/my_index
导出ES 数据到 S3(AWS) (使用 s3urls)
  elasticdump \
    --s3AccessKeyId "${access_key_id}" \
    --s3SecretAccessKey "${access_key_secret}" \
    --input=http://production.es.com:9200/my_index \
    --output "s3://${bucket_name}/${file_name}.json"

从 MINIO (s3 compatible) 导入数据到 ES (使用 s3urls)
	elasticdump \
  --s3AccessKeyId "${access_key_id}" \
  --s3SecretAccessKey "${access_key_secret}" \
  --input "s3://${bucket_name}/${file_name}.json" \
  --output=http://production.es.com:9200/my_index
  --s3ForcePathStyle true
  --s3Endpoint https://production.minio.com
导出ES 数据到 MINIO (s3 compatible) (使用 s3urls)
	elasticdump \
  --s3AccessKeyId "${access_key_id}" \
  --s3SecretAccessKey "${access_key_secret}" \
  --input=http://production.es.com:9200/my_index \
  --output "s3://${bucket_name}/${file_name}.json"
  --s3ForcePathStyle true
  --s3Endpoint https://production.minio.com
```

四、使用elasticdump进行数据备份
---------------------

```shell
1、数据备份之备份单个索引
当es集群配置了分词器后,首先需要导出分词
#导出分词器的时候要特别注意，我们只能根据索引单个导，不能全部导出，全部导出会出现索引不存在的错误：
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump --input http://用户名:密码@ip地址:9200/索引名 --output /export/分词文件名.json --type=analyzer(指定导出分词) --limit=10000 (限制大小)

导出mapping结构
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump --input http://用户名:密码@ip地址:9200/索引名 --output /export/mapping文件名.json --type=mapping(指定导出结构)  --limit=10000    #导出结构

导出数据
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump --input http://用户名:密码@ip地址:9200/索引名 --output /export/数据文件名.json --type=data(导出数据)   --limit=10000        #导出数据

elasticdump \
  --input=http://production.es.com:9200/my_index \
  --searchBody="{\"query\":{\"term\":{\"username\": \"admin\"}}}"
  #--input-params="{\"preference\":\"_shards:0\"}"  复制单个分片
  #--searchBody=@/data/searchbody.json  
  --output=$ \
  | gzip > /data/my_index.json.gz

```

```shell
2、数据备份之备份多个索引(前提是总索引数据不超过1G)
查看全部索引大小 如下所示
curl -X GET http://ip:9200/_cat/indices?v -u用户名:密码
health  status index  uuid  pri  rep docs.count docs.deleted store.size pri.store.size
green    open  test1  xxx   1     1   100104      0            508mb      508mb

index: 索引名称
docs.count: 索引中文档总数
store.size: 索引占用磁盘空间大小
pri.store.size: 主分片占用磁盘空间大小

使用awk命令获取索引名称并保存至unidom.txt文件中
	curl -X GET http://ip:9200/_cat/indices?v -u用户名:密码 | grep -v "status" |awk '{print $3}' > unidom.txt

编写脚本文件进行多索引备份,详细脚本如下:
#!/bin/bash
for item in `cat /root/unidom.txt`
do
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump --input http://icos:2y5CJH9S660ao70r3@10.20.0.8:9200/$item --output /export/unidom/analyzer/${item}_analyzer.json --type=analyzer
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump --input http://icos:2y5CJH9S660ao70r3@10.20.0.8:9200/$item --output /export/unidom/mapping/${item}_mapping.json --type=mapping
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump --input http://icos:2y5CJH9S660ao70r3@10.20.0.8:9200/$item --output /export/unidom/data/${item}_data.json --type=data
done

执行上述脚本，并输出对应的脚本执行日志
nohup ./script.sh >>backup.log 2>&1 & #执行脚本script.sh时将错误输出2以及标准输出1都一起以附加写方式导入backup.log文件
当最终执行完脚本后查看日志中末尾是否有success字样，这样即可备份成功。
```

五、使用elasticdump进行数据恢复
---------------------

```shell
1、检查新es集群中是否存在要导入的索引名称,如果不存在则先创建索引名称
curl -X PUT http://用户:密码@IP地址:9200/索引名 -u用户:密码
```

```shell
2、先导入分词
#!/bin/bash
#ls /export/cityos-oss/analyzer |awk -F'_' '{print $1}' 命令主要是拿出备份文件以json结尾的文件名
for item in `ls /export/backup_es/analyzer |awk -F'_' '{print $1}'` 
do
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump --input /export/backup_es/analyzer/${item}_analyzer.json --output http://用户:密码@IP地址:9200/$item --type=analyzer
done
执行上述脚本，并输出对应的脚本执行日志
nohup ./analyzer_in.sh >>/analyzer.log 2>&1 & #执行脚本script.sh时将错误输出2以及标准输出1都一起以附加写方式导入/analyzer.log文件
```

```shell
3、导入mapping结构
#!/bin/bash
for item in `ls /export/backup_es/mapping |awk -F'_' '{print $1}'`
do
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump --input /export/backup_es/mapping/${item}_mapping.json --output http://用户:密码@IP地址:9200/$item --type=mapping
done
执行上述脚本，并输出对应的脚本执行日志
nohup ./mapping_in.sh >>mapping.log 2>&1 & #执行脚本script.sh时将错误输出2以及标准输出1都一起以附加写方式导入mapping.log文件
```

```shell
4、导入数据
#!/bin/bash
for item in `ls /export/backup_es/data |awk -F'_' '{print $1}'`
do
/export/server/elasticdump/elasticsearch-dump/bin/elasticdump--input /export/backup_es/data/${item}_data.json --output http://用户:密码@IP地址:9200/$item --type=data
done
执行上述脚本，并输出对应的脚本执行日志
nohup ./data_in.sh >>data.log 2>&1 & #执行脚本script.sh时将错误输出2以及标准输出1都一起以附加写方式导入data.log文件
```

```shell
5、导入完成后进行大小验证
curl -X GET http://ip:9200/_cat/indices?v -u用户名:密码
查看索引名称，索引中文档总数， 索引占用磁盘空间大小，主分片占用磁盘空间大小是否与源es集群查出来的索引大小一致。
```



简介版本使用

```sh
#--input "csv:///data/cars.csv"
# 导出数据
elasticdump \
--input=http://es用户:es密码@esip:es端口/索引名 \
--searchBody="{\"query\":{\"range\":{\"@timestamp\":{\"gte\":\"2023-06-01T00:00:00\",\"lt\":\"2023-06-01T23:59:59\"}}},\"_source\":[\"message\"]}" \
--output=my_index_mapping.json \
--type=data \
--limit=10000 \   
--concurrency=15 \
--scrollTime=10m
#--searchBody:导出指定条件的数据，本例指定了了时间范围和只导_source里面的message值
#--output:导出到本地my_index_mapping.json文件
#--type:导出类型 data
#--limit:每个批次导出数量10000条，类似分页每页的数量
#--concurrency:指定 worker 的数量。默认情况下并发 worker 的数量为 3，可以根据需要适当增加 worker 数量来提高导出速度。
#--scrollTime:指定滚动窗口的时间，一旦滚动时间结束，Elasticsearch 就会关闭游标，这是一种较好的方式来减少游标的数量和内存占用。
#此命令在我本地电脑执行了18min，导出了153W+数据，文件大小1.2G+，仅供参考
```

