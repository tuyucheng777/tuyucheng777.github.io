---
layout: post
title:  SLF4J简介
category: log
copyright: log
excerpt: SLF4J
---

## 1. 概述

Java的简单日志门面(简称SLF4J)充当不同日志框架(例如[java.util.logging、logback、Log4j](https://www.baeldung.com/java-logging-intro))的[门面](https://en.wikipedia.org/wiki/Facade_pattern)，它提供通用API，使日志记录独立于实际实现。

这允许不同的日志记录框架共存，它有助于从一个框架迁移到另一个框架。最后，除了标准化API之外，它还提供了一些“语法糖”。

本教程将讨论将SLF4J与Log4j、Logback、Log4j 2和Jakarta Commons Logging集成所需的依赖和配置。

有关每个实现的更多信息，请参阅我们的文章[Java日志简介](https://www.baeldung.com/java-logging-intro)。

## 2. Log4j 2设置

为了将SLF4J与Log4j 2一起使用，我们向pom.xml添加以下库：

```xml
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-api</artifactId>
    <version>2.23.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-core</artifactId>
    <version>2.23.1</version>
</dependency>
<dependency>
    <groupId>org.apache.logging.log4j</groupId>
    <artifactId>log4j-slf4j2-impl</artifactId>
    <version>2.23.1</version>
</dependency>
```

最新版本可以在这里找到：[log4j-api](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-api)、[log4j-core](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core)、[log4j-slf4j-impl](https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-slf4j2-impl)。

实际日志记录配置遵循原生Log4j2配置。

让我们看看如何创建Logger实例：

```java
public class Slf4jExample {

    private static Logger logger = LoggerFactory.getLogger(Slf4jExample.class);

    public static void main(String[] args) {
        logger.debug("Debug log message");
        logger.info("Info log message");
        logger.error("Error log message");
    }
}
```

请注意，Logger和LoggerFactory属于org.slf4j包。

[此处](https://github.com/eugenp/tutorials/tree/master/logging-modules/log4j)提供了使用此配置运行的项目示例。

## 3. Logback设置

我们不需要将SLF4J添加到类路径中即可将其与Logback一起使用，因为Logback已经在使用SLF4J，它是参考实现。

因此，我们只需要包含Logback库：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.6</version>
</dependency>
```

最新版本可以在这里找到：[logback-classic](https://mvnrepository.com/artifact/ch.qos.logback/logback-classic)。

该配置特定于Logback，但可与SLF4J无缝协作。有了适当的依赖和配置，我们可以使用前面部分中的相同代码来处理日志记录。

## 4. Log4j设置

在前面的部分中，我们介绍了一个用例，其中SLF4J“位于”特定日志实现之上。这样使用，它完全抽象了底层框架。

在某些情况下，我们无法替换现有的日志解决方案，例如由于第三方要求；但这并不限制项目仅限于已使用的框架。

我们可以将SLF4J配置为桥梁，并将对现有框架的调用重定向到它。

让我们添加必要的依赖来为Log4j创建桥梁：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.30</version>
</dependency>
```

有了依赖(检查[log4j-over-slf4j](https://mvnrepository.com/artifact/org.slf4j/log4j-over-slf4j)的最新版本)，所有对Log4j的调用都将被重定向到SLF4J。

查看[官方文档](http://www.slf4j.org/legacy.html)来了解有关桥接现有框架的更多信息。

和其他框架一样，Log4j可以作为底层实现。

让我们添加必要的依赖：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-log4j12</artifactId>
    <version>1.7.30</version>
</dependency>
<dependency>
    <groupId>log4j</groupId>
    <artifactId>log4j</artifactId>
    <version>1.2.17</version>
</dependency>
```

以下是[slf4j-log4j12](https://mvnrepository.com/artifact/org.slf4j/slf4j-log4j12)和[log4j](https://mvnrepository.com/artifact/log4j/log4j)的最新版本。[此处](https://github.com/eugenp/tutorials/tree/master/testing-modules/rest-assured)提供了以此方式配置的示例项目。

## 5. JCL桥接设置

在前面的部分中，我们展示了如何使用相同的代码库来支持使用不同实现的日志记录。虽然这是SLF4J的主要承诺和优势，但它也是JCL(Jakarta Commons Logging或Apache Commons Logging)背后的目标。

JCL旨在成为类似于SLF4J的框架，主要区别在于JCL在运行时通过类加载系统解析底层实现，在使用自定义类加载器的情况下，这种方法似乎存在问题。

SLF4J在编译时解析其绑定，它被认为更简单但功能足够强大。

幸运的是，两个框架可以在桥接模式下协同工作：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>jcl-over-slf4j</artifactId>
    <version>1.7.30</version>
</dependency>
```

最新的依赖版本可以在这里找到：[jcl-over-slf4j](https://mvnrepository.com/artifact/org.slf4j/jcl-over-slf4j)。

与其他情况一样，相同的代码库将运行良好。

## 6. SLF4J的其他功能

SLF4J提供的附加功能可以使日志记录更高效、代码更具可读性。

例如，SLF4J提供了一个非常有用的用于处理参数的接口：

```java
String variable = "Hello John";
logger.debug("Printing variable value: {}", variable);Lambda
```

以下是执行相同操作的Log4j代码：

```java
String variable = "Hello John";
logger.debug("Printing variable value: " + variable);
```

我们看到，无论是否启用调试级别，Log4j都会拼接字符串。在高负载应用程序中，这可能会导致性能问题。另一方面，SLF4J仅在启用调试级别时才会拼接字符串。

要对Log4J执行相同操作，我们需要添加一个额外的if块，它将检查调试级别是否启用：

```java
String variable = "Hello John";
if (logger.isDebugEnabled()) {
    logger.debug("Printing variable value: " + variable);
}
```

SLF4J标准化了日志级别，这些级别因具体实现而异。它放弃了FATAL日志级别(在Log4j中引入)，其依据是，在日志框架中，我们不应该决定何时终止应用程序。

使用的日志级别为ERROR、WARN、INFO、DEBUG和TRACE，有关如何使用它们的更多信息，请参阅我们的[Java日志简介](https://www.baeldung.com/java-logging-intro)。

## 7. 总结

SLF4J有助于在日志框架之间进行静默切换，它简单而灵活，并且可提高可读性和性能。