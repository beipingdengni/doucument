# 打包部署

> 生成requirements.txt文件：pip freeze > requirements.txt
>
> 安装requirements.txt依赖：pip install -r requirements.txt

## 打包不含依赖

```python
# 安装
pip install -r requirements.txt
```



## 打包保护依赖

### 下载所有依赖到指定包

```python
# 下载， 指定：packages/
pip download -r requirements.txt -d packages/ -i https://pypi.tuna.tsinghua.edu.cn/simple
# 安装， 指定：./packages
pip install --no-index --find-links=./packages -r ./requirements.txt

```

