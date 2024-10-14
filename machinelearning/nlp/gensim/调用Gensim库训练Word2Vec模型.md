**目录**

[本周任务:](#t0)

 [1.安装Gensim库](#t1)

[2.对原始语料分词](#t2)

[3.停用词](#t3)

 [4.训练Woed2Vec模型](#t4)

[5\. 模型应用](#t5)

[1.计算词汇相似度](#t6)

[2.找出不匹配的词汇](#t7)

[3.计算词汇的词频](#t8)

 [6.总结：](#t9)

[jieba分词库的使用](#t10)

[2\. Word2Vec模型的基本使用](#t11)

[3\. 词向量操作](#t12)

### **本周任务:**

**1.阅读NLP基础知识里Word2vec详解一文，了解并学习Word2vec相关知识**

**2.创建一个 .txt 文件存放自定义词汇，防止其被切分**

###  1.安装Gensim库

> !pip3 install jieba
>
> !pip3 install gensim
>
> !pip3 install pot 
>
> !pip3 install emd2 # 句子相似

![](https://i-blog.csdnimg.cn/blog_migrate/dbc46397129d6e856128e0a63d0a9b1c.png)

### 2.对原始语料分词

选择《人民的名义》的小说原文作为语料，先采用iieba进行分词。这里是直接添加的自定义词汇没有选择创建自定义词汇文件。(任务2代码处) 

```python
import jieba
import jieba.analyse
jieba.suggest_freq('沙瑞金',True)#加入一些词，使得jieba分词准确率更高
jieba.suggest_freq('田国富',True)
jieba.suggest_freq('高育良',True)
jieba.suggest_freq('侯亮平',True)
jieba.suggest_freq('钟小艾',True)
jieba.suggest_freq('陈岩石',True)
jieba.suggest_freq('欧阳菁',True)
jieba.suggest_freq('易学习',True)
jieba.suggest_freq('王大路',True)
jieba.suggest_freq('蔡成功',True)
jieba.suggest_freq('孙连城',True)
jieba.suggest_freq('季昌明',True)
jieba.suggest_freq('丁义珍',True)
jieba.suggest_freq('郑西坡',True)
jieba.suggest_freq('赵东来',True)
jieba.suggest_freq('高小琴',True)
jieba.suggest_freq('赵瑞龙',True)
jieba.suggest_freq('林华华',True)
jieba.suggest_freq('陆亦可',True)
jieba.suggest_freq('刘新建',True)
jieba.suggest_freq('刘庆祝',True)
jieba.suggest_freq('赵德汉',True)
with open('./data/in_the_name_of_people.txt', encoding='utf-8')as f:
    result_cut = []
    lines = f.readlines()
    for line in lines:
        result_cut.append(list(jieba.cut(line)))
f.close()
```

 输出结果：

![](https://i-blog.csdnimg.cn/blog_migrate/56e992682c71f7796c5c9eafeada5227.png)

### 3.停用词

在自然语言处理（NLP）中，停用词（stop words）是指在文本中频繁出现但对于传达实际意义贡献不大的词。这些词通常是冠词、介词、连词等，例如“的”、“和”、“是”、“在”等。停用词在文本中几乎无处不在，但它们并不携带太多实际的语义信息。

 拿到了分词后的文件，在一般的NLP处理中，会需要[去停用词](https://so.csdn.net/so/search?q=%E5%8E%BB%E5%81%9C%E7%94%A8%E8%AF%8D&spm=1001.2101.3001.7020)。由于word2vec的算法依赖于上下文，而上下文有可能就是停词。因此对于word2vec，我们可以不用去停词，仅仅去掉一些标点符号，做一个简单的数据清洗。

现在我们可以直接读分词后的文件到内存。这里使用了word2vec提供的LineSentence类来读文然后套用word2vec的模型。在实际应用中，可以调参提高词的embedding的效果。

```python
#添加自定义停用词
stopwords_list = [",","。","\n","\u3000","",":","!","?","…"]
def remove_stopwords(ls):  #去除停用词
    return [word for word in ls if word not in stopwords_list]
 
result_stop=[remove_stopwords(x)for x in result_cut if remove_stopwords(x)]
 
print(result_stop[100:103])
```

![](https://i-blog.csdnimg.cn/blog_migrate/0535a8599b2ddfe9c72091b867d116a6.png)

###  4.训练Woed2Vec模型

```python
from gensim.models import Word2Vec
from gensim.models.keyedvectors import KeyedVectors

model =Word2Vec(result_stop,#用于训练的语料数据
                vector_size=100,#是指特征向量的维度，默认为100。一个句子中当前单词和被预测单词的最大距离。
                window=5, # window制定了我们在训练过程中的窗口大小
                sg=0, # sg指定了我们使用的算法：sg=0则使用CBOW算法，sg=1则使用了skip-gram算法
                hs=1, # hs这个参数指定了我们使用的层次softmax算法，如果为hs=0,则使用负采样算法，如果hs=1则使用层次softmax算法
                min_count=1)#可以对字典做截断.词频少于min_count次数的单词会被丢弃掉，

# binary: 决定了是否以二进制的格式保存
# 第一个为保存为二进制文件里，第二个为保存在txt文件里
model.wv.save_word2vec_format(r'路径', binary=True)
model.wv.save_word2vec_format(r'路径', binary=False)

# 加载存盘大的词汇量模型
word_vectors = KeyedVectors.load_word2vec_format(r'路径', binary=True)
print('词汇数量：', len(word_vectors.key_to_index))
print('词汇相似度：', word_vectors.similarity('抓住', '机遇'))
print('获取到最近似的词汇：', word_vectors.most_similar('机遇', topn=2))
print(word_vectors["你"]) #v3 
print(model.wv["你"] #v4
# 使用most_similar接口进行词语的类比推理
# 这里推理father-mother = 什么-woman
# positive: 我们要找到的词汇与positive的词汇相似
# negative: 我们要找到的词汇与negative不想似
# topn: 我们要找到的词汇个数
print(word_vectors.most_similar(positive=['抓住'], negative=['新闻'], topn=2))
# 两个句子的相似度
distance = model.wv.wmdistance('产品质量不错，但价格有点贵。', '产品好')
print(distance)
# 重新训练
model.save("word2vec_model") #保存模型
new_model = word2vec.word2vec.load("word2vec_model") #v3，重新加载模型 
new_model = Word2Vec.load("word2vec_model") #v4，重新加载模型 
new_model.train(new_texts) #根据新语料再次训练模型，new_texts即为新语料。
```

### 5\. 模型应用

#### 1.计算词汇相似度

我们可以使用 similarity()方法计算两个词汇之间的余弦相似度。

```python
#计算两个词的相似度
print(model.wv.similarity('沙瑞金','季昌明'))
print(model.wv.similarity('沙瑞金','田国富'))
```

> 0.9985029  
> 0.99909985

```python
#选出最相似的5个词
for e in model.wv.most_similar(positive=['沙瑞金'],topn=5):print(e[0],e[1])
```

> 肖钢玉 0.9993638396263123  
> 李达康 0.9993550777435303  
> 意外 0.9992921352386475  
> 赵东来 0.9992125034332275  
> 这样 0.9992020130157471 

#### 2.找出不匹配的词汇

使用 doesnt\_match()方法，我们可以找到一组词汇中与其他词汇不匹配的词汇 

```python
odd_word =model.wv.doesnt_match(["苹果","香蕉","橙子","书"])
print(f"在这组词汇中不匹配的词汇：{odd_word}")
```

> 在这组词汇中不匹配的词汇：书

#### 3.计算词汇的词频

我们可以使用 get\_vecattr()方法获取词汇的词频 

```python
word_frequency = model.wv.get_vecattr("沙瑞金", "count")
print(f"沙瑞金: {word_frequency}")
```

> 沙瑞金: 353

###  6.总结：

#### **jieba分词库的使用**

*   **分词功能**：通过`jieba.cut`方法对文本进行分词。`suggest_freq`函数用于添加一些特定的词汇，以提高分词的准确性。
*   **自定义停用词**：通过`remove_stopwords`函数过滤掉指定的停用词，这些词汇在文本处理中通常不携带有用的信息，如标点符号、换行符等。

#### 2\. **Word2Vec模型的基本使用**

*   **模型初始化**：`Word2Vec`类的初始化参数：
    *   `vector_size=100`：设定特征向量的维度，通常为100维。
    *   `window=5`：指定当前词和预测词之间的最大距离。
    *   `min_count=1`：指定忽略频率小于1的词。
*   **训练模型**：通过将分词后的文本数据传入`Word2Vec`模型中，进行训练。

#### 3\. **词向量操作**

*   **计算相似度**：使用`model.wv.similarity`计算两个词之间的相似度。
*   **获取最相似的词**：`model.wv.most_similar`可以找出与给定词最相似的词汇。
*   **找到不匹配的词**：`model.wv.doesnt_match`用于找出一组词中不属于同一类别的词汇。
*   **词频查询**：使用`model.wv.get_vecattr`获取特定词的词频信息。
