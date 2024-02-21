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

```
#清华源
pip3 install markdown -i https://pypi.tuna.tsinghua.edu.cn/simple
阿里源
pip3 install markdown -i https://mirrors.aliyun.com/pypi/simple/
腾讯源
pip3 install markdown -i http://mirrors.cloud.tencent.com/pypi/simple
豆瓣源
pip3 install markdown -i http://pypi.douban.com/simple/
```

## 永久换源

```
清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
阿里源
pip config set global.index-url https://mirrors.aliyun.com/pypi/simple/
腾讯源
pip config set global.index-url http://mirrors.cloud.tencent.com/pypi/simple
豆瓣源
pip config set global.index-url http://pypi.douban.com/simple/
换回默认源
pip config unset global.index-url 
```

