gensim 的几个基础模型：

*   1 - Word2Vec
*   2 - Doc2Vec
*   3 - Latent Dirichlet Allocation (LDA)
*   4 - TF-IDF
*   5 - FastText
*   6 - Hierarchical Dirichlet Process (HDP)
*   7 - Random Projection
*   8 - Latent Semantic Indexing (LSI)

1 Word2Vec
----------

Word2Vec 是一种用于将自然语言中的词语映射为向量的技术。它基于分布式假设，即在给定的语料库中，相似的词语通常出现在相似的上下文中。该模型通过使用神经网络来学习每个词语的向量表示，使得相似的词语在向量空间中的距离更近。

Gensim 中的 Word2Vec 实现提供了许多参数和选项，以允许用户自定义模型的训练方式。例如，用户可以选择使用 Skip-gram 或 CBOW（Continuous Bag-of-Words）模型，定义向量空间的维数，以及指定语料库中的词语数量和迭代次数。

训练完成后，Word2Vec 模型可以用于许多 NLP 任务，例如词汇相似性和句子分类。它还可以与其他深度学习模型结合使用，以提高这些任务的性能。

```python
from gensim.models import Word2Vec
sentences = [['this', 'is', 'the', 'first', 'sentence', 'for', 'word2vec'], 
             ['this', 'is', 'the', 'second', 'sentence'], 
             ['yet', 'another', 'sentence'], 
             ['one', 'more', 'sentence'], 
             ['and', 'the', 'final', 'sentence']]
model = Word2Vec(sentences, min_count=1)

## Prints the word vector for the word 'sentence'
print(model.wv['sentence']) 
```

<img src="https://pic3.zhimg.com/v2-f775883e81008e1d267c225c61ee64b6\_b.jpg"/>

![](https://pic3.zhimg.com/80/v2-f775883e81008e1d267c225c61ee64b6_720w.webp)

2 doc2vec
---------

doc2vec，是一种用于将文档嵌入到向量空间中的算法。doc2vec是对word2vec的扩展，它可以将整个文档转换为一个向量，而不是仅仅将文档中的单词转换为向量。

doc2vec使用的是分布式记忆（distributed memory）和分布式袋子（distributed bag of words）两种方法之一来生成文档向量。在分布式记忆方法中，doc2vec通过在词向量和文档向量之间添加额外的文档特定权重向量，来将每个文档映射到一个向量。在分布式袋子方法中，doc2vec将整个文档视为一个“袋子”，并将其表示为单词的总体统计信息。

在gensim中使用doc2vec通常包括以下步骤：

1.  准备文本数据，包括分词和预处理。
2.  创建一个doc2vec模型对象，并设置一些参数，例如向量维度和训练次数等。
3.  使用模型对象对文档进行训练，以生成每个文档的向量表示。
4.  对生成的向量进行可视化或用于其他任务，例如分类或聚类。

使用doc2vec可以将文档转换为向量表示，这有助于在文档级别进行语义相似性计算、文档分类和聚类等任务。

```python
from gensim.models.doc2vec import Doc2Vec, TaggedDocument
documents = [TaggedDocument(doc, [i]) for i, doc in enumerate(sentences)]
model = Doc2Vec(documents, vector_size=5, window=2, min_count=1, workers=4)

## Print the inferred vector for the given sentence
print(model.infer_vector(['this', 'is', 'a', 'test'])) 
```

<img src="https://pic3.zhimg.com/v2-77a4712d6b5f7a24f9c8f238a930b336\_b.png"/>

![](https://pic3.zhimg.com/80/v2-77a4712d6b5f7a24f9c8f238a930b336_720w.webp)

3 LDA
-----

LDA是一种主题模型，可用于发现给定文本集合中隐藏的主题。在LDA中，每个文档被表示为主题的分布，每个主题被表示为单词的分布。这些分布是概率分布，因此它们的值总和为1。

使用Gensim的LDA实现，您可以将文本数据作为输入，并生成主题分布。首先，LDA将文本数据转换为数字表示形式，称为“词袋”（bag of words）。在词袋中，每个单词都被分配了一个唯一的整数ID。然后，LDA使用这些数字来计算每个文档的主题分布。

在LDA中，主题是通过单词的联合概率分布来定义的。在Gensim的LDA实现中，这些概率分布是通过对输入文本数据执行EM算法（Expectation-Maximization algorithm）来估计的。在训练过程中，LDA根据文本数据中的单词共现情况调整主题分布和单词分布，直到达到最佳匹配。在此过程中，每个文档都会被赋予最有可能的主题分布。

最终，LDA将生成每个主题的单词分布和每个文档的主题分布。这些结果可以用于对文本数据进行聚类或分类，或者用于分析文本数据中的主题。

```python
from gensim import corpora, models
dictionary = corpora.Dictionary(sentences)
corpus = [dictionary.doc2bow(sentence) for sentence in sentences]
lda_model = models.LdaModel(corpus, num_topics=3, id2word=dictionary, passes=10)

## Print the top 3 words for each of the 3 topics
print(lda_model.print_topics(num_words=3)) 
```

<img src="https://pic1.zhimg.com/v2-f954f6bbac946cd6e3bcf2a23208f304\_b.jpg"/>

![](https://pic1.zhimg.com/80/v2-f954f6bbac946cd6e3bcf2a23208f304_720w.webp)

4 TF-IDF
--------

tf-idf是一种常用的文本特征提取方法，它将一个文档中的词语权重化，同时考虑到了文档频率（df）和词语频率（tf），以便更好地表示一个文档的内容。tf-idf的全称是term frequency-inverse document frequency，即“词频-逆文档频率”。

gensim中的tf-idf实现主要是通过TfidfModel类来完成的。它的输入是一个已经被分词并通过gensim提供的Dictionary类转换成了词袋（bag of words）表示的文档集合，输出是一个包含每个文档tf-idf值的矩阵。在计算过程中，TfidfModel会先计算每个词在所有文档中的文档频率，然后将每个文档中的每个词的tf值乘以该词的idf值，最终得到tf-idf值。可以通过调整参数，如设置不同的idf加权方式或忽略一些词语，来控制tf-idf的计算过程。

```python
from gensim import corpora, models
dictionary = corpora.Dictionary(sentences)
corpus = [dictionary.doc2bow(sentence) for sentence in sentences]
tfidf_model = models.TfidfModel(corpus)
tfidf_corpus = tfidf_model[corpus]
for doc in tfidf_corpus:
    
    ## Print the TF-IDF representation of each document in the corpus
    print(doc) 
```

<img src="https://pic1.zhimg.com/v2-0bf8cd2de7bd7ceb234c398578cfcd7a\_b.png">

![](https://pic1.zhimg.com/80/v2-0bf8cd2de7bd7ceb234c398578cfcd7a_720w.webp)

5 FastText
----------

FastText是一种词向量模型，它通过将每个单词表示为它的字符级别n-gram的平均值来建模。与传统的词向量模型（例如word2vec）不同，FastText可以捕捉单词内部的子词信息，因此它能够处理许多形态变化的单词。

Gensim中的FastText实现了Facebook Research团队在2016年发表的FastText论文中描述的算法。使用Gensim中的FastText，您可以将一个文本语料库转换为词向量表示，然后使用这些向量进行各种自然语言处理任务，例如文本分类，相似度计算等。您可以在Gensim的文档中找到更多关于如何使用FastText的信息和示例。

```python
from gensim.models import FastText
sentences = [['this', 'is', 'the', 'first', 'sentence', 'for', 'FastText'], 
             ['this', 'is', 'the', 'second', 'sentence'], 
             ['yet', 'another', 'sentence'], 
             ['one', 'more', 'sentence'], 
             ['and', 'the', 'final', 'sentence']]
model = FastText(sentences, min_count=1)
print(model.wv['sentence']) # Prints the word vector for the word 'sentence'
```

<img src="https://pic2.zhimg.com/v2-559bc4dcf107b70a0366061c89793289\_b.jpg"/>

![](https://pic2.zhimg.com/80/v2-559bc4dcf107b70a0366061c89793289_720w.webp)

6 Hierarchical Dirichlet Process (HDP)
--------------------------------------

Gensim中的HDP是一种基于概率图模型的文本主题建模算法，全称为Hierarchical Dirichlet Process。它是一种非参数贝叶斯模型，可以自适应地确定主题数量，因此不需要预先指定主题数。

HDP算法的基本思想是将主题分布看作无限深的Dirichlet过程的混合分布，用于描述主题数量的无限性和自适应性。在HDP模型中，每个文档的主题分布是由多个层级的Dirichlet过程进行混合而成，其中每个层级表示一组主题。HDP模型可以自动推断出主题数，并且可以有效地处理长尾词汇（频次较低的词汇）。

在Gensim中，HDP算法的实现基于文档-词频矩阵，通过迭代更新主题分布和单词分布，最终得到每个文档的主题分布和主题的单词分布。由于HDP算法具有很好的自适应性和灵活性，因此它被广泛应用于文本挖掘、信息检索和推荐系统等领域。

```python
from gensim.models import HdpModel
dictionary = corpora.Dictionary(sentences)
corpus = [dictionary.doc2bow(sentence) for sentence in sentences]
hdp_model = HdpModel(corpus, id2word=dictionary)
print(hdp_model.print_topics(num_topics=3, num_words=3)) # Prints the top 3 words for each of the 3 inferred topics
```

<img src="https://picx.zhimg.com/v2-26dd28fe1fe69bc9c8868c6296c89b8b\_b.png" />

![](https://picx.zhimg.com/80/v2-26dd28fe1fe69bc9c8868c6296c89b8b_720w.webp)

7 Random Projections
--------------------

Random Projection，可以用于将高维度的文本向量降维。

在自然语言处理中，经常需要将文本数据表示成向量形式，以便进行机器学习或其他算法的应用。通常情况下，每个文本被表示成一个高维度的向量，其中每个维度表示一个特征或词语。

然而，高维度向量会带来计算和存储上的挑战。为了克服这些问题，可以使用降维技术，将高维度向量转换为低维度向量，同时保留数据的关键信息。

Random Projection是一种常见的降维技术，其基本思想是通过随机投影将高维度向量映射到低维度空间中。具体来说，对于一个d维度的向量，可以通过一个随机的d×k矩阵R将其映射到一个k维度的向量。这里k是低维度向量的维度，通常情况下k<<d。

使用Random Projection的好处在于，随机映射可以帮助保留向量之间的距离关系。虽然经过映射后向量的维度变小，但它们之间的相对位置仍然得以保留，这是由于随机矩阵R可以帮助保留向量之间的内积。

在gensim中，Random Projection可以通过使用ProjectionIndex或者RandomIndexer实现。这些工具可以用于将高维度文本向量降维，并且在搜索和相似度计算时具有较高的效率。

```python
## 官方例子
from gensim.models import RpModel
from gensim.corpora import Dictionary
from gensim.test.utils import common_texts, temporary_file

dictionary = Dictionary(common_texts)  # fit dictionary
corpus = [dictionary.doc2bow(text) for text in common_texts]  # convert texts to BoW format

model = RpModel(corpus, id2word=dictionary)  # fit model
result = model[corpus[3]]  # apply model to document, result is vector in BoW format
print(result)
```

<img src="https://pic4.zhimg.com/v2-b2b0570b8be668c5cb9fa775e4c6e6a3\_b.jpg" />

![](https://pic4.zhimg.com/80/v2-b2b0570b8be668c5cb9fa775e4c6e6a3_720w.webp)

8 Latent Semantic Indexing (LSI)
--------------------------------

LSI（Latent Semantic Indexing）是一种基于奇异值分解（SVD）的降维技术，用于在文本语料库中发现潜在的语义结构。具体来说，它将文本语料库中的文档表示为向量空间模型中的向量，然后使用SVD来减少这些向量的维数。通过这种方式，LSI可以捕获文档中的潜在主题，以便更好地理解和比较文档。

gensim的LSI模块提供了一个实现LSI算法的API，可以用于计算文本语料库中文档之间的相似度、主题模型等。通常，使用LSI的流程包括以下几个步骤：

1.  准备文本语料库，并将其转换为向量空间模型中的向量。
2.  使用LSI模块中的API对文档向量进行SVD分解。
3.  使用分解后的矩阵来计算文档之间的相似度、主题模型等。

```python
from gensim import corpora, models
dictionary = corpora.Dictionary(sentences)
corpus = [dictionary.doc2bow(sentence) for sentence in sentences]
lsi_model = models.LsiModel(corpus, num_topics=2, id2word=dictionary)
print(lsi_model.print_topics(num_topics=2, num_words=3)) # Prints the top 3 words for each of the 2 topics
```

<img src="https://pic2.zhimg.com/v2-9180d1b8cd4aea4daa16963562a0485b\_b.png" />

![](https://pic2.zhimg.com/80/v2-9180d1b8cd4aea4daa16963562a0485b_720w.webp)

