---
layout: post
title:  Spring AI Advisor指南
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

人工智能驱动的应用程序是新的趋势，我们正在广泛实现各种[RAG应用程序](https://www.baeldung.com/cs/retrieval-augmented-generation)和提示API，并使用[LLM](https://www.baeldung.com/cs/large-language-models)创建令人印象深刻的项目。借助[Spring AI](https://www.baeldung.com/spring-ai)，我们可以更快、更一致地完成这些任务。

在本文中，我们将回顾一项名为Spring AI Advisors的宝贵功能，它可以为我们处理各种日常任务。

## 2. Spring AI Advisor是什么

Advisors是处理AI应用程序中的请求和响应的拦截器，我们可以使用它们为我们的提问流程设置额外的功能。例如，我们可以建立聊天记录、排除敏感词或为每个请求添加额外的上下文。

**此功能的核心组件是CallAroundAdvisor接口，我们实现此接口来创建会影响我们的请求或响应的Advisor链**，下图描述了Advisor的流程：

![](/assets/images/2025/springai/springaiadvisors01.png)

我们将提示发送给与顾问链相连的聊天模型，在发送提示之前，链中的每个顾问都会执行其before操作。同样，在我们收到聊天模型的响应之前，每个顾问都会调用自己的after操作。

## 3. Chat Memory Advisors

Chat Memory Advisors是一组非常有用的Advisor实现，我们可以使用这些Advisor为我们的聊天提示提供通信历史记录，从而提高聊天响应的准确性。

### 3.1 MessageChatMemoryAdvisor

**使用MessageChatMemoryAdvisor，我们可以使用messages属性提供聊天客户端调用的聊天历史记录**。我们将所有消息保存在ChatMemory实现中，并可以控制历史记录大小。

让我们为该顾问实现一个简单的展示：

```java
@SpringBootTest(classes = ChatModel.class)
@EnableAutoConfiguration
@ExtendWith(SpringExtension.class)
public class SpringAILiveTest {

    @Autowired
    @Qualifier("openAiChatModel")
    ChatModel chatModel;
    ChatClient chatClient;

    @BeforeEach
    void setup() {
        chatClient = ChatClient.builder(chatModel).build();
    }

    @Test
    void givenMessageChatMemoryAdvisor_whenAskingChatToIncrementTheResponseWithNewName_thenNamesFromTheChatHistoryExistInResponse() {
        ChatMemory chatMemory = new InMemoryChatMemory();
        MessageChatMemoryAdvisor chatMemoryAdvisor = new MessageChatMemoryAdvisor(chatMemory);

        String responseContent = chatClient.prompt()
                .user("Add this name to a list and return all the values: Bob")
                .advisors(chatMemoryAdvisor)
                .call()
                .content();

        assertThat(responseContent)
                .contains("Bob");

        responseContent = chatClient.prompt()
                .user("Add this name to a list and return all the values: John")
                .advisors(chatMemoryAdvisor)
                .call()
                .content();

        assertThat(responseContent)
                .contains("Bob")
                .contains("John");

        responseContent = chatClient.prompt()
                .user("Add this name to a list and return all the values: Anna")
                .advisors(chatMemoryAdvisor)
                .call()
                .content();

        assertThat(responseContent)
                .contains("Bob")
                .contains("John")
                .contains("Anna");
    }
}
```

在这个测试中，我们创建了一个MessageChatMemoryAdvisor实例，其中包含InMemoryChatMemory。然后我们发送了一些提示，要求聊天室返回包括历史数据在内的人员姓名。如我们所见，对话中的所有姓名都已返回。

### 3.2 PromptChatMemoryAdvisor

使用PromptChatMemoryAdvisor，我们实现了相同的目标-为聊天模型提供对话历史记录。**不同之处在于，使用此Advisor，我们将聊天记忆添加到提示中**。在后台，我们使用以下建议扩展了提示文本：

```text
Use the conversation memory from the MEMORY section to provide accurate answers.
---------------------
MEMORY:
{memory}
---------------------
```

让我们验证一下它是如何工作的：

```java
@Test
void givenPromptChatMemoryAdvisor_whenAskingChatToIncrementTheResponseWithNewName_thenNamesFromTheChatHistoryExistInResponse() {
    ChatMemory chatMemory = new InMemoryChatMemory();
    PromptChatMemoryAdvisor chatMemoryAdvisor = new PromptChatMemoryAdvisor(chatMemory);

    String responseContent = chatClient.prompt()
            .user("Add this name to a list and return all the values: Bob")
            .advisors(chatMemoryAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .contains("Bob");

    responseContent = chatClient.prompt()
            .user("Add this name to a list and return all the values: John")
            .advisors(chatMemoryAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .contains("Bob")
            .contains("John");

    responseContent = chatClient.prompt()
            .user("Add this name to a list and return all the values: Anna")
            .advisors(chatMemoryAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .contains("Bob")
            .contains("John")
            .contains("Anna");
}
```

再次，我们尝试创建一些提示，要求聊天模型使用PromptChatMemoryAdvisor考虑对话记忆。正如预期的那样，所有数据都正确返回给我们。

### 3.3 VectorStoreChatMemoryAdvisor

**使用VectorStoreChatMemoryAdvisor，我们获得了更强大的功能。我们通过向量存储中的[相似性匹配](https://www.baeldung.com/cs/semantic-similarity-of-two-phrases)来搜索消息的上下文**，我们在搜索相关文档时考虑对话ID。在我们的示例中，我们将采用略微重写的SimpleVectorStore，但我们也可以将其替换为任何[向量数据库](https://www.baeldung.com/cs/vector-databases)。

首先，让我们创建一个向量存储的Bean：

```java
@Configuration
public class SimpleVectorStoreConfiguration {

    @Bean
    public VectorStore vectorStore(@Qualifier("openAiEmbeddingModel")EmbeddingModel embeddingModel) {
        return new SimpleVectorStore(embeddingModel) {
            @Override
            public List<Document> doSimilaritySearch(SearchRequest request) {
                float[] userQueryEmbedding = embeddingModel.embed(request.query);
                return this.store.values()
                        .stream()
                        .map(entry -> Pair.of(entry.getId(),
                                EmbeddingMath.cosineSimilarity(userQueryEmbedding, entry.getEmbedding())))
                        .filter(s -> s.getSecond() >= request.getSimilarityThreshold())
                        .sorted(Comparator.comparing(Pair::getSecond))
                        .limit(request.getTopK())
                        .map(s -> this.store.get(s.getFirst()))
                        .toList();
            }
        };
    }
}
```

这里我们创建了一个SimpleVectorStore类的Bean并重写了它的doSimilaritySearch()方法，默认的SimpleVectorStore不支持元数据过滤，这里我们将忽略这一事实。由于我们在测试期间只会进行一次对话，因此这种方法非常适合我们。

现在，让我们测试历史上下文行为：

```java
@Test
void givenVectorStoreChatMemoryAdvisor_whenAskingChatToIncrementTheResponseWithNewName_thenNamesFromTheChatHistoryExistInResponse() {
    VectorStoreChatMemoryAdvisor chatMemoryAdvisor = new VectorStoreChatMemoryAdvisor(vectorStore);

    String responseContent = chatClient.prompt()
            .user("Find cats from our chat history, add Lion there and return a list")
            .advisors(chatMemoryAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .contains("Lion");

    responseContent = chatClient.prompt()
            .user("Find cats from our chat history, add Puma there and return a list")
            .advisors(chatMemoryAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .contains("Lion")
            .contains("Puma");

    responseContent = chatClient.prompt()
            .user("Find cats from our chat history, add Leopard there and return a list")
            .advisors(chatMemoryAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .contains("Lion")
            .contains("Puma")
            .contains("Leopard");
}
```

我们要求聊天填充列表中的几个项目，同时，在后台，我们进行了相似性搜索以获取所有相似的文档，并且我们的聊天LLM根据这些文档准备了答案。

## 4. QuestionAnswerAdvisor

在[RAG应用程序](https://www.baeldung.com/spring-ai-mongodb-rag)中，我们广泛使用QuestionAnswerAdvisor。**使用此顾问，我们准备一个提示，根据准备好的上下文请求信息，使用相似性搜索从向量存储中检索上下文**。

让我们检查一下这个行为：

```java
@Test
void givenQuestionAnswerAdvisor_whenAskingQuestion_thenAnswerShouldBeProvidedBasedOnVectorStoreInformation() {
    Document document = new Document("The sky is green");
    List<Document> documents = new TokenTextSplitter().apply(List.of(document));
    vectorStore.add(documents);
    QuestionAnswerAdvisor questionAnswerAdvisor = new QuestionAnswerAdvisor(vectorStore);

    String responseContent = chatClient.prompt()
            .user("What is the sky color?")
            .advisors(questionAnswerAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .containsIgnoringCase("green");
}
```

我们用文档中的特定信息填充向量存储，然后，我们使用QuestionAnswerAdvisor创建提示并验证其响应是否与文档内容相符。

## 5. SafeGuardAdvisor

有时我们必须防止在客户端提示中使用某些敏感词，不可否认，**我们可以使用SafeGuardAdvisor来实现这一点，方法是指定禁用词列表并将其包含在提示的顾问实例中**。如果在搜索请求中使用了任何这些词，它将被拒绝，并且顾问会提示我们重新措辞：

```java
@Test
void givenSafeGuardAdvisor_whenSendPromptWithSensitiveWord_thenExpectedMessageShouldBeReturned() {
    List<String> forbiddenWords = List.of("Word2");
    SafeGuardAdvisor safeGuardAdvisor = new SafeGuardAdvisor(forbiddenWords);

    String responseContent = chatClient.prompt()
            .user("Please split the 'Word2' into characters")
            .advisors(safeGuardAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .contains("I'm unable to respond to that due to sensitive content");
}
```

在此示例中，我们首先创建了一个包含单个禁用词的SafeGuardAdvisor。然后，我们尝试在提示中使用此词，并且如预期的那样，收到了禁用词验证消息。

## 6. 实现自定义顾问

当然，我们可以用任何我们需要的逻辑来实现我们的自定义顾问。让我们创建一个CustomLoggingAdvisor，我们将在其中记录所有聊天请求和响应：

```java
public class CustomLoggingAdvisor implements CallAroundAdvisor {
    private final static Logger logger = LoggerFactory.getLogger(CustomLoggingAdvisor.class);

    @Override
    public AdvisedResponse aroundCall(AdvisedRequest advisedRequest, CallAroundAdvisorChain chain) {
        advisedRequest = this.before(advisedRequest);

        AdvisedResponse advisedResponse = chain.nextAroundCall(advisedRequest);

        this.observeAfter(advisedResponse);

        return advisedResponse;
    }

    private void observeAfter(AdvisedResponse advisedResponse) {
        logger.info(advisedResponse.response()
                .getResult()
                .getOutput()
                .getContent());

    }

    private AdvisedRequest before(AdvisedRequest advisedRequest) {
        logger.info(advisedRequest.userText());
        return advisedRequest;
    }

    @Override
    public String getName() {
        return "CustomLoggingAdvisor";
    }

    @Override
    public int getOrder() {
        return Integer.MAX_VALUE;
    }
}
```

这里，我们实现了CallAroundAdvisor接口，并在调用前后添加了日志逻辑。**此外，我们从getOrder()方法返回了最大整数值，因此我们的顾问将是链中的最后一个**。

现在，让我们测试一下我们的新顾问：

```java
@Test
void givenCustomLoggingAdvisor_whenSendPrompt_thenPromptTextAndResponseShouldBeLogged() {

    CustomLoggingAdvisor customLoggingAdvisor = new CustomLoggingAdvisor();

    String responseContent = chatClient.prompt()
            .user("Count from 1 to 10")
            .advisors(customLoggingAdvisor)
            .call()
            .content();

    assertThat(responseContent)
            .contains("1")
            .contains("10");
}
```

我们创建了CustomLoggingAdvisor并将其附加到提示，让我们看看执行后日志中会发生什么：

```text
c.t.t.s.advisors.CustomLoggingAdvisor      : Count from 1 to 10
c.t.t.s.advisors.CustomLoggingAdvisor      : 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
```

我们可以看到，我们的顾问成功记录了提示文本和聊天响应。

## 7. 总结

在本教程中，我们探讨了一项名为Advisors的出色Spring AI功能。借助Advisors，我们可以获得聊天记忆功能、对敏感词的控制以及与向量存储的无缝集成。此外，我们还可以轻松创建自定义扩展来添加特定功能，使用Advisors使我们能够一致且直接地实现所有这些功能。