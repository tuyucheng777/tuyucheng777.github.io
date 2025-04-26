---
layout: post
title:  在Java中将JAR文件添加到Classpath的方法
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 简介

在编写Java项目时，我们经常依赖外部库来编写应用程序。

在本文中，我们将研究将这些库作为JAR添加到[类路径](https://www.baeldung.com/java-classpath-vs-build-path)上的不同方法。

## 2. 在命令行上使用-cp或-classpath

首先，如果我们从命令行启动程序，那么将JAR依赖项指定为命令的一部分是有意义的：

```shell
java -cp /path/to/jar/file.jar com.example.MyClass
```

这里，/path/to/jar/file.jar是JAR文件的路径，com.example.MyClass是要执行的类。

我们还可以添加多个jar：

```shell
java -cp "lib/jar1.jar:lib/jar2.jar" com.example.MyClass
```

## 3. 在命令行上使用CLASSPATH

在某些情况下，可能需要同一个JAR在同一台机器上运行多个Java程序。

在这种情况下，我们可以在macOS/Linux中设置CLASSPATH环境变量，而不是在每个命令中指定类路径：

```shell
export CLASSPATH=/path/to/jar/file.jar
```

以下是我们在Windows中执行相同操作的方法：

```shell
set CLASSPATH=C:\path\to\jar\file.jar
```

一旦我们设置了CLASSPATH，我们就可以运行我们的Java程序而无需指定–classpath选项。

**需要注意的是，如果我们像这样设置CLASSPATH，它只在该终端会话中有效，一旦终端关闭，设置就会丢失**。我们可以将Classpath添加为[环境变量](https://www.baeldung.com/java-home-on-windows-mac-os-x-linux)，使其永久生效。

## 4. 在MANIFEST.MF文件中指定Classpath

当我们创建一个独立的应用程序时，将所有依赖的JAR捆绑到一个应用程序JAR中会很有帮助。

为此，我们需要在JAR文件的[MANIFEST.MF](https://www.baeldung.com/java-jar-manifest)文件中包含类路径：

```properties
Manifest-Version: 1.0
Class-Path: lib/jar1.jar lib/jar2.jar
Main-Class: com.example.MainClass
```

然后我们在创建应用程序JAR时添加此清单：

```shell
jar cvfm app.jar MANIFEST.MF -C /path/to/classes .
```

然后我们可以运行它：

```shell
java -jar app.jar
```

这包括类路径上的lib/jar1.jar和lib/jar2.jar。

值得注意的是，Class-Path选项优先于CLASSPATH环境变量以及–classpath命令行选项。

## 5. 将JAR添加到lib/ext目录

**在Java安装目录的lib/ext目录中添加JAR文件是一种遗留机制，放置在其中的JAR文件会自动添加到类路径中**。但是，在大多数情况下，我们不推荐使用这种方法。

由于这些JAR是由外部类加载器加载的，因此它们优先于CLASSPATH环境变量中指定的JAR或–classpath或–cp选项中指定的目录。

## 6. 在Eclipse/Intellij IDE中添加JAR

我们可以使用流行的IDE(例如[Eclipse](https://www.baeldung.com/eclipse-sts-spring#Jar)或[IntelliJ](https://www.baeldung.com/intellij-basics#3-configuring-libraries))轻松地将JAR文件添加到项目的类路径中，这两个IDE都提供了用户友好的界面，可以在项目中包含外部库。

有关详细说明，我们可以参考IDE特定的文档，以获得最准确和最新的指导。

## 7. 使用构建工具

虽然上述方法适用于小型项目，但无论项目规模大小，都可以利用构建工具，[Maven](https://www.baeldung.com/maven)和[Gradle](https://www.baeldung.com/gradle)是用于此目的的常用构建工具。

假设我们的项目遵循标准的Maven项目结构，我们可以在项目中包含依赖及其传递依赖：

```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>example-library</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>
```

让我们运行命令：

```shell
mvn package
```

Maven将这些JAR包含在我们的类路径中。

我们还可以将构建工具与IDE结合使用，以进一步简化将JAR添加到类路径的过程。

## 8. 总结

使用外部库是Java开发中的一项基本任务。

添加JAR的方式取决于项目的复杂性和需求，对于快速测试或脚本，简单的命令行选项可能就足够了。对于大型项目，我们可能需要使用像Maven或Gradle这样强大的工具来管理项目依赖。