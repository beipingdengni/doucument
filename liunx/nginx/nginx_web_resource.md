

# 前端资源配置

```
server {    
	listen 80;    
	server_name 127.0.0.1 .*beipingdengni.com test-demo.*;         
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
}
```

