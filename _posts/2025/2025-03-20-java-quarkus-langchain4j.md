---
layout: post
title:  利用Quarkus和LangChain4j
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

**在本教程中，我们将学习[LangChain](https://www.baeldung.com/java-langchain-basics)，这是一个由大语言模型([LLM](https://www.baeldung.com/cs/large-language-models))驱动的应用程序开发框架**。更具体地说，我们将使用[LangChain4j](https://github.com/langchain4j/langchain4j)，这是一个Java框架，它简化了与[LangChain](https://www.baeldung.com/java-langchain-basics)的集成，并允许开发人员将LLM集成到他们的应用程序中。

**该框架在构建检索增强生成([RAG](https://www.promptingguide.ai/techniques/rag))方面非常流行**。在本文中，我们将了解所有这些术语，并了解如何利用Quarkus构建这样的应用程序。

## 2. 大语言模型(LLM)

大语言模型(LLM)是使用大量数据训练的AI系统，它们的目标是生成类似人类的输出，**OpenAI的GPT-4可能是当今最知名的LLM之一**。此外，LLM可以回答问题并执行各种[自然语言](https://www.baeldung.com/cs/natural-language-processing-understanding-generation)处理任务。

**这些模型是强大聊天机器人、内容生成、文档分析、视频和图像生成等的支柱**。

### 2.1 LangChain

[LangChain](https://www.baeldung.com/java-langchain-basics)是一个流行的开源框架，可帮助开发人员构建基于LLM的应用程序。它最初是为Python开发的，简化了使用LLM创建多步骤工作流程的过程，例如：

- 与不同LLM集成：LangChain不提供自己的LLM，而是提供与许多不同LLM(如[GPT-4](https://chatbotapp.ai/landing?utm_source=GoogleAds&utm_medium=cpc&utm_campaign={campaign}&utm_id=21163648510&utm_term=161191449032&utm_content=705556645648&gad_source=1&gclid=CjwKCAjw3P-2BhAEEiwA3yPhwLD7R9pvucx93IJS8hqqpMpSGqMl54JDW_cERE27g3DLzSEExrfKeRoCkkwQAvD_BwE)、[Lama-3](https://llama.meta.com/)等)交互的标准接口
- 集成外部工具：LangChain可以与对此类应用程序非常重要的其他工具集成，例如向量数据库、Web浏览器和其他API
- 管理记忆：需要对话流的应用程序需要历史记录或上下文持久性，LangChain具有记忆功能，允许模型在交互过程中“记住”信息

LangChain是Python开发人员构建AI应用程序的首选框架之一。然而，对于Java开发人员来说，LangChain4j框架提供了这个强大框架的类似Java的改编版。

### 2.2 LangChain4j

LangChain4j是一个受LangChain启发的Java库，旨在帮助使用LLM构建AI驱动的应用程序。项目创建者旨在填补Java框架与AI系统的众多Python和JavaScript选项之间的空白。

**LangChain4j使Java开发人员能够使用LangChain为Python开发人员提供的相同工具和灵活性，从而可以开发聊天机器人、摘要引擎或智能搜索系统等应用程序**。

可以使用其他框架(例如[Quarkus)](https://quarkus.io/)快速构建强大的应用程序。

### 2.3 Quarkus

Quarkus是一个专为云原生应用设计的Java框架，它能够显著减少内存使用量和启动时间，非常适合微服务、无服务器架构和Kubernetes环境。

将Quarkus整合到你的AI应用程序中可确保你的系统可扩展、高效且可用于企业级生产。**此外，Quarkus提供易于使用且学习曲线较低的功能，可将LangChain4j集成到我们的Quarkus应用程序中**。

## 3. 聊天机器人应用程序

为了展示Quarkus与LangChain4j集成的强大功能，我们将创建一个简单的聊天机器人应用程序来回答有关Quarkus的问题。为此，我们使用GPT-4。这将对我们有所帮助，因为它使用来自互联网的大量数据进行训练，因此，我们不需要训练自己的模型。

该应用程序将仅回答有关Quarkus的问题，并会记住特定聊天中的先前对话。

### 3.1 依赖

现在我们已经介绍了主要概念和工具，让我们使用LangChain4j和Quarkus构建我们的第一个应用程序。首先，让我们使用REST和[Redis](https://redis.io/)创建我们的Quarkus应用程序，然后添加以下依赖项：

```xml
<dependency>
    <groupId>io.quarkiverse.langchain4j</groupId>
    <artifactId>quarkus-langchain4j-openai</artifactId>
    <version>0.18.0.CR1</version>
</dependency>
<dependency>
    <groupId>io.quarkiverse.langchain4j</groupId>
    <artifactId>quarkus-langchain4j-memory-store-redis</artifactId>
    <version>0.18.0.CR1</version>
</dependency>
```

**第一步是将最新的[quarkus-langchain4j-openai](https://mvnrepository.com/artifact/io.quarkiverse.langchain4j/quarkus-langchain4j-openai)和[quarkus-langchain4j-memory-store-redis](https://mvnrepository.com/artifact/io.quarkiverse.langchain4j/quarkus-langchain4j-memory-store-redis)依赖添加到pom.xml中**。

第一个依赖将引入与Quarkus兼容的LangChain4J版本，此外，它将为我们提供开箱即用的组件，用于配置LangChain4j以使用[OpenAI](https://platform.openai.com/docs/models) LLM模型作为我们服务的骨干。有许多LLM可供选择，但为了简单起见，我们使用当前默认的[GPT-4o-mini](https://platform.openai.com/docs/models/gpt-4o-mini)。为了使用这样的服务，我们需要在OpenAI中创建一个Api Key。完成后，我们就可以继续了。我们稍后再讨论第二个依赖。

### 3.2 设置

现在，让我们设置我们的应用程序以允许它与GPT-4和Redis通信，这样我们就可以开始实现我们的聊天机器人了。为此，我们需要在Quarkus中配置一些应用程序属性，因为它有开箱即用的组件来执行此操作，我们只需要将以下属性添加到application.properties文件中：

```properties
quarkus.langchain4j.openai.api-key=${OPEN_AI_KEY}
quarkus.langchain4j.openai.organization-id=${OPEN_AI_ORG}

quarkus.redis.hosts=${REDIS_HOST}
```

Quarkus为该应用程序提供了许多其他有价值的配置，请参阅[Quarkus文档](https://docs.quarkiverse.io/quarkus-langchain4j/dev/openai.html)；但是，这对于我们的聊天机器人来说已经足够了。

或者，我们还可以使用环境变量来设置这样的属性：

```bash
QUARKUS_REDIS_HOSTS: redis://localhost:6379
QUARKUS_LANGCHAIN4J_OPENAI_API_KEY: changeme
QUARKUS_LANGCHAIN4J_OPENAI_ORGANIZATION_ID: changeme
```

### 3.3 组件

LangChain4j提供了丰富的功能来帮助我们实现复杂的流程，例如[文档检索](https://docs.quarkiverse.io/quarkus-langchain4j/dev/retrievers.html)。尽管如此，在本文中，我们将重点介绍简单的对话流程。话虽如此，让我们创建我们的简单聊天机器人：

```java
@Singleton
@RegisterAiService
public interface ChatBot {

    @SystemMessage("""
            During the whole chat please behave like a Quarkus specialist and only answer directly related to Quarkus,
            its documentation, features and components. Nothing else that has no direct relation to Quarkus.
            """)
    @UserMessage("""
            From the best of your knowledge answer the question below regarding Quarkus. Please favor information from the following sources:
            - https://docs.quarkiverse.io/
            - https://quarkus.io/
            
            And their subpages. Then, answer:
            
            ---
            {question}
            ---
            """)
    String chat(@MemoryId UUID memoryId, String question);
}
```

**在ChatBot类中，我们注册了一个将在对话期间使用LLM的服务。为此，Quarkus提供了[@RegisterAiService](https://docs.quarkiverse.io/quarkus-langchain4j/dev/ai-services.html)注解，它抽象了集成LangChain4j和GTP-4的设置**。接下来，我们应用一些[Prompt Engineering](https://www.baeldung.com/cs/prompt-engineering)来构建我们希望聊天机器人的行为方式。

简而言之，提示工程是一种塑造指令的技术，它将帮助LLM在解释和处理用户请求的过程中调整其行为。

因此，使用此类提示，我们为机器人实现了所需的行为。LangChain4j使用消息来实现这一点，如：

- SystemMessage是影响模型响应但对最终用户隐藏的内部指令
- UserMessage是终端用户发送的消息，但是，LangChain4j可以帮助我们在这些消息上应用模板来增强提示

接下来我们讨论一下@MemoryId。

### 3.4 记忆

尽管LLM非常擅长生成文本并提供相关问题的答案，但LLM本身还不足以实现聊天机器人。LLM自身无法实现的一个关键方面是记住之前消息中的上下文或数据，这就是我们需要记忆能力的原因。

LangChain4j提供了抽象ChatMemoryStore和ChatMemory的子集，这些抽象允许不同的实现记忆和管理代表聊天的消息列表。此外，需要一个标识符来存储和检索聊天记忆，@MemoryId用于标记此类ID。

**Quarkus为聊天记忆提供了一个基于Redis的开箱即用的实现，这就是我们添加第二个依赖和Redis设置的原因**。

### 3.5 API

最后，我们的示例聊天机器人应用程序缺少一个接口，以便用户与系统进行通信。让我们创建一个API，以便用户可以将他们的问题发送给我们的聊天机器人。

```java
@Path("/chat")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class ChatAPI {

    private ChatBot chatBot;

    public ChatAPI(ChatBot chatBot) {
        this.chatBot = chatBot;
    }

    @POST
    public Answer mesage(@QueryParam("q") String question, @QueryParam("id") UUID chatId) {
        chatId = chatId == null ? UUID.randomUUID() : chatId;
        String message = chatBot.chat(chatId, question);
        return new Answer(message, chatId);
    }
}
```

API接收一个问题和一个可选的聊天ID，如果未提供ID，我们将创建它，**这允许用户控制是继续现有聊天还是从头开始创建聊天**。然后，它将两个参数发送到我们的ChatBot类，该类集成了对话。或者，我们可以实现一个端点来检索聊天中的所有消息。

运行以下命令将会向我们的聊天机器人发送我们的第一个问题。

```bash
# Let's ask: Does quarkus support redis?
curl --location --request POST 'http://localhost:8080/chat?q=Does%20quarkus%20support%20redis%3F'
```

因此我们得到：

```json
{
    "message": "Yes, Quarkus supports Redis through the Quarkus Redis client extension...",
    "chatId": "d3414b32-454e-4577-af81-8fb7460f13cd"
}
```

请注意，由于这是我们的第一个问题，因此没有提供ID，而是创建了一个新ID。现在，我们可以使用此ID来跟踪我们的对话历史记录。接下来，让我们测试聊天机器人是否将我们的历史记录作为对话的背景。

```bash
# Replace the id param with the value we got from the last call. 
# So we ask: q=What was my last Quarkus question?&id=d3414b32-454e-4577-af81-8fb7460f13cd
curl --location --request POST 'http://localhost:8080/chat?q=What%20was%20my%20last%20Quarkus%20question%3F&id=d3414b32-454e-4577-af81-8fb7460f13cd'
```

正如预期的那样，我们的聊天机器人正确回答了。

```json
{
    "message": "Your last Quarkus question was asking whether Quarkus supports Redis.",
    "chatId": "d3414b32-454e-4577-af81-8fb7460f13cd"
}
```

这意味着我们的简单聊天机器人应用程序只需创建几个类并设置一些属性后即可。

## 4. 总结

使用Quarkus和LangChain4j简化了基于Quarkus的聊天机器人的构建过程，该聊天机器人可与OpenAI的语言模型交互并保留对话记忆。**Quarkus和LangChain4j的强大组合使开发AI驱动的应用程序成为可能，并且开销极小，同时仍提供丰富的功能**。

从此设置开始，我们可以通过添加更多特定领域的知识并提高其回答复杂问题的能力来继续扩展聊天机器人的功能。Quarkus和LangChain4j还为此提供了许多其他工具。

在本文中，我们看到了Quarkus和LangChain4j的组合为使用Java开发AI系统带来了多少生产力和效率。此外，我们还介绍了开发此类应用程序所涉及的概念。