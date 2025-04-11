---
layout: post
title:  在Java中将cURL请求转换为HTTP请求
category: java-net
copyright: java-net
excerpt: cURL
---

## 1. 简介

在使用API时，我们通常首先使用[cURL](https://www.baeldung.com/linux/curl-guide)测试请求，cURL是一个命令行工具，可帮助我们轻松发送HTTP请求。在本教程中，我们将介绍几种使用Java将cURL转换为HTTP请求的方法。

## 2. 了解cURL命令

让我们从一个典型的cURL命令开始：

```makefile
curl -X POST "http://example.com/api" \
  -H "Content-Type: application/json" \
  -d '{"key1":"value1", "key2":"value2"}'
```

此命令向URL http://example.com/api发送一个POST请求，具有以下特征：

- -X POST：指定HTTP方法，使用POST向服务器发送数据
- -H "Content-Type:application/json"：添加标头，表明请求主体为JSON格式
- –d '{"key1":"value1", "key2":"value2"}'：将有效负载(请求正文)作为包含两个键值对的JSON字符串提供

**虽然cURL非常适合测试端点，但将这些命令转换为Java代码可以使我们的API调用可重复使用、可测试，并且能够更好地处理实际项目中的错误**；我们可以使用Java中提供的几个不同的库和工具来实现这一点。

## 3. Java内置的HttpURLConnection

在Java中发出HTTP请求的最简单方法之一是使用[HttpURLConnection](https://www.baeldung.com/httpurlconnection-post)类，该类内置于Java标准库中。**此方法非常简单，不需要任何额外的依赖**。

下面是如何使用HttpURLConnection将早期的cURL命令转换为Java HTTP请求的示例：

```java
String sendPostWithHttpURLConnection(String targetUrl) {
    StringBuilder response = new StringBuilder();
    try {
        URL url = new URL(targetUrl);
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("POST");
        conn.setRequestProperty("Content-Type", "application/json; utf-8");
        conn.setRequestProperty("Accept", "application/json");
        conn.setDoOutput(true);

        String jsonInputString = "{\"key1\":\"value1\", \"key2\":\"value2\"}";

        try (OutputStream os = conn.getOutputStream()) {
            byte[] input = jsonInputString.getBytes("utf-8");
            os.write(input, 0, input.length);
        }

        int code = conn.getResponseCode();
        logger.info("Response Code: " + code);

        try (BufferedReader br = new BufferedReader(
                new InputStreamReader(conn.getInputStream(), "utf-8"))) {
            String responseLine;
            while ((responseLine = br.readLine()) != null) {
                response.append(responseLine.trim());
            }
        }
    } catch (Exception e) {
        // handle exception
    }
    return response.toString();
}
```

我们首先使用URL类设置连接以创建URL对象，然后使用HttpURLConnection打开一个连接。**接下来，我们将请求方法设置为POST，并使用setRequestProperty()添加标头，以指定我们的请求正文是JSON，并且我们期望JSON响应**。 

一旦请求被发送，我们从服务器获取响应码并使用[BufferedReader](https://www.baeldung.com/java-buffered-reader)读取响应，将每一行附加到[StringBuilder](https://www.baeldung.com/java-string-builder-string-buffer)。

下面是一个简单的测试用例，当发送有效的POST请求时，它会断言非空响应：

```java
String targetUrl = mockWebServer.url("/api").toString();
String response = CurlToHttpRequest.sendPostWithHttpURLConnection(targetUrl);

assertNotNull(response);
assertFalse(response.isEmpty());
assertEquals("{\"message\": \"Success\"}", response);
```

## 4. Apache HttpClient

[Apache HttpClient](https://www.baeldung.com/apache-httpclient-cookbook)是一个强大且流行的HTTP请求库，**它比内置的HttpURLConnection提供更多的控制和灵活性**。

首先，我们将其[依赖](https://mvnrepository.com/artifact/org.apache.httpcomponents.client5/httpclient5)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.4.2</version>
</dependency>
```

以下是我们使用Apache HttpClient转换cURL请求的方法：

```java
String sendPostRequestUsingApacheHttpClient(String targetUrl) {
    String result = "";
    try (CloseableHttpClient client = HttpClients.createDefault()) {
        HttpPost httpPost = new HttpPost(targetUrl);
        httpPost.setHeader("Content-Type", "application/json");

        String jsonInput = "{\"key1\":\"value1\", \"key2\":\"value2\"}";

        StringEntity entity = new StringEntity(jsonInput);
        httpPost.setEntity(entity);

        try (CloseableHttpResponse response = client.execute(httpPost)) {
            result = EntityUtils.toString(response.getEntity());
            logger.info("Response Code: " + response.getStatusLine().getStatusCode());
        }
    } catch (Exception e) {
        // handle exception
    }
    return result;
}
```

在此示例中，我们创建一个CloseableHttpClient实例并使用它来对目标URL执行POST请求，**我们设置Content-Type标头以指示我们正在发送JSON数据**。JSON有效负载包装在StringEntity中并附加到HttpPost请求。执行请求后，我们将响应实体转换为字符串。

Apache HttpClient使我们能够自定义HTTP通信的许多方面，**我们可以轻松调整连接超时、管理连接池、添加用于日志记录或身份验证的拦截器以及处理重试或重定向**。

下面是一个示例，展示了如何自定义Apache HttpClient来设置连接超时并添加一个用于记录请求详细信息的拦截器：

```java
String sendPostRequestUsingApacheHttpClientWithCustomConfig(String targetUrl) {
    String result = "";

    // Create custom configuration for connection and response timeouts.
    RequestConfig config = RequestConfig.custom()
            .setConnectTimeout(Timeout.ofSeconds(10))
            .setResponseTimeout(Timeout.ofSeconds(15))
            .build();

    // Create a custom HttpClient with a logging interceptor.
    CloseableHttpClient client = HttpClientBuilder.create()
            .setDefaultRequestConfig(config)
            .addRequestInterceptorFirst(new HttpRequestInterceptor() {
                @Override
                public void process(HttpRequest request, EntityDetails entity, HttpContext context) {
                    logger.info("Request URI: " + request.getRequestUri());
                    logger.info("Request Headers: " + request.getHeaders());
                }
            })
            .build();

    try {
        HttpPost httpPost = new HttpPost(targetUrl);
        httpPost.setHeader("Content-Type", "application/json");

        String jsonInput = "{\"key1\":\"value1\", \"key2\":\"value2\"}";
        StringEntity entity = new StringEntity(jsonInput);
        httpPost.setEntity(entity);

        try (CloseableHttpResponse response = client.execute(httpPost)) {
            result = EntityUtils.toString(response.getEntity());
            logger.info("Response Code: " + response.getCode());
        }
    } catch (Exception e) {
        // Handle exception appropriately
    } finally {
        // close the client and handle exception
    }
    return result;
}
```

在这种定制方法中，我们首先创建一个RequestConfig，将连接超时设置为10秒，将响应超时设置为15秒。

此外，我们添加了一个请求拦截器，用于在执行请求之前记录请求URI和标头。**此拦截器提供了有关请求详细信息的宝贵见解，这对于调试或监控非常有用**。

## 5. OkHttp

[OkHttp](https://www.baeldung.com/guide-to-okhttp)是Square开发的一款现代HTTP客户端，以易用性和卓越性能而闻名。**它支持HTTP/2，广泛应用于Android和Java应用程序**。

让我们将[依赖](https://mvnrepository.com/artifact/com.squareup.okhttp3/okhttp)添加到pom.xml中：

```xml
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
    <version>4.12.0</version>
</dependency>
```

下面展示了如何使用OkHttp将cURL请求转换为HTTP请求：

```java
String sendPostRequestUsingOkHttp(String targetUrl) {
    OkHttpClient client = new OkHttpClient();

    MediaType JSON = MediaType.get("application/json; charset=utf-8");

    String jsonInput = "{\"key1\":\"value1\", \"key2\":\"value2\"}";
    RequestBody body = RequestBody.create(jsonInput, JSON);
    Request request = new Request.Builder()
            .url(targetUrl)
            .post(body)
            .build();

    String result = "";
    try (Response response = client.newCall(request).execute()) {
        logger.info("Response Code: " + response.code());
        result = response.body().string();
    } catch (Exception e) {
        // handle exception
    }
    return result;
}
```

在这段代码中，我们定义一个RequestBody来保存JSON有效负载并指定其内容类型。接下来，我们使用Request.Builder()设置目标URL、HTTP方法(POST)、请求正文和标头。

最后，我们使用client.newCall(request).execute()执行请求，处理可能发生的任何异常。

## 6. Spring WebClient

此外，在使用Spring框架时，[WebClient](https://www.baeldung.com/spring-5-webclient)是一个简化HTTP请求的高级客户端。**它是一种现代的非阻塞HTTP客户端，也是在Spring应用程序中发出HTTP请求的推荐方式**。

我们将cURL示例转换为使用WebClient：

```java
String sendPostRequestUsingWebClient(String targetUrl) {
    WebClient webClient = WebClient.builder()
            .baseUrl(targetUrl)
            .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
            .build();

    String jsonInput = "{\"key1\":\"value1\", \"key2\":\"value2\"}";

    return webClient.post()
            .bodyValue(jsonInput)
            .retrieve()
            .bodyToMono(String.class)
            .doOnNext(response -> logger.info("Response received from WebClient: " + response))
            .block();
}
```

在此示例中，我们创建一个具有目标URL的WebClient实例，并将Content-Type标头设置为application/json。**然后，我们使用bodyValue(jsonInput)附加JSON有效负载**，此外，我们添加block()以确保该方法同步执行并返回响应。

或者，如果我们在异步或响应式应用程序中工作，我们可以删除block()方法并返回Mono<String\>而不是String。

## 7. 总结

在本文中，我们探讨了如何使用各种库和工具将简单的cURL命令转换为Java代码，HttpURLConnection非常适合我们想要避免外部依赖的简单用例；Apache HttpClient非常适合需要对HTTP请求进行细粒度控制的应用程序。

此外，OkHttp是一款现代HTTP客户端，以其性能而闻名，非常适合微服务或Android开发。Spring WebClient最适合响应式应用程序，尤其是在Spring生态系统中。