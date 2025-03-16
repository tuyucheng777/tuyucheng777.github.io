---
layout: post
title:  java.lang.ProcessBuilder API指南
category: java-os
copyright: java-os
excerpt: Java OS
---

## 1. 概述

[Process API](https://www.baeldung.com/java-process-api)提供了一种在Java中执行操作系统命令的强大方法，但是，它有几个选项可能会使其使用起来很麻烦。

在本教程中，**我们将了解Java如何使用ProcessBuilder API缓解这一问题**。

## 2. ProcessBuilder API

ProcessBuilder类提供创建和配置操作系统进程的方法，**每个ProcessBuilder实例允许我们管理一组进程属性**，然后我们可以使用这些给定的属性启动一个新的进程。

以下是我们可以使用此API的一些常见场景：

- 查找当前Java版本
- 为我们的环境设置自定义键值映射
- 更改Shell命令运行的工作目录
- 将输入和输出流重定向到自定义替换
- 继承当前JVM进程的两个流
- [从Java代码执行Shell命令](https://www.baeldung.com/run-shell-command-in-java)

我们将在后面的章节中查看每个内容的实际例子。

但在深入研究工作代码之前，让我们先看看这个API提供了什么样的功能。

### 2.1 方法摘要

在本节中，**我们将回顾一下并简要介绍一下ProcessBuilder类中最重要的方法**，这将有助于我们稍后深入研究一些真实的例子：

```java
ProcessBuilder(String... command)
```

要使用指定的操作系统程序和参数创建一个新的进程构建器，我们可以使用这个方便的构造函数。

```java
directory(File directory)
```

我们可以通过调用directory方法并传递一个File对象来覆盖当前进程的默认工作目录。**默认情况下，当前工作目录设置为user.dir系统属性返回的值**。

```java
environment()
```

如果我们想获取当前环境变量，只需调用environment方法即可。它使用System.getenv()返回当前进程环境的副本，但以Map形式返回。

```java
inheritIO()
```

如果我们想要指定子进程标准I/O的源和目标应该与当前Java进程的相同，我们可以使用inheritIO方法。

```java
redirectInput(File file), redirectOutput(File file), redirectError(File file)
```

当我们想要将进程生成器的标准输入、输出和错误目标重定向到文件时，我们可以使用这三种类似的重定向方法。

```java
start()
```

最后但同样重要的一点是，要使用我们配置的内容启动新进程，我们只需调用start()。

**我们应该注意，此类不是同步的**。例如，如果我们有多个线程同时访问ProcessBuilder实例，则必须在外部管理同步。

## 3. 示例

现在我们对ProcessBuilder API有了基本的了解，让我们逐步了解一些示例。

### 3.1 使用ProcessBuilder打印Java版本

**在第一个例子中，我们将运行带有一个参数的java命令来获取版本**。

```java
Process process = new ProcessBuilder("java", "-version").start();
```

首先，我们创建ProcessBuilder对象，并将命令和参数值传递给构造函数。接下来，我们使用start()方法启动进程以获取Process对象。

现在让我们看看如何处理输出：

```java
List<String> results = readOutput(process.getInputStream());

assertThat("Results should not be empty", results, is(not(empty())));
assertThat("Results should contain java version: ", results, hasItem(containsString("java version")));

int exitCode = process.waitFor();
assertEquals("No errors should be detected", 0, exitCode);
```

这里我们读取进程输出并验证内容是否符合预期。最后一步，我们使用process.waitFor()等待进程完成。

**一旦该进程完成，返回值将告诉我们该进程是否成功**。

需要记住的几个要点：

- 参数必须按正确顺序排列
- 此外，在此示例中，使用默认的工作目录和环境
- 我们故意在读取输出后才调用process.waitFor()，因为输出缓冲区可能会使进程停滞
- 我们假设java命令可通过PATH变量获得

### 3.2 启动已修改环境的进程

在下一个例子中，我们将看到如何修改工作环境。

**但在此之前，我们先来看看在默认环境中可以找到什么样的信息**：

```java
ProcessBuilder processBuilder = new ProcessBuilder();        
Map<String, String> environment = processBuilder.environment();
environment.forEach((key, value) -> System.out.println(key + value));
```

这只是打印出默认提供的每个变量条目：

```text
PATH/usr/bin:/bin:/usr/sbin:/sbin
SHELL/bin/bash
...
```

**现在我们将向我们的ProcessBuilder对象添加一个新的环境变量并运行一个命令来输出其值**：

```java
environment.put("GREETING", "Hola Mundo");

processBuilder.command("/bin/bash", "-c", "echo $GREETING");
Process process = processBuilder.start();
```

让我们分解这些步骤来了解我们所做的工作：

- 在我们的环境中添加一个名为“GREETING”的变量，其值为“Hola Mundo”，这是一个标准的Map<String, String\>
- 这次，我们没有使用构造函数，而是直接通过command(String... command)方法设置命令和参数
- 然后我们按照前面的例子开始我们的进程

为了完成示例，我们验证输出是否包含我们的问候语：

```java
List<String> results = readOutput(process.getInputStream());
assertThat("Results should not be empty", results, is(not(empty())));
assertThat("Results should contain java version: ", results, hasItem(containsString("Hola Mundo")));
```

### 3.3 使用修改后的工作目录启动进程

**有时更改工作目录会很有用**，在下一个示例中，我们将了解如何执行此操作：

```java
@Test
public void givenProcessBuilder_whenModifyWorkingDir_thenSuccess() throws IOException, InterruptedException {
    ProcessBuilder processBuilder = new ProcessBuilder("/bin/sh", "-c", "ls");

    processBuilder.directory(new File("src"));
    Process process = processBuilder.start();

    List<String> results = readOutput(process.getInputStream());
    assertThat("Results should not be empty", results, is(not(empty())));
    assertThat("Results should contain directory listing: ", results, contains("main", "test"));

    int exitCode = process.waitFor();
    assertEquals("No errors should be detected", 0, exitCode);
}
```

**在上面的例子中，我们使用便捷方法directory(File directory)将工作目录设置为项目的src目录**，然后我们运行一个简单的目录列表命令，并检查输出是否包含子目录main和test。

### 3.4 重定向标准输入和输出

**在现实世界中，我们可能希望将正在运行的进程的结果捕获到日志文件中，以便进一步分析**。幸运的是，ProcessBuilder API内置了对此的支持，正如我们将在此示例中看到的那样。

**默认情况下，我们的进程从管道读取输入，我们可以通过Process.getOutputStream()返回的输出流访问此管道**。

但是，我们很快就会看到，可以使用方法redirectOutput将标准输出重定向到另一个源(例如文件)。在这种情况下，getOutputStream()将返回ProcessBuilder.NullOutputStream。

让我们回到最初的例子来打印出Java的版本。但这次让我们将输出重定向到日志文件而不是标准输出管道：

```java
ProcessBuilder processBuilder = new ProcessBuilder("java", "-version");

processBuilder.redirectErrorStream(true);
File log = folder.newFile("java-version.log");
processBuilder.redirectOutput(log);

Process process = processBuilder.start();
```

在上面的例子中，**我们创建了一个名为log的新临时文件，并告诉我们的ProcessBuilder将输出重定向到此文件目标**。

在最后这段代码中，我们只需检查getInputStream()确实为空，以及文件的内容是否符合预期：

```java
assertEquals("If redirected, should be -1 ", -1, process.getInputStream().read());
List<String> lines = Files.lines(log.toPath()).collect(Collectors.toList());
assertThat("Results should contain java version: ", lines, hasItem(containsString("java version")));
```

**现在让我们看一下此示例的一个细微变化。例如，当我们希望将内容附加到日志文件而不是每次都创建一个新文件时**：

```java
File log = tempFolder.newFile("java-version-append.log");
processBuilder.redirectErrorStream(true);
processBuilder.redirectOutput(Redirect.appendTo(log));
```

**还需要提到对redirectErrorStream(true)的调用。如果发生任何错误，错误输出将合并到正常流程输出文件中**。

当然，我们可以为标准输出和标准错误输出指定单独的文件：

```java
File outputLog = tempFolder.newFile("standard-output.log");
File errorLog = tempFolder.newFile("error.log");

processBuilder.redirectOutput(Redirect.appendTo(outputLog));
processBuilder.redirectError(Redirect.appendTo(errorLog));
```

### 3.5 继承当前进程的I/O

在这个倒数第二个例子中，我们将看到inheritIO()方法的实际作用。**当我们想要将子进程I/O重定向到当前进程的标准I/O时，可以使用此方法**：

```java
@Test
public void givenProcessBuilder_whenInheritIO_thenSuccess() throws IOException, InterruptedException {
    ProcessBuilder processBuilder = new ProcessBuilder("/bin/sh", "-c", "echo hello");

    processBuilder.inheritIO();
    Process process = processBuilder.start();

    int exitCode = process.waitFor();
    assertEquals("No errors should be detected", 0, exitCode);
}
```

在上面的例子中，通过使用inheritIO()方法，我们可以在IDE的控制台中看到一个简单命令的输出。

在下一节中，我们将了解Java 9中ProcessBuilder API添加了哪些内容。

## 4. Java 9新增功能

**Java 9在ProcessBuilder API中引入了管道的概念**：

```java
public static List<Process> startPipeline(List<ProcessBuilder> builders)
```

使用startPipeline方法，我们可以传递一个ProcessBuilder对象列表。然后，此静态方法将为每个ProcessBuilder启动一个Process，从而创建一个由其标准输出和标准输入流链接的进程管道。

例如，如果我们要运行如下命令：

```shell
find . -name *.java -type f | wc -l
```

我们要做的是为每个独立命令创建一个流程构建器，并将它们组合成一个管道：

```java
@Test
public void givenProcessBuilder_whenStartingPipeline_thenSuccess() throws IOException, InterruptedException {
    List builders = Arrays.asList(
        new ProcessBuilder("find", "src", "-name", "*.java", "-type", "f"), 
        new ProcessBuilder("wc", "-l"));

    List processes = ProcessBuilder.startPipeline(builders);
    Process last = processes.get(processes.size() - 1);

    List output = readOutput(last.getInputStream());
    assertThat("Results should not be empty", output, is(not(empty())));
}
```

**在这个例子中，我们搜索src目录中的所有java文件，并将结果传送到另一个进程中进行计数**。

要了解Java 9中对Process API所做的其他改进，请查看我们关于[Java 9 Process API改进](https://www.baeldung.com/java-9-process-api)的文章。

## 5. 总结

总而言之，在本教程中，我们详细探讨了java.lang.ProcessBuilder API。

首先，我们首先解释了API可以做什么，并总结了最重要的方法。

接下来，我们看了一些实际示例。最后，我们了解了Java 9中API引入了哪些新功能。