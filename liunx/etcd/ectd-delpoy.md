## **tcd clientTLS 证书认证配置**

## **介绍**

etcd 是一个强一致的分布式键值(key-value)存储，它提供了一种可靠的方式来存储需要由分布式系统或机器集群访问的数据。它可以优雅地处理网络分区期间的领导者选举，即使在
领导者节点中也可以容忍机器故障。etcd 是用Go语言编写的，它具有出色的跨平台支持，小的二进制文件和强大的社区。etcd机器之间的通信通过Raft共识算法处理。

目前Kubernetes等项目都选择etcd作为KV存储服务,etcd项目也是云原生计算基金会[Clound Native Computing Foundation](https://link.zhihu.com/?target=https%3A//www.cncf.io/)下一个项目

## **快速体验**

可以从etcd github仓库的[release](https://link.zhihu.com/?target=https%3A//github.com/etcd-io/etcd/releases)下载编译好的二进制，也自己从源码编译或者用包管理工具等方式安装

直接运行 `etcd` 启动etcd server端
etcd启动之后会监听两个端口

- 2379 端口 监听clent请求
- 2380 端口 etcd作为一个分布式集群软件,2380端口监听集群内其它节点通信

KV 基本操作

```bash
# PUT操作: etcdctl put   key  value
etcdctl put  db_host 127.0.0.1```

# GET操作: etcdctl get  key  
etcdctl get db_host

# watch KEY的变化: etcdctl watch key
etcdctl watch db_host  
# db_host有更新会监听到

# watch KEY前缀 
etcdctl watch db_ --prefix
# 以db_开头的key都可以watch到
```

etcd更多使用这里就不做详细介绍，可以参考etcd官网文档[教程](https://link.zhihu.com/?target=https%3A//etcd.io/docs/v3.5/tutorials/)

## **etcd client TLS认证**

etcd证书分

- client请求证书
- etcd集群模式下peer节点之间通信证书

本文讲client证书生成和使用，下次介绍peer证书。 etcd的client证书和我们浏览网站时https证书有一点不一样，

浏览HTTPS网站时证书是由网站服务器提供，浏览器校验证书合法性。

而etcd client端访问时etcd server，则是client端请求时携带证书，由etcd server校验client证书的合法性

### **证书生成**

话不多说，敲的命令比较多，我们直接用bash脚本来生成

```bash
#!/bin/bash -x
# etcd集群由多个节点注册，这里假设3个节点，
export NAME1=etcd01
export ADDRESS1=192.168.31.17

export NAME2=etcd02
export ADDRESS2=192.168.31.18

export NAME3=etcd03
export ADDRESS3=192.168.31.19

days=3650

cat > openssl.conf << EOF
[req]
req_extensions = v3_req
distinguished_name = req_distinguished_name
[req_distinguished_name]
[ v3_req ]
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names
[alt_names]
DNS.1 = $NAME1
DNS.2 = $NAME2
DNS.3 = $NAME3
IP.1 = 127.0.0.1
IP.2 = $ADDRESS1
IP.3 = $ADDRESS2
IP.4 = $ADDRESS3
EOF


# 准备 CA 证书
[ -f ca.key ] || openssl genrsa -out ca.key 2048
[ -f ca.crt ] || openssl req -x509 -new -nodes -key ca.key -subj "/CN=etcd-ca" -days ${days} -out ca.crt

# 创建 etcd client 证书
[ -f client.key ] || openssl genrsa -out client.key 2048
[ -f client.csr ] || openssl req -new -key client.key -subj "/CN=etcd-client" -out client.csr -config openssl.conf
[ -f client.crt ] || openssl x509 -req -sha256 -in client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out client.crt -days ${days} -extensions v3_req  -extfile openssl.conf

# 创建 etcd 集群 peer 间证书
[ -f peer.key ] || openssl genrsa -out peer.key 2048
[ -f peer.csr ] || openssl req -new -key peer.key -subj "/CN=etcd-peer" -out peer.csr -config openssl.conf
[ -f peer.crt ] || openssl x509 -req -sha256 -in peer.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out peer.crt -days ${days} -extensions v3_req  -extfile openssl.conf
```

运行之后生成证书清单如下

```text
├── ca.crt
├── ca.key 
├── ca.srl 
├── client.crt
├── client.csr
├── client.key
├── openssl.conf
├── peer.crt
├── peer.csr
├── peer.key
└── pki.sh  #生成证书脚本
```

### **etcd启动证书**

**etcd server端配置，写入etcd.yaml，etcd.yaml内容如下**

```yaml
name: etcd01
data-dir: data/etcd/default.etcd
listen-peer-urls: https://192.168.31.17:2380
listen-client-urls: https://192.168.31.17:2379,https://127.0.0.1:2379
initial-advertise-peer-urls: https://192.168.31.17:2380
advertise-client-url: https://192.168.31.17:2379
initial-cluster: etcd01=https://192.168.31.17:2380
initial-cluster-token: etcd-cluster
initial-cluster-state: new
# client节点通信 证书配置
client-transport-security: 
  cert-file:  certs/client.crt
  key-file:   certs/client.key
  client-cert-auth: true
  trusted-ca-file: certs/ca.crt

# 集群peer节点间通信  证书配置
peer-transport-security:
  cert-file: certs/peer.crt
  key-file:  certs/peer.key
  client-cert-auth: false
  trusted-ca-file: certs/ca.crt
  auto-tls: false
```

etcd指定配置文件 启动

```bash
etcd --config-file etc/etcd/etcd.yaml
```

### **etcdctl端访问etcd**

```bash
etcdctl \
 --cacert=certs/ca.crt \
 --cert=certs/client.crt \
 --key=certs/client.key \
 --endpoints=https://192.168.31.17:2379 \
 put Name Jack
```

### **用Go代码访问etcd**

go.mod 添加依赖
`require go.etcd.io/etcd/client/v3 v3.5.0`

导入package

```go
import (
    "context"
    "crypto/tls"
    "crypto/x509"
    "go.etcd.io/etcd/clientv3"
    "io/ioutil"
    "log"
    "os"
    "testing"
    "time"
)
```

定义变量

```go
const (
    dialTimeout    = 5 * time.Second
    requestTimeout = 4 * time.Second
    endpoints      = []string{"192.168.31.17:2379"}
)
```

初始化client

```go
  var homedir = os.Getenv("HOME")
    var etcdCert = homedir + "/certs/client.crt"
    var etcdCertKey = homedir + "/certs/client.key"
    var etcdCa = homedir + "/certs/ca.crt"

  // 加载客户端证书
    cert, err := tls.LoadX509KeyPair(etcdCert, etcdCertKey)
    if err != nil {
        return
    }

    // 加载 CA 证书
    caData, err := ioutil.ReadFile(etcdCa)
    if err != nil {
        return
    }

    pool := x509.NewCertPool()
    pool.AppendCertsFromPEM(caData)

    _tlsConfig := &tls.Config{
        Certificates: []tls.Certificate{cert},
        RootCAs:      pool,
    }

    cfg := clientv3.Config{
        Endpoints: endpoints,
        TLS:       _tlsConfig, // Client.Config设置 TLS
    }

    cli, err := clientv3.New(cfg)

    if err != nil {
        log.Fatal(err)
    }
```

与etcd交互

```go
    key1, value1 := "testkey1", "testvalue"
    ctx, cancel := context.WithTimeout(context.Background(), requestTimeout)
    _, err = cli.Put(ctx, key1, value1)
    cancel()
    if err != nil {
        log.Println("Put failed. ", err)
    } else {
        log.Printf("Put {%s:%s} succeed\n", key1, value1)
    }

    rsp, err := cli.Get(context.Background(), key1)
    if err != nil {
        log.Println("Get failed. ", err)
    } else {
        for i, kv := range rsp.Kvs {
            log.Printf("Get index %d , key=%s, value= %s\n", i, kv.Key, kv.Value)
        }
    }
```