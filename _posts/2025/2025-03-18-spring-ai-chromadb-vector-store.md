---
layout: post
title:  Spring AI与ChromaDB向量存储
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

对于传统数据库，我们通常依靠精确的关键字或基本模式匹配来实现搜索功能。虽然这种方法对于简单的应用程序来说已经足够了，但它无法完全理解自然语言查询背后的含义和上下文。

[向量存储](https://www.baeldung.com/cs/vector-databases)通过将数据存储为能够捕捉其含义的数字向量来解决这一限制，相似的单词最终会彼此靠近，这允许进行语义搜索，即使相关结果不包含查询中使用的确切关键字，也会返回相关结果。

**在本教程中，我们将探讨如何将开源向量存储[ChromaDB](https://www.trychroma.com/)与[Spring AI](https://www.baeldung.com/tag/spring-ai)集成**。

为了将文本数据转换为ChromaDB可以存储和搜索的向量，我们需要一个嵌入模型，我们将使用[Ollama](https://github.com/ollama/ollama)在本地运行嵌入模型。

## 2. 依赖

让我们首先向项目的pom.xml文件中添加必要的依赖项：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-chroma-store-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

[ChromaDB Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-chroma-store-spring-boot-starter/latest)使我们能够与ChromaDB向量存储建立连接并与其进行交互。

此外，我们导入[Ollama Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-ollama-spring-boot-starter/latest)，我们将使用它来运行我们的嵌入模型。

由于当前版本1.0.0-M6是一个里程碑版本，我们还需要将Spring Milestones仓库添加到我们的pom.xml中：

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

**与标准Maven Central仓库不同，此仓库是发布里程碑版本的地方**。

由于我们在项目中使用了多个Spring AI Starter，因此我们还在pom.xml中包含了[Spring AI BOM](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-bom/latest)：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.ai</groupId>
            <artifactId>spring-ai-bom</artifactId>
            <version>1.0.0-M6</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

通过这个添加，我们现在可以从两个Starter依赖中删除version标签。

**BOM消除了版本冲突的风险，并确保我们的Spring AI依赖关系彼此兼容**。

## 3. 使用Testcontainers设置本地测试环境

**为了促进本地开发和测试，我们将使用[Testcontainers](https://www.baeldung.com/tag/testcontainers)来设置我们的ChromaDB向量存储和Ollama服务**。

通过Testcontainers运行所需服务的先决条件是有一个活动的[Docker](https://www.baeldung.com/ops/docker-guide)实例。

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
    <artifactId>chromadb</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>ollama</artifactId>
    <scope>test</scope>
</dependency>
```

这些依赖为我们提供了必要的类，以便为我们的两个外部服务启动临时Docker实例。

### 3.2 定义Testcontainers Bean

接下来，让我们创建一个[@TestConfiguration](https://www.baeldung.com/spring-boot-testing#test-configuration-withtestconfiguration)类来定义我们的Testcontainers Bean：

```java
@TestConfiguration(proxyBeanMethods = false)
class TestcontainersConfiguration {

    @Bean
    @ServiceConnection
    public ChromaDBContainer chromaDB() {
        return new ChromaDBContainer("chromadb/chroma:0.5.20");
    }

    @Bean
    @ServiceConnection
    public OllamaContainer ollama() {
        return new OllamaContainer("ollama/ollama:0.4.5");
    }
}
```

我们为我们的容器指定最新的稳定版本。

我们还使用[@ServiceConnection](https://www.baeldung.com/spring-boot-built-in-testcontainers#using-serviceconnection-for-dynamic-properties)标注我们的Bean方法，**这将动态注册与我们的两个外部服务建立连接所需的所有属性**。

即使不使用Testcontainers支持，Spring AI在本地运行时也会自动连接到ChromaDB和Ollama，它们的默认端口分别为8000和11434。

但是，在生产中，我们可以使用相应的Spring AI属性覆盖连接详细信息：

```yaml
spring:
    ai:
        vectorstore:
            chroma:
                client:
                    host: ${CHROMADB_HOST}
                    port: ${CHROMADB_PORT}
        ollama:
            base-url: ${OLLAMA_BASE_URL}
```

**一旦正确配置了连接详细信息，Spring AI就会自动为我们创建VectorStore和EmbeddingModel类型的Bean**，使我们能够分别与向量存储和嵌入模型进行交互。我们将在本教程的后面介绍如何使用这些Bean。

尽管@ServiceConnection自动定义了必要的连接详细信息，但我们仍然需要在application.yml文件中配置一些其他属性：

```yaml
spring:
    ai:
        vectorstore:
            chroma:
                initialize-schema: true
        ollama:
            embedding:
                options:
                    model: nomic-embed-text
            init:
                chat:
                    include: false
                pull-model-strategy: WHEN_MISSING
```

在这里，我们为ChromaDB启用schema初始化。然后，**我们将[nomic-embed-text](https://ollama.com/library/nomic-embed-text)配置为我们的嵌入模型，并指示Ollama在我们的系统中不存在该模型时提取该模型**。

或者，我们可以根据需要使用Ollama的不同[嵌入模型](https://ollama.com/search?c=embedding)或[Hugging Face模型](https://spring.io/blog/2024/10/22/leverage-the-power-of-45k-free-hugging-face-models-with-spring-ai-and-ollama)。

### 3.3 在开发过程中使用Testcontainers

虽然Testcontainers主要用于集成测试，但我们也可以在本地开发期间使用它。

为了实现这一点，我们将在src/test/java目录中创建一个单独的主类：

```java
class TestApplication {

    public static void main(String[] args) {
        SpringApplication.from(Application::main)
                .with(TestcontainersConfiguration.class)
                .run(args);
    }
}
```

我们创建一个TestApplication类，并在其主方法中，使用TestcontainersConfiguration类启动我们的主Application类。

**此设置可帮助我们在本地设置和管理外部服务，我们可以运行Spring Boot应用程序并让它连接到通过Testcontainers启动的外部服务**。

## 4. 在应用程序启动时填充ChromaDB

现在我们已经设置好了本地环境，让我们在应用程序启动期间用一些示例数据填充我们的ChromaDB向量存储。

### 4.1 从PoetryDB获取诗歌记录

为了演示，**我们将使用[PoetryDB API](https://poetrydb.org/index.html)来获取诗歌**。

让我们为此创建一个PoetryFetcher工具类：

```java
class PoetryFetcher {

    private static final String BASE_URL = "https://poetrydb.org/author/";
    private static final String DEFAULT_AUTHOR_NAME = "Shakespeare";

    public static List<Poem> fetch() {
        return fetch(DEFAULT_AUTHOR_NAME);
    }

    public static List<Poem> fetch(String authorName) {
        return RestClient
                .create()
                .get()
                .uri(URI.create(BASE_URL + authorName))
                .retrieve()
                .body(new ParameterizedTypeReference<>() {});
    }
}

record Poem(String title, List<String> lines) {}
```

我们使用[RestClient](https://www.baeldung.com/spring-boot-restclient)调用具有指定authorName的PoetryDB API，为了将API响应反序列化为Poem[记录](https://www.baeldung.com/java-record-keyword)列表，我们使用ParameterizedTypeReference而不明确指定通用响应类型，Java将为我们推断类型。

我们还重载了不带任何参数的fetch()方法来检索作者莎士比亚的诗歌，我们将在下一节中使用此方法。

### 4.2 将文档存储在ChromaDB向量存储中

现在，为了在应用程序启动期间用诗歌填充我们的ChromaDB向量存储，**我们将创建一个实现[ApplicationRunner](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#7-spring-boot-applicationrunner)接口的VectorStoreInitializer类**：

```java
@Component
class VectorStoreInitializer implements ApplicationRunner {

    private final VectorStore vectorStore;

    // standard constructor

    @Override
    public void run(ApplicationArguments args) {
        List<Document> documents = PoetryFetcher
                .fetch()
                .stream()
                .map(poem -> {
                    Map<String, Object> metadata = Map.of("title", poem.title());
                    String content = String.join("\n", poem.lines());
                    return new Document(content, metadata);
                })
                .toList();
        vectorStore.add(documents);
    }
}
```

在我们的VectorStoreInitializer中，我们自动注入VectorStore的一个实例。

在run()方法中，我们使用PoetryFetcher工具类来检索诗歌记录列表。然后，我们将每首诗映射到一个文档中，其中诗行作为内容，标题作为元数据。

最后，我们将所有文档存储在向量存储中。**当我们调用add()方法时，Spring AI会自动将纯文本内容转换为向量表示，然后再将其存储在向量存储中。我们不需要使用EmbeddingModel Bean显式转换它**。

默认情况下，Spring AI使用SpringAiCollection作为集合名称将数据存储在我们的向量存储中，但我们可以使用spring.ai.vectorstore.chroma.collection-name属性覆盖它。

## 5. 测试语义搜索

在填充了ChromaDB向量存储后，让我们验证我们的语义搜索功能：

```java
private static final int MAX_RESULTS = 3;

@ParameterizedTest
@ValueSource(strings = {"Love and Romance", "Time and Mortality", "Jealousy and Betrayal"})
void whenSearchingShakespeareTheme_thenRelevantPoemsReturned(String theme) {
    SearchRequest searchRequest = SearchRequest
        .builder()
        .query(theme)
        .topK(MAX_RESULTS)
        .build();
    List<Document> documents = vectorStore.similaritySearch(searchRequest);

    assertThat(documents)
        .hasSizeLessThanOrEqualTo(MAX_RESULTS)
        .allSatisfy(document -> {
                String title = String.valueOf(document.getMetadata().get("title"));
                assertThat(title)
                    .isNotBlank();
            });
}
```

在这里，我们使用[@ValueSource](https://www.baeldung.com/parameterized-tests-junit-5#1-simple-values)将一些常见的莎士比亚主题传递给我们的测试方法，然后我们创建一个SearchRequest对象，以主题作为查询，以MAX_RESULTS作为所需结果的数量。

接下来，我们使用searchRequest调用vectorStore bean的similaritySearch()方法。与VectorStore的add()方法类似，Spring AI在查询向量存储之前将查询转换为其向量表示。

**返回的文档将包含与给定主题语义相关的诗歌，即使它们不包含精确的关键字**。

## 6. 总结

在本文中，我们探讨了如何将ChromaDB向量存储与Spring AI集成。

使用Testcontainers，我们为ChromaDB和Ollama服务启动了Docker容器，创建了本地测试环境。

我们研究了如何在应用程序启动期间使用PoetryDB API中的诗歌填充我们的向量存储。然后，我们使用常见的诗歌主题来验证我们的语义搜索功能。