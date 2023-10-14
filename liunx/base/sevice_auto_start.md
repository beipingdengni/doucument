# 自动启动服务

### 服务自启动相关命令

#### 查看开机自启服务列表

`systemctl list-unit-files|grep enabled `

#### 查看指定服务是否开机自启

`systemctl is-enabled 服务名 `

#### 开启/停止服务开机自启

开启开机自启   `systemctl enable 服务名`

停止开机自启   `systemctl disable 服务名`

## 使用案例

### Tomcat设置开机自启动

#### 添加开机启动文件

sudo vi /etc/systemd/system/xxx.service 

```sh
[Unit]
# 服务名称
Description=xxx
# 前置服务
After=network.target

[Service]
Type=forking

# JDK路径
Environment="JAVA_HOME=/usr/lib/jvm/default-java"
Environment="JAVA_OPTS=-Djava.security.egd=file:///dev/urandom -Djava.awt.headless=true"
# Tomcat路径
Environment="CATALINA_BASE=/opt/tomcat"
# Tomcat路径
Environment="CATALINA_HOME=/opt/tomcat"
# Tomcat路径/temp/tomcat.pid
Environment="CATALINA_PID=/opt/tomcat/temp/tomcat.pid"
Environment="CATALINA_OPTS=-Xms512M -Xmx1024M -server -XX:+UseParallelGC"
# Tomcat路径启动脚本
ExecStart=/opt/tomcat/bin/startup.sh
# Tomcat路径停止脚本
ExecStop=/opt/tomcat/bin/shutdown.sh

[Install]
# 该服务后安装
WantedBy=multi-user.target
```

#### 设置开机自启动

```sh
sudo systemctl daemon-reload
sudo systemctl start xxx
sudo systemctl enable xxx
```

