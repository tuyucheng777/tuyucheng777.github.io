---
layout: post
title:  OpenAPI生成器自定义模板
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

[OpenAPI Generator](https://www.baeldung.com/spring-boot-openapi-api-first-development)是一款工具，可让我们从REST API定义快速生成客户端和服务器代码，支持多种语言和框架。虽然大多数情况下生成的代码无需修改即可使用，但在某些情况下，我们需要对其进行自定义。

**在本教程中，我们将学习如何使用自定义模板来解决这些情况**。

## 2. OpenAPI生成器项目设置

在探索自定义之前，让我们快速概述一下此工具的典型使用场景：从给定的API定义生成服务器端代码。**我们假设我们已经有一个使用[Maven](https://www.baeldung.com/maven-ebook)构建的基本[Spring Boot MVC](https://www.baeldung.com/spring-mvc-tutorial)应用程序，因此我们将使用适当的插件**：

```xml
<plugin>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator-maven-plugin</artifactId>
    <version>7.7.0</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <inputSpec>${project.basedir}/src/main/resources/api/quotes.yaml</inputSpec>
                <generatorName>spring</generatorName>
                <supportingFilesToGenerate>ApiUtil.java</supportingFilesToGenerate>
                <templateResourcePath>${project.basedir}/src/templates/JavaSpring</templateResourcePath>
                <configOptions>
                    <dateLibrary>java8</dateLibrary>
                    <openApiNullable>false</openApiNullable>
                    <delegatePattern>true</delegatePattern>
                    <apiPackage>cn.tuyucheng.taketoday.tutorials.openapi.quotes.api</apiPackage>
                    <modelPackage>cn.tuyucheng.taketoday.tutorials.openapi.quotes.api.model</modelPackage>
                    <documentationProvider>source</documentationProvider>
                </configOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

这样配置之后，生成的代码会进入到target/generated-sources/openapi文件夹中。另外我们的项目还需要添加对OpenAPI V3注解库的依赖：

```xml
<dependency>
    <groupId>io.swagger.core.v3</groupId>
    <artifactId>swagger-annotations</artifactId>
    <version>2.2.3</version>
</dependency>
```

插件和依赖的最新版本可在Maven Central上找到：

- [openapi-generator-maven-plugin](https://mvnrepository.com/artifact/org.openapitools/openapi-generator-maven-plugin)
- [swagger-annotations](https://mvnrepository.com/artifact/io.swagger.core.v3/swagger-annotations)

本教程的API包含一个GET操作，该操作返回给定金融工具符号的报价：

```yaml
openapi: 3.0.0
info:
    title: Quotes API
    version: 1.0.0
servers:
    - description: Test server
      url: http://localhost:8080
paths:
    /quotes/{symbol}:
        get:
            tags:
                - quotes
            summary: Get current quote for a security
            operationId: getQuote
            parameters:
                - name: symbol
                  in: path
                  required: true
                  description: Security's symbol
                  schema:
                      type: string
                      pattern: '[A-Z0-9]+'
            responses:
                '200':
                    description: OK
                    content:
                        application/json:
                            schema:
                                $ref: '#/components/schemas/QuoteResponse'
components:
    schemas:
        QuoteResponse:
            description: Quote response
            type: object
            properties:
                symbol:
                    type: string
                    description: security's symbol
                price:
                    type: number
                    description: Quote value
                timestamp:
                    type: string
                    format: date-time
```

即使没有任何书面代码，由于QuotesApi的默认实现，最终的项目已经可以提供API调用-尽管由于该方法未实现，它总是会返回502错误。

## 3. API实现

下一步是编写QuotesApiDelegate接口的实现，**由于我们使用委托模式，因此我们无需担心MVC或OpenAPI特定的注解，因为它们将在生成的控制器中分开保存**。

这种方法确保，如果我们稍后决定向项目添加[SpringDoc](https://www.baeldung.com/spring-rest-openapi-documentation)或类似的库，这些库所依赖的注解将始终与API定义同步。另一个好处是，契约修改也会更改委托接口，从而使项目无法构建。**这很好，因为它最大限度地减少了代码优先方法中可能发生的运行时错误**。

在我们的例子中，实现由一个使用BrokerService检索报价的方法组成：

```java
@Component
public class QuotesApiImpl implements QuotesApiDelegate {

    // ... fields and constructor omitted

    @Override
    public ResponseEntity<QuoteResponse> getQuote(String symbol) {
        var price = broker.getSecurityPrice(symbol);
        var quote = new QuoteResponse();
        quote.setSymbol(symbol);
        quote.setPrice(price);
        quote.setTimestamp(OffsetDateTime.now(clock));
        return ResponseEntity.ok(quote);
    }
}
```

我们还注入了一个[Clock](https://www.baeldung.com/java-clock)来提供返回的QuoteResponse所需的时间戳字段，这是一个很小的实现细节，可以更轻松地对使用当前时间的代码进行单元测试。例如，我们可以使用Clock.fixed()模拟被测代码在特定时间点的行为。[实现类的单元测试](https://github.com/eugenp/tutorials/blob/master/spring-boot-modules/spring-boot-openapi/src/test/java/com/baeldung/tutorials/openapi/quotes/controller/QuotesApiImplUnitTest.java)使用此方法。

最后，我们将实现一个仅返回随机报价的BrokerService，这对于我们的目的来说已经足够了。

我们可以通过运行集成测试来验证此代码是否按预期工作：

```java
@Test
void whenGetQuote_thenSuccess() {
    var response = restTemplate.getForEntity("http://localhost:" + port + "/quotes/BAEL", QuoteResponse.class);
    assertThat(response.getStatusCode())
        .isEqualTo(HttpStatus.OK);
}
```

## 4. OpenAPI生成器自定义场景

到目前为止，我们已经实现了一项没有自定义的服务。让我们考虑以下场景：作为API定义作者，我想指定给定操作可以返回缓存结果。OpenAPI规范通过一种称为[供应商扩展](https://swagger.io/docs/specification/openapi-extensions/)的机制允许这种非标准行为，该机制可应用于许多(但不是全部)元素。

在我们的示例中，我们将定义一个x-spring-cacheable扩展，以应用于我们想要具有此行为的任何操作。这是应用此扩展的初始API的修改版本：

```yaml
# ... other definitions omitted
paths:
    /quotes/{symbol}:
        get:
            tags:
                - quotes
            summary: Get current quote for a security
            operationId: getQuote
            x-spring-cacheable: true
            parameters:
# ... more definitions omitted
```

现在，如果我们再次使用mvn generate-sources运行生成器，什么也不会发生。**这是意料之中的，因为尽管仍然有效，但生成器不知道如何处理此扩展**。更准确地说，生成器使用的模板没有使用任何扩展。

仔细检查生成的代码后，我们发现，通过在与具有扩展的API操作匹配的委托接口方法上添加[@Cacheable](https://www.baeldung.com/spring-cache-tutorial)注解，我们可以实现目标。接下来让我们探索如何做到这一点。

### 4.1 自定义选项

OpenAPI Generator工具支持两种自定义方法：

- 添加新的自定义生成器，从头开始创建或扩展现有生成器
- 使用自定义模板替换现有生成器使用的模板

第一个选项更“重量级”，但允许完全控制生成的工件。当我们的目标是支持新框架或语言的代码生成时，这是唯一的选择，但我们不会在这里介绍它。

目前，我们需要做的就是更改单个模板，这是第二种选择。那么，**第一步就是找到这个模板**。[官方文档](https://openapi-generator.tech/docs/templating#retrieving-templates)建议使用该工具的CLI版本来提取给定生成器的所有模板。

但是，使用Maven插件时，直接在[GitHub仓库](https://github.com/OpenAPITools/openapi-generator/tree/v7.3.0/modules/openapi-generator/src/main/resources)中查找通常更方便。**请注意，为了确保兼容性，我们选择了与正在使用的插件版本相对应的标签的源代码树**。

在resources文件夹中，每个子文件夹都有用于特定生成器目标的模板。对于基于Spring的项目，文件夹名称为JavaSpring。在那里，我们会找到用于呈现服务器代码的[Mustache模板](https://www.baeldung.com/mustache)。大多数模板的命名都很合理，因此不难找出我们需要哪一个：apiDelegate.mustache。

### 4.2 模板自定义

**一旦找到了要自定义的模板，下一步就是将它们放入我们的项目中，以便Maven插件可以使用它们**。我们将即将自定义的模板放在src/templates/JavaSpring文件夹下，这样它就不会与其他源或资源混合。

接下来，我们需要向插件添加一个配置选项来指定我们的目录：

```xml
<configuration>
    <inputSpec>${project.basedir}/src/main/resources/api/quotes.yaml</inputSpec>
    <generatorName>spring</generatorName>
    <supportingFilesToGenerate>ApiUtil.java</supportingFilesToGenerate>
    <templateResourcePath>${project.basedir}/src/templates/JavaSpring</templateResourcePath>
    ... other unchanged properties omitted
</configuration>
```

为了验证生成器是否配置正确，让我们在模板顶部添加注解并重新生成代码：

```java
/*
* Generated code: do not modify !
* Custom template with support for x-spring-cacheable extension
*/
package {{package}};
... more template code omitted
```

接下来，运行mvn clean generate-sources将产生一个新版本的QuotesDelegateApi，并带有注解：

```java
/*
* Generated code: do not modify!
* Custom template with support for x-spring-cacheable extension
*/
package cn.tuyucheng.taketoday.tutorials.openapi.quotes.api;

... more code omitted
```

**这表明生成器选择了我们的自定义模板而不是原生模板**。

### 4.3 探索基础模板

现在，让我们看一下模板，找到添加自定义项的正确位置。我们可以看到，有一个由{{#operation}}{{/operation}}标记定义的部分，用于输出呈现的类中的委托方法：

```java
{{#operation}}
    // ... many mustache tags omitted
    {{#jdk8-default-interface}}default // ... more template logic omitted 

{{/operation}}
```

在本节中，模板使用当前上下文的几个属性(操作)来生成相应方法的声明。

具体来说，我们可以在{{vendorExtension}}下找到有关供应商扩展的信息。这是一个Map，其中的键是扩展名，值是定义中放入的任何数据的直接表示。**这意味着我们可以使用值为任意对象或只是简单字符串的扩展**。

要获取生成器传递给模板引擎的完整数据结构的JSON表示形式，请将以下globalProperties元素添加到插件的配置中：

```xml
<configuration>
    <inputSpec>${project.basedir}/src/main/resources/api/quotes.yaml</inputSpec>
    <generatorName>spring</generatorName>
    <supportingFilesToGenerate>ApiUtil.java</supportingFilesToGenerate>
    <templateResourcePath>${project.basedir}/src/templates/JavaSpring</templateResourcePath>
    <globalProperties>
        <debugOpenAPI>true</debugOpenAPI>
        <debugOperations>true</debugOperations>
    </globalProperties>
...more configuration options omitted
```

现在，当我们再次运行mvn generate-sources时，输出将在消息## Operation Info##之后立即显示以下JSON表示形式：

```text
[INFO] ############ Operation info ############
[ {
  "appVersion" : "1.0.0",
... many, many lines of JSON omitted
```

### 4.4 为操作添加@Cacheable

现在我们准备添加所需的逻辑来支持缓存操作结果，**一个可能有用的方面是允许用户指定缓存名称，但不要求他们这样做**。

为了满足这一要求，我们将支持两种供应商扩展。如果值仅为true，则将使用默认缓存名称：

```yaml
paths:
    /some/path:
        get:
            operationId: getSomething
            x-spring-cacheable: true
```

否则，它将期望一个具有name属性的对象，我们将其用作缓存名称：

```yaml
paths:
    /some/path:
        get:
            operationId: getSomething
            x-spring-cacheable:
                name: mycache
```

修改后的模板具有支持两种变体所需的逻辑，如下所示：

```java
{{#vendorExtensions.x-spring-cacheable}}
@org.springframework.cache.annotation.Cacheable({{#name}}"{{.}}"{{/name}}{{^name}}"default"{{/name}})
{{/vendorExtensions.x-spring-cacheable}}
{{#jdk8-default-interface}}default // ... template logic omitted
```

**我们添加了在方法签名定义之前添加注解的逻辑**，请注意使用{{#vendorExtensions.x-spring-cacheable}}来访问扩展值。**根据Mustache规则，只有当值为“true”时才会执行内部代码，即在布尔上下文中计算结果为true的内容**。尽管这个定义有些松散，但它在这里运行良好，并且可读性很强。

至于注解本身，我们选择使用“default”作为默认缓存名称。这允许我们进一步自定义缓存，尽管如何执行此操作的细节超出了本教程的范围。

## 5. 使用修改后的模板

最后，让我们修改API定义以使用我们的扩展：

```yaml
... more definitions omitted
paths:
    /quotes/{symbol}:
        get:
            tags:
                - quotes
            summary: Get current quote for a security
            operationId: getQuote
            x-spring-cacheable: true
                name: get-quotes
```

让我们再次运行mvn generate-sources来创建新版本的QuotesApiDelegate：

```java
... other code omitted
@org.springframework.cache.annotation.Cacheable("get-quotes")
default ResponseEntity<QuoteResponse> getQuote(String symbol) {
... default method's body omitted
```

我们看到委托接口现在具有@Cacheable注解。此外，我们看到缓存名称与API定义中的name属性相对应。

**现在，为了使此注解生效，我们还需要将@EnableCaching注解添加到@Configuration类中，或者像我们的例子一样，添加到主类中**：

```java
@SpringBootApplication
@EnableCaching
public class QuotesApplication {
    public static void main(String[] args) {
        SpringApplication.run(QuotesApplication.class, args);
    }
}
```

为了验证缓存是否按预期工作，让我们编写一个多次调用API的集成测试：

```java
@Test
void whenGetQuoteMultipleTimes_thenResponseCached() {
    var quotes = IntStream.range(1, 10).boxed()
            .map((i) -> restTemplate.getForEntity("http://localhost:" + port + "/quotes/BAEL", QuoteResponse.class))
            .map(HttpEntity::getBody)
            .collect(Collectors.groupingBy((q -> q.hashCode()), Collectors.counting()));

    assertThat(quotes.size()).isEqualTo(1);
}
```

我们期望所有响应都返回相同的值，因此我们将收集它们并按其哈希码对它们进行分组。如果所有响应都产生相同的哈希码，则生成的Map将只有一个条目。**请注意，此策略有效，因为生成的模型类使用所有字段实现了hashCode()方法**。

## 6. 总结

在本文中，我们展示了如何配置OpenAPI Generator工具以使用添加对简单供应商扩展的支持的自定义模板。