### 关于自己学习笔记相关记录

------

```
ssh-keygen -t rsa -C "your_email@example.com"

输入生成路径，下生成文件名称：id_rsa_qq  默认都在~/.ssh/ 路径下

ssh-add ~/.ssh/id_rsa_qq

git remote set-url origin git地址
# git remote set-url --add origin it@github.com:beipingdengni/doucument.git
git remote add origin git地址
或者
git remote add 自定义远程名称 git地址


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