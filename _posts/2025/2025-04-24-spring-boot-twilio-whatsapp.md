---
layout: post
title:  使用Twilio在Spring Boot中发送WhatsApp消息
category: saas
copyright: saas
excerpt: Twilio
---

## 1. 概述

WhatsApp Messenger是全球[领先的消息平台](https://explodingtopics.com/blog/messaging-apps-stats)，是企业与用户联系的重要工具。

通过WhatsApp进行交流，我们可以增强客户参与度、提供高效支持并与用户建立更牢固的关系。

**在本教程中，我们将探讨如何在[Spring Boot](https://www.baeldung.com/spring-boot)应用程序中使用[Twilio](https://www.twilio.com/en-us)发送WhatsApp消息**，我们将介绍必要的配置，并实现发送消息和处理用户回复的功能。

## 2. 设置Twilio

**要遵循本教程，我们首先需要一个Twilio帐户和一个[WhatsApp商业帐户(WABA)](https://en-gb.facebook.com/business/help/2087193751603668?id=2129163877102343)**。

我们需要创建一个WhatsApp发件人来连接这两个账户，Twilio提供了[详细的设置教程](https://www.twilio.com/docs/whatsapp/self-sign-up#1-create-a-whatsapp-sender)，可以参考它来指导我们完成整个过程。

一旦我们成功设置了WhatsApp发送器，我们就可以继续向用户发送消息和接收来自用户的消息。

## 3. 设置项目

在我们可以使用Twilio发送WhatsApp消息之前，我们需要包含SDK依赖并正确配置我们的应用程序。

### 3.1 依赖

让我们首先将[Twilio SDK依赖](https://mvnrepository.com/artifact/com.twilio.sdk/twilio/latest)添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>com.twilio.sdk</groupId>
    <artifactId>twilio</artifactId>
    <version>10.4.1</version>
</dependency>
```

### 3.2 定义Twilio配置属性

现在，为了与Twilio服务交互并向用户发送WhatsApp消息，我们需要配置[帐户SID和授权令牌](https://www.twilio.com/docs/iam/api#authentication:~:text=you%20will%20use%20your%20Twilio%20Account%20SID%20as%20the%20username%20and%20your%20Auth%20Token%20as%20the%20password%20for%20HTTP%20Basic%20authentication)来验证API请求，我们还需要消息服务SID来指定要使用哪个[消息服务](https://www.twilio.com/docs/messaging/services)(使用我们已启用WhatsApp的Twilio电话号码)发送消息。

我们将这些属性存储在项目的application.yaml文件中，并使用[@ConfigurationProperties](https://www.baeldung.com/configuration-properties-in-spring-boot)将值映射到POJO，我们的服务层在与Twilio交互时引用该POJO：

```java
@Validated
@ConfigurationProperties(prefix = "cn.tuyucheng.taketoday.twilio")
class TwilioConfigurationProperties {

    @NotBlank
    @Pattern(regexp = "^AC[0-9a-fA-F]{32}$")
    private String accountSid;

    @NotBlank
    private String authToken;

    @NotBlank
    @Pattern(regexp = "^MG[0-9a-fA-F]{32}$")
    private String messagingSid;

    // standard setters and getters
}
```

我们还添加了[验证注解](https://www.baeldung.com/java-validation)，以确保所有必需的属性都已正确配置。如果任何定义的验证失败，都会导致Spring ApplicationContext启动失败。**这允许我们符合快速失败原则**。

下面是我们的application.yaml文件的片段，它定义了将自动映射到我们的TwilioConfigurationProperties类的必需属性：

```yaml
cn:
    tuyucheng:
        twilio:
            account-sid: ${TWILIO_ACCOUNT_SID}
            auth-token: ${TWILIO_AUTH_TOKEN}
            messaging-sid: ${TWILIO_MESSAGING_SID}
```

**因此，此设置允许我们将Twilio属性外部化并在我们的应用程序中轻松访问它们**。

### 3.3 启动时初始化Twilio

**为了成功调用SDK公开的方法，我们需要在启动时对其进行一次初始化**。为此，我们将创建一个实现[ApplicationRunner](https://www.baeldung.com/running-setup-logic-on-startup-in-spring#7-spring-boot-applicationrunner)接口的TwilioInitializer类：

```java
@Component
@EnableConfigurationProperties(TwilioConfigurationProperties.class)
class TwilioInitializer implements ApplicationRunner {

    private final TwilioConfigurationProperties twilioConfigurationProperties;

    // standard constructor

    @Override
    public void run(ApplicationArguments args) {
        String accountSid = twilioConfigurationProperties.getAccountSid();
        String authToken = twilioConfigurationProperties.getAuthToken();
        Twilio.init(accountSid, authToken);
    }
}
```

使用[构造函数注入](https://www.baeldung.com/constructor-injection-in-spring)，我们注入了之前创建的TwilioConfigurationProperties类的实例。然后，我们使用配置的帐户SID和身份验证令牌在run()方法中初始化Twilio SDK。

这确保了Twilio在应用程序启动时即可使用，**这种方法比每次需要发送消息时在服务层初始化Twilio客户端要好得多**。

## 4. 发送WhatsApp消息

现在我们已经定义了属性，让我们创建一个WhatsAppMessageDispatcher类并引用它们与Twilio进行交互。

为了演示，**我们将举一个例子，每当我们在网站上发布新文章时，我们都希望通知用户**，我们将向他们发送一条带有文章链接的WhatsApp消息，以便他们轻松查看。

### 4.1 配置内容SID

为了限制企业发送未经请求或垃圾信息，**WhatsApp要求所有企业发起的通知都必须模板化并预先注册，这些[模板](https://www.twilio.com/docs/content/overview)由唯一的内容SID标识，该SID必须经过WhatsApp批准才能在我们的应用程序中使用**。

对于我们的示例，我们将配置以下消息模板：

```text
New Article Published. Check it out : {{ArticleURL}}
```

这里，{{ArticleURL}}是一个占位符，当我们发出通知时，它将被替换为新发布文章的实际URL。

现在，让我们在TwilioConfigurationProperties类中定义一个新的嵌套类来保存我们的内容SID：

```java
@Valid
private NewArticleNotification newArticleNotification = new NewArticleNotification();

class NewArticleNotification {

    @NotBlank
    @Pattern(regexp = "^HX[0-9a-fA-F]{32}$")
    private String contentSid;

    // standard setter and getter
}
```

我们再次添加验证注解，以确保正确配置内容SID并且它符合预期的格式。

类似地，让我们将相应的内容SID属性添加到我们的application.yaml文件中：

```yaml
cn:
    tuyucheng:
        twilio:
            new-article-notification:
                content-sid: ${NEW_ARTICLE_NOTIFICATION_CONTENT_SID}
```

### 4.2 实现消息调度器

现在我们已经配置了内容SID，让我们实现服务方法向我们的用户发送通知：

```java
public void dispatchNewArticleNotification(String phoneNumber, String articleUrl) {
    String messagingSid = twilioConfigurationProperties.getMessagingSid();
    String contentSid = twilioConfigurationProperties.getNewArticleNotification().getContentSid();
    PhoneNumber toPhoneNumber = new PhoneNumber(String.format("whatsapp:%s", phoneNumber));

    JSONObject contentVariables = new JSONObject();
    contentVariables.put("ArticleURL", articleUrl);

    Message.creator(toPhoneNumber, messagingSid)
            .setContentSid(contentSid)
            .setContentVariables(contentVariables.toString())
            .create();
}
```

在dispatchNewArticleNotification()方法中，我们使用已配置的消息SID和内容SID向指定的电话号码发送通知，我们还将文章URL作为内容变量传递，该变量将用于替换消息模板中的占位符。

**值得注意的是，我们也可以配置一个不带任何占位符的静态消息模板，在这种情况下，我们可以简单地省略对setContentVariables()方法的调用**。

## 5. 处理WhatsApp回复

我们发出通知后，用户可能会回复他们的想法或疑问。**当用户回复我们的WhatsApp企业账号时，会启动一个24小时的会话窗口，在此期间，我们可以使用自由格式的消息与用户沟通，无需预先批准的模板**。

**为了自动处理来自应用程序的用户回复，我们需要在Twilio消息服务中配置一个[Webhook](https://www.twilio.com/docs/usage/webhooks/messaging-webhooks)端点**，每当用户发送消息时，Twilio服务都会调用此端点。我们在已配置的API端点中接收[多个参数](https://www.twilio.com/docs/messaging/guides/webhook-request#parameters-in-twilios-request-to-your-application)，可用于自定义响应。

让我们看看如何在Spring Boot应用程序中创建这样的API端点。

### 5.1 实现回复消息调度器

首先，我们将在WhatsAppMessageDispatcher类中创建一个新的服务方法来发送自由格式的回复消息：

```java
public void dispatchReplyMessage(String phoneNumber, String username) {
    String messagingSid = twilioConfigurationProperties.getMessagingSid();
    PhoneNumber toPhoneNumber = new PhoneNumber(String.format("whatsapp:%s", phoneNumber));

    String message = String.format("Hey %s, our team will get back to you shortly.", username);
    Message.creator(toPhoneNumber, messagingSid, message).create();
}
```

在我们的dispatchReplyMessage()方法中，我们向用户发送个性化消息，通过他们的用户名来称呼他们，并让他们知道我们的团队将很快回复他们。

值得注意的是，我们甚至可以在24小时内向用户发送[多媒体消息](https://www.baeldung.com/java-sms-twilio#3-sending-an-mms)。

### 5.2 公开Webhook端点

接下来，我们将在应用程序中公开一个POST API端点，**此端点的路径应与我们在Twilio消息服务中配置的webhook URL匹配**：

```java
@PostMapping(value = "/api/v1/whatsapp-message-reply")
public ResponseEntity<Void> reply(@RequestParam("ProfileName") String username, @RequestParam("WaId") String phoneNumber) {
    whatsappMessageDispatcher.dispatchReplyMessage(phoneNumber, username);
    return ResponseEntity.ok().build();
}
```

在我们的控制器方法中，我们接收来自Twilio的ProfileName和WaId参数，**这些参数分别包含发送消息的用户的用户名和电话号码**，然后，我们将这些值传递给dispatchReplyMessage()方法，以将响应发送回用户。

我们在示例中使用了ProfileName和WaId参数，但如前所述，Twilio会向我们配置的API端点发送多个参数。例如，我们可以访问Body参数来检索用户消息的文本内容，我们可以将此消息存储在队列中，并将其路由到相应的支持团队进行进一步处理。

## 6. 测试Twilio集成

现在我们已经实现了使用Twilio发送WhatsApp消息的功能，让我们看看如何测试这种集成。

测试外部服务可能颇具挑战性，因为我们不想在测试期间对Twilio进行实际的API调用，**这时我们将使用[MockServer](https://www.baeldung.com/mockserver)，它允许我们模拟Twilio的传出调用**。

### 6.1 配置Twilio REST客户端

**为了将我们的Twilio API请求路由到MockServer，我们需要为Twilio SDK配置一个自定义HTTP客户端**。

我们将在测试套件中创建一个类，该类使用自定义HttpClient创建TwilioRestClient的实例：

```java
class TwilioProxyClient {

    private final String accountSid;
    private final String authToken;
    private final String host;
    private final int port;

    // standard constructor

    public TwilioRestClient createHttpClient() {
        SSLContext sslContext = SSLContextBuilder.create()
                .loadTrustMaterial((chain, authType) -> true)
                .build();

        HttpClientBuilder clientBuilder = HttpClientBuilder.create()
                .setSSLContext(sslContext)
                .setProxy(new HttpHost(host, port));

        HttpClient httpClient = new NetworkHttpClient(clientBuilder);
        return new Builder(accountSid, authToken)
                .httpClient(httpClient)
                .build();
    }
}
```

在TwilioProxyClient类中，我们创建了一个自定义的HttpClient，它将所有请求路由到由host和port参数指定的代理服务器。**我们还配置了SSL上下文以信任所有证书，因为MockServer默认使用自签名证书**。

### 6.2 配置测试环境

在编写测试之前，我们将在src/test/resources目录中创建一个application-integration-test.yaml文件，其中包含以下内容：

```yaml
cn:
    tuyucheng:
        twilio:
            account-sid: AC123abc123abc123abc123abc123abc12
            auth-token: test-auth-token
            messaging-sid: MG123abc123abc123abc123abc123abc12
            new-article-notification:
                content-sid: HX123abc123abc123abc123abc123abc12
```

**这些虚拟值绕过了我们之前在TwilioConfigurationProperties类中配置的验证**。

现在，让我们使用[@BeforeEach](https://www.baeldung.com/junit-before-beforeclass-beforeeach-beforeall#beforeeach-and-beforeall)注解设置我们的测试环境

```java
@Autowired
private TwilioConfigurationProperties twilioConfigurationProperties;

private MockServerClient mockServerClient;

private String twilioApiPath;

@BeforeEach
void setUp() {
    String accountSid = twilioConfigurationProperties.getAccountSid();
    String authToken = twilioConfigurationProperties.getAuthToken();

    InetSocketAddress remoteAddress = mockServerClient.remoteAddress();
    String host = remoteAddress.getHostName();
    int port = remoteAddress.getPort();

    TwilioProxyClient twilioProxyClient = new TwilioProxyClient(accountSid, authToken, host, port);
    Twilio.setRestClient(twilioProxyClient.createHttpClient());

    twilioApiPath = String.format("/2010-04-01/Accounts/%s/Messages.json", accountSid);
}
```

**在setUp()方法中，我们创建TwilioProxyClient类的一个实例，并传入正在运行的MockServer实例的主机和端口**。然后，此客户端用于为Twilio SDK设置自定义RestClient，我们还将发送消息的API路径存储在twilioApiPath变量中。

### 6.3 验证Twilio请求

最后，让我们编写一个测试用例来验证我们的dispatchNewArticleNotification()方法是否向Twilio发送了预期的请求：

```java
// Set up test data
String contentSid = twilioConfigurationProperties.getNewArticleNotification().getContentSid();
String messagingSid = twilioConfigurationProperties.getMessagingSid();
String contactNumber = "+911001001000";
String articleUrl = RandomString.make();

// Configure mock server expectations
mockServerClient
    .when(request()
        .withMethod("POST")
        .withPath(twilioApiPath)
        .withBody(new ParameterBody(
            param("To", String.format("whatsapp:%s", contactNumber)),
            param("ContentSid", contentSid),
            param("ContentVariables", String.format("{\"ArticleURL\":\"%s\"}", articleUrl)),
            param("MessagingServiceSid", messagingSid)
        ))
    )
        .respond(response()
            .withStatusCode(200)
            .withBody(EMPTY_JSON));

// Invoke method under test
whatsAppMessageDispatcher.dispatchNewArticleNotification(contactNumber, articleUrl);

// Verify the expected request was made
mockServerClient.verify(request()
    .withMethod("POST")
    .withPath(twilioApiPath)
    .withBody(new ParameterBody(
        param("To", String.format("whatsapp:%s", contactNumber)),
        param("ContentSid", contentSid),
        param("ContentVariables", String.format("{\"ArticleURL\":\"%s\"}", articleUrl)),
        param("MessagingServiceSid", messagingSid)
    )), VerificationTimes.once()
);
```

在我们的测试方法中，我们首先设置测试数据，并配置MockServer以接收对Twilio API路径的POST请求，并在请求正文中携带特定参数，我们还指示MockServer在发出此请求时以200状态码和空的JSON正文进行响应。

接下来，我们使用测试数据调用dispatchNewArticleNotification()方法，并验证是否向MockServer发出了预期的请求一次。

**通过使用MockServer模拟Twilio API，我们确保我们的集成按预期工作，而无需实际发送任何消息或产生任何成本**。

## 7. 总结

在本文中，我们探讨了如何使用Twilio从 Spring Boot应用程序发送WhatsApp消息。

我们完成了必要的配置并实现了使用动态占位符向用户发送模板通知的功能。

最后，我们通过公开一个webhook端点来接收来自Twilio的回复数据，从而处理用户对我们通知的回复，并创建了一个服务方法来分派通用的非模板化回复消息。