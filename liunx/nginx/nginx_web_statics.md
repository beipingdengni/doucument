

## log format 配置

```
log_format main                 '$remote_addr - $remote_user [$time_local] '
                                                        '"$request" $status $bytes_sent '
                                                        '"$http_referer" "$http_user_agent" '
                                                        '"$gzip_ratio"';
```

## 请求浏览器

```
$http_user_agent
```

## nginx 配置前端

```shell
upstream minio-server {
  server 10.28.1.30:9000 weight=10 max_fails=2 fail_timeout=30s;
  server 10.28.1.31:9000 weight=10 max_fails=2 fail_timeout=30s;
}

server {
  listen 80;
  server_name kefu-ikbs-test.*;
 
   location = / {
    if ($request_method !~* GET|POST) {
        return 403;
     }
    proxy_pass http://minio-server/zhinengkefu/yypt/yx-ikbs/index.html;
  }
 
  location ^~ /static/ {
    proxy_pass http://minio-server/zhinengkefu/yypt/yx-ikbs/;
  }

  location ^~ /api/ {
    proxy_pass http://127.0.0.1:8080;
  }

  location ^~ /minio-server/ {
    proxy_pass http://minio-server/;
  }

  location / {
    try_files $uri /;
  }

  access_log  /export/servers/nginx/logs/access.log  main;
  error_log  /export/servers/nginx/logs/error.log debug;
}
```

