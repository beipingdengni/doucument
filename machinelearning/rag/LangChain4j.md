## langchain4j

官方网站：https://docs.langchain4j.dev/

### maven openai

```xml
<!-- openai 对应API -->
<!-- 
String apiKey = System.getenv("OPENAI_API_KEY");
测试可以使用：String apiKey = "demo";
调用 
OpenAiChatModel model = OpenAiChatModel.withApiKey(apiKey);
String answer = model.generate("Say 'Hello World'");
System.out.println(answer); // Hello World
-->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>0.31.0</version>
</dependency>
```

### maven llm

```xml
<!-- 个性定义llm -->
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>0.31.0</version>
</dependency>
```

### rag

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-easy-rag</artifactId>
    <version>0.31.0</version>
</dependency>
```

加载文档

```java
List<Document> documents = FileSystemDocumentLoader.loadDocuments("/home/langchain4j/documentation");
// List<Document> documents = FileSystemDocumentLoader.loadDocumentsRecursively("/home/langchain4j", new TextDocumentParser()); // langchain4j-document-parser-apache-tika 使这个加载多种格式的文档
// 
interface Assistant {
    String chat(String userMessage);
}
// 对话
Assistant assistant = AiServices.builder(Assistant.class)
    .chatLanguageModel(OpenAiChatModel.withApiKey(OPENAI_API_KEY))
    .chatMemory(MessageWindowChatMemory.withMaxMessages(10)) // 内存存储
    .contentRetriever(EmbeddingStoreContentRetriever.from(embeddingStore))
    .build();

String answer = assistant.chat("How to do Easy RAG with LangChain4j?");
```

