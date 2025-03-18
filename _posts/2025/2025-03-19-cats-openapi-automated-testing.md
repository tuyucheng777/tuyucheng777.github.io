---
layout: post
title:  使用CATS自动测试OpenAPI端点
category: automatedtest
copyright: automatedtest
excerpt: CATS
---

## 1. 简介

在本教程中，我们将探索使用[CATS](https://endava.github.io/cats/)自动测试使用[OpenAPI](https://www.baeldung.com/spring-rest-openapi-documentation)配置的REST API。手动编写API测试可能很繁琐且耗时，但CATS通过自动生成和运行数百个测试简化了该过程。

通过在开发早期识别潜在问题，这可以减少人工工作量并提高API可靠性。即使是简单的API，也可能出现常见错误，而CATS可以帮助我们高效地找到并解决这些错误。

虽然CATS可与任何带有OpenAPI注解的应用程序一起使用，但我们将使用基于Spring和[Jackson](https://www.baeldung.com/jackson-annotations)的应用程序进行演示。

## 2. CATS让测试变得简单

CATS代表契约自动测试服务，契约指的是我们的REST API的OpenAPI规范。自动测试是一种[模糊测试](https://en.wikipedia.org/wiki/Fuzzing)，使用随机数据和某些情况下API操作返回的数据(如ID)。**它是一个外部CLI应用程序，需要访问我们的API的URL及其OpenAPI契约(以文件或URL的形式)**。

其主要特点包括：

- 根据API契约自动生成并运行测试
- 自动生成详细测试结果的HTML报告
- 授权要求的简单配置

**由于测试是自动生成的，因此无需维护**，只需在更改我们的OpenAPI规范时重新运行生成器即可。

**这对于具有许多端点的API尤其有用**。而且由于它包括模糊测试，因此它会生成我们从未考虑过的测试。

### 2.1 安装CATS

我们有几个[安装选项](https://endava.github.io/cats/docs/getting-started/installation)，最简单的两个是下载并运行[JAR](https://github.com/Endava/cats/releases)或[二进制](https://github.com/Endava/cats/releases)文件。我们将选择二进制选项，因为它不需要安装和配置Java的环境，从而更容易从任何地方运行测试。

**下载后，我们必须将cats二进制文件添加到我们的[环境路径](https://www.baeldung.com/linux/path-variable)中，以便从任何地方运行它**。

### 2.2 运行测试

我们需要指定至少两个[参数](https://endava.github.io/cats/docs/commands-and-arguments/arguments/)来运行cats：contract和server。在我们的例子中，OpenAPI规范URL位于[/api-docs](https://www.baeldung.com/spring-rest-openapi-documentation#1-openapi-description-path)：

```shell
$ cats --contract=http://localhost:8080/api-docs --server=http://localhost:8080
```

我们还可以将契约作为包含规范的JSON或YAML本地文件传递。

让我们检查一个示例，其中该文件位于我们运行CATS的同一目录中：

```bash
$ cats --contract=api-docs.yml --server=http://localhost:8080
```

默认情况下，CATS将对规范中的所有路径运行测试，但也可以使用模式匹配将其限制为仅几个路径：

```bash
$ cats --server=http://localhost:8080 --paths="/path/a*,/path/b"
```

**如果我们在广泛的规范中每次关注几个路径，这个参数将会很有帮助**。

### 2.3 包括Authorization Headers

通常，我们的API通过某种形式的[身份验证](https://endava.github.io/cats/docs/getting-started/authentication/)来[保护](https://www.baeldung.com/spring-boot-api-key-secret)。在这种情况下，我们可以在命令中包含授权标头。让我们检查一下使用[Bearer Authorization](https://swagger.io/docs/specification/v3_0/authentication/bearer-authentication/)时它是什么样子：

```bash
$ cats --server=http://localhost:8080 -H "Authorization=Bearer a-valid-token"
```

### 2.4 报告生成

运行后，它会在本地创建一个HTML报告：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting01.png)

稍后，我们将回顾一些错误以了解如何重构我们的代码。

## 3. 项目设置

为了展示CATS，我们将从一个简单的REST CRUD API开始，其中包含[@RestController](https://www.baeldung.com/spring-controller-vs-restcontroller)和Bearer Authorization。**必须包含[@ApiResponse](https://www.baeldung.com/swagger-operation-vs-apiresponse)注解，因为它们包含CATS使用的OpenAPI定义中的重要细节，例如媒体类型和未授权请求的预期状态代码**：

```java
@RestController
@RequestMapping("/api/item")
@ApiResponse(responseCode = "401", description = "Unauthorized", content = {
        @Content(mediaType = MediaType.TEXT_PLAIN_VALUE, schema =
        @Schema(implementation = String.class)
        )
})
public class ItemController {

    private ItemService service;

    // endpoints ...
}
```

我们的请求映射定义了最少数量的Swagger注解，尽可能依赖默认值：

```java
@PostMapping
@ApiResponse(responseCode = "200", description = "Success", content = {
        @Content(mediaType = MediaType.APPLICATION_JSON_VALUE, schema =
        @Schema(implementation = Item.class)
        )
})
public ResponseEntity<Item> post(@RequestBody Item item) {
    service.insert(item);
    return ResponseEntity.ok(item);
}

// GET and DELETE endpoints ...
```

对于我们的有效负载类，我们将包括一些基本属性：

```java
public class Item {

    private String id;
    private String name;
    private int value;

    // default getters and setters...
}
```

## 4. 报告中常见错误分析

让我们分析一下报告中出现的一些错误，以便解决它们。通常每个字段都会进行多次类似的测试，因此我们只会显示每个字段的详细页面。

### 4.1 缺少推荐的安全标头

有一组[OWASP推荐](https://owasp.org/www-project-secure-headers/)的安全标头，报告中的详细测试页面显示了我们应默认包含的标头：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting02.png)

[Spring Security](https://www.baeldung.com/spring-security-oauth-resource-server)[默认](https://docs.spring.io/spring-security/site/docs/4.0.2.RELEASE/reference/html/headers.html)包含所有这些标头，因此让我们在项目中包含[spring-boot-starter-security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>3.3.2</version>
</dependency>
```

我们的[SecurityFilterChain](https://www.baeldung.com/spring-security-custom-filter)中不需要特定的配置来包含安全标头，因此我们将使用JWT定义一个简单的配置，以便在运行cats时传递有效的令牌：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .oauth2ResourceServer(rs -> rs.jwt(jwt -> jwt.decoder(jwtDecoder())))
                .build();
    }
}
```

实现[jwtDecoder()](https://www.baeldung.com/spring-security-oauth-jwt)方法取决于我们的需求，我们可以使用任何其他使用授权标头的身份验证方法。

### 4.2 在请求字段中发送非常大的值或超出边界的值

当我们的字段指定了[最大长度](https://endava.github.io/cats/docs/fuzzers/field-fuzzers/string-right-boundary)时，CATS会发送更大的值，并期望服务器以4XX状态拒绝这些请求。未指定时，最大长度将回落到1万：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting03.png)

类似地，它发送具有[巨大值](https://endava.github.io/cats/docs/fuzzers/field-fuzzers/very-large-strings)和相同期望的请求：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting04.png)

让我们首先定制应用程序中使用的[ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)来解决这些问题。

JsonFactoryBuilder包含一个StreamReadConstraints配置，我们可以使用它来设置一些约束，包括String的最大长度。让我们**定义最大长度为100**：

```java
@Configuration
public class JacksonConfig {

    @Bean
    public ObjectMapper objectMapper() {
        JsonFactory factory = new JsonFactoryBuilder()
                .streamReadConstraints(
                        StreamReadConstraints.builder()
                                .maxStringLength(100)
                                .build()
                ).build();

        return new ObjectMapper(factory);
    }
}
```

当然，这个最大长度会根据我们应用程序的要求而有所不同。**最重要的是，虽然这会阻碍我们的应用程序接收超大的请求，但它不会在我们的API规范中定义约束**。

**为此，我们可以在有效负载类中包含一些[校验注解](https://www.baeldung.com/java-validation)**：

```java
@Size(min = 37, max = 37)
private String id;

@NotNull
@Size(min = 1, max = 20)
private String name;

@Min(1)
@Max(100)
@NotNull
private int value;
```

同样，这里的值取决于我们的要求，但包括这些边界有助于定义CATS如何生成测试。最后，为了拒绝无效请求，我们将修改POST方法以使用@Valid标注：

```java
ResponseEntity<Item> post(@Valid @RequestBody Item item) { 
    // ... 
}
```

### 4.3 格式错误的JSON和虚假请求

默认情况下，[Jackson](https://www.baeldung.com/jackson-object-mapper-tutorial)对请求非常宽容，甚至接受一些[格式错误的JSON](https://endava.github.io/cats/docs/fuzzers/http-fuzzers/malformed-json)：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting05.png)

为了防止这种情况，让我们回到我们的JacksonConfig并启用一个在尾随令牌上失败的选项：

```java
mapper.enable(DeserializationFeature.FAIL_ON_TRAILING_TOKENS);
```

它还会接受将不属于Item类的字段与属于Item类的字段[混合在一起](https://endava.github.io/cats/docs/fuzzers/field-fuzzers/new-fields-fuzzer)的请求，以及[虚拟请求](https://endava.github.io/cats/docs/fuzzers/http-fuzzers/dummy-request)和[空JSON体](https://endava.github.io/cats/docs/fuzzers/http-fuzzers/empty-json)，我们可以通过强制反序列化在未知属性上失败来摆脱这些问题：

```java
mapper.enable(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES);
```

### 4.4 整数中的小数

当我们有一个int属性时，Jackson将截断十进制值以适应：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting06.png)

例如，值0.34会截断为0。为了避免这种情况，让我们关闭此功能：

```java
mapper.disable(DeserializationFeature.ACCEPT_FLOAT_AS_INT);
```

### 4.5 值中的零宽度字符

一些模糊测试器在字段名称和值中包含零宽度字符：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting07.png)

**我们已经启用了FAIL_ON_UNKNOWN_PROPERTIES，因此我们需要对字段值进行一些清理并删除零宽度字符**。让我们为此使用自定义JSON反序列化器，首先使用一个实用程序类，该类为一些零宽度字符定义[正则表达式模式](https://www.baeldung.com/regular-expressions-java)：

```java
public class RegexUtils {

    private static final Pattern ZERO_WIDTH_PATTERN =
            Pattern.compile("[\u200B\u200C\u200D\u200F\u202B\u200E\uFEFF]");

    public static String removeZeroWidthChars(String value) {
        return value == null ? null
                : ZERO_WIDTH_PATTERN.matcher(value).replaceAll("");
    }
}
```

首先，我们在自定义[反序列化器](https://www.baeldung.com/jackson-deserialization)中使用它来处理字符串字段：

```java
public class ZeroWidthStringDeserializer extends JsonDeserializer<String> {

    @Override
    public String deserialize(JsonParser parser, DeserializationContext context)
            throws IOException {
        return RegexUtils.removeZeroWidthChars(parser.getText());
    }
}
```

然后，我们为整数字段创建另一个版本：

```java
public class ZeroWidthIntDeserializer extends JsonDeserializer<Integer> {

    @Override
    public Integer deserialize(JsonParser parser, DeserializationContext context) throws IOException {
        return Integer.valueOf(RegexUtils.removeZeroWidthChars(parser.getText()));
    }
}
```

最后，我们使用[@JsonDeserialize](https://www.baeldung.com/jackson-annotations#5-jsondeserialize)注解在Item字段中引用这些反序列化器：

```java
@JsonDeserialize(using = ZeroWidthStringDeserializer.class)
private String id;

@JsonDeserialize(using = ZeroWidthStringDeserializer.class)
private String name;

@JsonDeserialize(using = ZeroWidthIntDeserializer.class)
private int value;
```

### 4.6 错误的请求响应和模式

在我们进行到目前为止的更改之后，许多测试将导致“[Bad Request”](https://www.baeldung.com/rest-api-error-handling-best-practices#1-basic-responses)，因此我们需要在控制器中添加适当的@ApiResponse注解，以避免报告中出现警告。**此外，由于错误请求的JSON响应由Spring的BasicErrorController动态处理，我们需要创建一个类作为注解中的模式**：

```java
public class BadApiRequest {

    private long timestamp;
    private int status;
    private String error;
    private String path;

    // default getters and setters...
}
```

现在，我们可以在控制器中包含另一个定义：

```java
@ApiResponse(responseCode = "400", description = "Bad Request", content = {
    @Content(
        mediaType = MediaType.APPLICATION_JSON_VALUE, 
        schema = @Schema(implementation = BadApiRequest.class)
    )
})
```

## 5. 重构结果

重新运行报告时，我们可以看到我们的更改使错误减少了40％以上：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting08.png)

让我们回顾一下我们处理过的一些测试用例。我们现在包括默认安全标头：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting09.png)

拒绝格式错误的JSON：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting10.png)

并校验输入：

![](/assets/images/2025/automatedtest/catsopenapiautomatedtesting11.png)

因此，我们拥有一个总体上更安全的API。

## 6. 有用的子命令

CATS有[子命令](https://endava.github.io/cats/docs/commands-and-arguments/sub-commands)，我们可以使用它们来检查契约、重放测试等。让我们看几个有趣的命令。

### 6.1 检查API

列出API规范中定义的所有路径和操作：

```bash
$ cats list --paths -c http://localhost:8080/api-docs
```

此命令返回按路径分组的结果：

```text
2 paths and 4 operations:
◼ /api/v1/item: [POST, GET]
◼ /api/v1/item/{id}: [GET, DELETE]
```

### 6.2 重放测试

在修复错误期间，一个有用的命令是replay，它重新运行特定的测试：

```bash
cats replay Test216
```

我们可以通过查看报告获取测试编号并将其替换到命令中。**每个测试的详细报告还包括完整的重放命令，因此我们可以将其复制并粘贴到我们的终端中**。

## 7. 总结

在本文中，我们探讨了如何使用CATS进行自动化OpenAPI测试，从而显著减少手动工作量并提高测试覆盖率。通过应用添加安全标头、强制输入校验和配置严格反序列化等更改，我们的示例应用程序报告的错误数量减少了40%以上。