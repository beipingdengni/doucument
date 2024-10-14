三种方法实现PCA算法（Python）：https://www.jianshu.com/p/9cf498a1ab56

> 主要作用是对高维数据进行降维（计算流程，如下）
>
> - 组织数据形式，以便于模型使用；
> - 计算样本每个特征的平均值；
> - 每个样本数据减去该特征的平均值（归一化处理）；
> - 求协方差矩阵；
> - 找到协方差矩阵的特征值和特征向量；
> - 对特征值和特征向量重新排列（特征值从大到小排列）；
> - 对特征值求取累计贡献率；
> - 对累计贡献率按照某个特定比例，选取特征向量集的字迹合；
> - 对原始数据（第三步后）。
>
> ```python
> from sklearn.decomposition import PCA
> 
> mat = [[-1,-1,0,2,1],[2,0,0,-1,-1],[2,0,1,1,0]]
> 
> # PCA by Scikit-learn
> pca = PCA(n_components=2) # n_components can be integer or float in (0,1)
> pca.fit(mat)  # fit the model
> print(pca.fit_transform(mat))  # transformed data
> ```
>
> 协防差
>
> 概念: https://blog.51cto.com/emanlee/7750022
>
> 实现：https://zhuanlan.zhihu.com/p/363213507
>
> ```python
> # !pip3 install torch
> import torch
> t_x = torch.randn((3,4)) #三个样本，四个维度，也即4个random variable
> t_x_mean = torch.mean(t_x, dim=0).view(1,4)
> t_x_1 = t_x - t_x_mean
> t_x_matrix = torch.matmul(t_x_1.t(), t_x_1)
> t_x_matrix = t_x_matrix/(t_x.size(0)-1)
> 
> t_x_matrix_torch = torch.cov(t_x.t())
> print(t_x_matrix)
> print(t_x_matrix_torch)
> ```
>
> 

