# docker swarm

## 基础指令

docker swarm：集群管理，子命令有init, join, leave, update。（docker swarm --help查看帮助）

docker node：节点管理，子命令有accept, promote, demote, inspect, update, tasks, ls, rm。（docker node --help查看帮助）

docker service：服务创建，子命令有create, inspect, update, remove, tasks。（docker service--help查看帮助）

## 创建集群

### master

```sh
docker swarm init --advertise-addr 192.168.21.142
# 或生成一个token
```

### work

```sh
# 利用token添加节点到master
docker swarm join --token SWMTKN-1-4kdb7ysckerhxc6gxc3bpltkxtm8o45yq1ikyv1kie825zibhh-bhwazb9y8i3s92v8mpv5c06so 192.168.21.142:2377
```

## 指令操作

### 查看节点

> docker node ls

### 节点状态修改

> docker node update --availability drain node0313
>
> > 当node0313节点的状态改为drain后，那么该节点就不会接受task任务分发

swarm集群中node的availability状态可以为 active或者drain，其中：

​	active状态下，node可以接受来自manager节点的任务分派；
​	drain状态下，node节点会结束task，且不再接受来自manager节点的任务分派（也就是下线节点）

### 创建负载均衡网络

docker network create -d overlay nginx_net

docker network ls | grep nginx_net

### 部署服务

> docker service create --replicas 1 --network nginx_net --name my_nginx -p 80:80 nginx:latest 	#部署服务
>
> docker service ls	#查看正在运行的服务
>
> docker service ps my_nginx	#查询到哪个节点正在运行my_nginx这个服务
>
> docker service rm my_nginx	#删除指定的服务
>
> docker service inspect --pretty my_nginx	#查询swarm中的服务信息( --pretty 是格式化输出)

### 动态扩缩

> docker service scale my_nginx=4	#扩展到4个（扩容）
>
> docker service scale my_nginx=3	#减到3个（缩容）
>
> docker service update --replicas 3 my_nginx   # update命令是对服务参数进行修改的

### 容器升级

> docker service update --image nginx:new my_nginx
>
> > docker service update --image nginx --update-parallelism 2 --update-delay 5s my_cluster
> >
> > –update-parallelism 指定最大同步更新的任务数
> >
> > –update-delay 指定更新间隔

### stack创建服务集群及监控

```shell
Usage:    docker stack [OPTIONS] COMMAND
#Docker stack任务管理
Manage Docker stacks
#选项
Options:
      --orchestrator string   Orchestrator to use (swarm|kubernetes|all)|选择协调器是swarm还是k8s或者2者
#子命令
Commands:
  docker stack deploy                  ##加载一个新的stack任务或者更新某一stack任务
  docker stack  ls                     ##显示所有stack任务的列表
 docker  stack   ps                    ##列出某一个stack的详细任务
  docker stack   rm                    ##删除一个或者多个stack任务
  docker stack  services               ##列出某一个stack的所有服务
```

docker stack deploy -c docker-compose.yml mycluster	#运行脚本创建集群，并添加监控

> 访问：172.0.0.1：8080 #可视化页面服务

docker stack ps mycluster

>  docker compose -f /user/local/compose.yaml --dry-run
>
> > 构建： docker compose --dry-run up --build -d
> >
> > 启动：docker compose up -d

#### 集群运行：docker-compose.yml

```yaml
version: "3.7"
services:
  web:
    image: library/nginx
    ports:
      - "80:80"
    volumes:
      - web-data:/usr/share/nginx/html
    networks:
      - vm_net
   # environment: # 环境变量
   # 	- APP_HOME: /user/local/service
    privileged: true			#设置容器的权限为root
    deploy:
      replicas: 6									#副本扩容为6
  visualizer:											#添加监控visualizer，docker官方模板
    image: dockersamples/visualizer
    ports:
      - "8080:8080"               #监听的端口为8080
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
volumes:
  web-data:
networks:
  vm_net:
```

