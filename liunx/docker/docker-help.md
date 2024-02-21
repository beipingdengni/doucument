## Docker —help

## 删除容器

```shell
docker rm 容器id                  #删除指定的容器 #不能删除正在运行的容器,如果强制删除 rm -f
docker rm -f $(docker ps -aq)     #删除所有的容器
docker ps -a -qlxargs docker rm   #删除所有容器

# shell脚本（运行完成就删除）
docker run -d centos /bin/sh -c "while true;do echo cpf; sleep 1;done"
# 进入容器
docker exec -it 14c64d9110c8 /bin/bash #进入容器后,开一个新的终端,可以在里面进行操作(常用)
docker attach 14c64d9110c8  #进入容器正在执行的终端,不会启动新的进程

# 复制 docker cp 源文件 目的位置
docker cp bbb090833a4e:home/text.txt /home # 容器下载
docker cp /home/text.txt bbb090833a4e:home/ # 上传容器

#显示日志   
 -tf           #显示全部
 -tail  number #要显示的日志条数
docker logs -tf --tail 10  14c64d9110c8

# 查看镜像的元数据
docker inspect 14c64d9110c8
# go的template语法
docker inspect -f 'Hello from container {{.Name}}' jenkins
#输出结果：Hello from container /jenkins
```

## help

```
Options:
      --config string      Location of client config files (default "/Users/mac/.docker")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/Users/mac/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/Users/mac/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/Users/mac/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  builder     Manage builds
  config      Manage Docker configs
  container   Manage containers
  image       Manage images
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes
```

