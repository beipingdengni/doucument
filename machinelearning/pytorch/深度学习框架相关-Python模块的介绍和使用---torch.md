 

# 深度学习框架相关-Python模块的介绍和使用---torch

1.torch模块，是一个开源的深度学习框架，主要用于构建和训练神经网络。PyTorch 的设计目标是提供灵活且高效的工具集，用于深度学习和科学计算；

2.下面主要介绍torch模块的五个功能：数据加载和处理，GPU加速，建立网络模型，模型的保存和加载，梯度更新和参数优化；

上面功能主要用到的子模块如下：torch.utils.data、torch.cuda、torch.nn、torch.optim

模型的保存和加载主要是主模块torch中的torch.save函数和torch.load\_state\_dict函数；

3.torch模块的很多函数设计和numpy中的函数设计是相似的；

4.文章持续修改更新；

一、数据加载和处理(torch.utils.data)
--------------------------------

```python
'''
1.预定义数据集，pytorch提供了一些数据集，如MNIST，CIFAR-10等，在torchvision.datasets中，下载使用
2.自定义数据集：torch.utils.data.Dataset是一个抽象类。所有自定义的数据集都可以继承这个类
并实现两个方法，__len__(self):返回数据集的大小和__getitem__(self,idx)：支持索引，返回一个数据集的样本
'''
import torch
from torch.utils.data import Dataset,DataLoader
 
class mydataset(Dataset):
    def __init__(self,data,labels):
        self.data = data
        self.labels = labels
    def __len__(self):
        return len(self.data)
    def __getitem__(self, idx):
        sample = {'data': self.data[idx], 'label': self.labels[idx]} 
        return sample
    #dataloader是字典形式
###实例，用上面的类设置dataset
data = torch.tensor([[1,1,1],[2,2,2],[3,3,3]])
label = torch.tensor([1,2,3])
###传入data和label两个参数到类中，使其初始化
dataset = mydataset(data,label)
dataloader = DataLoader(dataset,batch_size=2,shuffle=True,num_workers=0)
# num_workers参数支持并行加载数据，设置num_workers个子进程
for data_ in dataloader:
    print(data_)
'''
3.在加载数据之前，通常需要对数据进行预处理或变换。torchvision.transforms 提供了一些常见的变换操作，如裁剪、缩放、归一化等。
'''
from torchvision import transforms
 
transform = transforms.Compose([
    transforms.Resize((128, 128)),  ##变换形状
    transforms.ToTensor(),   
    transforms.Normalize(mean=[0.5], std=[0.5])
])
 
# 应用变换到自定义数据集
class MyDatasetWithTransform(Dataset):
    def __init__(self, data, labels, transform=None):
        self.data = data
        self.labels = labels
        self.transform = transform
 
    def __getitem__(self, idx):
        sample = {'data': self.data[idx], 'label': self.labels[idx]}
        if self.transform:
            sample['data'] = self.transform(sample['data'])
        return sample
 
# 示例使用
dataset_with_transform = MyDatasetWithTransform(data, labels, transform=transform)
##初始化类
dataloader = DataLoader(dataset_with_transform, batch_size=2, shuffle=True)
##用dataloader将dataset中的数据转化为两个batch_size，并且随机打乱
```

 ### 二、GPU加速(torch.cuda)
------------------------

```python
import torch
#检查CUDA是否存在
if torch.cuda.is_available():
    print(torch.cuda.get_device_name(0))
#模型转移到GPU上
model = torch.nn.Linear(10,1)
device = torch.device('cuda')
model.to(device)
 
#张量转移到GPU上
data = torch.randn(5,10).to(device)
output = model(data)
```

三、建立网络模型 (torch.nn)
------------------------

```python
import numpy as np
import torch
from torch import nn
import torch.nn.functional as F
 
class PositionEmbed(nn.Module):
    def __init__(self) -> None:
        super().__init__()
        pass
    def forward(self,x):
        pass
 
class Embed(nn.Module):
    def __init__(self, ) -> None:
        super().__init__()
        self.embed = nn.Embedding(32,32)  #第一个32是词汇表的大小，第二个32是想嵌入的向量的维度
        self.position = PositionEmbed()
    def forward(self,x):
        embed_x = self.embed(x)
        position_x = self.position(x)
        if position_x:
            output = embed_x + position_x
        else:
            output = embed_x
        return output 
 
class Attention(nn.Module):
    def __init__(self):
        super(Attention,self).__init__()
        self.wq = nn.Parameter(torch.randn(32,32))
        self.wk = nn.Parameter(torch.randn(32,32))
        self.norm = nn.LayerNorm(32)
    def forward(self,x):
        x = self.norm(x)
        q = torch.matmul(x,self.wq)
        k = torch.matmul(x,self.wk)
        score = F.softmax((torch.matmul(q,k.T) / np.sqrt(self.wq.shape[-1])),dim = 1)
        att_score = torch.matmul(score,k)   # 10 32
        att_score = att_score + x
        return att_score   
 
class FFN(nn.Module):
    def __init__(self):
        super(FFN,self).__init__()
        self.linear1 = nn.Linear(32,128)
        self.linear2 = nn.Linear(128,32)
        self.relu = nn.ReLU()
        self.norm = nn.LayerNorm(32)
    def forward(self,x):
        x = self.norm(x)    
        y1 = self.linear1(x)
        y2 = self.relu(y1)
        y3 = self.linear2(y2)
        result = y3 + x
        return result
 
class Model(nn.Module):
    def __init__(self) -> None:
        super().__init__()
        self.embed = Embed()
        self.attention = Attention()
        self.ffn = FFN()
    def forward(self,x):
        embed_x = self.embed(x)
        att_score = self.attention(embed_x)      #这样的话，x只在forward中有用  
        result = self.ffn(att_score)
        return result
 
x = torch.tensor([1,2,3,4,5,6,7,8,9,10],dtype=torch.long)
model = Model()
result = model(x)
print(result.shape)
##保存模型
# torch.save(model,'model.pth')
```

 ### 四、模型的保存和加载
---------------

### (torch.save函数、torch.load\_state\_dict函数)

```python
import torch
# 需要设置将model.py中的模型框架导入
from model import *
# 假设 model 是你训练好的模型
torch.save(model.state_dict(), 'model.pth')
# 假设 model 是你要加载的模型
model.load_state_dict(torch.load('model.pth'))
model.eval()  # 将模型设置为评估模式
```

五、梯度更新和参数优化(torch.optim)
-----------------------------

```python
optimizer = torch.optim.SGD(model.parameters(), lr=0.01)
outputs = model(inputs) #前向传播
optimizer.zero_grad() #清零梯度
loss = criterion(outputs, targets) #计算损失
loss.backward() #反向传播
optimizer.step() #更新参数
```
