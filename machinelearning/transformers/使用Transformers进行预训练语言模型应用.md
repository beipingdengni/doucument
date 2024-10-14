 

使用Transformers库（Hugging Face提供）进行预训练语言模型的应用涉及几个步骤：安装库、加载预训练模型、进行文本生成或分类任务。以下是一个详细的示例流程。

#### 安装依赖

首先，确保你安装了Transformers和其他必要的库：

```bash
pip install transformers torch
```

#### 文本生成

以GPT-3（或其他GPT系列模型）为例，演示如何进行文本生成。

```python
from transformers import GPT2LMHeadModel, GPT2Tokenizer

# 加载预训练模型和分词器
model_name = "gpt2"
model = GPT2LMHeadModel.from_pretrained(model_name)
tokenizer = GPT2Tokenizer.from_pretrained(model_name)

# 输入文本
input_text = "Once upon a time"

# 编码输入文本
input_ids = tokenizer.encode(input_text, return_tensors='pt')

# 生成文本
output = model.generate(input_ids, max_length=100, num_return_sequences=1)

# 解码生成的文本
generated_text = tokenizer.decode(output[0], skip_special_tokens=True)
print(generated_text)
```

#### 文本分类

以BERT（或其他BERT系列模型）为例，演示如何进行文本分类。

```python
from transformers import BertTokenizer, BertForSequenceClassification
from transformers import TextClassificationPipeline

# 加载预训练模型和分词器
model_name = "bert-base-uncased"
tokenizer = BertTokenizer.from_pretrained(model_name)
model = BertForSequenceClassification.from_pretrained(model_name, num_labels=2)

# 创建分类管道
pipeline = TextClassificationPipeline(model=model, tokenizer=tokenizer)

# 输入文本
texts = ["I love this movie!", "I hate this movie."]

# 分类
predictions = pipeline(texts)
for text, pred in zip(texts, predictions):
    print(f"Text: {text}\nLabel: {pred['label']}, Score: {pred['score']}\n")
```

#### 文本相似度

使用BERT的句子嵌入进行文本相似度计算。

```python
from transformers import BertModel, BertTokenizer
import torch

# 加载预训练模型和分词器
model_name = "bert-base-uncased"
tokenizer = BertTokenizer.from_pretrained(model_name)
model = BertModel.from_pretrained(model_name)

# 编码文本
texts = ["I love machine learning.", "I enjoy learning about AI."]
encoded_input = tokenizer(texts, padding=True, truncation=True, return_tensors='pt')

# 获取嵌入
with torch.no_grad():
    outputs = model(**encoded_input)
    embeddings = outputs.last_hidden_state.mean(dim=1)

# 计算相似度
cosine_sim = torch.nn.functional.cosine_similarity(embeddings[0], embeddings[1], dim=0)
print(f"Cosine similarity: {cosine_sim.item()}")
```

#### 完整示例

综合以上步骤，以下是完整的代码示例：

```python
import torch
from transformers import GPT2LMHeadModel, GPT2Tokenizer, BertTokenizer, BertForSequenceClassification, BertModel, TextClassificationPipeline

# GPT-2文本生成
def generate_text(input_text, max_length=100):
    model_name = "gpt2"
    model = GPT2LMHeadModel.from_pretrained(model_name)
    tokenizer = GPT2Tokenizer.from_pretrained(model_name)

    input_ids = tokenizer.encode(input_text, return_tensors='pt')
    output = model.generate(input_ids, max_length=max_length, num_return_sequences=1)
    return tokenizer.decode(output[0], skip_special_tokens=True)

# BERT文本分类
def classify_texts(texts):
    model_name = "bert-base-uncased"
    tokenizer = BertTokenizer.from_pretrained(model_name)
    model = BertForSequenceClassification.from_pretrained(model_name, num_labels=2)
    pipeline = TextClassificationPipeline(model=model, tokenizer=tokenizer)

    return pipeline(texts)

# BERT文本相似度
def compute_similarity(text1, text2):
    model_name = "bert-base-uncased"
    tokenizer = BertTokenizer.from_pretrained(model_name)
    model = BertModel.from_pretrained(model_name)

    encoded_input = tokenizer([text1, text2], padding=True, truncation=True, return_tensors='pt')
    with torch.no_grad():
        outputs = model(**encoded_input)
        embeddings = outputs.last_hidden_state.mean(dim=1)

    cosine_sim = torch.nn.functional.cosine_similarity(embeddings[0], embeddings[1], dim=0)
    return cosine_sim.item()

# 示例文本
input_text = "Once upon a time"
texts = ["I love this movie!", "I hate this movie."]
text1 = "I love machine learning."
text2 = "I enjoy learning about AI."

# 生成文本
generated_text = generate_text(input_text)
print(f"Generated Text:\n{generated_text}\n")

# 文本分类
predictions = classify_texts(texts)
for text, pred in zip(texts, predictions):
    print(f"Text: {text}\nLabel: {pred['label']}, Score: {pred['score']}\n")

# 计算文本相似度
similarity = compute_similarity(text1, text2)
print(f"Cosine similarity between \"{text1}\" and \"{text2}\": {similarity}")
```

这个示例展示了如何使用Transformers库进行文本生成、文本分类和文本相似度计算。你可以根据具体需求调整预训练模型和参数。

 

![](https://img-blog.csdnimg.cn/2e298b24aada4f86b975242d6bffa237.jpeg)
