---
layout: post
title: OpenAI API Java客户端
category: libraries
copyright: libraries
excerpt: OpenAI
---

## 1. 简介

在本文中，我们将介绍集成OpenAI的Java客户端API的过程。

我们将首先在开发环境中设置Java客户端，验证我们的API请求，并演示如何与OpenAI模型交互以进行文本生成和AI驱动的任务。

## 2. 依赖

首先，我们必须导入项目所需的依赖，可以在[Maven仓库](https://mvnrepository.com/artifact/com.openai/openai-java)中找到这些库：

```xml
<dependency>
    <groupId>com.openai</groupId>
    <artifactId>openai-java</artifactId>
    <version>0.22.0</version>
</dependency>
```

## 3. 学习助手

我们将构建一个工具，旨在帮助我们根据Taketoday上的文章和教程创建个性化课程。

**虽然互联网提供了大量资源，但将这些信息组织成连贯的学习路径可能很困难**。学习新主题很快就会变得令人不知所措，因为很难找到最有效的资源并过滤掉不相关的内容。

**为了解决这个问题，我们将开发一个与ChatGPT交互的简单客户端**，该客户端将允许我们浏览大量Taketoday文章，并接收针对我们的学习目标量身定制的指导建议。

## 4. OpenAI API Token

**获取API令牌是将我们的应用程序连接到OpenAI API的第一步**，此令牌是用于验证我们向OpenAI服务器发出的请求的密钥。

要生成此令牌，我们必须登录我们的OpenAI帐户并从[API设置页面](https://platform.openai.com/api-keys)创建一个新的API Key：

![](/assets/images/2025/libraries/javaopenaiapiclient01.png)

获取令牌后，我们可以在Java应用程序中使用它与OpenAI的模型进行安全通信，确保我们的代码不会暴露API令牌。

对于此示例，我们将在IDE(如IntelliJ IDEA)中配置[环境变量](https://www.baeldung.com/intellij-idea-environment-variables)。这适用于单个实验；然而，在生产中，我们更有可能使用机密管理工具(如[AWS Secrets Manager](https://www.baeldung.com/spring-boot-integrate-aws-secrets-manager)、[HashiCorp Vault](https://www.baeldung.com/vault)或[Azure Key Vault)](https://www.baeldung.com/spring-cloud-azure-key-vault)来安全地存储和管理我们的API密钥。

**我们可以生成两种类型的令牌：个人和服务账户**。个人令牌的含义不言自明，服务账户令牌用于连接到我们的OpenAI项目的机器人或应用程序。虽然两者都有效，但个人令牌足以满足我们的目的。

## 5. OpenAIClient

一旦我们将令牌设置到环境变量中，我们就会初始化一个OpenAIClient实例，这使我们能够与API交互并接收来自ChatGPT的响应：

```java
OpenAIClient client = OpenAIOkHttpClient.fromEnv();
```

现在我们已经初始化了客户端，让我们深入了解完成和助手API。

## 6. Completion API与Assistants API

**让我们首先考虑一下[Completion API](https://platform.openai.com/docs/guides/text-generation)和[Assistants API](https://platform.openai.com/docs/assistants/overview)之间的区别，以帮助我们确定哪个最适合不同的任务**。

Completion API方法非常适合执行一些简单任务，例如生成简短响应、编写代码片段或制作内容部分。它还非常适合涉及特定问题或请求并产生简洁答案的单轮互动。

Assistants API可处理需要持续交互的复杂问题，适用于多种专业任务，例如编写整本书、开发软件应用程序或管理复杂的工作流程。它提供了文件搜索、代码解释器和[函数调用](https://www.baeldung.com/spring-ai-mistral-api-function-calling)等工具。

Completion API适合快速任务，而Assistants API则以结构化的方式支持长期项目。

## 7. 简单Completion

使用Completion API，我们利用ChatCompletionCreateParams创建请求。最低设置[需要](https://platform.openai.com/docs/guides/text-generation/chat-completions-api)选择模型和消息列表，但它也可以接受温度、最大令牌和其他调整选项。

现在让我们看看如何单独使用addDeveloperMessage()和addUserMessage()方法：

```java
Builder createParams = ChatCompletionCreateParams.builder()
    .model(ChatModel.GPT_4O_MINI)
    .addDeveloperMessage("You're helping me to create a curriculum to learn programming. Use only the articles from www.baeldung.com")
    .addUserMessage(userMessage);
```

或者，我们可以将这些消息作为ChatCompletionMessageParam实例或JSON格式提供。

**简单Completion用法很简单-AI模型接收消息，经过处理后，它会以我们可以流式传输和打印的文本结构进行响应**：

```java
client.chat()
    .completions()
    .create(createParams.build())
    .choices()
    .stream()
    .flatMap(choice -> choice.message()
        .content()
        .stream())
  .forEach(System.out::println);
```

## 8. 对话Completion

 **对话功能允许我们与AI进行持续对话，直到我们退出程序**。

让我们观察一下在模型和消息保持不变的情况下响应和交互如何变化：

```java
do {

    List<ChatCompletionMessage> messages = client.chat()
        .completions()
        .create(createParamsBuilder.build())
        .choices()
        .stream()
        .map(ChatCompletion.Choice::message)
        .toList();

    messages.stream()
        .flatMap(message -> message.content().stream())
        .forEach(System.out::println);

    System.out.println("-----------------------------------");
    System.out.println("Anything else you would like to know? Otherwise type EXIT to stop the program.");

    String userMessageConversation = scanner.next();

    if ("exit".equalsIgnoreCase(userMessageConversation)) {
        scanner.close();
        return;
    }

    messages.forEach(createParamsBuilder::addMessage);
    createParamsBuilder
        .addDeveloperMessage("Continue providing help following the same rules as before.")
        .addUserMessage(userMessageConversation);

} while (true);
```

## 9. Assistants

现在，让我们看看如何使用具有最少必需参数的Assistant类。[Assistant](https://www.baeldung.com/spring-ai-assistant)支持文件搜索、代码解释器和函数调用等工具。

**首先，我们必须用名称和模型初始化Assistant，提供指令，分配角色，提供内容，并将其分配给Thread和Run。这些是强制性的，特定于OpenAI，并促进AI交互**：

```java
Assistant assistant = client.beta()
    .assistants()
    .create(BetaAssistantCreateParams.builder()
        .name("Taketoday Tutor")
        .instructions("You're a personal programming tutor specialized in research online learning courses.")
        .model(ChatModel.GPT_4O_MINI)
        .build());

Thread thread = client.beta()
    .threads()
    .create(BetaThreadCreateParams.builder().build());

client.beta()
    .threads()
    .messages()
    .create(BetaThreadMessageCreateParams.builder()
        .threadId(thread.id())
        .role(BetaThreadMessageCreateParams.Role.USER)
        .content("I want to learn about Strings")
        .build());

Run run = client.beta()
    .threads()
    .runs()
    .create(BetaThreadRunCreateParams.builder()
        .threadId(thread.id())
        .assistantId(assistant.id())
        .instructions("You're helping me to create a curriculum to learn programming. Use only the articles from www.taketoday.com")
        .build());
```

在OpenAI的API中，Thread代表用户与助手之间的一系列交互。使用Assistants API时，我们会创建一个Thread来维护多个交换之间的上下文。

Run是助手处理用户输入并生成响应的执行实例，每个Run都属于一个Thread，并经历不同的状态。

**接下来，我们执行新创建的Thread并循环，直到AI完成分配的任务，继续，直到状态发生变化并不同于QUEUED或IN_PROGRESS**：

```java
while (run.status().equals(RunStatus.QUEUED) || run.status().equals(RunStatus.IN_PROGRESS)) {
    System.out.println("Polling run...");
    java.lang.Thread.sleep(500);
    run = client.beta()
        .threads()
        .runs()
        .retrieve(BetaThreadRunRetrieveParams.builder()
            .threadId(thread.id())
            .runId(run.id())
            .build());
}
```

**现在状态已更改为COMPLETED，我们可以继续处理响应流并打印消息**：

```java
System.out.println("Run completed with status: " + run.status() + "\n");

if (!run.status().equals(RunStatus.COMPLETED)) {
    return;
}

BetaThreadMessageListPage page = client.beta()
    .threads()
    .messages()
    .list(BetaThreadMessageListParams.builder()
        .threadId(thread.id())
        .order(BetaThreadMessageListParams.Order.ASC)
        .build());

page.autoPager()
    .stream()
    .forEach(currentMessage -> {
        System.out.println(currentMessage.role());
        currentMessage.content()
            .stream()
            .flatMap(content -> content.text().stream())
            .map(textBlock -> textBlock.text().value())
            .forEach(System.out::println);
        System.out.println();
    });
```

在处理大型数据集时，OpenAI的API会以页面形式返回结果。autoPager()有助于自动处理分页响应，使我们能够有效地检索和处理多个结果，而无需手动遍历页面。

**最后，当不再需要时，我们可以删除Assistant，或者在未来的任务中重新使用相同的assistant.id()**：

```java
AssistantDeleted assistantDeleted = client.beta()
    .assistants()
    .delete(BetaAssistantDeleteParams.builder()
        .assistantId(assistant.id())
        .build());

System.out.println("Assistant deleted: " + assistantDeleted.deleted());
```

值得注意的是，**带有“Beta”前缀的类别表示OpenAI API中的实验性或早期功能**，这些功能可能会发生变化，因此我们应在生产应用程序中谨慎使用它们。

## 10. 总结

将OpenAI的API集成到我们的Java应用程序中，使我们能够创建提高生产力和学习能力的工具。我们可以自动化工作流程并开发响应式应用程序，为更多AI创新铺平道路。

接下来是[LangChain4j与Quarkus](https://www.baeldung.com/java-quarkus-langchain4j)和[Spring AI Assistant](https://www.baeldung.com/spring-ai-assistant)的强大抽象，可简化使用Java中的LLM的工作。这些框架最大限度地减少了对单一供应商的依赖，并实现了灵活的AI集成。