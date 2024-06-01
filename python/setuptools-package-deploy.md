# 打包部署

> 生成requirements.txt文件：pip freeze > requirements.txt
>
> 安装requirements.txt依赖：pip install -r requirements.txt

## 打包不含依赖

```python
# 虚拟环境问题
pip cache purge
# 安装
pip install -r requirements.txt

pip list --format=freeze > requirements.txt
/#使用上述命令导出的文件中，会包含如下几个包：distribute，pip，setuptools，wheel，建议手动删除！
```

## 打包保护依赖

### 下载所有依赖到指定包

```python
# 下载， 指定：packages/
pip download -r requirements.txt -d packages/ -i https://pypi.tuna.tsinghua.edu.cn/simple
# 安装， 指定：./packages
pip install --no-index --find-links=./packages -r ./requirements.txt


# https://pypi.org 官方网站上下载，离线安装
python3 -m pip install ~/PyTweening-1.0.3.zip

```

## setuptools工具使用

### 安装：pip install setuptools

> python3 -m pip install setuptools

根目录下,创建一个setup.py

### 内容说明

1. name : 打包起来的包的文件名
2. version : 版本号,添加为打包文件的后缀名
3. author : 作者
4. author_email : 作者的邮箱
5. py_modules : 打包的.py文件
6. packages: 打包的python文件夹
7. package_data :  项目里会有一些非py文件,比如html和js等,这时候就要靠include_package_data 和 package_data 来指了。package_data:一般写成{‘your_package_name’: [“files”]}, include_package_data还没完,还需要修改MANIFEST.in文件.MANIFEST.in文件的语法为: include xxx/xxx/xxx/.ini/(所有以.ini结尾的文件,也可以直接指定文件名)
8. license : 支持的开源协议
9. description : 对项目简短的一个形容
10. ext_modules : 是一个包含Extension实例的列表,Extension的定义也有一些参数
11. ext_package : 定义extension的相对路径
12. requires : 定义依赖哪些模块
13. provides : 定义可以为哪些模块提供依赖
14. data_files :指定其他的一些文件(如配置文件),规定了哪些文件被安装到哪些目录中。如果目录名是相对路径,则是相对于sys.prefix或sys.exec_prefix的路径。如果没有提供模板,会被添加到MANIFEST文件中

#### MANIFEST.in 文件语法说明

有如下几种语法

- include pat1 pat2 ...：引入所有匹配后面正则表达式的文件
- exclude pat1 pat2 ...：不引入所有匹配后面正则表达式的文件
- recursive-include dir-pattern pat1 pat2 ...：递归引入匹配 dir-pattern 目录下匹配后面正则表达式的文件
- recursive-exclude dir-pattern pat1 pat2 ...：递归不引入匹配 dir-pattern 目录下匹配后面正则表达式的文件
- global-include pat1 pat2 ...：引入源码树中所有匹配后面正则表达式的文件，无论文件在哪里
- global-exclude pat1 pat2 ...：不引入源码树中所有匹配后面正则表达式的文件，无论文件在哪里
- graft dir-pattern：引入匹配 dir-pattern 正则表达式的目录下的所有文件
- prune dir-pattern：不引入匹配 dir-pattern 正则表达式的目录下的所有文件

使用案例

```
include CHANGES.rst
graft docs
prune docs/_build
```



### 使用demo演示

#### setup.py 文件

```python
from setuptools import setup,find_packages

set_up(
   name = 'spider-redis',#包名
   version = '0.0.1',#版本号
   packages = find_packages(), #搜索当前目前下的所有包 ； packages=["demo01","demo02"] 
   # packages=find_packages("src"),  package_dir={"": "src"},
   # 以下两个属性使用
   #include_package_data=True, # 不引入 README.txt 文件
   #exclude_package_data={"": ["README.txt"]},
   # MANIFEST.in 文件位于 setup.py 同级的项目根目录上，内容类似下面
   # 
   package_data = {
        # 任何包中含有.txt文件，都包含它
        '': ['*.txt'],
        # 包含demo包data文件夹中的 *.dat文件
        'demo': ['data/*.dat'],
    },
    entry_points={
        'console_scripts': [
            # app.py 文件
            'app_flask = app:main',
        ],
    }
)
```

#### 打包wheel、安装

>  python setup.py bdist_wheel
>
> pip install XXXXX.whl   # 使用安装

#### 打包xxx-version.tar.gz、安装

> python setup.py sdist  --formats=gztar,zip
>
> 解压缩gz文档后，打开文件夹,执行setup.py,模块将会被安装到解释器对应的Lib/site-packages目录下
>
> python setup.py install

#### 打包egg、安装

>  python setup.py bdist_egg

#### 发布服务（develop）

>  python setup.py develop
>
> 配置文件控制： entry_points ={ 'console_scripts': ['app_flask = app:main']  }



## 创建虚拟环境部署

>创建虚拟环境

python -m virtualenv venv

> 激活虚拟环境

source venv/bin/activate

> 安装项目依赖包

pip3 install -r requirements.txt



## 发布包到 PyPi

若你觉得自己开发的模块非常不错，想要 share 给其他人使用，你可以将其上传到 PyPi （Python Package Index）上，它是 Python 官方维护的第三方包仓库，用于统一存储和管理开发者发布的 Python 包

如果要发布自己的包，需要先到 pypi 上注册账号。然后创建 `~/.pypirc`(windows `~/.pypirc`) 文件，此文件中配置 PyPI 访问地址和账号。.pypirc文件内容请根据自己的账号来修改

### 典型的 .pypirc 文件

```ini
[distutils]
index-servers = pypi
 
[pypi]
username:xxx
password:xxx
```

### 上传

```undefined
setup.py sdist upload
```

想更省劲的话可以考虑安装 `twine`，然后使用 `twine` 上传，这个还可以上传whl文件

```bash
twine upload dist/* # 上传dist下的所有文件
```

pypi 好像不能覆盖文件，要上传新的需要修改版本号

上传以后就可以使用pip下载自己的包了，顺便清华源是10分钟更新一次，所以说10分钟后清华的pypi源就能找到自己的包了