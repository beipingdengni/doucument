# firewall

> firewalld是Red Hat Enterprise Linux 7中用于管理主机级别防火墙的默认方法。firewalld通过firewalld.service systemd服务来启动，可使用低级别的iptables、ip6tables和ebtables 命令来管理 Linux 内核 netfilter 子系统。

## 配置

```
1.使用命令行工具:firewall-cmd
2.使用图形工具:firewall-config
3.使用/etc/firewalld 中的配置文件 注:大部分情况下，不建议直接编辑配置文件，但在使用配置管理工具时，以这种方法复制 配置会很有用
```



## 1、安装

```shell
yum install firewalld firewall-config

# 启动防火墙
systemctl start firewalld
```

## 2、查看

```shell
firewall-cmd --get-active-zones #查看激活
firewall-cmd --zone=public --list-ports #查看开放端口
firewall-cmd --zone=public --list-service #查看开放服务
```

## 3、开放端口

```shell
firewall-cmd --zone=public --add-port=80/tcp --permanent

#更新防火墙规则
firewall-cmd --reload 或 firewall-cmd --complete-reload
#两者的区别就是第一个无需断开连接，就是firewalld特性之一动态添加规则，第二个需要断开连接，类似重启服务
```

## 4、关闭防火墙

```shell
systemctl stop firewalld
```

## 详解

```
1.–get-default-zone  查询当前默认区域
2.–set-default-zone=  设置默认区域，此命令会同时更改运行时配置和永久配置。
3.–get-zones  列出所有可用区域
4.–get-services  列出所有预定义服务
5.–get-active-zones  列出当前正在使用的所有区域(具有关联的接口或源)及其接口和源信息
6.–add-source= [–zone=] 将来自 IP 地址或网络/子网掩码的所有流量路由到指定区域。如果未提供–zone= 选项，则将使用默认区域。
7.–remove-source= [–zone=]  从指定区域中删除来自路由来自 IP 地址或网络/子网掩码的所有流量的规则。如果未提供–zone=选项，则将使用默认区域。
8.–add-interface= [–zone=]  将来自的所有流量路由到指定区域。如果未提供–zone=选项，则将使用默认 区域
9.–change-interface=[–zone=] 将接口与而非当前区域关联。如果未提供–zone 选项，则将使用默认区域
10.–list-all [–zone=] 列出的所有已配置接口、源、服务和端口。如果未提供–zone=选项，则将使用默认区域
11.–list-all-zones  检索所有区域的所有信息(接口、源、端口、服务等等)
12.–add-service= 允许到的流量，如果未提供–zone=选项，则将使用默认区域
13.–add-port=<PORT/PROTOCOL> 允许到<PORT/PROTOCOL>端口的流量。如果未提供–zone=选项，则将使用默认区域
14.–remove-service= 从区域的允许列表中删除。如果未提供–zone=选项，则将使用默认区域。
15.–remove-port=<PORT/PROTOCOL> 从区域的允许列表中删除<PORT/PROTOCOL>端口。如果未提供–zone 选项，则将使用默认区域。
16.–reload
丢弃运行时配置并使用持久配置 例:将默认区域设置为 dmz
firewall-cmd --set-default-zone=dmz
来自 192.168.0.0/24 网络的所有流量都分配给 internal 区域
firewall-cmd --permanent --zone=internal --add-source=192.168.0.0/24 在 internal 区域上打开用于 mysql 的网络端口
firewall-cmd --permanent --zone=internal --add-service=mysql
使永久配置生效
firewall-cmd --reload
```

