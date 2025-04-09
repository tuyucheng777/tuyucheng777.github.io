---
layout: post
title:  Java中的Log4j和log4j.properties文件指南
category: log
copyright: log
excerpt: Log4j
---

## 1. 简介

Log4J是一个流行的开源[日志框架](https://www.baeldung.com/java-logging-intro)，用Java编写，各种基于Java的应用程序都广泛使用Log4j。此外，它线程安全、速度快，并且提供命名的Logger层次结构。

**Log4j 1.x于2015年8月5日终止使用，因此，截至今天，[Log4j2](https://www.baeldung.com/log4j2-appenders-layouts-filters)是Log4j的最新升级**。

在本教程中，我们将了解Log4j以及如何使用Java中的log4j.properties文件配置核心Log4j组件。

## 2. Maven设置

首先，我们需要pom.xml中的log4j-core依赖：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.17.1</version>
</dependency>
```

可以在[此处](https://mvnrepository.com/artifact/log4j/log4j)找到log4j-core的最新版本。

## 3. Log4j API

Log4j API提供了根据不同优先级传递日志信息的机制，并将其定向到不同目的地，例如文件、控制台、数据库等。它还支持在将日志事件传递给logger或[appender](https://www.baeldung.com/log4j2-appenders-layouts-filters)之前对其进行过滤。

Log4j API具有分层架构，在Log4j框架中提供两种类型的对象-核心对象和支持对象。

## 4. Log4j组件

Log4j有3个主要组件：logger、appender和layout，它们可以一起使用，在所需的目标位置打印自定义日志语句，让我们简要地看一下它们。

### 4.1 Logger

Logger对象负责表示日志信息，它是Log4j架构中的第一个强制层，Logger类在org.apache.log4j包中定义。

通常，**我们为每个应用程序类创建一个Logger实例，以记录属于该类的重要事件**。此外，我们通常在类的开头使用接收类名作为参数的静态工厂方法创建此实例：

```java
private static final Logger logger = Logger.getLogger(JavaClass.class.getName());
```

随后，我们可以使用Logger类的各种方法根据类别记录或打印重要事件，这些方法是trace()、debug()、info()、warn()、error()、fatal()，确定日志记录请求的级别。

Logger方法的优先级顺序为：TRACE < DEBUG < INFO < WARN < ERROR < FATAL。因此，这些方法根据log4j.properties文件中设置的记录器级别打印日志消息，这意味着如果我们将记录器级别设置为INFO，那么所有INFO、WARN、ERROR和FATAL事件都将被记录。

### 4.2 Appender

Appender表示日志输出的目标，我们可以使用Log4j将日志打印到多个首选目标，如控制台、文件、远程套接字服务器、数据库等，我们将这些输出目标称为Appender。此外，我们可以将多个appender附加到一个Logger。

Appender按照[附加器可加性规则](https://logging.apache.org/log4j/1.2/manual.html)工作，**该规则规定，任何Logger的日志语句输出都将发送到其所有附加器及其祖先(层次结构中较高的附加器)**。

Log4j为文件、控制台、GUI组件、远程套接字服务器、JMS等定义了多个appender。

### 4.3 Layout

我们使用layout来自定义日志语句的格式，我们可以通过将布局与已定义的appender关联来实现这一点。因此，layout和appender的组合可以帮助我们将格式化的日志语句发送到所需的目的地。

我们可以使用转换模式指定日志语句的格式，**[PatternLayout](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html)类详细说明了我们可以根据需要使用的转换字符**。

我们还将通过以下章节中的示例了解一些转换字符。

## 5. log4j.properties文件

我们可以使用XML或属性文件来配置Log4j，log4j.properties文件以键值对的形式存储配置。

log4j属性配置文件的默认名称是log4j.properties，Logger在[CLASSPATH](https://www.baeldung.com/java-classpath-vs-build-path)中查找此文件名；但是，**如果我们需要使用不同的配置文件名，我们可以使用系统属性log4j.configuration进行设置**。

log4j.properties文件包含appender的规范、其名称和类型以及layout模式，它还包含有关默认根Logger及其日志级别的规范。

## 6. log4j.properties文件的语法

在一般的log4j.properties文件中，我们定义以下配置：

- 根记录器及其级别，并在此处为附加器提供一个名称
- 然后，我们为定义的附加器名称分配一个有效的附加器
- 最后，我们为定义的appender定义布局，目标，级别等

我们来看一下一般的log4.properties文件的语法：

```properties
# The root logger with appender name 
log4j.rootLogger = DEBUG, NAME
  
# Assign NAME a valid appender  
log4j.appender.NAME = org.apache.log4j.FileAppender

# Define the layout for NAME
log4j.appender.NAME.layout=org.apache.log4j.PatternLayout
log4j.appender.NAME.layout.conversionPattern=%m%n
```

这里的NAME是Appender的名字，正如前面所讨论的，**我们可以将多个appender附加到一个Logger，以将日志发送到不同的目的地**。

## 7. 示例

现在我们通过一些示例来了解不同appender的log4j.properties文件配置。

### 7.1 示例程序

让我们从一个记录一些消息的示例应用程序开始：

```java
import org.apache.log4j.Logger;

public class Log4jExample {

    private static Logger logger = Logger.getLogger(Log4jExample.class);

    public static void main(String[] args) throws InterruptedException {
        for(int i = 1; i <= 2000; i++) {
            logger.info("This is the " + i + " time I say 'Hello World'.");
            Thread.sleep(100);
        }
    }
}
```

该应用程序很简单-它会循环写入一些消息，每次迭代之间会有短暂的延迟。它有2000次迭代，每次迭代都会暂停100毫秒。因此，完成执行大约需要三分半钟，我们将在下面的示例中使用此应用程序。

### 7.2 控制台日志记录

如果没有配置文件，控制台是记录消息的默认位置；让我们使用根记录器为ConsoleAppender创建log4j.properties配置，并为其定义日志记录级别：

```properties
# Root Logger
log4j.rootLogger=INFO, stdout

# Direct log messages to stdout
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target=System.out
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
```

在这里，我们定义了一个具有以下规范的log4j.properties文件：

- 我们将根记录器的级别定义为INFO，这意味着所有级别为INFO及以上的日志事件都将被记录，我们还将附加器的名称定义为stdout。
- 由于我们希望将日志直接发送到控制台，因此我们将Appender指定为org.apache.log4j.ConsoleAppender，并将目标指定为System.out。
- 最后，我们指定了PatternLayout的格式，我们想要使用ConversionPattern来打印日志。

再来了解一下ConversionPattern中使用的每个转换字符的含义：

- %d以定义的格式添加时间戳
- %-5p将日志级别信息添加到每个日志语句中，它表示日志事件的优先级应左对齐，宽度为5个字符
- %c{1}打印限定的类名，后面可以选择跟着包名(精度限定符)，也就是记录具体的日志语句
- %L打印特定日志事件的行号
- %m打印实际的日志消息
- %n在每个日志语句后添加一个新行

因此，当我们运行示例应用程序时，我们会在控制台上打印以下几行：

```text
2023-08-01 00:27:25 INFO Log4jExample:15 - This is the 1 time I say 'Hello World'.
...
...
2023-08-01 00:27:25 INFO Log4jExample:15 - This is the 2000 time I say 'Hello World'.
```

**[PatternLayout](https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/PatternLayout.html)类的文档详细介绍了我们可以根据需要使用的转换字符**。

### 7.3 多个目的地

如前所述，我们可以将日志事件重定向到多个目的地：

```properties
# Root logger  
log4j.rootLogger=INFO, file, stdout  
  
# Direct to a file
log4j.appender.file=org.apache.log4j.RollingFileAppender  
log4j.appender.file.File=C:\\Tuyucheng\\app.log  
log4j.appender.file.MaxFileSize=5KB  
log4j.appender.file.MaxBackupIndex=2  
log4j.appender.file.layout=org.apache.log4j.PatternLayout  
log4j.appender.file.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n  
   
# Direct to console
log4j.appender.stdout=org.apache.log4j.ConsoleAppender  
log4j.appender.stdout.Target=System.out  
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout  
log4j.appender.stdout.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n  
```

这里，我们使用了两个appender将日志消息重定向到文件和控制台。此外，我们还为文件Appender分配了一个[RollingFileAppender](https://www.baeldung.com/java-logging-rolling-file-appenders)，当我们知道日志文件的大小可能会随着时间的推移而增长时，我们会使用RollingFileAppender。

**在上述示例中，我们使用了RollingFileAppender，它使用MaxFileSize和MaxBackupIndex参数根据日志文件的大小和数量滚动日志文件**。因此，当日志文件的大小达到5KB时，它将滚动，并且我们最多保留两个滚动日志文件作为备份。

当我们运行示例应用程序时，我们获得包含与上一个示例相同的日志语句的以下文件：

```text
31/01/2023  10:28    138 app.log
31/01/2023  10:28  5.281 app.log.1
31/01/2023  10:28  5.281 app.log.2
```

我们可以在[基于大小滚动日志文件](https://www.baeldung.com/java-logging-rolling-file-appenders#2-rolling-based-on-file-size)的示例中发现有关执行的详细解释。

## 8. 总结

在本文中，我们探索了Log4j及其3个组件-logger、appender和layout，并介绍了log4j.properties文件的语法以及配置log4j.properties文件的一些简单示例。