---
layout: post
title:  Java中getResourceAsStream()和FileInputStream指南
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将探讨[Java中读取文件](https://www.baeldung.com/reading-file-in-java)的不同方法之间的差异。我们将比较getResourceAsStream()方法和FileInputStream类并讨论它们的用例，我们还将讨论Files.newInputStream()方法，由于其内存和性能优势，我们推荐使用该方法而不是FileInputStream。

然后我们将查看代码示例来了解如何使用这些方法读取文件。

## 2. 基础知识

在深入研究代码示例之前，让我们先了解getResourceAsStream()和FileInputStream之间的区别及其常见用例。

### 2.1 使用getResourceAsStream()读取文件

getResourceAsStream()方法从[classpath](https://www.baeldung.com/java-classpath-vs-build-path)读取文件，**传递给getResourceAsStream()方法的文件路径应相对于classpath**，该方法返回可用于读取文件的InputStream。

**此方法通常用于读取配置文件、属性文件以及与应用程序一起打包的其他资源**。

### 2.2 使用FileInputStream读取文件

另一方面，FileInputStream类用于从文件系统读取文件，当我们需要读取未与应用程序打包的文件时，这很有用。

**传递给FileInputStream构造函数的文件路径应该是绝对路径或相对于当前工作目录的路径**。

由于使用finalizer，FileInputStream对象可能会出现内存和性能问题。FileInputStream的一个更好的替代方案是使用Files.newInputStream()方法，它的工作方式相同，我们将在代码示例中使用Files.newInputStream()方法从文件系统读取文件。

**这些方法通常用于读取文件系统外部的文件，例如日志文件、用户数据文件和秘密文件**。

## 3. 代码示例

让我们看一个例子来演示getResourceAsStream()和Files.newInputStream()的用法。我们将创建一个简单的实用程序类，使用这两种方法读取文件，然后我们将通过从类路径和文件系统读取示例文件来测试这些方法。

### 3.1 使用getResourceAsStream()

首先，我们来看看getResourceAsStream()方法的用法。我们将创建一个名为FileIOUtil的类，并添加一个从资源中读取文件的方法：

```java
static String readFileFromResource(String filePath) {
    try (InputStream inputStream = FileIOUtil.class.getResourceAsStream(filePath)) {
        String result = null;
        if (inputStream != null) {
            result = new BufferedReader(new InputStreamReader(inputStream))
                    .lines()
                    .collect(Collectors.joining("\n"));
        }
        return result;
    } catch (IOException e) {
        LOG.error("Error reading file:", e);
        return null;
    }
}
```

在此方法中，我们通过将文件路径作为参数传递给getResourceAsStream()方法来获取InputStream，**此文件路径应相对于类路径**。然后，我们使用BufferedReader读取文件的内容。该方法逐行读取内容，并使用Collectors.joining()方法将它们拼接起来。最后，我们将文件的内容作为String返回。

如果发生异常(例如找不到文件)，我们会捕获异常并返回null。

### 3.2 使用Files.newInputStream()

接下来，让我们使用Files.newInputStream()方法定义一个类似的方法：

```java
static String readFileFromFileSystem(String filePath) {
    try (InputStream inputStream = Files.newInputStream(Paths.get(filePath))) {
        return new BufferedReader(new InputStreamReader(inputStream))
                .lines()
                .collect(Collectors.joining("\n"));
    } catch (IOException e) {
        LOG.error("Error reading file:", e);
        return null;
    }
}
```

在此方法中，我们使用Files.newInputStream()方法从文件系统读取文件，**文件路径应该是绝对路径或相对于项目目录的路径**。与上一种方法类似，我们读取并返回文件的内容。

## 4. 测试

现在，让我们通过读取示例文件来测试这两种方法，我们将观察在资源文件和外部文件的情况下文件路径如何传递给方法。

### 4.1 从Classpath读取文件

首先，我们来比较一下这些方法如何从类路径读取文件。我们在src/main/resources目录下创建一个名为test.txt的文件，并向其中添加一些内容：

```text
Hello!
Welcome to the world of Java NIO.
```

我们将使用这两种方法读取此文件并验证其内容：

```java
@Test
void givenFileUnderResources_whenReadFileFromResource_thenSuccess() {
    String result = FileIOUtil.readFileFromResource("/test.txt");
    assertNotNull(result);
    assertEquals(result, "Hello!\n" + "Welcome to the world of Java NIO.");
}

@Test
void givenFileUnderResources_whenReadFileFromFileSystem_thenSuccess() {
    String result = FileIOUtil.readFileFromFileSystem("src/test/resources/test.txt");
    assertNotNull(result);
    assertEquals(result, "Hello!\n" + "Welcome to the world of Java NIO.");
}
```

我们可以看到，这两个方法都读取了文件test.txt并返回其内容，然后我们比较内容以确保它们与预期值匹配，**这两个方法的区别在于我们作为参数传递的文件路径**。

readFileFromResource()方法需要相对于类路径的路径，由于文件直接位于src/main/resources目录下，因此我们传递/test.txt作为文件路径。

另一方面，readFileFromFileSystem()方法需要绝对路径或相对于当前工作目录的路径，我们传递src/main/resources/test.txt作为文件路径。或者，我们可以传递文件的绝对路径，如/path/to/project/src/main/resources/test.txt。

### 4.2 从文件系统读取文件

接下来，让我们测试一下这些方法如何从文件系统读取文件。我们将在项目目录之外创建一个名为external.txt的文件，并尝试使用这两种方法读取此文件。

让我们创建测试方法来使用这两种方法读取文件：

```java
@Test
void givenFileOutsideResources_whenReadFileFromFileSystem_thenSuccess() {
    String result = FileIOUtil.readFileFromFileSystem("../external.txt");
    assertNotNull(result);
    assertEquals(result, "Hello!\n" + "Welcome to the world of Java NIO.");
}

@Test
void givenFileOutsideResources_whenReadFileFromResource_thenNull() {
    String result = FileIOUtil.readFileFromResource("../external.txt");
    assertNull(result);
}
```

这里，我们传递了external.txt文件的相对路径，readFileFromFileSystem()方法直接从文件系统读取文件并返回其内容。

**如果我们尝试使用readFileFromResource()方法读取文件，它会返回null，因为该文件位于类路径之外**。

## 5. 总结

在本文中，我们探讨了使用getResourceAsStream()从类路径读取文件与使用Files.newInputStream()从文件系统读取文件之间的区别。我们讨论了这两种方法的用例和行为，并查看了演示其用法的示例。