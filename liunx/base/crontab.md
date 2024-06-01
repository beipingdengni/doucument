

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

## crontab命令权限

/etc/cron.deny

说明：该文件中所列用户不允许使用crontab命令

/etc/cron.allow

说明：该文件中所列用户允许使用crontab命令

## 系统任务调度配置文件

文件位置：/etc/crontab

1. shell变量指定了系统要使用的shell
2. path变量指定了系统执行命令的路径
3. mailto变量指定了crond的任务执行信息将通过电子邮件发给root（mailto空表示不发送）
4. HOME变量指定使用者运行的路径,这里是根目录

```bash
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/ 

# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

