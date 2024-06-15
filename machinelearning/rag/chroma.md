Chroma向量数据库

```python
# https://docs.trychroma.com/getting-started
pip install chromadb

import chromadb
basePath = "/dev/chromadbDemo/"
chroma_client = chromadb.PersistentClient(path=basePath + "chromadata")
print("数据库已启动：" + str(chroma_client))

# 创建数据集
# collection = chroma_client.get_or_create_collection(name="collection_default")

#查询数据集
collections = chroma_client.list_collections()
# print("现有数据集：" + str(collections))
```

