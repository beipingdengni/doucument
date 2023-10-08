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

