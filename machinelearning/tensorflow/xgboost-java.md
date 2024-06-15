

## xgboost4j

```xml
<dependency>
  <groupId>ml.dmlc</groupId>
  <artifactId>xgboost4j</artifactId>
  <version>0.82</version>
</dependency>
<dependency>
  <groupId>biz.k11i</groupId>
  <artifactId>xgboost-predictor</artifactId>
  <version>0.3.0</version>
</dependency>

<!-- 
https://github.com/bwaldvogel/liblinear-java

LIBLINEAR 是一个简单的包，用于解决大规模正则化线性分类、回归和异常值检测。它目前支持
	L2 正则化逻辑回归/L2 损失支持向量分类/L1 损失支持向量分类
	L1 正则化 L2 损失支持向量分类/L1 正则化逻辑回归
	L2 正则化 L2 损失支持向量回归/L1 损失支持向量回归
	一类支持向量机。本文档讲解了 LIBLINEAR 的用法。

您可能感兴趣的三种最重要的编程使用方法是：
	Linear.train(…)
	Linear.predict(…)
	Linear.predictProbability(…)
-->
<dependency>
  <groupId>de.bwaldvogel</groupId>
  <artifactId>liblinear</artifactId>
  <version>1.94</version>
</dependency>
<!-- 
 序列化
-->
<dependency>
  <groupId>com.esotericsoftware.kryo</groupId>
  <artifactId>kryo</artifactId>
  <version>2.21</version>
</dependency>

<!-- 
https://github.com/NLPchina/ansj_seg
Ansj_Seg 使用了基于统计和字典结合的方法进行分词。它包含以下关键技术：
	HMM（ Hidden Markov Model）：通过统计词语出现的概率，对未见过的新词进行预测，提高了新词识别能力。
	动态规划分词算法：这种高效的算法可以快速找到最优解，确保分词的准确性。
	自定义词典：用户可以根据需求添加或修改词典，以适应特定领域的需求。
	N-gram模型：进一步提升长词和专有名词的识别率。

命名实体识别 (NER)
	Ansj_Seg 不仅限于分词，还提供了命名实体识别功能。它能够识别出如人名、地名、组织名等具有特定意义的实体，这对于文本挖掘和信息提取至关重要。

-->
<dependency>
  <groupId>org.ansj</groupId>
  <artifactId>ansj_seg</artifactId>
</dependency>

<!--
官方网站：
https://stanfordnlp.github.io/CoreNLP/
https://github.com/stanfordnlp/CoreNLP

Stanford CoreNLP是由斯坦福大学自然语言处理组开发的一套自然语言处理工具集合，旨在帮助用户进行文本分析和理解。它集成了一系列强大的自然语言处理工具，包括分词、词性标注、命名实体识别、句法分析、情感分析和依存关系分析等功能。Stanford CoreNLP能够处理英语等多种语言的文本数据，并提供丰富的API接口，方便用户进行自然语言处理相关应用的开发和调用。
-->
<dependency>
  <groupId>edu.stanford.nlp</groupId>
  <artifactId>stanford-corenlp</artifactId>
</dependency>

<!--
在Java应用内部直接加载ONNX模型并执行推理，可以使用ONNX Runtime for Java库来实现
-->
<dependency>
  <groupId>com.microsoft.onnxruntime</groupId>
  <artifactId>onnxruntime</artifactId>
  <version>1.7.0</version>
</dependency>

<!--
Java高效矩阵运算库EJML使用
	https://blog.csdn.net/qq_43276566/article/details/131482309
	https://github.com/lessthanoptimal/ejml
-->
<dependency>
  <groupId>org.ejml</groupId>
  <artifactId>ejml-all</artifactId>
  <version>0.40</version>
</dependency>

<!--
Togglz的资料很少，找到了像点人话的简介：如果一个story在当前迭代中无法完成，那样需要给它加上toggle, 这样只需在生产环境将toggle关闭，不为担心未完成的功能被release出去；另一方面，如果发现新的实现、体验不被欢迎，那么只需将toggle关闭就可以快速地返回到旧的实现了。为了满足上面的需求，我们选择了togglz。
参考博客：https://www.jianshu.com/p/2e18aa8a5941
官方文档：https://www.togglz.org/quickstart
-->
<dependency>
  <groupId>org.togglz</groupId>
  <artifactId>togglz-spring-boot-starter</artifactId>
  <version>2.4.1.Final</version>
</dependency>
```

