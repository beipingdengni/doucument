# docker



## 容器界面化Portainer

```shell
运行Portainer

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

