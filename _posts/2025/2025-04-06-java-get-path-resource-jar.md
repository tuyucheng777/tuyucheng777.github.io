---
layout: post
title:  获取Java JAR文件中资源的路径
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在Java中，通常使用相对于[JAR](https://www.baeldung.com/java-create-jar)文件根目录的路径来访问JAR文件中的[资源](https://www.baeldung.com/spring-classpath-file-access)。此外，了解这些路径的结构对于有效检索资源至关重要。

**在本教程中，我们将探索获取Java JAR文件中资源路径的不同方法**。

## 2. 使用Class.getResource()方法获取资源的URL

[Class.getResource()](https://www.baeldung.com/java-class-vs-classloader-getresource)方法提供了一种直接的方法来获取JAR文件中资源的URL，让我们看看如何使用此方法：

```java
@Test
public void givenFile_whenClassUsed_thenGetResourcePath() {
    URL resourceUrl = GetPathToResourceUnitTest.class.getResource("/myFile.txt");
    assertNotNull(resourceUrl);
}
```

在这个测试中，我们调用GetPathToResourceUnitTest.class上的getResource()方法，并将资源文件“/myFile.txt”的路径作为参数传递。然后，我们断言获取的resourceUrl不为空，这表明已成功找到资源文件。

## 3. 使用ClassLoader.getResource()方法

或者，我们可以使用[ClassLoader.getResource()](https://www.baeldung.com/java-classloaders)方法来访问JAR文件中的资源，当编译时不知道资源路径时，此方法很有用：

```java
@Test
public void givenFile_whenClassLoaderUsed_thenGetResourcePath() {
    URL resourceUrl = GetPathToResourceUnitTest.class.getClassLoader().getResource("myFile.txt");
    assertNotNull(resourceUrl);
}
```

在这个测试中，我们使用类加载器GetPathToResourceUnitTest.class.getClassLoader()来获取资源文件。与之前的方法不同，此方法不依赖于类的包结构。相反，它会在类路径的根级别搜索资源文件。

**这意味着它可以定位资源，而不管它们在项目结构中的位置，从而使得访问位于类包之外的资源更加灵活**。

## 4. 使用Paths.get()方法

从[Java 7](https://www.baeldung.com/java-check-is-installed)开始，我们可以使用[Paths.get()](https://www.baeldung.com/java-nio-2-path)方法获取表示JAR文件中资源的Path对象，操作方法如下：

```java
@Test
public void givenFile_whenPathUsed_thenGetResourcePath() throws Exception {
    Path resourcePath = Paths.get(Objects.requireNonNull(GetPathToResourceUnitTest.class.getResource("/myFile.txt")).toURI());
    assertNotNull(resourcePath);
}
```

这里，我们首先调用getResource()方法来获取资源文件的URL。然后，我们将此URL转换为URI，并将其传递给Paths.get()以获取表示资源文件位置的Path对象。

**当我们需要将资源文件作为Path对象使用时，这种方法非常有用。它支持更高级的文件操作，例如读取或写入文件内容。此外，它还提供了一种在[Java NIO](https://www.baeldung.com/java-io-vs-nio)包上下文中与资源交互的便捷方法**。

## 5. 总结

总之，访问Java JAR文件中的资源对于许多应用程序来说都至关重要。无论我们喜欢Class.getResource()的简单性、ClassLoader.getResource()的灵活性，还是使用Paths.get()的现代方法，Java都提供了各种方法来高效地完成此任务。