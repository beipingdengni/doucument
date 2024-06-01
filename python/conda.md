## CondaConda And Anaconda 操作

Miniconda 安装： https://conda.io/miniconda.html  【最小安装】

Anaconda 安装：https://www.anaconda.com/distribution    【图形界面安装】

>  参考博客：https://blog.csdn.net/Code_LT/article/details/134928013

安装好切换中国镜像源

```shell
查看数据源： conda config --show channels
删除数据源： conda config --remove channels

# 恢复默认
conda config --remove-key channels
conda config --add channels defaults

# 添加清华源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud//pytorch/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge/

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


# 配置完后如果不生效可清空conda缓存再试试
conda clean -i
```



##### conda基本操作

```shell
conda的作用：
升级：
conda版本：conda --version
更新conda：conda update conda
删除conda：rm -rf ~/miniconda OR  rm -rf ~/anaconda

创建：
	conda可以给我们提供一个独立的环境，相当于python的virtualenv
	# conda create --prefix=指定目录 python=[version]
		# 或 conda create -p=指定目录 python=[version]
	# conda activate [指定的目录]
		conda create -name envname python=2.7
		#conda create -n myenv python=2.75 -c conda-forge   # -c 指定渠道
	激活：conda activate envname
	退出：conda deactivate
查看
	列出conda创建所有的环境: conda info --envs
	查看当前的环境目录：conda config --show envs_dirs
切换：
	切换回当前环境：conda deactivate
删除：
	删除环境：conda remove -n flowers --all 
clone复制
	制作环境副本：conda create --name flowers --clone snowflakes

环境空间：
	添加：conda config --append envs_dirs ~/Documents/python/env 追加的环境目录
	删除：conda config --remove envs_dirs ~/Documents/python/env c追加的环境目录
	查看：conda config --show envs_dirs （当前环境的目录有哪些）
		# 可以在 ~/Documents/python/env 目录下创建虚拟环境
		# conda create -p=~/Documents/python/env/pd_env python=3
		# conda env list 查看就会多一个： pd_env 环境

包管理
	查看在环境中安装的第三方包：conda list
	搜索可安装的包：conda search
	安装新软件：conda install --name packagename beautifulsoup4
	也可以使用pip安装：pip install pkg
	指定源安装
		安装：conda install bottleneck --channel https://conda.anaconda.org/pandas
	删除环境的第三方包：conda remove --name envname pck或者 pip uninstall pck

conda deactivate # 退出当前环境 
conda env remove -n py38 # 删除名为py38的环境 
conda create -n py38 python=3.8 # 创建一个新的Python 3.8环境 
conda activate py38 # 激活新环境
```