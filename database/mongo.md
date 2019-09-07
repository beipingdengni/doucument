## mongo 操作使用



##### 链接数据库

```
mongod  --port 27017 --dbpath /data/db1
mongo --port 27017 -u "lyl" -p "123456" --authenticationDatabase "admin"

db的帮助  db.help();
表的帮助  db.tableName.help();

```

##### 添加密码：

```
use admin 
db.createUser(
  {
    user: "adminUser",
    pwd: "adminPass",
    roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
  }
)

创建普通用户：

use foo 
db.createUser(
  {
    user: "simpleUser",
    pwd: "simplePass",
    roles: [ { role: "readWrite", db: "foo" },
             { role: "read", db: "bar" } ]
  }
)
// db.createUser({user: "root",pwd: "123456",roles: [ { role: "readWrite", db: "tian" }]});
```

##### 处理密码【密码忘记】

```
vim /etc/mongodb.conf          # 修改 mongodb 配置，将 auth = true 注释掉，或者改成 false
service mongodb restart        # 重启 mongodb 服务
 
mongo                          # 运行客户端（也可以去mongodb安装目录下运行这个）
use admin                      # 切换到系统帐户表
db.system.users.find()         # 查看当前帐户（密码有加密过）
db.system.users.remove({})     # 删除所有帐户
db.addUser('admin','password') # 添加新帐户
 
vim /etc/mongodb.conf          # 恢复 auth = true
service mongodb restart        # 重启 mongodb 服务
```



##### 用户权限：

```
Read：允许用户读取指定数据库
readWrite：允许用户读写指定数据库
dbAdmin：允许用户在指定数据库中执行管理函数，如索引创建、删除，查看统计或访问system.profile
userAdmin：允许用户向system.users集合写入，可以找指定数据库里创建、删除和管理用户
clusterAdmin：只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限。
readAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读权限
readWriteAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的读写权限
userAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的userAdmin权限
dbAdminAnyDatabase：只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限。
root：只在admin数据库中可用。超级账号，超级权限
```

##### 服务中配置UrL

```
mongodb://your.db.ip.address:27017/foo
mongodb://simpleUser:simplePass@your.db.ip.address:27017/foo
```

