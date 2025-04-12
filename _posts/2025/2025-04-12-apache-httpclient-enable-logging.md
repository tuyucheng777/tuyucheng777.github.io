---
layout: post
title:  启用Apache HttpClient的日志记录
category: apache
copyright: apache
excerpt: Apache HttpClient
---

## 1. 概述

在本教程中，我们将展示如何**在[Apache HttpClient](https://www.baeldung.com/httpclient-guide)中启用日志记录功能**。此外，我们还将解释该库内部是如何实现日志记录的。之后，我们将展示如何启用不同级别的日志记录。

## 2. 日志记录实现

HttpClient库提供了高效、最新、功能丰富的HTTP协议客户端站点实现。

**事实上，作为一个库，HttpClient并不强制要求日志记录实现**。出于这个目的，4.5版本使用[Commons Logging](http://commons.apache.org/proper/commons-logging/)提供日志记录功能。同样，最新版本5.1使用了[SLF4J](https://baeldung.com/slf4j-with-log4j2-logback)提供的日志记录门面；这两个版本都使用层次结构模式来将记录器与其配置进行匹配。

因此，可以为单个类或与相同功能相关的所有类设置记录器。

## 3. 日志类型

我们来看看库定义的日志级别。我们可以区分3种类型的日志：

- 上下文日志：记录有关HttpClient所有内部操作的信息。它还包含线路日志和头日志。
- 线路日志记录：仅记录传输到服务器和从服务器传输的数据
- 标头日志记录：仅记录HTTP标头

在4.5版本中对应的包是**org.apache.http.impl.client和org.apache.http.wire、org.apache.http.headers**。

因此，在5.1版本中，有**org.apache.hc.client5.http、org.apache.hc.client5.http.wire和org.apache.hc.client5.http.headers**包。

## 4. Log4j配置

让我们看看如何在两个版本中启用日志功能，我们的目标是在两个版本中实现相同的灵活性；**在4.1版本中，我们将日志重定向到SLF4j，这样，就可以使用不同的日志框架了**。

### 4.1 版本4.5配置

让我们添加[httpclient](https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient)依赖：

```xml
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.8</version>
    <exclusions>
        <exclusion>
            <artifactId>commons-logging</artifactId>
            <groupId>commons-logging</groupId>
        </exclusion>
    </exclusions>
</dependency>
```

我们将使用[jcl-over-slf4j](https://mvnrepository.com/artifact/org.slf4j/jcl-over-slf4j)将日志重定向到SLF4J，因此我们排除了commons-logging。接下来，让我们在JCL和SLF4J之间的桥接中添加一个依赖：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
    <version>1.7.32</version>
</dependency>
```

因为SLF4J只是一个门面，所以我们需要一个绑定。在我们的示例中，我们将使用[logback](https://mvnrepository.com/artifact/ch.qos.logback/logback-classic)：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

现在让我们创建ApacheHttpClientUnitTest类：

```java
public class ApacheHttpClientUnitTest {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    public static final String DUMMY_URL = "https://postman-echo.com/get";

    @Test
    public void whenUseApacheHttpClient_thenCorrect() throws IOException {
        HttpGet request = new HttpGet(DUMMY_URL);

        try (CloseableHttpClient client = HttpClients.createDefault();
             CloseableHttpResponse response = client.execute(request)) {
            HttpEntity entity = response.getEntity();
            logger.debug("Response -> {}",  EntityUtils.toString(entity));
        }
    }
}
```

测试获取虚拟网页并将内容打印到日志中。

现在让我们用logback.xml文件定义一个记录器配置：

```xml
<configuration debug="false">
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%date [%level] %logger - %msg %n</pattern>
        </encoder>
    </appender>

    <logger name="cn.tuyucheng.taketoday.httpclient.readresponsebodystring" level="debug"/>
    <logger name="org.apache.http" level="debug"/>

    <root level="WARN">
        <appender-ref ref="stdout"/>
    </root>
</configuration>
```

运行我们的测试后，可以在控制台中找到所有HttpClient的日志：

```text
...
2021-06-19 22:24:45,378 [DEBUG] org.apache.http.impl.execchain.MainClientExec - Executing request GET /get HTTP/1.1 
2021-06-19 22:24:45,378 [DEBUG] org.apache.http.impl.execchain.MainClientExec - Target auth state: UNCHALLENGED 
2021-06-19 22:24:45,379 [DEBUG] org.apache.http.impl.execchain.MainClientExec - Proxy auth state: UNCHALLENGED 
2021-06-19 22:24:45,382 [DEBUG] org.apache.http.headers - http-outgoing-0 >> GET /get HTTP/1.1 
...
```

### 4.2 版本5.1配置

现在我们来看看更高版本，**它包含重新设计的日志记录，因此，它使用SLF4J而不是Commons Logging**。因此，记录器门面的绑定是唯一的附加依赖，因此，我们将像第一个示例中一样使用logback-classic。

让我们添加[httpclient5](https://mvnrepository.com/artifact/org.apache.httpcomponents.client5/httpclient5)依赖：

```xml
<dependency>
    <groupId>org.apache.httpcomponents.client5</groupId>
    <artifactId>httpclient5</artifactId>
    <version>5.1</version>
</dependency>
```

让我们添加一个与前面的例子类似的测试：

```java
public class ApacheHttpClient5UnitTest {
    private final Logger logger = LoggerFactory.getLogger(this.getClass());
    public static final String DUMMY_URL = "https://postman-echo.com/get";

    @Test
    public void whenUseApacheHttpClient_thenCorrect() throws IOException, ParseException {
        HttpGet request = new HttpGet(DUMMY_URL);

        try (CloseableHttpClient client = HttpClients.createDefault();
             CloseableHttpResponse response = client.execute(request)) {
            HttpEntity entity = response.getEntity();
            logger.debug("Response -> {}", EntityUtils.toString(entity));
        }
    }
}
```

接下来，我们需要在logback.xml文件中添加一个记录器：

```xml
<configuration debug="false">
...
    <logger name="org.apache.hc.client5.http" level="debug"/>
...
</configuration>
```

我们来运行测试类ApacheHttpClient5UnitTest并检查输出，它与旧版本类似：

```text
...
2021-06-19 22:27:16,944 [DEBUG] org.apache.hc.client5.http.impl.classic.InternalHttpClient - ep-0000000000 endpoint connected 
2021-06-19 22:27:16,944 [DEBUG] org.apache.hc.client5.http.impl.classic.MainClientExec - ex-0000000001 executing GET /get HTTP/1.1 
2021-06-19 22:27:16,944 [DEBUG] org.apache.hc.client5.http.impl.classic.InternalHttpClient - ep-0000000000 start execution ex-0000000001 
2021-06-19 22:27:16,944 [DEBUG] org.apache.hc.client5.http.impl.io.PoolingHttpClientConnectionManager - ep-0000000000 executing exchange ex-0000000001 over http-outgoing-0 
2021-06-19 22:27:16,960 [DEBUG] org.apache.hc.client5.http.headers - http-outgoing-0 >> GET /get HTTP/1.1 
...
```

## 5. 总结

首先，我们解释了该库是如何实现日志记录的，其次，我们配置了两个版本的日志记录，并执行了一些简单的测试用例来展示输出。