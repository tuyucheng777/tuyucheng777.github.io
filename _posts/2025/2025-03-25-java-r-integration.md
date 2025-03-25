---
layout: post
title:  Java-R集成
category: libraries
copyright: libraries
excerpt: Java/R
---

## 1. 概述

[R](https://www.r-project.org/)是一种流行的统计学编程语言，由于它有各种各样的函数和包，将R代码嵌入到其他语言中并不罕见。

在本文中，我们将介绍将R代码集成到Java的一些最常见方法。

## 2. R脚本

对于我们的项目，我们将首先实现一个非常简单的R函数，该函数以向量作为输入并返回其值的平均值。我们将在专用文件中定义它：

```text
customMean <- function(vector) {
    mean(vector)
}
```

在本教程中，我们将使用Java辅助方法来读取此文件并将其内容作为字符串返回：

```java
String getMeanScriptContent() throws IOException, URISyntaxException {
    URI rScriptUri = RUtils.class.getClassLoader().getResource("script.R").toURI();
    Path inputScript = Paths.get(rScriptUri);
    return Files.lines(inputScript).collect(Collectors.joining());
}
```

现在，让我们看一下从Java调用此函数的不同选项。

## 3. RCaller

我们要考虑的第一个库是[RCaller](https://github.com/jbytecode/rcaller)，它可以通过在本地机器上生成专用的R进程来执行代码。

RCaller可从[Maven Central](https://mvnrepository.com/artifact/com.github.jbytecode/RCaller)获得，我们可以将其包含在我们的pom.xml中：

```xml
<dependency>
    <groupId>com.github.jbytecode</groupId>
    <artifactId>RCaller</artifactId>
    <version>3.0</version>
</dependency>
```

接下来，让我们使用原始R脚本编写一个自定义方法，返回值的平均值：

```java
public double mean(int[] values) throws IOException, URISyntaxException {
    String fileContent = RUtils.getMeanScriptContent();
    RCode code = RCode.create();
    code.addRCode(fileContent);
    code.addIntArray("input", values);
    code.addRCode("result <- customMean(input)");
    RCaller caller = RCaller.create(code, RCallerOptions.create());
    caller.runAndReturnResult("result");
    return caller.getParser().getAsDoubleArray("result")[0];
}
```

在这个方法中我们主要使用两个对象：

- RCode：代表我们的代码上下文，包括我们的函数、函数输入和调用语句
- RCaller：它允许我们运行代码并返回结果

值得注意的是，**RCaller不适合小规模和频繁计算**，因为启动R进程需要时间，这是一个明显的缺点。

此外，**RCaller仅适用于本地机器上安装的R**。

## 4. Renjin

[Renjin](https://www.renjin.org/)是R集成领域中另一种流行的解决方案，**它被更广泛地采用，并且还提供企业支持**。

将Renjin添加到我们的项目中并不那么简单，因为我们必须添加[Mulesoft](https://repository.mulesoft.org/nexus/content/repositories/public/)仓库以及Maven依赖：

```xml
<repositories>
    <repository>
        <id>mulesoft</id>
        <name>Mulesoft Repository</name>
        <url>https://repository.mulesoft.org/nexus/content/repositories/public/</url>
    </repository>
</repositories>

<dependencies>
    <dependency>
        <groupId>org.renjin</groupId>
        <artifactId>renjin-script-engine</artifactId>
        <version>RELEASE</version>
    </dependency>
</dependencies>
```

再次，让我们为R函数构建一个Java包装器：

```java
public double mean(int[] values) throws IOException, URISyntaxException, ScriptException {
    RenjinScriptEngine engine = new RenjinScriptEngine();
    String meanScriptContent = RUtils.getMeanScriptContent();
    engine.put("input", values);
    engine.eval(meanScriptContent);
    DoubleArrayVector result = (DoubleArrayVector) engine.eval("customMean(input)");
    return result.asReal();
}
```

我们可以看到，**这个概念与RCaller非常相似，但不那么冗长**，因为我们可以使用eval方法直接通过名称调用函数。

**Renjin的主要优势在于它使用基于JVM的解释器，因此无需安装R。不过，Renjin目前与GNUR并不完全兼容**。

## 5. Rserve

到目前为止，我们讨论过的库都是在本地运行代码的不错选择。但是，如果我们想让多个客户端调用我们的R脚本，该怎么办？这就是[Rserve](https://www.rforge.net/Rserve/index.html)发挥作用的地方，**它允许我们通过TCP服务器在远程机器上运行R代码**。

设置Rserve包括安装相关包并通过R控制台启动加载我们脚本的服务器：

```text
> install.packages("Rserve")
...
> library("Rserve")
> Rserve(args = "--RS-source ~/script.R")
Starting Rserve...
```

接下来，我们现在可以像往常一样通过添加[Maven依赖](https://mvnrepository.com/artifact/org.rosuda.REngine/Rserve)将Rserve包含在我们的项目中：

```xml
<dependency>
    <groupId>org.rosuda.REngine</groupId>
    <artifactId>Rserve</artifactId>
    <version>1.8.1</version>
</dependency>
```

最后，让我们将R脚本包装成Java方法。这里我们将使用带有服务器地址的RConnection对象，如果未提供，则默认为127.0.0.1:6311：

```java
public double mean(int[] values) throws REngineException, REXPMismatchException {
    RConnection c = new RConnection();
    c.assign("input", values);
    return c.eval("customMean(input)").asDouble();
}
```

## 6. FastR

我们要讨论的最后一个库是[FastR](https://github.com/oracle/fastr)，一个基于[GraalVM](https://www.graalvm.org/)构建的高性能R实现。在撰写本文时，**FastR仅在Linux和Darwin x64系统上可用**。

为了使用它，我们首先需要从官方网站安装GraalVM。之后，我们需要使用Graal Component Updater安装FastR本身，然后运行随附的配置脚本：

```shell
$ bin/gu install R
...
$ languages/R/bin/configure_fastr
```

这次我们的代码将依赖于[Polyglot](https://www.graalvm.org/reference-manual/polyglot-programming/)，这是GraalVM内部API，用于在Java中嵌入不同的客户端语言。由于Polyglot是一个通用API，因此我们指定要运行的代码的语言。此外，我们将使用cR函数将输入转换为向量：

```java
public double mean(int[] values) {
    Context polyglot = Context.newBuilder().allowAllAccess(true).build();
    String meanScriptContent = RUtils.getMeanScriptContent(); 
    polyglot.eval("R", meanScriptContent);
    Value rBindings = polyglot.getBindings("R");
    Value rInput = rBindings.getMember("c").execute(values);
    return rBindings.getMember("customMean").execute(rInput).asDouble();
}
```

**遵循此方法时，请记住它会使我们的代码与JVM紧密耦合**。要了解有关GraalVM的更多信息，请查看我们关于[Graal Java JIT编译器](https://www.baeldung.com/graal-java-jit-compiler)的文章。

## 7. 总结

在本文中，我们介绍了一些在Java中集成R的最流行技术。总结一下：

- RCaller更易于集成，因为它可以在Maven Central上使用
- Renjin提供企业支持，不需要在本地机器上安装R，但它与GNUR并不100%兼容
- Rserve可用于在远程服务器上执行R代码
- FastR可以与Java无缝集成，但会使我们的代码依赖于VM，并且不适用于所有操作系统