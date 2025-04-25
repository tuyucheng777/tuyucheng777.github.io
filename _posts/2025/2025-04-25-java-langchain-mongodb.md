---
layout: post
title:  使用Langchain4j和MongoDB Atlas在Java中构建AI聊天机器人
category: libraries
copyright: libraries
excerpt: Langchain4j
---

## 1. 概述

聊天机器人系统通过提供快速、智能的响应来增强用户体验，使交互更加高效。

在本教程中，我们将介绍使用Langchain4j和MongoDB Atlas构建聊天机器人的过程。

LangChain4j是一个受[LangChain](https://www.baeldung.com/java-langchain-basics)启发的Java库，旨在帮助使用LLM构建AI应用，我们使用它来开发聊天机器人、摘要引擎或智能搜索系统等应用程序。

**我们将使用[MongoDB Atlas Vector Search](https://www.mongodb.com/docs/atlas/atlas-search/?utm_source=email&utm_campaign=java_influencer_baeldung&utm_medium=influencers)，使我们的聊天机器人能够根据含义(而非仅仅根据关键词)检索相关信息**。传统的基于关键词的搜索方法依赖于精确的词语匹配，当用户以不同的方式表达问题或使用同义词时，通常会导致不相关的结果。

通过使用向量存储和向量搜索，我们的应用将用户查询的含义与存储的内容映射到高维向量空间中进行比较，这使得聊天机器人能够理解并以更高的语境准确度响应复杂的自然语言问题，即使源内容中没有出现确切的词语。因此，我们获得了更具语境感知能力的结果。

## 2. AI聊天机器人应用程序架构

让我们看一下我们的应用程序组件：

![](/assets/images/2025/libraries/javalangchainmongodb01.png)

我们的应用程序使用HTTP端点与聊天机器人进行交互，**它包含两个流程：文档加载流程和聊天机器人流程**。

对于文档加载流程，我们将获取一个文章数据集。然后，我们将使用嵌入模型生成向量嵌入。最后，我们将这些嵌入与我们的原始数据一起保存在MongoDB中，这些嵌入代表了文章的语义内容，从而实现高效的相似性搜索。

对于聊天机器人流程，我们将根据用户输入在MongoDB实例中执行相似度搜索，以检索最相关的文档。之后，我们将使用检索到的文章作为LLM提示的上下文，并根据LLM输出生成聊天机器人的响应。

## 3. 依赖和配置

让我们首先添加[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)依赖，因为我们将构建HTTP API：

```xml
<dependency> 
    <groupId>org.springframework.boot</groupId>         
    <artifactId>spring-boot-starter-web</artifactId> 
    <version>3.3.2</version> 
</dependency>
```

接下来，让我们添加[langchain4j-mongodb-atlas](https://mvnrepository.com/artifact/dev.langchain4j/langchain4j-mongodb-atlas)依赖，它提供与MongoDB向量存储和嵌入模型通信的接口：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-mongodb-atlas</artifactId>
    <version>1.0.0-beta1</version>
</dependency>
```

最后，我们将添加[langchain4j](https://mvnrepository.com/artifact/dev.langchain4j/langchain4j)依赖，这将为我们提供与嵌入模型和LLM交互所需的接口：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j</artifactId>
    <version>1.0.0-beta1</version>
</dependency>
```

为了演示目的，我们将设置[本地MongoDB集群](https://www.mongodb.com/cloud/atlas/register?utm_source=email&utm_campaign=java_influencer_baeldung&utm_medium=influencers)。接下来，我们将获取[OpenAI API密钥](https://platform.openai.com/api-keys)。现在，我们可以在application.properties文件中配置MongoDB URL、数据库名称和OpenAI API密钥：

```properties
app.mongodb.url=mongodb://chatbot:password@localhost:27017/admin
app.mongodb.db-name=chatbot_db
app.openai.apiKey=${OPENAI_API_KEY}
```

现在，让我们创建一个ChatBotConfiguration类，在这里，我们将定义MongoDB客户端Bean以及与嵌入相关的Bean：

```java
@Configuration
public class ChatBotConfiguration {

    @Value("${app.mongodb.url}")
    private String mongodbUrl;

    @Value("${app.mongodb.db-name}")
    private String databaseName;

    @Value("${app.openai.apiKey}")
    private String apiKey;


    @Bean
    public MongoClient mongoClient() {
        return MongoClients.create(mongodbUrl);
    }

    @Bean
    public EmbeddingStore<TextSegment> embeddingStore(MongoClient mongoClient) {
        String collectionName = "embeddings";
        String indexName = "embedding";
        Long maxResultRatio = 10L;
        CreateCollectionOptions createCollectionOptions = new CreateCollectionOptions();
        Bson filter = null;
        IndexMapping indexMapping = IndexMapping.builder()
                .dimension(TEXT_EMBEDDING_3_SMALL.dimension())
                .metadataFieldNames(new HashSet<>())
                .build();
        Boolean createIndex = true;

        return new MongoDbEmbeddingStore(
                mongoClient,
                databaseName,
                collectionName,
                indexName,
                maxResultRatio,
                createCollectionOptions,
                filter,
                indexMapping,
                createIndex
        );
    }

    @Bean
    public EmbeddingModel embeddingModel() {
        return OpenAiEmbeddingModel.builder()
                .apiKey(apiKey)
                .modelName(TEXT_EMBEDDING_3_SMALL)
                .build();
    }
}
```

我们使用OpenAI的text-embedding-3-small模型构建了EmbeddingModel，当然，我们也可以选择其他符合我们需求的嵌入[模型](https://platform.openai.com/docs/guides/embeddings)。然后，我们创建一个MongoDbEmbeddingStore Bean，该存储由MongoDB Atlas集合支持，嵌入信息将保存并索引到该集合中，以便快速进行语义检索。接下来，我们将维度设置为默认的text-embedding-3-small值，**使用EmbeddingModel时，我们需要确保创建的向量的维度与上述模型匹配**。

## 4. 将文档数据加载到向量存储

**我们将使用MongoDB文章作为我们的ChatBot数据**，为了演示目的，我们可以从[Hugging Face](https://huggingface.co/)手动下载包含文章的数据集。接下来，我们将此数据集作为[Articles.json](https://huggingface.co/datasets/MongoDB/devcenter-articles)文件保存在resources文件夹中。 

我们希望在应用程序启动期间提取这些文章，将它们转换为向量嵌入并存储在我们的MongoDB Atlas向量存储中。

现在，让我们将属性添加到application.properties文件，我们将使用它来控制是否需要数据加载：

```properties
app.load-articles=true
```

### 4.1 ArticlesRepository

现在，让我们创建ArticlesRepository，负责读取数据集、生成嵌入并存储它们：

```java
@Component
public class ArticlesRepository {
    private static final Logger log = LoggerFactory.getLogger(ArticlesRepository.class);

    private final EmbeddingStore<TextSegment> embeddingStore;
    private final EmbeddingModel embeddingModel;
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Autowired
    public ArticlesRepository(@Value("${app.load-articles}") Boolean shouldLoadArticles,
                              EmbeddingStore<TextSegment> embeddingStore, EmbeddingModel embeddingModel) throws IOException {
        this.embeddingStore = embeddingStore;
        this.embeddingModel = embeddingModel;

        if (shouldLoadArticles) {
            loadArticles();
        }
    }
}
```

这里我们设置了EmbeddingStore、嵌入模型和一个配置标志，如果app.load-articles设置为true，我们会在启动时触发文档加载。现在，让我们实现loadArticles()方法：

```java
private void loadArticles() throws IOException {
    String resourcePath = "articles.json";
    int maxTokensPerChunk = 8000;
    int overlapTokens = 800;

    List<TextSegment> documents = loadJsonDocuments(resourcePath, maxTokensPerChunk, overlapTokens);

    log.info("Documents to store: " + documents.size());

    for (TextSegment document : documents) {
        Embedding embedding = embeddingModel.embed(document.text()).content();
        embeddingStore.add(embedding, document);
    }

    log.info("Documents are uploaded");
}
```

这里我们使用loadJsonDocuments()方法加载资源文件夹中存储的数据，我们创建一个TextSegment实例集合，对于每个TextSegment，我们创建一个嵌入并将其存储在向量存储中。**我们将使用maxTokensPerChunk变量指定向量存储文档块中的最大标记数**，此值应低于模型维度。此外，**我们使用overlapTokens来指示文本段之间可以重叠的标记数**，这有助于我们保留段之间的上下文。

### 4.2 loadJsonDocuments()实现

接下来，我们来介绍一下loadJsonDocuments()方法，该方法读取原始JSON文章，将其解析为LangChain4j Document对象，并准备嵌入：

```java
private List<TextSegment> loadJsonDocuments(String resourcePath, int maxTokensPerChunk, int overlapTokens) throws IOException {

    InputStream inputStream = ArticlesRepository.class.getClassLoader().getResourceAsStream(resourcePath);

    if (inputStream == null) {
        throw new FileNotFoundException("Resource not found: " + resourcePath);
    }

    BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));

    int batchSize = 500;
    List<Document> batch = new ArrayList<>();
    List<TextSegment> textSegments = new ArrayList<>();

    String line;
    while ((line = reader.readLine()) != null) {
        JsonNode jsonNode = objectMapper.readTree(line);

        String title = jsonNode.path("title").asText(null);
        String body = jsonNode.path("body").asText(null);
        JsonNode metadataNode = jsonNode.path("metadata");

        if (body != null) {
            addDocumentToBatch(title, body, metadataNode, batch);

            if (batch.size() >= batchSize) {
                textSegments.addAll(splitIntoChunks(batch, maxTokensPerChunk, overlapTokens));
                batch.clear();
            }
        }
    }

    if (!batch.isEmpty()) {
        textSegments.addAll(splitIntoChunks(batch, maxTokensPerChunk, overlapTokens));
    }

    return textSegments;
}
```

在这里，我们解析JSON文件并迭代每个条目。然后，我们将文章标题、正文和元数据作为文档添加到批处理中。当批处理达到500个条目(或达到末尾)时，我们使用splitIntoChunks()进行处理，将内容拆分为可管理的文本段。该方法返回一个完整的TextSegment对象列表，可供嵌入和存储。

让我们实现addDocumentToBatch()方法：

```java
private void addDocumentToBatch(String title, String body, JsonNode metadataNode, List<Document> batch) {
    String text = (title != null ? title + "\n\n" + body : body);

    Metadata metadata = new Metadata();
    if (metadataNode != null && metadataNode.isObject()) {
        Iterator<String> fieldNames = metadataNode.fieldNames();
        while (fieldNames.hasNext()) {
            String fieldName = fieldNames.next();
            metadata.put(fieldName, metadataNode.path(fieldName).asText());
        }
    }

    Document document = Document.from(text, metadata);
    batch.add(document);
}
```

文章的标题和正文会被拼接成一个文本块，如果包含元数据，我们会解析字段并将其添加到Metadata对象中。合并后的文本和元数据会被包装在一个Document对象中，我们会将其添加到当前批次中。

### 4.3 splitIntoChunks()实现及获取上传结果

一旦我们将文章组装成包含内容和元数据的Document对象，下一步就是将它们拆分成更小的、可识别token的块，这些块要与嵌入模型的限制兼容。最后，让我们看看splitIntoChunks()是什么样子的：

```java
private List<TextSegment> splitIntoChunks(List<Document> documents, int maxTokensPerChunk, int overlapTokens) {
    OpenAiTokenizer tokenizer = new OpenAiTokenizer(OpenAiEmbeddingModelName.TEXT_EMBEDDING_3_SMALL);

    DocumentSplitter splitter = DocumentSplitters.recursive(
            maxTokensPerChunk,
            overlapTokens,
            tokenizer
    );

    List<TextSegment> allSegments = new ArrayList<>();
    for (Document document : documents) {
        List<TextSegment> segments = splitter.split(document);
        allSegments.addAll(segments);
    }

    return allSegments;
}
```

首先，我们初始化一个与OpenAI的text-embedding-3-small模型兼容的分词器。然后，我们使用DocumentSplitter将文档拆分成多个块，同时保留相邻块之间的重叠部分。每个Document经过处理后，拆分成多个TextSegment实例，然后返回到我们的向量存储(MongoDB)中进行嵌入。在启动过程中，我们应该看到以下日志：

```text
Documents to store: 328

Stored embeddings
```

此外，如果我们使用[MongoDB Compass](https://www.mongodb.com/products/tools/compass?utm_source=email&utm_campaign=java_influencer_baeldung&utm_medium=influencers)查看MongoDB中存储的内容，我们将看到所有生成了嵌入的文档内容：

![](/assets/images/2025/libraries/javalangchainmongodb02.png)

这个过程非常重要，因为大多数嵌入模型都有标记限制，这意味着一次只能将一定量的数据嵌入到向量中。分块使我们能够遵守这些限制，而重叠部分则有助于我们保持段之间的连续性，这对于基于段落的内容尤其重要。

**我们仅使用了整个数据集的一部分进行演示**，上传整个数据集可能需要一些时间，并且需要更多积分。

## 5. 聊天机器人API

现在，让我们实现聊天机器人API流程(我们的聊天机器人接口)。在这里，**我们将创建一些Bean，用于从向量存储中检索文档并与LLM通信，从而创建上下文感知的响应。最后，我们将构建聊天机器人API并验证其工作原理**。

### 5.1 ArticleBasedAssistant实现

我们首先创建ContentRetriever Bean，使用用户的输入在MongoDB Atlas中执行向量搜索：

```java
@Bean
public ContentRetriever contentRetriever(EmbeddingStore<TextSegment> embeddingStore, EmbeddingModel embeddingModel) {
    return EmbeddingStoreContentRetriever.builder()
        .embeddingStore(embeddingStore)
        .embeddingModel(embeddingModel)
        .maxResults(10)
        .minScore(0.8)
        .build();
}
```

此检索器使用嵌入模型对用户查询进行编码，并将其与已存储的文章嵌入进行比较。**此外，我们还指定了返回结果项的最大数量以及分数，这将控制响应与请求的匹配程度**。

接下来，让我们创建一个ChatLanguageModel Bean，它将根据检索到的内容生成响应：

```java
@Bean
public ChatLanguageModel chatModel() {
    return OpenAiChatModel.builder()
        .apiKey(apiKey)
        .modelName("gpt-4o-mini")
        .build();
}
```

在这个Bean中，我们使用了gpt-4o-mini模型，但始终可以选择另一个更符合我们要求的[模型](https://platform.openai.com/docs/models)。

现在，我们将创建一个ArticleBasedAssistant接口。在这里，我们将定义answer()方法，该方法接收文本请求并返回文本响应：

```java
public interface ArticleBasedAssistant {
    String answer(String question);
}
```

LangChain4j通过结合已配置的语言模型和内容检索器来动态实现此接口，接下来，让我们为我们的助手接口创建一个Bean：

```java
@Bean
public ArticleBasedAssistant articleBasedAssistant(ChatLanguageModel chatModel, ContentRetriever contentRetriever) {
    return AiServices.builder(ArticleBasedAssistant.class)
        .chatLanguageModel(chatModel)
        .contentRetriever(contentRetriever)
        .build();
}
```

如此设置意味着我们现在可以调用assistant.answer("...")，并在底层嵌入查询，并从向量存储中获取相关文档。这些文档将用作LLM提示的上下文，并生成并返回自然语言答案。

### 5.2 ChatBotController实现及测试结果

最后，让我们创建ChatBotController，它将GET请求映射到聊天机器人逻辑：

```java
@RestController
public class ChatBotController {
    private final ArticleBasedAssistant assistant;

    @Autowired
    public ChatBotController(ArticleBasedAssistant assistant) {
        this.assistant = assistant;
    }

    @GetMapping("/chat-bot")
    public String answer(@RequestParam("question") String question) {
        return assistant.answer(question);
    }
}
```

在这里，我们实现了聊天机器人端点，并将其与ArticleBasedAssistant集成。此端点通过question请求参数接受用户查询，将其委托给ArticleBasedAssistant，并以纯文本形式返回生成的响应。

让我们调用我们的聊天机器人API并看看它响应什么：

```java
@AutoConfigureMockMvc
@SpringBootTest(classes = {ChatBotConfiguration.class, ArticlesRepository.class, ChatBotController.class})
class ChatBotLiveTest {

    Logger log = LoggerFactory.getLogger(ChatBotLiveTest.class);

    @Autowired
    private MockMvc mockMvc;

    @Test
    void givenChatBotApi_whenCallingGetEndpointWithQuestion_thenExpectedAnswersIsPresent() throws Exception {
        String chatResponse = mockMvc
                .perform(get("/chat-bot")
                        .param("question", "Steps to implement Spring boot app and MongoDB"))
                .andReturn()
                .getResponse()
                .getContentAsString();

        log.info(chatResponse);
        Assertions.assertTrue(chatResponse.contains("Step 1"));
    }
}
```

在我们的测试中，我们调用了聊天机器人端点，并要求它提供创建集成MongoDB的Spring Boot应用程序的步骤。之后，我们检索并记录了预期结果，完整的响应也显示在日志中：

```text
To implement a MongoDB Spring Boot Java Book Tracker application, follow these steps. This guide will help you set up a simple CRUD application to manage books, where you can add, edit, and delete book records stored in a MongoDB database.

### Step 1: Set Up Your Environment

1. **Install Java Development Kit (JDK)**:
   Make sure you have JDK (Java Development Kit) installed on your machine. You can download it from the [Oracle website](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html) or use OpenJDK.

2. **Install MongoDB**:
   Download and install MongoDB from the [MongoDB official website](https://www.mongodb.com/try/download/community). Follow the installation instructions specific to your operating system.

//shortened
```

## 6. 总结

在本文中，我们使用Langchain4j和MongoDB Atlas实现了聊天机器人Web应用程序。使用我们的应用程序，我们可以与聊天机器人交互，从下载的文章中获取信息。为了进一步改进，我们可以添加查询预处理功能并处理模糊查询，此外，我们还可以轻松扩展聊天机器人用于回答问题的数据集。