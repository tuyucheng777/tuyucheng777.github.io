---
layout: post
title:  将LangChain4j与Micronaut结合使用
category: micronaut
copyright: micronaut
excerpt: Micronaut
---

## 1. 概述

LangChain4j是一个基于[LangChain](https://www.baeldung.com/java-langchain-basics)的Java库，我们在Java应用程序中使用它来与LLM集成。

**[Micronaut](https://www.baeldung.com/micronaut)是一个基于JVM的现代框架，旨在构建轻量级、模块化且快速的应用程序。我们使用它来创建微服务、无服务器应用程序和云原生解决方案，同时最大程度地缩短启动时间和减少内存使用量**。其强大的依赖项注入和AOT(提前)编译可确保高性能和可扩展性。在Micronaut中，我们与LangChain4j进行了出色的集成，使我们能够在单个应用程序中充分利用这两个框架的优势。

**在本教程中，我们将学习如何使用LangChain4j和Micronaut构建AI驱动的应用程序**。

## 2. 搬迁顾问应用程序

我们将构建一个集成LangChain4j的Micronaut应用程序，**在此应用程序中，我们将创建一个简单的聊天机器人来提供有关潜在搬迁国家的建议。聊天机器人将依靠一些提供的链接来获取信息**。

该应用程序仅回答有关其所知的国家的问题。

### 2.1 依赖

让我们从添加依赖开始。首先，我们添加[langchain4j-open-ai依赖](https://mvnrepository.com/artifact/dev.langchain4j/langchain4j-open-ai)：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-open-ai</artifactId>
    <version>0.36.2</version>
</dependency>
```

然后，让我们添加与[Micronaut相关的依赖](https://mvnrepository.com/artifact/io.micronaut.langchain4j)：

```xml
<dependency>  
    <groupId>io.micronaut.langchain4j</groupId>  
    <artifactId>micronaut-langchain4j-core</artifactId>  
    <version>0.0.1</version>  
</dependency>  
<dependency>  
    <groupId>io.micronaut.langchain4j</groupId>  
    <artifactId>micronaut-langchain4j-openai</artifactId>  
    <version>0.0.1</version>  
</dependency>
```

### 2.2 配置

由于我们将使用OpenAI的LLM，因此我们将[OpenAI API Key](https://platform.openai.com/api-keys)添加到我们的应用程序YAML文件中：

```yaml
langchain4j:
    open-ai:
        api-key: ${OPENAI_API_KEY}
```

### 2.3 搬迁顾问

现在，让我们创建聊天机器人接口：

```java
public interface RelocationAdvisor {

    @SystemMessage("""  
            You are a relocation advisor. Answer using official tone.        
            Provide the numbers you will get from the resources.            
            From the best of your knowledge answer the question below regarding possible relocation.            
            Please get information only from the following sources:             
                   - https://www.numbeo.com/cost-of-living/country_result.jsp?country=Spain             
                   - https://www.numbeo.com/cost-of-living/country_result.jsp?country=Romania
                   And their subpages. Then, answer. If you don't have the information - answer exact text 'I don't have  
                     information about {Country}'            
            """)
    String chat(@UserMessage String question);
}
```

在RelocationAdvisor类中，我们定义了chat()方法，该方法处理用户消息并以字符串形式返回模型响应。**我们使用@SystemMessage注解来指定聊天机器人的基本行为。在本例中，我们希望聊天机器人充当搬迁顾问，并仅依靠两个特定链接获取有关国家/地区的信息**。

### 2.4 搬迁顾问工厂

现在，让我们创建RelocationAdvisorFactory：

```java
@Factory
public class RelocationAdvisorFactory {

    @Value("${langchain4j.open-ai.api-key}")
    private String apiKey;

    @Singleton
    public RelocationAdvisor advisor() {
        ChatLanguageModel model = OpenAiChatModel.builder()
                .apiKey(apiKey)
                .modelName(OpenAiChatModelName.GPT_4_O_MINI)
                .build();

        return AiServices.create(RelocationAdvisor.class, model);
    }
}
```

使用此工厂，我们将RelocationAdvisor生成为Bean。**我们使用OpenAiChatModel构建器创建模型实例，然后利用AiServices根据接口和模型实例创建聊天机器人实例**。

### 2.5 测试有现存来源的情况

让我们测试一下我们的顾问，看看它提供了哪些有关它所知道的国家的信息：

```java
@MicronautTest(rebuildContext = true)
public class RelocationAdvisorLiveTest {

    Logger logger = LoggerFactory.getLogger(RelocationAdvisorLiveTest.class);

    @Inject
    RelocationAdvisor assistant;

    @Test
    void givenAdvisor_whenSendChatMessage_thenExpectedResponseShouldContainInformationAboutRomania() {
        String response = assistant.chat("Tell me about Romania");
        logger.info(response);
        Assertions.assertTrue(response.contains("Romania"));
    }

    @Test
    void givenAdvisor_whenSendChatMessage_thenExpectedResponseShouldContainInformationAboutSpain() {
        String response = assistant.chat("Tell me about Spain");
        logger.info(response);
        Assertions.assertTrue(response.contains("Spain"));
    }
}
```

我们将RelocationAdvisor的一个实例注入到测试中，并询问它有关几个国家的信息。正如预期的那样，响应包括了这些国家的名称。

此外，我们还记录了模型的响应。如下所示：

```text
15:43:47.334 [main] INFO  c.t.t.m.l.RelocationAdvisorLiveTest - Spain has a cost of living index of 58.54, which is relatively moderate compared to other countries.   
The average rent for a single-bedroom apartment in the city center is approximately €906.52, while outside the city center, it is around €662.68...
```

### 2.6 来源中缺失信息的测试用例

现在，让我们向我们的顾问询问一个它没有信息的国家：

```java
@Test  
void givenAdvisor_whenSendChatMessage_thenExpectedResponseShouldNotContainInformationAboutNorway() {  
    String response = assistant.chat("Tell me about Norway");  
    logger.info(response);  
    Assertions.assertTrue(response.contains("I don't have information about Norway"));  
}
```

在这种情况下，我们的聊天机器人会回复一条预定义的消息，表明它不知道这个国家。

## 3. 总结

在本文中，我们回顾了如何将LangChain4j与Micronaut集成。我们发现，实现LLM支持的功能并将其集成到Micronaut应用程序中非常简单。此外，我们可以很好地控制我们的AI服务，从而使我们能够通过其他行为增强它们并创建更复杂的解决方案。