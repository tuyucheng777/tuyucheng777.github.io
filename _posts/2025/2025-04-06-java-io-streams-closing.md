---
layout: post
title:  关闭Java IO流
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在Java IO操作领域中，确保正确关闭IO流非常重要，这对于资源管理和代码健壮性至关重要。

在本教程中，我们将详细探讨为什么需要关闭IO流。

## 2. 当IO流未关闭时会发生什么？

在完成所有操作后立即显式关闭IO流始终是一种很好的做法，忽略关闭它们可能会导致各种问题。

在本节中，我们将讨论这些问题。

### 2.1 资源泄漏

每当我们打开一个IO流时，它总会占用一些系统资源，**直到调用IO流的close()方法时，这些资源才会被释放**。

某些IO流实现可以在其finalize()方法中自动关闭自身，每当触发[垃圾收集器](https://www.baeldung.com/jvm-garbage-collectors)(GC)时，都会调用finalize()方法。

但是，无法保证一定会调用GC，而且调用时间也不固定，有可能在调用GC之前资源就用完了。因此，**我们不应该仅仅依赖GC来回收系统资源**。

### 2.2 数据损坏

我们经常将[BufferedOutputStream](https://www.baeldung.com/java-outputstream#outputstream-buffering)包装在[OutputStream](https://www.baeldung.com/java-outputstream)周围，以提供缓冲功能，从而减少每次写入操作的开销。这是一种常见的做法，旨在提高写入数据的性能。

BufferedOutputStream中的内部缓冲区是用于临时存储数据的暂存区，每当缓冲区达到一定大小或调用flush()方法时，数据就会写入目标。

当我们将数据写入BufferedOutputStream后，最后一块数据可能尚未写入目标，从而导致数据损坏，**调用close()方法会调用flush()将剩余数据写入缓冲区**。

### 2.3 文件锁定

当我们使用FileOutputStream将数据写入文件时，某些操作系统(例如Windows)会将该文件保留在我们的应用程序中。**这样，在FileOutputStream关闭之前，其他应用程序无法写入甚至访问该文件**。

## 3. 关闭IO流

现在让我们看看关闭Java IO流的几种方法，这些方法有助于避免我们上面讨论的问题并确保正确的资源管理。

### 3.1 try-catch-finally

这是关闭IO流的传统方法，**我们在[finally](https://www.baeldung.com/java-exceptions#3-finally)块中关闭IO流。这确保无论操作是否成功，都会调用close()方法**：

```java
InputStream inputStream = null;
OutputStream outputStream = null;

try {
    inputStream = new BufferedInputStream(wrappedInputStream);
    outputStream = new BufferedOutputStream(wrappedOutputStream);
    // Stream operations...
}
finally {
    try {
        if (inputStream != null)
            inputStream.close();
    }
    catch (IOException ioe1) {
        log.error("Cannot close InputStream");
    }
    try {
        if (outputStream != null)
            outputStream.close();
    }
    catch (IOException ioe2) {
        log.error("Cannot close OutputStream");
    }
}
```

正如我们所演示的，close()方法也可能引发IOException。因此，在关闭IO流时，我们必须在finally块中放置另一个[try-catch](https://www.baeldung.com/java-exceptions#2-try-catch)块。**当我们必须处理大量IO流时，这个过程会变得繁琐**。

### 3.2 Apache Commons IO

[Apache Commons IO](https://www.baeldung.com/apache-commons-io)是一个多功能Java库，为IO操作提供实用类和方法。

要使用它，让我们在pom.xml中包含以下[依赖](https://mvnrepository.com/artifact/commons-io/commons-io)：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.15.1</version>
</dependency>
```

Apache Commons库简化了复杂的任务，例如在finally块中关闭IO流：

```java
InputStream inputStream = null;
OutputStream outputStream = null;

try {
    inputStream = new BufferedInputStream(wrappedInputStream);
    outputStream = new BufferedOutputStream(wrappedOutputStream);
    // Stream operations...
}
finally {
    IOUtils.closeQuietly(inputStream);
    IOUtils.closeQuietly(outputStream);
}
```

**IOUtils.closeQuietly()可以有效地关闭IO流，而无需进行空检查，也不需要处理关闭过程中发生的异常**。

除了IOUtils.closeQuietly()之外，**该库还提供了AutoCloseInputStream类来自动关闭包装的InputStream**：

```java
InputStream inputStream = AutoCloseInputStream.builder().setInputStream(wrappedInputStream).get();

byte[] buffer = new byte[256];
while (inputStream.read(buffer) != -1) {
    // Other operations...
}
```

上面的例子从InputStream读取数据，**AutoCloseInputStream在到达输入末尾时自动关闭InputStream**，这可以通过从InputStream中的read()方法获取-1来确定。在这种情况下，我们甚至不需要显式调用close()方法。

### 3.3 try-with-resources

try-with-resources块是在Java 7中引入的，它被认为是关闭IO流的首选方式。

**这种方法允许我们在try语句中定义资源**，资源是一个对象，使用完毕后必须关闭。

例如，实现AutoClosable接口的InputStream和OutputStream等类被用作资源，**它们将在try-catch块之后自动关闭，这样就无需在finally块中显式调用close()方法**：

```java
try (BufferedInputStream inputStream = new BufferedInputStream(wrappedInputStream);
    BufferedOutputStream outputStream = new BufferedOutputStream(wrappedOutputStream)) {
    // Stream operations...
}
```

Java 9中出现了进一步的改进，改进了try-with-resources语法；**我们可以在try-with-resources块之前声明资源变量，并直接在try语句中指定它们的变量名**：

```java
InputStream inputStream = new BufferedInputStream(wrappedInputStream);
OutputStream outputStream = new BufferedOutputStream(wrappedOutputStream);

try (inputStream; outputStream) {
    // Stream operations...
}
```

## 4. 总结

在本文中，我们研究了关闭IO流的各种策略，从在finally块中调用close()方法的传统方法到Apache Commons IO等库提供的更简化的方法以及try-with-resources的优雅方法。

通过各种技术，我们可以选择最适合我们的代码库的方法并确保顺畅且无错误的IO操作。