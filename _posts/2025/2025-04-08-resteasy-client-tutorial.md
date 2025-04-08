---
layout: post
title:  RESTEasy客户端API
category: webmodules
copyright: webmodules
excerpt: RESTEasy
---

## 1. 简介

在[上一篇文章](https://www.baeldung.com/resteasy-tutorial)中，我们重点介绍了RESTEasy服务器端的JAX-RS 2.0实现。

JAX-RS 2.0引入了新的客户端API，以便你可以向远程RESTful Web服务发出HTTP请求。Jersey、Apache CXF、Restlet和RESTEasy只是最流行的实现的一部分。

在本文中，我们将探讨如何通过使用RESTEasy API发送请求来使用REST API。

## 2. 项目设置

在你的pom.xml中添加以下依赖：

```xml
<properties>
    <resteasy.version>6.2.9.Final</resteasy.version>
</properties>

<dependencies>
    <dependency>
        <groupId>org.jboss.resteasy</groupId>
        <artifactId>resteasy-client</artifactId>
        <version>${resteasy.version}</version>
    </dependency>

    <dependency>
        <groupId>jakarta.servlet</groupId>
        <artifactId>jakarta.servlet-api</artifactId>
        <version>6.1.0</version>
    </dependency>
    ...
</dependencies>
```

## 3. 客户端代码

客户端实现非常小，由3个主要类组成：

- **Client**
- **WebTarget**
- **Response**

Client接口是WebTarget实例的构建器。

WebTarget代表一个独特的URL或URL模板，你可以从中构建更多子资源WebTarget或调用请求。

实际上，创建客户端有两种方法：

- 标准方法是使用org.jboss.resteasy.client.ClientRequest
- **RESTEasy代理框架**：通过使用ResteasyClientBuilder类

这里我们将重点介绍RESTEasy代理框架。

客户端框架不会使用JAX-RS注解将传入请求映射到RESTFul Web服务方法，而是构建一个HTTP请求，用于调用远程RESTful Web服务。

因此让我们开始编写Java接口并在方法和接口上使用JAX-RS注解。

### 3.1 ServicesClient接口

```java
@Path("/movies")
public interface ServicesInterface {

    @GET
    @Path("/getinfo")
    @Produces({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    Movie movieByImdbId(@QueryParam("imdbId") String imdbId);
    
    @POST
    @Path("/addmovie")
    @Consumes({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    Response addMovie(Movie movie);
    
    @PUT
    @Path("/updatemovie")
    @Consumes({ MediaType.APPLICATION_JSON, MediaType.APPLICATION_XML })
    Response updateMovie(Movie movie);
    
    @DELETE
    @Path("/deletemovie")
    Response deleteMovie(@QueryParam("imdbId") String imdbId);
}
```

### 3.2 Movie类

```java
@XmlAccessorType(XmlAccessType.FIELD)
@XmlType(name = "movie", propOrder = { "imdbId", "title" })
public class Movie {

    protected String imdbId;
    protected String title;

    // getters and setters
}
```

### 3.3 请求创建

我们现在将生成一个可以用来使用API的代理客户端：

```java
String transformerImdbId = "tt0418279";
Movie transformerMovie = new Movie("tt0418279", "Transformer 2");
UriBuilder FULL_PATH = UriBuilder.fromPath("http://127.0.0.1:8082/resteasy/rest");
 
ResteasyClient client = (ResteasyClient)ClientBuilder.newClient();
ResteasyWebTarget target = client.target(FULL_PATH);
ServicesInterface proxy = target.proxy(ServicesInterface.class);

// POST
Response moviesResponse = proxy.addMovie(transformerMovie);
System.out.println("HTTP code: " + moviesResponse.getStatus());
moviesResponse.close();

// GET
Movie movies = proxy.movieByImdbId(transformerImdbId);

// PUT
transformerMovie.setTitle("Transformer 4");
moviesResponse = proxy.updateMovie(transformerMovie);
moviesResponse.close();

// DELETE
moviesResponse = proxy.deleteMovie(batmanMovie.getImdbId());
moviesResponse.close();
```

请注意，RESTEasy客户端API基于Apache HttpClient。

还要注意，每次操作后，我们需要关闭响应才能执行新操作。这是必要的，因为默认情况下，客户端只有一个可用的HTTP连接。

最后，请注意我们如何直接使用DTO-我们不处理JSON或XML的编组/解组逻辑；由于Movie类已被正确标注，因此这些操作在幕后使用JAXB或Jackson进行。

### 3.4 使用连接池创建请求

上例中需要注意的一点是，我们只有一个可用连接。例如，如果我们尝试执行以下操作：

```java
Response batmanResponse = proxy.addMovie(batmanMovie);
Response transformerResponse = proxy.addMovie(transformerMovie);
```

没有在batmanResponse上调用close()-执行第2行时将引发异常：

```text
java.lang.IllegalStateException:
Invalid use of BasicClientConnManager: connection still allocated.
Make sure to release the connection before allocating another one.
```

再次强调，这种情况发生的原因很简单，因为RESTEasy使用的默认HttpClient是org.apache.http.impl.conn.SingleClientConnManager，这当然只能提供单个连接。

现在为了解决这个限制，必须以不同的方式创建RestEasyClient实例(使用连接池)：

```java
PoolingHttpClientConnectionManager cm = new PoolingHttpClientConnectionManager();
CloseableHttpClient httpClient = HttpClients.custom().setConnectionManager(cm).build();
cm.setMaxTotal(200); // Increase max total connection to 200
cm.setDefaultMaxPerRoute(20); // Increase default max connection per route to 20
ApacheHttpClient43Engine engine = new ApacheHttpClient43Engine(httpClient);

ResteasyClient client = ((ResteasyClientBuilder) ClientBuilder.newBuilder()).httpEngine(engine).build();
ResteasyWebTarget target = client.target(FULL_PATH);
ServicesInterface proxy = target.proxy(ServicesInterface.class);
```

现在，我们可以从适当的连接池中受益，并且可以通过我们的客户端运行多个请求，而不必每次都释放连接。

## 4. 总结

在此快速教程中，我们介绍了RESTEasy代理框架，并用它构建了一个超级简单的客户端API。

该框架为我们提供了一些辅助方法来配置客户端，并且可以定义为JAX-RS服务器端规范的镜像。