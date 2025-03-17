---
layout: post
title:  将Hugging Face模型与Spring AI和Ollama结合使用
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

[人工智能](https://www.baeldung.com/cs/category/ai)正在改变我们构建Web应用程序的方式，[Hugging Face](https://huggingface.co/)是一个流行的平台，它提供了大量[开源](https://www.baeldung.com/cs/open-source-explained)和预训练的[LLM](https://www.baeldung.com/cs/large-language-models)。

我们可以使用开源工具[Ollama](https://github.com/ollama/ollama)在本地机器上运行LLM，它支持运行Hugging Face的[GGUF](https://huggingface.co/docs/hub/en/gguf)格式模型。

**在本教程中，我们将探索如何将Hugging Face模型与[Spring AI](https://www.baeldung.com/tag/spring-ai)和Ollama结合使用**，我们将使用聊天完成模型构建一个简单的聊天机器人，并使用嵌入模型实现语义搜索。

## 2. 依赖

让我们首先在项目的pom.xml文件中添加必要的依赖项：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

[Ollama Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-ollama-spring-boot-starter)可帮助我们与Ollama服务建立连接，我们将使用它来提取和运行我们的聊天完成和嵌入模型。

由于当前版本1.0.0-M5是一个里程碑版本，我们还需要将Spring Milestones仓库添加到pom.xml中：

```xml
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```

**此仓库是发布里程碑版本的地方，与标准Maven Central仓库不同**。

## 3. 使用Testcontainers设置Ollama

**为了方便本地开发和测试，我们将使用[Testcontainers](https://www.baeldung.com/docker-test-containers)来设置Ollama服务**。

### 3.1 测试依赖

首先，让我们在pom.xml中添加必要的测试依赖项：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>ollama</artifactId>
    <scope>test</scope>
</dependency>
```

我们导入Spring Boot的[Spring AI Testcontainers](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-testcontainers)依赖和Testcontainers的[Ollama模块](https://mvnrepository.com/artifact/org.testcontainers/ollama)。

### 3.2 定义Testcontainers Bean

接下来，让我们创建一个[@TestConfiguration](https://www.baeldung.com/spring-boot-testing#test-configuration-withtestconfiguration)类来定义我们的Testcontainers Bean：

```java
@TestConfiguration(proxyBeanMethods = false)
class TestcontainersConfiguration {
    @Bean
    public OllamaContainer ollamaContainer() {
        return new OllamaContainer("ollama/ollama:0.5.4");
    }

    @Bean
    public DynamicPropertyRegistrar dynamicPropertyRegistrar(OllamaContainer ollamaContainer) {
        return registry -> {
            registry.add("spring.ai.ollama.base-url", ollamaContainer::getEndpoint);
        };
    }
}
```

我们在创建OllamaContainer Bean时指定了Ollama镜像的最新稳定版本。

然后，**我们定义一个DynamicPropertyRegistrar Bean来配置Ollama服务的base-url**，这允许我们的应用程序连接到已启动的Ollama容器。

### 3.3 在开发过程中使用Testcontainers

虽然Testcontainers主要用于集成测试，但我们也可以在本地开发期间使用它。

为了实现这一点，我们将在src/test/java目录中创建一个单独的主类：

```java
public class TestApplication {
    public static void main(String[] args) {
        SpringApplication.from(Application::main)
                .with(TestcontainersConfiguration.class)
                .run(args);
    }
}
```

我们创建一个TestApplication类，并在其main()方法内，使用TestcontainersConfiguration类启动我们的主Application类。

此设置帮助我们运行Spring Boot应用程序并让它连接到通过Testcontainers启动的Ollama服务。

## 4. 使用聊天完成模型

现在我们已经设置了本地Ollama容器，**让我们使用聊天完成模型来构建一个简单的聊天机器人**。

### 4.1 配置聊天模型和聊天机器人Bean

让我们首先在application.yaml文件中配置聊天完成模型：

```yaml
spring:
    ai:
        ollama:
            init:
                pull-model-strategy: when_missing
            chat:
                options:
                    model: hf.co/microsoft/Phi-3-mini-4k-instruct-gguf
```

要配置Hugging Face模型，我们使用hf.co/{username}/{repository}的格式。在这里，我们指定微软提供的[Phi-3-mini-4k-instruct](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct-gguf)模型的GGUF版本。

在我们的实现中，使用此模型并不是一个严格的要求。**我们的建议是在本地设置代码库并尝试使用[更多的聊天完成模型](https://huggingface.co/models?library=gguf&other=text-generation-inference&sort=downloads)**。

此外，我们将pull-model-strategy设置为when_missing，这可确保Spring AI在本地不可用时拉取指定的模型。

**在配置有效模型时，Spring AI会自动创建ChatModel类型的Bean**，允许我们与聊天完成模型进行交互。

让我们用它来定义我们的聊天机器人所需的附加Bean：

```java
@Configuration
class ChatbotConfiguration {
    @Bean
    public ChatMemory chatMemory() {
        return new InMemoryChatMemory();
    }

    @Bean
    public ChatClient chatClient(ChatModel chatModel, ChatMemory chatMemory) {
        return ChatClient
                .builder(chatModel)
                .defaultAdvisors(new MessageChatMemoryAdvisor(chatMemory))
                .build();
    }
}
```

首先，我们定义一个ChatMemory Bean并使用InMemoryChatMemory实现，这通过将聊天历史记录存储在内存中来维护对话上下文。

接下来，使用ChatMemory和ChatModel Bean，**我们创建一个ChatClient类型的Bean，它是我们与聊天完成模型交互的主要入口点**。

### 4.2 实现聊天机器人

配置完成后，**让我们创建一个ChatbotService类，我们将注入之前定义的ChatClient Bean来与我们的模型进行交互**。

但首先，让我们定义两个简单的[记录](https://www.baeldung.com/java-record-keyword)来表示聊天请求和响应：

```java
record ChatRequest(@Nullable UUID chatId, String question) {}

record ChatResponse(UUID chatId, String answer) {}
```

ChatRequest包含用户的question和一个可选的chatId来识别正在进行的对话。

类似地，ChatResponse包含chatId和聊天机器人的answer。

现在，让我们实现预期的功能：

```java
public ChatResponse chat(ChatRequest chatRequest) {
    UUID chatId = Optional
        .ofNullable(chatRequest.chatId())
        .orElse(UUID.randomUUID());
    String answer = chatClient
        .prompt()
        .user(chatRequest.question())
        .advisors(advisorSpec -> advisorSpec.param("chat_memory_conversation_id", chatId))
        .call()
        .content();
    return new ChatResponse(chatId, answer);
}
```

如果传入请求不包含chatId，我们将生成一个新的，**这允许用户开始新的对话或继续现有的对话**。

我们将用户的问题传递给chatClient Bean，并将chat_memory_conversation_id参数设置为已解析的chatId，以维护对话历史记录。

最后，我们返回聊天机器人的answer以及chatId。

### 4.3 与我们的聊天机器人互动

现在我们已经实现了服务层，让我们在其上公开一个REST API：

```java
@PostMapping("/chat")
public ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest chatRequest) {
    ChatResponse chatResponse = chatbotService.chat(chatRequest);
    return ResponseEntity.ok(chatResponse);
}
```

我们将使用上述API端点与我们的聊天机器人进行交互。

让我们使用[HTTPie](https://www.baeldung.com/httpie-http-client-command-line) CLI开始新的对话：

```shell
http POST :8080/chat question="Who wanted to kill Harry Potter?"
```

我们向聊天机器人发送一个简单的问题，看看我们得到的答复是什么：

```json
{
    "chatId": "7b8a36c7-2126-4b80-ac8b-f9eedebff28a",
    "answer": "Lord Voldemort, also known as Tom Riddle, wanted to kill Harry Potter because of a prophecy that foretold a boy born at the end of July would have the power to defeat him."
}
```

响应包含唯一的chatId和聊天机器人对我们的问题的回答。

让我们通过使用上述回复中的chatId发送后续问题来继续此对话：

```shell
http POST :8080/chat chatId="7b8a36c7-2126-4b80-ac8b-f9eedebff28a" question="Who should he have gone after instead?"
```

让我们看看聊天机器人是否能保持我们谈话的上下文并提供相关的回应：

```json
{
    "chatId": "7b8a36c7-2126-4b80-ac8b-f9eedebff28a",
    "answer": "Based on the prophecy's criteria, Voldemort could have targeted Neville Longbottom instead, as he was also born at the end of July to parents who had defied Voldemort three times."
}
```

我们可以看到，聊天机器人确实保持了对话上下文，因为它引用了我们在上一条消息中讨论的预言。

**chatId保持不变，表明后续答案是同一次对话的延续**。

## 5. 使用嵌入模型

从聊天完成模型继续，**我们现在将使用嵌入模型在小型引语数据集上实现语义搜索**。

我们将从外部API获取报价，将其存储在内存[向量存储](https://www.baeldung.com/cs/vector-databases)中，然后执行语义搜索。

### 5.1 从外部API获取报价记录

为了演示，**我们将使用[QuoteSlate API](https://quoteslate.vercel.app/)来获取报价**。

让我们为此创建一个QuoteFetcher工具类：

```java
class QuoteFetcher {
    private static final String BASE_URL = "https://quoteslate.vercel.app";
    private static final String API_PATH = "/api/quotes/random";
    private static final int DEFAULT_COUNT = 50;

    public static List<Quote> fetch() {
        return RestClient
                .create(BASE_URL)
                .get()
                .uri(uriBuilder ->
                        uriBuilder
                                .path(API_PATH)
                                .queryParam("count", DEFAULT_COUNT)
                                .build())
                .retrieve()
                .body(new ParameterizedTypeReference<>() {});
    }
}

record Quote(String quote, String author) {}
```

使用[RestClient](https://www.baeldung.com/spring-boot-restclient)，我们调用默认计数50的QuoteSlate API，并使用ParameterizedTypeReference将API响应反序列化为Quote[记录](https://www.baeldung.com/java-record-keyword)列表。

### 5.2 配置和填充内存向量存储

现在，让我们在application.yaml中配置一个嵌入模型：

```yaml
spring:
    ai:
        ollama:
            embedding:
                options:
                    model: hf.co/nomic-ai/nomic-embed-text-v1.5-GGUF
```

**我们使用nomic-ai提供的[nomic-embed-text-v1.5](https://huggingface.co/nomic-ai/nomic-embed-text-v1.5-GGUF)模型的GGUF版本**，同样，你可以随意尝试使用[不同的嵌入模型](https://huggingface.co/models?library=gguf&other=text-embeddings-inference&sort=downloads)来实现此实现。

指定有效模型后，Spring AI会自动为我们创建一个EmbeddingModel类型的Bean。

让我们用它来创建一个VectorStore Bean：

```java
@Bean
public VectorStore vectorStore(EmbeddingModel embeddingModel) {
    return SimpleVectorStore
        .builder(embeddingModel)
        .build();
}
```

为了演示，我们创建了一个SimpleVectorStore类的Bean。**它是一个内存实现，使用java.util.Map类模拟向量存储**。

现在，为了在应用程序启动期间用报价填充我们的向量存储，我们将创建一个实现[ApplicationRunner](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#7-spring-boot-applicationrunner)接口的VectorStoreInitializer类：

```java
@Component
class VectorStoreInitializer implements ApplicationRunner {
    private final VectorStore vectorStore;

    // standard constructor

    @Override
    public void run(ApplicationArguments args) {
        List<Document> documents = QuoteFetcher
                .fetch()
                .stream()
                .map(quote -> {
                    Map<String, Object> metadata = Map.of("author", quote.author());
                    return new Document(quote.quote(), metadata);
                })
                .toList();
        vectorStore.add(documents);
    }
}
```

在我们的VectorStoreInitializer中，我们自动注入VectorStore的一个实例。

在run()方法中，我们使用QuoteFetcher工具类来检索Quote记录列表。然后，我们将每条报价映射到Document中，并将author字段配置为metadata。

最后，我们将所有文档存储在向量存储中。**当我们调用add()方法时，Spring AI会自动将纯文本内容转换为向量表示，然后再将其存储在向量存储中**。我们不需要使用EmbeddingModel Bean显式转换它。

### 5.3 测试语义搜索

在向量存储填充之后，让我们验证我们的语义搜索功能：

```java
private static final int MAX_RESULTS = 3;

@ParameterizedTest
@ValueSource(strings = {"Motivation", "Happiness"})
void whenSearchingQuotesByTheme_thenRelevantQuotesReturned(String theme) {
    SearchRequest searchRequest = SearchRequest
        .builder()
        .query(theme)
        .topK(MAX_RESULTS)
        .build();
    List<Document> documents = vectorStore.similaritySearch(searchRequest);

    assertThat(documents)
        .hasSizeBetween(1, MAX_RESULTS)
        .allSatisfy(document -> {
            String title = String.valueOf(document.getMetadata().get("author"));
            assertThat(title).isNotBlank();
        });
}
```

在这里，我们使用[@ValueSource](https://www.baeldung.com/parameterized-tests-junit-5#1-simple-values)将一些常见的报价主题传递给我们的测试方法。然后我们创建一个SearchRequest对象，以主题作为查询，以MAX_RESULTS作为所需结果的数量。

接下来，我们使用searchRequest调用vectorStore bean的similaritySearch()方法。与VectorStore的add()方法类似，Spring AI在查询向量存储之前将我们的查询转换为其向量表示。

**返回的文档将包含与给定主题语义相关的报价，即使它们不包含精确的关键字**。

## 6. 总结

在本文中，我们探索了将Hugging Face模型与Spring AI结合使用。

使用Testcontainers，我们设置了Ollama服务，创建了本地测试环境。

首先，我们使用聊天补全模型构建了一个简单的聊天机器人。然后，我们使用嵌入模型实现了语义搜索。