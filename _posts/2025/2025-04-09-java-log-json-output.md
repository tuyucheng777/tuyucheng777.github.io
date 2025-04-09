---
layout: post
title:  以JSON格式获取日志输出
category: log
copyright: log
excerpt: Log4j
---

## 1. 简介

当今，大多数Java日志库都提供不同的日志格式化布局选项，以准确满足每个项目的需求。

在本快速教程中，我们希望**将日志条目格式化并输出为JSON**。我们将了解如何针对两个最广泛使用的日志库执行此操作：[Log4j2](https://www.baeldung.com/log4j2-appenders-layouts-filters)和[Logback](https://www.baeldung.com/custom-logback-appender)。

两者内部都使用[Jackson](https://www.baeldung.com/java-json#jackson)以JSON格式表示日志。

有关这些库的介绍，请参阅我们的[Java日志简介](https://www.baeldung.com/java-logging-intro)文章。

## 2. Log4j2

Log4j2是Java最流行的日志库Log4j的直接继承者。

由于它是Java项目的新标准，我们将展示如何配置它以输出JSON。

### 2.1 Maven依赖

首先，我们必须在pom.xml文件中包含以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-core</artifactId>
        <version>2.19.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.logging.log4j</groupId>
        <artifactId>log4j-api</artifactId>
        <version>2.19.0</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.17.2</version>
    </dependency>
</dependencies>
```

可以在Maven Central上找到以前依赖的最新版本：[log4j-api](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api)、[log4j-core](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core)、[jackson-databind](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind)。

### 2.2 使用JsonLayout

然后，在我们的log4j2.xml文件中，**我们可以创建一个使用JsonLayout的新Appender和一个使用此Appender的新Logger**：

```xml
<Appenders>
    <Console name="ConsoleJSONAppender" target="SYSTEM_OUT">
        <JsonLayout complete="false" compact="false">
            <KeyValuePair key="myCustomField" value="myCustomValue"/>
        </JsonLayout>
    </Console>
</Appenders>

<Logger name="CONSOLE_JSON_APPENDER" level="TRACE" additivity="false">
    <AppenderRef ref="ConsoleJSONAppender"/>
</Logger>
```

正如我们在示例配置中看到的，可以使用KeyValuePair将我们自己的值添加到日志中，它甚至支持查看日志上下文。

将compact参数设置为false将增加输出的大小并使其更易于阅读。

现在，让我们测试一下我们的配置。在我们的代码中，我们可以实例化新的JSON记录器并创建新的调试级别跟踪：

```java
Logger logger = LogManager.getLogger("CONSOLE_JSON_APPENDER");
logger.debug("Debug message");
```

上述代码的调试输出消息如下：

```json
{
    "instant" : {
        "epochSecond" : 1696419692,
        "nanoOfSecond" : 479118362
    },
    "thread" : "main",
    "level" : "DEBUG",
    "loggerName" : "CONSOLE_JSON_APPENDER",
    "message" : "Debug message",
    "endOfBatch" : false,
    "loggerFqcn" : "org.apache.logging.log4j.spi.AbstractLogger",
    "threadId" : 1,
    "threadPriority" : 5,
    "myCustomField" : "myCustomValue"
}
```

### 2.3 使用JsonTemplateLayout

在上一节中，我们了解了如何使用JsonLayout属性。从2.14.0版本开始，该属性已被弃用并由JsonTemplateLayout取代。

**JsonTemplateLayout提供了增强的功能和更高的效率，因为它默认经过优化以尽快对日志事件进行编码**。

此外，它还支持无垃圾日志记录，从而为其带来一些性能优势，因为垃圾收集器暂停会影响性能。要启用无垃圾日志记录，我们需要将log4j2.garbagefreeThreadContextMap和log4j2.enableThreadLocals属性设置为true：

```properties
-Dlog4j2.garbagefreeThreadContextMap=true 
-Dlog4j2.enableThreadlocals=true 
```

要使用JsonTemplateLayout，我们需要将[log4j-layout-template-json](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-layout-template-json)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-layout-template-json</artifactId>
    <version>2.24.3</version>
</dependency>
```

接下来，让我们修改我们的Appender以使用JsonTemplateLayout：

```xml
<Appenders> 
    <Console name="ConsoleJSONAppender" target="SYSTEM_OUT">
        <JsonTemplateLayout eventTemplateUri="classpath:JsonLayout.json">
            <EventTemplateAdditionalField key="myCustomField" value="myCustomValue"/>
        </JsonTemplateLayout>
    </Console>
</Appenders>
```

这里我们使用JsonTemplateLayout并使用eventTemplateUri指定JSON布局格式，eventTemplateUri定义JSON输出的格式，当未指定eventTemplateUri时，它默认使用Elastic Common Schema(ECS)格式(classpath:EcsLayout.json)。

其他支持的模板包括[Graylog扩展日志格式](https://www.baeldung.com/graylog-with-spring-boot)(GELF)，其值为-classpath:GelfLayout.json。

JsonLayout.json模板旨在轻松地从JsonLayout过渡到JsonTemplateLayout，在我们的例子中，我们使用JsonLayout.json来维护上一节示例中的初始格式：

```json
{
    "instant": {
        "epochSecond": 1736320992,
        "nanoOfSecond": 804274875
    },
    "thread": "main",
    "level": "DEBUG",
    "loggerName": "CONSOLE_JSON_APPENDER",
    "message": "Debug message",
    "endOfBatch": false,
    "loggerFqcn": "org.apache.logging.log4j.spi.AbstractLogger",
    "threadId": 1,
    "threadPriority": 5,
    "myCustomField": "myCustomValue"
}
```

与使用KeyValuePair属性添加自定义键值对的JsonLayout不同，我们使用名为EventTemplateAdditionalField的属性来添加键和值。

## 3. Logback

Logback可以视为Log4j的另一个继承者，它由相同的开发人员编写，据称比其前身更高效、更快速。

那么，让我们看看如何配置它以获取JSON格式的日志输出。

### 3.1 Maven依赖

让我们在pom.xml中包含以下依赖：

```xml
<dependencies>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.4.8</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback.contrib</groupId>
        <artifactId>logback-json-classic</artifactId>
        <version>0.1.5</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback.contrib</groupId>
        <artifactId>logback-jackson</artifactId>
        <version>0.1.5</version>
    </dependency>
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
        <version>2.15.2</version>
    </dependency>
</dependencies>
```

可以在这里检查这些依赖的最新版本：[logback-classic](https://mvnrepository.com/artifact/ch.qos.logback/logback-classic)、[logback-json-classic](https://mvnrepository.com/artifact/ch.qos.logback.contrib/logback-json-classic)、[logback-jackson](https://mvnrepository.com/artifact/ch.qos.logback.contrib/logback-jackson)、[jackson-databind](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind)。

### 3.2 使用JsonLayout

首先，**我们在logback-test.xml中创建一个使用JsonLayout和JacksonJsonFormatter的新附加器**。

之后，我们可以创建一个使用此附加器的新记录器：

```xml
<appender name="json" class="ch.qos.logback.core.ConsoleAppender">
    <layout class="ch.qos.logback.contrib.json.classic.JsonLayout">
        <jsonFormatter
            class="ch.qos.logback.contrib.jackson.JacksonJsonFormatter">
            <prettyPrint>true</prettyPrint>
        </jsonFormatter>
        <timestampFormat>yyyy-MM-dd' 'HH:mm:ss.SSS</timestampFormat>
    </layout>
</appender>

<logger name="jsonLogger" level="TRACE">
    <appender-ref ref="json" />
</logger>
```

如我们所见，启用参数prettyPrint可以获得人类可读的JSON。

为了测试我们的配置，让我们在代码中实例化记录器并记录调试消息：

```java
Logger logger = LoggerFactory.getLogger("jsonLogger");
logger.debug("Debug message");
```

通过此，我们将获得以下输出：

```json
{
    "timestamp":"2017-12-14 23:36:22.305",
    "level":"DEBUG",
    "thread":"main",
    "logger":"jsonLogger",
    "message":"Debug message",
    "context":"default"
}
```

### 3.3 使用JsonEncoder

**以JSON格式记录输出的另一种方法是使用[JsonEncoder](https://logback.qos.ch/manual/encoders.html#JsonEncoder)**，它将日志事件转换为有效的JSON文本。

让我们添加一个使用JsonEncoder的新附加器以及一个使用此附加器的新记录器：

```xml
<appender name="jsonEncoder" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="ch.qos.logback.classic.encoder.JsonEncoder"/>
</appender>

<logger name="jsonEncoderLogger" level="TRACE">
    <appender-ref ref="jsonEncoder" />
</logger>
```

现在，让我们实例化记录器并调用debug()来生成日志消息：

```java
Logger logger = LoggerFactory.getLogger("jsonEncoderLogger");
logger.debug("Debug message");
```

执行此代码后，我们得到以下输出：

```json
{
    "sequenceNumber":0,
    "timestamp":1696689301574,
    "nanoseconds":574716015,
    "level":"DEBUG",
    "threadName":"main",
    "loggerName":"jsonEncoderLogger",
    "context":
        {
            "name":"default",
            "birthdate":1696689301038,
            "properties":{}
        },
    "mdc": {},
    "message":"Debug message",
    "throwable":null
}
```

这里，message字段表示日志消息。此外，context字段显示日志上下文。除非我们设置了多个日志上下文，否则它通常是默认的。

## 4. 总结

在本文中，我们了解了如何轻松配置Log4j2和Logback以获得JSON输出格式，我们将解析的所有复杂性委托给日志库，因此我们不需要更改任何现有的记录器调用。