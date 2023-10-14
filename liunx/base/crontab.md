

## 检查安装crontab

> yum install crontabs 

服务启动/关闭和查看

```
systemctl status crond
systemctl start crond
systemctl stop crond
systemctl reload crond
```

### 检查定时任务

> crontab -l

### 编辑任务

> crontab -e

规则编写如下

```sh
# 时间规则 + 执行命令
# 每天凌晨三点执行
0 3 * * * /etc/init.d/nginx reload
```

