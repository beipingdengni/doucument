##### K8S 处理

------

##### 文档

阿里云docker镜像：https://cr.console.aliyun.com/cn-hangzhou/instances/repositories

中文概念文档：https://www.kubernetes.org.cn/k8s

官方文档：https://kubernetes.io/docs/setup/

------

阿里云参考： https://yq.aliyun.com/articles/221687

git 地址：https://github.com/kubernetes/minikube

------

##### mac安装minikube搭建K8S实现

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
==========================================================================
【启动下载minikube/iso】:
    https://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/iso/minikube-v1.1.1.iso
    https://github.com/coreos/minikube-iso/releases
    https://raw.githubusercontent.com/cilium/minikube-iso/master/minikube.iso
--iso-url= #iso-url

==========================================================================
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
==========================================================================

查看minikube的状态
minikube status
删除一个集群
minikube delete
启动UI界面
minikube dashboard
进入minikube实例机器环境：参考docker-machine ls 使用方式
eval $(minikube docker-env) 
==========================================================================

kubectl get namespaces 【查看空间】
==========================================================================

kubectl get pods -n 空间名称  【查看pod】
kubectl describe pod 【podid】 -n 空间名称  【查看文件描述】
==========================================================================

kubectl get deployment -n 空间名称  【查看deployment】
kubectl describe deployment 【deployment名称】 -n 空间名称   【查看文件描述】
kubectl delete deployment 【deployment名称】 -n 空间名称 	【删除deployment】
==========================================================================

获取service
kubectl get svc -n vipcitest
删除service
kubectl delete svc  【service名字】 -n 空间名称
通过curl测试服务是否可访问
curl $(minikube service hello-minikube --url)
==========================================================================

查看pod日志
kubectl logs --tail=5 -f 【podid】 -n vipcitest
查看pod中某个container的日志
kubectl logs --tail=5 -f 【podid】 -c 【容器名称】 -n vipcitest
==========================================================================

查看集群下docker环境：
minikube docker-env
==========================================================================

指定文件启动服务
kubectl create -f ./redis.yaml
```

##### 创建一个docker 服务参考如下文档

参考：https://cloud.tencent.com/developer/article/1010554

​        ：https://cloud.tencent.com/developer/article/1010584





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