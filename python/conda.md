## CondaConda And Anaconda 操作

Miniconda 安装： https://conda.io/miniconda.html  【最小安装】

Anaconda 安装：https://www.anaconda.com/distribution    【图形界面安装】

安装好切换中国镜像源

```
查看数据源： conda config --show channels
删除数据源： conda config --remove channels

# 添加清华源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge 
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/

# 设置搜索时显示通道地址
conda config --set show_channel_urls yes
=============================================================================

# 中科大源
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/msys2/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/bioconda/
conda config --add channels https://mirrors.ustc.edu.cn/anaconda/cloud/menpo/

conda config --set show_channel_urls yes
=============================================================================

文件配置 ~/.condarc
# vim ~/.condarc
channels:
  - https://mirrors.ustc.edu.cn/anaconda/pkgs/main/
  - https://mirrors.ustc.edu.cn/anaconda/cloud/conda-forge/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults
show_channel_urls: true
```



##### conda基本操作

```
conda的作用：
1. conda可以给我们提供一个独立的环境，相当于python的virtualenv
conda create -name envname python=2.7
activate envname
2. conda info -envs 列出conda创建所有的环境
conda版本：conda --version
更新conda：conda update conda
切换回当前环境：deactivate
删除环境：conda remove --name flowers --all
制作环境副本：conda create --name flowers --clone snowflakes
查看在环境中安装的第三方包：conda list
搜索可安装的包：conda search
安装新软件：conda install --name packagename beautifulsoup4
也可以使用pip安装：pip install pkg
也可以从其它页面下载安装：conda install --channel https://conda.anaconda.org/pandas bottleneck
删除环境的第三方包：conda remove --name envname pck或者 pip uninstall pck
删除conda：rm -rf ~/miniconda OR  rm -rf ~/anaconda
```