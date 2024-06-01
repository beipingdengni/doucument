# Python

```python
# 创建虚拟环境
python3 -m venv .

# 进入虚拟环境
source bin/activate

python -m pip install -r requirements.txt
```

### 安装依赖

> pip3 install -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com -r requirements.txt

# pip、conda 包管理

## 临时换源：临时换源只需要在[pip安装](https://so.csdn.net/so/search?q=pip安装&spm=1001.2101.3001.7020)包时，加上一个-i参数后接源的url即可

```shell
#清华源
pip3 install markdown -i https://pypi.tuna.tsinghua.edu.cn/simple

# 非https
pip3 install pandas -i http://nexus.localhost.com/repository/pip-public/simple  --trusted-host nexus.localhost.com
```

## 永久换源

```shell
清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

换回默认源
pip config unset global.index-url 

国内pip源
阿里云：https://mirrors.aliyun.com/pypi/simple/
清华：https://pypi.tuna.tsinghua.edu.cn/simple
中国科技大学 https://pypi.mirrors.ustc.edu.cn/simple/
华中理工大学：http://pypi.hustunique.com/
山东理工大学：http://pypi.sdutlinux.org/
豆瓣：http://pypi.douban.com/simple/
```

### Mac

1. 创建一个文件 ~/.pip/pip.conf
2. 在pip.conf中添加如下

```ini
[global]
timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple/
trusted-host = mirrors.aliyun.com
```

### windows

pip/pip.ini

```ini
[global]
timeout = 6000
index-url = https://mirrors.aliyun.com/pypi/simple/
trusted-host = mirrors.aliyun.com
```

