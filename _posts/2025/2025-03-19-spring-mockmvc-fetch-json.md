---
layout: post
title:  使用MockMVC获取JSON内容作为对象
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

在测试REST端点时，有时我们希望获取响应并将其转换为对象以进行进一步检查和验证。众所周知，一种方法是使用[RestAssured](https://www.baeldung.com/rest-assured-response)等库来验证响应，而无需将其转换为对象。

在本教程中，我们将探讨使用[MockMVC](https://www.baeldung.com/spring-boot-testing)和[Spring Boot](https://www.baeldung.com/spring-boot)将JSON内容作为对象获取的几种方法。

## 2. 示例设置

在深入研究之前，让我们创建一个用于测试的简单REST端点。

让我们从依赖设置开始，我们将[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)依赖添加到pom.xml中，以便我们可以创建REST端点：

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

接下来我们定义Article类：

```java
public class Article {
    private Long id;
    private String title;
    
    // standard getters and setters
}
```

更进一步，让我们创建具有两个端点的ArticleController，一个返回单个文章，另一个返回文章列表：

```java
@RestController
@RequestMapping
public class ArticleController {
    @GetMapping("/article")
    public Article getArticle() {
        return new Article(1L, "Learn Spring Boot");
    }

    @GetMapping("/articles")
    public List<Article> getArticles() {
        return List.of(new Article(1L, "Guide to JUnit"), new Article(2L, "Working with Hibernate"));
    }
}
```

## 3. 测试类

为了测试我们的控制器，我们将使用@WebMvcTest注解来修饰我们的测试类。**当我们使用此注解时，Spring Boot会自动配置MockMvc并仅为Web层启动上下文**。

此外，我们仅指定实例化ArticleController控制器的上下文，这在具有多个控制器的应用程序中很有用：

```java
@WebMvcTest(ArticleController.class)
class ArticleControllerUnitTest {
    @Autowired
    private MockMvc mockMvc;
}
```

我们还可以使用@AutoConfigureMockMvc注解配置MockMVC，但是，这种方法需要Spring Boot运行整个应用程序上下文，这会降低我们的测试运行速度。

现在我们已经完成所有设置，让我们探索如何使用MockMvc执行请求并将响应作为对象获取。

## 4. 使用Jackson

将JSON内容转换为对象的一种方法是使用[Jackson](https://www.baeldung.com/jackson)库。

### 4.1 获取单个对象

让我们创建一个测试来验证HTTP GET /article端点是否按预期工作。

由于我们要将响应转换为对象，因此我们首先调用mockMvc上的andReturn()方法来检索结果：

```java
MvcResult result = this.mockMvc.perform(get("/article"))
    .andExpect(status().isOk())
    .andReturn();
```

**andReturn()方法返回MvcResult对象，这使我们能够执行该工具不支持的额外验证**。

此外，我们可以调用getContentAsString()方法以String形式检索响应。不幸的是，MockMvc没有定义可用于将响应转换为特定对象类型的方法，我们需要自己指定逻辑。

**我们将使用Jackson的[ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)将JSON内容转换为所需的类型**。

让我们调用readValue()方法并传递字符串格式的响应以及我们想要将响应转换为的类型：

```java
String json = result.getResponse().getContentAsString();
Article article = objectMapper.readValue(json, Article.class);

assertNotNull(article);
assertEquals(1L, article.getId());
assertEquals("Learn Spring Boot", article.getTitle());
```

### 4.2 获取对象集合

让我们看看当端点返回集合时如何获取响应。

在上一节中，当我们想要获取单个对象时，我们将类型指定为Article.class。但是，对于集合等泛型类型，这是不可能的。我们不能将类型指定为List<Article\>.class。

**我们可以[使用Jackson反序列化集合](https://www.baeldung.com/jackson-collection-array)的一种方法是使用[TypeReference](https://fasterxml.github.io/jackson-core/javadoc/2.2.0/com/fasterxml/jackson/core/type/TypeReference.html)泛型类**：

```java
@Test
void whenGetArticle_thenReturnListUsingJacksonTypeReference() throws Exception {
    MvcResult result = this.mockMvc.perform(get("/articles"))
            .andExpect(status().isOk())
            .andReturn();

    String json = result.getResponse().getContentAsString();
    List<Article> articles = objectMapper.readValue(json, new TypeReference<>(){});

    assertNotNull(articles);
    assertEquals(2, articles.size());
}
```

由于[类型擦除](https://www.baeldung.com/java-type-erasure)，运行时无法获得泛型类型信息。为了克服这一限制，TypeReference在编译时捕获了我们想要将JSON转换为的类型。

此外，**我们可以通过指定[CollectionType](https://fasterxml.github.io/jackson-databind/javadoc/2.4/com/fasterxml/jackson/databind/type/CollectionType.html)来实现相同的功能**：

```java
String json = result.getResponse().getContentAsString();
CollectionType collectionType = objectMapper.getTypeFactory().constructCollectionType(List.class, Article.class);
List<Article> articles = objectMapper.readValue(json, collectionType);

assertNotNull(articles);
assertEquals(2, articles.size());
```

## 5. 使用Gson

现在，让我们看看如何使用[Gson](https://www.baeldung.com/gson-deserialization-guide)库将JSON内容转换为对象。

首先，让我们在pom.xml中添加所需的[依赖](https://mvnrepository.com/artifact/com.google.code.gson/gson)：

```xml
<dependency>
    <groupId>com.google.code.gson</groupId>
    <artifactId>gson</artifactId>
    <version>2.10.1</version>
</dependency>
```

### 5.1 获取单个对象

我们可以通过调用Gson实例上的fromJson()方法将JSON转换为对象，传递内容和所需的类型：

```java
@Test
void whenGetArticle_thenReturnArticleObjectUsingGson() throws Exception {
    MvcResult result = this.mockMvc.perform(get("/article"))
            .andExpect(status().isOk())
            .andReturn();

    String json = result.getResponse().getContentAsString();
    Article article = new Gson().fromJson(json, Article.class);

    assertNotNull(article);
    assertEquals(1L, article.getId());
    assertEquals("Learn Spring Boot", article.getTitle());
}
```

### 5.2 获取对象集合

最后，让我们看看如何使用Gson处理集合。

**要使用Gson反序列化集合，我们可以指定[TypeToken](https://www.javadoc.io/doc/com.google.code.gson/gson/2.6.2/com/google/gson/reflect/TypeToken.html)**：

```java
@Test
void whenGetArticle_thenReturnArticleListUsingGson() throws Exception {
    MvcResult result = this.mockMvc.perform(get("/articles"))
            .andExpect(status().isOk())
            .andReturn();

    String json = result.getResponse().getContentAsString();
    TypeToken<List<Article>> typeToken = new TypeToken<>(){};
    List<Article> articles = new Gson().fromJson(json, typeToken.getType());

    assertNotNull(articles);
    assertEquals(2, articles.size());
}
```

在这里，我们为Article元素列表定义了TypeToken。然后，在fromJson()方法中，我们调用getType()来返回Type对象。Gson使用[反射](https://www.baeldung.com/java-reflection)来确定我们要将JSON转换为哪种类型的对象。

## 6. 总结

在本文中，我们学习了使用MockMVC工具时将JSON内容作为对象检索的几种方法。

总而言之，我们可以使用Jackson的ObjectMapper将String响应转换为所需的类型。处理集合时，我们需要指定TypeReference或CollectionType。同样，我们可以使用Gson库反序列化对象。