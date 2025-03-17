---
layout: post
title: 在Java中使用OpenAI DALL·E 3生成AI图像
category: springai
copyright: springai
excerpt: Spring AI
---

## 1. 概述

[人工智能](https://www.baeldung.com/cs/category/ai)正在改变我们构建Web应用程序的方式，人工智能的一个令人兴奋的应用是从基于文本的描述生成图像。[OpenAI的DALL·E 3](https://openai.com/index/dall-e-3/)是一种流行的文本转图像模型，可帮助我们实现这一点。

**在本教程中，我们将探索如何使用[Spring AI](https://www.baeldung.com/tag/spring-ai)通过OpenAI的DALL·E 3模型生成图像**。

要遵循本教程，我们需要一个[OpenAI API Key](https://platform.openai.com/api-keys)。

## 2. 设置项目

在开始生成AI图像之前，我们需要包含[Spring Boot Starter](https://www.baeldung.com/spring-boot-starters)依赖并正确配置我们的应用程序。

### 2.1 依赖

让我们首先将[spring-ai-openai-spring-boot-starter](https://mvnrepository.com/artifact/org.springframework.ai/spring-ai-openai/latest)依赖添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>org.springframework.ai</groupId>
    <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
    <version>1.0.0-M3</version>
</dependency>
```

由于当前版本1.0.0-M3是一个里程碑版本，我们还需要将Spring Milestones仓库添加到我们的pom.xml中：

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

上述Starter依赖为我们提供了从我们的应用程序与OpenAI服务交互并使用其DALL·E 3模型生成AI图像所需的类。

### 2.2 配置OpenAI API Key

现在，要与OpenAI服务交互，我们需要在application.yaml文件中配置API Key：

```yaml
spring:
    ai:
        openai:
            api-key: ${OPENAI_API_KEY}
```

我们使用${}属性占位符从[环境变量](https://www.baeldung.com/spring-boot-properties-env-variables#use-environment-variable-in-applicationyml-file)中加载我们的属性值。

**配置有效的API Key后，Spring AI将自动为我们创建一个ImageModel Bean**，我们将在服务层中自动注入它并发送请求以生成AI图像。

### 2.3 配置默认图像选项

接下来，我们还配置一些用于生成图像的默认图像选项：

```yaml
spring:
    ai:
        openai:
            image:
                options:
                    model: dall-e-3
                    size: 1024x1024
                    style: vivid
                    quality: standard
                    response-format: url
```

我们首先将dall-e-3配置为用于图像生成的模型。

为了得到完美的方形图像，我们指定1024 × 1024作为size；另外两个允许的size选项是1792 × 1024和1024 × 1792。

接下来，我们将style设置为vivid-告诉AI模型生成超现实和戏剧性的图像。另一个可用选项是natural，我们可以使用它来生成更自然、更少超现实的图像。

对于quality选项，我们将其设置为standard，这适用于大多数用例。但是，如果我们需要具有增强细节和更好一致性的图像，我们可以将值设置为hd。**需要注意的是，hd质量的图像将需要更多时间来生成**。

最后，我们将response-format选项设置为url，**生成的图像将可通过有效期为60分钟的URL访问。或者，我们可以将其值设置为b64_json，以[Base64编码](https://www.baeldung.com/java-base64-encode-and-decode)的字符串形式接收图像**。

在本教程的后面我们将研究如何覆盖这些默认图像选项。

## 3. 使用DALL·E 3生成AI图像

现在我们已经设置了应用程序，让我们创建一个ImageGenerator类，我们将自动注入ImageModel Bean并引用它来生成AI图像：

```java
public String generate(String prompt) {
    ImagePrompt imagePrompt = new ImagePrompt(prompt);
    ImageResponse imageResponse = imageModel.call(imagePrompt);
    return resolveImageContent(imageResponse);
}

private String resolveImageContent(ImageResponse imageResponse) {
    Image image = imageResponse.getResult().getOutput();
    return Optional
        .ofNullable(image.getUrl())
        .orElseGet(image::getB64Json);
}
```

在这里，我们创建一个generate()方法，它接收一个prompt字符串，表示我们要生成的图像的文本描述。

接下来，我们用prompt参数创建一个ImagePrompt对象。然后，我们将其传递给ImageModel Bean的call()方法，以发送图像生成请求。

**imageResponse将包含图像的URL或Base64编码的字符串表示形式，具体取决于我们之前在application.yaml文件中配置的response-format选项**。为了解析正确的输出属性，我们创建了一个resolveImageContent()工具方法并将其作为响应返回。

## 4. 覆盖默认图像选项

在某些情况下，我们可能想要覆盖我们在application.yaml文件中配置的默认图像选项。

让我们看看如何通过重载generate()方法来做到这一点：

```java
public String generate(ImageGenerationRequest request) {
    ImageOptions imageOptions = OpenAiImageOptions
        .builder()
        .withUser(request.username())
        .withHeight(request.height())
        .withWidth(request.width())
        .build();
    ImagePrompt imagePrompt = new ImagePrompt(request.prompt(), imageOptions);
    
    ImageResponse imageResponse = imageModel.call(imagePrompt);
    return resolveImageContent(imageResponse);
}

record ImageGenerationRequest(
    String prompt,
    String username,
    Integer height,
    Integer width
) {}
```

我们首先创建一个ImageGenerationRequest记录，除了保存我们的prompt之外，它还包含username和所需的图像height和width。

我们使用这些附加值创建一个ImageOptions实例并将其传递给ImagePrompt构造函数，需要注意的是，OpenAiImageOptions类没有size属性，因此我们分别提供height和width值。

**[user选项](https://platform.openai.com/docs/guides/safety-best-practices/end-user-ids#end-user-ids)帮助我们将图像生成请求与特定的最终用户联系起来。建议将其作为防止滥用的最佳安全做法**。

根据要求，我们还可以覆盖ImageOptions对象中的其他图像选项，如style、quality和response-format。

## 5. 测试我们的ImageGenerator类

现在我们已经实现了ImageGenerator类，让我们测试一下它：

```java
String prompt = """
            A cartoon depicting a gangster donkey wearing 
            sunglasses and eating grapes in a city street.
        """;
String response = imageGenerator.generate(prompt);
```

在这里，我们将prompt传递给ImageGenerator的generate()方法。经过短暂的处理时间后，我们将收到一个响应，其中包含我们生成的图像的URL或Base64编码字符串，具体取决于配置的response-format属性。

让我们看看DALL·E 3为我们生成了什么：

![](/assets/images/2025/springai/springaiopenaidalle301.png)

我们可以看到，生成的图像与我们的提示准确匹配。**这证明了DALL·E 3在理解自然语言描述并将其转换为图像方面的强大功能**。

## 6. 总结

在本文中，我们探讨了如何使用Spring AI根据文本描述生成AI图像，我们在底层使用了OpenAI的DALL·E 3模型。

我们介绍了必要的配置并开发了一个服务类来生成AI图像。此外，我们还研究了默认图像选项以及如何动态覆盖它们。

通过Spring AI将DALL·E 3集成到我们的Java应用程序中，我们可以轻松添加图像生成功能，而无需训练和托管我们自己的模型的开销。