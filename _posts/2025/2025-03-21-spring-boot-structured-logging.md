---
layout: post
title:  Spring Boot中的结构化日志
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

日志记录是任何软件应用程序的基本功能，它通过记录错误、警告和其他事件来帮助跟踪应用程序在运行时的行为。

默认情况下，Spring Boot应用程序会生成非结构化、人性化且易于阅读的日志。虽然这些日志对开发人员很有用，但它们不易被日志聚合工具解析或分析。[结构化日志记录](https://www.baeldung.com/java-structured-logging)解决了这一限制。

在本教程中，我们将学习如何利用Spring Boot版本3.4.0中引入的功能实现结构化日志记录。

## 2. Maven依赖

首先，让我们在pom.xml中添加[spring-boot-starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter)来启动一个Spring Boot项目：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
    <version>3.4.0</version>
</dependency>
```

上述依赖项为典型的Spring Boot应用程序中的自动配置和日志记录提供支持。

## 3. Spring Boot默认日志

以下是默认的Spring Boot日志：

```shell
INFO 22059 --- [ main] c.t.t.s.StructuredLoggingApp  : No active profile set, falling back to 1 default profile: "default"
INFO 22059 --- [ main] c.t.t.s.StructuredLoggingApp   : Started StructuredLoggingApp in 2.349 seconds (process running for 3.259)
```

虽然这些日志信息量很大，但它们无法被[Elasticsearch](https://www.baeldung.com/java-elasticsearch)等工具轻松提取或分析指标。JSON等结构化日志格式通过标准化日志内容解决了这个问题。

## 4. 配置

**从Spring Boot 3.4.0版本开始，结构化日志记录已内置并支持Elastic Common Schema(ECS)、[Graylog Extended Log Format(GELF)](https://www.baeldung.com/graylog-with-spring-boot)和LogstashJSON等格式**。

我们可以直接在application.properties文件中配置结构日志记录。

### 4.1 Elastic Common Schema

Elastic Common Schema(ECS)是一种基于JSON的标准化日志格式，可无缝集成[Elasticsearch和Kibana](https://www.baeldung.com/ops/elk)。要在我们的应用程序中配置ECS，让我们将其属性添加到我们的application.properties文件中：

```properties
logging.structured.format.console=ecs
```

以下是示例输出：

```json
{
    "@timestamp": "2024-12-19T01:17:47.195098997Z",
    "log.level": "INFO",
    "process.pid": 16623,
    "process.thread.name": "main",
    "log.logger": "cn.tuyucheng.taketoday.springstructuredlogging.StructuredLoggingApp",
    "message": "Started StructuredLoggingApp in 3.15 seconds (process running for 4.526)",
    "ecs.version": "8.11"
}
```

输出包含可以在Elasticsearch和Kibana中轻松解析的键值对。

此外，我们可以通过添加服务名称、环境和节点名称等字段来增强ECS日志，以提高[可观察性](https://www.baeldung.com/spring-boot-3-observability)：

```properties
logging.structured.ecs.service.name=MyService
logging.structured.ecs.service.version=1
logging.structured.ecs.service.environment=Production
logging.structured.ecs.service.node-name=Primary
```

这是新的输出：

```json
{
    "@timestamp": "2024-12-19T01:25:15.123108416Z",
    "log.level": "INFO",
    "process.pid": 18763,
    "process.thread.name": "main",
    "service.name": "TaketodayService",
    "service.version": "1",
    "service.environment": "Production",
    "service.node.name": "Primary",
    "log.logger": "cn.tuyucheng.taketoday.springstructuredlogging.StructuredLoggingApp",
    "message": "Started StructuredLoggingApp in 3.378 seconds (process running for 4.376)",
    "ecs.version": "8.11"
}
```

输出包含我们在application.properties文件中定义的服务信息。

### 4.2 Graylog扩展日志格式

[Graylog](https://www.baeldung.com/graylog-with-spring-boot) Extend Log Format(GELF)是另一种基于JSON的支持结构化日志格式。让我们在application.properties文件中启用它：

```properties
logging.structured.format.console=gelf
```

GELF格式与ECS格式类似，但属性名称不同：

```json
{
    "version": "1.1",
    "short_message": "Started StructuredLoggingApp in 2.77 seconds (process running for 3.89)",
    "timestamp": 1734572549.172,
    "level": 6,
    "_level_name": "INFO",
    "_process_pid": 23929,
    "_process_thread_name": "main",
    "_log_logger": "cn.tuyucheng.taketoday.springstructuredlogging.StructuredLoggingApp"
}
```

与ECS配置类似，我们可以通过在application.properties文件中定义主机和服务版本来进一步增强输出：

```properties
logging.structured.gelf.host=MyService
logging.structured.gelf.service.version=1
```

这通过添加主机和服务键值对来扩展日志。

### 4.3 Logstash格式

Logstash格式也是开箱即用的，为了将日志构造为该格式，我们在application.properties中指定它：

```properties
logging.structured.format.file=logstash
```

以下是结构化日志的示例：

```json
{
    "@timestamp": "2024-12-19T02:49:33.017851728+01:00",
    "@version": "1",
    "message": "Started StructuredLoggingApp in 2.749 seconds (process running for 3.605)",
    "logger_name": "cn.tuyucheng.taketoday.springstructuredlogging.StructuredLoggingApp",
    "thread_name": "main",
    "level": "INFO",
    "level_value": 20000
}
```

使用支持Logstash格式的日志聚合可以轻松分析上述格式。

### 4.4 附加信息

**我们可以使用[映射诊断上下文(MDC)](https://www.baeldung.com/mdc-in-log4j-2-logback)类向结构化日志添加更多信息**。例如，我们可以将userId添加到日志中，以便通过userId过滤日志：

```java
private static final Logger LOGGER = LoggerFactory.getLogger(CustomLog.class);
```

```java
public void additionalDetailsWithMdc() {
    MDC.put("userId", "1");
    MDC.put("userName", "Taketoday");
    LOGGER.info("Hello structured logging!");
    MDC.remove("userId");
    MDC.remove("userName");
}
```

在上面的代码中，我们随后删除每个条目以清理MDC上下文，防止内存泄漏。

让我们看看包含用户详细信息的日志输出：

```json
{
    "@timestamp": "2024-12-19T07:52:30.556819106+01:00",
    "@version": "1",
    "message": "Hello structured logging!",
    "logger_name": "cn.tuyucheng.taketoday.springstructuredlogging.CustomLog",
    "thread_name": "main",
    "level": "INFO",
    "level_value": 20000,
    "userId": "1",
    "userName": "Taketoday"
}
```

在这里，我们向日志消息添加了更多标记。我们可以轻松地根据userId过滤日志。我们可以使用MDC类向日志添加更多属性。

另外，**我们可以使用流式的日志记录API来实现类似的目的**：

```java
public void additionalDetailsUsingFluentApi() {
    LOGGER.atInfo()
        .setMessage("Hello Structure logging!")
        .addKeyValue("userId", "1")
        .addKeyValue("userName", "Taketoday")
        .log();
}
```

这种方法更简洁，并且可以自动处理上下文清理，从而不易出错。

### 4.5 自定义日志格式

此外，**我们可以定义自己的自定义结构日志格式，并在application.properties中使用它**。这在支持的日志格式不符合我们的用例的情况下很有用。

首先，我们需要实现StructuredLogFormatter接口并重写其format()方法：

```java
class MyStructuredLoggingFormatter implements StructuredLogFormatter<ILoggingEvent> {
    @Override
    public String format(ILoggingEvent event) {
       return "time=" + event.getTimeStamp() + " level=" + event.getLevel() + " message=" + event.getMessage() + "\n";
    }
}
```

在这里，我们的自定义格式是文本格式，而不是标准JSON。这提供了灵活性，我们可以根据任何格式(JSON、XML等)构建日志。

接下来，让我们在application.properties中定义自定义配置：

```properties
logging.structured.format.console=cn.tuyucheng.taketoday.springstructuredlogging.MyStructuredLoggingFormatter
```

在这里，我们定义了MyStructuredLoggingFormatter的完全限定类名。

这是日志输出：

```text
time=1734598194538 level=INFO message=Hello structured logging!
```

输出为文本格式，其中的键和值对代表日志详细信息。

如果支持的格式不符合我们的需求，自定义格式可能会很有优势。

此外，我们可以使用JSONWriter编写自定义格式的JSON：

```java
private final JsonWriter<ILoggingEvent> writer = JsonWriter.<ILoggingEvent>of((members) -> {
    members.add("time", ILoggingEvent::getInstant);
    members.add("level", ILoggingEvent::getLevel);
    members.add("thread", ILoggingEvent::getThreadName);
    members.add("message", ILoggingEvent::getFormattedMessage);
    members.add("application").usingMembers((application) -> {
        application.add("name", "StructuredLoggingDemo");
        application.add("version", "1.0.0");
    });
    members.add("node").usingMembers((node) -> {
        node.add("hostname", "node-1");
        node.add("ip", "10.0.0.7");
    });
}).withNewLineAtEnd();
```

接下来，我们将writer()方法集成到format()方法中：

```java
@Override
public String format(ILoggingEvent event) {
    return this.writer.writeToString(event);
}
```

输出的日志采用JSON格式：

```json
{
    "time": "2024-12-19T08:55:13.284101533Z",
    "level": "INFO",
    "thread": "main",
    "message": "No active profile set, falling back to 1 default profile: \"default\"",
    "application": {
        "name": "StructuredLoggingDemo",
        "version": "1.0.0"
    },
    "node": {
        "hostname": "node-1",
        "ip": "10.0.0.7"
    }
}
```

根据用例，编写自定义格式可以更灵活地处理Spring Boot中默认不支持的日志聚合。

### 4.6 记录到文件

我们之前的示例直接将日志记录到控制台。但是，我们可以在控制台中保留人工日志格式，并通过修改配置将结构化日志写入文件：

```properties
logging.structured.format.file=ecs
logging.file.name=log.json
```

这里我们使用file属性而不是console属性，这会在项目根目录中创建一个包含结构化日志的log.json文件。

## 5. 总结

在本文中，我们学习了如何使用application.properties配置来配置应用程序以进行结构化日志记录。此外，我们还看到了支持的结构化日志格式的不同示例。

最后，我们了解了如何编写自定义格式并通过配置使用它。