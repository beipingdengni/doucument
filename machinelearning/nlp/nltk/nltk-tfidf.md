# NLTK

<img src="https://i-blog.csdnimg.cn/blog_migrate/ecda799a3f88210065b4ff7d49e96b03.png" alt="img" style="zoom:5%;" />

## tf_idf

```python
import nltk
from nltk.text import TextCollection
 
sents = ['this is sentence one', 'this is sentence two', 'this is sentence three']
corpus = TextCollection(sents)
 
#计算tf
print(corpus.tf('three', nltk.word_tokenize('one two three, go')))
#0.2			# 1/5
 
#之前上面已将全部的语料库fit进corpus去，计算 idf 只需传递要计算的 word 即可：
#计算idf
print(corpus.idf('this'))
#0.0			# log(3/3)
print(corpus.idf('three'))
#1.0986122886681098		# log(3/1)
 
#计算tf-idf
print(corpus.tf_idf('three', nltk.word_tokenize('one two three, go')))
```



## sklearn

```python
from sklearn.feature_extraction.text import TfidfTransformer
from sklearn.feature_extraction.text import CountVectorizer
 
#语料
corpus = ["I come to China to travel",
          "This is a car popular in China",
          "I love tea and Apple",
          "The work is to write some papers in science"]
 
vectorizer = CountVectorizer() #将文本中的词语转换为词频矩阵。
transformer = TfidfTransformer() #TfidfTransformer用于统计vectorizer中每个词语的TF-IDF值
 
tfidf = transformer.fit_transform(vectorizer.fit_transform(corpus))  #fit_transform函数计算各个词语出现的次数
```

