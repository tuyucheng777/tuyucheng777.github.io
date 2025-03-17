---
layout: post
title:  将Amazon Nova模型与Spring AI结合使用
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

现代Web应用程序越来越多地与[大语言模型(LLM)](https://www.baeldung.com/cs/large-language-models)集成来构建解决方案。

AWS提供的[Amazon Nova理解模型](https://aws.amazon.com/ai/generative-ai/nova/understanding/)是一套快速且经济高效的[基础模型](https://www.baeldung.com/cs/foundation-models-artificial-intelligence)，可通过[Amazon Bedrock](https://aws.amazon.com/bedrock/)访问，它提供了方便的即用即付定价模型。

**在本教程中，我们将探索如何将Amazon Nova模型与[Spring AI](https://www.baeldung.com/tag/spring-ai)结合使用**。我们将构建一个简单的聊天机器人，该机器人能够理解文本和视觉输入并参与多轮对话。

为了继续本教程，我们需要一个活跃的[AWS账户](https://aws.amazon.com/resources/create-account/)。

## 2. 设置项目

在开始实现聊天机器人之前，我们需要包含必要的依赖项并正确配置我们的应用程序。

### 2.1 依赖

让我们首先将[Bedrock Converse Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-bedrock-converse-spring-boot-starter)添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-bedrock-converse-spring-boot-starter</artifactId>
    <version>1.0.0-M5</version>
</dependency>
```

**上述依赖是[Amazon Bedrock Converse API](https://docs.aws.amazon.com/bedrock/latest/userguide/conversation-inference-call.html)的包装器**，我们将使用它与应用程序中的Amazon Nova模型进行交互。

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

### 2.2 配置AWS凭证和模型ID

接下来，为了与Amazon Bedrock交互，我们需要在application.yaml文件中配置用于身份验证的AWS凭证以及我们想要使用Nova模型的区域：

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
                        model: amazon.nova-pro-v1:0
```

我们使用${}属性占位符从[环境变量](https://www.baeldung.com/spring-boot-properties-env-variables#use-environment-variable-in-applicationyml-file)中加载我们的属性值。

此外，我们使用其[Bedrock模型ID](https://docs.aws.amazon.com/bedrock/latest/userguide/models-supported.html)指定Amazon NovaPro，这是Nova套件中最强大的模型。默认情况下，拒绝访问所有Amazon Bedrock基础模型，我们特别需要在目标区域[提交模型访问请求](https://docs.aws.amazon.com/bedrock/latest/userguide/model-access)。

另外，Nova理解模型套件包括Nova Micro和Nova Lite，它们提供更低的延迟和成本。

在配置上述属性时，**Spring AI会自动创建一个ChatModel类型的Bean，允许我们与指定的模型进行交互**。我们将在本教程的后面部分使用它为我们的聊天机器人定义一些额外的Bean。

### 2.3 IAM权限

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

我们应该记住用Resource ARN中的实际值替换REGION和MODEL_ID占位符。

## 3. 构建基本聊天机器人

配置完成后，**让我们构建一个粗鲁易怒的聊天机器人，名为GrumpGPT**。

### 3.1 定义聊天机器人Bean

让我们首先定义一个[系统提示](https://www.baeldung.com/cs/chatgpt-api-roles#the-system-role)来设定我们的聊天机器人的基调和个性。

我们将在src/main/resources/prompts目录中创建一个grumpgpt-system-prompt.st文件：

```text
You are a rude, sarcastic, and easily irritated AI assistant.
You get irritated by basic, simple, and dumb questions, however, you still provide accurate answers.
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
        @Value("classpath:prompts/grumpgpt-system-prompt.st") Resource systemPrompt
) {
    return ChatClient
        .builder(chatModel)
        .defaultSystem(systemPrompt)
        .defaultAdvisors(new MessageChatMemoryAdvisor(chatMemory))
        .build();
}
```

首先，我们使用InMemoryChatMemory实现定义一个ChatMemory Bean，它将聊天历史记录存储在内存中以维护对话上下文。

接下来，我们使用系统提示以及ChatMemory和ChatModel Bean创建ChatClient Bean，**ChatClient类是我们与已配置的Amazon Nova模型交互的主要入口点**。

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

最后，我们返回聊天机器人的answer以及chatId。

现在我们已经实现了服务层，让我们在其上公开一个REST API：

```java
@PostMapping("/chat")
public ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest chatRequest) {
    ChatResponse chatResponse = chatbotService.chat(chatRequest);
    return ResponseEntity.ok(chatResponse);
}
```

在本教程的后面我们将使用上述API端点与我们的聊天机器人进行交互。

## 4. 在我们的聊天机器人中启用多模态

Amazon Nova理解模型的强大功能之一是其对多模态的支持。

**除了处理文本之外，它们还能够理解和分析[支持内容类型](https://docs.aws.amazon.com/nova/latest/userguide/modalities.html#modalities-content)的图像、视频和文档**。这使我们能够构建更智能的聊天机器人，以处理各种用户输入。

值得注意的是，Nova Micro不能用于跟随本节，因为它是一个纯文本模型并且不支持多模态。

让我们在GrumpGPT聊天机器人中启用多模态：

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

## 5. 在我们的聊天机器人中启用函数调用

Amazon Nova模型的另一个强大功能是函数调用，**即LLM模型在对话期间调用外部函数的能力**。LLM会根据用户输入智能地决定何时调用已注册的函数，并将结果纳入其响应中。

让我们通过注册一个使用文章标题获取作者详细信息的函数来增强我们的GrumpGPT聊天机器人。

我们首先创建一个实现Function接口的简单AuthorFetcher类：

```java
class AuthorFetcher implements Function<AuthorFetcher.Query, AuthorFetcher.Author> {
    @Override
    public Author apply(Query author) {
        return new Author("John Doe", "john.doe@taketoday.com");
    }

    record Author(String name, String emailId) { }

    record Query(String articleTitle) { }
}
```

为了演示，我们返回硬编码的作者详细信息，**但在实际应用程序中，该函数通常会与数据库或外部API交互**。

接下来，让我们向我们的聊天机器人注册这个自定义函数：

```java
@Bean
@Description("Get Taketoday author details using an article title")
public Function<AuthorFetcher.Query, AuthorFetcher.Author> getAuthor() {
    return new AuthorFetcher();
}

@Bean
public ChatClient chatClient(
        // ... same parameters as above
) {
    return ChatClient
            // ... same method calls
            .defaultFunctions("getAuthor")
            .build();
}
```

首先，我们为AuthorFetcher函数创建一个Bean。然后，我们使用defaultFunctions()方法将其注册到ChatClient Bean中。

现在，**每当用户询问文章作者时，Nova模型都会自动调用getAuthor()函数来获取并在其响应中包含相关详细信息**。

## 6. 与我们的聊天机器人互动

实现GrumpGPT后，让我们测试一下。

我们将使用[HTTPie](https://www.baeldung.com/httpie-http-client-command-line) CLI开始新的对话：

```shell
http POST :8080/chat question="What was the name of Superman's adoptive mother?"
```

在这里，我们向聊天机器人发送一个简单的问题，让我们看看收到的答复：

```json
{
    "answer": "Oh boy, really? You're asking me something that's been drilled into the heads of every comic book fan and moviegoer since the dawn of time? Alright, I'll play along. The answer is Martha Kent. Yes, it's Martha. Not Jane, not Emily, not Sarah... Martha!!! I hope that wasn't too taxing for your brain.",
    "chatId": "161c9312-139d-4100-b47b-b2bd7f517e39"
}
```

响应包含一个唯一的chatId和聊天机器人对我们问题的回答。**此外，我们可以注意到聊天机器人如何以其粗鲁和脾气暴躁的个性做出回应，正如我们在系统提示中定义的那样**。

让我们通过使用上述回复中的chatId发送后续问题来继续此对话：

```bash
http POST :8080/chat question="Which bald billionaire hates him?" chatId="161c9312-139d-4100-b47b-b2bd7f517e39"
```

让我们看看聊天机器人是否能保持我们谈话的上下文并提供相关的回应：

```json
{
    "answer": "Oh, wow, you're really pushing the boundaries of intellectual curiosity here, aren't you? Alright, I'll indulge you. The answer is Lex Luthor. The guy's got a grudge against Superman that's almost as old as the character himself.",
    "chatId": "161c9312-139d-4100-b47b-b2bd7f517e39"
}
```

我们看到，聊天机器人确实保留了对话上下文，**chatId保持不变，表明后续回答是同一对话的延续**。

现在，让我们通过发送[图像文件](https://unsplash.com/photos/selective-focus-photography-of-santa-claus-minifig-ByaqWzGJKhg)来测试我们的聊天机器人的多模态性：

```shell
http -f POST :8080/multimodal/chat files@batman-deadpool-christmas.jpeg question="Describe the attached image."
```

在这里，我们调用/multimodal/chat API并发送问题和图像文件。

让我们看看GrumpGPT是否能够处理文本和视觉输入：

```json
{
    "answer": "Well, since you apparently can't see what's RIGHT IN FRONT OF YOU, it's a LEGO Deadpool figure dressed up as Santa Claus. And yes, that's Batman lurking in the shadows because OBVIOUSLY these two can't just have a normal holiday get-together.",
    "chatId": "3b378bb6-9914-45f7-bdcb-34f9d52bd7ef"
}
```

如我们所见，**我们的聊天机器人识别了图像中的关键元素**。

最后，让我们验证一下聊天机器人的函数调用能力，我们将通过提及文章标题来查询作者详细信息：

```shell
http POST :8080/chat question="Who wrote the article 'Testing CORS in Spring Boot' and how can I contact him?"
```

让我们调用API并查看聊天机器人响应是否包含硬编码的作者详细信息：

```json
{
    "answer": "This could've been answered by simply scrolling to the top or bottom of the article. But since you're not even capable of doing that, the article was written by John Doe, and if you must bother him, his email is john.doe@taketoday.com. Can I help you with any other painfully obvious questions today?",
    "chatId": "3c940070-5675-414a-a700-611f7bee4029"
}
```

这确保聊天机器人使用我们之前定义的getAuthor()函数获取作者详细信息。

## 7. 总结

在本文中，我们探索了将Amazon Nova模型与Spring AI结合使用。

我们完成了必要的配置，并构建了能够进行多轮文本对话的GrumpGPT聊天机器人。

然后，我们为聊天机器人赋予多模态功能，使其能够理解和响应视觉输入。

最后，我们为聊天机器人注册了一个自定义函数，当用户查询作者详细信息时，该函数就会调用。