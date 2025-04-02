---
layout: post
title:  使用Java将垃圾回收记录到文件中
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

垃圾回收是Java编程语言的一个奇迹，它为我们提供了自动内存管理。垃圾回收隐藏了必须手动分配和释放内存的细节，虽然这个机制很棒，但有时它并不能按照我们想要的方式工作。在本教程中，我们将探索**Java的垃圾回收统计日志记录选项**，并了解如何**将这些统计信息重定向到文件**。

## 2. Java 8及更早版本中的GC日志标志

首先，让我们探讨一下Java 9之前的Java版本中与GC日志记录相关的JVM标志。

### 2.1 -XX:+PrintGC

-XX:+PrintGC标志是-verbose:gc的别名，用于打开基本的GC日志记录。在此模式下，为每个年轻代和每个全代回收打印一行，现在让我们将注意力转向提供详细的GC信息。

### 2.2 -XX:+PrintGCDetails

类似地，我们有标志-XX:+PrintGCDetails用于激活详细的GC日志记录而不是-XX:+PrintGC。

请注意，-XX:+PrintGCDetails的输出会根据所使用的GC算法而变化。

接下来，我们将研究如何使用日期和时间信息注释日志。

### 2.3 -XX:+PrintGCDateStamps和-XX:+PrintGCTimeStamps

我们可以分别使用标志-XX:+PrintGCDateStamps和-XX:+PrintGCTimeStamps将日期和时间信息添加到我们的GC日志中。

首先，-XX:+PrintGCDateStamps将日志条目的日期和时间添加到每一行的开头。

其次，-XX:PrintGCTimeStamps为日志的每一行添加一个时间戳，详细说明自JVM启动以来经过的时间(以秒为单位)。

### 2.4 -Xloggc

最后，我们将GC日志重定向到文件。此标志使用语法-Xloggc:file将可选文件名作为参数，如果没有文件名，GC日志将写入标准输出。

此外，此标志还为我们设置了-XX:PrintGC和-XX:PrintGCTimestamps标志，让我们看一些例子：

如果我们想将GC日志写入标准输出，我们可以运行：

```shell
java -cp $CLASSPATH -Xloggc mypackage.MainClass
```

或者将GC日志写入文件，我们可以运行：

```shell
java -cp $CLASSPATH -Xloggc:/tmp/gc.log mypackage.MainClass
```

## 3. Java 9及更高版本中的GC日志记录标志

在Java 9+中，-XX:PrintGC(-verbose:gc的别名)已被弃用，取而代之的是**统一日志记录选项-Xlog**。上面提到的所有其他GC标志在Java 9+ 中仍然有效，这个新的日志记录选项允许我们**指定应显示哪些消息、设置日志级别以及重定向输出**。

我们可以运行以下命令来查看日志级别、日志装饰器和标签集的所有可用选项：

```shell
java -Xlog:logging=debug -version
```

例如，如果我们想将所有GC消息记录到一个文件中，我们可以运行：

```shell
java -cp $CLASSPATH -Xlog:gc*=debug:file=/tmp/gc.log mypackage.MainClass
```

此外，这个新的统一日志记录标志是可重复的，因此你可以**将所有GC消息记录到标准输出和文件中**：

```shell
java -cp $CLASSPATH -Xlog:gc*=debug:stdout -Xlog:gc*=debug:file=/tmp/gc.log mypackage.MainClass
```

## 4. 总结

在本文中，我们展示了如何在Java 8和Java 9+中记录垃圾回收输出，包括如何将该输出重定向到文件。