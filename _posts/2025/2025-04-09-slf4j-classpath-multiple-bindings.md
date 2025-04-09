---
layout: post
title:  SLF4J警告：类路径包含多个SLF4J绑定
category: log
copyright: log
excerpt: SLF4J
---

## 1. 概述

当我们在应用程序中使用SLF4J时，我们有时会看到打印到控制台的有关类路径中的多个绑定的警告消息。

在本教程中，我们将尝试了解为什么会看到此消息以及如何解决它。

## 2. 理解警告

首先，让我们看一个示例警告：

```text
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:.../slf4j-log4j12-1.7.21.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:.../logback-classic-1.1.7.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
```

这个警告告诉我们SLF4J发现了两个绑定，一个在slf4j-log4j12-1.7.21.jar中，另一个在logback-classic-1.1.7.jar中。

现在让我们了解为什么会看到这个警告。

Java的简单日志门面[(SLF4J)](https://www.baeldung.com/slf4j-with-log4j2-logback)是各种[日志框架](https://www.baeldung.com/java-logging-intro)的简单门面或抽象，它允许我们在部署时插入所需的日志框架。

为了实现这一点，SLF4J在类路径上寻找绑定(又称提供程序)。绑定基本上是特定SLF4J类的实现，旨在扩展以插入特定的日志记录框架。

根据设计，SLF4J每次仅与一个日志框架绑定。因此，**如果类路径上存在多个绑定，它将发出警告**。

值得注意的是，嵌入式组件(例如库或框架)永远不应声明对任何SLF4J绑定的依赖。这是因为当库声明对SLF4J绑定的编译时依赖时，它会将该绑定强加给最终用户。显然，这否定了SLF4J的基本目的；因此，它们应该只依赖于slf4j-api库。

还要注意的是，这只是一个警告。如果SLF4J发现多个绑定，它将从列表中选择一个日志框架并与其绑定。从警告的最后一行可以看出，SLF4J已选择Log4j，并使用org.slf4j.impl.Log4jLoggerFactory进行实际绑定。

## 3. 查找冲突的JAR

警告列出了它找到的所有绑定的位置，通常，这些信息足以识别将不需要的SLF4J绑定引入我们项目的问题依赖。

如果无法从警告中识别依赖，我们可以使用dependency:tree maven目标：

```shell
mvn dependency:tree
```

这将显示项目的依赖树：

```text
[INFO] +- org.docx4j:docx4j:jar:3.3.5:compile 
[INFO] |  +- org.slf4j:slf4j-log4j12:jar:1.7.21:compile 
[INFO] |  +- log4j:log4j:jar:1.2.17:compile 
[INFO] +- ch.qos.logback:logback-classic:jar:1.1.7:compile 
[INFO] +- ch.qos.logback:logback-core:jar:1.1.7:compile
```

我们在应用程序中使用Logback进行日志记录，因此，我们特意添加了logback-classic JAR中的Logback绑定；但docx4j依赖还引入了与slf4j-log4j12 JAR的另一个绑定。

## 4. 解决方案

现在我们知道了有问题的依赖，我们只需要从docx4j依赖中排除slf4j-log4j12 JAR：

```xml
<dependency>
    <groupId>org.docx4j</groupId>
    <artifactId>docx4j</artifactId>
    <version>${docx4j.version}</version>
    <exclusions>
        <exclusion>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
        </exclusion>
        <exclusion>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

因为我们不会使用Log4j，所以将其排除可能也是个好主意。

## 5. 总结

在本文中，我们了解了如何解决SLF4J发出的有关多重绑定的常见警告。