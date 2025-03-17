---
layout: post
title:  使用Spring AI Evaluators测试LLM响应
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

现代Web应用程序越来越多地与[大语言模型(LLM)](https://www.baeldung.com/cs/large-language-models)相结合，以构建聊天机器人和虚拟助手等解决方案。

然而，虽然LLM很强大，但它们很容易产生[幻觉](https://www.baeldung.com/cs/llm-fix-hallucinations-why-happen)，并且它们的回答可能并不总是相关的、适当的或事实上准确的。

**评估LLM答案的一个解决方案是使用LLM本身**，最好是单独的。

为了实现这一点，Spring AI定义了Evaluator接口并提供了两个实现来检查LLM响应的相关性和事实准确性，即RelevanceEvaluator和FactCheckingEvaluator。

**在本教程中，我们将探讨如何使用Spring AI评估器测试LLM响应**。我们将使用Spring AI提供的两个基本实现来评估[检索增强生成(RAG)](https://www.baeldung.com/cs/retrieval-augmented-generation)聊天机器人的响应。

## 2. 构建RAG聊天机器人

在开始测试LLM响应之前，我们需要一个聊天机器人进行测试。为了进行演示，**我们将构建一个简单的RAG聊天机器人，该机器人会根据一组文档回答用户的问题**。

我们将使用开源工具[Ollama](https://github.com/ollama/ollama)在本地提取和运行我们的聊天完成和嵌入模型。

### 2.1 依赖项

让我们首先向项目的pom.xml文件中添加必要的依赖项：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
    <version>1.0.0-M5</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-markdown-document-reader</artifactId>
    <version>1.0.0-M5</version>
</dependency>
```

[Ollama Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-ollama-spring-boot-starter)帮助我们与Ollama服务建立连接。

此外，我们导入了Spring AI的[Markdown文档读取器依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-markdown-document-reader)，我们将使用它将.md文件转换为可以存储在向量存储中的文档。

由于当前版本1.0.0-M5是一个里程碑版本，我们还需要将Spring Milestones仓库添加到我们的pom.xml中：

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

鉴于我们在项目中使用了多个Spring AI启动器，我们还将在pom.xml中包含[Spring AI BOM](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-bom)：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.0.0-M5</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

通过这个添加，我们现在可以从两个Starter依赖中删除version标签。

**BOM消除了版本冲突的风险，并确保我们的Spring AI依赖关系彼此兼容**。

### 2.2 配置聊天完成和嵌入模型

接下来，让我们在application.yaml文件中配置我们的聊天完成和嵌入模型：

```yaml
spring:
  ai:
    ollama:
      chat:
        options:
          model: llama3.3
      embedding:
        options:
          model: nomic-embed-text
      init:
        pull-model-strategy: when_missing
```

在这里，**我们指定Meta提供的[llama3.3](https://ollama.com/library/llama3.3)模型作为我们的聊天完成模型，并指定Nomic AI提供的[nomic-embed-text](https://ollama.com/library/nomic-embed-text)模型作为我们的嵌入模型**。

此外，我们将pull-model-strategy设置为when_missing，这可确保Spring AI在本地不可用时提取指定的模型。

在配置有效模型时，**Spring AI会自动创建ChatModel和EmbeddingModel类型的Bean，分别允许我们与聊天完成和嵌入模型进行交互**。

让我们用它们来定义我们的聊天机器人所需的附加Bean：

```java
@Bean
public VectorStore vectorStore(EmbeddingModel embeddingModel) {
    return SimpleVectorStore
        .builder(embeddingModel)
        .build();
}

@Bean
public ChatClient contentGenerator(ChatModel chatModel, VectorStore vectorStore) {
    return ChatClient.builder(chatModel)
        .defaultAdvisors(new QuestionAnswerAdvisor(vectorStore))
        .build();
}
```

首先，我们定义一个VectorStore Bean并使用SimpleVectorStore实现，**这是一个使用java.util.Map类模拟向量存储的内存实现**。

在生产应用中，我们可以考虑使用真正的向量存储，例如[ChromaDB](https://www.baeldung.com/spring-ai-chromadb-vector-store)。

接下来，使用ChatModel和VectorStore Bean，**我们创建一个ChatClient类型的Bean，它是我们与聊天完成模型交互的主要入口点**。

我们用QuestionAnswerAdvisor对其进行配置，它使用向量存储根据用户的问题检索存储文档的相关部分，并将它们作为上下文提供给聊天模型。

### 2.3 填充内存向量存储

为了演示，我们在src/main/resources/documents目录中包含一个leave-policy.md文件，其中包含有关休假政策的示例信息。

现在，为了在应用程序启动期间用我们的文档填充向量存储，我们将创建一个实现[ApplicationRunner](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#7-spring-boot-applicationrunner)接口的VectorStoreInitializer类：

```java
@Component
class VectorStoreInitializer implements ApplicationRunner {
    private final VectorStore vectorStore;
    private final ResourcePatternResolver resourcePatternResolver;

    // standard constructor

    @Override
    public void run(ApplicationArguments args) {
        List<Document> documents = new ArrayList<>();
        Resource[] resources = resourcePatternResolver.getResources("classpath:documents/*.md");
        Arrays.stream(resources).forEach(resource -> {
            MarkdownDocumentReader markdownDocumentReader = new MarkdownDocumentReader(resource, MarkdownDocumentReaderConfig.defaultConfig());
            documents.addAll(markdownDocumentReader.read());
        });
        vectorStore.add(new TokenTextSplitter().split(documents));
    }
}
```

在run()方法中，我们首先使用注入的[ResourcePatternResolver](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/core/io/support/ResourcePatternResolver.html)类从src/main/resources/documents目录中获取所有Markdown文件。虽然我们只处理单个Markdown文件，但我们的方法是可扩展的。

然后，我们使用MarkdownDocumentReader类将获取的资源转换为Document对象。

最后，我们使用TokenTextSplitter类将文档分成更小的块，然后将其添加到向量存储中。

当我们调用add()方法时，**Spring AI会自动将纯文本内容转换为向量表示形式，然后将其存储在向量存储中**。我们不需要使用EmbeddingModel Bean显式转换它。

## 3. 使用Testcontainers设置Ollama

**为了方便本地开发和测试，我们将使用[Testcontainers](https://www.baeldung.com/docker-test-containers)来设置Ollama服务**，其先决条件是有一个活动的[Docker](https://www.baeldung.com/ops/docker-guide)实例。

### 3.1 测试依赖项

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

这些依赖提供了为Ollama服务启动临时Docker实例所需的类。

### 3.2 定义Testcontainers Bean

接下来，让我们创建一个[@TestConfiguration](https://www.baeldung.com/spring-boot-testing#test-configuration-withtestconfiguration)类来定义我们的Testcontainers Bean：

```java
@TestConfiguration(proxyBeanMethods = false)
class TestcontainersConfiguration {
    @Bean
    public OllamaContainer ollamaContainer() {
        return new OllamaContainer("ollama/ollama:0.5.7");
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

然后，**我们定义一个DynamicPropertyRegistrar Bean来配置Ollama服务的base-url**，这允许我们的应用程序连接到已启动的容器。

现在，我们可以通过使用@Import(TestcontainersConfiguration.class)注解来标注我们的测试类，在我们的集成测试中使用此配置。

## 4. 使用Spring AI Evaluator

现在我们已经构建了RAG聊天机器人并设置了本地测试环境，**让我们看看如何使用Spring AI的Evaluator接口的两个可用实现来测试它生成的响应**。

### 4.1 配置评估模型

我们测试的质量最终取决于我们使用的评估模型的质量，**我们将选择当前行业标准[bespoke-minicheck](https://ollama.com/library/bespoke-minicheck)模型，这是一个由Bespoke Labs专门为评估测试训练的开源模型**。它在[LLM-AggreFact排行榜](https://huggingface.co/datasets/lytang/LLM-AggreFact)上名列前茅，并且只产生yes/no响应。

让我们在application.yaml文件中进行配置：

```yaml
cn: 
  tuyucheng: 
    taketoday:
      evaluation:
        model: bespoke-minicheck
```

接下来，我们将创建一个单独的ChatClient Bean来与我们的评估模型进行交互：

```java
@Bean
public ChatClient contentEvaluator(OllamaApi olamaApi, @Value("${cn.tuyucheng.taketoday.evaluation.model}") String evaluationModel) {
    ChatModel chatModel = OllamaChatModel.builder()
        .ollamaApi(olamaApi)
        .defaultOptions(OllamaOptions.builder()
            .model(evaluationModel)
            .build())
        .modelManagementOptions(ModelManagementOptions.builder()
            .pullModelStrategy(PullModelStrategy.WHEN_MISSING)
            .build())
        .build();
    return ChatClient.builder(chatModel)
        .build();
}
```

在这里，我们使用Spring AI为我们创建的OllamaApi Bean和我们的自定义评估模型属性定义一个新的ChatClient Bean，我们使用[@Value](https://www.baeldung.com/spring-value-annotation)注解注入该Bean。

值得注意的是，我们对评估模型使用了自定义属性并手动创建其对应的ChatModel类，因为**OllamaAutoConfiguration类只允许我们通过spring.ai.ollama.chat.options.model属性配置单个模型**，我们已经将其用于内容生成模型。

### 4.2 使用RelevancyEvaluator评估LLM响应的相关性

**Spring AI提供了RelevancyEvaluator实现来检查LLM响应是否与用户的查询以及从向量存储中检索到的上下文相关**。

首先，让我们为其创建一个Bean：

```java
@Bean
public RelevancyEvaluator relevancyEvaluator(@Qualifier("contentEvaluator") ChatClient chatClient) {
    return new RelevancyEvaluator(chatClient.mutate());
}
```

我们使用@Qualifier注解来注入我们之前定义的relevancyEvaluator ChatClient Bean并创建RelevancyEvaluator类的实例。

由于**其构造函数需要一个构建器而不是直接的ChatClient实例**，我们调用mutate()方法，该方法返回使用我们现有客户端的配置初始化的ChatClient.Builder对象。

现在，让我们测试聊天机器人的响应是否相关：

```java
String question = "How many days sick leave can I take?";
ChatResponse chatResponse = contentGenerator.prompt()
    .user(question)
    .call()
    .chatResponse();

String answer = chatResponse.getResult().getOutput().getContent();
List<Document> documents = chatResponse.getMetadata().get(QuestionAnswerAdvisor.RETRIEVED_DOCUMENTS);
EvaluationRequest evaluationRequest = new EvaluationRequest(question, documents, answer);

EvaluationResponse evaluationResponse = relevancyEvaluator.evaluate(evaluationRequest);
assertThat(evaluationResponse.isPass()).isTrue();

String nonRelevantAnswer = "A lion is the king of the jungle";
evaluationRequest = new EvaluationRequest(question, documents, nonRelevantAnswer);
evaluationResponse = relevancyEvaluator.evaluate(evaluationRequest);
assertThat(evaluationResponse.isPass()).isFalse();
```

我们首先通过question调用contentGenerator ChatClient，然后从返回的ChatResponse中提取生成的answer和用于生成该答案的documents。

然后，我们创建一个EvaluationRequest，其中包含question、检索到的documents和聊天机器人的answer。我们将其传递给relevancyEvaluator Bean，并使用isPass()方法断言答案是相关的。

然而，当我们传递一个关于狮子的完全不相关的答案时，评估器正确地将其识别为不相关的。

### 4.3 使用FactCheckingEvaluator评估LLM响应的事实准确性

类似地，**Spring AI提供了一个FactCheckingEvaluator实现，以根据检索到的上下文验证LLM响应的事实准确性**。

让我们使用contentEvaluator ChatClient创建一个FactCheckingEvaluator Bean：

```java
@Bean
public FactCheckingEvaluator factCheckingEvaluator(@Qualifier("contentEvaluator") ChatClient chatClient) {
    return new FactCheckingEvaluator(chatClient.mutate());
}
```

最后，让我们测试一下聊天机器人响应的事实准确性：

```java
String question = "How many days sick leave can I take?";
ChatResponse chatResponse = contentGenerator.prompt()
    .user(question)
    .call()
    .chatResponse();

String answer = chatResponse.getResult().getOutput().getContent();
List<Document> documents = chatResponse.getMetadata().get(QuestionAnswerAdvisor.RETRIEVED_DOCUMENTS);
EvaluationRequest evaluationRequest = new EvaluationRequest(question, documents, answer);

EvaluationResponse evaluationResponse = factCheckingEvaluator.evaluate(evaluationRequest);
assertThat(evaluationResponse.isPass()).isTrue();

String wrongAnswer = "You can take no leaves. Get back to work!";
evaluationRequest = new EvaluationRequest(question, documents, wrongAnswer);
evaluationResponse = factCheckingEvaluator.evaluate(evaluationRequest);
assertThat(evaluationResponse.isPass()).isFalse();
```

与之前的方法类似，我们使用question、检索到的documents和聊天机器人的answer创建一个EvaluationRequest，并将其传递给我们的factCheckingEvaluator Bean。

我们断言，根据检索到的上下文，聊天机器人的响应在事实上是准确的。此外，我们用硬编码的事实错误答案重新测试评估，并断言isPass()方法为其返回false。

**值得注意的是，如果我们将硬编码的errorAnswer传递给RelevancyEvaluator，那么评估就会通过，因为即使响应在事实上是不正确的，它仍然与用户询问的病假主题相关**。

## 5. 总结

在本文中，我们探讨了使用Spring AI的Evaluator接口测试LLM响应。

我们构建了一个简单的RAG聊天机器人，它根据一组文档回答用户的问题，并使用Testcontainers设置Ollama服务，创建本地测试环境。

然后，我们使用Spring AI提供的RelevancyEvaluator和FactCheckingEvaluator实现来评估我们的聊天机器人响应的相关性和事实准确性。