---
layout: post
title:  Apache Camel中的日志记录
category: apache
copyright: apache
excerpt: Apache Camel
---

## 1. 概述

[日志记录](https://www.baeldung.com/java-logging-intro)在软件开发中至关重要，因为它有助于记录应用程序的每一个足迹，它有助于跟踪应用程序的活动和状态。本质上，它对于调试目的很有用。

**[Apache Camel](https://www.baeldung.com/apache-camel-intro)提供了用于记录消息和交换的组件、接口和拦截器**，它通过在各种日志记录框架上提供抽象层来简化日志记录。

在本教程中，我们将介绍在Camel应用程序中记录消息和交换的4种方法。

## 2. 使用日志EIP

**Apache Camel 2.2提供了一个轻量级的log() DSL，用于记录来自路由的可读消息，它的主要用途是快速将消息输出到日志控制台。此外，我们可以将它与Camel简单表达式语言结合使用，以便将[路由](https://www.baeldung.com/spring-apache-camel-conditional-routing)的详细信息进一步记录到日志控制台**。

让我们看一个将文件从一个文件夹复制到另一个文件夹的示例：

```java
class FileCopierCamelRoute extends RouteBuilder {
    void configure() {
        from("file:data/inbox?noop=true")
                .log("We got an incoming file ${file:name} containing: ${body}")
                .to("file:data/outbox")
                .log("Successfully transfer file: ${file:name}");
    }
}
```

在上面的代码中，我们配置了一个RouteBuilder，用于将文件从收件箱传输到发件箱文件夹。首先，我们定义传入文件的位置；**接下来，我们使用log() DSL输出关于传入文件及其内容的可读日志。此外，我们使用简单表达式语言获取文件名和文件内容作为日志消息的一部分**。

这是日志输出：

```text
14:39:23.389 [Camel (camel-1) thread #1 - file://data/inbox] INFO  route1 - We got an incoming file welcome.txt containing: Welcome to Tuyucheng
14:39:23.423 [Camel (camel-1) thread #1 - file://data/inbox] INFO  route1 - Successlly transfer file: welcome.txt
```

与Log组件和Tracer拦截器相比，log() DSL是轻量级的。

此外，我们可以明确指定日志级别和名称：

```java
// ...
.log(LoggingLevel.DEBUG,"Output Process","The Process ${id}")
// ...
```

在这里，我们在传递日志消息之前指定日志级别和名称。我们还提供了WARN、TRACE和 OFF选项作为日志级别，**当未指定调试级别时，log() DSL将使用INFO级别**。

## 3. 使用Processor接口

Processor是Apache Camel中的一个重要接口，它允许访问exchange进行进一步的操作，它使我们能够灵活地修改exchange主体。此外，我们也可以使用它来输出易于阅读的日志消息。

首先，Processor本身并不是一个日志记录工具，因此，我们需要创建一个Logger实例来使用它。Apache Camel默认使用[SLF4J](https://www.baeldung.com/slf4j-with-log4j2-logback)库，让我们创建一个Logger实例：

```java
private static final Logger LOGGER = LoggerFactory.getLogger(FileCopierCamelRoute.class);
```

接下来，我们看一个将消息传递给Bean以进行进一步操作的示例：

```java
void configure() {
    from("file:data/inbox?noop=true")
     .to("log:cn.tuyucheng.taketoday.apachecamellogging?level=INFO")
     .process(process -> LOGGER.info("We are passing the message to a FileProcesor bean to capitalize the message body"))
     .bean(FileProcessor.class)
     .to("file:data/outbox")
}
```

这里，我们将传入的消息传递给FileProcessor Bean，以将文件内容转换为大写。但是，在将消息传递给用于处理的Bean之前，我们会通过创建Processor的实例来记录一条信息。

最后我们看看日志输出：

```text
14:50:47.048 [Camel (camel-1) thread #1 - file://data/inbox] INFO  c.t.t.a.FileCopierCamelRoute - We are passing the message to a FileProcesor to Capitalize the message body
```

从上面的输出来看，自定义日志消息输出到控制台。

## 4. 使用Log组件

Apache Camel提供了一个Log组件，用于将Camel消息记录到控制台输出。**要使用Log组件，我们可以将消息路由到它**：

```java
void configure() {
    from("file:data/inbox?noop=true")
        .to("log:cn.tuyucheng.taketoday.apachecamellogging?level=INFO")
        .bean(FileProcessor.class)
        .to("file:data/outbox")
        .to("log:cn.tuyucheng.taketoday.apachecamellogging")
}
```

这里，我们在两个地方使用了Log组件。首先，我们使用level选项以INFO日志级别记录消息正文。其次，我们在操作文件后记录消息正文，但我们没有指定日志级别。

**值得注意的是，在未指定日志记录级别的情况下，日志组件使用INFO级别作为默认级别**。

这是日志输出：

```text
09:36:32.432 [Camel (camel-1) thread #1 - file://data/inbox] INFO  cn.tuyucheng.taketoday.apachecamellogging - Exchange[ExchangePattern: InOnly, BodyType: org.apache.camel.component.file.GenericFile, Body: [Body is file based: GenericFile[welcome.txt]]]
09:36:32.454 [Camel (camel-1) thread #1 - file://data/inbox] INFO  cn.tuyucheng.taketoday.apachecamellogging - Exchange[ExchangePattern: InOnly, BodyType: String, Body: WELCOME TO TUYUCHENG]
```

此外，我们可以通过添加showBodyType和maxChars选项来减少输出的冗余度：

```java
.to("log:cn.tuyucheng.taketoday.apachecamellogging?showBodyType=false&maxChars=20")
```

上述代码中，我们忽略了消息体的时间，精简了消息体字符数为20个。

## 5. 使用Tracer

Tracer是Apache Camel架构的一部分，用于记录运行时消息的路由方式，**它跟踪路由过程中的交换快照**，可以拦截从一个节点到另一个节点的消息移动。

要使用Tracer，我们必须在路由配置方法中启用它：

```java
getContext().setTracing(true);
```

这使得Tracer拦截器能够拦截所有交换过程并将其记录到日志控制台。

让我们看一个使Tracer能够跟踪交换过程的示例代码：

```java
void configure() {
    getContext().setTracing(true);
    from("file:data/json?noop=true")
        .unmarshal().json(JsonLibrary.Jackson)
        .bean(FileProcessor.class, "transform")
        .marshal().json(JsonLibrary.Jackson)
        .to("file:data/output");
}
```

在上面的代码中，我们从源复制一个[JSON](https://www.baeldung.com/java-camel-jackson-json-array)文件，并将其转换为Camel可以操作的数据结构，我们将内容传递给一个Bean来修改文件内容。接下来，我们将消息转换为JSON并将其发送到目标。

这是来自Tracer拦截器的日志：

```text
// ...
09:23:10.767 [Camel (camel-1) thread #1 - file://data/json] INFO  FileCopierTracerCamelRoute:14 - *--> [route1      ] [from[file:data/json?noop=true]   ]
09:23:10.768 [Camel (camel-1) thread #1 - file://data/json] INFO  FileCopierTracerCamelRoute:14 -      [route1      ] [log:input?level=INFO             ]
// ...
```

在上面的输出中，Tracer记录了从生产者到消费者发生的每一个流程和交换，这对于调试非常有用。

值得注意的是，日志输出很详细，但为了简单起见，我们只显示必要的日志消息。

## 6. 总结

在本文中，我们学习了4中Apache Camel日志记录的方法。此外，我们还看到了使用log ()DSL、Tracer、Processor和Log组件将消息记录到控制台的示例，log() DSL和Processor接口非常适合用于人类可读的日志，而Log组件和Tracer则适用于调试中使用的更复杂的日志。