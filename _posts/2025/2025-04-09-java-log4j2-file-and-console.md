---
layout: post
title:  Log4j2–记录到文件和控制台
category: log
copyright: log
excerpt: Log4j2
---

## 1. 概述

在本教程中，我们将探讨如何使用[Apache Log4j2](https://www.baeldung.com/java-logging-intro#Log4j)库将消息记录到文件和控制台。

这在非生产环境中非常有用，我们可能希望在控制台中看到调试消息，并且我们可能希望将更高级别的日志保存到文件中以供以后分析。

## 2. 项目设置

让我们先创建一个Java项目，我们将添加log4j2依赖，并了解如何配置和使用记录器。

### 2.1 Log4j2依赖

让我们将log4j2依赖添加到我们的项目中，我们需要[Apache Log4J Core](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core)和[Apache Log4J API](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api)依赖：

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
</dependencies>
```

### 2.2 应用程序类

现在让我们使用log4j2库为我们的应用程序添加一些日志记录：

```java
public class Log4j2ConsoleAndFile {

    private static final Logger logger = LogManager.getLogger(Log4j2ConsoleAndFile.class);

    public static void main(String[] args) {
        logger.info("Hello World!");
        logger.debug("Hello World!");
    }
}
```

## 3. Log4j2配置

要自动配置记录器，**我们需要在类路径上有一个配置文件，它可以是JSON、XML、YAML或属性格式**。该文件应命名为log4j2，对于我们的示例，让我们使用名为log4j2.properties的配置文件。

### 3.1 记录到控制台

要将日志记录到任何目标，我们首先需要定义一个将日志记录到控制台的[附加程序](https://www.baeldung.com/log4j2-appenders-layouts-filters)，让我们看看执行此操作的配置：

```properties
appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = [%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n
```

让我们了解配置的每个组件：

- appender.console.type：在这里，我们**指定用于记录日志的附加器的类型**；类型Console指定附加器将仅写入控制台，我们应该注意，键名中的单词console只是一种惯例，并非强制性的
- appender.console.name：我们**可以给出任何唯一的名称，以便稍后引用该附加器**
- appender.console.layout.type：**决定用于格式化日志消息的布局类的名称**
- appender.console.layout.pattern：这是**用于格式化日志消息的模式**

**要启用控制台记录器，我们需要将控制台附加器添加到根记录器，我们可以使用上面指定的名称来执行此操作**：

```properties
rootLogger=debug, STDOUT
```

使用此配置，我们会将所有debug及以上级别的消息记录到控制台。对于在本地环境中运行的控制台，debug级别日志记录很常见。

### 3.2 记录到文件

类似地，我们可以配置记录器以记录到文件中，这通常对于持久化日志很有用。让我们定义一个文件附加器：

```properties
appender.file.type = File
appender.file.name = LOGFILE
appender.file.fileName=logs/log4j.log
appender.file.layout.type=PatternLayout
appender.file.layout.pattern=[%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n
appender.file.filter.threshold.type = ThresholdFilter
appender.file.filter.threshold.level = info
```

**对于文件附加器来说，还必须指定文件名**。

除此之外，我们还要设置阈值级别。由于我们要记录到文件，因此我们不想记录所有消息，因为这会占用大量持久存储空间。我们只想记录级别为info或以上的消息，**可以使用过滤器ThresholdFilter并设置其级别info来做到这一点**。 

**要启用文件记录器，我们需要将文件附加器添加到根记录器**，需要更改rootLogger配置以包含文件附加器：

```properties
rootLogger=debug, STDOUT, LOGFILE
```

即使我们在根级别使用了debug级别，文件记录器也只会记录info及以上级别的消息。

## 4. 测试

现在让我们运行应用程序并检查控制台中的输出：

```text
12:43:47,891 INFO  Application:8 - Hello World!
12:43:47,892 DEBUG Application:9 - Hello World!
```

正如预期的那样，我们可以在控制台中看到这两条日志消息。如果我们检查位于路径logs/log4j.log的日志文件，我们只能看到info级别的日志消息：

```text
12:43:47,891 INFO  Application:8 - Hello World!
```

## 5. 总结

在本文中，我们学习了如何将消息记录到控制台和文件。我们创建了一个Java项目，使用属性文件配置了Log4j2，并测试了消息是否同时打印到控制台和文件。