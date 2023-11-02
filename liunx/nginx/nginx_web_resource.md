

DNS增加

```
resolver 114.114.114.114 8.8.8.8 valid=30s; 
```



# 前端资源配置、跨域

```sh
server {    
	listen 80;    
	server_name 127.0.0.1 .*beipingdengni.com test-demo.*; 
  
  # 前端资源
	location / {        
			root /data/htdocs/FactoryModel/micro-front-end/industrial-internet-platform-main-vue;        
			index index.php index.html index.htm;        
			# add_header Cache-Control;        
			add_header Access-Control-Allow-Origin *;        
			if ( $request_uri ~* ^.+.(js|css|jpg|png|gif|tif|dpg|jpeg|eot|svg|ttf|woff|json|mp4|rmvb|rm|wmv|avi|3gp)$ ){ 
    		add_header Cache-Control max-age=7776000;          
    		add_header Access-Control-Allow-Origin *;        
    	}        
    	try_files $uri $uri/ /index.html;    
    }
    
    # 跨域
    location /king-boot {
        if ($request_method = 'OPTIONS') { #处理预检请求
            add_header 'Access-Control-Allow-Origin' '*'; #此处理客户端预检请求->nginx服务器跨域问题
           #此允许客户端请求携带header自定义参数，其它固定的header也需要配置上
            add_header 'Access-Control-Allow-Headers' 'my-header,DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range'; 
            add_header 'Access-Control-Max-Age' 1728000; #用来指定本次预检请求的有效期，单位为秒
            add_header 'Content-Type' 'text/plain charset=UTF-8';
            add_header 'Content-Length' 0;
            return 204;
        }
        if ($request_method != 'OPTIONS') { #正常请求
 　　　　　　　add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
            #add_header 'Access-Control-Allow-Origin' '*'; #此处根据服务端api是否配置跨域决定是否配置，不能重复配置
　　　　　　　 add_header 'Access-Control-Expose-Headers' '*'; #此处设置客户端可以获取到的 服务端自定义header名称
        } 
        # proxy_set_header area-code 'CWHT'; #设置固定自定义header参数，不由客户端传递
        proxy_pass https://localhost:9090/;
    }
}
```

# Nginx rewrite 和 proxy_pass共用

### 地址栏会发生变化

```
server {
		listen       19001;
		server_name  localhost;
		charset UTF-8;
		client_max_body_size 1000m;
		location /tmp/img {
			rewrite ^/tmp/img/(.*)$ http://100.*.*.*:9001/$1;
		}
	}

```

### 地址栏不会发生变化

```
server {
		listen       19001;
		server_name  localhost;
		charset UTF-8;
		client_max_body_size 1000m;
		location /tmp/img {
			# /$1的意思是, 先将 /tmp/img 删除, 只保留其后面的路径
			rewrite ^/tmp/img/(.*)$ /$1 break;
			# 改写完之后, 再进行代理; 最终结果: http://100.*.*.*:9001/$1 
			proxy_pass  http://100.*.*.*:9001;
		}
	}

```

## 跨域处理

```sh
# proxy_pass末尾有/,请求地址：http://localhost/api/test,转发地址：http://127.0.0.1:8000/test
upstream apiserver {
	server 127.0.0.1:8090 weight=10 max_fails=2 fail_timeout=30s;
	server 127.0.0.1:8091 weight=10 max_fails=2 fail_timeout=30s;
}

location /api/ {
  add_header Access-Control-Allow-Origin $http_origin; # 指出允许的跨域
  add_header Access-Control-Allow-Credentials 'true'; # 很重要
  add_header Access-Control-Allow-Methods 'GET, POST, OPTIONS';
  add_header Access-Control-Allow-Headers 'DNT,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';
  add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';
  
  if ($request_method = 'OPTIONS') { 
    add_header Access-Control-Max-Age 1728000;
    add_header Content-Type 'text/plain; charset=utf-8';
    add_header Content-Length 0;
    return 200;
	}
  proxy_set_header Host $host;
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header X-Forwarded-Proto $scheme;
  proxy_connect_timeout 5;
  
  proxy_pass http://apiserver/; 
}
```

