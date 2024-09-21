

# Kaldi

https://github.com/kaldi-asr/kaldi

官方：https://kaldi-asr.org/



博客

kaldi入门：搭建第一个中文ASR (AISHELL-1)： https://blog.csdn.net/Ephemeroptera/article/details/106634471



```
新一代kaodi项目的相关链接：
1. 核心算法库k2 
(https://github.com/k2-fsa/k2)
2. 通用语音数据处理工具包Lhotse
(https://github.com/lhotse-speech/lhotse)
3. 语音识别完整解决方案Icefall
(https://github.com/k2-fsa/icefall)
```



## **特征提取**、**声学模型**、**发音词典**、**语言模型**、**语音解码**五个部分

### 特征提取

 目前语音识别系统常用的声学特征有：**梅尔频率倒谱系数（MFCC）、感知线性预测（PLP）、Fbank（Filter-bank）** 等。其中MFCC是相对用的比较多的，Fbank是不做DCT（离散余弦变换）的MFCC。一般经过MFCC等的特征提取后还要经过**CMVN**（Cepstral Mean and Variance Normalization，倒谱均值方差归一化）处理，以确保各个特征参数形式一致，特征参数形式不一致就不好比较，在检索匹配特征时会很麻烦。


### 声学模型

 声学模型早期多使用**DTW（Dynamic Time Warping，动态时间规整）**，现在多使用**HMM（Hidden Markov Model，隐马尔可夫模型）**。声学模型训练常使用**GMM（Gaussian mixture model，高斯混合模型）**，随着神经网络算法的兴起**DNN（Deep Neural Networks，深度神经网络）**慢慢成了主流。不过GMM也没有完全废弃，一般的声学模型训练过程还是要先进行**单音素（mono-phone）**的**GMM**训练，接着是**三音素（Triphone）**的**GMM**训练，之后是在此基础之上的**三音素DNN**训练。三音素的GMM训练和DNN训练一般都会迭代好多次，虽然每次迭代后的模型都可以使用，但是一般迭代的次数越多模型越理想。
