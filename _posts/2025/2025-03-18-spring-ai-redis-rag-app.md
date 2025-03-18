---
layout: post
title:  使用Redis和Spring AI创建RAG应用程序
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

在本教程中，我们将使用[Spring AI框架](https://www.baeldung.com/spring-ai)和[RAG(检索增强生成)](https://www.baeldung.com/cs/retrieval-augmented-generation)技术构建一个ChatBot。借助Spring AI，我们将与[Redis Vector数据库](https://redis.io/docs/latest/develop/get-started/vector-database/)集成以存储和检索数据，以增强[LLM(大语言模型)](https://www.baeldung.com/cs/large-language-models)的提示。一旦LLM收到包含相关数据的提示，它就会有效地生成包含最新数据的自然语言响应来响应用户查询。

## 2. RAG是什么？

LLM是使用来自互联网的大量数据集进行预训练的[机器学习](https://www.baeldung.com/cs/ml-fundamentals)模型，要使LLM在私营企业中发挥作用，我们必须使用特定于组织的知识库对其进行微调。然而，微调通常是一个耗时的过程，需要大量的计算资源。此外，微调后的LLM很有可能对查询产生不相关或误导性的响应；这种行为通常被称为LLM幻觉。

在这种情况下，**RAG是一种限制或情境化LLM响应的绝佳技术**。[向量数据库](https://www.baeldung.com/cs/vector-databases)在RAG架构中起着重要作用，可以为LLM提供上下文信息。但是，在应用程序可以在RAG架构中使用它之前，必须先进行ETL(提取、转换和加载)过程来填充它：

![](/assets/images/2025/springai/springairedisragapp01.png)

**读取器从不同来源检索组织的知识库文档；然后，转换器将检索到的文档拆分成更小的块，并使用嵌入模型对内容进行向量化**。最后，写入器将向量或嵌入加载到向量数据库中。向量数据库是可以将这些嵌入存储在多维空间中的专用数据库。

在RAG中，如果向量数据库定期从组织的知识库更新，LLM就可以响应几乎实时的数据。

一旦向量数据库准备好数据，应用程序就可以使用它来检索用户查询的上下文数据：

![](/assets/images/2025/springai/springairedisragapp02.png)

应用程序将用户查询与向量数据库中的上下文数据相结合形成提示，并最终将其发送给LLM，**LLM在上下文数据的边界内以自然语言生成响应并将其发送回应用程序**。

## 3. 使用Spring AI和Redis实现RAG

Redis栈提供向量搜索服务，我们将使用Spring AI框架与其集成并构建基于RAG的ChatBot应用程序。此外，我们将使用OpenAI的GPT-3.5 TurboLLM模型来生成最终响应。

### 3.1 先决条件

对于ChatBot服务，为了认证OpenAI服务，我们需要API Key。在创建[OpenAI帐户](https://platform.openai.com/)后，我们将创建一个：

![](/assets/images/2025/springai/springairedisragapp03.png)

我们还将创建一个[Redis Cloud](https://app.redislabs.com/)帐户来访问免费的Redis Vector DB：

![](/assets/images/2025/springai/springairedisragapp04.png)

为了与Redis Vector DB和OpenAI服务集成，我们将使用Spring AI库更新[Maven依赖](https://repo.spring.io/ui/native/milestone/org/springframework/ai/spring-ai-bom/1.0.0-M1/)：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    <version>1.0.0-M1</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-transformers-spring-boot-starter</artifactId>
    <version>1.0.0-M1</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-redis-spring-boot-starter</artifactId>
    <version>1.0.0-M1</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-pdf-document-reader</artifactId>
    <version>1.0.0-M1</version>
</dependency>
```

### 3.2 将数据加载到Redis中的关键类

在Spring Boot应用程序中，**我们将创建用于从Redis Vector DB加载和检索数据的组件**。例如，我们将员工手册PDF文档加载到Redis DB中。

现在，我们来看看涉及的类：

![](/assets/images/2025/springai/springairedisragapp05.png)

[DocumentReader](https://docs.spring.io/spring-ai/docs/current/api/org/springframework/ai/document/DocumentReader.html)是用于读取文档的Spring AI接口，我们将使用开箱即用的[PagePdfDocumentReader](https://docs.spring.io/spring-ai/docs/current/api/org/springframework/ai/reader/pdf/PagePdfDocumentReader.html)实现DocumentReader。同样，[DocumentWriter](https://docs.spring.io/spring-ai/docs/current/api/org/springframework/ai/document/DocumentWriter.html)和[VectorStore](https://docs.spring.io/spring-ai/docs/current/api/org/springframework/ai/vectorstore/VectorStore.html)是用于将数据写入存储系统的接口，[RedisVectorStore](https://docs.spring.io/spring-ai/docs/current/api/org/springframework/ai/vectorstore/RedisVectorStore.html)是VectorStore的众多开箱即用实现之一，我们将使用它在Redis Vector DB中加载和搜索数据。我们将使用迄今为止讨论过的Spring AI框架类编写DataLoaderService。

### 3.3 实现数据加载器服务

让我们了解一下DataLoaderService类中的load()方法：

```java
@Service
public class DataLoaderService {
    private static final Logger logger = LoggerFactory.getLogger(DataLoaderService.class);

    @Value("classpath:/data/Employee_Handbook.pdf")
    private Resource pdfResource;

    @Autowired
    private VectorStore vectorStore;

    public void load() {
        PagePdfDocumentReader pdfReader = new PagePdfDocumentReader(this.pdfResource,
                PdfDocumentReaderConfig.builder()
                        .withPageExtractedTextFormatter(ExtractedTextFormatter.builder()
                                .withNumberOfBottomTextLinesToDelete(3)
                                .withNumberOfTopPagesToSkipBeforeDelete(1)
                                .build())
                        .withPagesPerDocument(1)
                        .build());

        var tokenTextSplitter = new TokenTextSplitter();
        this.vectorStore.accept(tokenTextSplitter.apply(pdfReader.get()));
    }
}
```

load()方法使用PagePdfDocumentReader类读取PDF文件并将其加载到Redis Vector DB，**Spring AI框架使用命名空间spring.ai.vectorstore中的[配置属性](https://docs.spring.io/spring-ai/reference/api/vectordbs/redis.html#_configuration_properties)自动配置VectorStore接口**：

```yaml
spring:
    ai:
        vectorstore:
            redis:
                uri: redis://:PQzkkZLOgOXXX@redis-19438.c330.asia-south1-1.gce.redns.redis-cloud.com:19438
                index: faqs
                prefix: "faq:"
                initialize-schema: true
```

该框架将RedisVectorStore对象(VectorStore接口的实现)注入到DataLoaderService中。

[TokenTextSplitter](https://docs.spring.io/spring-ai/docs/current/api/org/springframework/ai/transformer/splitter/TokenTextSplitter.html)类分割文档，最后VectorStore类将块加载到Redis Vector DB中。

### 3.4 生成最终响应的关键类

一旦Redis Vector DB准备就绪，我们就可以检索与用户查询相关的上下文信息。之后，此上下文用于形成LLM的提示以生成最终响应。让我们看看关键的类：

![](/assets/images/2025/springai/springairedisragapp06.png)

DataRetrievalService类中的searchData()方法接收查询，然后从VectorStore检索上下文数据。ChatBotService使用此数据通过PromptTemplate类形成提示，然后将其发送到OpenAI服务。Spring Boot框架从application.yml文件中读取与OpenAI相关的相关属性，然后自动配置OpenAIChatModel对象。在本文中，我们将Spring的Profile设置为“airag”。

让我们直接进入实现部分来详细了解。

### 3.5 实现聊天机器人服务

我们来看看ChatBotService类：

```java
@Service
public class ChatBotService {
    @Qualifier("openAiChatModel")
    @Autowired
    private ChatModel chatClient;
    @Autowired
    private DataRetrievalService dataRetrievalService;

    private final String PROMPT_BLUEPRINT = """
              Answer the query strictly referring the provided context:
              {context}
              Query:
              {query}
              In case you don't have any answer from the context provided, just say:
              I'm sorry I don't have the information you are looking for.
            """;

    public String chat(String query) {
        return chatClient.call(createPrompt(query, dataRetrievalService.searchData(query)));
    }

    private String createPrompt(String query, List<Document> context) {
        PromptTemplate promptTemplate = new PromptTemplate(PROMPT_BLUEPRINT);
        promptTemplate.add("query", query);
        promptTemplate.add("context", context);
        return promptTemplate.render();
    }
}
```

Spring AI框架使用命名空间spring.ai.openai中的[OpenAI](https://docs.spring.io/spring-ai/reference/api/chat/openai-chat.html)配置属性创建[ChatModel](https://docs.spring.io/spring-ai/reference/api/chatmodel.html) Bean：

```yaml
spring:
    ai:
        vectorstore:
            redis:
            # Redis vector store related properties...
        openai:
            temperature: 0.3
            api-key: ${SPRING_AI_OPENAI_API_KEY}
            model: gpt-3.5-turbo
            #embedding-base-url: https://api.openai.com
            #embedding-api-key: ${SPRING_AI_OPENAI_API_KEY}
            #embedding-model: text-embedding-ada-002
```

该框架还可以从环境变量SPRING_AI_OPENAI_API_KEY中读取API Key，这是一个非常安全的选项。我们可以启用以文本嵌入开头的Key来创建OpenAiEmbeddingModel Bean，该Bean用于从知识库文档中创建向量嵌入。

**OpenAI服务的提示必须明确，因此，我们在提示蓝图PROMPT_BLUEPRINT中严格指示仅从上下文信息中形成响应**。

在chat()方法中，我们检索与Redis Vector DB中的查询匹配的文档。然后，我们使用这些文档和用户查询在createPrompt()方法中生成提示。最后，我们调用ChatModel类的call()方法来接收来自OpenAI服务的响应。

现在，让我们通过向之前加载到Redis Vector DB中的员工手册询问一个问题来检查聊天机器人服务的实际运行情况：

```java
@Test
void whenQueryAskedWithinContext_thenAnswerFromTheContext() {
    String response = chatBotService.chat("How are employees supposed to dress?");
    assertNotNull(response);
    logger.info("Response from LLM: {}", response);
}
```

然后，我们将看到输出：

```text
Response from LLM: Employees are supposed to dress appropriately for their individual work responsibilities and position.
```

输出与加载到Redis Vector DB中的员工手册PDF文档一致。

让我们看看如果我们询问员工手册中没有的内容会发生什么：

```java
@Test
void whenQueryAskedOutOfContext_thenDontAnswer() {
    String response = chatBotService.chat("What should employees eat?");
    assertEquals("I'm sorry I don't have the information you are looking for.", response);
    logger.info("Response from the LLM: {}", response);
}
```

以下是最终的输出：

```text
Response from the LLM: I'm sorry I don't have the information you are looking for.
```

LLM在提供的上下文中找不到任何内容，因此无法回答查询。

## 4. 总结

在本文中，我们讨论了使用Spring AI框架实现基于RAG架构的应用程序。使用上下文信息形成提示对于从LLM生成正确的响应至关重要，因此，Redis Vector DB是存储和对文档向量执行相似性搜索的绝佳解决方案。**此外，对文档进行分块对于获取正确的记录和限制提示Token的成本同样重要**。