---
layout: post
title:  如何在Java中运行Shell命令
category: java-os
copyright: java-os
excerpt: Java OS
---

## 1. 概述

在本文中，我们将学习**如何从Java应用程序执行Shell命令**。

首先，我们将使用Runtime类提供的.exec()方法。然后，我们将了解可定制性更高的ProcessBuilder。

## 2. 操作系统依赖性

**Shell命令依赖于操作系统，因为它们的行为在不同系统上有所不同**。因此，在创建任何进程来运行Shell命令之前，我们需要了解JVM所运行的操作系统。

此外，在Windows上，Shell通常称为cmd.exe。而在Linux和macOS上，Shell命令使用/bin/sh运行。为了在这些不同的机器上兼容，如果在Windows机器上，我们可以以编程方式附加cmd.exe，否则附加 /bin/sh。例如，我们可以通过从System类中读取“os.name”属性来检查运行代码的机器是否是Windows机器：

```java
boolean isWindows = System.getProperty("os.name")
    .toLowerCase().startsWith("windows");
```

## 3. 输入和输出

通常，我们需要连接进程的输入和输出流。具体来说，InputStream充当进程的标准输入，OutputStream充当进程的标准输出。**我们必须始终使用输出流**，否则，我们的进程将不会返回并将永远挂起。

让我们实现一个常用的类，名为StreamGobbler，它使用一个InputStream：

```java
private static class StreamGobbler implements Runnable {
    private InputStream inputStream;
    private Consumer<String> consumer;

    public StreamGobbler(InputStream inputStream, Consumer<String> consumer) {
        this.inputStream = inputStream;
        this.consumer = consumer;
    }

    @Override
    public void run() {
        new BufferedReader(new InputStreamReader(inputStream)).lines()
                .forEach(consumer);
    }
}
```

此类实现了Runnable接口，这意味着任何[Executor](https://www.baeldung.com/java-executor-service-tutorial)都可以执行它。

## 4. Runtime.exec()

接下来，我们将使用.exec()方法生成一个新进程并使用之前创建的StreamGobbler。

例如，我们可以列出用户主目录内的所有目录，然后将其打印到控制台：

```java
Process process;
if (isWindows) {
    process = Runtime.getRuntime()
        .exec(String.format("cmd.exe /c dir %s", homeDirectory));
} else {
    process = Runtime.getRuntime()
        .exec(String.format("/bin/sh -c ls %s", homeDirectory));
}
StreamGobbler streamGobbler = 
  new StreamGobbler(process.getInputStream(), System.out::println);
Future<?> future = executorService.submit(streamGobbler);

int exitCode = process.waitFor();

assertDoesNotThrow(() -> future.get(10, TimeUnit.SECONDS));
assertEquals(0, exitCode);
```

在这里，我们使用.newSingleThreadExecutor()创建了一个新的子进程，然后使用.submit()运行包含Shell命令的进程。此外，.submit()返回一个[Future](https://www.baeldung.com/guava-futures-listenablefuture#1-future)对象，我们利用它来检查进程的结果。此外，请确保在返回的对象上调用.get()方法以等待计算完成。如果你从主方法运行上述代码，请确保在executorService对象上调用shutdown，否则代码将永远不会停止，这也适用于下面的所有示例。在我们的代码中，我们使用Junit生命周期方法来执行类似的必要清理。

**注意：JDK 18弃用了Runtime类中的.exec(String command)**。

### 4.1 处理管道

目前，没有办法用.exec()处理管道。幸运的是，管道是Shell的一个特性。因此，我们可以创建想要使用管道的整个命令并将其传递给.exec()：

```java
if (IS_WINDOWS) {
    process = Runtime.getRuntime()
        .exec(String.format("cmd.exe /c dir %s | findstr \"Desktop\"", homeDirectory));
} else {
    process = Runtime.getRuntime()
        .exec(String.format("/bin/sh -c ls %s | grep \"Desktop\"", homeDirectory));
}
```

在这里，我们列出用户主目录中的所有目录并搜索“Desktop”文件夹。

## 5. ProcessBuilder

**或者，我们可以使用[ProcessBuilder](https://www.baeldung.com/java-lang-processbuilder-api)，它比Runtime方法更受欢迎**，因为我们可以自定义它，而不仅仅是运行字符串命令。

简而言之，通过这种方法，我们能够：

- 使用.directory()更改Shell命令正在运行的工作目录
- 通过向.environment()提供键值Map来更改环境变量
- 以自定义方式重定向输入和输出流
- 使用.inheritIO()将它们都继承到当前JVM进程的流中

类似地，我们可以运行与上例相同的Shell命令：

```java
ProcessBuilder builder = new ProcessBuilder();
if (isWindows) {
    builder.command("cmd.exe", "/c", "dir");
} else {
    builder.command("sh", "-c", "ls");
}
builder.directory(new File(System.getProperty("user.home")));
Process process = builder.start();
StreamGobbler streamGobbler = 
  new StreamGobbler(process.getInputStream(), System.out::println);
Future<?> future = executorService.submit(streamGobbler);

int exitCode = process.waitFor();

assertDoesNotThrow(() -> future.get(10, TimeUnit.SECONDS));
assertEquals(0, exitCode); 
```

### 5.1 处理管道

Java 9在ProcessBuilder API中引入了管道的概念：

```java
public static List<Process> startPipeline(List<ProcessBuilder> builders) throws IOException
```

使用[startPipeline](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ProcessBuilder.html#startPipeline(java.util.List))方法，我们可以传递一个ProcessBuilder对象列表。然后，此静态方法将为每个ProcessBuilder启动一个Process。从而创建一个由其标准输出和标准输入流链接的进程管道。

例如，我们可以为每个隔离命令创建一个流程构建器，并将它们组合到管道中：

```java
@Test
public void givenProcessBuilder_whenStartingPipeline_thenSuccess() throws IOException, InterruptedException {
    List<ProcessBuilder> builders = Arrays.asList(
        new ProcessBuilder("find", "src", "-name", "*.java", "-type", "f"), 
        new ProcessBuilder("wc", "-l"));

    List<Process> processes = ProcessBuilder.startPipeline(builders);
    Process last = processes.get(processes.size() - 1);

    List<String> output = readOutput(last.getInputStream());
    assertThat("Results should not be empty", output, is(not(empty())));
}
```

在上面的例子中，我们搜索src目录中的所有java文件，并将结果传送到另一个进程中进行计数。

要了解Java 9中对Process API所做的其他改进，请查看我们关于[Java 9 Process API改进](https://www.baeldung.com/java-9-process-api)的文章。

## 6. 总结

正如我们在本快速教程中看到的，我们可以通过两种不同的方式在Java中执行Shell命令。

一般来说，如果我们计划自定义生成的进程的执行，例如，改变它的工作目录，我们应该考虑使用ProcessBuilder。