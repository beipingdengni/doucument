

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
        proxy_pass https://wxapp.jktv.tv/;
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