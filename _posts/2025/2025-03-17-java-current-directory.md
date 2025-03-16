---
layout: post
title:  在Java中获取当前工作目录
category: java-os
copyright: java-os
excerpt: Java OS
---

## 1. 概述

在Java中获取当前工作目录是一项简单的任务，但不幸的是，JDK中没有可用的直接API来执行此任务。

在本教程中，我们将学习如何使用java.lang.System、java.io.File、java.nio.file.FileSystems和java.nio.file.Paths获取Java中的当前工作目录。

## 2. System

让我们从使用System#getProperty的标准解决方案开始，假设我们当前的工作目录名称在整个代码中都是Taketoday：

```java
static final String CURRENT_DIR = "Taketoday";

@Test
void whenUsingSystemProperties_thenReturnCurrentDirectory() {
    String userDirectory = System.getProperty("user.dir");
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

我们使用Java内置属性键user.dir从System的属性Map中获取当前工作目录，此解决方案适用于所有JDK版本。

## 3. File

让我们看看使用java.io.File的另一种解决方案：

```java
@Test
void whenUsingJavaIoFile_thenReturnCurrentDirectory() {
    String userDirectory = new File("").getAbsolutePath();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

这里，File#getAbsolutePath内部使用System#getProperty来获取目录名称，与我们的第一个解决方案类似。这是一个获取当前工作目录的非标准解决方案，适用于所有JDK版本。

## 4. FileSystems

另一个有效的替代方法是使用新的java.nio.file.FileSystems API：

```java
@Test
void whenUsingJavaNioFileSystems_thenReturnCurrentDirectory() {
    String userDirectory = FileSystems.getDefault()
        .getPath("")
        .toAbsolutePath()
        .toString();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

该解决方案使用新的[Java NIO API](https://www.Taketoday.com/java-nio-2-file-api)，并且仅适用于JDK 7或更高版本。

## 5. Paths

最后，让我们看一个使用java.nio.file.Paths API获取当前目录的更简单的解决方案：

```java
@Test
void whenUsingJavaNioPaths_thenReturnCurrentDirectory() {
    String userDirectory = Paths.get("")
        .toAbsolutePath()
        .toString();
    assertTrue(userDirectory.endsWith(CURRENT_DIR));
}
```

这里，Paths#get内部使用FileSystem#getPath来获取路径，它使用新的[Java NIO API](https://www.Taketoday.com/java-nio-2-file-api)，因此此解决方案仅适用于JDK 7或更高版本。

## 6. 总结

在本教程中，我们探索了四种不同的方法来获取Java中的当前工作目录。前两种解决方案适用于所有版本的JDK，而后两种解决方案仅适用于JDK 7或更高版本。

我们建议使用System解决方案，因为它高效且直接，我们可以通过将这个API调用包装在静态实用程序方法中并直接访问它来简化它。