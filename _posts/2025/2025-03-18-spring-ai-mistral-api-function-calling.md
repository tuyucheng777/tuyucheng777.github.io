---
layout: post
title:  使用Mistral AI API在Java和Spring AI中进行函数调用
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

使用[大语言模型](https://www.baeldung.com/cs/large-language-models)，我们可以检索大量有用的信息。我们可以了解有关任何事物的许多新事实，并根据互联网上现有的数据获得答案。我们可以要求它们处理输入数据并执行各种操作，但是，如果我们要求模型使用API来准备输出呢？

为此，我们可以使用函数调用。函数调用允许LLM与数据交互并操纵数据、执行计算或检索超出其固有文本功能的信息。

在本文中，我们将探讨什么是函数调用以及如何使用它来将LLM与我们的内部逻辑集成。作为模型提供者，我们将使用[Mistral AI API](https://mistral.ai/)。

## 2. Mistral AI API

[Mistral AI](https://www.baeldung.com/cs/top-llm-comparative-analysis#4-mistral-by-mistral-ai)专注于为开发者和企业提供开放且可移植的生成式AI模型。我们可以使用它进行简单的提示，也可以使用它进行函数调用集成。

### 2.1 检索API Key

要开始使用[Mistral API](https://www.baeldung.com/cs/top-llm-comparative-analysis#4-mistral-by-mistral-ai)，我们首先需要检索API Key，让我们转到[API Key管理控制台](https://console.mistral.ai/api-keys/)：

![](/assets/images/2025/springai/springaimistralapifunctioncalling01.png)

要激活任何Key，我们必须设置[计费配置](https://console.mistral.ai/billing/)或使用试用期(如果可用)：

![](/assets/images/2025/springai/springaimistralapifunctioncalling02.png)

**一切解决后，我们可以按“Create new key”按钮来获取Mistral API Key**。

### 2.2 使用示例

**让我们从一个简单的提示开始**，我们将要求Mistral API返回患者状态列表，让我们实现这样的调用：

```java
@Test
void givenHttpClient_whenSendTheRequestToChatAPI_thenShouldBeExpectedWordInResponse() throws IOException, InterruptedException {

    String apiKey = System.getenv("MISTRAL_API_KEY");
    String apiUrl = "https://api.mistral.ai/v1/chat/completions";
    String requestBody = "{"
            + "\"model\": \"mistral-large-latest\","
            + "\"messages\": [{\"role\": \"user\", "
            + "\"content\": \"What the patient health statuses can be?\"}]"
            + "}";

    HttpClient client = HttpClient.newHttpClient();
    HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(apiUrl))
            .header("Content-Type", "application/json")
            .header("Accept", "application/json")
            .header("Authorization", "Bearer " + apiKey)
            .POST(HttpRequest.BodyPublishers.ofString(requestBody))
            .build();

    HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
    String responseBody = response.body();
    logger.info("Model response: " + responseBody);

    Assertions.assertThat(responseBody)
            .containsIgnoringCase("healthy");
}
```

我们创建了一个HTTP请求并将其发送到/chat/completions端点。**然后，我们使用API Key作为Authorization标头值**。正如预期的那样，在响应中我们既看到了元数据，也看到了内容本身：

```text
Model response: {"id":"585e3599275545c588cb0a502d1ab9e0","object":"chat.completion",
"created":1718308692,"model":"mistral-large-latest",
"choices":[{"index":0,"message":{"role":"assistant","content":"Patient health statuses can be
categorized in various ways, depending on the specific context or medical system being used.
However, some common health statuses include:
1.Healthy: The patient is in good health with no known medical issues.
...
10.Palliative: The patient is receiving care that is focused on relieving symptoms and improving quality of life, rather than curing the underlying disease.",
"tool_calls":null},"finish_reason":"stop","logprobs":null}],
"usage":{"prompt_tokens":12,"total_tokens":291,"completion_tokens":279}}
```

[函数调用](https://docs.mistral.ai/capabilities/function_calling/)的例子比较复杂，在调用之前需要做很多准备工作，我们将在下一节中探索它。

## 3. Spring AI集成

让我们看几个使用Mistral API进行函数调用的示例。**使用[Spring AI](https://www.baeldung.com/spring-ai)，我们可以避免大量的准备工作，让框架为我们完成**。

### 3.1 依赖

所需的依赖项位于[Spring Milestones](https://repo.spring.io/milestone)仓库中，让我们将其添加到pom.xml中：

```xml
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring milestones</name>
        <url>https://repo.spring.io/milestone</url>
    </repository>
</repositories>
```

现在，让我们添加Mistral API集成的[依赖](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-mistral-ai-spring-boot-starter)：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-mistral-ai-spring-boot-starter</artifactId>
    <version>0.8.1</version>
</dependency>
```

### 3.2 配置

现在让我们将之前获得的API Key添加到属性文件中：

```yaml
spring:
    ai:
        mistralai:
            api-key: ${MISTRAL_AI_API_KEY}
            chat:
                options:
                    model: mistral-small-latest
```

这就是我们开始使用Mistral API所需要的全部内容。

### 3.3 只有一个函数的用例

**在我们的演示示例中，我们将创建一个根据患者的ID返回其健康状况的函数**。

让我们从创建病人[记录](https://www.baeldung.com/java-record-keyword)开始：

```java
public record Patient(String patientId) {
}
```

现在让我们为患者的健康状况创建另一条记录：

```java
public record HealthStatus(String status) {
}
```

下一步，我们将创建一个配置类：

```java
@Configuration
public class MistralAIFunctionConfiguration {
    public static final Map<Patient, HealthStatus> HEALTH_DATA = Map.of(
            new Patient("P001"), new HealthStatus("Healthy"),
            new Patient("P002"), new HealthStatus("Has cough"),
            new Patient("P003"), new HealthStatus("Healthy"),
            new Patient("P004"), new HealthStatus("Has increased blood pressure"),
            new Patient("P005"), new HealthStatus("Healthy"));

    @Bean
    @Description("Get patient health status")
    public Function<Patient, HealthStatus> retrievePatientHealthStatus() {
        return (patient) -> new HealthStatus(HEALTH_DATA.get(patient).status());
    }
}
```

在这里，我们指定了包含患者健康数据的数据集。此外，我们还创建了retrievePatientHealthStatus()函数，该函数返回给定患者ID的健康状况。

现在，让我们通过在集成中调用它来测试我们的函数：

```java
@Import(MistralAIFunctionConfiguration.class)
@ExtendWith(SpringExtension.class)
@SpringBootTest
public class MistralAIFunctionCallingManualTest {

    @Autowired
    private MistralAiChatModel chatClient;

    @Test
    void givenMistralAiChatClient_whenAskChatAPIAboutPatientHealthStatus_thenExpectedHealthStatusIsPresentInResponse() {
        var options = MistralAiChatOptions.builder()
                .withFunction("retrievePatientHealthStatus")
                .build();

        ChatResponse paymentStatusResponse = chatClient.call(
                new Prompt("What's the health status of the patient with id P004?",  options));

        String responseContent = paymentStatusResponse.getResult().getOutput().getContent();
        logger.info(responseContent);

        Assertions.assertThat(responseContent)
                .containsIgnoringCase("has increased blood pressure");
    }
}
```

我们导入了MistralAIFunctionConfiguration类，以将我们的retrievePatientHealthStatus()函数添加到测试Spring上下文中。我们还注入了MistralAiChatClient，它将由Spring AI Starter自动实例化。

**在对聊天API的请求中，我们指定了包含其中一个患者ID的提示文本以及用于检索健康状况的函数名称，然后我们调用API并验证响应是否包含预期的健康状况**。

此外，我们记录了整个响应文本，如下所示：

```text
The patient with id P004 has increased blood pressure.
```

### 3.4 具有多个函数的用例

**我们还可以指定多个函数，AI会根据我们发送的提示来决定使用哪一个函数**。 

为了证明这一点，让我们扩展HealthStatus记录：

```java
public record HealthStatus(String status, LocalDate changeDate) {
}
```

我们添加了上次更改状态的日期。

现在我们来修改一下配置类：

```java
@Configuration
public class MistralAIFunctionConfiguration {
    public static final Map<Patient, HealthStatus> HEALTH_DATA = Map.of(
            new Patient("P001"), new HealthStatus("Healthy",
                    LocalDate.of(2024,1, 20)),
            new Patient("P002"), new HealthStatus("Has cough",
                    LocalDate.of(2024,3, 15)),
            new Patient("P003"), new HealthStatus("Healthy",
                    LocalDate.of(2024,4, 12)),
            new Patient("P004"), new HealthStatus("Has increased blood pressure",
                    LocalDate.of(2024,5, 19)),
            new Patient("P005"), new HealthStatus("Healthy",
                    LocalDate.of(2024,6, 1)));

    @Bean
    @Description("Get patient health status")
    public Function<Patient, String> retrievePatientHealthStatus() {
        return (patient) -> HEALTH_DATA.get(patient).status();
    }

    @Bean
    @Description("Get when patient health status was updated")
    public Function<Patient, LocalDate> retrievePatientHealthStatusChangeDate() {
        return (patient) -> HEALTH_DATA.get(patient).changeDate();
    }
}
```

我们为每个状态元素填充了更改日期，并创建了retrievePatientHealthStatusChangeDate()函数，该函数返回有关状态更改日期的信息。

让我们看看如何通过Mistral API使用这两个新函数：

```java
@Test
void givenMistralAiChatClient_whenAskChatAPIAboutPatientHealthStatusAndWhenThisStatusWasChanged_thenExpectedInformationInResponse() {
    var options = MistralAiChatOptions.builder()
            .withFunctions(
                    Set.of("retrievePatientHealthStatus",
                            "retrievePatientHealthStatusChangeDate"))
            .build();

    ChatResponse paymentStatusResponse = chatClient.call(
            new Prompt(
                    "What's the health status of the patient with id P005",
                    options));

    String paymentStatusResponseContent = paymentStatusResponse.getResult()
            .getOutput().getContent();
    logger.info(paymentStatusResponseContent);

    Assertions.assertThat(paymentStatusResponseContent)
            .containsIgnoringCase("healthy");

    ChatResponse changeDateResponse = chatClient.call(
            new Prompt(
                    "When health status of the patient with id P005 was changed?",
                    options));

    String changeDateResponseContent = changeDateResponse.getResult().getOutput().getContent();
    logger.info(changeDateResponseContent);

    Assertions.assertThat(paymentStatusResponseContent)
            .containsIgnoringCase("June 1, 2024");
}
```

**在本例中，我们指定了两个函数名称并发送了两个提示**。首先，我们询问患者的健康状况。然后我们询问此状态何时发生变化，我们已验证结果包含预期信息。除此之外，我们记录了所有响应，如下所示：

```text
The patient with id P005 is currently healthy.
The health status of the patient with id P005 was changed on June 1, 2024.
```

## 4. 总结

函数调用是扩展LLM功能的绝佳工具，我们还可以使用它将LLM与我们的逻辑集成在一起。

在本教程中，我们探索了如何通过调用一个或多个函数来实现基于LLM的流程。使用这种方法，我们可以实现与AI API集成的现代应用程序。