---
layout: post
title:  滚动文件附加器指南
category: log
copyright: log
excerpt: Log4j
---

## 1. 概述

虽然日志文件通常包含有用的信息，但随着时间的推移，它们自然会变得越来越大。如果任其无限增长，其大小可能会成为一个问题。

日志库使用滚动文件附加程序解决了这个问题，**它自动“滚动”或存档当前日志文件，并在某些预定义条件发生时恢复在新文件中的日志记录**，从而防止不必要的停机。

在本教程中，我们将学习如何在一些最广泛使用的日志库中配置滚动文件附加器：Log4j、Log4j2和Slf4j。

我们将演示如何根据大小、日期/时间以及大小和日期/时间的组合滚动日志文件。我们还将探讨如何配置每个库以自动压缩并在之后删除旧日志文件，从而免去编写繁琐的日常管理代码的麻烦。

## 2. 示例应用程序

让我们从一个记录一些消息的示例应用程序开始，此代码基于Log4j，但我们可以轻松修改它以与Log4j2或Slf4j一起使用：

```java
import org.apache.log4j.Logger;

public class Log4jRollingExample {

    private static Logger logger = Logger.getLogger(Log4jRollingExample.class);

    public static void main(String[] args) throws InterruptedException {
        for(int i = 0; i < 2000; i++) {
            logger.info("This is the " + i + " time I say 'Hello World'.");
            Thread.sleep(100);
        }
    }
}
```

该应用程序非常简单；它在一个循环中写入一些消息，循环之间有短暂的延迟。运行2000个循环，每个循环暂停100毫秒，应用程序应该需要三分钟多一点的时间才能完成。

我们将使用此示例来演示不同类型的滚动文件附加器的几种特性。

## 3. Log4j中的滚动文件附加器

### 3.1 Maven依赖

首先，为了在我们的应用程序中使用Log4j，我们将此依赖添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

对于我们将在下一个示例中使用的apache-log-extras提供的附加附加程序，我们将添加以下依赖，确保使用与我们为Log4j声明的相同版本，以确保完全兼容：

```xml
<dependency>
    <groupId>log4j</groupId>
    <artifactId>apache-log4j-extras</artifactId>
    <version>1.2.17</version>
</dependency>
```

可以在Maven Central上找到[Log4j](https://mvnrepository.com/artifact/log4j/log4j)和[Apache Log4j Extras](https://mvnrepository.com/artifact/log4j/apache-log4j-extras)的最新版本。

### 3.2 根据文件大小滚动

在Log4j中，与其他日志库一样，文件滚动由附加器负责，让我们看一下Log4j中根据文件大小滚动文件附加器的配置：

```xml
<appender name="roll-by-size" class="org.apache.log4j.RollingFileAppender">
    <param name="file" value="target/log4j/roll-by-size/app.log" />
    <param name="MaxFileSize" value="5KB" />
    <param name="MaxBackupIndex" value="2" />
        <layout class="org.apache.log4j.PatternLayout">
            <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n" />
        </layout>
</appender>
```

这里我们使用MaxFileSize参数配置Log4j在日志文件大小达到5KB时滚动日志文件，我们还使用MaxBackupIndex参数指示Log4j最多保留两个滚动日志文件。

当运行示例应用程序时，我们获得以下文件：

```text
27/11/2016  10:28    138 app.log
27/11/2016  10:28  5.281 app.log.1
27/11/2016  10:28  5.281 app.log.2
```

那么发生了什么？Log4j开始写入app.log文件，当文件大小超过5KB限制时，Log4j将app.log移至app.log.1，创建一个新的空app.log，并继续将新的日志消息写入app.log。

然后，当新的app.log超过5KB限制后，重复此滚动过程。这一次，app.log.1被移动到app.log.2，为另一个新的空app.log腾出空间。

此滚动过程在运行期间重复了几次，但由于我们将附加器配置为最多保留两个滚动文件，因此没有名为app.log.3的文件。

因此，我们解决了原来的一个问题，因为现在我们可以对生成的日志文件的大小设置限制。

当我们检查app.log.2的第一行时，它包含与第700次迭代相关的消息，这意味着所有先前的日志消息都丢失了：

```text
2016-11-27 10:28:34 INFO  This is the 700 time I say 'Hello World'.
```

现在让我们看看是否可以设计一个更适合生产环境的设置，在生产环境中丢失日志消息不被认为是最好的方法。

为此，我们将使用其他更强大、更灵活、更可配置的Log4j附加程序，这些附加程序包含在专用包中，称为apache-log4j-extras。

此工件中包含的附加程序提供了许多选项来微调日志滚动，它们引入了触发策略和滚动策略的不同概念。触发策略描述了何时应该发生滚动，而滚动策略描述了应该如何进行滚动。这两个概念是滚动日志文件的关键，其他库也或多或少明确地使用它们。

### 3.3 自动压缩滚动

让我们回到Log4j示例并通过添加滚动文件的自动压缩来改进我们的设置以节省空间：

```xml
<appender name="roll-by-size" class="org.apache.log4j.rolling.RollingFileAppender">
    <rollingPolicy class="org.apache.log4j.rolling.FixedWindowRollingPolicy">
        <param name="ActiveFileName" value="target/log4j/roll-by-size/app.log" />
        <param name="FileNamePattern" value="target/log4j/roll-by-size/app.%i.log.gz" />
        <param name="MinIndex" value="7" />
        <param name="MaxIndex" value="17" />
    </rollingPolicy>
    <triggeringPolicy class="org.apache.log4j.rolling.SizeBasedTriggeringPolicy">
        <param name="MaxFileSize" value="5120" />
    </triggeringPolicy>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p %m%n" />
    </layout>
</appender>
```

通过triggeringPolicy元素，我们规定当日志超过5120字节大小时应进行滚动。

在triggeringPolicy标签中，ActiveFileName参数指定包含最新消息的主日志文件的路径，FileNamePattern参数指定一个模板，该模板描述滚动文件的路径应该是什么。请注意，这确实是一个模式，因为特殊占位符%i将被替换为滚动文件的索引。

我们还要注意，FileNamePattern以“.gz”扩展名结尾，每当我们使用与受支持的压缩格式关联的扩展名时，我们都会压缩旧的滚动文件，而无需我们付出任何额外的努力。

现在，当我们运行应用程序时，我们会获得一组不同的日志文件：

```text
03/12/2016 19:24 88 app.1.log.gz
...
03/12/2016 19:26 88 app.2.log.gz
03/12/2016 19:26 88 app.3.log.gz
03/12/2016 19:27 70 app.current.log
```

文件app.current.log是最后日志发生的位置，当先前日志的大小达到设定的限制时，它们将被滚动并压缩。

### 3.4 根据日期和时间滚动

在其他情况下，我们可能希望将Log4j配置为根据日志消息的日期和时间(而不是文件大小)滚动文件。例如，在Web应用程序中，我们可能希望将一天内发出的所有日志消息放在同一个日志文件中。

为此，我们可以使用TimeBasedRollingPolicy。使用此策略，必须为包含时间相关占位符的日志文件路径指定一个模板。每次发出日志消息时，附加器都会验证生成的日志路径是什么。如果它与上次使用的路径不同，则会发生滚动。以下是配置此类附加器的简单示例：

```xml
<appender name="roll-by-time"
          class="org.apache.log4j.rolling.RollingFileAppender">
    <rollingPolicy class="org.apache.log4j.rolling.TimeBasedRollingPolicy">
        <param name="FileNamePattern" value="target/log4j/roll-by-time/app.%d{HH-mm}.log.gz" />
    </rollingPolicy>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p - %m%n" />
    </layout>
</appender>
```

### 3.5 根据大小和时间进行滚动

结合SizeBasedTriggeringPolicy和TimeBasedRollingPolicy，我们可以得到一个基于日期/时间滚动的附加器，并且当文件大小达到设置的限制时，它也会根据大小滚动：

```xml
<appender name="roll-by-time-and-size" class="org.apache.log4j.rolling.RollingFileAppender">
    <rollingPolicy class="org.apache.log4j.rolling.TimeBasedRollingPolicy">
        <param name="ActiveFileName" value="log4j/roll-by-time-and-size/app.log" />
        <param name="FileNamePattern" value="log4j/roll-by-time-and-size/app.%d{HH-mm}.%i.log.gz" />
    </rollingPolicy>
    <triggeringPolicy
            class="org.apache.log4j.rolling.SizeBasedTriggeringPolicy">
        <param name="MaxFileSize" value="100" />
    </triggeringPolicy>
    <layout class="org.apache.log4j.PatternLayout">
        <param name="ConversionPattern" value="%d{yyyy-MM-dd HH:mm:ss} %-5p - %m%n" />
    </layout>
</appender>
```

当我们使用此设置运行应用程序时，我们将获得以下日志文件：

```text
03/12/2016 19:25 234 app.19-25.1481393432120.log.gz
03/12/2016 19:25 234 app.19-25.1481393438939.log.gz
03/12/2016 19:26 244 app.19-26.1481393441940.log.gz
03/12/2016 19:26 240 app.19-26.1481393449152.log.gz
03/12/2016 19:26 3.528 app.19-26.1481393470902.log
```

app.19-26.1481393470902.log文件是当前日志记录的位置；我们可以看到，19:25到19:26之间的所有日志都存储在多个压缩日志文件中，文件名以“app.19-25”开头；“%i”占位符被一个不断增加的数字所取代。

## 4. Log4j2中的滚动文件附加器

### 4.1 Maven依赖

要使用Log4j2作为我们的首选日志库，我们需要使用以下依赖更新项目的POM：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.7</version>
</dependency>
```

和往常一样，我们可以在Maven Central上找到[最新版本](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core)。

### 4.2 根据文件大小滚动

让我们将示例应用程序更改为使用Log4j2日志库，我们将根据log4j2.xml配置文件中的日志文件大小设置文件滚动：

```xml
<RollingFile
        name="roll-by-size"
        fileName="target/log4j2/roll-by-size/app.log"
        filePattern="target/log4j2/roll-by-size/app.%i.log.gz"
        ignoreExceptions="false">
    <PatternLayout>
        <Pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</Pattern>
    </PatternLayout>
    <Policies>
        <OnStartupTriggeringPolicy />
        <SizeBasedTriggeringPolicy size="5 KB" />
    </Policies>
</RollingFile>
```

在Policies标签中，我们指定了要应用的所有触发策略。OnStartupTriggeringPolicy会在应用程序每次启动时触发滚动，这对于独立应用程序非常有用。我们还指定了SizeBasedTriggeringPolicy，规定只要日志文件达到5KB就应发生滚动。

### 4.3 根据日期和时间滚动

使用Log4j2提供的策略，我们将设置一个附加器来根据时间滚动和压缩日志文件：

```xml
<RollingFile name="roll-by-time"
             fileName="target/log4j2/roll-by-time/app.log"
             filePattern="target/log4j2/roll-by-time/app.%d{MM-dd-yyyy-HH-mm}.log.gz"
             ignoreExceptions="false">
    <PatternLayout>
        <Pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</Pattern>
    </PatternLayout>
    <TimeBasedTriggeringPolicy />
</RollingFile>
```

这里的关键是TimeBasedTriggeringPolicy，它允许我们在滚动文件名模板中使用与时间相关的占位符。请注意，由于我们只需要一个触发策略，因此我们不必像上一个示例那样使用Policies标签。

### 4.4 根据大小和时间进行滚动

如前所述，更引人注目的场景是根据时间和大小滚动和压缩日志文件，以下是我们如何为此任务设置Log4j2的示例：

```xml
<RollingFile name="roll-by-time-and-size"
             fileName="target/log4j2/roll-by-time-and-size/app.log"
             filePattern="target/log4j2/roll-by-time-and-size/app.%d{MM-dd-yyyy-HH-mm}.%i.log.gz"
             ignoreExceptions="false">
    <PatternLayout>
        <Pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</Pattern>
    </PatternLayout>
    <Policies>
        <OnStartupTriggeringPolicy />
        <SizeBasedTriggeringPolicy size="5 KB" />
        <TimeBasedTriggeringPolicy />
    </Policies>
    <DefaultRolloverStrategy>
        <Delete basePath="${baseDir}" maxDepth="2">
            <IfFileName glob="target/log4j2/roll-by-time-and-size/app.*.log.gz" />
            <IfLastModified age="20d" />
        </Delete>
    </DefaultRolloverStrategy>
</RollingFile>
```

通过此配置，我们声明应根据时间和大小进行滚动。由于文件名使用的模式是“app.%d{MM-dd-yyyy-HH-mm}.%i.log.gz”，因此附加器能够理解我们所指的时间间隔，该模式隐式地将滚动设置为每分钟发生一次并压缩滚动的文件。

我们还添加了DefaultRolloverStrategy来删除符合特定条件的旧滚动文件，我们将其配置为删除符合给定模式的文件(如果文件超过20天)。

### 4.5 Maven依赖

要使用Log4j2作为我们的首选日志库，我们需要使用以下依赖更新项目的POM：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.7</version>
</dependency>
```

和往常一样，可以在Maven Central上找到[最新版本](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core)。

## 5. Slf4j中的滚动文件附加器

### 5.1 Maven依赖

当我们想使用带有Logback后端的Slf4j2作为我们的日志库时，我们将此依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

和往常一样，可以在Maven Central上找到[最新版本](https://mvnrepository.com/artifact/ch.qos.logback/logback-classic)。

### 5.2 根据文件大小滚动

现在让我们看看如何使用Slf4j及其默认后端Logback，我们将在配置文件logback.xml中设置文件滚动，该文件位于应用程序的类路径中：

```xml
<appender name="roll-by-size" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>target/slf4j/roll-by-size/app.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.FixedWindowRollingPolicy">
        <fileNamePattern>target/slf4j/roll-by-size/app.%i.log.zip</fileNamePattern>
        <minIndex>1</minIndex>
        <maxIndex>3</maxIndex>
        <totalSizeCap>1MB</totalSizeCap>
    </rollingPolicy>
    <triggeringPolicy class="ch.qos.logback.core.rolling.SizeBasedTriggeringPolicy">
        <maxFileSize>5KB</maxFileSize>
    </triggeringPolicy>
    <encoder>
        <pattern>%-4relative [%thread] %-5level %logger{35} - %msg%n</pattern>
    </encoder>
</appender>
```

滚动策略基本机制与Log4j和Log4j2使用的机制相同，FixedWindowRollingPolicy允许我们在滚动文件的名称模式中使用索引占位符。

当日志文件的大小超出配置的限制时，将分配一个新文件。旧内容将存储为列表中的第一个文件，将现有文件向前移动一个位置。

### 5.3 根据时间滚动

在Slf4j中，我们可以使用提供的TimeBasedRollingPolicy根据时间滚动日志文件，此策略允许我们使用与时间和日期相关的占位符指定滚动文件的模板名称：

```xml
<appender name="roll-by-time" class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>target/slf4j/roll-by-time/app.log</file>
    <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
        <fileNamePattern>target/slf4j/roll-by-time/app.%d{yyyy-MM-dd-HH-mm}.log.zip
        </fileNamePattern>
        <maxHistory>20</maxHistory>
        <totalSizeCap>1MB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</pattern>
    </encoder>
</appender>
```

### 5.4 根据大小和时间进行滚动

如果我们需要根据时间和大小滚动文件，我们可以使用提供的SizeAndTimeBasedRollingPolicy。使用此策略时，我们必须同时指定与时间相关的占位符和索引占位符。

每当一定时间间隔内的日志文件大小超出配置的大小限制时，就会创建另一个具有与时间相关的占位符相同的值但索引递增的日志文件：

```xml
<appender name="roll-by-time-and-size"
          class="ch.qos.logback.core.rolling.RollingFileAppender">
    <file>target/slf4j/roll-by-time-and-size/app.log</file>
    <rollingPolicy
            class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
        <fileNamePattern>
            target/slf4j/roll-by-time-and-size/app.%d{yyyy-MM-dd-mm}.%i.log.zip
        </fileNamePattern>
        <maxFileSize>5KB</maxFileSize>
        <maxHistory>20</maxHistory>
        <totalSizeCap>1MB</totalSizeCap>
    </rollingPolicy>
    <encoder>
        <pattern>%d{yyyy-MM-dd HH:mm:ss} %p %m%n</pattern>
    </encoder>
</appender>
```

## 6. 总结

在本文中，我们了解到利用日志库滚动文件可以节省我们手动管理日志文件的负担。因此，我们可以专注于业务逻辑的开发。