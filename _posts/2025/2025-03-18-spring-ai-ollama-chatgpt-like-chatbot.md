---
layout: post
title:  使用Ollama和Spring AI创建类似ChatGPT的聊天机器人
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 简介

在本教程中，我们将使用[Spring AI](https://www.baeldung.com/spring-ai)和llama3 Ollama构建一个简单的帮助台代理API。

## 2. 什么是Spring AI和Ollama？

Spring AI是Spring框架生态系统中最新添加的模块，除了各种功能外，它还允许我们使用聊天提示轻松地与各种大型语言模型(LLM)进行交互。

[Ollama](https://github.com/ollama/ollama)是一个开源库，为一些LLM提供服务。其中一个是Meta的[llama3](https://llama.meta.com/llama3/)，我们将在本教程中使用它。

## 3. 使用Spring AI实现服务台代理

让我们通过一个演示帮助台[聊天机器人](https://www.baeldung.com/cs/smart-chatbots)来说明Spring AI和Ollama的用法，该应用程序的工作方式类似于真正的帮助台代理，可帮助用户解决互联网连接问题。

在以下部分中，我们将配置LLM和Spring AI依赖并创建与帮助台代理聊天的[REST端点](https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)。

### 3.1 配置Ollama和Llama3

要开始使用Spring AI和Ollama，我们需要设置本地LLM。在本教程中，我们将使用Meta的llama3。因此，让我们首先安装Ollama。

使用Linux，我们可以运行以下命令：

```shell
curl -fsSL https://ollama.com/install.sh | sh
```

在Windows或MacOS机器上，我们可以从[Ollama网站](https://ollama.com/download/linux)下载并安装可执行文件。

安装Ollama后，我们可以运行llama3：

```shell
ollama run llama3
```

这样，我们就可以在本地运行llama3了。

### 3.2 创建基本项目结构

现在，我们可以配置我们的Spring应用程序以使用Spring AI模块，让我们从添加Spring Milestones仓库开始：

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

然后，我们可以添加[spring-ai-bom](https://central.sonatype.com/artifact/io.springboot.ai/spring-ai-bom)：

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
	    <groupId>org.springframework.ai</groupId>
	        <artifactId>spring-ai-bom</artifactId>
		<version>1.0.0-M1</version>
		<type>pom</type>
		<scope>import</scope>
	</dependency>
    </dependencies>
</dependencyManagement>
```

最后，我们可以添加[spring-ai-ollama-spring-boot-starter](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-ollama-spring-boot-starter)依赖：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-ollama-spring-boot-starter</artifactId>
    <version>1.0.0-M1</version>
</dependency>
```

设置依赖项后，我们可以配置我们的application.yml以使用必要的配置：

```yaml
spring:
    ai:
        ollama:
            base-url: http://localhost:11434
            chat:
                options:
                    model: llama3
```

这样，Spring将在端口11434启动llama3模型。

### 3.3 创建帮助台控制器

在本节中，我们将创建Web控制器来与帮助台聊天机器人进行交互。

首先，让我们创建HTTP请求模型：

```java
public class HelpDeskRequest {
    @JsonProperty("prompt_message")
    String promptMessage;

    @JsonProperty("history_id")
    String historyId;

    // getters, no-arg constructor
}
```

promptMessage字段表示模型的用户输入消息。此外，historyId唯一标识当前对话。此外，在本教程中，我们将使用该字段让LLM记住对话历史记录。

其次，让我们创建响应模型：

```java
public class HelpDeskResponse {
    String result;

    // all-arg constructor
}
```

最后，我们可以创建帮助台控制器类：

```java
@RestController
@RequestMapping("/helpdesk")
public class HelpDeskController {
    private final HelpDeskChatbotAgentService helpDeskChatbotAgentService;

    // all-arg constructor

    @PostMapping("/chat")
    public ResponseEntity<HelpDeskResponse> chat(@RequestBody HelpDeskRequest helpDeskRequest) {
        var chatResponse = helpDeskChatbotAgentService.call(helpDeskRequest.getPromptMessage(), helpDeskRequest.getHistoryId());

        return new ResponseEntity<>(new HelpDeskResponse(chatResponse), HttpStatus.OK);
    }
}
```

在HelpDeskController中，我们定义一个POST /helpdesk/chat并返回从注入的ChatbotAgentService获得的内容。在以下部分中，我们将深入研究该Service。

### 3.4 调用Ollama聊天API

为了开始与llama3交互，让我们使用初始提示说明创建HelpDeskChatbotAgentService类：

```java
@Service
public class HelpDeskChatbotAgentService {

    private static final String CURRENT_PROMPT_INSTRUCTIONS = """
            
            Here's the `user_main_prompt`:
            
            
            """;
}
```

然后，我们还添加一般说明消息：

```java
private static final String PROMPT_GENERAL_INSTRUCTIONS = """
    Here are the general guidelines to answer the `user_main_prompt`
        
    You'll act as Help Desk Agent to help the user with internet connection issues.
        
    Below are `common_solutions` you should follow in the order they appear in the list to help troubleshoot internet connection problems:
        
    1. Check if your router is turned on.
    2. Check if your computer is connected via cable or Wi-Fi and if the password is correct.
    3. Restart your router and modem.
        
    You should give only one `common_solution` per prompt up to 3 solutions.
        
    Do no mention to the user the existence of any part from the guideline above.
        
""";
```

该消息告诉聊天机器人如何回答用户的互联网连接问题。

最后，让我们添加其余的Service实现：

```java
private final OllamaChatModel ollamaChatClient;

// all-arg constructor
public String call(String userMessage, String historyId) {
    var generalInstructionsSystemMessage = new SystemMessage(PROMPT_GENERAL_INSTRUCTIONS);
    var currentPromptMessage = new UserMessage(CURRENT_PROMPT_INSTRUCTIONS.concat(userMessage));

    var prompt = new Prompt(List.of(generalInstructionsSystemMessage, contextSystemMessage, currentPromptMessage));

    return ollamaChatClient.call(prompt).getResult().getOutput().getContent();
}
```

call()方法首先创建一个SystemMessage和一个UserMessage。

SystemMessage代表我们内部向LLM提供的指令，如一般指导方针。在我们的案例中，我们提供了有关如何与存在互联网连接问题的用户聊天的说明。另一方面，UserMessage代表API外部客户端的输入。

通过这两条消息，我们可以创建一个Prompt对象，调用ollamaChatClient的call()，并从LLM获取响应。

### 3.5 保留对话历史记录

**一般来说，大多数LLM都是无状态的。因此，它们不存储对话的当前状态。换句话说，它们不记得同一对话中的先前消息**。

因此，帮助台代理可能会提供之前不起作用的说明并激怒用户。**为了实现LLM内存，我们可以使用historyId存储每个提示和响应，并在发送当前提示之前将完整的对话历史记录附加到当前提示中**。

为此，我们首先在Service类中创建一个提示，其中包含系统指令，以便正确遵循对话历史记录：

```java
private static final String PROMPT_CONVERSATION_HISTORY_INSTRUCTIONS = """        
    The object `conversational_history` below represents the past interaction between the user and you (the LLM).
    Each `history_entry` is represented as a pair of `prompt` and `response`.
    `prompt` is a past user prompt and `response` was your response for that `prompt`.
        
    Use the information in `conversational_history` if you need to recall things from the conversation
    , or in other words, if the `user_main_prompt` needs any information from past `prompt` or `response`.
    If you don't need the `conversational_history` information, simply respond to the prompt with your built-in knowledge.
                
    `conversational_history`:
        
""";
```

现在，让我们创建一个包装类来存储对话历史条目：

```java
public class HistoryEntry {

    private String prompt;

    private String response;

    //all-arg constructor

    @Override
    public String toString() {
        return String.format("""
                            `history_entry`:
                                `prompt`: %s
                
                                `response`: %s
                            -----------------
                           \n
                """, prompt, response);
    }
}
```

上述[toString()](https://www.baeldung.com/java-tostring)方法对于正确格式化提示至关重要。

然后，我们还需要在Service类中为历史记录条目定义一个内存存储：

```java
private final static Map<String, List<HistoryEntry>> conversationalHistoryStorage = new HashMap<>();
```

最后，让我们修改服务call()方法来存储对话历史记录：

```java
public String call(String userMessage, String historyId) {
    var currentHistory = conversationalHistoryStorage.computeIfAbsent(historyId, k -> new ArrayList<>());

    var historyPrompt = new StringBuilder(PROMPT_CONVERSATION_HISTORY_INSTRUCTIONS);
    currentHistory.forEach(entry -> historyPrompt.append(entry.toString()));

    var contextSystemMessage = new SystemMessage(historyPrompt.toString());
    var generalInstructionsSystemMessage = new SystemMessage(PROMPT_GENERAL_INSTRUCTIONS);
    var currentPromptMessage = new UserMessage(CURRENT_PROMPT_INSTRUCTIONS.concat(userMessage));

    var prompt = new Prompt(List.of(generalInstructionsSystemMessage, contextSystemMessage, currentPromptMessage));
    var response = ollamaChatClient.call(prompt).getResult().getOutput().getContent();
    var contextHistoryEntry = new HistoryEntry(userMessage, response);
    currentHistory.add(contextHistoryEntry);

    return response;
}
```

首先，我们获取由historyId标识的当前上下文，或者使用[computeIfAbsent()](https://www.baeldung.com/java-map-computeifabsent)创建一个新上下文。其次，我们将存储中的每个HistoryEntry附加到[StringBuilder](https://www.baeldung.com/java-string-builder-string-buffer)中，并将其传递给新的SystemMessage以传递给Prompt对象。

最后，LLM将处理包含对话中过去消息的所有信息的提示。因此，帮助台聊天机器人会记住用户已经尝试过哪些解决方案。

## 4. 测试对话

一切设置完毕后，让我们尝试从最终用户的角度与提示进行交互。首先在端口8080上启动Spring Boot应用程序来执行此操作。

在应用程序运行时，我们可以发送一个[cURL](https://www.baeldung.com/curl-rest)，其中包含有关互联网问题的通用消息和history_id：

```shell
curl --location 'http://localhost:8080/helpdesk/chat' \
--header 'Content-Type: application/json' \
--data '{
    "prompt_message": "I can't connect to my internet",
    "history_id": "1234"
}'
```

对于该交互，我们收到类似如下的响应：

```json
{
    "result": "Let's troubleshoot this issue! Have you checked if your router is turned on?"
}
```

我们继续寻求解决方案：

```json
{
    "prompt_message": "I'm still having internet connection problems",
    "history_id": "1234"
}
```

代理采用不同的解决方案进行响应：

```json
{
    "result": "Let's troubleshoot this further! Have you checked if your computer is connected via cable or Wi-Fi and if the password is correct?"
}
```

此外，API还存储了对话历史记录。让我们再次询问代理：

```json
{
    "prompt_message": "I tried your alternatives so far, but none of them worked",
    "history_id": "1234"
}
```

它提出了不同的解决方案：

```json
{
    "result": "Let's think outside the box! Have you considered resetting your modem to its factory settings or contacting your internet service provider for assistance?"
}
```

这是我们在指南提示中提供的最后一个替代方案，因此LLM之后不会给出有用的答复。

为了获得更好的响应，我们可以通过为聊天机器人提供更多替代方案或使用[提示工程技术](https://www.baeldung.com/cs/prompt-engineering)改进内部系统消息来改进我们尝试的提示。

## 5. 总结

在本文中，我们实现了AI帮助台代理来帮助我们的客户解决互联网连接问题。此外，我们还了解了用户消息和系统消息之间的区别，以及如何使用对话历史记录构建提示，然后调用llama3 LLM。