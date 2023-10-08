## Nginx 配置

```
mac安装：brew install nginx
	Docroot默认为/usr/local/var/www
	配置文件：/usr/local/etc/nginx/nginx.conf

基本操作
使用 nginx -t 命令对文件对配置文件进行校验
配置完成如下：重启 nginx -s reload
加上 -c /etc/nginx/nginx.conf 指定配置文件路径
nginx【启动】
nginx -s stop 
nginx -s quit

http 级别下配置：
sendfile on：配置on让sendfile发挥作用，将文件的回写过程交给数据缓冲去去完成，而不是放在应用中完成，这样的话在性能提升有有好处
tc_nopush on：让nginx在一个数据包中发送所有的头文件，而不是一个一个单独发
tcp_nodelay on：让nginx不要缓存数据，而是一段一段发送，如果数据的传输有实时性的要求的话可以配置它，发送完一小段数据就立刻能得到返回值，但是不要滥用哦

keepalive_timeout 10：给客户端分配连接超时时间，服务器会在这个时间过后关闭连接。一般设置时间较短，可以让nginx工作持续性更好
client_header_timeout 10：设置请求头的超时时间
client_body_timeout 10:设置请求体的超时时间
send_timeout 10：指定客户端响应超时时间，如果客户端两次操作间隔超过这个时间，服务器就会关闭这个链接

limit_conn_zone $binary_remote_addr zone=addr:5m ：设置用于保存各种key的共享内存的参数，
limit_conn addr 100: 给定的key设置最大连接数

server_tokens：虽然不会让nginx执行速度更快，但是可以在错误页面关闭nginx版本提示，对于网站安全性的提升有好处哦
include /etc/nginx/mime.types：指定在当前文件中包含另一个文件的指令
default_type application/octet-stream：指定默认处理的文件类型可以是二进制
type_hash_max_size 2048：混淆数据，影响三列冲突率，值越大消耗内存越多，散列key冲突率会降低，检索速度更快；值越小key，占用内存较少，冲突率越高，检索速度变慢
日志配置
access_log logs/access.log：设置存储访问记录的日志
error_log logs/error.log：设置存储记录错误发生的日志


【 upstream 负载均衡】核心配置信息如下
ip_hash：指定请求调度算法，默认是weight权重轮询调度，可以指定
server host:port：分发服务器的列表配置
-- down：表示该主机暂停服务
-- max_fails：表示失败最大次数，超过失败最大次数暂停服务
-- fail_timeout：表示如果请求受理失败，暂停指定的时间之后重新发起请求
```

```
【参考的基本配置】

#user  nobody; 【进程运行用户以及用户组，默认nobody账号运行】
worker_processes  1;【nginx要开启的子进程数量、通常数量是CPU内核数量的整数倍】
#error_log  logs/error.log;
#error_log  logs/error.log  error; 【错误日志文件的位置及输出级别（debug/info/notice/warn/ error/crit）】
#pid        logs/nginx.pid;  【进程id的存储文件的位置】
worker_rlimit_nofile 1024; 【指定一个进程可以打开最多文件数量的描述】

events {
    worker_connections  1024; 【最大可以同时接收的连接数量】
    #multi_accept on 【配置指定nginx在收到一个新连接通知后尽可能多的接受更多的连接】
    #use epoll 【配置指定了线程轮询的方法】
}

upstream name {
    ip_hash;
    server 192.168.1.100:8000;
    server 192.168.1.100:8001 down;
    server 192.168.1.100:8002 max_fails=3;
    server 192.168.1.100:8003 fail_timeout=20s;
    server 192.168.1.100:8004 max_fails=3 fail_timeout=20s;
}
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    
    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;
    #gzip  on;

    server {
　　　　 #侦听8080端口
        listen       8080;
　　　　 #定义使用 localhost访问
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
　　　　　　　#定义服务器的默认网站根目录位置
            root   html;
　　　　　　  #定义首页索引文件的名称
            index  index.html index.htm;
        }
        location / {
             proxy_pass https://internal_docker_registry; 【代理的URl域名】
             proxy_read_timeout  90;

             proxy_http_version 1.1;
             proxy_set_header Upgrade $http_upgrade;
             proxy_set_header Connection $http_connection;
             proxy_set_header Host $host;
             proxy_set_header X-Real-IP $remote_addr;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
             proxy_cache_bypass $http_upgrade;
         }
		}
	 include servers/*; 【加载其他配置文件、虚拟主机配置】
}
```

