### 工程中使用的工具

###### gradle

###### maven

发布到Sonatype（Maven Central Repository）服务器上：https://www.jianshu.com/p/1bd36edab4ee



##### docker 创建nexus3

```
nexux3 docker 参考：  https://hub.docker.com/r/sonatype/nexus3

$ mkdir -p /Users/mac/Documents/data/nexus-data && chown -R 200 /Users/mac/Documents/data/nexus-data
$ docker run -d -p 8081:8081 --privileged=true --name nexus3 -v /Users/mac/Documents/data/nexus-data:/nexus-data sonatype/nexus3
```

docker ui 管理

```
docker 注册中心管理：
	https://github.com/kwk/docker-registry-frontend
	https://github.com/jc21/docker-registry-ui
	
dockerui 有 portainerUI、Shipyard、lazydocker
k8s监控：Prometheus
时间数据库：influx db

这些都可以结合：grafa
```

