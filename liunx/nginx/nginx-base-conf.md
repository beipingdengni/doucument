

## Nginx.conf

```perl
user                                    admin  admin;
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
        client_max_body_size            300m;
        sendfile                        on;
        tcp_nopush                      on;
        keepalive_timeout               0;
        tcp_nodelay                     on;
        client_body_buffer_size         512k;
        fastcgi_intercept_errors        on;
        proxy_connect_timeout           90;
        proxy_read_timeout              180;
        proxy_send_timeout              180;
        proxy_buffer_size               256k;
        proxy_buffers                   4 256k;
        proxy_busy_buffers_size         256k;
        proxy_temp_file_write_size      256k;
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

