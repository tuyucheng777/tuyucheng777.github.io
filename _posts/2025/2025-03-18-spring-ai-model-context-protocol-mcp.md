---
layout: post
title:  使用Spring AI探索模型上下文协议(MCP)
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

现代Web应用程序越来越多地与[大型语言模型(LLM)](https://www.baeldung.com/cs/large-language-models)相结合来构建解决方案，而这些解决方案不仅限于基于一般知识的问答。

为了增强AI模型的响应能力并使其更能感知环境，我们可以将其连接到搜索引擎、数据库和文件系统等外部源。但是，集成和管理具有不同格式和协议的多个数据源是一项挑战。

[Anthropic](https://www.anthropic.com/)推出的模型上下文协议(MCP)解决了这一集成挑战，并提供了一种将AI驱动的应用程序与外部数据源连接起来的标准化方法。通过MCP，我们可以在原生LLM之上构建复杂的代理和工作流。

**在本教程中，我们将通过使用Spring AI实际实现其客户端-服务器架构来理解MCP的概念**。我们将创建一个简单的聊天机器人，并通过MCP服务器扩展其功能以执行Web搜索、执行文件系统操作和访问自定义业务逻辑。

## 2. 模型上下文协议101

在深入实现之前，让我们仔细看看MCP及其各个组件：

![](/assets/images/2025/springai/springaimodelcontextprotocolmcp01.png)

MCP遵循客户端-服务器架构，围绕几个关键组件：

- **MCP主机**：是我们的主要应用程序，它与LLM集成并需要它与外部数据源连接
- **MCP客户端**：是与MCP服务器建立并维持1:1连接的组件
- **MCP服务器**：是与外部数据源集成并公开与其交互功能的组件
- **工具**：指MCP服务器向客户端开放的可执行函数/方法

此外，为了处理客户端和服务器之间的通信，MCP提供了两个传输通道。

为了通过标准输入和输出流与本地进程和命令行工具进行通信，它提供了标准输入/输出(stdio)传输类型。或者，对于客户端和服务器之间的基于HTTP的通信，它提供了服务器发送事件(SSE)传输类型。

MCP是一个复杂而庞大的主题，请参阅[官方文档](https://modelcontextprotocol.io/introduction)以了解更多信息。

## 3. 创建MCP主机

现在我们已经对MCP有了高层次的了解，让我们开始实际实现MCP架构。

**我们将使用[Anthropic Claude](https://www.baeldung.com/spring-ai-anthropics-claude-models)模型构建一个聊天机器人，它将充当我们的MCP主机**。或者，我们可以通过[Hugging Face或Ollama](https://www.baeldung.com/spring-ai-ollama-hugging-face-models)使用本地LLM，因为特定的AI模型与此演示无关。

### 3.1 依赖

让我们首先向项目的pom.xml文件中添加必要的依赖项：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-anthropic-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mcp-client-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

[Anthropic Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-anthropic-spring-boot-starter)是[Anthropic Message API](https://docs.anthropic.com/en/api/messages)的包装器，我们将使用它在我们的应用程序中与Claude模型进行交互。

此外，我们导入了[MCP客户端Starter依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-mcp-client-spring-boot-starter)，**这将允许我们在Spring Boot应用程序内配置与MCP服务器保持1:1连接的客户端**。

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

鉴于我们在项目中使用了多个Spring AI Starter，我们还将在pom.xml中包含[Spring AI BOM](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-bom)：

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

通过此添加，我们现在可以从两个Starter依赖中删除version标签。**BOM消除了版本冲突的风险，并确保我们的Spring AI依赖彼此兼容**。

接下来，让我们在application.yaml文件中配置我们的[Anthropic API Key](https://console.anthropic.com/settings/keys)和聊天模型：

```yaml
spring:
    ai:
        anthropic:
            api-key: ${ANTHROPIC_API_KEY}
            chat:
                options:
                    model: claude-3-7-sonnet-20250219
```

我们使用${}属性占位符从[环境变量](https://www.baeldung.com/spring-boot-properties-env-variables#use-environment-variable-in-applicationyml-file)中加载API Key的值。

此外，我们指定了Anthropic最智能的模型[Claude 3.7 Sonnet](https://www.anthropic.com/news/claude-3-7-sonnet)，使用claude-3-7-sonnet-20250219模型ID，你可以根据需要随意探索和使用[其他模型](https://docs.anthropic.com/en/docs/about-claude/models#model-names)。

配置上述属性后，**Spring AI会自动创建一个ChatModel类型的Bean，允许我们与指定的模型进行交互**。

### 3.2 为Brave Search和Filesystem 服务器配置MCP客户端

**现在，让我们为两个预构建的MCP服务器实现配置MCP客户端：[Brave Search](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search)和[Filesystem](https://github.com/modelcontextprotocol/servers/tree/main/src/filesystem)**，这些服务器将使我们的聊天机器人能够执行Web搜索和文件系统操作。

让我们首先在application.yaml文件中为Brave搜索MCP服务器注册一个MCP客户端：

```yaml
spring:
    ai:
        mcp:
            client:
                stdio:
                    connections:
                        brave-search:
                            command: npx
                            args:
                                - "-y"
                                - "@modelcontextprotocol/server-brave-search"
                            env:
                                BRAVE_API_KEY: ${BRAVE_API_KEY}
```

在这里，我们配置一个带有stdio传输的客户端，**我们指定[npx](https://docs.npmjs.com/cli/v11/commands/npx)命令来下载并运行基于TypeScript的[@modelcontextprotocol/server-brave-search](https://www.npmjs.com/package/@modelcontextprotocol/server-brave-search)包，并使用-y标志确认所有安装提示**。

此外，我们还提供[BRAVE_API_KEY](https://api-dashboard.search.brave.com/app/keys)作为环境变量。

接下来，让我们为文件系统MCP服务器配置一个MCP客户端：

```yaml
spring:
    ai:
        mcp:
            client:
                stdio:
                    connections:
                        filesystem:
                            command: npx
                            args:
                                - "-y"
                                - "@modelcontextprotocol/server-filesystem"
                                - "./"
```

与前面的配置类似，我们指定运行[Filesystem MCP服务器包](https://www.npmjs.com/package/@modelcontextprotocol/server-filesystem)所需的命令和参数，此设置允许我们的聊天机器人执行在指定目录中创建、读取和写入文件等操作。

这里，我们只配置当前目录(./)用于文件系统操作，但是，我们可以通过将多个目录添加到args列表来指定多个目录。

在应用程序启动期间，Spring AI将扫描我们的配置，创建MCP客户端，并与相应的MCP服务器建立连接。**它还会创建一个SyncMcpToolCallbackProvider类型的Bean，该Bean提供已配置的MCP服务器公开的所有工具的列表**。

### 3.3 构建基本聊天机器人

配置好AI模型和MCP客户端后，让我们构建一个简单的聊天机器人：

```java
@Bean
ChatClient chatClient(ChatModel chatModel, SyncMcpToolCallbackProvider toolCallbackProvider) {
    return ChatClient
        .builder(chatModel)
        .defaultTools(toolCallbackProvider.getToolCallbacks())
        .build();
}
```

我们首先使用ChatModel和SyncMcpToolCallbackProvider Bean创建ChatClient类型的Bean，**ChatClient类将作为我们与聊天完成模型(即Claude 3.7 Sonnet)交互的主要入口点**。

接下来，让我们注入ChatClient Bean来创建一个新的ChatbotService类：

```java
String chat(String question) {
    return chatClient
        .prompt()
        .user(question)
        .call()
        .content();
}
```

我们创建一个chat()方法，将用户的question传递给聊天客户端Bean，并简单地返回AI模型的响应。

现在我们已经实现了服务层，让我们在其上公开一个REST API：

```java
@PostMapping("/chat")
ResponseEntity<ChatResponse> chat(@RequestBody ChatRequest chatRequest) {
    String answer = chatbotService.chat(chatRequest.question());
    return ResponseEntity.ok(new ChatResponse(answer));
}

record ChatRequest(String question) {}

record ChatResponse(String answer) {}
```

在本教程的后面我们将使用上述API端点与我们的聊天机器人进行交互。

## 4. 创建自定义MCP服务器

除了使用预先构建的MCP服务器之外，**我们还可以创建自己的MCP服务器，以使用我们的业务逻辑扩展聊天机器人的功能**。

让我们探索如何使用Spring AI创建自定义MCP服务器。

我们将在本节中创建一个新的Spring Boot应用程序。

### 4.1 依赖

首先，让我们在pom.xml文件中包含必要的依赖项：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mcp-server-webmvc-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

我们导入[Spring AI的MCP服务器依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-mcp-server-webmvc-spring-boot-starter)，它提供了创建支持基于HTTP的SSE传输的自定义MCP服务器所需的类。

### 4.2 定义和公开自定义工具

接下来，让我们定义一些MCP服务器将公开的自定义工具。

我们将创建一个AuthorRepository类，提供获取作者详细信息的方法：

```java
class AuthorRepository {
    @Tool(description = "Get Taketoday author details using an article title")
    Author getAuthorByArticleTitle(String articleTitle) {
        return new Author("John Doe", "john.doe@taketoday.com");
    }

    @Tool(description = "Get highest rated Taketoday authors")
    List<Author> getTopAuthors() {
        return List.of(
                new Author("John Doe", "john.doe@taketoday.com"),
                new Author("Jane Doe", "jane.doe@taketoday.com")
        );
    }

    record Author(String name, String email) {
    }
}
```

为了演示，我们返回硬编码的作者详细信息，但在实际应用程序中，这些工具通常会与数据库或外部API交互。

我们用@Tool注解标注了这两个方法，并为它们各自提供了简短的描述。**该描述可帮助AI模型根据用户输入决定是否以及何时调用工具，并将结果纳入其响应中**。

接下来，让我们将工具注册到MCP服务器上：

```java
@Bean
ToolCallbackProvider authorTools() {
    return MethodToolCallbackProvider
        .builder()
        .toolObjects(new AuthorRepository())
        .build();
}
```

我们使用MethodToolCallbackProvider从AuthorRepository类中定义的工具创建一个ToolCallbackProvider Bean，**使用@Tool标注的方法将在应用程序启动时作为MCP工具公开**。

### 4.3 为我们的自定义MCP服务器配置MCP客户端

最后，为了在我们的聊天机器人应用程序中使用我们的自定义MCP服务器，我们需要针对它配置一个MCP客户端：

```yaml
spring:
    ai:
        mcp:
            client:
                sse:
                    connections:
                        author-tools-server:
                            url: http://localhost:8081
```

在application.yaml文件中，我们针对自定义MCP服务器配置新客户端。请注意，我们在此处使用SSE传输类型。

此配置假定MCP服务器运行于http://localhost:8081，如果它运行于不同的主机或端口，请确保更新URL。

通过此配置，**除了Brave Search和Filesystem MCP服务器提供的工具之外，我们的MCP客户端现在还可以调用我们自定义服务器公开的工具**。

## 5. 与我们的聊天机器人互动

现在我们已经构建了聊天机器人并将其与各种MCP服务器集成，让我们与它进行交互并进行测试。

我们将使用[HTTPie](https://www.baeldung.com/httpie-http-client-command-line) CLI来调用聊天机器人的API端点：

```shell
http POST :8080/chat question="How much was Elon Musk's initial offer to buy OpenAI in 2025?"
```

在这里，**我们向聊天机器人发送一个简单的问题，询问LLM[知识截止](https://github.com/HaoooWang/llm-knowledge-cutoff-dates?tab=readme-ov-file)日期之后发生的事件**。让我们看看得到的答复：

```text
{
    "answer": "Elon Musk's initial offer to buy OpenAI was $97.4 billion. [Source](https://www.reuters.com/technology/openai-board-rejects-musks-974-billion-offer-2025-02-14/)."
}
```

我们可以看到，**聊天机器人能够使用配置的Brave Search MCP服务器执行Web搜索，并提供准确的答案以及来源**。

接下来，**让我们验证聊天机器人是否可以使用Filesystem MCP服务器执行文件系统操作**：

```shell
http POST :8080/chat question="Create a text file named 'mcp-demo.txt' with content 'This is awesome!'."
```

我们指示聊天机器人创建一个包含特定内容的mcp-demo.txt文件。让我们看看它是否能够满足请求：

```json
{
    "answer": "The text file named 'mcp-demo.txt' has been successfully created with the content you specified."
}
```

聊天机器人响应成功，我们可以验证该文件是否在application.yaml文件中指定的目录中创建。

最后，**让我们验证聊天机器人是否可以调用我们自定义MCP服务器公开的工具之一**。我们将通过提及文章标题来查询作者详细信息：

```shell
http POST :8080/chat question="Who wrote the article 'Testing CORS in Spring Boot?' on Taketoday, and how can I contact them?"
```

让我们调用API并查看聊天机器人响应是否包含硬编码的作者详细信息：

```json
{
    "answer": "The article 'Testing CORS in Spring Boot' on Taketoday was written by John Doe. You can contact him via email at [john.doe@taketoday.com](mailto:john.doe@taketoday.com)."
}
```

上述响应验证聊天机器人是否使用我们自定义MCP服务器公开的getAuthorByArticleTitle()工具获取作者详细信息。

我们强烈建议在本地设置代码库并使用不同的提示来尝试聊天机器人。

## 6. 总结

在本文中，我们探讨了模型上下文协议并使用Spring AI实现了其客户端-服务器架构。

首先，我们使用Anthropic的Claude 3.7 Sonnet模型构建了一个简单的聊天机器人作为我们的MCP主机。

然后，为了让我们的聊天机器人具有Web搜索功能并使其能够执行文件系统操作，我们根据Brave Search API和Filesystem的预构建MCP服务器实现配置了MCP客户端。

最后，我们创建了一个自定义MCP服务器，并在我们的MCP主机应用程序内配置了其相应的MCP客户端。