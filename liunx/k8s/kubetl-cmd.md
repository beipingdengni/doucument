# kubectl指令使用

博客文章：

1. [k8s资源清单创建pod、deployment、service](https://blog.csdn.net/a913909626/article/details/119967438)
2. [k8s核心yml--Pod、Deployment、Service](https://cloud.tencent.com/developer/article/1462777)
3. [k8s - pod卷的使用 - pod镜像的升级与回滚 - 探针](https://blog.csdn.net/qq_48391148/article/details/127097131)

# 理论

## Kubectl的Kind类型

在Kubernetes中，`kind` 是用来定义资源对象类型的关键字。以下是一些常见的 `kind`：

- `Pod`：代表一个或多个容器的运行实例。
- `Service`：定义一组Pod的访问方式，通常作为其他应用程序访问Pod的入口。
- `Deployment`：用于定义Pod的部署方式，可以管理Pod的创建、更新和删除。（工作在ReplicaSet之上）
- `StatefulSet`：管理有状态的应用
- `DaemonSet`：可以在每个节点运行特定的pod副本
- `Namespace`：用于将集群内的资源划分为不同的逻辑部分。
- `ConfigMap`：用于存储非敏感的配置数据。
- `Secret`：用于存储敏感的配置数据，如密码、API密钥等。
- `Endpoints`: Endpoints可以把外部的链接到k8s系统中（可以理解为引用外部资源，如将一个外部mysql连接到k8s中）
- `Job`：批处理，只要完成就立即退出，不需要重启或重建
- `CronJob`：批处理，周期性任务控制，不需要持续后台运行

除此之外，还有许多其他类型的 `kind`，每个都用于定义Kubernetes集群中的不同类型的资源对象。

### apiversion版本

**以使用kubectl api-versions 获取版本信息**

- alpha（内测版）

- beta（公测版）
- stable（稳定版）：例如v1

## namespace(空间)

- default：你的service和app默认被创建于此。
- kube-system：kubernetes系统组件使用。
- kube-public：公共资源使用。但实际上现在并不常用。

## Service(网络服务)

Service 的四种类型及使用方式，分别是 ClusterIP、NodePort、LoadBalancer 和 ExternalName。

1. `ClusterIP`：是 Service 的默认类型，它会为 Service 创建一个 Cluster IP，这个 IP 只能在集群内部访问。通过 ClusterIP，用户可以访问该 Service 关联的 Pod。
2. `NodePort`：在每个节点上绑定一个端口，从而将 Service 暴露到集群外部。用户可以通过任意一个节点的 IP 地址和该端口号来访问 Service。
3. `LoadBalancer`：在云厂商提供的负载均衡器上创建一个 VIP，从而将 Service 暴露到集群外部。用户可以通过该 VIP 地址来访问 Service。（对外发布服务）
4. `ExternalName`：可以将 Service 映射到集群外部的一个 DNS 名称上，从而将 Service 暴露到集群外部。（对外发布服务）
5. `lngress`：使用ingress规则控制器（七层）。（对外发布服务）

Service 是由 kube-proxy 组件，加上 iptables 来共同实现的.

> - kube-proxy 通过 iptables 处理 Service 的过程，需要在宿主机上设置相当多的 iptables 规则，如果宿主机有大量的Pod，不断刷新iptables规则，会消耗大量的CPU资源。
> - IPVS模式的service，可以使K8s集群支持更多量级的Pod。
> - IPVS模式下，kube-proxy会在service创建后，在宿主机上添加一个虚拟网卡：kube-ipvs0，并分配service IP。
>   - yum install -y ipvsadm	ipvsadm -l （查看策略）
>   - ipvs基于netfilter的hook功能，但使用哈希表作为底层数据结构并在内核空间中工作
>   - rr：轮询调度 lc：最小连接数 dh：目标哈希 sh：源哈希 sed：最短期望延迟 nq： 不排队调度
> - 将service的工作模式改为lvs(ipvs) >>  iptables -t nat -nL | grep 10.224.2.26
>
> 当pod创建的越来越多时，iptables的刷新频率就会越来越大，刷新策略会对cpu造成一定的压力，我们可以使用lvs来实现pod的负载均衡，直接使用linux内核
>
> - 将模式改为ipvs模式 (删除原来的 kube-system pod，更新kube-proxy pod)
>   - kubectl -n kube-system get cm
>   - kubectl -n kube-system edit cm kube-proxy 
> -  除了手动删除更新，也可以使用命令
>   - kubectl get pod -n kube-system |grep kube-proxy | awk '{system("kubectl delete pod "$1" -n kube-system")}' 

## Pod

### Pod启动阶段（对应 phase）

#### 启动阶段包含以下几个步骤：

- 调度到某台 node 上。kubernetes 根据一定的优先级算法选择一台 node 节点将其作为 Pod 运行的 node
- 拉取镜像
- 挂载存储配置等
- 运行起来。如果有健康检查，会根据检查的结果来设置其状态

#### phase 的可能状态有：

- Pending：表示APIServer创建了Pod资源对象并已经存入了etcd中，但是它并未被调度完成（比如还没有调度到某台node上），或者仍然处于从仓库下载镜像的过程中。

- Running：Pod已经被调度到某节点之上，并且Pod中所有容器都已经被kubelet创建。至少有一个容器正在运行，或者正处于启动或者重启状态（也就是说Running状态下的Pod不一定能被正常访问）。

- Succeeded：有些pod不是长久运行的，比如job、cronjob，一段时间后Pod中的所有容器都被成功终止，并且不会再重启。需要反馈任务执行的结果。

- Failed：Pod中的所有容器都已终止了，并且至少有一个容器是因为失败终止。也就是说，容器以非0状态退出或者被系统终止，比如 command 写的有问题。

- Unknown：表示无法读取 Pod 状态，通常是 kube-controller-manager 无法与 Pod 通信。



## deployment

博客文章

[k8s原理之-Pod控制器--ReplicaSet、Deployment ](https://www.cnblogs.com/Lqdream/p/16993815.html)

### 配置机器node节点标签

> kubectl label nodes node01 kgc=a  --overwrite
>
> > kubectl  label nodes node{01,02} kgc=a --overwrite
>
> kubectl get pods --show-labels -o wide

### pod亲和性与反亲和性

#### 策略比较如下

| 调度策略        | 匹配 | 操作符                                 | 拓扑域支持 | 调度目标                  |
| --------------- | ---- | -------------------------------------- | ---------- | ------------------------- |
| nodeAffinity    | 主机 | In, NotIn, Exists,DoesNotExist, Gt, Lt | 否         | 定主机                    |
| podAffinity     | POD  | In, NotIn, Exists,DoesNotExist         | 是         | POD与指定POD同一拓同扑域  |
| podAnitAffinity | POD  | In, NotIn, Exists,DoesNotExist         | 是         | POD是DoesNotExist一拓扑域 |

#### 键值运算关系

- **In**：label 的值在某个列表中 不在则为pending
- **NotIn**：label 的值不在某个列表中
- **Gt**：label 的值大于某个值
- **Lt**：label 的值小于某个值
- **Exists**：某个 label 存在
- **DoesNotExist**：某个 label 不存在

#### topologyKey

##### 概念

指的是一个 `拓扑域`，是指一个范围的概念，比如一个 Node、一个机柜、一个机房或者是一个地区（如杭州、上海）等，实际上对应的还是 Node 上的标签。这里的 `topologyKey` 对应的是 Node 上的标签的 Key（没有Value），可以看出，其实 `topologyKey` 就是用于筛选 Node 的。通过这种方式，我们就可以将各个 Pod 进行跨集群、跨机房、跨地区的调度了；

##### 默认标签

```properties
kubernetes.io/hostname
failure-domain.beta.kubernetes.io/zone
failure-domain.beta.kubernetes.io/region
beta.kubernetes.io/instance-type
beta.kubernetes.io/os
beta.kubernetes.io/arch
```

##### 注意实现

> 配置是与 `labelSelector` 同一级
>
> 1. 对于亲和性和软反亲和性，不允许空topologyKey；
> 2. 对于硬反亲和性，LimitPodHardAntiAffinityTopology控制器用于限制topologyKey只能是kubernetes.io/hostname；
> 3. 对于软反亲和性，空topologyKey被解读成kubernetes.io/hostname, failure-domain.beta.kubernetes.io/zone and failure-domain.beta.kubernetes.io/region的组合

- **requiredDuringSchedulingIgnoredDuringExecution**：**hard，严格执行**，满足规则调度，否则不调度，在预选阶段执行，所以违反hard约定一定不会调度到
- **preferredDuringSchedulingIgnoredDuringExecution**：**soft，尽力执行**，优先满足规则调度，在优选阶段执行，后缀IgnoredDuringExecution表示如果labels发生改变，使得原本运行的pod不在满足规则，那么这个pod将忽视这个改变，继续运行

##### 使用案例

```yaml
apiVersion: apps/v1beta1 # for versions before 1.6.0 use extensions/v1beta1
kind: Deployment
metadata:
  name: redis-cache
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: store
    spec:
      affinity: # 约束条件
      	nodeAffinity:    # (nodeSelector 升级版：nodeAffinity)
      		# 先满足硬策略，排除有kubernetes.io/hostname=node02标签的节点
      		requiredDuringSchedulingIgnoredDuringExecution: 
      			nodeSelectorTerms:
        		- matchExpressions:
          		- key: kubernetes.io/hostname    #指定node的标签
            		operator: NotIn     #设置Pod安装到kubernetes.io/hostname的标签值不在values列表中的node上
           			values:
            			- node02    #因为是NotIn所以不能在node02上
					 # 再满足软策略，优先选择有kgc=a标签的节点
           preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
                preference:
                  matchExpressions:
                  - key: kgc
                    operator: In
                    values:
                # topologyKey: "kubernetes.io/hostname"
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - store
            topologyKey: "kubernetes.io/hostname"
      containers:
      - name: redis-server
        image: redis:3.2-alpine
```

### 污点和容忍（节点调度）

#### 污点(Taint) 

污点的组成格式：`key=value:effect`   每个污点有一个 key 和 value 作为污点的标签，其中 value 可以为空，effect 描述污点的作用。

当前 taint effect 支持如下三个选项：

- NoSchedule：表示 k8s 将不会将 Pod 调度到具有该污点的 Node 上
- PreferNoSchedule：表示 k8s 将尽量避免将 Pod 调度到具有该污点的 Node 上
- NoExecute：表示 k8s 将不会将 Pod 调度到具有该污点的 Node 上，同时会将 Node 上已经存在的 Pod 驱逐出去

> 设置污点: `kubectl taint node node01 key1=value1:NoSchedule`
>
> 节点说明中，查找 Taints 字段: ` kubectl describe node node-name `
>
> 去除污点: `kubectl taint node node01 key1:NoSchedule-`

#### 容忍(Tolerations)

设置了容忍的 Pod 将可以容忍污点的存在，可以被调度到存在污点的 Node 上

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp01
  labels:
    app: myapp01
spec:
  containers:
  - name: with-node-affinity
    image: soscscs/myapp:v1
  tolerations:
  - key: "check"  # 当不指定 key 值时，表示容忍所有的污点 key
    operator: "Equal"
    value: "mycheck"
    effect: "NoExecute"  # 当不指定 effect 值时，表示容忍所有的污点作用
    tolerationSeconds: 3600
```

#### 使用注意点

```shell
#其它注意事项
#（1）当不指定 key 值时，表示容忍所有的污点 key
#  tolerations:
#  - operator: "Exists"
  
#（2）当不指定 effect 值时，表示容忍所有的污点作用
#  tolerations:
#  - key: "key"
#    operator: "Exists"

#（3）有多个 Master 存在时，防止资源浪费，可以如下设置
kubectl taint node Master-Name node-role.kubernetes.io/master=:PreferNoSchedule

#如果某个 Node 更新升级系统组件，为了防止业务长时间中断，可以先在该 Node 设置 NoExecute 污点，把该 Node 上的 Pod 都驱逐出去
kubectl taint node node01 check=mycheck:NoExecute

#此时如果别的 Node 资源不够用，可临时给 Master 设置 PreferNoSchedule 污点，让 Pod 可在 Master 上临时创建
kubectl taint node master node-role.kubernetes.io/master=:PreferNoSchedule

#待所有 Node 的更新操作都完成后，再去除污点
kubectl taint node node01 check=mycheck:NoExecute-

# 维护操作
#cordon 和 drain
##对节点执行维护操作：
kubectl get nodes -o wide
# =============================》cordon《=============================
#将 Node 标记为不可调度的状态，这样就不会让新创建的 Pod 在此 Node 上运行
kubectl cordon <NODE_NAME>   #该node将会变为SchedulingDisabled状态

# =============================》drain《=============================
#kubectl drain 可以让 Node 节点开始释放所有 pod，并且不接收新的 pod 进程。drain 本意排水，意思是将出问题的 Node 下的 Pod 转移到其它 Node 下运行
kubectl drain <NODE_NAME> --ignore-daemonsets --delete-local-data --force
#--ignore-daemonsets：无视 DaemonSet 管理下的 Pod。
#--delete-local-data：如果有 mount local volume 的 pod，会强制杀掉该 pod。
#--force：强制释放不是控制器管理的 Pod，例如 kube-proxy。
#注：执行 drain 命令，会自动做了两件事情:
#（1）设定此 node 为不可调度状态（cordon)
#（2）evict（驱逐）了 Pod
# =============================》uncordon《=============================
#kubectl uncordon 将 Node 标记为可调度的状态
kubectl uncordon <NODE_NAME>
```

### 滚动更新策略

- spec.strategy: 可选项为Recreate,RollingUpdate。默认是 RollingUpdate，滚动发布
  - Recreate:  `如果.spec.strategy.type==Recreate`，在创建新 Pods 之前，所有现有的 Pods 会被杀死
  - RollingUpdate: `如果.spec.strategy.type==RollingUpdate`时，采取 滚动更新的方式更新 Pods。可以指定 `maxUnavailable` 和 `maxSurge` 来控制滚动更新

- `spec.strategy.rollingUpdate.maxUnavailable` 是可选配置项，用来指定在升级过程中不可用Pod的最大数量（则代表在滚动更新时，我们可以忍受多少个 Pod 无法提供服务）。可以配置绝对值与百分比（5 / 10%）。

  - > 该值设置成30%，启动rolling update后旧的ReplicatSet将会立即缩容到期望的Pod数量的70%。新的Pod ready后，随着新的ReplicaSet的扩容，旧的ReplicaSet会进一步缩容，确保在升级的所有时刻可以用的Pod数量至少是期望Pod数量的70%。
    >
    >  假如replicas=100。他的历程可能如下（100旧 - > 70旧+30新  ->  49旧+21新+30新 -> ......-> 100新 ）

- `spec.strategy.rollingUpdate.maxSurge` 是可选配置项，用来指定可以超过期望的Pod数量的最大个数（可以有多少个额外的 Pod）。可以配置绝对值与百分比（5 / 10%）。

  - > 该值设置成30%，启动rolling update后新的ReplicatSet将会立即扩容，新老Pod的总数不能超过期望的Pod数量的130%。旧的Pod被杀掉后，新的ReplicaSet将继续扩容，旧的ReplicaSet会进一步缩容，确保在升级的所有时刻所有的Pod数量和不会超过期望            Pod数量的130%。他的历程可能如下（100旧 - >100旧+30新  ->  70旧+30新+30新 -> ......->100新 ）

- MaxUnavailable 设置为 0 意味着：“在新 Pod 启动并就绪之前，不要关闭任何旧 Pod”。

- MaxSurge 设置为 100% 的意思是：“立即启动所有新 Pod”，也就是说我们有足够的资源，我们希望尽快完成更新。

- pod总数：在replicas-MaxUnavailable 至 replicas+MaxSurge 之间

```yaml
spec:
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
```



### 探针健康检查机制

#### [k8s探针详解](https://www.cnblogs.com/wangjiayu/p/17973214)

- 存活探针（Liveness Probe）主要作用是检测容器内主进程或服务是否仍然运行正常且响应健康检查
  - 监控状态： 存活探针会定期对容器内的应用进行检查，以判断其是否处于“存活”状态，即应用程序没有崩溃、死锁或其他不可恢复的错误。
  - 自动恢复： 当存活探针检测失败时，kubelet将认为该容器内的主进程已经不再健康或者已停止提供预期的服务。此时，kubelet会根据Pod的重启策略来决定是否应该重新启动这个容器。通过这种方式，存活探针可以帮助实现故障自愈，及时恢复服务的可用性。
  - 避免僵死进程： 如果一个容器由于内部错误而进入不可用状态但并未退出，存活探针能够识别出这种情况，并触发容器重启，从而避免资源被僵死进程占用。
  - 保持服务质量： 通过持续监控和及时重启不健康的容器，存活探针有助于确保整个集群的服务质量，减少因单个容器异常导致的整体服务失效的可能性
- 就绪探针（Readiness Probe）主要作用是检测容器是否已经准备好对外提供服务
  - 状态评估： 就绪探针会定期对容器进行检查，以确定容器内应用程序是否完成了必要的初始化工作，并且能够处理来自外部的请求或流量。
  - 流量路由控制： 当就绪探针成功时，表示该容器内部的应用程序已处于可接受请求的状态，此时kubelet会将该容器标记为“就绪”状态，Service将会将其IP地址添加到后端服务列表中，允许Service开始将网络流量转发至这个Pod。
  - 避免无效请求： 如果就绪探针失败，则意味着容器可能还在启动过程中、正在重启服务、或者由于某种原因暂时无法正常响应请求。在这种情况下，kubelet会将容器从Service的后端池中移除，确保不会向其发送任何用户请求，从而避免了因应用未准备完毕而引起的错误响应和用户体验下降。
  - 平滑过渡： 通过就绪探针，Kubernetes可以实现滚动更新或部署过程中的平滑过渡，新版本的容器在通过就绪探针验证前，不会承担任何实际流量，直到它们完全启动并做好处理请求的准备。
- 启动探针（Startup Probe）（自1.16版本引入） 主要用于检测容器内的应用是否已经成功启动并完成初始化任务
  - 状态评估： 就绪探针会定期对容器进行检查，以确定容器内应用程序是否完成了必要的初始化工作，并且能够处理来自外部的请求或流量。
  - 流量路由控制： 当就绪探针成功时，表示该容器内部的应用程序已处于可接受请求的状态，此时kubelet会将该容器标记为“就绪”状态，Service将会将其IP地址添加到后端服务列表中，允许Service开始将网络流量转发至这个Pod。
  - 避免无效请求： 如果就绪探针失败，则意味着容器可能还在启动过程中、正在重启服务、或者由于某种原因暂时无法正常响应请求。在这种情况下，kubelet会将容器从Service的后端池中移除，确保不会向其发送任何用户请求，从而避免了因应用未准备完毕而引起的错误响应和用户体验下降。
  - 平滑过渡： 通过就绪探针，Kubernetes可以实现滚动更新或部署过程中的平滑过渡，新版本的容器在通过就绪探针验证前，不会承担任何实际流量，直到它们完全启动并做好处理请求的准备。

#### 探针探测周期

1. 启动探针（Startup Probe）：
   - 启动探针仅在容器启动阶段执行，探测成功后就不在探测。
   - 在容器启动后等待`initialDelaySeconds`开始探测。
   - 当容器成功通过启动探针检查，即连续成功达到`successThreshold`次数时，kubelet会停止执行启动探针，并开始执行存活探针和就绪探针。
2. 存活探针（Liveness Probe）：
   - 在容器启动并完成启动探针之后开始执行。
   - 在容器启动后等待`initialDelaySeconds`开始执行。如果配置了启动探针（Startup Probe），在启动探针成功后等待`initialDelaySeconds开始探测。`
   - 存活探针在整个容器生命周期内持续进行健康检查，除非被暂时禁用或容器重启。
3. 就绪探针（Readiness Probe）：
   - 与存活探针类似，也是在容器启动并可能完成启动探针之后开始执行。
   - 在容器启动后等待`initialDelaySeconds`开始执行。如果配置了启动探针（Startup Probe），在启动探针成功后等待`initialDelaySeconds开始探测。`
   - 就绪探针在整个容器生命周期内持续进行健康检查，除非被暂时禁用或容器重启。

```yaml
livenessProbe:
  # 类型选择器，可以选择 httpGet、tcpSocket 或 exec 中的一种
  httpGet:         # HTTP GET 请求方式
    path: /health   # 要请求的路径
    port: 8080      # 要请求的端口
    httpHeaders:     # 可选，HTTP 请求头列表
    - name: X-Custom-Header
      value: Awesome
  tcpSocket:       # TCP Socket 检查方式
    port: 8080      # 要连接的端口
  exec:            # 执行命令检查方式
    command:
    - cat
    - /tmp/healthy

  # 基本探测间隔参数：
  initialDelaySeconds: 30  # 容器启动后延迟多少秒开始执行第一次探测，默认为0秒
  periodSeconds: 10        # 探测的时间间隔，即每隔多少秒执行一次，默认为10秒（最小值1秒）
  
  # 控制何时判断容器健康或不健康的阈值参数：
  timeoutSeconds: 1          # 探测超时时间，默认为1秒（最小值1秒）
  successThreshold: 1        # 在连续失败之后需要多少次连续成功才能认为容器是健康的，默认为1
  failureThreshold: 3        # 连续失败多少次才触发相应动作（如重启容器对于存活探针）

readinessProbe: # 就绪探针配置类似
startupProbe:   # 启动探针配置也相似，不过主要用于检测应用是否完成启动过程
```



# 操作

## 输出默认格式(dry-run)

- `--dry-run=client` 会在本地计算机上执行验证，不会向API服务器发送请求，因此速度更快，但可能无法检测到一些服务器端的问题。
- `--dry-run=server` 会将请求发送到Kubernetes集群的API服务器，以便对创建或更新资源对象的请求进行验证，这样可以更全面地模拟实际操作，但可能会比较慢。

### pod、deployment

```shell
# =============================》pod《=============================
# 部署容器服务pod
# kubectl run 名字 --image=镜像 --dry-run=client -o yaml > pod.yaml
kubectl run web-app --image=nginx --dry-run=client -n my-cus-ns -o yaml > pod.yaml
kubectl create -f pod.xml
kubectl get pods # 查看所有的pod
kubectl explain pods # 查看pod的一级参数，可以使用如下命令
kubectl explain pods.spec	# 查看参数下有几个参数
kubectl delete pod web-app # 删除
kubectl describe pod web-app # 查看pod的名称
kubectl exec -it web-app -- bash

# =============================》deployment《=============================
# 创建文件尝试部署
# kubectl create deployment <pod_name> --image=<image_name>
kubectl create deployment web-app --image=ngin --dry-run -n my-cus-ns -o yaml > webp-app.yaml

# =============================》扩容《=============================
# 指定控制器名称，指定扩容pod的数量扩容
kubectl scale web-app --replicas=2 -n my-cus-ns
# 最小10个最大15个副本，当cpu利用率高于80%的时候，会增加副本的数量
kubectl autoscale deployment nginx-deployment --min=10 --max=15 --cpu-percent=80

# =============================》更新镜像《=============================
kubectl apply -f deploy-ui.yaml
kubectl set image deployment/web-app -n my-cus-ns *=web-app:4.2.3.8 --record=true
# 注意：--record=true 起到记录的作用，下面回滚操作查看历史的时候，可以看到change-reason
kubectl patch deployment web-app -n my-cus-ns --patch \'{"spec": {"template": {"spec": {"containers": [{"name": "web-app","image":"web-app:4.2.3.1"}]}}}}\'

# 查询
kubectl get deploy
```

### service

>  网络类型方式小写`nodeport` `clusterip` `loadbalancer` `externalname`
>
> ```shell
> # 尝试运行，输出
> kubectl create service clusterip nginx --tcp=80:80 --dry-run=client -n my-cus-ns -o yaml > my-nginx.yaml
> # 导出配置
> kubectl expose deploy mysql --port=3306 --type=NodePort --dry-run -o yaml > svc.yaml
> 
> # 应用（创建、更新）
> kubectl apply -f nginx-service.yaml 
> # 删除资源
> kubectl delete -f pod-apache.yaml 
> # 查看服务
> kubectl get services 
> # 查看描述
> kubectl describe service nginx 
> ```







## POD操作

进入容器

> kubectl exec -it pod名称 -- /bin/bash