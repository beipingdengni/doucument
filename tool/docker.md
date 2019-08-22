## Docker基本操作使用



##### 基本命令操作

```
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
5、
```

