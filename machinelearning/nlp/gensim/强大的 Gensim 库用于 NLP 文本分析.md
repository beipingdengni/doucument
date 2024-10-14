###### 自然语言处理，简称 NLP，是人工智能的一个分支，它允许机器理解、处理和操纵人类语言。Gensim是在做自然语言处理时较为经常用到的一个工具库，主要用来以无监督的方式从原始的非结构化文本当中来学习到文本隐藏层的主题向量表达。云朵君将和大家一起学习几个关键的 NLP 主题，帮助我们更加熟悉使用 Gensim 进行文本数据操作。

### NLP基础

NLP就是处理自然语言，可以是文本、音频和视频。本文将重点了解如何使用文本数据并讨论文本数据的构建块。

#### **基本概念**

**`标记(Token)`：** 是具有已知含义的字符串，标记可以是单词、数字或只是像标点符号的字符。`“你好”`、`“123”`和`“-”`是标记的一些示例。

**`句子(Sentence)`：** 是一组意义完整的记号。**“天气看起来不错”** 是一个句子的例子，句子的标记是**【“天气”, “看起来”, “不错“】**。

**`段落(Paragraph)`：** 是句子或短语的集合，也可以将句子视为段落的标记。

**`文档(Documents)`：** 可能是一个句子、一个段落或一组段落。发送给个人的文本消息是文档的一个示例。

**`语料(Corpus)`：** 通常是作为词袋的原始文档集合。语料库包括每个记录中每个单词的 id 和频率计数。语料库的一个例子是发送给特定人的电子邮件或文本消息的集合。

![](https://ask.qcloudimg.com/http-save/yehe-8756457/3be09e696b345a90b9ad1fdb6c69e26b.png)

**`稀疏向量(SparseVector)`：** 通常，我们可以略去向量中多余的0元素。此时，向量中的每一个元素是一个(key, value)的元组

**`模型(Model)`：** 是一个抽象的术语。定义了两个向量空间的变换（即从文本的一种向量表达变换为另一种向量表达）。

### Gensim简介

大名鼎鼎的 Gensim 是一款具备多种功能的神器。它是一个著名的开源 Python 库，**用于从原始的非结构化的文本中，无监督地学习到文本隐层的主题向量表达**。它处理大量文本数据的能力和训练向量embedding的速度使其有别于其他 NLP 库。此外，Gensim 支持包括`TF-IDF，LSA，LDA，和 word2vec`在内的多种主题模型算法，用此很多算法工程师会将其作为主题建模的首选库。

Gensim支持流式训练，并提供了诸如相似度计算，信息检索等一些常用任务的API接口。

### 安装和使用

可直接使用 pip 安装或 conda 环境安装 Gensim。

代码语言：javascript

复制

```javascript
#Using Pip installer:
pip install --upgrade gensim

#Using Conda environment:
conda install -c conda-forge gensim
```

![](https://ask.qcloudimg.com/http-save/yehe-8756457/93eac21c3e34975094af3a6c1fe24572.png)

### 训练语料的预处理

训练语料的预处理指的是将文档中原始的字符文本转换成Gensim模型所能理解的稀疏向量的过程。

通常，我们要处理的原生语料是一堆文档的集合，每一篇文档又是一些原生字符的集合。在交给Gensim的模型训练之前，我们需要将这些原生字符解析成Gensim能处理的稀疏向量的格式。由于语言和应用的多样性，我们需要先对原始的文本进行分词、去除停用词等操作，得到每一篇文档的特征列表。

#### **创建字典**

首先，从句子列表中制作字典。

调用Gensim提供的API建立语料特征（word）的索引字典，并将文本特征的原始表达转化成词袋模型对应的稀疏向量的表达。可以使用 Gensim 从句子列表和文本文件中生成字典。

代码语言：javascript

复制

```javascript
import gensim
from gensim import corpora

text1 = ["""Gensim is a free open-source Python library for representing documents as semantic vectors,
           as efficiently and painlessly as possible. Gensim is designed 
           to process raw, unstructured digital texts using unsupervised machine learning algorithms."""]

tokens1 = [[item for item in line.split()] for line in text1]
g_dict1 = corpora.Dictionary(tokens1)

print("The dictionary has: " +str(len(g_dict1)) + " tokens\n")
print(g_dict1.token2id)
```

代码语言：javascript

复制

```javascript
The dictionary has: 29 tokens

{'Gensim': 0, 'Python': 1, 'a': 2, 'algorithms.': 3, 
'and': 4, 'as': 5, 'designed': 6, 'digital': 7, 
'documents': 8, 'efficiently': 9, 'for': 10, 'free': 11, 
'is': 12, 'learning': 13, 'library': 14, 'machine': 15, 
'open-source': 16, 'painlessly': 17, 'possible.': 18, 
'process': 19, 'raw,': 20, 'representing': 21, 
'semantic': 22, 'texts': 23, 'to': 24, 'unstructured': 25, 
'unsupervised': 26, 'using': 27, 'vectors,': 28}
```

可以从输出中看到字典中的每个标记都分配了一个唯一的 `id`。

现在，用文本文件中的tokens创建一个字典。开始时使用 Gensim 的 `simple_preprocess()` 函数对文件进行预处理，从文件中检索tokens列表。

代码语言：javascript

复制

```javascript
from gensim.utils import simple_preprocess
from gensim import corpora
# text2 = open('sample_text.txt', encoding ='utf-8').read()
text2 = """
NLP is a branch of data science that consists of systematic processes for analyzing,
understanding, and deriving information from the text data in a smart and efficient manner. 
By utilizing NLP and its components, one can organize the massive chunks of text data, 
perform numerous automated tasks and solve a wide range of problems such as – 
automatic summarization, machine translation, named entity recognition, 
relationship extraction, sentiment analysis, speech recognition, and topic segmentation etc.
"""
 
tokens2 =[]
for line in text2.split('.'):
    tokens2.append(simple_preprocess(line, deacc = True))
g_dict2 = corpora.Dictionary(tokens2)
print("The dictionary has: " +str(len(g_dict2)) + " tokens\n")
print(g_dict2.token2id)
```

![](https://ask.qcloudimg.com/http-save/yehe-8756457/de33965376a829cfc5bc13ac8af60108.png)

现在已经成功地从文本文件中创建了一个字典。

还可以使用新文档中的标记更新现有字典。

代码语言：javascript

复制

```javascript
g_dict1.add_documents(tokens2)
print("The dictionary has: " +str(len(g_dict1)) + " tokens\n")
print(g_dict1.token2id)
```

![](https://ask.qcloudimg.com/http-save/yehe-8756457/4a560ff6a45fa9ea92bbb22aaa5eb015.png)

#### **创建一个词袋**

使用 Gensim 的 `doc2bow` 函数从创建的字典中生成 **Bag of Words** (词袋)。词袋返回一个元组向量，其中包含每个标记的唯一 **id** 和文档中出现的次数。

代码语言：javascript

复制

```javascript
g_bow =[g_dict1.doc2bow(token, allow_update = True)
        for token in tokens1]
print("Bag of Words : ", g_bow)
```

![](https://ask.qcloudimg.com/http-save/yehe-8756457/6c46eebcbfde6acb972257be96bcd859.png)

#### **保存和加载语料库**

可以保存 Gensim 字典和 BOW语料库，并在需要时加载它们。

代码语言：javascript

复制

```javascript
# Save the Dictionary and BOW
g_dict1.save('./g_dict1.dict') 
corpora.MmCorpus.serialize('./g_bow1.mm', g_bow)  

# Load the Dictionary and BOW
g_dict_load = corpora.Dictionary.load('./g_dict1.dict')
g_bow_load = corpora.MmCorpus('./g_bow1.mm')
```

到这里，训练语料的预处理工作就完成了。我们得到了语料中每一篇文档对应的稀疏向量（这里是bow向量）；向量的每一个元素代表了一个 word在这篇文档中出现的次数。值得注意的是，虽然词袋模型是很多主题模型的基本假设，这里介绍的 `doc2bow` 函数并不是将文本转化成稀疏向量的唯一途径。后面我们将介绍更多的向量变换函数。

其次，出于内存优化的考虑，Gensim 支持文档的流式处理。我们需要做的，只是将上面的列表封装成一个Python迭代器；每一次迭代都返回一个稀疏向量即可。

代码语言：javascript

复制

```javascript
class MyCorpus(object):
def __iter__(self):
    for line in open('mycorpus.txt'):
        # 假设每行有一个文档，标记用空格分隔
        yield dictionary.doc2bow(line.lower().split())
```

### 主题向量的变换

对文本向量的变换是 Gensim 的核心。通过挖掘语料中隐藏的语义结构特征，我们最终可以变换出一个简洁高效的文本向量。

在 Gensim 中，每一个向量变换的操作都对应着一个主题模型，例如上一小节提到的对应着词袋模型的 `doc2bow` 变换。每一个模型又都是一个标准的Python对象。下面以TF-IDF模型为例，介绍 Gensim 模型的一般使用方法。

#### **创建 TF-IDF**

**词频—逆文档频率（TF-IDF）** 是一种通过计算词的权重来衡量文档中每个词的重要性的技术。在 TF-IDF 向量中，每个词的权重与该词在该文档中的出现频率成反比。

首先是模型对象的初始化。通常，Gensim模型都接受一段训练语料（注意在Gensim中，语料对应着一个稀疏向量的迭代器）作为初始化的参数。显然，越复杂的模型需要配置的参数越多。

代码语言：javascript

复制

```javascript
from gensim import models
import numpy as np

text = ["The food is excellent but the service can be better",
        "The food is always delicious and loved the service",
        "The food was mediocre and the service was terrible"]
g_dict = corpora.Dictionary([simple_preprocess(line) for line in text])
g_bow = [g_dict.doc2bow(simple_preprocess(line)) for line in text]

print("Dictionary : ")
for item in g_bow:
    print([[g_dict[id], freq] for id, freq in item])

g_tfidf = models.TfidfModel(g_bow, smartirs='ntc')
print("TF-IDF Vector:")
for item in g_tfidf[g_bow]:
    print([[g_dict[id], np.around(freq, decimals=2)] for id, freq in item])
```

![](https://ask.qcloudimg.com/http-save/yehe-8756457/008597dde6e3cbc0ef96cca8fb7567af.png)

代码`tfidf = models.TfidfModel(corpus)`，完成对语料库corpus中出现的每一个特征的IDF值的统计工作。其中，corpus是一个返回bow向量的迭代器。需要注意的是，**这里的bow向量必须与训练语料的bow向量共享同一个特征字典（即共享同一个向量空间**）。

注意，同样是出于内存的考虑，`model[corpus]`方法返回的是一个迭代器。如果要多次访问`model[corpus]`的返回结果，可以先将结果向量序列化到磁盘上。

将训练好的模型保存到磁盘上，以便下一次使用：

代码语言：javascript

复制

```javascript
tfidf.save("./model.tfidf")
tfidf = models.TfidfModel.load("./model.tfidf")
```

#### **创建Bigrams和Trigrams**

一些单词通常出现在一个大文档的文本中。当这些词同时出现时，它们可能作为一个实体出现，与单独出现时的意思完全不同。

以`“世界之窗”`为例，当它们同时出现（世界之窗）的时候和单独出现（世界，窗）的时候有完全不同的意思，这些词组被称为`“N-gram”`。

`Bigrams`二元组是由2个单词组成的`N-gram`，`Trigrams` 三元组是由3个单词组成的。

接下来将为`“text8”`数据集创建二元组和三元组，可通过 **Gensim Downloader API** 下载。并使用 **Gensim** 的 `Phrases` 功能。

`Trigram` 模型是通过将之前获得的 `bigram` 模型传递给 `Phrases` 函数来生成的。

代码语言：javascript

复制

```javascript
import gensim.downloader as api
from gensim.models.phrases import Phrases
 
dataset = api.load("text8")
tokens = [word for word in dataset]
            
bigram_model = Phrases(tokens, min_count = 3, threshold = 10)
print(bigram_model[tokens[0]]) 

trigram_model = Phrases(bigram_model[data], threshold = 10)
print(trigram_model[bigram_model[data[0]]])
```

第一次运行时会下载数据集：

![](https://ask.qcloudimg.com/http-save/yehe-8756457/c4656289075ed3d09b2d7bc02af2fea8.png)

运行结束后，输出结果。

![](https://ask.qcloudimg.com/http-save/yehe-8756457/c16fb6116b8096cf33daf8c21f8b9cb5.png)

#### **创建 Word2Vec 模型**

Word Embedding 模型是将文本表示为数字向量的模型。

`Word2Vec` 是 Gensim 的一个预先构建的词嵌入模型，它使用外部神经网络将词嵌入到低维向量空间中。Gensim 的 `Word2Vec` 模型可以实现 `Skip-grams` 模型和 `Continuous Bag of Words` 模型。

接下来为`“text8”`数据集的前 1000 个单词训练 `Word2Vec` 模型。

代码语言：javascript

复制

```javascript
from gensim.models.word2vec import Word2Vec
from multiprocessing import cpu_count

# import gensim.downloader as api
# dataset = api.load("text8")

words = [d for d in dataset]
data1 = words[:1000]
w2v_model = Word2Vec(data1, min_count = 0,
                     workers=cpu_count())
print(w2v_model.wv['social'])
```

![](https://ask.qcloudimg.com/http-save/yehe-8756457/bee4edcbe0962940843b6cf4c61581e8.png)

上面的输出是通过这个模型找到的`“Social”`的词向量。

使用 `most_similar` 函数，可以得到所有与该“Social”词相似的词。

代码语言：javascript

复制

```javascript
print(w2v_model.wv.most_similar('social'))
```

代码语言：javascript

复制

```javascript
[('political', 0.7943845987319946),
 ('cultural', 0.7576460838317871),
 ('ideology', 0.7451750040054321),
 ('progressive', 0.725498378276825),
 ('intellectual', 0.7218820452690125),
 ('discipline', 0.7171103954315186),
 ('religious', 0.7035976052284241),
 ('radical', 0.7034412026405334),
 ('socio', 0.700967013835907),
 ('promoting', 0.6993831396102905)]
```

同样，还可以保存 `Word2Vec` 模型并在需要的时候将其加载回来。

代码语言：javascript

复制

```javascript
w2v_model.save('./w2v_model1')
w2v_model = Word2Vec.load('./w2v_model1')
```

Gensim 还具有一项功能，可更新现有的 `Word2Vec` 模型。可以通过调用 `build_vocab` 函数和 `train` 函数来更新模型。

代码语言：javascript

复制

```javascript
data2 = words[1000:]
w2v_model.build_vocab(data2, update=True)
w2v_model.train(data2, total_examples=w2v_model.corpus_count, 
                epochs=w2v_model.epochs)
print(w2v_model.wv['social']) # numpy.array()
```

![](https://ask.qcloudimg.com/http-save/yehe-8756457/f4138601aec1d7095aba551f17ed73ed.png)

### 文档相似度的计算

在得到每一篇文档对应的主题向量后，我们就可以计算文档之间的相似度，进而完成如文本聚类、信息检索之类的任务。在Gensim中，也提供了这一类任务的API接口。

以信息检索为例。对于一篇待检索的query，我们的目标是从文本集合中检索出主题相似度最高的文档。

首先，我们需要将待检索的query和文本放在同一个向量空间里进行表达（以LSI向量空间为例）

代码语言：javascript

复制

```javascript
# 构造LSI模型并将待检索的query和文本转化为LSI主题向量
# 转换之前的corpus和query均是BOW向量
lsi_model = models.LsiModel(corpus, id2word=dictionary,
                            num_topics=2)
documents = lsi_model[corpus]
query_vec = lsi_model[query]
```

接下来，我们用待检索的文档向量**初始化一个相似度计算的对象**：

代码语言：javascript

复制

```javascript
index = similarities.MatrixSimilarity(documents)
```

我们也可以通过`save()`和`load()`方法持久化保存这个相似度矩阵：

代码语言：javascript

复制

```javascript
index.save('/tmp/test.index')
index = similarities.MatrixSimilarity.load('/tmp/test.index')
```

注意，如果待检索的目标文档过多，使用`similarities.MatrixSimilarity`类往往会带来内存不够用的问题。此时，可以改用`similarities.Similarity`类。二者的接口基本保持一致。

最后，我们借助`index`对象计算任意一段`query`和所有文档的（余弦）相似度：

代码语言：javascript

复制

```javascript
sims = index[query_vec]
# 返回一个元组类型的迭代器：(idx, sim)
```

