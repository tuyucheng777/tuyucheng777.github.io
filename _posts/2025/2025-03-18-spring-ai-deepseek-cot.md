---
layout: post
title:  使用Spring AI和DeepSeek模型构建AI聊天机器人
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

现代Web应用程序越来越多地与[大型语言模型(LLM)](https://www.baeldung.com/cs/large-language-models)集成来构建解决方案。

DeepSeek是一家中国人工智能研究公司，致力于开发强大的LLM，最近其[DeepSeek-V3](https://api-docs.deepseek.com/news/news1226)和[DeepSeek-R1](https://api-docs.deepseek.com/news/news250120)模型震撼了AI世界。后者模型及其响应揭示了其[思路链(CoT)](https://www.baeldung.com/cs/chatgpt-o1-o3#1-chain-of-thought)，让我们深入了解AI模型如何解释和处理给定的提示。

**在本教程中，我们将探索如何将DeepSeek模型与Spring AI集成**，我们将构建一个能够进行多轮文本对话的简单聊天机器人。

## 2. 依赖和配置

有多种方法可以将DeepSeek模型集成到我们的应用程序中，在本节中，我们将讨论一些常用的选项，可以选择最适合我们要求的一种。

### 2.1 使用OpenAI API

**DeepSeek模型与[OpenAI API](https://platform.openai.com/docs/api-reference/introduction)完全兼容，可以通过任何OpenAI客户端或库访问**。

让我们首先将Spring AI的[OpenAI Starter依赖项](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-openai-spring-boot-starter)添加到我们项目的pom.xml文件中：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

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

**此仓库是发布里程碑版本的地方，与标准Maven Central仓库不同**。无论我们选择哪种配置选项，我们都需要添加此里程碑仓库。

接下来，让我们在application.yaml文件中配置我们的[DeepSeek API Key](https://platform.deepseek.com/api_keys)和[聊天模型](https://api-docs.deepseek.com/quick_start/pricing)：

```yaml
spring:
    ai:
        openai:
            api-key: ${DEEPSEEK_API_KEY}
            chat:
                options:
                    model: deepseek-reasoner
            base-url: https://api.deepseek.com
            embedding:
                enabled: false
```

此外，我们指定DeepSeek API的基本URL并禁用[嵌入](https://www.baeldung.com/cs/dimensionality-word-embeddings)，因为DeepSeek目前不提供任何嵌入兼容模型。

在配置上述属性时，**Spring AI会自动创建一个ChatModel类型的Bean，允许我们与指定的模型进行交互**。我们将在本教程的后面部分使用它为我们的聊天机器人定义一些额外的Bean。

### 2.2 使用Amazon Bedrock Converse API

或者，我们可以使用[Amazon Bedrock Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html)将DeepSeek R1模型集成到我们的应用程序中。

要完成此配置步骤，我们需要一个[有效的AWS账户](https://aws.amazon.com/resources/create-account/)。**DeepSeek-R1模型可通过[Amazon Bedrock Marketplace](https://aws.amazon.com/bedrock/marketplace/)获取，并可使用[Amazon SageMaker](https://aws.amazon.com/sagemaker/)托管。可参考此[部署指南](https://aws.amazon.com/cn/blogs/machine-learning/deepseek-r1-model-now-available-in-amazon-bedrock-marketplace-and-amazon-sagemaker-jumpstart/#:~:text=Deploy%20DeepSeek%2DR1%20in%20Amazon%20Bedrock%20Marketplace)进行设置**。

让我们首先将[Bedrock Converse Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-bedrock-converse-spring-boot-starter)添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-bedrock-converse-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

接下来，为了与Amazon Bedrock交互，我们需要在application.yaml文件中配置用于身份验证的AWS凭证以及托管DeepSeek模型的区域：

```yaml
spring:
    ai:
        bedrock:
            aws:
                region: ${AWS_REGION}
                access-key: ${AWS_ACCESS_KEY}
                secret-key: ${AWS_SECRET_KEY}
            converse:
                chat:
                    options:
                        model: arn:aws:sagemaker:REGION:ACCOUNT_ID:endpoint/ENDPOINT_NAME
```

我们使用${}属性占位符从[环境变量](https://www.baeldung.com/spring-boot-properties-env-variables#use-environment-variable-in-applicationyml-file)中加载我们的属性值。

此外，我们指定托管DeepSeek模型的SageMaker端点URL ARN。我们应该记得用实际值替换REGION、ACCOUNT_ID和ENDPOINT_NAME占位符。

最后，为了与模型交互，我们需要将以下IAM策略分配给我们在应用程序中配置的IAM用户：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "bedrock:InvokeModel",
            "Resource": "arn:aws:bedrock:REGION:ACCOUNT_ID:marketplace/model-endpoint/all-access"
        }
    ]
}
```

再次，我们应该记住用Resource ARN中的实际值替换REGION和ACCOUNT_ID占位符。

### 2.3 使用Ollama进行本地设置

**对于本地开发和测试，我们可以通过[Ollama](https://github.com/ollama/ollama)运行DeepSeek模型，这是一个开源工具**，允许我们在本地机器上运行LLM。

让我们在项目的pom.xml文件中导入必要的依赖项：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

[Ollama Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-ollama-spring-boot-starter)帮助我们与Ollama服务建立连接。

接下来，我们在application.yaml文件中配置我们的聊天模型：

```yaml
spring:
    ai:
        ollama:
            chat:
                options:
                    model: deepseek-r1
            init:
                pull-model-strategy: when_missing
            embedding:
                enabled: false
```

这里我们指定了[deepseek-r1](https://ollama.com/library/deepseek-r1)模型，不过我们也可以使用[不同的可用模型](https://ollama.com/search?q=deepseek)来尝试这个实现。

此外，我们将pull-model-strategy设置为when_missing，**这可确保Spring AI在本地不可用时拉取指定的模型**。

Spring AI在localhost上运行时会自动连接到Ollama的默认端口11434，但是，我们可以使用spring.ai.ollama.base-url属性覆盖连接URL。或者，我们可以使用[Testcontainers](https://www.baeldung.com/spring-ai-ollama-hugging-face-models#setting-up-ollama-with-testcontainers)来设置Ollama服务。

在这里，Spring AI会再次为我们自动创建ChatModel Bean。如果出于某种原因，我们的类路径上同时包含OpenAI API、Bedrock Converse和Ollama这三个依赖项，**我们可以分别使用openAiChatModel、bedrockProxyChatModel或ollamaChatModel[限定符](https://www.baeldung.com/spring-qualifier-annotation#qualifierVsAutowiringByName)来引用我们想要的特定Bean**。

## 3. 构建聊天机器人

现在我们已经讨论了各种配置选项，**让我们使用配置的DeepSeek模型构建一个简单的聊天机器人**。

### 3.1 定义聊天机器人Bean

让我们首先定义聊天机器人所需的Bean：

```java
@Bean
ChatMemory chatMemory() {
    return new InMemoryChatMemory();
}

@Bean
ChatClient chatClient(ChatModel chatModel, ChatMemory chatMemory) {
    return ChatClient
        .builder(chatModel)
        .defaultAdvisors(new MessageChatMemoryAdvisor(chatMemory))
        .build();
}
```

首先，我们使用InMemoryChatMemory实现定义一个ChatMemory Bean，它将聊天历史记录存储在内存中以维护对话上下文。

接下来，我们使用ChatModel和ChatMemory Bean创建ChatClient Bean，**ChatClient类是我们与已配置的DeepSeek模型交互的主要入口点**。

### 3.2 创建自定义StructuredOutputConverter

如前所述，DeepSeek-R1模型的响应包括其CoT，我们得到的响应格式如下：

```text
<think>
Chain of Thought
</think>
Answer
```

不幸的是，由于这种独特的格式，当我们尝试将响应解析为Java类时，当前版本Spring AI中存在的所有[结构化输出](https://www.baeldung.com/spring-artificial-intelligence-structure-output)转换器都会失败并引发异常。

**因此，让我们创建自己的自定义StructuredOutputConverter实现来分别解析AI模型的答案和CoT**：

```java
record DeepSeekModelResponse(String chainOfThought, String answer) {
}

class DeepSeekModelOutputConverter implements StructuredOutputConverter<DeepSeekModelResponse> {
    private static final String OPENING_THINK_TAG = "<think>";
    private static final String CLOSING_THINK_TAG = "</think>";

    @Override
    public DeepSeekModelResponse convert(@NonNull String text) {
        if (!StringUtils.hasText(text)) {
            throw new IllegalArgumentException("Text cannot be blank");
        }
        int openingThinkTagIndex = text.indexOf(OPENING_THINK_TAG);
        int closingThinkTagIndex = text.indexOf(CLOSING_THINK_TAG);

        if (openingThinkTagIndex != -1 && closingThinkTagIndex != -1 && closingThinkTagIndex > openingThinkTagIndex) {
            String chainOfThought = text.substring(openingThinkTagIndex + OPENING_THINK_TAG.length(), closingThinkTagIndex);
            String answer = text.substring(closingThinkTagIndex + CLOSING_THINK_TAG.length());
            return new DeepSeekModelResponse(chainOfThought, answer);
        } else {
            logger.debug("No <think> tags found in the response. Treating entire text as answer.");
            return new DeepSeekModelResponse(null, text);
        }
    }
}
```

在这里，我们的转换器从AI模型的响应中提取chainOfThought和answer，并将它们作为DeepSeekModelResponse[记录](https://www.baeldung.com/java-record-keyword)返回。

如果AI响应不包含<think\>标签，我们会将整个响应视为答案，**这确保了与其他响应中不包含CoT的DeepSeek模型的兼容性**。

### 3.3 实现服务层

配置完成后，**我们来创建一个ChatbotService类。我们将注入之前定义的ChatClient Bean，以便与指定的DeepSeek模型进行交互**。

但首先，让我们定义两个简单的记录来表示聊天请求和响应：

```java
record ChatRequest(@Nullable UUID chatId, String question) {}

record ChatResponse(UUID chatId, String chainOfThought, String answer) {}
```

ChatRequest包含用户的question和一个可选的chatId来识别正在进行的对话。

类似地，ChatResponse包含chatId，以及聊天机器人的chainOfThought和answer。

现在，让我们实现预期的功能：

```java
ChatResponse chat(ChatRequest chatRequest) {
    UUID chatId = Optional
        .ofNullable(chatRequest.chatId())
        .orElse(UUID.randomUUID());
    DeepSeekModelResponse response = chatClient
        .prompt()
        .user(chatRequest.question())
        .advisors(advisorSpec -> advisorSpec.param("chat_memory_conversation_id", chatId))
        .call()
        .entity(new DeepSeekModelOutputConverter());
    return new ChatResponse(chatId, response.chainOfThought(), response.answer());
}
```

如果传入请求不包含chatId，我们将生成一个新的，**这允许用户开始新的对话或继续现有的对话**。

我们将用户的question传递给chatClient Bean，并将chat_memory_conversation_id参数设置为已解析的chatId，以维护对话历史记录。

最后，我们创建自定义DeepSeekModelOutputConverter类的实例，并将其传递给entity()方法，以将AI模型的响应解析为DeepSeekModelResponse记录。然后，我们从中提取chainOfThought和answer，并将它们与chatId一起返回。

### 3.4 与我们的聊天机器人互动

现在我们已经实现了服务层，让我们在其上公开一个REST API：

```java
@PostMapping("/chat")
ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest chatRequest) {
    ChatResponse chatResponse = chatbotService.chat(chatRequest);
    return ResponseEntity.ok(chatResponse);
}
```

让我们使用[HTTPie](https://www.baeldung.com/httpie-http-client-command-line) CLI调用上述API端点并开始新的对话：

```shell
http POST :8080/chat question="What was the name of Superman's adoptive mother?"
```

在这里，我们向聊天机器人发送一个简单的问题，让我们看看收到的答复：

![](/assets/images/2025/springai/springaideepseekcot01.png)

响应包含一个唯一的chatId，以及聊天机器人的chainOfThought和对我们问题的回答。**我们可以看到AI模型如何使用chainOfThought属性推理并解决给定的提示**。

让我们通过使用上述回复中的chatId发送后续问题来继续此对话：

```shell
http POST :8080/chat question="Which bald billionaire hates him?" chatId="1e3c151f-cded-4f10-a5fc-c52c5952411c"
```

看看聊天机器人是否能保持我们谈话的上下文并提供相关的回应：

![](/assets/images/2025/springai/springaideepseekcot02.png)

我们看到，聊天机器人确实保留了对话上下文，**chatId保持不变，表明后续回答是同一对话的延续**。

## 4. 总结

在本文中，我们探索了将DeepSeek模型与Spring AI结合使用。

我们讨论了将DeepSeek模型集成到我们的应用程序中的各种选项，其中一种是直接使用OpenAI API，因为DeepSeek与它兼容，另一种是使用亚马逊的Bedrock Converse API。此外，我们还探讨了使用Ollama设置本地测试环境。

然后，我们构建了一个能够进行多轮文本对话的简单聊天机器人，并使用自定义的StructuredOutputConverter实现从AI模型的响应中提取思路链和答案。