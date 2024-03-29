## Nginx 配置中的变量参数

```sh
$args                    #请求中的参数值
$query_string            #同 $args
$arg_NAME                #GET请求中NAME的值
$is_args                 #如果请求中有参数，值为"?"，否则为空字符串
$uri                     #请求中的当前URI(不带请求参数，参数位于$args)，可以不同于浏览器传递的$request_uri的值，它可以通过内部重定向，或者使用index指令进行修改，$uri不包含主机名，如"/foo/bar.html"。
$document_uri            #同 $uri
$document_root           #当前请求的文档根目录或别名
$host                    #优先级：HTTP请求行的主机名>"HOST"请求头字段>符合请求的服务器名.请求中的主机头字段，如果请求中的主机头不可用，则为服务器处理请求的服务器名称
$hostname                #主机名
$https                   #如果开启了SSL安全模式，值为"on"，否则为空字符串。
$binary_remote_addr      #客户端地址的二进制形式，固定长度为4个字节
$body_bytes_sent         #传输给客户端的字节数，响应头不计算在内；这个变量和Apache的mod_log_config模块中的"%B"参数保持兼容
$bytes_sent              #传输给客户端的字节数
$connection              #TCP连接的序列号
$connection_requests     #TCP连接当前的请求数量
$content_length          #"Content-Length" 请求头字段
$content_type            #"Content-Type" 请求头字段
$cookie_name             #cookie名称
$limit_rate              #用于设置响应的速度限制
$msec                    #当前的Unix时间戳
$nginx_version           #nginx版本
$pid                     #工作进程的PID
$pipe                    #如果请求来自管道通信，值为"p"，否则为"."
$proxy_protocol_addr     #获取代理访问服务器的客户端地址，如果是直接访问，该值为空字符串
$realpath_root           #当前请求的文档根目录或别名的真实路径，会将所有符号连接转换为真实路径
$remote_addr             #客户端地址
$remote_port             #客户端端口
$remote_user             #用于HTTP基础认证服务的用户名
$request                 #代表客户端的请求地址
$request_body            #客户端的请求主体：此变量可在location中使用，将请求主体通过proxy_pass，fastcgi_pass，uwsgi_pass和scgi_pass传递给下一级的代理服务器
$request_body_file       #将客户端请求主体保存在临时文件中。文件处理结束后，此文件需删除。如果需要之一开启此功能，需要设置client_body_in_file_only。如果将次文件传 递给后端的代理服务器，需要禁用request body，即设置proxy_pass_request_body off，fastcgi_pass_request_body off，uwsgi_pass_request_body off，or scgi_pass_request_body off
$request_completion      #如果请求成功，值为"OK"，如果请求未完成或者请求不是一个范围请求的最后一部分，则为空
$request_filename        #当前连接请求的文件路径，由root或alias指令与URI请求生成
$request_length          #请求的长度 (包括请求的地址，http请求头和请求主体)
$request_method          #HTTP请求方法，通常为"GET"或"POST"
$request_time            #处理客户端请求使用的时间,单位为秒，精度毫秒； 从读入客户端的第一个字节开始，直到把最后一个字符发送给客户端后进行日志写入为止。
$request_uri             #这个变量等于包含一些客户端请求参数的原始URI，它无法修改，请查看$uri更改或重写URI，不包含主机名，例如："/cnphp/test.php?arg=freemouse"
$scheme                  #请求使用的Web协议，"http" 或 "https"
$server_addr             #服务器端地址，需要注意的是：为了避免访问linux系统内核，应将ip地址提前设置在配置文件中
$server_name             #服务器名
$server_port             #服务器端口
$server_protocol         #服务器的HTTP版本，通常为 "HTTP/1.0" 或 "HTTP/1.1"
$status                  #HTTP响应代码
$time_iso8601            #服务器时间的ISO 8610格式
$time_local              #服务器时间（LOG Format 格式）
$cookie_NAME             #客户端请求Header头中的cookie变量，前缀"$cookie_"加上cookie名称的变量，该变量的值即为cookie名称的值
$http_NAME               #匹配任意请求头字段；变量名中的后半部分NAME可以替换成任意请求头字段，如在配置文件中需要获取http请求头："Accept-Language"，$http_accept_language即可
$http_cookie　　　　　　　 #请求的所有cookie
$http_host               #请求地址，即浏览器中你输入的地址（IP或域名）
$http_referer            #url跳转来源,用来记录从那个页面链接访问过来的
$http_user_agent         #用户终端浏览器等信息
$http_x_forwarded_for    #客户端的IP和代理服务器的IP，以逗号隔开；可伪造
$http_x_forwarded_proto  #请求的协议
$sent_http_NAME          #可以设置任意http响应头字段；变量名中的后半部分NAME可以替换成任意响应头字段，如需要设置响应头Content-length，$sent_http_content_length即可
$sent_http_cache_control
$sent_http_connection
$sent_http_content_type
$sent_http_keep_alive
$sent_http_last_modified
$sent_http_location

```

### lua

```sh
# 使用lua获取$request_body值, server中使用lua_need_request_body on; 或者在location lua代码块中使用 ngx.req.read_body(),注意：1、lua代码块中必须有执行语句，否则lua不执行，无法获取request_body；2、不要使用return 200；等命令，有return命令，lua代码不执行
# 自定义变量存放request body； 1、在server 块中使用set $resp_body ""; 声明变量；2、在location使用  ngx.var.resp_body = ngx.req.get_body_data() or "-"    为变量赋值

worker_processes  1;        #nginx worker 数量
error_log /home/shuhao/openresty-test/logs/error.log debug;   #指定错误日志文件路径
events {
    worker_connections 1024;
}
 
http {
    log_format  dm  ' "$request_body"  --  "$resp_body"';
    lua_need_request_body on;
    server {
        listen 6699;
        set $resp_body "";
        location /post/ {
            lua_need_request_body on;
            content_by_lua '
```



## Nginx.conf

```perl
user                                    admin  admin; # 启动的用户
worker_processes                        8;
#worker_cpu_affinity                     00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;
error_log                               /export/servers/nginx/logs/nginx_error.log  warn;
pid                                     /export/servers/nginx/run/nginx.pid;
worker_rlimit_nofile                    65535;
events 
{
                                        use epoll;
                                        worker_connections 65535;
}

http 
{
        include                         mime.types;
        default_type                    application/octet-stream;
        server_tokens                   off;
        log_format main                 '$remote_addr - $remote_user [$time_local] '
                                                        '"$request" $status $bytes_sent '
                                                        '"$http_referer" "$http_user_agent" '
                                                        '"$gzip_ratio"';
        #charset                        utf-8;
        server_names_hash_bucket_size   128;
        client_header_buffer_size       32k;
        large_client_header_buffers     4 32k;
        client_max_body_size            300m;	#运行上传和传递的数据大小
        sendfile                        on;
        tcp_nopush                      on;
        keepalive_timeout               0;		#不超时，保持
        tcp_nodelay                     on;
        client_body_buffer_size         512k;
        fastcgi_intercept_errors        on;
        proxy_connect_timeout           90;		# 连接服务超时
        proxy_read_timeout              180;	# 读取服务接口超时
        proxy_send_timeout              180;  # 发送到服务接口超时
        proxy_buffer_size               256k;   #代理返回内容，超过会写完临时文件
        proxy_buffers                   4 256k; #代理返回内容
        proxy_busy_buffers_size         256k;   
        #proxy_temp_file_write_size      256k;		#代理返回内容
        proxy_intercept_errors          on;
        server_name_in_redirect         off;
        proxy_hide_header               X-Powered-By;

        gzip                            on;
        gzip_min_length                 100;
        gzip_buffers                    4 16k;
        gzip_http_version               1.0;
        gzip_comp_level                 9;
        gzip_types                      text/plain application/x-javascript text/css application/xml;
        gzip_vary                       on;
        gzip_proxied                    any;
        error_page 400 401 402 403 404 405 408 410 412 413 414 415 500 501 502 503 506 = /error.html;

        include domains/*;
}
```

## Domain server 配置

```perl
upstream open-api {
    server 127.0.0.1:80 weight=10 max_fails=2 fail_timeout=30s;
}

server {
    listen          80;
    server_name     jimi.open.* jimi-open.* kefu.*;
    gzip on;

    location / {
        proxy_next_upstream     http_500 http_502 http_503 http_504 error timeout invalid_header;
        proxy_set_header        Host  $host;
        proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header        Upgrade         $http_upgrade;
        proxy_set_header        Connection      "upgrade";
        proxy_http_version      1.1;
        proxy_pass              http://open-api;
        expires                 0;
    }
}
```

