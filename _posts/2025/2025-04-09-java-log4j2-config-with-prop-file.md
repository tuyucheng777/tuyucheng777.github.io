---
layout: post
title:  使用属性文件配置Log4j2
category: log
copyright: log
excerpt: Log4j2
---

## 1. 简介

Log4j2是一个流行的开源[日志框架](https://www.baeldung.com/java-logging-intro)，用Java编写，它是为了克服Log4j的各种架构缺陷而推出的。它线程安全、速度快，并且比其前身有各种改进。

**[Log4j2](https://www.baeldung.com/java-logging-intro)是经典Log4j框架的最新改进版本，该框架于2015年8月5日终止其生命周期。但是，Log4j仍然作为日志记录框架广泛用于许多Java企业应用程序中**。

在本教程中，我们将了解Log4j 2、它相对于Log4j的优势以及如何使用Java中的log4j2.properties文件配置其核心组件。

## 2. Maven设置

首先，我们在pom.xml中添加log4j-core依赖：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.20.0</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.20.0</version>
</dependency>
```

可以在Maven Repository中找到最新版本的[log4j-core](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core)和[log4j-api](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api)。

## 3. Log4j2记录器

与Log4j不同，在Log4j中我们使用Logger.getLogger()来获取具有特定名称的Logger实例，而在Log4j 2中我们使用LogManager.getLogger()：

```java
private static final Logger logger = LogManager.getLogger(Log4j2Example.class);
```

LogManager从配置文件或配置类中读取初始配置参数，Logger与LoggerConfig相关联，后者与实际传递日志事件的Appender相关联。

**如果我们通过传递相同的(类)名称来调用LogManager.getLogger()，我们将始终获得相同记录器实例的引用**。

Logger和LoggerConfig都是命名实体，每个Logger引用一个LoggerConfig，LoggerConfig可以引用其父级，从而达到相同的效果。

Logger遵循命名层次结构，这意味着名为“cn.tuyucheng.taketoday”的LoggerConfig是名为“cn.tuyucheng.taketoday.foo”的LoggerConfig的父级。

## 4. Log4j2配置

与仅支持通过属性和XML格式进行配置的Log4j不同，我们**可以使用JSON、XML、YAML或属性格式定义Log4j2配置**，所有这些格式在功能上都是等效的。因此，我们可以轻松地将以一种格式完成的配置转换为任何其他格式。

**此外，Logj2支持[自动配置](https://logging.apache.org/log4j/2.x/manual/configuration.html#automatic-configuration)，这意味着它能够在初始化期间自动配置自身**。

它在开始时扫描并定位所有ConfigurationFactory插件，然后按从高到低的加权顺序排列它们-属性文件的优先级最高，为8，其次是YAML、JSON和XML。

这意味着如果我们同时具有属性文件和XML文件形式的日志配置，则属性文件将优先。

## 5. log4j2.properties文件

Log4j2发布时还不支持通过properties文件进行配置，从2.4版本开始才支持properties文件。

默认的属性配置文件始终是[log4j2.properties](https://www.baeldung.com/java-classpath-vs-build-path)，Logger从CLASSPATH获取此文件的引用。

但是，**如果我们需要使用不同的配置文件名，我们可以使用系统属性log4j.configurationFile来设置它**。

系统属性可能引用本地文件系统，也可能包含URL。如果无法找到配置文件，Log4j2会提供DefaultConfiguration。在这种情况下，我们会将日志输出重定向到控制台，并将根记录器级别设置为ERROR。

## 6. log4j2.properties文件的语法

log4j2.properties文件的语法与log4j.properties不同，在log4j.properties文件中，每个配置都以“log4j”开头，而在log4j2.properties配置中已省略该部分。

我们来看一下通用的log4j2.properties文件的语法：

```properties
# The root logger with appender name 
rootLogger = DEBUG, STDOUT

# Assign STDOUT a valid appender & define its layout  
appender.console.name = STDOUT
appender.console.type = Console
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = %msg%n
```

这里的STDOUT是Appender的名字，正如前面所讨论的，我们可以将多个Appender附加到一个Logger上，以将日志发送到不同的目的地。

**此外，我们应该在每个Log4j2配置中定义一个根记录器，否则，将使用具有ERROR级别和ConsoleAppender的默认根LoggerConfig**。

## 7. 示例

现在我们通过一些示例来了解[不同Appender](https://www.baeldung.com/log4j2-appenders-layouts-filters)的log4j2.properties文件配置。

### 7.1 示例程序

让我们从一个记录一些消息的示例应用程序开始：

```java
public class Log4j2ConsoleAndFile {

    private static final Logger logger = LogManager.getLogger(Log4j2ConsoleAndFile.class);

    public static void main(String[] args) {
        logger.info("Hello World!");
        logger.debug("Hello World!");
    }
}
```

### 7.2 控制台日志记录

如果没有配置文件，控制台是记录消息的默认位置。让我们为具有根记录器的控制台Appender创建log4j2.properties配置，并为其定义日志记录级别：

```properties
# Root Logger
rootLogger=DEBUG, STDOUT

# Direct log messages to stdout
appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = [%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n
```

在这里，我们定义了一个具有以下规范的log4j2.properties文件：

- 我们将根记录器的级别定义为DEBUG，这意味着我们将获取所有级别为DEBUG及以上的日志事件，我们还将附加器的名称定义为STDOUT。
- 由于我们希望将日志直接发送到控制台，因此我们将Appender类型指定为Console；我们应该注意，键名中的console一词只是一种惯例，而不是强制性的。
- 然后，我们指定想要打印日志消息的模式。

让我们再了解一下使用的layout模式中每个转换字符的含义：

- %-5level在每个日志语句中添加日志级别信息，它表示日志事件的优先级左对齐，宽度为5个字符
- %d以定义的格式添加时间戳
- %t将线程名称添加到日志语句中
- %c{1}打印限定的类名，可选择后跟包名(精度限定符)，用于记录特定的日志语句
- %msg打印实际的日志消息
- %n在每个日志语句后添加一个新行

因此，当我们运行示例应用程序时，我们会在控制台上打印以下几行：

```text
[INFO ] 2023-08-05 23:04:03.255 [main] Log4j2ConsoleAndFile - Hello World!
[DEBUG] 2023-08-05 23:04:03.255 [main] Log4j2ConsoleAndFile - Hello World!
```

**[PatternLayout](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html)类详细说明了我们可以根据需要使用的转换字符**。

### 7.3 多个目的地

如前所述，我们可以将日志事件重定向到多个目的地：

```properties
# Root Logger
rootLogger=INFO, STDOUT, LOGFILE

# Direct log messages to STDOUT
appender.console.type = Console
appender.console.name = STDOUT
appender.console.layout.type = PatternLayout
appender.console.layout.pattern = [%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n

# Direct to a file
appender.file.type = File
appender.file.name = LOGFILE
appender.file.fileName = baeldung/logs/log4j2.log
appender.file.layout.type = PatternLayout
appender.file.layout.pattern = [%-5level] %d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %c{1} - %msg%n
appender.file.filter.threshold.type = ThresholdFilter
appender.file.filter.threshold.level = info
```

这里，我们使用了两个appender将日志消息重定向到文件和控制台，我们将它们命名为STDOUT和LOGFILE。此外，我们将这两个appender都添加到了root logger中。

**要将日志消息重定向到文件，我们需要指定文件名及其位置**。

我们还使用了ThresholdFilter，它可以过滤掉特定日志级别及以上的日志消息。最后，我们将threshold.level指定为INFO。因此，所有级别为INFO或以上的日志消息都将打印到文件中。

当我们运行示例应用程序时，我们只会在控制台和log4j2.log文件中打印以下行：

```text
[INFO ] 2023-08-05 23:04:03.255 [main] Log4j2ConsoleAndFile - Hello World!
```

## 8. 总结

在本文中，我们探讨了Log4j2及其相对于Log4j的优势，我们还了解了log4j2.properties文件的语法以及配置log4j2.properties文件的一些简单示例。