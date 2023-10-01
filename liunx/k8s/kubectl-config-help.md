## kubectl-config-help

帮助文档：

```
Modify kubeconfig files using subcommands like "kubectl config set current-context my-context" 

The loading order follows these rules: 

1. If the --kubeconfig flag is set, then only that file is loaded.  The flag may only be set once and no merging takes place.  
2. If $KUBECONFIG environment variable is set, then it is used a list of paths (normal path delimitting rules for your system).  These paths are merged.  When a value is modified, it is modified in the file that defines the stanza.  When a value is created, it is created in the first file that exists.  If no files in the chain exist, then it creates the last file in the list.  
3. Otherwise, ${HOME}/.kube/config is used and no merging takes place.

Available Commands:
  current-context 显示 current_context
  delete-cluster  删除 kubeconfig 文件中指定的集群
  delete-context  删除 kubeconfig 文件中指定的 context
  get-clusters    显示 kubeconfig 文件中定义的集群
  get-contexts    描述一个或多个 contexts
  rename-context  Renames a context from the kubeconfig file.
  set             设置 kubeconfig 文件中的一个单个值
  set-cluster     设置 kubeconfig 文件中的一个集群条目
  set-context     设置 kubeconfig 文件中的一个 context 条目
  set-credentials 设置 kubeconfig 文件中的一个用户条目
  unset           取消设置 kubeconfig 文件中的一个单个值
  use-context     设置 kubeconfig 文件中的当前上下文
  view            显示合并的 kubeconfig 配置或一个指定的 kubeconfig 文件

Usage:
  kubectl config SUBCOMMAND [options]

=============================================================================
参考使用：
kubectl config set-credentials produce-admin --username=admin --password=6666o9oIB2gHD88882quIfLMy6666
kubectl config set-cluster produce-cluster --server=https://cls-66668888.ccs.tencent-cloud.com --certificate-authority=/etc/kubernetes/cluster-ca.crt
kubectl config set-context produce-system --cluster=produce-cluster --user=produce-admin

=============================================================================

kubectl config set-credentials NAME [--username=basic_user] [--password=basic_password]
kubectl config set-cluster NAME [--server=server] [--certificate-authority=path/to/certficate/authority] 
kubectl config set-context NAME [--cluster=cluster_nickname] [--user=user_nickname] [--namespace=namespace]
set-credentials 在kubeconfig配置文件中设置一个用户项。
set-cluster 在kubeconfig配置文件中设置一个集群项。
set-context 在kubeconfig配置文件中设置一个环境项。
```

