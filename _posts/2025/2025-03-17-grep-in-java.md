---
layout: post
title:  在Java中使用Grep进行模式搜索
category: java-os
copyright: java-os
excerpt: Java OS
---

## 1. 概述

在本教程中，我们将学习如何使用Java和第三方库(例如[Unix4J](https://github.com/tools4j/unix4j)和[Grep4J)](https://code.google.com/archive/p/grep4j/)在给定的文件中搜索模式。

## 2. 背景

Unix有一个强大的命令叫grep，它代表“全局正则表达式打印”，可以在给定的一组文件中搜索模式或正则表达式。

可以使用0个或多个选项以及grep命令来丰富搜索结果，我们将在下一节中详细讨论。

如果你使用的是Windows，则可以按照[此处](https://code.google.com/archive/p/grep4j/wikis/WindowSupport.wiki)的帖子所述安装bash。

## 3. 使用Unix4j库

首先，让我们看看如何使用Unix4J库在文件中查找模式。

在下面的例子中，我们将研究如何用Java翻译Unix grep命令。

### 3.1 构建配置

在你的pom.xml或build.gradle上添加以下依赖项：

```xml
<dependency>
    <groupId>org.unix4j</groupId>
    <artifactId>unix4j-command</artifactId>
    <version>0.4</version>
</dependency>
```

### 3.2 Grep示例

Unix中的grep示例：

```shell
grep "NINETEEN" dictionary.txt
```

Java中的对应代码为：

```java
@Test 
public void whenGrepWithSimpleString_thenCorrect() {
    int expectedLineCount = 4;
    File file = new File("dictionary.txt");
    List<Line> lines = Unix4j.grep("NINETEEN", file).toLineList(); 
    
    assertEquals(expectedLineCount, lines.size());
}
```

另一个例子是我们可以在文件中使用逆向文本搜索，以下是相同的Unix版本：

```shell
grep -v "NINETEEN" dictionary.txt
```

以下是上述命令的Java版本：

```java
@Test
public void whenInverseGrepWithSimpleString_thenCorrect() {
    int expectedLineCount = 178687;
    File file = new File("dictionary.txt");
    List<Line> lines = Unix4j.grep(Grep.Options.v, "NINETEEN", file). toLineList();
    
    assertEquals(expectedLineCount, lines.size()); 
}
```

让我们看看如何使用正则表达式在文件中搜索模式，以下是Unix版本，用于统计在整个文件中找到的所有正则表达式模式：

```shell
grep -c ".*?NINE.*?" dictionary.txt
```

以下是上述命令的Java版本：

```java
@Test
public void whenGrepWithRegex_thenCorrect() {
    int expectedLineCount = 151;
    File file = new File("dictionary.txt");
    String patternCount = Unix4j.grep(Grep.Options.c, ".*?NINE.*?", file).cut(CutOption.fields, ":", 1).toStringResult();
    
    assertEquals(expectedLineCount, patternCount); 
}
```

## 4. 使用Grep4J

接下来让我们看看如何使用Grep4J库来grep位于本地或远程位置的文件中模式。

在下面的例子中，我们将研究如何用Java翻译Unix grep命令。

### 4.1 构建配置

在你的pom.xml或build.gradle中添加以下依赖项：

```xml
<dependency>
    <groupId>com.googlecode.grep4j</groupId>
    <artifactId>grep4j</artifactId>
    <version>1.8.7</version>
</dependency>
```

### 4.2 Grep示例

Java中的grep示例相当于：

```bash
grep "NINETEEN" dictionary.txt
```

这是命令的Java版本：

```java
@Test 
public void givenLocalFile_whenGrepWithSimpleString_thenCorrect() {
    int expectedLineCount = 4;
    Profile localProfile = ProfileBuilder.newBuilder().
                           name("dictionary.txt").filePath(".").
                           onLocalhost().build();
    GrepResults results = Grep4j.grep(Grep4j.constantExpression("NINETEEN"), localProfile);
    
    assertEquals(expectedLineCount, results.totalLines());
}
```

另一个例子是我们可以在文件中使用逆向文本搜索，以下是相同的Unix版本：

```bash
grep -v "NINETEEN" dictionary.txt
```

Java版本如下：

```java
@Test
public void givenRemoteFile_whenInverseGrepWithSimpleString_thenCorrect() {
    int expectedLineCount = 178687;
    Profile remoteProfile = ProfileBuilder.newBuilder().
                            name("dictionary.txt").filePath(".").
                            filePath("/tmp/dictionary.txt").
                            onRemotehost("172.168.192.1").
                            credentials("user", "pass").build();
    GrepResults results = Grep4j.grep(Grep4j.constantExpression("NINETEEN"), remoteProfile, Option.invertMatch());
    
    assertEquals(expectedLineCount, results.totalLines()); 
}

```

让我们看看如何使用正则表达式在文件中搜索模式，以下是Unix版本，用于统计在整个文件中找到的所有正则表达式模式：

```shell
grep -c ".*?NINE.*?" dictionary.txt
```

这是Java版本：

```java
@Test
public void givenLocalFile_whenGrepWithRegex_thenCorrect() {
    int expectedLineCount = 151;
    Profile localProfile = ProfileBuilder.newBuilder().
                           name("dictionary.txt").filePath(".").
                           onLocalhost().build();
    GrepResults results = Grep4j.grep(Grep4j.regularExpression(".*?NINE.*?"), localProfile, Option.countMatches());
    
    assertEquals(expectedLineCount, results.totalLines()); 
}
```

## 5. 总结

在此快速教程中，我们演示了如何使用Grep4j和Unix4J在给定的文件中搜索模式。

最后，你也可以自然地使用JDK中的[正则表达式](https://www.baeldung.com/tag/regex/)功能执行一些类似grep的基本功能。