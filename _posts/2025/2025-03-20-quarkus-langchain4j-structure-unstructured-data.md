---
layout: post
title:  使用Camel Quarkus和Quarkus LangChain4j将非结构化数据转换为结构化数据
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

处理非结构化内容时，数据提取是一项常见挑战，我们可以使用[大语言模型](https://www.baeldung.com/cs/large-language-models)来解决这一挑战。

在本文中，我们将学习如何使用[Apache Camel](https://www.baeldung.com/apache-camel-intro)构建集成管道。我们将使用[LangChain4j](https://www.baeldung.com/java-langchain-basics)将HTTP端点与LLM集成，并使用[Quarkus](https://www.baeldung.com/quarkus-io)作为框架来一起运行所有组件。

我们还将回顾如何创建使用LLM作为构建数据的组件之一的集成路线。

本文基于[Alexandre Gallice](https://github.com/aldettinger)的文章[使用Apache Camel Quarkus和LangChain4j进行非结构化数据提取](https://camel.apache.org/blog/2024/09/data-extraction-example/)。要阅读有关此主题的更多信息，还请查看后续文章[通过接口解析LangChain4j AI服务](https://camel.apache.org/blog/2024/12/langchain4j-bean-interface/)。

## 2. 组件介绍

让我们回顾一下能够帮助我们处理集成管道的每个组件。

### 2.1 Quarkus

Quarkus是一个Kubernetes原生Java框架，针对构建和部署云原生应用程序进行了优化。我们可以使用它来开发高性能、轻量级的应用程序，这些应用程序启动速度快，占用的内存极少。**我们将使用Quarkus作为框架来运行我们的集成应用程序**。

### 2.2 LangChain4j

LangChain4j是一个Java库，旨在与应用程序中的大语言模型配合使用，**我们将使用它向LLM发送提示来构建内容**。此外，LangChain4j与Quarkus有很好的[集成](https://www.baeldung.com/java-quarkus-langchain4j)。

### 2.3 OpenAI

OpenAI是一家专注于创造和推进人工智能技术的AI研发公司。我们可以使用OpenAI的模型(如GPT)来执行语言生成、数据分析和对话式AI等任务。**我们将使用它从非结构化内容中提取数据**。

### 2.4 Apache Camel

Apache Camel是一个集成框架，可简化不同系统和应用程序的连接。**我们可以使用它来构建复杂的工作流，通过定义路由来在各个端点之间移动和转换数据**。

## 3. HTTP源与同步响应的集成

让我们构建一个集成应用程序，它将处理具有非结构化内容的HTTP调用，提取数据并返回结构化响应。

### 3.1 依赖

我们将从添加依赖开始，我们添加[jsonpath依赖](https://mvnrepository.com/artifact/org.apache.camel.quarkus/camel-quarkus-jsonpath)，它将帮助我们在集成管道中提取JSON内容：

```xml
<dependency>
    <groupId>org.apache.camel.quarkus</groupId>
    <artifactId>camel-quarkus-jsonpath</artifactId>
    <version>${camel-quarkus.version}</version>
</dependency>
```

接下来，我们添加[camel-quarkus-langchain4j依赖](https://mvnrepository.com/artifact/org.apache.camel.quarkus/camel-quarkus-langchain4j)以在我们的路由中支持LangChain4j处理程序：

```xml
<dependency>
    <groupId>org.apache.camel.quarkus</groupId>
    <artifactId>camel-quarkus-langchain4j</artifactId>
    <version>${quarkus-camel-langchain4j.version}</version>
</dependency>
```

最后，我们添加[camel-quarkus-platform-http依赖](https://mvnrepository.com/artifact/org.apache.camel.quarkus/camel-quarkus-platform-http)以支持HTTP端点作为我们路由的数据输入：

```xml
<dependency>
    <groupId>org.apache.camel.quarkus</groupId>
    <artifactId>camel-quarkus-platform-http</artifactId>
    <version>${camel-quarkus.version}</version>
</dependency>
```

### 3.2 结构化服务

现在，让我们创建一个StructurizingService，并在其中添加提示逻辑：

```java
@RegisterAiService
@ApplicationScoped
public interface StructurizingService {

    String EXTRACT_PROMPT = """
            Extract information about a patient from the text delimited by triple backticks: ```{text}```.
            The customerBirthday field should be formatted as {dateFormat}.
            The summary field should concisely relate the patient visit reason.
            The expected fields are: patientName, patientBirthday, visitReason, allergies, medications.
            Return only a data structure without format name.
            """;

    @UserMessage(EXTRACT_PROMPT)
    @Handler
    String structurize(@JsonPath("$.content") String text, @Header("expectedDateFormat") String dateFormat);
}
```

我们添加了structurize()方法来构建聊天模型请求，**我们使用EXTRACT_PROMPT文本作为提示的模板，我们将从输入参数中提取非结构化文本并将其添加到聊天消息中**。此外，我们将从第二个方法参数中获取日期格式，我们将该方法标记为Apache Camel Route @Handler，这样我们就可以在路由构建器中使用它，而无需指定方法名称。

### 3.3 路线构建器

我们使用[路由](https://www.baeldung.com/apache-camel-intro#defining-a-route)来指定集成管道，我们可以使用XML配置或带有[RouteBuilder](https://camel.apache.org/manual/route-builder.html)的Java DSL创建路由。

让我们使用RouteBuilder来配置我们的管道：

```java
@ApplicationScoped
public class Routes extends RouteBuilder {

    @Inject
    StructurizingService structurizingService;

    @Override
    public void configure() {
        from("platform-http:/structurize?produces=application/json")
                .log("A document has been received by the camel-quarkus-http extension: ${body}")
                .setHeader("expectedDateFormat", constant("YYYY-MM-DD"))
                .bean(structurizingService)
                .transform()
                .body();
    }
}
```

在路由配置中，我们添加了HTTP端点作为数据源。**我们创建了一个具有日期格式的预配置标头，并附加了StructurizingServiceBean来处理请求，将输出主体转换为路由响应**。

### 3.4 测试路由

现在，让我们调用新的端点并检查它如何处理非结构化数据：

```java
@QuarkusTest
class CamelStructurizeAPIResourceLiveTest {
    Logger logger = LoggerFactory.getLogger(CamelStructurizeAPIResourceLiveTest.class);

    String questionnaireResponses = """
            Operator: Could you provide your name?
            Patient: Hello, My name is Sara Connor.
            //The rest of the conversation...
            """;

    @Test
    void givenHttpRouteWithStructurizingService_whenSendUnstructuredDialog_thenExpectedStructuredDataIsPresent() throws JsonProcessingException {
        ObjectWriter writer = new ObjectMapper().writer();
        String requestBody = writer.writeValueAsString(Map.of("content", questionnaireResponses));

        Response response = RestAssured.given()
                .when()
                .contentType(ContentType.JSON)
                .body(requestBody)
                .post("/structurize");

        logger.info(response.prettyPrint());

        response
                .then()
                .statusCode(200)
                .body("patientName", containsString("Sara Connor"))
                .body("patientBirthday", containsString("1986-07-10"))
                .body("visitReason", containsString("Declaring an accident on main vehicle"));
    }
}
```

我们调用了structurize端点；然后，我们发送了患者与医疗服务运营商之间的对话。在响应中，我们获得了结构化数据，并验证了我们是否在预期字段中拥有有关患者的信息。

此外，我们已经记录了整个响应，因此让我们看一下输出：

```json
{
    "patientName": "Sara Connor",
    "patientBirthday": "1986-07-10",
    "visitReason": "Declaring an accident on main vehicle",
    "allergies": "Allergic to penicillin; mild reactions to certain over-the-counter antihistamines",
    "medications": "Lisinopril 10 mg, multivitamin, Vitamin D occasionally"
}
```

**我们可以看到，所有内容都是结构化的，并以JSON格式返回**。

## 4. 总结

在本文中，我们讨论了如何使用Quarkus、Apache Camel和LangChain4j来构造内容。借助Apache Camel，我们可以访问各种数据源，从而为内容创建转换管道。使用LangChain4j，我们可以实现数据结构化过程并将其集成到我们的管道中。