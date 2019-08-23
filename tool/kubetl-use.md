### kubetl

------

安装：

```
mac：
brew install kubernetes-cli
liunx：
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

查看版本、测试安装是否成功：kubectl version
=============================================================================
案例：
配置kubeconfig
如下配置文件：创建文件：mkdir -p $HOME/.kube  迁移文件位置： mv -f config $HOME/.kube/config
文件config配置如下内容：
apiVersion: v1
clusters:
- cluster:
    server: https://10.10.20.86:5443
    certificate-authority-data: Q2VydGlmaWNhdGU6CiAgICBEYXRhOgogI。。。。
  name: internalCluster
contexts:
- context:
    cluster: internalCluster
    user: "logman"
  name: internal
current-context: internal
kind: Config
preferences: {}
users:
- name: "logman"
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9。。。。。。
=============================================================================
设置完成后、命令查看 kubernetes 集群信息
kubectl cluster-info
查看集群配置内容
kubectl config view --flatten

```

