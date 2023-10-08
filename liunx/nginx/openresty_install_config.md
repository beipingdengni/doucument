# Openresty

## 安装操作

### 安装源

```shell
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://openresty.org/package/amazon/openresty.repo #亚马逊

# 阿里云 # 新增仓库
wget https://openresty.org/package/alinux/openresty.repo
sudo mv openresty.repo /etc/yum.repos.d/

# 更新索引
sudo yum check-update
```

### 执行安装

```sh
sudo yum install -y openresty
# sudo apt-get -y install openresty
```

### 查看依赖

```sh
sudo yum --disablerepo="*" --enablerepo="openresty" list available
```

### 依赖安装

```sh
# Debian 和 Ubuntu
apt-get install libreadline-dev libncurses5-dev libpcre3-dev libssl-dev perl make build-essential
# Fedora 和 RedHat 用户
yum install pcre-devel openssl-devel gcc curl
```

### 自定义安装

```shell
# 默认, --prefix=/usr/local/openresty 程序会被安装到/usr/local/openresty目录
# 自定义配置
./configure --prefix=/opt/openresty \
            --with-luajit \
            --without-http_redis2_module \
            --with-http_iconv_module \
            --with-http_postgres_module

# j2 使用2核编译
make -j2  && make install
```

## 基本配置

### 路径下使用lua

#### content_by_lua_block

```perl
location / {
  default_type text/html;
  content_by_lua_block {  # content_by_lua 'ngx.say("hello world")'
      local redisModule=require "resty.redis";
      local redis=redisModule:new();
      redis:set_timeout("1000");
      
      ngx.say("=====begin connect redis server");
      local ok,err=redis:connect("127.0.0.1",6379);
      if not ok then
        ngx.say("===== connect redis failed");
        return;
      end
      
      ngx.say("begin set key value");
      ok,err=redis:set("hello","zhang");
      if not ok then
      	ngx.say("set key value fail");
      	return;
      end 
      
      ngx.say(" set key value success ");
      redis:close();
  }
}
```

#### content_by_lua_file

```
content_by_lua_file
```

### 模块引入

> lua模块路径，多个之间”;”分隔，其中”;;”表示默认搜索路径，默认到/usr/local/openrest/nginx下找

```
lua_package_path “/usr/local/openrest/lualib/?.lua;;”;  #lua 模块
lua_package_cpath “/usr/local/openrest/lualib/?.so;;”;  #c模块
```

