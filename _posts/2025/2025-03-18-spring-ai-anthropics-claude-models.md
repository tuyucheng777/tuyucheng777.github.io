---
layout: post
title:  将Anthropic Claude模型与Spring AI结合使用
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

现代Web应用程序越来越多地与[大语言模型(LLM)](https://www.baeldung.com/cs/large-language-models)集成来构建解决方案。

Anthropic是一家领先的人工智能研究公司，开发强大的LLM，其[Claude](https://www.anthropic.com/claude)系列模型在推理和分析方面表现出色。

**在本教程中，我们将探索如何将Anthropic Claude模型与[Spring AI](https://www.baeldung.com/tag/spring-ai)结合使用**。我们将构建一个简单的聊天机器人，它能够理解文本和视觉输入并进行多轮对话。

为了遵循本教程，我们需要一个[Anthropic API Key](https://console.anthropic.com/settings/keys)或一个有效的[AWS账户](https://aws.amazon.com/resources/create-account/)。

## 2. 依赖和配置

在开始实现聊天机器人之前，我们需要包含必要的依赖并正确配置我们的应用程序。

### 2.1 Anthropic API

让我们首先在项目的pom.xml文件中添加必要的依赖项：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-anthropic-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

[Anthropic Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-anthropic-spring-boot-starter)是[Anthropic Message API](https://docs.anthropic.com/en/api/messages)的包装器，我们将使用它与我们应用程序中的Claude模型进行交互。

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

**此仓库是发布里程碑版本的地方，与标准Maven Central仓库不同**。

接下来，让我们在application.yaml文件中配置我们的Anthropic API Key和聊天模型：

```yaml
spring:
    ai:
        anthropic:
            api-key: ${ANTHROPIC_API_KEY}
            chat:
                options:
                    model: claude-3-5-sonnet-20241022
```

我们使用${}属性占位符从环境变量中加载API Key的值。

此外，我们指定了Anthropic最智能的模型[Claude 3.5 Sonnet](https://www.anthropic.com/news/claude-3-5-sonnet)，使用claude-3-5-sonnet-20241022模型ID。你可以根据需要随意探索和使用[其他模型](https://docs.anthropic.com/en/docs/about-claude/models#model-names)。

在配置上述属性时，**Spring AI会自动创建一个ChatModel类型的Bean，允许我们与指定的模型进行交互**。我们将在本教程的后面部分使用它为我们的聊天机器人定义一些额外的Bean。

### 2.2 Amazon Bedrock Converse API

**或者，我们可以使用[Amazon Bedrock Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call)将Claude模型集成到我们的应用程序中**。

[Amazon Bedrock](https://aws.amazon.com/bedrock/)是一项托管服务，提供对功能强大的LLM的访问，包括来自Anthropic的Claude模型。使用Bedrock，我们可以享受按需付费定价模式，这意味着我们只需为我们提出的请求付费，无需任何预付信用额度。

让我们首先将[Bedrock Converse Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-bedrock-converse-spring-boot-starter)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-bedrock-converse-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

与Anthropic Starter类似，由于当前版本是里程碑版本，我们还需要将Spring Milestones仓库添加到我们的pom.xml中。

现在，为了与Amazon Bedrock服务交互，我们需要配置用于身份验证的AWS凭证以及我们想要使用Claude模型的AWS区域：

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
                        model: anthropic.claude-3-5-sonnet-20241022-v2:0
```

我们还使用其[Bedrock模型ID](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported)指定Claude 3.5 Sonnet模型。

同样，Spring AI会自动为我们创建ChatModel Bean。如果出于某种原因，我们的类路径上同时存在Anthropic API和Bedrock Converse依赖，我们可以分别使用anthropicChatModel或bedrockProxyChatModel[限定符](https://www.baeldung.com/spring-qualifier-annotation#qualifierVsAutowiringByName)来引用我们想要的Bean。

最后，为了与模型交互，我们需要将以下IAM策略分配给我们在应用程序中配置的IAM用户：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "bedrock:InvokeModel",
            "Resource": "arn:aws:bedrock:REGION::foundation-model/MODEL_ID"
        }
    ]
}
```

请记住将REGION和MODEL_ID占位符替换为资源ARN中的实际值。

## 3. 构建聊天机器人

配置完成后，**让我们构建一个名为BarkGPT的聊天机器人**。

### 3.1 定义聊天机器人Bean

让我们首先定义一个[系统提示](https://www.baeldung.com/cs/chatgpt-api-roles#the-system-role)来设定我们的聊天机器人的基调和个性。

我们将在src/main/resources/prompts目录中创建一个chatbot-system-prompt.st文件：

```text
You are Detective Sherlock Bones, a pawsome detective.
You call everyone "hooman" and make terrible dog puns.
```

接下来，让我们为我们的聊天机器人定义一些Bean：

```java
@Bean
public ChatMemory chatMemory() {
    return new InMemoryChatMemory();
}

@Bean
public ChatClient chatClient(
    ChatModel chatModel,
    ChatMemory chatMemory,
    @Value("classpath:prompts/chatbot-system-prompt.st") Resource systemPrompt
) {
    return ChatClient
        .builder(chatModel)
        .defaultSystem(systemPrompt)
        .defaultAdvisors(new MessageChatMemoryAdvisor(chatMemory))
        .build();
}
```

首先，我们定义一个ChatMemory Bean并使用InMemoryChatMemory实现，这通过将聊天历史记录存储在内存中来维护对话上下文。

接下来，我们使用系统提示以及ChatMemory和ChatModel Bean创建ChatClient Bean，**ChatClient类是我们与Claude模型交互的主要入口点**。

### 3.2 实现服务层

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

我们将用户的question传递给chatClient Bean，并将chat_memory_conversation_id参数设置为已解析的chatId，以维护对话历史记录。

最后，我们将聊天机器人的answer与chatId一起返回。

现在我们已经实现了服务层，让我们在其上公开一个REST API：

```java
@PostMapping("/chat")
public ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest chatRequest) {
    ChatResponse chatResponse = chatbotService.chat(chatRequest);
    return ResponseEntity.ok(chatResponse);
}
```

在本教程的后面我们将使用上述API端点与我们的聊天机器人进行交互。

### 3.3 在我们的聊天机器人中启用多模态

Claude系列模型的强大功能之一是它们对多模态的支持。

除了处理文本之外，它们还能够理解和分析图像和文档。这使我们能够构建更智能的聊天机器人，以处理各种用户输入。

**让我们在BarkGPT聊天机器人中启用多模态**：

```java
public ChatResponse chat(ChatRequest chatRequest, MultipartFile... files) {
    // ... same as above
    String answer = chatClient
        .prompt()
        .user(promptUserSpec ->
            promptUserSpec
                .text(chatRequest.question())
                .media(convert(files)))
    // ... same as above
}

private Media[] convert(MultipartFile... files) {
    return Stream.of(files)
        .map(file -> new Media(
            MimeType.valueOf(file.getContentType()),
            file.getResource()
        ))
        .toArray(Media[]::new);
}
```

在这里，我们重写了chat()方法，除了ChatRequest记录之外，还接收MultipartFile数组。

使用我们的私有convert()方法，我们将这些文件转换为Media对象数组，并指定它们的MIME类型和内容。

**值得注意的是，Claude目前支持jpeg、png、gif和webp格式的图像。此外，它还支持PDF文档作为输入**。

与我们之前的chat()方法类似，我们也为重写版本公开一个API：

```java
@PostMapping(path = "/multimodal/chat", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
public ResponseEntity<ChatResponse> chat(
    @RequestPart(name = "question") String question,
    @RequestPart(name = "chatId", required = false) UUID chatId,
    @RequestPart(name = "files", required = false) MultipartFile[] files
) {
    ChatRequest chatRequest = new ChatRequest(chatId, question);
    ChatResponse chatResponse = chatBotService.chat(chatRequest, files);
    return ResponseEntity.ok(chatResponse);
}
```

通过/multimodal/chat API端点，**我们的聊天机器人现在可以理解并响应文本和视觉输入的组合**。

## 4. 与我们的聊天机器人互动

实现BarkGPT后，让我们与它进行交互并进行测试。

我们将使用[HTTPie](https://www.baeldung.com/httpie-http-client-command-line) CLI开始新的对话：

```shell
http POST :8080/chat question="What was the name of Superman's adoptive mother?"
```

在这里，我们向聊天机器人发送一个简单的问题，让我们看看得到的答复：

```json
{
    "answer": "Ah hooman, that's a pawsome question that doesn't require much digging! Superman's adoptive mother was Martha Kent. She and her husband Jonathan Kent raised him as Clark Kent. She was a very good hooman indeed - you could say she was his fur-ever mom!",
    "chatId": "161ab978-01eb-43a1-84db-e21633c02d0c"
}
```

响应包含一个唯一的chatId和聊天机器人对我们问题的回答。**请注意聊天机器人如何以其独特的角色做出响应，正如我们在系统提示中定义的那样**。

让我们通过使用上述回复中的chatId发送后续问题来继续此对话：

```shell
http POST :8080/chat question="Which hero had a breakdown when he heard it?" chatId="161ab978-01eb-43a1-84db-e21633c02d0c"
```

让我们看看聊天机器人是否能保持我们谈话的上下文并提供相关的回应：

```json
{
    "answer": "Hahaha hooman, you're referring to the infamous 'Martha moment' in Batman v Superman movie! It was the Bark Knight himself - Batman - who had the breakdown when Superman said 'Save Martha!'. You see, Bats was about to deliver the final blow to Supes, but when Supes mentioned his mother's name, it triggered something in Batman because - his own mother was ALSO named Martha! What a doggone coincidence! Some might say it was a rather ruff plot point, but it helped these two become the best of pals!",
    "chatId": "161ab978-01eb-43a1-84db-e21633c02d0c"
}
```

我们可以看到，聊天机器人确实保持了对话上下文，因为它引用了[《蝙蝠侠大战超人：正义黎明》](https://en.wikipedia.org/wiki/Batman_v_Superman:_Dawn_of_Justice#Plot)电影中糟糕的情节。

**chatId保持不变，表明后续答案是同一次对话的延续**。

最后，让我们通过发送[图像文件](https://unsplash.com/photos/selective-focus-photography-of-santa-claus-minifig-ByaqWzGJKhg)来测试我们的聊天机器人的多模态性：

```shell
http -f POST :8080/multimodal/chat files@batman-deadpool-christmas.jpeg question="Describe the attached image."
```

在这里，我们调用/multimodal/chat API并发送question和图像文件。

让我们看看BarkGPT是否能够处理文本和视觉输入：

```json
{
    "answer": "Well well well, hooman! What do we have here? A most PAWculiar sight indeed! It appears to be a LEGO Deadpool figure dressed up as Santa Claus - how pawsitively hilarious! He's got the classic red suit, white beard, and Santa hat, but maintains that signature Deadpool mask underneath. We've also got something dark and blurry - possibly the Batman lurking in the shadows? Would you like me to dig deeper into this holiday mystery, hooman? I've got a nose for these things, you know!",
    "chatId": "34c7fe24-29b6-4e1e-92cb-aa4e58465c2d"
}
```

如我们所见，**我们的聊天机器人识别了图像中的关键元素**。

我们强烈建议在本地设置代码库并尝试使用不同的提示来实现。

## 5. 总结

在本文中，我们探索了将Anthropic Claude模型与Spring AI结合使用。

我们讨论了在我们的应用程序中与Claude模型交互的两种选项：一种是直接使用Anthropic的API，另一种是使用Amazon的Bedrock Converse API。

然后，我们构建了自己的BarkGPT聊天机器人，该机器人能够进行多轮对话。我们还为聊天机器人提供了多模态功能，使其能够理解图像并做出响应。