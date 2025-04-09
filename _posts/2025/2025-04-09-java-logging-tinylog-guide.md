---
layout: post
title:  使用Tinylog 2进行轻量级日志记录
category: log
copyright: log
excerpt: Tinylog
---

## 1. 概述

tinylog是一个适用于Java应用程序和Android应用程序的轻量级日志记录框架，在本教程中，我们将了解如何使用tinylog发布日志条目以及如何配置日志记录框架。我们将看到，tinylog的一些方法与[Log4j](https://www.baeldung.com/log4j2-appenders-layouts-filters)和[Logback](https://www.baeldung.com/logback)有很大不同。

## 2. 开始

首先，我们需要将[tinylog API](https://mvnrepository.com/artifact/org.tinylog/tinylog-api)和[tinylog实现](https://mvnrepository.com/artifact/org.tinylog/tinylog-impl)的依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.tinylog</groupId>
    <artifactId>tinylog-api</artifactId>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>org.tinylog</groupId>
    <artifactId>tinylog-impl</artifactId>
    <version>2.5.0</version>
</dependency>
```

现在，让我们创建一个新的Java应用程序并添加一条日志语句：

```java
package cn.tuyucheng.taketoday.tinylog;

import org.tinylog.Logger;

public class Application {

    public static void main(String[] args) {
        Logger.info("Hello World!");
    }
}
```

**与其他日志记录框架不同，tinylog有一个静态记录器类**。因此，我们不必为每个要使用日志记录的类创建一个记录器实例，这为我们节省了一些样板代码。尽管如此，tinylog可以输出日志消息以及源代码位置信息。

例如，如果我们在没有任何配置的情况下执行我们的应用程序，则日志消息将默认输出到控制台：

```text
2023-01-01 14:17:42 [main] cn.tuyucheng.taketoday.tinylog.Application.main()
INFO: Hello World!
```

## 3. 日志记录

通常，我们希望记录的不仅仅是静态文本，这与上一个示例不同。tinylog针对不同用例提供了各种日志记录方法，**每种日志记录方法都适用于所有5种受支持的严重性级别：trace()、debug()、info()、warn()和error()**。为了便于阅读，我们在示例中始终只使用其中一种。

### 3.1 占位符

我们可以使用带有占位符的文本在运行时组装日志消息：

```java
Logger.info("Hello {}!", "Alice"); // Hello Alice!
```

记录数字时，我们可以按照自己想要的方式格式化它们。例如，让我们输出带有两位小数的π：

```java
Logger.info("π = {0.00}", Math.PI); // π = 3.14
```

tinylog使用Lambda支持延迟日志记录，如果将日志消息或占位符参数作为Lambda传递，则只有在启用严重性级别时才会解析它。**如果已知在生产中禁用了严重性级别，则使用Lambda的延迟日志记录可以显著提高性能**：

```java
Logger.debug("Expensive computation: {}", () -> compute());
```

### 3.2 异常

在tinylog中，**异常始终是发出日志条目时的第一个参数，以防止异常与占位符参数混合**：

```java
try {
    int i = a / b;
} catch (Exception ex) {
    Logger.error(ex, "Cannot divide {} by {}", a, b);
}
```

尽管如此，在没有任何明确日志消息的情况下记录异常也是完全合法的。在这种情况下，tinylog会自动使用异常中的消息：

```java
try {
    int i = a / b;
} catch (Exception ex) {
    Logger.error(ex);
}
```

## 4. 配置

配置tinylog的推荐方法是使用属性文件，启动时，tinylog会自动在默认包中查找tinylog.properties并从此文件加载配置。tinylog.properties的正确位置通常是src/main/resources，但是，这可能取决于所使用的构建工具。

### 4.1 控制台

在Java应用程序中，tinylog默认将日志条目输出到控制台。但是，**我们也可以明确配置控制台写入器并自定义输出**。

让我们为所有严重级别为info及以上的日志条目启用控制台写入器，这意味着不会输出trace和debug日志条目：

```properties
writer       = console
writer.level = info
```

正如我们之前所见，tinylog默认将所有日志条目输出为两行。因此，让我们定义一个可以在单行中输出日志条目的自定义格式模式：

```properties
writer        = console
writer.format = {date: HH:mm:ss.SSS} [{level}] {class}: {message}
```

{class}占位符输出完全限定名称，{level}占位符输出日志条目的严重性级别，{date}占位符输出日期和/或时间，{message}占位符输出日志消息(如果存在则输出异常)。

**对于日期和时间的格式化，tinylog使用[SimpleDateFormat](https://www.baeldung.com/java-simple-date-format)或[DateTimeFormatter](https://www.baeldung.com/java-datetimeformatter)，具体取决于Java版本**。因此，我们可以使用这些Java类支持的所有日期和时间模式。

当我们执行之前的应用程序时，我们将在控制台上看到日志输出：

```text
14:17:42.452 [INFO] cn.tuyucheng.taketoday.tinylog.Application: Hello World!
```

### 4.2 日志文件

tinylog有4种[不同的文件写入器](https://tinylog.org/v2/configuration/#writers)，实际上，**实际应用通常使用滚动文件写入器**。因此，本教程重点介绍此写入器。

滚动文件写入器可以在定义的事件发生时启动新的日志文件；例如，让我们配置一个写入器，在启动时以及每天早上6点启动一个新的日志文件：

```properties
writer          = rolling file
writer.file     = logs/myapp_{date: yyyy-MM-dd}_{count}.log
writer.policies = startup, daily: 06:00
writer.format   = {class} [{thread}] {level}: {message}
```

**对于日志文件路径，我们应该使用占位符来避免覆盖现有的日志文件**。在上面的配置中，我们使用一个占位符表示日期，另一个占位符表示连续的计数数字。例如，如果我们在2023年1月1日多次启动应用程序，则第一个日志文件将是logs/myapp_2023-01-01_0.log，第二个将是logs/myapp_2023-01-01_1.log，依此类推。

策略是定义应启动新日志文件的事件，日志条目的格式模式可以自由配置，方式与控制台写入器相同。

**为了节省文件系统的空间，我们还可以压缩日志文件并限制要存储的日志文件的数量**：

```properties
writer         = rolling file
writer.file    = logs/myapp_{date: yyyy-MM-dd}_{count}.log
writer.convert = gzip
writer.backups = 100
```

启用gzip压缩后，当前日志文件仍然是未压缩的纯文本文件。但是，只要tinylog启动新的日志文件，日志记录框架就会使用gzip压缩以前的日志文件，并添加.gz作为文件扩展名。如果有超过100个压缩备份文件，tinylog会开始删除最旧的文件，以确保现有文件数量不超过100个。

### 4.3 Logcat

**在Android应用中，tinylog默认通过Logcat输出日志条目**。但是，我们也可以明确配置Logcat写入器并自定义输出。

首先，让我们创建一个发出日志条目的主要活动，我们可以将其用作未来配置的基础：

```java
package cn.tuyucheng.taketoday.tinylog;

import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;
import org.tinylog.Logger;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        Logger.info("Hello World!");
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

现在，我们可以启用Logcat写入器，例如，针对所有严重级别为debug及以上的日志条目，这意味着不会输出trace日志条目：

```properties
writer       = logcat
writer.level = debug
```

**tinylog可以自动计算标签，在使用Android的Log类时，我们通常必须将其作为第一个参数传递**。默认情况下，日志记录框架使用不带包名的简单类名作为标签，例如，tinylog使用MainActivity作为类cn.tuyucheng.taketoday.tinylog.MainActivity中发布的所有日志条目的标签。

但是，标签计算是可以自由配置的。例如，如果我们使用带标签的记录器而不是静态记录器，我们也可以使用tinylog标签作为Logcat标签：

```java
public class MainActivity extends AppCompatActivity {

    private final TaggedLogger logger = Logger.tag("UI");

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        logger.info("Hello World!");
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
}
```

现在，我们可以重新配置我们的Logcat写入器以使用标记记录器的UI标签：

```properties
writer         = logcat
writer.tagname = {tag}
```

### 4.4 多写入器

**我们可以在tinylog中同时使用多个写入器，配置多个写入器的一种方法是连续编号**：

```properties
writer1       = console
writer1.level = debug

writer2       = rolling file
writer2.file  = myapp_{count}.log
```

但是，我们也可以给写入器起一个有意义的名字：

```properties
writerConsole       = console
writerConsole.level = debug

writerFile          = rolling file
writerFile.file     = myapp_{count}.log
```

tinylog可识别以写入器开头的每个属性名称，但是，每个写入器都必须具有唯一的属性名称，以符合Java属性文件标准。

### 4.5 异步输出

为了避免在输出日志条目时阻塞应用程序，**tinylog可以异步输出日志条目，在使用基于文件的写入器时尤其推荐这样做**。

我们只需要在tinylog配置中启用写入线程即可受益于异步输出：

```properties
writingthread = true
```

通过此配置，tinylog使用单独的线程来输出日志条目。

此外，我们还可以**通过启用缓冲输出来进一步提高滚动文件写入器的性能**。默认情况下，基于文件的写入器将每个日志条目分别写入日志文件。但是，使用缓冲输出，tinylog可以在单个IO操作中将多个日志条目输出到日志文件：

```properties
writer          = rolling file
writer.file     = myapp_{count}.log
writer.buffered = true
```

## 5. 总结

在本文中，我们讨论了tinylog 2的基本日志记录方法和配置参数，我们重点介绍了实际应用中最常用的功能。此外，tinylog还提供更多功能，可以在[官方文档](https://tinylog.org/v2/documentation/)中找到。