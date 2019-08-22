##### K8S 处理

------


阿里云docker：https://cr.console.aliyun.com/cn-hangzhou/instances/repositories

中文概念文档：https://www.kubernetes.org.cn/k8s

官方文档：https://kubernetes.io/docs/setup/

mac安装minikube

```
阿里云：
liunx: curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.1.1/minikube-linux-amd64 && chmod +x minikube

mac: curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.1.1/minikube-darwin-amd64 && chmod +x minikube

or
官方
1、brew cask install minikube

2、curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

启动处理步骤
【配置自己的中心】registry:
    https://registry.docker-cn.com
    https://xejq87d0.mirror.aliyuncs.com 【自己中心】
安全做法：  --registry-mirror=#url   【https】 
非安全做法：--insecure-registry=#url 【http】

【启动下载minikube/iso】:
    https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.1.1.iso
    https://github.com/coreos/minikube-iso/releases
    https://raw.githubusercontent.com/cilium/minikube-iso/master/minikube.iso
--iso-url= #iso-url

【启动单例，内存控制】
内存控制：--disk-size 20g 【默认 20g】
内存控制： --memory 2g 【默认 2g】
虚拟机控制： --vm-driver=virtualbox 【默认 virtualbox】

【启动命令、创建并启动 minikube 虚拟机】
# 参考实例： minikube start --registry-mirror=https://xejq87d0.mirror.aliyuncs.com
minikube start --disk-size 2g --vm-driver=virtualbox --iso-url=#url --registry-mirror=#url

【启动存在镜像拉去失败，采用如下方式】阿里云中心镜像 转 google镜像
docker pull registry.cn-hangzhou.aliyuncs.com/google-containers/kube-addon-manager-amd64:v6.3
docker tag registry.cn-hangzhou.aliyuncs.com/google-containers/kube-addon-manager-amd64:v6.3 gcr.io/google-containers/kube-addon-manager:v6.3

kubectl get namespaces 【查看空间】
==========================================================================
kubectl get pods -n 空间名称  【查看pod】
kubectl describe pod 【podid】 -n 空间名称  【查看文件描述】
==========================================================================
kubectl get deployment -n 空间名称  【查看deployment】
kubectl describe deployment 【deployment名称】 -n 空间名称   【查看文件描述】
kubectl delete deployment 【deployment名称】 -n 空间名称 	【删除deployment】
==========================================================================
查看pod日志
kubectl logs --tail=5 -f 【podid】 -n vipcitest
查看pod中某个container的日志
kubectl logs --tail=5 -f 【podid】 -c 【容器名称】 -n vipcitest
==========================================================================
获取service
kubectl get svc -n vipcitest
删除service
kubectl delete svc  【service名字】 -n 空间名称



```



git 地址：https://github.com/kubernetes/minikube

官方网站：https://kubernetes.io/docs/setup/

阿里云参考： https://yq.aliyun.com/articles/221687



1、创建并启动 minikube 虚拟机

minikube start

2、创建 hello-minikube 部署

kubectl run hello-minikube --image=tomcat:8.0 --port=8080

3、发布服务 hello-minikube

kubectl expose deployment hello-minikube --type=NodePort

4、查看 pods

kubectl get pods

5、获取服务地址

minikube service hello-minikube —url

6、停止 minikube 虚拟机

minikube stop

参考：https://cloud.tencent.com/developer/article/1010554

​        ：https://cloud.tencent.com/developer/article/1010584



minikube docker-env  查看：

连接到本地 docker 服务

eval $(minikube docker-env) 

设置 minikube 虚拟机的 docker 环境变量即可



minikube dashboard  查看ui 启动页面，也可使用  openshift 机构



kubectl describe --namespace=kube-system po kube-addon-manager-minikube

gcr.io/google-containers/kube-addon-manager:v6.4-beta.2

reg.docker.lc/share/kubernetes-dashboard-amd64:v1.5.1



[http://chuansong.me/n/1748751651224](http://chuansong.me/n/1748751651224)



设置 namespace

kubectl config set-context  kube-system --namespace=kube-system

kubectl config use-context kube-system



当前Kubernetes支持GCE、vShpere、CoreOS、**OpenShift**、Azure等平台，除此之外，也可以直接运行在物理机上

pod

Pod是Kubernetes的基本操作单元，把相关的一个或多个容器构成一个Pod，通常Pod里的容器运行相同的应用。Pod包含的容器运行在同一个Minion(Host)上，看作一个统一管理单元，共享相同的volumes和network namespace/IP和Port空间。

service

Services也是Kubernetes的基本操作单元，是真实应用服务的抽象，每一个服务后面都有很多对应的容器来支持，通过Proxy的port和服务selector决定服务请求传递给后端提供服务的容器，对外表现为一个单一访问接口，外部不需要了解后端如何运行，这给扩展或维护后端带来很大的好处。

replication controller

Replication Controller确保任何时候Kubernetes集群中有指定数量的pod

副本(replicas)

在运行， 如果少于指定数量的pod副本(replicas)，Replication Controller会启动新的Container，反之会杀死多余的以保证数量不变。Replication Controller使用预先定义的pod模板创建pods，一旦创建成功，pod 模板和创建的pods没有任何关联，可以修改pod 模板而不会对已创建pods有任何影响，也可以直接更新通过Replication Controller创建的pods。对于利用pod 模板创建的pods，Replication Controller根据label selector来关联，通过修改pods的label可以删除对应的pods。

labels

Labels是用于区分Pod、Service、Replication Controller的key/value键值对，Pod、Service、 Replication Controller可以有多个label，但是每个label的key只能对应一个value。Labels是Service和Replication Controller运行的基础，为了将访问Service的请求转发给后端提供服务的多个容器，正是通过标识容器的labels来选择正确的容器。同样，Replication Controller也使用labels来管理通过pod 模板创建的一组容器，这样Replication Controller可以更加容易，方便地管理多个容器，无论有多少容器。

proxy

Proxy是为了解决外部网络能够访问跨机器集群中容器提供的应用服务而设计的，从上图3-3可知Proxy服务也运行在每个Minion上。Proxy提供TCP/UDP sockets的proxy，每创建一种Service，Proxy主要从etcd获取Services和Endpoints的配置信息，或者也可以从file获取，然后根据配置信息在Minion上启动一个Proxy的进程并监听相应的服务端口，当外部请求发生时，Proxy会根据Load Balancer将请求分发到后端正确的容器处理。





阿里云本来练习教材：https://yq.aliyun.com/articles/221687

# [kubernetes 集群的安装部署](http://www.cnblogs.com/galengao/p/5780938.html) 

简单的介绍：https://www.cnblogs.com/galengao/p/5780938.html 【参考-粗】

​                    [http://blog.csdn.net/chenleiking/article/details/78076588](http://blog.csdn.net/chenleiking/article/details/78076588)  【参考-细致】

gitbooks: 中文文档  https://feisky.gitbooks.io/kubernetes/introduction/



官方网站：https://kubernetes.io/docs/home/

githup地址：https://github.com/kubernetes/kubernetes

基本课程教材： https://classroom.udacity.com/courses/ud615

用户名：1271727449@qq.com

密码：懂的？？？



pea  :https://lzw.me/a/pwa-service-worker.html 



minikube version

minikube start

kubectl version

kubectl cluster-info

kubectl get nodes



kubectl run kubernetes-bootcamp --image=docker.io/jocatalin/kubernetes-bootcamp:v1 --port=8080

kubectl get  deployments

kubectl proxy

kubectl get pods

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}’)

curl http://localhost:8001/api/v1/proxy/namespaces/default/pods/$POD_NAME/



kubectl describe pods

**kubectl config view**

kubectl logs

$POD_NAME

传递给docker，让其执行

env

命令

kubectl exec $POD_NAME env

进入到pod的docker中

kubectl exec -it $POD_NAME bash



start service

kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080

kubectl get services

kubectl describe services/kubernetes-bootcamp

export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}’)curl host01:$NODE_PORT

**using label**

kubectl describe deployment

kubectl get pods -l run=kubernetes-bootcamp

kubectl get services -l run=kubernetes-bootcamp

kubectl label pod $POD_NAME app=v1

kubectl describe pods $POD_NAME

kubectl get pods -l app=v1

**delete service**

kubectl delete service -l run=kubernetes-bootcamp

kubectl delete deployment hello-node

kubectl get servicescurl host01:$NODE_PORT

kubectl exec -it $POD_NAME curl localhost:8080

**scale a deployment**

kubectl scale deployments/kubernetes-bootcamp --replicas=4

kubectl get deployments

kubectl get pods -o wide

**load balancing**

【外部访问的前提一定是有service，service相比于deployment的区别在于开放了端口号码】

kubectl describe services/kubernetes-bootcamp

export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}’)

echo NODE_PORT=$NODE_PORT

curl host01:$NODE_PORT

**scale down**

kubectl scale deployments/kubernetes-bootcamp --replicas=2

kubectl get deployments

kubectl get pods -o wide

**update the version of the app**

kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2

**verify an update**

kubectl describe services/kubernetes-bootcamp

kubectl rollout status deployments/kubernetes-bootcamp

kubectl describe pods

**roolback update**

kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v10【仓库中没有这个镜像，进行回退】

kubectl rollout undo deployments/kubernetes-bootcamp

**master运行三个组件：**

- **apiserver**：作为kubernetes系统的入口，封装了核心对象的增删改查操作，以RESTFul接口方式提供给外部客户和内部组件调用。它维护的REST对象将持久化到etcd（一个分布式强一致性的key/value存储）。
- **scheduler**：负责集群的资源调度，为新建的pod分配机器。这部分工作分出来变成一个组件，意味着可以很方便地替换成其他的调度器。
- **controller-manager**：负责执行各种控制器，目前有两类：

- endpoint-controller：定期关联service和pod(关联信息由endpoint对象维护)，保证service到pod的映射总是最新的。
- replication-controller：定期关联replicationController和pod，保证replicationController定义的复制数量与实际运行pod的数量总是一致的。



**slave(称作minion)运行两个组件：**

- **kubelet**：负责管控docker容器，如启动/停止、监控运行状态等。它会定期从etcd获取分配到本机的pod，并根据pod信息启动或停止相应的容器。同时，它也会接收apiserver的HTTP请求，汇报pod的运行状态。
- **proxy**：负责为pod提供代理。它会定期从etcd获取所有的service，并根据service信息创建代理。当某个客户pod要访问其他pod时，访问请求会经过本机proxy做转发。