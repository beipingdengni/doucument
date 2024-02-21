# docker

## 创建容器

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

# docker run --name 容器名字 --rm -it -d -p [ip:]主机端口:容器端口 -v 主机文件:容器文件 --dns=ip 镜像名称:tag
# --dns-search=DOMAIN 
# 设置容器的主机名: -h hostname 或 --h=hostname 
# 设置环境变量: -e key1=value1
# 指定文件读取环境配置: --env-file=[]
# 设置运行的用户: -u 用户名
# 设置容器可使用的最大内存: -m 内存大小值
# 设置工作目录: -w 目录
# 也可以复用其他容器的卷的设置: -volumes-from 其他容器名或id
```

## 镜像中心

### 容器界面化Portainer

```shell
# 运行Portainer
docker run -d --restart=always --name portainer \
	-p 9000:9000 \
	-v /var/run/docker.sock:/var/run/docker.sock 
	-v /data/portainer/data:/data \
	-v /data/portainer/public:/public \
	portainer/portainer:1.20.2

汉化:
	运行Portainer内操作
 	创建目录，并解压文件
	mkdir -p /data/portainer/data /data/portainer/public
	cd /data/portainer
	wget https://dl.quchao.net/Soft/Portainer-CN.zip
	unzip Portainer-CN.zip -d public
```

