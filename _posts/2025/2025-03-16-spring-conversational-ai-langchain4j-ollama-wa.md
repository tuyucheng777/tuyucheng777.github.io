---
layout: post
title:  使用Apache Camel、LangChain4j和WhatsApp构建对话式AI
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，**我们将了解如何将[Apache Camel](https://www.baeldung.com/apache-camel-spring-boot)和[LangChain4j](https://www.baeldung.com/java-langchain-basics)集成到Spring Boot应用程序中，以处理WhatsApp上的AI驱动对话**，并使用本地安装的Ollama进行AI处理。Apache Camel处理不同系统之间的数据路由和转换，而LangChain4j提供与大型语言模型交互并提取有意义信息的工具。

我们在教程[如何在Linux上安装Ollama Generative AI](https://www.baeldung.com/linux/genai-ollama-installation)中讨论了Ollama的主要优势、安装和硬件要求。无论如何，它是跨平台的，也适用于Windows和macOS。

我们将使用[Postman](https://www.baeldung.com/java-postman)来测试Ollama API、WhatsApp API和我们的Spring Boot控制器。

## 2. Spring Boot的初始设置

首先，让我们确保本地端口8080未被使用，因为Spring Boot需要它。

由于我们将使用@RequestParam注解将请求参数绑定到Spring Boot控制器，**因此我们需要添加-parameters编译器参数**：

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <configuration>
        <source>23</source>
        <target>23</target>
        <compilerArgs>
            <arg>-parameters</arg>
        </compilerArgs>
    </configuration>
</plugin>
```

如果我们错过了它，关于参数名称的信息将无法通过反射获得，因此我们的REST调用将抛出java.lang.IllegalArgumentException。

此外，**传入和传出消息的DEBUG级别日志记录可以帮助我们**，因此让我们在application.properties中启用它：

```properties
# Logging configuration
logging.level.root=INFO
logging.level.cn.tuyucheng.taketoday.chatbot=DEBUG
```

如果遇到麻烦，我们还可以使用Linux和macOS的[tcpdump](https://www.baeldung.com/linux/tcpdump-command-tutorial)或Windows的windump分析Ollama和Spring Boot之间的本地网络流量。另一方面，探测Spring Boot和WhatApp Cloud之间的流量要困难得多，因为它是通过HTTPS协议进行的。

## 3. 适用于Ollama的LangChain4j

典型的Ollama安装监听端口11434，在本例中，我们将使用qwen2:1.5b模型运行它，因为它的速度足够快，可以进行聊天，但我们可以自由选择[任何其他模型](https://ollama.com/library)。

LangChain4j为我们提供了几种[ChatLanguageModel.generate(..)](https://docs.langchain4j.dev/apidocs/dev/langchain4j/model/chat/ChatLanguageModel.html)方法，它们的参数有所不同。**所有这些方法都调用Ollama的REST API [/api/chat](https://github.com/ollama/ollama/blob/main/docs/api.md#generate-a-chat-completion)**，我们可以通过检查网络流量来验证。因此，让我们使用Ollama文档中的一个[JSON示例](https://github.com/ollama/ollama/blob/main/docs/api.md#request-no-streaming)来确保它正常工作：

![](/assets/images/2025/springboot/springconversationalailangchain4jollamawa01.png)

我们的查询得到了有效的JSON响应，因此我们准备转到LangChain4j。

为避免出现问题，我们一定要注意参数的大小写。例如，“role”：“user”将产生正确的响应，而“role”：“USER”则不会。

### 3.1 配置LangChain4j

在pom.xml中，我们需要LangChain4j的两个依赖项：

```xml
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-core</artifactId>
    <version>0.33.0</version>
</dependency>
<dependency>
    <groupId>dev.langchain4j</groupId>
    <artifactId>langchain4j-ollama</artifactId>
    <version>0.33.0</version>
</dependency>
```

然后让我们将这些参数添加到application.properties中：

```properties
# Ollama API configuration
ollama.api_url=http://localhost:11434/
ollama.model=qwen2:1.5b
ollama.timeout=30
ollama.max_response_length=1000
```

参数ollama.timeout和ollama.max_response_length是可选的，我们将它们包含在内是作为一种安全措施，因为已知某些模型存在导致响应过程循环的错误。

### 3.2 实现ChatbotService

使用@Value注解，让我们在运行时从application.properties注入这些值，确保配置与应用程序逻辑分离：

```java
@Value("${ollama.api_url}")
private String apiUrl;

@Value("${ollama.model}")
private String modelName;

@Value("${ollama.timeout}")
private int timeout;

@Value("${ollama.max_response_length}")
private int maxResponseLength;
```

以下是服务Bean完全构建后需要运行的初始化逻辑，**[OllamaChatModel](https://docs.langchain4j.dev/apidocs/dev/langchain4j/model/ollama/OllamaChatModel.html)对象包含与对话式AI模型交互所需的配置**：

```java
private OllamaChatModel ollamaChatModel;

@PostConstruct
public void init() {
    this.ollamaChatModel = OllamaChatModel.builder()
        .baseUrl(apiUrl)
        .modelName(modelName)
        .timeout(Duration.ofSeconds(timeout))
        .numPredict(maxResponseLength)
        .build();
}
```

此方法获取一个问题，将其发送到聊天模型，接收响应，并处理在此过程中可能发生的任何错误：

```java
public String getResponse(String question) {
    logger.debug("Sending to Ollama: {}",  question);
    String answer = ollamaChatModel.generate(question);
    logger.debug("Receiving from Ollama: {}",  answer);
    if (answer != null && !answer.isEmpty()) {
        return answer;
    } else {
        logger.error("Invalid Ollama response for:nn" + question);
        throw new ResponseStatusException(
            HttpStatus.SC_INTERNAL_SERVER_ERROR,
            "Ollama didn't generate a valid response",
            null);
    }
}
```

我们已为控制器做好准备。

### 3.3 创建ChatbotController

此控制器在开发过程中有助于测试ChatbotService是否正常工作：

```java
@Autowired
private ChatbotService chatbotService;

@GetMapping("/api/chatbot/send")
public String getChatbotResponse(@RequestParam String question) {
    return chatbotService.getResponse(question);
}
```

让我们尝试一下：

![](/assets/images/2025/springboot/springconversationalailangchain4jollamawa02.png)

它按预期工作。

## 4. 适用于WhatsApp的Apache Camel

在继续之前，让我们在[Meta Developers](https://developers.facebook.com/)上创建一个帐户。出于测试目的，使用WhatsApp API是免费的。

### 4.1 ngrok反向代理

要将本地Spring Boot应用程序与WhatsApp Business服务集成，**我们需要一个跨平台反向代理(如[ngrok)](https://www.baeldung.com/linux/ngrok-remote-connection)连接到[免费的静态域](https://ngrok.com/blog-post/free-static-domains-ngrok-users)**。它从使用HTTPS协议的公共URL到使用HTTP协议的本地服务器创建安全隧道，允许WhatsApp与我们的应用程序通信。在此命令中，让我们将xxx.ngrok-free.app替换为ngrok分配给我们的静态域：

```bash
ngrok http --domain=xxx.ngrok-free.app 8080
```

这会将https://xxx.ngrok-free.app转发到http://localhost:8080。

### 4.2 设置 Apache Camel

第一个依赖项**camel-spring-boot-starter将Apache Camel集成到Spring Boot应用程序中**，并为[Camel路由](https://camel.apache.org/manual/routes.html)提供必要的配置。第二个依赖项camel-http-starter支持创建基于HTTP(S)的路由，使应用程序能够处理HTTP和HTTPS请求。第三个依赖项camel-jackson使用Jackson库促进JSON处理，允许Camel路由转换和[编组JSON数据](https://camel.apache.org/manual/data-format.html)：

```xml
<dependency>
    <groupId>org.apache.camel.springboot</groupId>
    <artifactId>camel-spring-boot-starter</artifactId>
    <version>4.7.0</version>
</dependency>
<dependency>
    <groupId>org.apache.camel.springboot</groupId>
    <artifactId>camel-http-starter</artifactId>
    <version>4.7.0</version>
</dependency>
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-jackson</artifactId>
    <version>4.7.0</version>
</dependency>
```

最后，我们将此配置添加到application.properties中：

```properties
# WhatsApp API configuration
whatsapp.verify_token=BaeldungDemo-Verify-Token
whatsapp.api_url=https://graph.facebook.com/v20.0/PHONE_NUMBER_ID/messages
whatsapp.access_token=ACCESS_TOKEN
```

获取PHONE_NUMBER_ID和ACCESS_TOKEN的实际值来替换属性值并非易事，我们将详细了解如何操作。

### 4.3 控制器验证Webhook Token

作为初步步骤，**我们还需要一个Spring Boot控制器来验证[WhatsApp webhook](https://developers.facebook.com/docs/whatsapp/webhooks/)令牌**，目的是在开始从WhatsApp服务接收实际数据之前验证我们的webhook端点：

```java
@Value("${whatsapp.verify_token}")
private String verifyToken;

@GetMapping("/webhook")
public String verifyWebhook(@RequestParam("hub.mode") String mode, @RequestParam("hub.verify_token") String token, @RequestParam("hub.challenge") String challenge) {
    if ("subscribe".equals(mode) && verifyToken.equals(token)) {
        return challenge;
    } else {
        return "Verification failed";
    }
}
```

那么，让我们回顾一下迄今为止所做的工作：

- ngrok使用HTTPS在公共IP上公开我们的本地Spring Boot服务器
- 添加了Apache Camel依赖项
- 我们有一个控制器来验证WhatsApp webhook令牌
- 但是，**我们还没有PHONE_NUMBER_ID和ACCESS_TOKEN的实际值**

现在是时候设置我们的WhatsApp Business帐户来获取这些值并订阅webhook服务了。

### 4.4 WhatsApp Business帐户

官方的[入门指南](https://developers.facebook.com/docs/whatsapp/cloud-api/get-started)很难理解，不符合我们的需求。这就是为什么接下来的视频将有助于我们了解Spring Boot应用程序的相关步骤。

创建名为“Baeldung Chatbot”的[业务组合](https://business.facebook.com/)后，让我们[创建我们的业务应用程序](https://developers.facebook.com/apps/creation/)：

<video class="wp-video-shortcode" id="video-185324-1_html5" width="580" height="235" preload="metadata" src="https://www.baeldung.com/wp-content/uploads/2024/08/1-Create-Whatsapp-Business-APP.mp4?_=1" style="box-sizing: border-box; font-family: Helvetica, Arial; max-width: 100%; display: inline-block; width: 580px; height: 235.021px;"></video>

然后让我们获取WhatsApp企业电话号码的ID，将其到application.properties中的whatsapp.api_url中，并向我们的个人手机发送测试消息。**让我们将此快速入门API设置页面添加到书签中，因为我们在代码开发过程中可能需要它**：

<video class="wp-video-shortcode" id="video-185324-2_html5" width="580" height="235" preload="metadata" src="https://www.baeldung.com/wp-content/uploads/2024/08/2-Get-Our-Whatsapp-Business-Phone-Number-ID-and-Send-a-Test-Message-to-Our-Personal-Mobile-Phone.mp4?_=2" style="box-sizing: border-box; font-family: Helvetica, Arial; max-width: 100%; display: inline-block; width: 580px; height: 235.021px;"></video>

此时，我们应该已经在手机上收到了这条消息：

![](/assets/images/2025/springboot/springconversationalailangchain4jollamawa03.png)

现在我们需要application.properties中的whatsapp.access_token值，让我们转到[系统用户](https://business.facebook.com/settings/system-users)，使用具有管理员完全访问权限的帐户生成一个没有过期的令牌：


<video class="wp-video-shortcode" id="video-185324-3_html5" width="580" height="315" preload="metadata" src="https://www.baeldung.com/wp-content/uploads/2024/08/3-Generate-Token-Without-Expiration.mp4?_=3" style="box-sizing: border-box; font-family: Helvetica, Arial; max-width: 100%; display: inline-block; width: 580px; height: 315.375px;"></video>

我们已准备好配置我们之前使用@GetMapping(“/webhook”)控制器创建的webhook端点。在继续之前，让我们**先启动Spring Boot应用程序**。

作为webhook的回调URL，我们需要插入以/webhook为后缀的ngrok静态域，而我们的验证令牌是BaeldungDemo-Verify-Token：

<video class="wp-video-shortcode" id="video-185324-4_html5" width="580" height="235" preload="metadata" src="https://www.baeldung.com/wp-content/uploads/2024/08/4-Configure-Webhook.mp4?_=4" style="box-sizing: border-box; font-family: Helvetica, Arial; max-width: 100%; display: inline-block; width: 580px; height: 235.266px;"></video>


按照我们展示的顺序执行这些步骤非常重要，以避免错误。

### 4.5 配置WhatsAppService发送消息 

作为参考，在进入init()和sendWhatsAppMessage(...)方法之前，让我们使用Postman向我们的手机发送一条短信。**这样我们就可以看到所需的JSON和标头，并将它们与代码进行比较**。

Authorization标头值由Bearer后跟空格和我们的whatsapp.access_token组成，而Content-Type标头由Postman自动处理：

![](/assets/images/2025/springboot/springconversationalailangchain4jollamawa04.png)

JSON结构非常简单，我们必须注意，HTTP 200响应代码并不意味着消息已实际发送。**只有当我们通过从手机向WhatsApp商业号码发送消息开始对话时，我们才会收到它**。换句话说，我们创建的聊天机器人永远无法发起对话，它只能回答用户的问题：

![](/assets/images/2025/springboot/springconversationalailangchain4jollamawa05.png)

也就是说，让我们注入whatsapp.api_url和whatsapp.access_token：

```java
@Value("${whatsapp.api_url}")
private String apiUrl;

@Value("${whatsapp.access_token}")
private String apiToken;
```

**init()方法负责设置通过WhatsApp API发送消息所需的配置**，它定义并向[CamelContext](https://camel.apache.org/manual/camelcontext.html)添加一条新路由，该路由负责处理我们的Spring Boot应用程序和WhatsApp服务之间的通信。

在此路由配置中，我们指定身份验证和内容类型所需的标头，使用Postman测试API时使用的标头：

```java
@Autowired
private CamelContext camelContext;

@PostConstruct
public void init() throws Exception {
    camelContext.addRoutes(new RouteBuilder() {
        @Override
        public void configure() {
            JacksonDataFormat jacksonDataFormat = new JacksonDataFormat();
            jacksonDataFormat.setPrettyPrint(true);

            from("direct:sendWhatsAppMessage")
                .setHeader("Authorization", constant("Bearer " + apiToken))
                .setHeader("Content-Type", constant("application/json"))
                .marshal(jacksonDataFormat)
                .process(exchange -> {
                    logger.debug("Sending JSON: {}", exchange.getIn().getBody(String.class));
                }).to(apiUrl).process(exchange -> {
                    logger.debug("Response: {}", exchange.getIn().getBody(String.class));
                });
        }
    });
}
```

这样，direct：sendWhatsAppMessage[端点](https://camel.apache.org/components/4.4.x/eips/message-endpoint.html)允许在应用程序内以编程方式触发路由，确保消息由Jackson正确编组并使用必要的标头发送。

sendWhatsAppMessage(...)使用Camel [ProducerTemplate](https://camel.apache.org/manual/producertemplate.html)将JSON负载发送到direct:sendWhatsAppMessage路由，HashMap的结构遵循我们之前在Postman中使用的JSON结构。**此方法确保与WhatsApp API无缝集成，提供一种从Spring Boot应用程序发送消息的结构化方式**：

```java
@Autowired
private ProducerTemplate producerTemplate;

public void sendWhatsAppMessage(String toNumber, String message) {
    Map<String, Object> body = new HashMap<>();
    body.put("messaging_product", "whatsapp");
    body.put("to", toNumber);
    body.put("type", "text");

    Map<String, String> text = new HashMap<>();
    text.put("body", message);
    body.put("text", text);

    producerTemplate.sendBody("direct:sendWhatsAppMessage", body);
}
```

发送消息的代码已经准备好了。

### 4.6 配置WhatsAppService接收消息

为了处理来自WhatsApp用户的传入消息，**processIncomingMessage(...)方法处理从我们的webhook端点收到的有效负载**，提取相关信息(例如发件人的电话号码和消息内容)，然后使用我们的聊天机器人服务生成适当的响应。最后，它使用sendWhatsAppMessage(...)方法将Ollama的响应发送回用户：

```java
@Autowired
private ObjectMapper objectMapper;

@Autowired
private ChatbotService chatbotService;

public void processIncomingMessage(String payload) {
    try {
        JsonNode jsonNode = objectMapper.readTree(payload);
        JsonNode messages = jsonNode.at("/entry/0/changes/0/value/messages");
        if (messages.isArray() && messages.size() > 0) {
            String receivedText = messages.get(0).at("/text/body").asText();
            String fromNumber = messages.get(0).at("/from").asText();
            logger.debug(fromNumber + " sent the message: " + receivedText);
            this.sendWhatsAppMessage(fromNumber, chatbotService.getResponse(receivedText));
        }
    } catch (Exception e) {
        logger.error("Error processing incoming payload: {} ", payload, e);
    }
}
```

下一步是编写控制器来测试我们的WhatsAppService方法。

### 4.7 创建WhatsAppController

sendWhatsAppMessage(...)控制器在开发过程中很有用，可以测试发送消息的过程：

```java
@Autowired
private WhatsAppService whatsAppService;    

@PostMapping("/api/whatsapp/send")
public String sendWhatsAppMessage(@RequestParam String to, @RequestParam String message) {
    whatsAppService.sendWhatsAppMessage(to, message);
    return "Message sent";
}
```

让我们尝试一下：

![](/assets/images/2025/springboot/springconversationalailangchain4jollamawa06.png)

它按预期工作。一切准备就绪，可以编写接收用户发送的消息的receiveMessage(...)控制器：

```java
@PostMapping("/webhook")
public void receiveMessage(@RequestBody String payload) {
    whatsAppService.processIncomingMessage(payload);
}
```

这是最终测试：

![](/assets/images/2025/springboot/springconversationalailangchain4jollamawa07.png)

Ollama使用LaTeX语法回答了我们的数学问题，我们使用的qwen2:1.5b LLM支持29种语言，以下是[完整列表](https://ollama.com/library/qwen2:1.5b)。

## 5. 总结

在本文中，**我们演示了如何将Apache Camel和LangChain4j集成到Spring Boot应用程序中，以管理WhatsApp上的AI驱动对话**，并使用本地安装的Ollama进行AI处理。我们首先设置Ollama并配置我们的Spring Boot应用程序来处理请求参数。

然后，我们集成了LangChain4j与Ollama模型进行交互，使用ChatbotService处理AI响应并确保无缝通信。

为了与WhatsApp集成，我们设置了一个WhatsApp Business帐户并使用ngrok作为反向代理，以促进本地服务器与WhatsApp之间的通信。我们配置了Apache Camel并创建了WhatsAppService来处理传入消息，使用我们的ChatbotService生成响应并做出适当的响应。

我们使用专用控制器测试了ChatbotService和WhatsAppService，以确保完整的功能。