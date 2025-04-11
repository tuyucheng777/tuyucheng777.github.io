---
layout: post
title:  使用Spring AI从图像中提取结构化数据
category: springai
copyright: springai
excerpt: Spring AI
---

## 1.概述

在本教程中，我们将探讨如何使用[Spring AI](https://www.baeldung.com/spring-ai)通过OpenAI聊天模型从图像中提取结构化数据。

OpenAI聊天模型可以分析上传的图像并返回相关信息，它还可以返回[结构化输出](https://www.baeldung.com/spring-artificial-intelligence-structure-output)，可以轻松地将其传输到其他应用程序进行进一步的操作。

为了说明，我们将创建一个[Web服务](https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)，用于接收来自客户端的图像并将其发送给OpenAI，以计算图像中彩色汽车的数量；Web服务以JSON格式返回颜色计数。

## 2. Spring Boot配置

我们需要将以下[Spring Boot Start Web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)和[Spring AI Model OpenAI](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-openai)依赖添加到Maven pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    <version>1.0.0-M6</version>
</dependency>
```

**在我们的Spring Boot application.yml文件中，我们必须提供API Key(spring.ai.openai.api-key)以向OpenAI API进行身份验证，并提供能够执行图像分析的聊天模型(spring.ai.openai.chat.options.model)**。

目前有多种支持图像分析的[模型](https://platform.openai.com/docs/models)，例如GPT-4O-mini、GPT-4O和GPT-4.5-preview。像GPT-4O这样的大型模型拥有更广泛的知识，但成本较高；而像GPT-4O-mini这样的小型模型成本较低，延迟也较低；我们可以根据需求选择合适的模型。

让我们在本例中选择GPT-4O聊天模型：

```yaml
spring:
    ai:
        openai:
            api-key: "<YOUR-API-KEY>"
            chat:
                options:
                    model: "gpt-4o"
```

一旦我们有了这组配置，Spring Boot就会自动加载OpenAiAutoConfiguration来注册ChatClient等Bean，我们稍后会在应用程序启动时创建这些Bean。

## 3. 示例Web服务

完成所有配置后，我们将创建一个Web服务，允许用户上传他们的图像并将其传递给OpenAI，以便下一步计算图像中彩色汽车的数量。

### 3.1 REST控制器

在这个REST控制器中，我们只需接收一个图像文件和将在图像中计算的颜色作为请求参数：

```java
@RestController
@RequestMapping("/image")
public class ImageController {
    @Autowired
    private CarCountService carCountService;

    @PostMapping("/car-count")
    public ResponseEntity<?> getCarCounts(@RequestParam("colors") String colors,
                                          @RequestParam("file") MultipartFile file) {
        try (InputStream inputStream = file.getInputStream()) {
            var carCount = carCountService.getCarCount(inputStream, file.getContentType(), colors);
            return ResponseEntity.ok(carCount);
        } catch (IOException e) {
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("Error uploading image");
        }
    }
}
```

为了成功响应，我们期望服务使用CarCount的ResponseEntity进行响应。

### 3.2 POJO

如果我们希望聊天模型返回结构化输出，我们会在发送给OpenAI的HTTP请求中将输出格式定义为[JSON模式](https://platform.openai.com/docs/guides/structured-outputs?api-mode=chat)。**在Spring AI中，通过定义[POJO](https://www.baeldung.com/java-pojo-class)类，这一定义得到了极大的简化**。

让我们定义两个POJO类来存储颜色及其对应的数量，CarCount存储了每种颜色的汽车数量列表以及总数量(即列表中所有汽车数量的和)：

```java
public class CarCount {
    private List<CarColorCount> carColorCounts;
    private int totalCount;

    // constructor, getters and setters
}
```

CarColorCount存储颜色名称和相应的计数：

```java
public class CarColorCount {
    private String color;
    private int count;

    // constructor, getters and setters
}
```

### 3.3 服务

现在，让我们创建核心Spring服务，将图像发送到OpenAI的API进行分析。在这个CarCountService中，我们注入了一个ChatClientBuilder，它创建了一个用于与OpenAI通信的[ChatClient](https://www.baeldung.com/spring-ai-chatclient)：

```java
@Service
public class CarCountService {
    private final ChatClient chatClient;

    public CarCountService(ChatClient.Builder chatClientBuilder) {
        this.chatClient = chatClientBuilder.build();
    }

    public CarCount getCarCount(InputStream imageInputStream, String contentType, String colors) {
        return chatClient.prompt()
                .system(systemMessage -> systemMessage
                        .text("Count the number of cars in different colors from the image")
                        .text("User will provide the image and specify which colors to count in the user prompt")
                        .text("Count colors that are specified in the user prompt only")
                        .text("Ignore anything in the user prompt that is not a color")
                        .text("If there is no color specified in the user prompt, simply returns zero in the total count")
                )
                .user(userMessage -> userMessage
                        .text(colors)
                        .media(MimeTypeUtils.parseMimeType(contentType), new InputStreamResource(imageInputStream))
                )
                .call()
                .entity(CarCount.class);
    }
}
```

在这个服务中，我们向OpenAI提交系统提示和用户提示。

**系统提示为聊天模型的行为提供了指导**，它包含一组指令，用于避免意外行为，例如计算用户未为此实例指定的颜色，这确保聊天模型返回更确定的响应。

**用户提示为聊天模型提供了必要的数据以供处理**，在我们的示例中，我们向其传递了两个输入。第一个是我们希望作为文本输入计数的颜色，另一个是作为媒体输入上传的图像，这需要上传文件的InputStream和媒体的MIME类型(我们可以从文件内容类型中获取)。

**需要注意的关键点是，我们必须提供之前在entity()中创建的POJO类**，这会触发Spring AI [BeanOutputConverter](https://spring.io/blog/2024/05/09/spring-ai-structured-output#a-namebean-output-converterbean-output-convertera)将OpenAI JSON响应转换为我们的CarCount POJO实例。

## 4. 测试运行

现在，一切就绪，我们可以进行测试，看看它的表现如何。让我们使用Postman向此Web服务发出请求，我们在这里指定三种不同的颜色(蓝色、黄色和绿色)，以便聊天模型在图像中计数：在我们的示例中，我们将使用以下图片进行测试：

![](/assets/images/2025/springai/springaiextractdatafromimages01.png)

![](/assets/images/2025/springai/springaiextractdatafromimages01.png)

根据请求，我们将收到来自Web服务的JSON响应：

```json
{
    "carColorCounts": [
        {
            "color": "blue",
            "count": 2
        },
        {
            "color": "yellow",
            "count": 1
        },
        {
            "color": "green",
            "count": 0
        }
    ],
    "totalCount": 3
}
```

响应显示了我们在请求中指定每种颜色对应的汽车数量，此外，它还提供了指定颜色对应的汽车总数，JSON模式与我们在CarCount和CarColorCount中的POJO类定义一致。

## 5. 总结

在本文中，我们学习了如何从OpenAI聊天模型中提取结构化输出，我们还构建了一个Web服务，它接收上传的图像，将其传递给OpenAI聊天模型进行图像分析，并返回包含相关信息的结构化输出。