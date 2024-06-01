# 安装依赖

```python
pip install -U scikit-learn -i https://mirrors.aliyun.com/pypi/simple/
pip install -U jieba -i https://mirrors.aliyun.com/pypi/simple/

#  引入包
from sklearn.feature_extraction.text import TfidfVectorizer
import numpy as np
from scipy.linalg import norm
import jieba
```

### 网站

jieba： https://github.com/fxsjy/jieba

sklearn： https://scikit-learn.org/stable/user_guide.html

# 句子相似性

## 编辑距离计算

```java
String question = "咨询问题的句子";
String query="配置问题的句子"
int questionLen = question.length();
int queryLen = query.length();
int count = 0;
int num = 0;
double score = 0;
if (queryLen >= questionLen) {
  count = minDistance(query, question);
  num = queryLen - count;
  score = (double) num / questionLen;
  log.info("query:{}  question:{}  count:{}  score:{}", query, question, count, score);
} else {
  count = minDistance(question, query);
  num = questionLen - count;
  score = (double) num / queryLen;
  log.info("query:{}  question:{}  count:{}  score:{}", question, query, count, score);
}
// 打印得分
System.out.println(score)

/**
* 计算边距
*
* @param word1 问法1
* @param word2 问法2
* @return int
*/
private int minDistance(String word1, String word2) {
  int[][] dp = new int[word1.length() + 1][word2.length() + 1];
  for (int i = 0; i < word1.length() + 1; i++) {
    // 从i个字符变成0个字符，需要i步（删除）
    dp[i][0] = i;
  }
  for (int i = 0; i < word2.length() + 1; i++) {
    // 当从0个字符变成i个字符，需要i步(增加)
    dp[0][i] = i;
  }
  for (int i = 1; i < word1.length() + 1; i++) {
    for (int j = 1; j < word2.length() + 1; j++) {
      //当相同的时，dp[i][j] = dp[i - 1][j - 1]
      if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
        dp[i][j] = dp[i - 1][j - 1];
      } else {
        //当不同的时候，我们需要求三种操作的最小值
        //其中dp[i - 1][j - 1]表示的是替换，dp[i - 1][j]表示删除字符，do[i][j - 1]表示的是增加字符
        dp[i][j] = 1 + Math.min(dp[i - 1][j - 1], Math.min(dp[i - 1][j], dp[i][j - 1]));
      }
    }
  }
  return dp[word1.length()][word2.length()];
}
```

Python 版本

```python
# pip3 install distance
import distance
def edit_distance(s1, s2):
    return distance.levenshtein(s1, s2)

strings = [
    '你在干什么',
    '你在干啥子',
    '你在做什么',
    '你好啊',
    '我喜欢吃香蕉'
]
target = '你在干啥'
results = list(filter(lambda x: edit_distance(x, target) <= 2, strings))
print(results)
```



## 杰卡德系数计算

```python
from sklearn.feature_extraction.text import CountVectorizer
import numpy as np

def jaccard_similarity(s1, s2):
    def add_space(s):
        return ' '.join(list(s))
    
    # 将字中间加入空格
    s1, s2 = add_space(s1), add_space(s2)
    # 转化为TF矩阵
    cv = CountVectorizer(tokenizer=lambda s: s.split())
    corpus = [s1, s2]
    vectors = cv.fit_transform(corpus).toarray()
    # 求交集
    numerator = np.sum(np.min(vectors, axis=0))
    # 求并集
    denominator = np.sum(np.max(vectors, axis=0))
    # 计算杰卡德系数
    return 1.0 * numerator / denominator

s1 = '你在干嘛呢'
s2 = '你在干什么呢'
print(jaccard_similarity(s1, s2))
```

## df（词频）

### 两个向量夹角的余弦值

>  cosθ=a·b/|a|*|b|

python 代码实现

```python
from sklearn.feature_extraction.text import CountVectorizer
import numpy as np
from scipy.linalg import norm

def tf_similarity(s1, s2):
    def add_space(s):
        return ' '.join(list(s))
    # 将字中间加入空格
    s1, s2 = add_space(s1), add_space(s2)
    # 转化为TF矩阵
    cv = CountVectorizer(tokenizer=lambda s: s.split())
    corpus = [s1, s2]
    vectors = cv.fit_transform(corpus).toarray()
    # 计算TF系数
    return np.dot(vectors[0], vectors[1]) / (norm(vectors[0]) * norm(vectors[1]))
# 使用了 np.dot() 方法获取了向量的点乘积，然后通过 norm() 方法获取了向量的模长，经过计算得到二者的 TF 系数

s1 = '你在干嘛呢'
s2 = '你在干什么呢'
print(tf_similarity(s1, s2))
```

## df(词频)-idf(逆文档频率)



Python 代码实现

```python
from sklearn.feature_extraction.text import TfidfVectorizer
import numpy as np
from scipy.linalg import norm

def tfidf_similarity(s1, s2):
    def add_space(s):
        return ' '.join(list(s))
    
    # 将字中间加入空格
    s1, s2 = add_space(s1), add_space(s2)
    # 转化为TF矩阵
    cv = TfidfVectorizer(tokenizer=lambda s: s.split())
    corpus = [s1, s2]
    vectors = cv.fit_transform(corpus).toarray()
    # 计算TF系数
    return np.dot(vectors[0], vectors[1]) / (norm(vectors[0]) * norm(vectors[1]))

s1 = '你在干嘛呢'
s2 = '你在干什么呢'
print(tfidf_similarity(s1, s2))
```

## Word2Vec 计算

将每一个词转换为向量的过程 , 计算两个向量的夹角

Python 

```python
import gensim
import jieba
import numpy as np
from scipy.linalg import norm

model_file = './word2vec/news_12g_baidubaike_20g_novel_90g_embedding_64.bin'
model = gensim.models.KeyedVectors.load_word2vec_format(model_file, binary=True)

def vector_similarity(s1, s2):
    def sentence_vector(s):
        words = jieba.lcut(s)
        v = np.zeros(64)
        for word in words:
            v += model[word]
        v /= len(words)
        return v
    
    v1, v2 = sentence_vector(s1), sentence_vector(s2)
    return np.dot(v1, v2) / (norm(v1) * norm(v2))
  
s1 = '你在干嘛'
s2 = '你正做什么'
vector_similarity(s1, s2)
```

## 使用jieba计算相似

```python
# -*- coding: utf-8 -*-
import jieba
import numpy as np
import re
 
def get_word_vector(s1,s2):
    """
    :param s1: 句子1
    :param s2: 句子2
    :return: 返回句子的余弦相似度
    """
    # 分词
    cut1 = jieba.cut(s1,cut_all=True)
    cut2 = jieba.cut(s2,cut_all=True)
    list_word1 = (','.join(cut1)).split(',')
    list_word2 = (','.join(cut2)).split(',')
    print(list_word1)
    print(list_word2)
 
    # 列出所有的词,取并集
    key_word = list(set(list_word1 + list_word2))
    # 给定形状和类型的用0填充的矩阵存储向量
    word_vector1 = np.zeros(len(key_word))
    word_vector2 = np.zeros(len(key_word))
 
    # 计算词频
    # 依次确定向量的每个位置的值
    for i in range(len(key_word)):
        # 遍历key_word中每个词在句子中的出现次数
        for j in range(len(list_word1)):
            if key_word[i] == list_word1[j]:
                word_vector1[i] += 1
        for k in range(len(list_word2)):
            if key_word[i] == list_word2[k]:
                word_vector2[i] += 1
 
    # 输出向量
    print(word_vector1)
    print(word_vector2)
    return word_vector1, word_vector2
 
def cos_dist(vec1,vec2):
    """
    :param vec1: 向量1
    :param vec2: 向量2
    :return: 返回两个向量的余弦相似度
    """
    dist1=float(np.dot(vec1,vec2)/(np.linalg.norm(vec1)*np.linalg.norm(vec2)))
    return dist1
 
def filter_html(html):
    """
    :param html: html
    :return: 返回去掉html的纯净文本
    """
    dr = re.compile(r'<[^>]+>\，,',re.S)
    dd = dr.sub('',html).strip()
    return dd
 
 
# s1="很高兴见到你"
# s2="我也很高兴见到你"
s1 = '嘉实同舟债券基金产品介绍'
s2 = '先生您好我这边给您看了一下您通过民生银行现在特有的基金叫嘉实同舟债券俟'
vec1,vec2=get_word_vector(s1,s2)
dist1=cos_dist(vec1,vec2)
print(dist1)
```

