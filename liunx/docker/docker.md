## Docker基本操作使用

> docker run -it -d --rm --privileged=true --restart=always -p 宿主机:容器port -v 宿主机目录:容器目录

### mac安装使用

> 启动一个容器，暴露2375端口

```sh
docker run -it -d --name=socat -p 2375:2375 -v /var/run/docker.sock:/var/run/docker.sock bobrik/socat TCP4-LISTEN:2375,fork,reuseaddr UNIX-CONNECT:/var/run/docker.sock
```

## 基本命令操作

### 启动容器

```shell
1、查看镜像：docker images 
2、拉去：docker pull 镜像
3、运行容器：docker run -it -d --rm --privileged=true --restart=always -p 宿主机:容器port -v 宿主机目录:容器目录
  解释：
    -id 创建守护式容器
    --privileged=true 授予root权限（挂载多级目录必须为true，否则容器访问宿主机权限不足）
    --name=名字 给你的容器起个名字
    -p 宿主机端口：容器端口映射
    -v 宿主机目录：容器目录 目录挂载
4、查看容器内： docker exec -it 容器id cat /nexus-data/admin.password

# docker run --name 容器名字 --rm -it -d -p [ip:]主机端口:容器端口 -v 主机文件:容器文件 --dns=ip 镜像名称 [命令
# --dns-search=DOMAIN 
# 设置容器的主机名: -h hostname 或 --h=hostname 
# 设置环境变量: -e key1=value1
# 指定文件读取环境配置: --env-file=[]
# 设置运行的用户: -u 用户名
# 设置容器可使用的最大内存: -m 内存大小值
# 设置工作目录: -w 目录
# 也可以复用其他容器的卷的设置: -volumes-from 其他容器名或id
```

### volume

#### 基础操作

```shell
docker volume create my-vol
docker volume ls
docker volume inspect my-vol
docker volume rm my-vol
docker run -d --name devtest --mount source=myvol2,target=/app nginx:latest

# 完整的创建挂在volume
docker service create \
--mount 'type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>,"volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw"'
--name myservice \
<IMAGE>

#清除volume资源
docker volume prune -f

# 查看服务
docker service ps devtest-service
docker service rm devtest-service
```

#### 单容器（nfs）

```sh
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=<NFS服务器IP地址>,rw \
  --opt device=:/path/to/shared_folder \
  --name <volume-name>
# 案例
docker volume create --driver local \
	--opt type=nfs \
	--opt o=addr=192.168.11.129,rw \
	--opt device=:/nfsdir \
	--name volume-nfs
	
# java代码设置挂载
# Map<String,String> map = new HashMap<>();
# map.put("type",nfs);
# map.put("o","addr=192.168.11.129,rw");
# map.put("device",":/nfsdir");
# Volume volume = Volume.buidler().name("test").driverOpts(map).build();
```

#### 全局(挂载nfs)

```shell
docker service create --mode global \
  --name web122 \
  --publish 8082:80 \
  --mount 'type=volume,source=nfs122,target=/home/nfs,volume-driver=local,volume-opt=type=nfs,volume-opt=device=:/home/nfs,volume-opt=o=addr=192.168.56.120' \
 nginx:1.22
```

#### nfs全局关联 docker-compose.yml

```yaml
version: "3"
services:
  web:
    image: nginx:1.22
    volumes:
      - nfs121:/data
    ports:
      - "8081:80"  
    deploy:    
       mode: global  
volumes:
  nfs121:
    driver: local
    driver_opts:
      type: "nfs"
      o: "addr=192.168.56.120,nolock,soft,rw"
      device: ":/home/nfs"
```



### 阿里云服务：

https://helpcdn.aliyun.com/product/25972.html?spm=a2c4g.11186623.6.540.14f66d24C3kn7M

### Dockerfile编写参考

https://www.cnblogs.com/panwenbin-logs/p/8007348.html

### Docker 日志收集

```
docker log 

参考如下
log-pilot
	https://www.jianshu.com/p/120c77d155c4
```

