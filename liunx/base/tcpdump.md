## tcpdump抓包

### 安装

> yum install -y tcpdump

抓网卡所有包

> tcpdump -nn -i {网卡名称}

### 指定端口

> tcpdump -nn -i {网卡名称} port {port}

### 过滤端口

> tcpdump -nn -i {网卡名称} not port {port}

### 指定ip

> tcpdump -nn -i {网卡名称} host {ip}

### 指定ip过滤指定端口

> tcpdump -nn -i {网卡名称} not port {port} and host {ip}

### 指定抓取数据包的数量

加上-c选项可以指定抓取数据包的数量，例如指定只抓取20个数据包：

> tcpdump -nn -i {网卡名称} -c 20 not port {port} and host {ip}

### 输出到文件

> tcpdump -nn -i {网卡名称} host {ip} -w /home/xxx.cap



