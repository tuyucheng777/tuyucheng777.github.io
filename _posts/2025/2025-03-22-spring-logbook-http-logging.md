---
layout: post
title:  在Spring中使用Logbook记录HTTP请求和响应
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

[HTTP API](https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)请求现在是大多数应用程序的一部分。**[Logbook](https://github.com/zalando/logbook)是一个可扩展的Java库，可用于为不同的客户端和服务器端技术实现完整的请求和响应日志记录**。它允许开发人员记录应用程序接收或发送的任何HTTP流量，这可用于日志分析、审计或调查流量问题。

在本文中，让我们了解如何将Logbook库与[Spring Boot](https://www.baeldung.com/spring-boot)应用程序集成。

## 2. 依赖

为了在Spring Boot中使用Logbook库，我们需要在项目中添加以下依赖项：

```xml
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>logbook-spring-boot-starter</artifactId>
    <version>3.9.0</version>
</dependency>
```

可以在Maven Central找到最新版本的[Logbook](https://mvnrepository.com/artifact/org.zalando/logbook-spring-boot-starter)库。

## 3. 配置

**Logbook与Spring Boot应用程序中的[logback](https://www.baeldung.com/spring-boot-logback-log4j2)日志记录配合使用，我们需要在logback-spring.xml和[application.properties](https://github.com/zalando/logbook?tab=readme-ov-file#configuration)文件中添加配置**。

一旦我们将Logbook库添加到pom.xml中，Logbook库就会使用Spring Boot自动配置。让我们向application.properties文件添加一个日志级别：

```properties
logging.level.org.zalando.logbook.Logbook=TRACE
```

日志级别TRACE启用HTTP请求和响应的日志记录。

另外，我们在logback-spring.xml文件中添加Logbook配置：

```xml
<logger name="org.zalando.logbook" level="INFO" additivity="false">
    <appender-ref ref="RollingFile"/>
</logger>
```

添加完成后，我们可以使用HTTP请求运行应用程序。**每次HTTP请求调用后，Logbook库都会将请求和响应记录到logback-spring.xml配置中指定的日志文件路径**：

```text
11:08:14.737 [http-nio-8083-exec-10] TRACE org.zalando.logbook.Logbook - Incoming Request: eac2321df47c4414
Remote: 0:0:0:0:0:0:0:1
GET http://localhost:8083/api/hello?name=James HTTP/1.1
accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
accept-encoding: gzip, deflate, br, zstd
...
user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/125.0.0.0 Safari/537.36
11:08:14.741 [http-nio-8083-exec-10] TRACE org.zalando.logbook.Logbook - Outgoing Response: eac2321df47c4414
Duration: 4 ms
HTTP/1.1 200 OK
...
Date: Tue, 18 Jun 2024 05:38:14 GMT
Keep-Alive: timeout=60

Hello, James!

```

我们已经了解了如何使用最少的配置将Logbook库与Spring Boot集成，但是，这只是基本的请求和响应日志记录。让我们在下一小节中进一步了解配置。

## 4. 过滤和格式化

我们可以声明Logbook库配置[Bean](https://www.baeldung.com/spring-bean)：

```java
@Bean
public Logbook logbook() {
    Logbook logbook = Logbook.builder()
            .condition(Conditions.exclude(Conditions.requestTo("/api/welcome"),
                    Conditions.contentType("application/octet-stream"),
                    Conditions.header("X-Secret", "true")))
            .sink(new DefaultSink(new DefaultHttpLogFormatter(), new DefaultHttpLogWriter()))
            .build();
    return logbook;
}
```

在上面的代码示例中，我们声明了Logbook Bean，以便Spring Boot选择它来加载配置。

现在，我们在构建Logbook Bean时指定了条件。**exclude()方法中指定的[请求映射](https://www.baeldung.com/spring-requestmapping)被排除在日志记录之外，在这种情况下，Logbook库不会记录映射到路径”/api/welcome”的API的请求或响应。同样，我们使用contentType()方法对具有内容类型的请求设置过滤器，并使用header()方法对具有标头的请求设置过滤器**。

**同样，应该包含的HTTP API应该在include()方法中指定。如果我们不使用include()方法，Logbook会记录除exclude()方法中提到的请求之外的所有请求**。

我们应该在application.properties文件中设置过滤器属性，以使此过滤器配置正常工作。属性logbook.filter.enabled 应设置为true：

```properties
logbook.filter.enabled=true
```

Logbook库使用[SLF4J记录器](https://www.baeldung.com/slf4j-with-log4j2-logback)记录请求和响应，该记录器默认使用org.zalando.logbook.Logbook类别和日志级别TRACE：

```java
sink(new DefaultSink(
    new DefaultHttpLogFormatter(),
    new DefaultHttpLogWriter()
))
```

此默认配置允许在应用程序中进行日志记录：

```text
GET http://localhost:8083/api/hello?name=John HTTP/1.1
accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
....
HTTP/1.1 200 OK
Connection: keep-alive
Content-Length: 47
Content-Type: text/html;charset=UTF-8
Date: Fri, 07 Jun 2024 11:12:27 GMT
Keep-Alive: timeout=60
Hello, John
```

可以将这些响应记录到System.out或System.err中，并在控制台上打印：

```java
Logbook logbook = Logbook.builder()
    .sink(new DefaultSink(
        new DefaultHttpLogFormatter(),
        new StreamHttpLogWriter(System.out)
    ))
    .build();
```

我们应该避免在生产环境中使用控制台打印，但如果有必要，可以在开发环境中使用它。

## 5. Sinks

直接实现Sink接口可以实现更复杂的用例，例如将请求/响应写入数据库等结构化持久存储。

### 5.1 常见接收器

Logbook提供了一些常用日志格式的Sink的实现，即[CommonsLogFormatSink](https://en.wikipedia.org/wiki/Common_Log_Format)、[ExtendedLogFormatSink](https://en.wikipedia.org/wiki/Extended_Log_Format)。

ChunkingSink将长消息分割成较小的块并单独写入，同时将它们委托给另一个接收器：

```java
Logbook logbook = Logbook.builder()
    .sink(new ChunkingSink(sink, 1000))
    .build();
```

### 5.2 Logstash接收器

Logbook在附加库中提供了logstash编码器，让我们看一个LogstashLogbackSink的示例。为此，我们在pom.xml中添加[logstash](https://mvnrepository.com/artifact/org.zalando/logbook-logstash)和[logstash-encoder](https://mvnrepository.com/artifact/net.logstash.logback/logstash-logback-encoder)依赖项：

```xml
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
<dependency>
    <groupId>org.zalando</groupId>
    <artifactId>logbook-logstash</artifactId>
    <version>3.9.0</version>
 </dependency>
```

然后，我们在logback-spring.xml中更改appender下的编码器，日志存储编码器启用LogstashLogbackSink：

```xml
<appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
    ... 
    <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
</appender>
```

现在我们声明LogstashLogbackSink并将其添加到Logbook对象构建器：

```java
HttpLogFormatter formatter = new JsonHttpLogFormatter(); 
LogstashLogbackSink logstashsink = new LogstashLogbackSink(formatter);
Logbook logbook = Logbook.builder()
    .sink(logstashsink)
    .build();
```

这里我们将JsonHttpLogFormatter与LogstashLogbackSink结合使用，此自定义以[JSON格式](https://www.baeldung.com/java-org-json)打印日志：

```json
{
    "@timestamp": "2024-06-07T16:46:24.5673233+05:30",
    "@version": "1",
    "message": "200 OK GET http://localhost:8083/api/hello?name=john",
    "logger_name": "org.zalando.logbook.Logbook",
    "thread_name": "http-nio-8083-exec-6",
    "level": "TRACE",
    "http":  {
        ...
        "Content-Length": [
            "12"
        ],
        ...
        "body": "Hello, john!"
    }
}
```

JSON日志以单行打印；我们在这里对其进行了格式化以提高可读性。

**我们可以在声明LogstashLogbackSink对象时更改日志级别**：

```java
LogstashLogbackSink logstashsink = new LogstashLogbackSink(formatter, Level.INFO);
```

我们还可以将SplunkHttpLogFormatter与Logbook接收器一起使用，它以键值格式打印日志：

```text
origin=remote ... method=GET uri=http://localhost:8083/api/hello?name=John host=localhost path=/api/hello ...
```

### 5.3 复合Sink

结合多个接收器，我们可以形成一个CompositeSink：

```java
CompositeSink compsink = new CompositeSink(Arrays.asList(logstashsink, new CommonsLogFormatSink(new DefaultHttpLogWriter())));
Logbook logbook = Logbook.builder()
    .sink(compsink)
    .build();
```

此配置使用复合接收器中指定的所有接收器记录请求详细信息，此处，Logbook通过组合两个接收器来记录请求：

```text
... "message":"GET http://localhost:8083/api/hello?name=John",... uri":"http://localhost:8083/api/hello?name=John",...
... "message":"200 OK GET http://localhost:8083/api/hello?name=John",.."headers":{"Connection":["keep-alive"],...
```

## 6. 总结

在本文中，我们学习了如何使用最少的配置将Logbook库与Spring Boot集成。此外，我们还学习了如何使用exclude()和 include()方法过滤请求路径。我们还学习了如何使用LogstashLogbackSink等Sink实现和JsonHttpLogFormatter等格式化程序根据需要自定义日志格式。