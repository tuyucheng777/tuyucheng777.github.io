---
layout: post
title:  Google Http Client指南
category: libraries
copyright: libraries
excerpt: Google Http Client
---

## 1. 概述

在本文中，我们将了解[Java版GoogleHTTP客户端库](https://developers.google.com/api-client-library/java/google-http-java-client/)，这是一个快速、抽象良好的库，可用于通过HTTP连接协议访问任何资源。

该客户端的主要特点是：

- 一个HTTP抽象层，可让你解耦任何低级库
- 快速、高效、灵活的HTTP响应和请求内容的JSON和XML解析模型
- 易于使用HTTP资源映射的注解和抽象

该库还可以在Java 5及更高版本中使用，这使其成为传统(SE和EE)项目的重要选择。

在本文中，我们将开发一个简单的应用程序，它将连接到GitHub API并检索用户，同时介绍该库的一些最有趣的功能。

## 2. Maven依赖

要使用该库，我们需要google-http-client依赖：

```xml
<dependency>
    <groupId>com.google.http-client</groupId>
    <artifactId>google-http-client</artifactId>
    <version>1.23.0</version>
</dependency>
```

最新版本可以在[Maven Central](https://mvnrepository.com/search?q=google-http-client)找到。

## 3. 发送简单请求

让我们首先向GitHub页面发出一个简单的GET请求，以展示Google HTTP Client的开箱即用方式：

```java
HttpRequestFactory requestFactory
    = new NetHttpTransport().createRequestFactory();
HttpRequest request = requestFactory.buildGetRequest(new GenericUrl("https://github.com"));
String rawResponse = request.execute().parseAsString()
```

为了发出最简单的请求，我们至少需要：

- HttpRequestFactory：用于构建我们的请求
- HttpTransport：是低级HTTP传输层的抽象
- GenericUrl：包装URL的类
- HttpRequest：处理请求的实际执行

在以下章节中，我们将介绍所有这些内容，以及一个返回JSON格式的实际API的更复杂示例。

## 4. 可插拔HTTP传输

该库有一个良好抽象的HttpTransport类，允许我们在此基础上构建并**更改为所选的底层低级HTTP传输库**：

```java
public class GitHubExample {
    static HttpTransport HTTP_TRANSPORT = new NetHttpTransport();
}
```

在本例中，我们使用NetHttpTransport，它基于所有Java SDK中都有的HttpURLConnection。这是一个不错的入门选择，因为它广为人知且可靠。

当然，在某些情况下我们可能需要一些高级定制，因此需要更复杂的低级库。

对于这种情况，有ApacheHttpTransport：

```java
public class GitHubExample {
    static HttpTransport HTTP_TRANSPORT = new ApacheHttpTransport();
}
```

ApacheHttpTransport基于流行的[Apache HttpClient](https://hc.apache.org/httpcomponents-client-5.3.x/index.html)，它包含多种配置连接的选择。

此外，该库还提供了构建低级实现的选项，使其非常灵活。

## 5. JSON解析

Google HTTP Client包含另一个JSON解析抽象，这样做的一个主要优点是低级解析库的选择是可互换的。

有3个内置选择，它们都扩展了JsonFactory，并且还包括我们自己实现的可能性。

### 5.1 可互换解析库

在我们的示例中，我们将使用Jackson2实现，它需要[google-http-client-jackson2](https://mvnrepository.com/search?q=google-http-client-jackson2)依赖：

```xml
<dependency>
    <groupId>com.google.http-client</groupId>
    <artifactId>google-http-client-jackson2</artifactId>
    <version>1.23.0</version>
</dependency>
```

接下来，我们现在可以包含JsonFactory：

```java
public class GitHubExample {
    static HttpTransport HTTP_TRANSPORT = new NetHttpTransport();
    staticJsonFactory JSON_FACTORY = new JacksonFactory();
}
```

**JacksonFactory是用于解析/序列化操作的最快和最受欢迎的库**。

这是以牺牲库大小为代价的(在某些情况下这可能是一个问题)。出于这个原因，Google还提供了GsonFactory，它是Google GSON库(一个轻量级JSON解析库)的一个实现。

我们也可以编写自己的低级解析器实现。

### 5.2 @Key注解

我们可以使用@Key注解来指示需要从JSON解析或序列化为JSON的字段：

```java
public class User {

    @Key
    private String login;
    @Key
    private long id;
    @Key("email")
    private String email;

    // standard getters and setters
}
```

在这里，我们创建一个User抽象，我们从GitHub API批量接收它(我们将在本文后面进行实际的解析)。

请注意，**没有@Key注解的字段被视为内部字段，不会从JSON解析或序列化为JSON**。此外，字段的可见性并不重要，Getter或Setter方法的存在性也不重要。

我们可以指定@Key注解的值，将其映射到正确的JSON键。

### 5.3 GenericJson

仅解析我们声明并标记为@Key的字段。

为了保留其他内容，我们可以声明我们的类来扩展GenericJson：

```java
public class User extends GenericJson {
    //...
}
```

**GenericJson实现了Map接口，这意味着我们可以使用get和put方法来设置/获取请求/响应中的JSON内容**。

## 6. 发送调用

要使用Google HTTP Client连接到端点，我们需要一个HttpRequestFactory，它将使用我们之前的抽象HttpTransport和JsonFactory进行配置：

```java
public class GitHubExample {

    static HttpTransport HTTP_TRANSPORT = new NetHttpTransport();
    static JsonFactory JSON_FACTORY = new JacksonFactory();

    private static void run() throws Exception {
        HttpRequestFactory requestFactory
                = HTTP_TRANSPORT.createRequestFactory(
                (HttpRequest request) -> {
                    request.setParser(new JsonObjectParser(JSON_FACTORY));
                });
    }
}
```

接下来我们需要一个要连接的URL，该库将其作为扩展GenericUrl的类来处理，其中声明的任何字段都被视为查询参数：

```java
public class GitHubUrl extends GenericUrl {

    public GitHubUrl(String encodedUrl) {
        super(encodedUrl);
    }

    @Key
    public int per_page;
}
```

在我们的GitHubUrl中，我们声明了per_page属性来指示我们希望在一次调用GitHub API时有多少个用户。

让我们继续使用GitHubUrl构建我们的调用：

```java
private static void run() throws Exception {
    HttpRequestFactory requestFactory
            = HTTP_TRANSPORT.createRequestFactory(
            (HttpRequest request) -> {
                request.setParser(new JsonObjectParser(JSON_FACTORY));
            });
    GitHubUrl url = new GitHubUrl("https://api.github.com/users");
    url.per_page = 10;
    HttpRequest request = requestFactory.buildGetRequest(url);
    Type type = new TypeToken<List<User>>() {}.getType();
    List<User> users = (List<User>)request
            .execute()
            .parseAs(type);
}
```

请注意我们如何指定API调用需要多少个用户，然后我们使用HttpRequestFactory构建请求。

接下来，由于GitHub API的响应包含用户列表，我们需要提供一个复杂的类型，即List<User\>。

然后，在最后一行，我们进行调用并将响应解析为我们的用户类列表。

## 7. 自定义标头

在发出API请求时，我们通常会做的一件事是包含某种自定义标头，甚至是修改后的标头：

```java
HttpHeaders headers = request.getHeaders();
headers.setUserAgent("Tuyucheng Client");
headers.set("Time-Zone", "Europe/Amsterdam");
```

我们通过在创建请求之后但在执行请求并添加必要的值之前获取HttpHeaders来实现这一点。

请注意，Google HTTP Client包含一些标头作为特殊方法。例如，如果我们尝试仅使用set方法包含User-Agent标头，则会引发错误。

## 8. 指数退避

Google HTTP Client的另一个重要功能是可以根据某些状态码和阈值重试请求。

我们可以在创建请求对象之后立即包含指数退避设置：

```java
ExponentialBackOff backoff = new ExponentialBackOff.Builder()
    .setInitialIntervalMillis(500)
    .setMaxElapsedTimeMillis(900000)
    .setMaxIntervalMillis(6000)
    .setMultiplier(1.5)
    .setRandomizationFactor(0.5)
    .build();
request.setUnsuccessfulResponseHandler(new HttpBackOffUnsuccessfulResponseHandler(backoff));
```

**HttpRequest中的指数退避默认是关闭的**，所以我们必须在HttpRequest中包含一个HttpUnsuccessfulResponseHandler实例来激活它。

## 9. 日志记录

Google HTTP Client使用java.util.logging.Logger记录HTTP请求和响应详细信息，包括URL、标头和内容。

通常，使用logging.properties文件来管理日志：

```properties
handlers = java.util.logging.ConsoleHandler
java.util.logging.ConsoleHandler.level = ALL
com.google.api.client.http.level = ALL
```

在我们的示例中，我们使用ConsoleHandler，但也可以选择FileHandler。

属性文件配置JDK日志记录工具的操作，此配置文件可以指定为系统属性：

```shell
-Djava.util.logging.config.file=logging.properties
```

因此，在设置文件和系统属性后，该库将产生如下日志：

```text
-------------- REQUEST  --------------
GET https://api.github.com/users?page=1&per_page=10
Accept-Encoding: gzip
User-Agent: Google-HTTP-Java-Client/1.23.0 (gzip)

Nov 12, 2017 6:43:15 PM com.google.api.client.http.HttpRequest execute
curl -v --compressed -H 'Accept-Encoding: gzip' -H 'User-Agent: Google-HTTP-Java-Client/1.23.0 (gzip)' -- 'https://api.github.com/users?page=1&per_page=10'
Nov 12, 2017 6:43:16 PM com.google.api.client.http.HttpResponse 
-------------- RESPONSE --------------
HTTP/1.1 200 OK
Status: 200 OK
Transfer-Encoding: chunked
Server: GitHub.com
Access-Control-Allow-Origin: *
...
Link: <https://api.github.com/users?page=1&per_page=10&since=19>; rel="next", <https://api.github.com/users{?since}>; rel="first"
X-GitHub-Request-Id: 8D6A:1B54F:3377D97:3E37B36:5A08DC93
Content-Type: application/json; charset=utf-8
...
```

## 10. 总结

在本教程中，我们展示了Java版Google HTTP客户端库及其更多有用功能。