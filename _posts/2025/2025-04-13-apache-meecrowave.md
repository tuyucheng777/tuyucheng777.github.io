---
layout: post
title:  使用Apache Meecrowave构建微服务
category: apache
copyright: apache
excerpt: Apache Meecrowave
---

## 1. 概述

在本教程中，我们将探索[Apache Meecrowave](http://openwebbeans.apache.org/meecrowave/)框架的基本功能。

**Meecrowave是Apache的一个轻量级微服务框架**，能够与CDI、JAX-RS和JSON API完美兼容。它的设置和部署非常简单，也省去了部署Tomcat、Glassfish、Wildfly等大型应用服务器的麻烦。

## 2. Maven依赖

要使用Meecrowave，让我们在pom.xml中定义依赖：

```xml
<dependency>
    <groupId>org.apache.meecrowave</groupId>
    <artifactId>meecrowave-core</artifactId>
    <version>1.2.15</version>
    <classifier>jakarta</classifier>
    <exclusions>
        <exclusion>
            <groupId>*</groupId>
            <artifactId>*</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

在[Maven Central](https://mvnrepository.com/artifact/org.apache.meecrowave/meecrowave-core)上检查最新版本。

## 3. 启动简单服务器

为了启动Meecrowave服务器，我们需要做的就是编写main方法，**创建Meecrowave实例并调用主bake()方法**：

```java
public static void main(String[] args) {
    try (Meecrowave meecrowave = new Meecrowave()) {
        meecrowave.bake().await();
    }
}
```

如果我们将应用程序打包为分发包，则不需要此main方法；我们将在后面的部分中讨论这一点。从IDE测试应用程序时，main类很有用。

其优点是，在IDE中进行开发时，一旦我们使用主类运行应用程序，它就会自动重新加载代码更改，从而省去了一次又一次重新启动服务器进行测试的麻烦。

请注意，如果我们使用的是Java 9，请不要忘记将javax.xml.bind模块添加到VM：

```text
--add-module javax.xml.bind
```

以这种方式创建服务器将以默认配置启动，我们可以使用Meecrowave.Builder类以编程方式更新默认配置：

```java
Meecrowave.Builder builder = new Meecrowave.Builder();
builder.setHttpPort(8080);
builder.setScanningPackageIncludes("cn.tuyucheng.taketoday.meecrowave");
builder.setJaxrsMapping("/api/*");
builder.setJsonpPrettify(true);
```

并在构建服务器时使用此构建器实例：

```java
try (Meecrowave meecrowave = new Meecrowave(builder)) { 
    meecrowave.bake().await();
}
```

[这里](http://openwebbeans.apache.org/meecrowave/meecrowave-core/configuration.html)有更多可配置的属性。

## 4. REST端点

现在，一旦服务器准备就绪，让我们创建一些REST端点：

```java
@RequestScoped
@Path("article")
public class ArticleEndpoints {

    @GET
    public Response getArticle() {
        return Response.ok().entity(new Article("name", "author")).build();
    }

    @POST
    public Response createArticle(Article article) {
        return Response.status(Status.CREATED).entity(article).build();
    }
}
```

请注意，我们主要**使用JAX-RS注解来创建REST端点**，[此处](https://www.baeldung.com/jax-rs-spec-and-implementations)介绍了关于JAX-RS的更多信息。

在下一节中，我们将了解如何测试这些端点。

## 5. 测试

使用Meecrowave编写的REST API的单元测试用例就像编写带注解的JUnit测试用例一样简单。

让我们首先将测试依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.meecrowave</groupId>
    <artifactId>meecrowave-junit</artifactId>
    <version>1.2.15</version>
    <scope>test</scope>
</dependency>
```

要查看最新版本，请访问[Maven Central](https://mvnrepository.com/artifact/org.apache.meecrowave/meecrowave-junit)。

另外，让我们添加[OkHttp](https://www.baeldung.com/guide-to-okhttp)作为测试的HTTP客户端：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.12.0</version>
</dependency>
```

在[此处](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)查看最新版本。

现在依赖添加完毕，我们可以继续编写测试了：

```java
@RunWith(MonoMeecrowave.Runner.class)
public class ArticleEndpointsIntegrationTest {

    @ConfigurationInject
    private Meecrowave.Builder config;
    private static OkHttpClient client;

    @BeforeClass
    public static void setup() {
        client = new OkHttpClient();
    }

    @Test
    public void whenRetunedArticle_thenCorrect() {
        String base = "http://localhost:" + config.getHttpPort();

        Request request = new Request.Builder()
                .url(base + "/article")
                .build();
        Response response = client.newCall(request).execute();
        assertEquals(200, response.code());
    }
}
```

在编写测试用例时，使用MonoMeecrowave.Runner类标注测试类，并注入配置，以访问Meecrowave用于测试服务器的随机端口。

## 6. 依赖注入

要将依赖注入到类中，我们需要在特定范围内标注这些类。

我们以ArticleService类为例：

```java
@ApplicationScoped
public class ArticleService {
    public Article createArticle(Article article) {
        return article;
    }
}
```

现在让我们使用javax.inject.Inject注解将其注入到我们的ArticleEndpoints实例中：

```java
@Inject
ArticleService articleService;
```

## 7. 打包应用程序

使用Meecrowave Maven插件创建分发包非常简单：

```xml
<build>
    ...
    <plugins>
        <plugin>
            <groupId>org.apache.meecrowave</groupId>
            <artifactId>meecrowave-maven-plugin</artifactId>
            <version>1.2.1</version>
        </plugin>
    </plugins>
</build>
```

一旦添加了插件，就使用Maven目标meecrowave:bundle来打包应用程序。

打包后，它将在target目录中创建一个zip文件：

```shell
meecrowave-meecrowave-distribution.zip
```

此zip文件包含部署应用程序所需的构件：

```text
|____meecrowave-distribution
| |____bin
| | |____meecrowave.sh
| |____logs
| | |____you_can_safely_delete.txt
| |____lib
| |____conf
| | |____log4j2.xml
| | |____meecrowave.properties
```

让我们导航到bin目录并启动应用程序：

```shell
./meecrowave.sh start
```

停止应用程序：

```shell
./meecrowave.sh stop
```

## 8. 总结

在本文中，我们学习了如何使用Apache Meecrowave创建微服务。此外，我们还研究了应用程序的一些基本配置以及如何准备分发包。