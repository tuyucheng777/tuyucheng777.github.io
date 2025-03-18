---
layout: post
title:  使用MongoDB和Spring AI构建RAG应用程序
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

人工智能技术的使用正在成为现代开发的一项关键技能，**在本文中，我们将构建一个可以根据存储的文档回答问题的RAG Wiki应用程序**。

我们将使用Spring AI将我们的应用程序与[MongoDB Vector数据库](https://www.mongodb.com/products/platform/atlas-vector-search)和LLM集成。

## 2. RAG应用

当自然语言生成需要依赖上下文数据时，我们会使用[检索增强生成(RAG)](https://www.baeldung.com/cs/retrieval-augmented-generation)应用程序。**RAG应用程序的一个关键组件是[向量数据库](https://www.baeldung.com/cs/vector-databases)，它在有效管理和检索这些数据方面发挥着至关重要的作用**：

![](/assets/images/2025/springai/springaimongodbrag01.png)

我们使用嵌入模型来处理源文档，嵌入模型将文档中的文本转换为高维向量。**这些向量捕获内容的语义含义，使我们能够根据上下文(而不仅仅是关键字匹配)比较和检索相似内容，然后我们将文档存储在向量存储中**。

保存文档后，我们可以通过以下方式根据它们发送提示：

- 首先，我们使用嵌入模型来处理问题，将其转换为捕捉其语义含义的向量。
- 接下来，我们执行相似性搜索，将问题的向量与存储在向量存储中的文档的向量进行比较。
- 从最相关的文档中，我们为问题构建了一个上下文。
- **最后，我们将问题及其上下文发送给[LLM](https://www.baeldung.com/cs/large-language-models)，LLM会构建与查询相关并通过提供的上下文进行丰富的响应**。

## 3. MongoDB Atlas向量搜索

在本教程中，我们将使用[MongoDB Atlas Search](https://www.baeldung.com/mongodb-spring-data-atlas-search)作为向量存储，它提供的[向量搜索](https://www.mongodb.com/products/platform/atlas-vector-search)功能可以满足我们在此项目中的需求。为了设置MongoDB Atlas Search的本地实例以进行测试，我们将使用mongodb-atlas-local [Docker容器](https://www.baeldung.com/ops/docker-guide)，让我们创建一个docker-compose.yml文件：

```yaml
version: '3.1'

services:
    my-mongodb:
        image: mongodb/mongodb-atlas-local:7.0.9
        container_name: my-mongodb
        environment:
            - MONGODB_INITDB_ROOT_USERNAME=wikiuser
            - MONGODB_INITDB_ROOT_PASSWORD=password
        ports:
            - 27017:27017
```

## 4. 依赖和配置

让我们首先添加必要的依赖，由于我们的应用程序将提供HTTP API，因此我们将包含[spring-boot-starter-web依赖](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId> 
    <artifactId>spring-boot-starter-web</artifactId>
    <version>LATEST_VERSION</version>
</dependency>
```

此外，我们将使用[OpenAI API客户端](https://www.baeldung.com/java-openai-api-client)连接到LLM，因此我们也添加它的[依赖](https://mvnrepository.com/artifact/group.springframework.ai/spring-ai-openai-spring-boot-starter)：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    <version>LATEST_VERSION</version>
</dependency>
```

最后，我们将添加[MongoDB Atlas Store依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-mongodb-atlas-store-spring-boot-starter)：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mongodb-atlas-store-spring-boot-starter</artifactId>
    <version>LATEST_VERSION</version>
</dependency>
```

现在，让我们为应用程序添加配置属性：

```yaml
spring:
    data:
        mongodb:
            uri: mongodb://wikiuser:password@localhost:27017/admin
            database: wiki
    ai:
        vectorstore:
            mongodb:
                collection-name: vector_store
                initialize-schema: true
                path-name: embedding
                indexName: vector_index
        openai:
            api-key: ${OPENAI_API_KEY}
            chat:
                options:
                    model: gpt-3.5-turbo
```

我们指定了MongoDB URL和数据库，还通过设置集合名称、嵌入字段名称和向量索引名称配置了向量存储。**借助initialize-schema属性，所有这些工件都将由Spring AI框架自动创建**。

最后，我们添加了[OpenAI API Key](https://platform.openai.com/api-keys)和[模型版本](https://platform.openai.com/docs/models)。

## 5. 将文档保存到向量存储

现在，我们添加将数据保存到向量存储的过程。我们的应用程序将负责根据现有文档为用户问题提供答案-本质上是一种Wiki。

让我们添加一个模型，将文件内容与文件路径一起存储：

```java
public class WikiDocument {
    private String filePath;
    private String content;

    // standard getters and setters
}
```

下一步，我们将添加WikiDocumentsRepository。在这个Repository中，我们封装了所有持久层逻辑：

```java
import org.springframework.ai.document.Document;
import org.springframework.ai.transformer.splitter.TokenTextSplitter;

@Component
public class WikiDocumentsRepository {
    private final VectorStore vectorStore;

    public WikiDocumentsRepository(VectorStore vectorStore) {
        this.vectorStore = vectorStore;
    }

    public void saveWikiDocument(WikiDocument wikiDocument) {

        Map<String, Object> metadata = new HashMap<>();
        metadata.put("filePath", wikiDocument.getFilePath());
        Document document = new Document(wikiDocument.getContent(), metadata);
        List<Document> documents = new TokenTextSplitter().apply(List.of(document));

        vectorStore.add(documents);
    }
}
```

这里我们注入了[VectorStore](https://docs.spring.io/spring-ai/reference/api/vectordbs.html)接口Bean，它将由spring-ai-mongodb-atlas-store-spring-boot-starter提供的MongoDBAtlasVectorStore实现。在saveWikiDocument方法中，我们创建一个Document实例并用内容和元数据填充它。

**然后我们使用TokenTextSplitter将文档分成更小的块并将它们保存在我们的向量存储中**，现在让我们创建一个WikiDocumentsServiceImpl：

```java
@Service
public class WikiDocumentsServiceImpl {
    private final WikiDocumentsRepository wikiDocumentsRepository;

    // constructors

    public void saveWikiDocument(String filePath) {
        try {
            String content = Files.readString(Path.of(filePath));
            WikiDocument wikiDocument = new WikiDocument();
            wikiDocument.setFilePath(filePath);
            wikiDocument.setContent(content);

            wikiDocumentsRepository.saveWikiDocument(wikiDocument);
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}
```

在Service层，我们检索文件内容，创建WikiDocument实例，并将其发送到Repository进行持久保存。

在控制器中，我们只需将文件路径传递给Service层，如果文档保存成功，则返回201状态码：

```java
@RestController
@RequestMapping("wiki")
public class WikiDocumentsController {
    private final WikiDocumentsServiceImpl wikiDocumentsService;

    // constructors

    @PostMapping
    public ResponseEntity<Void> saveDocument(@RequestParam String filePath) {
        wikiDocumentsService.saveWikiDocument(filePath);

        return ResponseEntity.status(201).build();
    }
}
```

**我们应该注意此端点的安全性方面，存在一个潜在的漏洞，用户可以使用此端点上传意外文件，例如配置或系统文件。作为解决方案，我们可以限制可以上传文件的目录**。现在，让我们启动我们的应用程序并查看我们的流程如何工作。让我们添加Spring Boot Test依赖项，这将允许我们设置测试Web上下文：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>LATEST_VERSION</version>
</dependency>
```

现在，我们将启动测试应用程序实例并调用两个文档的POST端点：

```java
@AutoConfigureMockMvc
@ExtendWith(SpringExtension.class)
@SpringBootTest
class RAGMongoDBApplicationManualTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void givenMongoDBVectorStore_whenCallingPostDocumentEndpoint_thenExpectedResponseCodeShouldBeReturned() throws Exception {
        mockMvc.perform(post("/wiki?filePath={filePath}",
                        "src/test/resources/documentation/owl-documentation.md"))
                .andExpect(status().isCreated());

        mockMvc.perform(post("/wiki?filePath={filePath}",
                        "src/test/resources/documentation/rag-documentation.md"))
                .andExpect(status().isCreated());
    }
}
```

两个调用都应返回201状态码，因此文档已添加。我们可以使用[MongoDB Compass](https://www.mongodb.com/docs/compass/current/install/)来确认文档已成功保存到向量存储中：

![](/assets/images/2025/springai/springaimongodbrag02.png)

**我们可以看到，两个文档都已保存。我们可以看到原始内容以及嵌入数组**。

## 6. 相似性搜索

让我们添加相似性搜索功能，我们将在Repository中包含一个findSimilarDocuments方法：

```java
@Component
public class WikiDocumentsRepository {
    private final VectorStore vectorStore;

    public List<WikiDocument> findSimilarDocuments(String searchText) {
        return vectorStore
                .similaritySearch(SearchRequest
                        .query(searchText)
                        .withSimilarityThreshold(0.87)
                        .withTopK(10))
                .stream()
                .map(document -> {
                    WikiDocument wikiDocument = new WikiDocument();
                    wikiDocument.setFilePath((String) document.getMetadata().get("filePath"));
                    wikiDocument.setContent(document.getContent());

                    return wikiDocument;
                })
                .toList();
    }
}
```

我们从VectorStore调用了[similaritySearch](https://docs.spring.io/spring-ai/reference/api/vectordbs/mongodb.html#_performing_similarity_search)方法，除了搜索文本之外，我们还指定了结果限制和相似度阈值。**相似度阈值参数允许我们控制文档内容与搜索文本的匹配程度**。

在Service层，我们将代理对Repository的调用：

```java
public List<WikiDocument> findSimilarDocuments(String searchText) {
    return wikiDocumentsRepository.findSimilarDocuments(searchText);
}
```

在控制器中，让我们添加一个GET端点，接收搜索文本作为参数并将其传递给Service：

```java
@RestController
@RequestMapping("/wiki")
public class WikiDocumentsController {
    @GetMapping
    public List<WikiDocument> get(@RequestParam("searchText") String searchText) {
        return wikiDocumentsService.findSimilarDocuments(searchText);
    }
}
```

现在让我们调用新的端点并看看相似性搜索如何工作：

```java
@Test
void givenMongoDBVectorStoreWithDocuments_whenMakingSimilaritySearch_thenExpectedDocumentShouldBePresent() throws Exception {
    String responseContent = mockMvc.perform(get("/wiki?searchText={searchText}", "RAG Application"))
        .andExpect(status().isOk())
        .andReturn()
        .getResponse()
        .getContentAsString();

    assertThat(responseContent)
        .contains("RAGAIApplication is responsible for storing the documentation");
}
```

我们调用了端点，搜索文本与文档中不完全匹配。**但是，我们仍然检索了具有相似内容的文档，并确认其中包含我们存储在rag-documentation.md文件中的文本**。

## 7. 提示端点

让我们开始构建提示流程，这是我们应用程序的核心功能。我们将从AdvisorConfiguration开始：

```java
@Configuration
public class AdvisorConfiguration {

    @Bean
    public QuestionAnswerAdvisor questionAnswerAdvisor(VectorStore vectorStore) {
        return new QuestionAnswerAdvisor(vectorStore, SearchRequest.defaults());
    }
}
```

我们创建了一个QuestionAnswerAdvisor Bean，负责构建提示请求，包括初始问题。**此外，它将附加向量存储的相似性搜索响应作为问题的上下文**。现在，让我们将搜索端点添加到我们的API：

```java
@RestController
@RequestMapping("/wiki")
public class WikiDocumentsController {
    private final WikiDocumentsServiceImpl wikiDocumentsService;
    private final ChatClient chatClient;
    private final QuestionAnswerAdvisor questionAnswerAdvisor;

    public WikiDocumentsController(WikiDocumentsServiceImpl wikiDocumentsService,
                                   @Qualifier("openAiChatModel") ChatModel chatModel,
                                   QuestionAnswerAdvisor questionAnswerAdvisor) {
        this.wikiDocumentsService = wikiDocumentsService;
        this.questionAnswerAdvisor = questionAnswerAdvisor;
        this.chatClient = ChatClient.builder(chatModel).build();
    }

    @GetMapping("/search")
    public String getWikiAnswer(@RequestParam("question") String question) {
        return chatClient.prompt()
                .user(question)
                .advisors(questionAnswerAdvisor)
                .call()
                .content();
    }
}
```

在这里，我们通过将用户的输入添加到提示并附加我们的QuestionAnswerAdvisor来构建提示请求。

最后，让我们调用我们的端点，看看它告诉我们有关RAG应用程序的信息：

```java
@Test
void givenMongoDBVectorStoreWithDocumentsAndLLMClient_whenAskQuestionAboutRAG_thenExpectedResponseShouldBeReturned() throws Exception {
    String responseContent = mockMvc.perform(get("/wiki/search?question={question}", "Explain the RAG Applications"))
        .andExpect(status().isOk())
        .andReturn()
        .getResponse()
        .getContentAsString();

    logger.atInfo().log(responseContent);

    assertThat(responseContent).isNotEmpty();
}
```

我们向端点发送了问题“Explain the RAG applications”并记录了API响应：

```text
b.s.r.m.RAGMongoDBApplicationManualTest : I'm sorry, but the economic theory is not directly related to the information provided about owls and the RAG AI Application.
If you have a specific question about economic theory, please feel free to ask.
```

可以看到，**端点根据我们之前保存在向量数据库中的文档文件返回了有关RAG应用程序的信息**。

现在让我们尝试询问一些我们知识库中肯定没有的问题：

```java
@Test
void givenMongoDBVectorStoreWithDocumentsAndLLMClient_whenAskUnknownQuestion_thenExpectedResponseShouldBeReturned() throws Exception {
    String responseContent = mockMvc.perform(get("/wiki/search?question={question}", "Explain the Economic theory"))
        .andExpect(status().isOk())
        .andReturn()
        .getResponse()
        .getContentAsString();

    logger.atInfo().log(responseContent);

    assertThat(responseContent).isNotEmpty();
}
```

现在我们询问了经济理论，以下是答案：

```text
b.s.r.m.RAGMongoDBApplicationManualTest : I'm sorry, but the economic theory is not directly related to the information provided about owls and the RAG AI Application.
If you have a specific question about economic theory, please feel free to ask.
```

这次，我们的应用程序没有找到任何相关文档，**也没有使用任何其他来源来提供答案**。

## 8. 总结

在本文中，我们成功使用Spring AI框架实现了一个RAG应用程序，该框架是集成各种AI技术的绝佳工具。此外，MongoDB被证明是处理向量存储的强大选择。

**通过这种强大的组合，我们可以构建用于各种目的的基于现代人工智能的应用程序，包括聊天机器人、自动化维基系统和搜索引擎**。