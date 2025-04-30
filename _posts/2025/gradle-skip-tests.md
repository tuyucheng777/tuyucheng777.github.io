---
layout: post
title:  使用Gradle跳过测试
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

虽然跳过测试通常不是一个好主意，但在某些情况下它可能有用，并且可以节省一些时间。例如，假设我们正在开发一个新功能，并且希望在中间构建中看到结果。在这种情况下，我们可能会暂时跳过测试，以减少编译和运行它们的开销。**毫无疑问，忽略测试可能会导致许多严重的问题**。

在这个简短的教程中，我们将学习如何在使用[Gradle](https://www.baeldung.com/gradle)构建工具时跳过测试。

## 2. 使用命令行标志

首先，让我们创建一个我们想要跳过的简单测试：

```java
@Test
void skippableTest() {
    Assertions.assertTrue(true);
}
```

当我们运行构建命令时：

```shell
gradle build
```

我们将看到正在运行的任务：

```text
> ...
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses
> Task :test
> ...
```

**要跳过Gradle构建中的任何任务，我们可以使用-x或–exclude-task选项**。在本例中，**我们将使用“-x test”跳过构建中的测试**。

为了查看它的实际效果，让我们使用-x选项运行构建命令：

```shell
gradle build -x test
```

我们将看到正在运行的任务：

```text
> Task :compileJava NO-SOURCE 
> Task :processResources NO-SOURCE 
> Task :classes UP-TO-DATE 
> Task :jar 
> Task :assemble 
> Task :check 
> Task :build
```

结果，测试源未被编译，因此未被执行。

## 3. 使用Gradle构建脚本

使用Gradle构建脚本，我们提供了更多跳过测试的选项。例如，**我们可以根据某些条件跳过测试，或者仅在特定环境下使用onlyIf()方法跳过测试**。如果此方法返回false，则测试将被跳过。

让我们跳过基于检查项目属性的测试：

```shell
test.onlyIf { !project.hasProperty('someProperty') }
```

现在我们将运行构建命令，并将someProperty传递给Gradle：

```shell
gradle build -PsomeProperty
```

因此，Gradle跳过运行测试：

```text
> ...
> Task :compileTestJava 
> Task :processTestResources NO-SOURCE 
> Task :testClasses 
> Task :test SKIPPED 
> Task :check UP-TO-DATE 
> ...
```

此外，**我们可以使用build.gradle文件中的exclude属性根据包或类名排除测试**：

```shell
test {
    exclude 'org/boo/**'
    exclude '**/Bar.class'
}
```

**我们还可以根据正则表达式跳过测试**，例如，我们可以跳过所有类名以“Integration”结尾的测试：

```shell
test {
    exclude '**/**Integration'
}
```

## 4. 总结

在本文中，我们学习了如何在使用Gradle构建工具时跳过测试，我们还介绍了所有可以在命令行上使用的相关选项，以及可以在Gradle构建脚本中使用的选项。