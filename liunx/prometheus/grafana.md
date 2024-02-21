## 安装运行

### docker

```shell
docker run -d -p 3000:3000 --name=grafana grafana/grafana-enterprise
# 时区: -e TZ=Asia/Shanghai
# 默认密码
# admin/admin

# root 权限进入容器
sudo docker exec -it -u 0 <container_id> /bin/bash

# 中文
/usr/share/grafana/conf/defaults.ini 
#default_language = en-US
default_language = zh-Hans
```

## 官方面板搜索

 https://grafana.com/grafana/dashboards/

