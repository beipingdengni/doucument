

## 查询相应时间长

```shell
curl 'http://127.0.01:9200/cs_event_info/_search' \
  -o /dev/null -s -w '\nDNS解析时长：%{time_namelookup}\n建立tcp时长：%{time_connect}\n客户端到服务器时长：%{time_starttransfer}\n从开始到结束时长：%{time_total}\n下载速度：%{speed_download}\n' \
  -v -H 'Content-Type: application/json' \
  -d '{"query":{"bool":{"filter":[{"term":{"companyId":{"value":"3"}}}]}}}'
```

> 核心 需求打印返回值，删除掉  `-o /dev/null`
>
> ```sh
> curl http://baidu.com -o /dev/null -s -w '\nDNS解析时长：%{time_namelookup}\n建立tcp时长：%{time_connect}\n客户端到服务器时长：%{time_starttransfer}\n从开始到结束时长：%{time_total}\n下载速度：%{speed_download}\n'
> ```

## 调用传递参数

> 在输入数据为json格式的时候直接使用`-D`选项传参 curl 会默认对数据进行压缩后传递,但如果将`--data`选项修改为 `--data-binary`时,便可以完整的将文本的内容进行传输了

| 选项                        | 功能                   |
| --------------------------- | ---------------------- |
| --basic                     | 使用HTTP基本验证       |
| -B/--use-ascii              | 使用ASCII文本传输      |
| -d/--data <data>            | HTTP POST方式传送数据  |
| --data-ascii <data>         | 以ascii的方式post数据  |
| --data-binary <data>        | 以二进制的方式post数据 |
| --connect-timeout <seconds> | 设置最大请求时间       |

### 文件传递

```sh
curl localhost:9000/sync -H "Content-type:application/json" -X POST --data @req_data.json
curl localhost:9000/sync -H "Content-type:application/json" -X POST --data-binary @req_data.json
```

