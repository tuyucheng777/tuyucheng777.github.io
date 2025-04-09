---
layout: post
title:  Java日志简介
category: log
copyright: log
excerpt: JUL
---

## 1. 概述

日志记录是理解和调试程序运行时行为的有力辅助工具，日志可捕获并保存重要数据，并可随时进行分析。

本文讨论了最流行的Java日志框架Log4j 2和Logback以及它们的前身Log4j，并简要介绍了SLF4J，这是一种为不同日志框架提供通用接口的日志门面。

## 2. 启用日志记录

本文讨论的所有日志记录框架都具有记录器、附加器和布局的概念。在项目内部启用日志记录遵循3个通用步骤：

1. 添加所需的库
2. 配置
3. 放置日志语句

接下来的部分将分别讨论每个框架的步骤。

## 3. Log4j2

Log4j 2是Log4j日志框架的一个新改进版本，最引人注目的改进是可以进行异步日志记录，Log4j 2需要以下库：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.6.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.6.1</version>
</dependency>
```

你可以在[这里](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api)找到log4j-api的最新版本，在[这里](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core)找到log4j-core的最新版本。

### 3.1 配置

配置Log4j 2主要基于log4j2.xml文件，首先要配置的是appender。

这些决定了日志消息将被路由到哪里，目的地可以是控制台、文件、套接字等。

Log4j 2有许多用于不同目的的附加器；你可以在官方[Log4j 2](https://logging.apache.org/log4j/2.x/manual/appenders.html)网站上找到更多信息。

让我们看一个简单的配置示例：

```xml
<Configuration status="debug" name="tuyucheng" packages="">
    <Appenders>
        <Console name="stdout" target="SYSTEM_OUT">
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss} %p %m%n"/>
        </Console>
    </Appenders>
</Configuration>
```

你可以为每个附加程序设置一个名称；例如，使用名称console而不是stdout。

注意PatternLayout元素-它决定了消息的外观。在我们的示例中，模式是根据模式参数设置的，其中%d确定日期模式，%p-日志级别输出，%m-记录消息的输出，%n-添加新行符号。有关模式的更多信息，请参阅官方[Log4j 2](https://logging.apache.org/log4j/2.x/manual/layouts.html)页面。

最后-为了启用一个附加器(或多个)，你需要将其添加到<Loggers\>部分：

```xml
<Loggers> 
    <Root level="error">
        <AppenderRef ref="STDOUT"/>
    </Root>
</Loggers>
```

### 3.2 记录到文件

有时，你需要将日志记录到文件中，因此我们将fout记录器添加到我们的配置中：

```xml
<Appenders>
    <File name="fout" fileName="tuyucheng.log" append="true">
        <PatternLayout>
            <Pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n</Pattern>
        </PatternLayout>
    </File>
</Appenders>
```

文件附加器有几个可以配置的参数：

- file：确定日志文件的文件名
- append：此参数的默认值为true，这意味着默认情况下，文件附加器将附加到现有文件而不是截断它
- 上例中描述的PatternLayout

为了启用File Appender，你需要将其添加到<Root\>部分：

```xml
<Root level="INFO">
    <AppenderRef ref="stdout" />
    <AppenderRef ref="fout"/>
</Root>
```

### 3.3 异步日志记录

如果要使Log4j 2异步，则需要将LMAX Disruptor库添加到pom.xml中，LMAX Disruptor是一个无锁的线程间通信库。

将Disruptor添加到pom.xml中：

```xml
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.3.4</version>
</dependency>
```

Disruptor的最新版本可以在[这里](https://mvnrepository.com/artifact/com.lmax/disruptor)找到。

如果你想使用LMAX Disruptor，你需要在配置中使用<asyncRoot\>而不是<Root\>。

```xml
<AsyncRoot level="DEBUG">
    <AppenderRef ref="stdout" />
    <AppenderRef ref="fout"/>
</AsyncRoot>
```

或者，你可以通过将系统属性Log4jContextSelector设置为org.apache.logging.log4j.core.async.AsyncLoggerContextSelector来启用异步日志记录。

当然，你可以在[Log4j2官方页面](https://logging.apache.org/log4j/2.x/manual/async.html)上阅读有关Log4j2异步记录器的配置的更多信息，并查看一些性能图。

### 3.4 使用

下面是一个简单示例，演示如何使用Log4j进行日志记录：

```java
import org.apache.logging.log4j.Logger;
import org.apache.logging.log4j.LogManager;

public class Log4jExample {

    private static Logger logger = LogManager.getLogger(Log4jExample.class);

    public static void main(String[] args) {
        logger.debug("Debug log message");
        logger.info("Info log message");
        logger.error("Error log message");
    }
}
```

运行后，应用程序将把以下消息记录到控制台和名为tuyucheng.log的文件中：

```text
2016-06-16 17:02:13 INFO  Info log message
2016-06-16 17:02:13 ERROR Error log message
```

如果将根日志级别提升至ERROR：

```xml
<level value="ERROR" />
```

输出如下所示：

```text
2016-06-16 17:02:13 ERROR Error log message
```

可以看到，将日志级别改为较高的参数会导致较低日志级别的消息不会打印到附加器。

方法logger.error也可用于记录发生的异常：

```java
try {
    // Here some exception can be thrown
} catch (Exception e) {
    logger.error("Error log message", throwable);
}
```

### 3.5 包级别配置

假设你需要显示日志级别为TRACE的消息-例如来自特定包(如cn.tuyucheng.taketoday.log4j2)的消息：

```java
logger.trace("Trace log message");
```

对于所有其他包，你希望继续仅记录INFO消息。

请记住，TRACE低于我们在配置中指定的根日志级别INFO。

要仅为其中一个包启用日志记录，你需要在log4j2.xml中的<Root\>之前添加以下部分：

```xml
<Logger name="cn.tuyucheng.taketoday.log4j2" level="debug">
    <AppenderRef ref="stdout"/>
</Logger>
```

它将启用cn.tuyucheng.taketoday.log4j包的日志记录，输出将如下所示：

```text
2016-06-16 17:02:13 TRACE Trace log message
2016-06-16 17:02:13 DEBUG Debug log message
2016-06-16 17:02:13 INFO  Info log message
2016-06-16 17:02:13 ERROR Error log message
```

## 4. Logback

Logback旨在成为Log4j的改进版本，由开发Log4j的同一位开发人员开发。

Logback还具有比Log4j多得多的功能，其中许多功能也被引入到了Log4j 2中；这里在[官方网站](http://logback.qos.ch/reasonsToSwitch.html)上快速浏览一下Logback的所有优点。

让我们首先向pom.xml添加以下依赖：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

此依赖将间接引入另外两个依赖，即logback-core和slf4j-api；请注意，可以在[此处](https://mvnrepository.com/artifact/ch.qos.logback/logback-classic)找到Logback的最新版本。

### 4.1 配置

现在我们来看一个Logback配置示例：

```xml
<configuration>
    # Console appender
    <appender name="stdout" class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            # Pattern of log message for console appender
            <Pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n</Pattern>
        </layout>
    </appender>

    # File appender
    <appender name="fout" class="ch.qos.logback.core.FileAppender">
        <file>tuyucheng.log</file>
        <append>false</append>
        <encoder>
            # Pattern of log message for file appender
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n</pattern>
        </encoder>
    </appender>

    # Override log level for specified package
    <logger name="cn.tuyucheng.taketoday.log4j" level="TRACE"/>

    <root level="INFO">
        <appender-ref ref="stdout" />
        <appender-ref ref="fout" />
    </root>
</configuration>
```

Logback使用SLF4J作为接口，因此需要导入SLF4J的Logger和LoggerFactory。

### 4.2 SLF4J

SLF4J为大多数Java日志框架提供了通用接口和抽象，它充当门面，并提供标准化API来访问日志框架的底层功能。

Logback使用SLF4J作为其功能的原生API，以下是使用Logback日志记录的示例：

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Log4jExample {

    private static Logger logger = LoggerFactory.getLogger(Log4jExample.class);

    public static void main(String[] args) {
        logger.debug("Debug log message");
        logger.info("Info log message");
        logger.error("Error log message");
    }
}
```

输出将与前面的示例保持相同。

## 5. Log4J

最后，让我们来看看Log4j日志框架。

目前，它已经过时了，但值得讨论，因为它为更现代的继任者奠定了基础。

许多配置细节与Log4j 2部分中讨论的细节一致。

### 5.1 配置

首先，你需要将Log4j库添加到项目的pom.xml中：

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

在[这里](https://mvnrepository.com/artifact/log4j/log4j)能够找到最新版本的Log4j。

让我们看一个仅具有一个控制台附加器的简单Log4j配置的完整示例：

```xml
<!DOCTYPE log4j:configuration SYSTEM "log4j.dtd" >
<log4j:configuration debug="false">

    <!--Console appender-->
    <appender name="stdout" class="org.apache.log4j.ConsoleAppender">
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern"
                   value="%d{yyyy-MM-dd HH:mm:ss} %p %m%n" />
        </layout>
    </appender>

    <root>
        <level value="INFO" />
        <appender-ref ref="stdout" />
    </root>

</log4j:configuration>
```

<log4j:configuration debug="false"\>是整个配置的开始标签，它有一个属性debug，决定是否要将Log4j调试信息添加到日志中。

### 5.2 使用

添加Log4j库和配置后，你可以在代码中使用该记录器，让我们看一个简单的示例：

```java
import org.apache.log4j.Logger;

public class Log4jExample {
    private static Logger logger = Logger.getLogger(Log4jExample.class);

    public static void main(String[] args) {
        logger.debug("Debug log message");
        logger.info("Info log message");
        logger.error("Error log message");
    }
}
```

## 6. 总结

本文通过一些非常简单的示例展示了如何使用不同的日志记录框架(例如Log4j、Log4j2和 Logback)，它涵盖了上述所有框架的简单配置示例。