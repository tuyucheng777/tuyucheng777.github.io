---
layout: post
title:  如何在Quarkus中使用REST API
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 简介

微服务架构将单体系统分解为更小、松散耦合的服务，彻底改变了我们设计和构建应用程序的方式。**这些服务主要通过REST API相互通信，因此了解如何有效地使用这些API至关重要**。

Quarkus是一个针对微服务优化的现代Java框架。

在本教程中，我们将探索如何在Quarkus中创建虚拟REST API，并演示使用不同客户端调用它的各种方法。这些知识对于构建强大而高效的基于微服务的应用程序至关重要。

## 2. 创建API

首先，我们需要设置一个基本的Quarkus应用程序并创建一个返回帖子列表的虚拟REST API。

### 2.1 创建Post实体

我们将创建一个我们的API将返回的Post实体：

```java
public class Post {
    public Long id;
    public String title;
    public String description;

    // getters, setters, constructors
}
```

### 2.2 创建PostResource

此外，对于此示例，我们将创建一个返回JSON格式的帖子列表的资源：

```java
@Path("/posts")
public class PostResource {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Post> getPosts() {
        return Arrays.asList(
                new Post(1L, "Post One", "This is the first post"),
                new Post(2L, "Post Two", "This is the second post")
        );
    }
}
```

我们将在新的应用程序中使用该API。

### 2.3 测试API

我们可以使用curl测试我们的新API：

```bash
curl -X GET http://localhost:8080/posts
```

通过调用此方法，我们将获得帖子的JSON列表：

```json
[
    {
        "id": 1,
        "title": "Post One",
        "description": "This is the first post"
    },
    {
        "id": 2,
        "title": "Post Two",
        "description": "This is the second post"
    }
]
```

现在，此API已可运行，我们将看到如何在另一个Quarkus应用程序中使用它，而不是curl。

## 3. 使用Rest Client调用API

**Quarkus支持MicroProfile Rest Client，这是一个强大且类型安全的HTTP客户端，它通过提供接口驱动的方法简化了RESTful API的使用**。

### 3.1 Maven依赖

首先，我们需要在pom.xml中包含[Rest Client依赖](https://mvnrepository.com/artifact/io.quarkus/quarkus-rest-client)：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-rest-client</artifactId>
    <version>3.13.3</version>
</dependency>
```

这将提供与MicroProfile Rest Client协同工作所需的组件。

### 3.2 定义客户端接口

我们将定义一个接口来表示我们想要使用的远程API，此接口应反映API端点的结构：

```java
@Path("/posts")
@RegisterRestClient(configKey = "post-api")
public interface PostRestClient {
    @GET
    @Produces(MediaType.APPLICATION_JSON)
    List<Post> getAllPosts();
}
```

@RegisterRestClient注解将此接口注册为REST客户端，configKey属性用于绑定配置属性，@Path注解指定API的基本路径。

### 3.3 配置

可以使用接口中定义的configKey在application.properties文件中指定REST客户端的基本URL：

```properties
quarkus.rest-client.post-api.url=http://localhost:8080
```

通过这样做，我们可以轻松修改API的基本URL，而无需更改源代码。除此之外，我们还可以更改应用程序的默认端口：

```properties
quarkus.http.port=9000
```

我们这样做是因为第一个API在默认端口8080上运行。

### 3.4 使用Rest客户端

定义并配置Rest Client接口后，我们可以使用@RestClient注解将其注入Quarkus服务或资源类。此注解告诉Quarkus提供使用基本URL和其他设置配置的指定接口的实例：

```java
@Path("rest-client/consume-posts")
public class PostClientResource {
    @Inject
    @RestClient
    PostRestClient postRestClient;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Post> getPosts() {
        return postRestClient.getAllPosts();
    }
}
```

### 3.5 测试应用程序

现在，一切设置完毕，可以测试我们的应用程序了。我们可以通过运行curl命令来做到这一点：

```shell
curl -X GET localhost:9000/rest-client/consume-posts
```

这应该返回我们的帖子的JSON列表。

## 4. 使用JAX-RS客户端API

JAX-RS客户端API是RESTful Web服务的Java API(JAX-RS)规范的一部分，它提供了一种标准、编程的方式来创建HTTP请求和使用RESTful Web服务。

### 4.1 Maven依赖

首先，我们需要在pom.xml中包含[RESTEasy客户端依赖](https://mvnrepository.com/artifact/org.jboss.resteasy/resteasy-client)：

```xml
<dependency>
    <groupId>org.jboss.resteasy</groupId>
    <artifactId>resteasy-client</artifactId>
    <version>6.2.10.Final</version>
</dependency>
```

此依赖项引入了RESTEasy、Quarkus使用的JAX-RS实现以及发出HTTP请求所需的客户端API。

### 4.2 实现JAX-RS客户端

为了使用REST API，我们将创建一个服务类，设置JAX-RS客户端、配置目标URL并处理响应：

```java
@ApplicationScoped
public class JaxRsPostService  {
    private final Client client;
    private final WebTarget target;

    public JaxRsPostService() {
        this.client = ClientBuilder.newClient();
        this.target = client.target("http://localhost:8080/posts");
    }

    public List<Post> getPosts() {
        return target
                .request()
                .get(new GenericType<List<Post>>() {});
    }
}
```

我们使用[构建器模式](https://www.baeldung.com/java-builder-pattern)初始化客户端，并使用API请求的基本URL配置目标。

### 4.3 通过资源类公开API

现在，我们需要做的就是将我们的服务注入到我们的资源中：

```java
@Path("jax-rs/consume-posts")
public class PostClientResource {
    @Inject
    JaxRsPostService jaxRsPostService;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Post> getJaxRsPosts() {
        return jaxRsPostService.getPosts();
    }
}
```

### 4.4 测试应用程序

现在我们可以使用curl再次测试我们的API：

```shell
curl -X GET localhost:9000/jax-rs/consume-posts
```

## 5. 使用Java 11 HttpClient的API

Java 11引入了一个新的HTTP Client API，它提供了一种现代、异步且功能丰富的方法来处理HTTP通信。[java.net.http.HttpClient](https://www.baeldung.com/java-9-http-client)类允许我们轻松发送HTTP请求和处理响应，在本节中，我们将学习如何做到这一点。

### 5.1 创建HttpClient服务

此示例中不需要任何额外的依赖，Java 11的HttpClient是标准库的一部分。

现在，我们将创建一个管理HttpClient的服务类：

```java
@ApplicationScoped
public class JavaHttpClientPostService {
    private final HttpClient httpClient;
    private final ObjectMapper objectMapper;

    public JavaHttpClientPostService() {
        this.httpClient = HttpClient.newHttpClient();
        this.objectMapper = new ObjectMapper();
    }

    public List<Post> getPosts() {
        HttpRequest request = HttpRequest.newBuilder()
                .uri(URI.create("http://localhost:8080/posts"))
                .GET()
                .build();

        try {
            HttpResponse<String> response = httpClient.send(request, HttpResponse.BodyHandlers.ofString());
            return objectMapper.readValue(response.body(), new TypeReference<ArrayList<Post>>() { });
        }
        catch (IOException | InterruptedException e) {
            throw new RuntimeException("Failed to fetch posts", e);
        }
    }
}
```

在这个类中，我们初始化HttpClient实例和来自[Jackson](https://www.baeldung.com/jackson)的[ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)实例，我们将使用它们将JSON响应解析为Java对象。

我们创建一个HttpRequest对象，指定API端点的URI和HTTP方法。之后，我们使用HttpClient实例的send()方法发送请求，并使用BodyHandlers.ofString()处理响应。它将主体转换为字符串，我们将使用ObjectMapper将该字符串转换为我们的Post对象。

### 5.2 创建资源

为了使我们的应用程序能够获取到数据，我们将通过资源类公开JavaHttpClientPostService ：

```java
@Path("/java-http-client/consume-posts")
public class JavaHttpClientPostResource {
    @Inject
    JavaHttpClientPostService postService;

    @GET
    @Produces(MediaType.APPLICATION_JSON)
    public List<Post> getPosts() {
        return postService.getPosts();
    }
}
```

### 5.3 测试应用程序

现在我们可以使用curl再次测试该应用程序：

```shell
curl -X GET localhost:9000/java-http-client/consume-posts
```

## 6. 总结

在本文中，我们演示了如何使用Quarkus RestClient、JAX-RS客户端API和Java 11 HttpClient在Quarkus中使用REST API。

每种方法都有优点：RestClient与Quarkus无缝集成，JAX-RS客户端API提供灵活性，而Java 11的HttpClient带来了JDK的现代功能。掌握这些技术可以实现微服务之间的有效通信，从而更轻松地在Quarkus中构建可扩展且高效的架构。