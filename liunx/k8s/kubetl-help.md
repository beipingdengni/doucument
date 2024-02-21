### kubetl

```shell
kubectl 控制 Kubernetes 集群管理器
# kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/
基础指令（初学者）
Basic Commands (Beginner):
  create          Create a resource from a file or from stdin
  expose          Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run             在集群上运行特定镜像
  set             为对象设置指定特性

基础指令（中间的）
# Basic Commands (Intermediate):
  explain         Get documentation for a resource
  get             显示一个或多个资源
  edit            编辑服务器上的资源
  delete          Delete resources by file names, stdin, resources and names, or by resources and label selector

部署操作
# Deploy Commands:
  rollout         Manage the rollout of a resource
  scale           Set a new size for a deployment, replica set, or replication controller
  autoscale       Auto-scale a deployment, replica set, stateful set, or replication controller

集群管理
# Cluster Management Commands:
  certificate     Modify certificate resources
  cluster-info    Display cluster information
  top             Display resource (CPU/memory) usage
  cordon          标记节点为不可调度
  uncordon        标记节点为可调度
  drain           清空节点以准备维护
  taint           更新一个或者多个节点上的污点

故障排除和调试命令
# Troubleshooting and Debugging Commands:
  describe        显示特定资源或资源组的详细信息
  logs            打印 Pod 中容器的日志
  attach          挂接到一个运行中的容器
  exec            在某个容器中执行一个命令
  port-forward    将一个或多个本地端口转发到某个 Pod
  proxy           运行一个指向 Kubernetes API 服务器的代理
  cp              Copy files and directories to and from containers
  auth            Inspect authorization
  debug           Create debugging sessions for troubleshooting workloads and nodes
  events          List events

高级命令
# Advanced Commands:
  diff            Diff the live version against a would-be applied version
  apply           Apply a configuration to a resource by file name or stdin
  patch           Update fields of a resource
  replace         Replace a resource by file name or stdin
  wait            Experimental: Wait for a specific condition on one or many resources
  kustomize       Build a kustomization target from a directory or URL

设置命令
# Settings Commands:
  label           更新某资源上的标签
  annotate        更新一个资源的注解
  completion      Output shell completion code for the specified shell (bash, zsh, fish, or powershell)

插件提供的子命令
# Subcommands provided by plugins:

Other Commands:
  api-resources   Print the supported API resources on the server
  api-versions    Print the supported API versions on the server, in the form of "group/version"
  config          修改 kubeconfig 文件
  plugin          Provides utilities for interacting with plugins
  version         输出客户端和服务端的版本信息

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).
```

