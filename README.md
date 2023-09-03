### 关于自己学习笔记相关记录

------

```
ssh-keygen -t rsa -C "your_email@example.com"

输入生成路径，下生成文件名称：id_rsa_qq  默认都在~/.ssh/ 路径下

# 本地电脑产生多个key，用于推送
ssh-add ~/.ssh/id_rsa_qq

git remote set-url origin git地址
# git remote set-url --add origin it@github.com:beipingdengni/doucument.git
git remote add origin git地址
或者
git remote add 自定义远程名称 git地址

用法：git remote [-v | --verbose]
  或：git remote add [-t <分支>] [-m <master>] [-f] [--tags | --no-tags] [--mirror=<fetch|push>] <名称> <地址>
  或：git remote rename <旧名称> <新名称>
  或：git remote remove <名称>
  或：git remote set-head <名称> (-a | --auto | -d | --delete | <分支>)
  或：git remote [-v | --verbose] show [-n] <名称>
  或：git remote prune [-n | --dry-run] <名称>
  或：git remote [-v | --verbose] update [-p | --prune] [(<组> | <远程>)...]
  或：git remote set-branches [--add] <名称> <分支>...
  或：git remote get-url [--push] [--all] <名称>
  或：git remote set-url [--push] <名称> <新的地址> [<旧的地址>]
  或：git remote set-url --add <名称> <新的地址>
  或：git remote set-url --delete <名称> <地址>

```



```
netflix ribbon:
参考：
	https://blog.csdn.net/qq_41895733/article/details/117464837
	https://blog.csdn.net/mytobaby00/article/details/79837382

netflix feign:
参考：https://www.jianshu.com/p/3d597e9d2d67/
openfeign:
jax-rs 参考：https://blog.csdn.net/chengqiuming/article/details/81140852
参考：https://blog.csdn.net/MrSpirit/article/details/80347876

只了解：
Dropwizard 
参考：https://segmentfault.com/a/1190000000359827

```



#### kaluos/Thiem  tian



```
mac brew install 

Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To have launchd start nginx now and restart at login:
  brew services start nginx
Or, if you don't want/need a background service you can just run:
  nginx
```



```

通用分割线

=============================================================================


```



#### [工具](./tool)

aliyun maven包管理中心：[https://maven.aliyun.com](https://maven.aliyun.com/)

###### [gradle](https://github.com/beipingdengni/doucument/blob/master/tool/gradle.md)

###### maven

###### docker

###### kubetl-help

------

#### [database](./database)

###### [mysql](https://github.com/beipingdengni/doucument/blob/master/database/mysql.md)

###### postgresql

###### monogodb

###### [redis](https://github.com/beipingdengni/doucument/blob/master/database/redis.md)

------

#### [JAVA](./java)

###### spring 

###### spring mvc

###### spring boot

###### spring cloud

------

#### Python

###### Flask

###### requests

###### django

------

##### GO 

###### goworker

###### beego



### 网关

Openresty: https://openresty.org/cn/

ApiSix：https://github.com/apache/incubator-apisix

kong：https://github.com/Kong/kong

------