---
layout: post
title:  在Spring Boot中使用SendGrid发送电子邮件
category: saas
copyright: saas
excerpt: SendGrid
---

## 1. 概述

发送电子邮件是现代Web应用程序的一项重要功能，无论是用于用户注册、密码重置还是促销活动。

**在本教程中，我们将探讨如何在[Spring Boot](https://www.baeldung.com/spring-boot)应用程序中使用[SendGrid](https://sendgrid.com/en-us)发送电子邮件**，我们将介绍必要的配置，并针对不同的用例实现电子邮件发送功能。

## 2. 设置SendGrid

**要学习本教程，我们首先需要一个SendGrid帐户**。SendGrid提供免费套餐，允许我们每天最多发送100封电子邮件，这对于我们的演示来说已经足够了。

注册后，我们就需要创建一个[API密钥](https://www.twilio.com/docs/sendgrid/ui/account-and-settings/api-keys)来验证我们对SendGrid服务的请求。

最后，我们需要验证[发件人身份](https://www.twilio.com/docs/sendgrid/for-developers/sending-email/sender-identity)才能成功发送电子邮件。

## 3. 设置项目

在我们开始使用SendGrid发送电子邮件之前，我们需要包含SDK依赖并正确配置我们的应用程序。

### 3.1 依赖

让我们首先将[SendGrid SDK依赖](https://mvnrepository.com/artifact/com.sendgrid/sendgrid-java)添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>com.sendgrid</groupId>
    <artifactId>sendgrid-java</artifactId>
    <version>4.10.2</version>
</dependency>
```

该依赖为我们提供了与SendGrid服务交互并从我们的应用程序发送电子邮件所需的类。

### 3.2 定义SendGrid配置属性

现在，为了与SendGrid服务交互并向用户发送电子邮件，我们需要配置API密钥来验证API请求。我们还需要配置发件人的姓名和电子邮件地址，这些应该与我们在SendGrid帐户中设置的发件人身份相匹配。

我们将这些属性存储在项目的application.yaml文件中，并使用[@ConfigurationProperties](https://www.baeldung.com/configuration-properties-in-spring-boot)将值映射到POJO，我们的服务层在与SendGrid交互时引用该POJO：

```java
@Validated
@ConfigurationProperties(prefix = "cn.tuyucheng.taketoday.sendgrid")
class SendGridConfigurationProperties {
    @NotBlank
    @Pattern(regexp = "^SG[0-9a-zA-Z._]{67}$")
    private String apiKey;

    @Email
    @NotBlank
    private String fromEmail;

    @NotBlank
    private String fromName;

    // standard setters and getters
}
```

我们还添加了[校验注解](https://www.baeldung.com/java-validation)，以确保所有必需的属性都已正确配置。如果任何定义的验证失败，Spring [ApplicationContext](https://www.baeldung.com/spring-application-context)将无法启动，**这使我们能够遵循快速失败原则**。

下面是我们的application.yaml文件的片段，它定义了将自动映射到我们的SendGridConfigurationProperties类的必需属性：

```yaml
cn:
    tuyucheng:
        sendgrid:
            api-key: ${SENDGRID_API_KEY}
            from-email: ${SENDGRID_FROM_EMAIL}
            from-name: ${SENDGRID_FROM_NAME}
```

**我们使用${}属性占位符从[环境变量](https://www.baeldung.com/spring-boot-properties-env-variables#use-environment-variable-in-applicationyml-file)中加载属性值，因此，此设置允许我们将SendGrid属性外部化，并在我们的应用程序中轻松访问它们**。

### 3.3 配置SendGrid Bean

现在我们已经配置了属性，让我们引用它们来定义必要的Bean：

```java
@Configuration
@EnableConfigurationProperties(SendGridConfigurationProperties.class)
class SendGridConfiguration {
    private final SendGridConfigurationProperties sendGridConfigurationProperties;

    // standard constructor

    @Bean
    public SendGrid sendGrid() {
        String apiKey = sendGridConfigurationProperties.getApiKey();
        return new SendGrid(apiKey);
    }
}
```

使用[构造函数注入](https://www.baeldung.com/constructor-injection-in-spring)，我们注入了之前创建的SendGridConfigurationProperties类的实例。然后，我们使用配置的API密钥创建一个SendGrid [Bean](https://www.baeldung.com/spring-Bean)。

接下来，我们将创建一个Bean来代表所有外发电子邮件的发件人：

```java
@Bean
public Email fromEmail() {
    String fromEmail = sendGridConfigurationProperties.getFromEmail();
    String fromName = sendGridConfigurationProperties.getFromName();
    return new Email(fromEmail, fromName);
}
```

有了这些Bean，我们可以在服务层中自动装配它们以与SendGrid服务进行交互。

## 4. 发送简单的电子邮件

现在我们已经定义了Bean，让我们创建一个EmailDispatcher类并引用它们来发送一封简单的电子邮件：

```java
private static final String EMAIL_ENDPOINT = "mail/send";

public void dispatchEmail(String emailId, String subject, String body) {
    Email toEmail = new Email(emailId);
    Content content = new Content("text/plain", body);
    Mail mail = new Mail(fromEmail, subject, toEmail, content);

    Request request = new Request();
    request.setMethod(Method.POST);
    request.setEndpoint(EMAIL_ENDPOINT);
    request.setBody(mail.build());

    sendGrid.api(request);
}
```

在我们的dispatchEmail()方法中，我们创建一个新的Mail对象来代表我们要发送的电子邮件，然后将其设置为我们的Request对象的请求正文。

**最后，我们使用SendGrid Bean将请求发送到SendGrid服务**。

## 5. 发送带附件的电子邮件

除了发送简单的纯文本电子邮件之外，SendGrid还允许我们发送带有附件的电子邮件。

首先，我们将创建一个辅助方法，将MultipartFile转换为来自SendGrid SDK的Attachments对象：

```java
private Attachments createAttachment(MultipartFile file) {
    byte[] encodedFileContent = Base64.getEncoder().encode(file.getBytes());
    Attachments attachment = new Attachments();
    attachment.setDisposition("attachment");
    attachment.setType(file.getContentType());
    attachment.setFilename(file.getOriginalFilename());
    attachment.setContent(new String(encodedFileContent, StandardCharsets.UTF_8));
    return attachment;
}
```

在我们的createAttachment()方法中，我们创建一个新的Attachments对象并根据MultipartFile参数设置其属性。

**值得注意的是，在将文件内容设置到Attachments对象之前，我们会对其进行[Base64编码](https://www.baeldung.com/java-base64-encode-and-decode)**。

接下来，让我们更新dispatchEmail()方法以接收MultipartFile对象的可选列表：

```java
public void dispatchEmail(String emailId, String subject, String body, List<MultipartFile> files) {
    // ... same as above

    if (files != null && !files.isEmpty()) {
        for (MultipartFile file : files) {
            Attachments attachment = createAttachment(file);
            mail.addAttachments(attachment);
        }
    }

    // ... same as above
}
```

我们遍历files参数中的每个文件，使用createAttachment()方法创建其对应的Attachments对象，并将其添加到Mail对象中，该方法的其余部分保持不变。

## 6. 使用动态模板发送电子邮件

SendGrid还允许我们使用HTML和[Handlebars语法](https://www.twilio.com/docs/sendgrid/for-developers/sending-email/using-handlebars)创建动态电子邮件模板。

为了进行此演示，我们将举一个例子，**假设想向我们的用户发送个性化的水合警报电子邮件**。

### 6.1 创建HTML模板

首先，我们将为水合警报电子邮件创建一个HTML模板：

```html
<html>
<head>
    <style>
        body { font-family: Arial; line-height: 2; text-align: Center; }
        h2 { color: DeepSkyBlue; }
        .alert { background: Red; color: White; padding: 1rem; font-size: 1.5rem; font-weight: bold; }
        .message { border: .3rem solid DeepSkyBlue; padding: 1rem; margin-top: 1rem; }
        .status { background: LightCyan; padding: 1rem; margin-top: 1rem; }
    </style>
</head>
<body>
<div class="alert">⚠️ URGENT HYDRATION ALERT ⚠️</div>
<div class="message">
    <h2>It's time to drink water!</h2>
    <p>Hey {{name}}, this is your friendly reminder to stay hydrated. Your body will thank you!</p>
    <div class="status">
        <p><strong>Last drink:</strong> {{lastDrinkTime}}</p>
        <p><strong>Hydration status:</strong> {{hydrationStatus}}</p>
    </div>
</div>
</body>
</html>
```

**在我们的模板中，我们使用Handlebars语法来定义{{name}}、{{lastDrinkTime}}和{{hydrationStatus}}的占位符**。发送电子邮件时，我们会将这些占位符替换为实际值。

我们还使用内部CSS来美化我们的电子邮件模板。

### 6.2 配置模板ID

一旦我们在SendGrid中创建模板，它就会被分配一个唯一的模板ID。

**为了保存此模板ID，我们将在SendGridConfigurationProperties类中定义一个[嵌套类](https://www.baeldung.com/java-nested-classes)**：

```java
@Valid
private HydrationAlertNotification hydrationAlertNotification = new HydrationAlertNotification();

class HydrationAlertNotification {
    @NotBlank
    @Pattern(regexp = "^d-[a-f0-9]{32}$")
    private String templateId;

    // standard setter and getter
}
```

我们再次添加校验注解，以确保正确配置模板ID并且它符合预期的格式。

类似地，让我们将相应的模板ID属性添加到application.yaml文件中：

```yaml
cn:
    tuyucheng:
        sendgrid:
            hydration-alert-notification:
                template-id: ${HYDRATION_ALERT_TEMPLATE_ID}
```

发送水合警报电子邮件时，我们将在EmailDispatcher类中使用此配置的模板ID。

### 6.3 发送模板电子邮件

现在我们已经配置了模板ID，让我们创建一个自定义个性化类来保存占位符键名称及其对应的值：

```java
class DynamicTemplatePersonalization extends Personalization {
    private final Map<String, Object> dynamicTemplateData = new HashMap<>();

    public void add(String key, String value) {
        dynamicTemplateData.put(key, value);
    }

    @Override
    public Map<String, Object> getDynamicTemplateData() {
        return dynamicTemplateData;
    }
}
```

**我们覆盖getDynamicTemplateData()方法来返回我们的dynamicTemplateData Map，我们使用add()方法填充该Map**。

现在，让我们创建一个新的服务方法来发送水合警报：

```java
public void dispatchHydrationAlert(String emailId, String username) {
    Email toEmail = new Email(emailId);
    String templateId = sendGridConfigurationProperties.getHydrationAlertNotification().getTemplateId();

    DynamicTemplatePersonalization personalization = new DynamicTemplatePersonalization();
    personalization.add("name", username);
    personalization.add("lastDrinkTime", "Way too long ago");
    personalization.add("hydrationStatus", "Thirsty as a camel");
    personalization.addTo(toEmail);

    Mail mail = new Mail();
    mail.setFrom(fromEmail);
    mail.setTemplateId(templateId);
    mail.addPersonalization(personalization);

    // ... sending request process same as previous   
}
```

在我们的dispatchHydrationAlert()方法中，我们创建DynamicTemplatePersonalization类的实例，并为我们在HTML模板中定义的占位符添加自定义值。

然后，在将请求发送到SendGrid之前，我们将此personalization对象与Mail对象上的templateId一起设置。

SendGrid会用提供的动态数据替换HTML模板中的占位符，**这有助于我们向用户发送个性化电子邮件，同时保持一致的设计和布局**。

## 7. 测试SendGrid集成

现在我们已经实现了使用SendGrid发送电子邮件的功能，让我们看看如何测试这种集成。

测试外部服务可能颇具挑战性，因为我们不想在测试期间对SendGrid进行实际的API调用。这时，**我们将使用[MockServer](https://www.baeldung.com/mockserver)来模拟SendGrid的传出调用**。

### 7.1 配置测试环境

在编写测试之前，我们将在src/test/resources目录中创建一个application-integration-test.yaml文件，其中包含以下内容：

```yaml
cn:
    tuyucheng:
        sendgrid:
            api-key: SG0101010101010101010101010101010101010101010101010101010101010101010
            from-email: no-reply@tuyucheng.com
            from-name: Tuyucheng
            hydration-alert-notification:
                template-id: d-01010101010101010101010101010101
```

**这些虚拟值绕过了我们之前在SendGridConfigurationProperties类中配置的校验**。

现在，让我们设置测试类：

```java
@SpringBootTest
@ActiveProfiles("integration-test")
@MockServerTest("server.url=http://localhost:${mockServerPort}")
@EnableConfigurationProperties(SendGridConfigurationProperties.class)
class EmailDispatcherIntegrationTest {
    private MockServerClient mockServerClient;

    @Autowired
    private EmailDispatcher emailDispatcher;
    
    @Autowired
    private SendGridConfigurationProperties sendGridConfigurationProperties;
    
    private static final String SENDGRID_EMAIL_API_PATH = "/v3/mail/send";
}
```

我们使用[@ActiveProfiles](https://www.baeldung.com/spring-boot-junit-5-testing-active-profile)注解来加载我们的集成测试特定的属性。

**我们还使用@MockServerTest注解来启动MockServer实例，并使用${mockServerPort}占位符创建一个server.url测试属性，该占位符将被MockServer选择的空闲端口替换**，我们将在下一节配置自定义SendGrid REST客户端时引用该端口。

### 7.2 配置自定义SendGrid REST客户端

为了将我们的SendGrid API请求路由到MockServer，我们需要为SendGrid SDK配置一个自定义REST客户端。

我们将创建一个[@TestConfiguration](https://www.baeldung.com/spring-boot-testing#test-configuration-withtestconfiguration)类，该类使用自定义HttpClient定义一个新SendGrid Bean：

```java
@TestConfiguration
@EnableConfigurationProperties(SendGridConfigurationProperties.class)
class TestSendGridConfiguration {
    @Value("${server.url}")
    private URI serverUrl;

    @Autowired
    private SendGridConfigurationProperties sendGridConfigurationProperties;

    @Bean
    @Primary
    public SendGrid testSendGrid() {
        SSLContext sslContext = SSLContextBuilder.create()
                .loadTrustMaterial((chain, authType) -> true)
                .build();

        HttpClientBuilder clientBuilder = HttpClientBuilder.create()
                .setSSLContext(sslContext)
                .setProxy(new HttpHost(serverUrl.getHost(), serverUrl.getPort()));

        Client client = new Client(clientBuilder.build(), true);
        client.buildUri(serverUrl.toString(), null, null);

        String apiKey = sendGridConfigurationProperties.getApiKey();
        return new SendGrid(apiKey, client);
    }
}
```

在TestSendGridConfiguration类中，我们创建了一个自定义客户端，它将所有请求路由到由server.url属性指定的代理服务器。**我们还配置了SSL上下文以信任所有证书，因为MockServer默认使用自签名证书**。

为了在我们的集成测试中使用此测试配置，我们需要在测试类中添加@ContextConfiguration注解：

```java
@ContextConfiguration(classes = TestSendGridConfiguration.class)
```

**这确保我们的应用程序在运行集成测试时使用我们在TestSendGridConfiguration类中定义的Bean，而不是我们在SendGridConfiguration类中定义的Bean**。

### 7.3 验证SendGrid请求

最后，让我们编写一个测试用例来验证我们的dispatchEmail()方法是否将预期的请求发送到SendGrid：

```java
// Set up test data
String toEmail = RandomString.make() + "@tuyucheng.it";
String emailSubject = RandomString.make();
String emailBody = RandomString.make();
String fromName = sendGridConfigurationProperties.getFromName();
String fromEmail = sendGridConfigurationProperties.getFromEmail();
String apiKey = sendGridConfigurationProperties.getApiKey();

// Create JSON body
String jsonBody = String.format("""
    {
        "from": {
            "name": "%s",
            "email": "%s"
        },
        "subject": "%s",
        "personalizations": [{
            "to": [{
                "email": "%s"
            }]
        }],
        "content": [{
            "value": "%s"
        }]
    }
    """, fromName, fromEmail, emailSubject, toEmail, emailBody);

// Configure mock server expectations
mockServerClient
    .when(request()
        .withMethod("POST")
        .withPath(SENDGRID_EMAIL_API_PATH)
        .withHeader("Authorization", "Bearer " + apiKey)
        .withBody(new JsonBody(jsonBody, MatchType.ONLY_MATCHING_FIELDS)
    ))
  .respond(response().withStatusCode(202));

// Invoke method under test
emailDispatcher.dispatchEmail(toEmail, emailSubject, emailBody);

// Verify the expected request was made
mockServerClient
    .verify(request()
        .withMethod("POST")
        .withPath(SENDGRID_EMAIL_API_PATH)
        .withHeader("Authorization", "Bearer " + apiKey)
        .withBody(new JsonBody(jsonBody, MatchType.ONLY_MATCHING_FIELDS)
    ), VerificationTimes.once());
```

在我们的测试方法中，我们首先设置测试数据，并为SendGrid请求创建预期的JSON主体。然后，我们将MockServer配置为接收一个包含Authorization标头和JSON主体的POST请求，该请求指向SendGrid API路径，我们还指示MockServer在发出此请求时以202状态码进行响应。

接下来，我们使用测试数据调用dispatchEmail()方法并验证是否向MockServer发出了预期的请求一次。

**通过使用MockServer模拟SendGrid API，我们确保我们的集成按预期工作，而无需实际发送任何电子邮件或产生任何成本**。

## 8. 总结

在本文中，我们探讨了如何使用SendGrid从Spring Boot应用程序发送电子邮件。

我们完成了必要的配置并实现了发送简单电子邮件、带附件的电子邮件和带动态模板的HTML电子邮件的功能。

最后，为了验证我们的应用程序向SendGrid发送了正确的请求，我们使用MockServer编写了一个集成测试。