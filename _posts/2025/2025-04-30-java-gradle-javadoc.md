---
layout: post
title:  使用Gradle生成Javadoc
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

众所周知，**创建清晰全面的文档对于代码维护至关重要**。在Java中，实现这一点的一种方法是使用[Javadoc](https://www.baeldung.com/javadoc)，这是一个文档生成器，它可以将Java源代码注释创建为HTML文件。

在本教程中，我们将学习如何使用流行的构建自动化工具[Gradle](https://www.baeldung.com/gradle)生成Javadoc。

## 2. 设置Gradle项目

简而言之，设置[Gradle项目](https://www.baeldung.com/gradle-building-a-java-app)非常简单。首先，我们需要在机器上[安装](https://docs.gradle.org/current/userguide/installation.html)Gradle构建工具。接下来，创建一个空文件夹，并通过终端切换到该文件夹。然后，通过终端初始化一个新的Gradle项目：

```shell
$ gradle init
```

该命令会询问我们一些问题来设置项目，我们将选择一个应用程序模板作为要生成的项目类型。接下来，我们将选择Java作为实现语言，并选择Groovy作为构建脚本。最后，我们将使用默认测试框架[JUnit 4](https://www.baeldung.com/junit)，并为项目命名。

**或者，我们也可以使用[IntelliJ IDEA](https://www.baeldung.com/intellij-basics)生成Gradle项目**。为此，我们创建一个新项目并选择Gradle作为构建系统，它会自动生成包含所有必需文件夹的项目。

## 3. 项目设置

初始化Gradle项目后，让我们用我们最喜欢的IDE打开它。接下来，我们将创建一个名为“addition”的新包，并添加一个名为Sum的类：

```java
package cn.tuyucheng.taketoday.addition;

/**
 * This is a sample class that demonstrates Javadoc comments.
 */
public class Sum {
    /**
     * This method returns the sum of two integers.
     *
     * @param a the first integer
     * @param b the second integer
     * @return the sum of a and b
     */
    public int add(int a, int b) {
        return a + b;
    }
}
```

这个类演示了简单的加法功能，我们创建一个add()方法，它接收两个参数并返回参数的和。

此外，我们添加了介绍性文档[注释](https://www.baeldung.com/javadoc-see-vs-link)，并在[注释](https://www.baeldung.com/javadoc-multi-line-code)中描述了add()方法，我们指定了它所接受的参数以及它返回的值。

接下来，让我们创建另一个名为“subtraction”的包，并添加一个名为Difference的类：

```java
package cn.tuyucheng.taketoday.subtraction;

/**
 * This is a sample class that demonstrates Javadoc comments.
 */
public class Difference {
    /**
     * This method returns the difference between the two integers.
     *
     * @param a the first integer
     * @param b the second integer
     * @return the difference between a and b
     */
    public int subtract(int a, int b) {
        return a - b;
    }
}
```

此类演示了一种获取两个Integer之间的差的简单方法。

在下一节中，我们将学习如何通过指定要包含和排除的包来生成Javadoc。

## 4. 使用Gradle生成Javadoc

现在我们有了一个包含文档注释的示例项目，我们想通过Gradle生成Javadoc。为此，我们需要在gradle.build文件中添加一些配置。此文件包含项目的配置，例如插件、依赖、项目组、版本等。

首先，让我们将Java插件应用到项目中：

```groovy
plugins {
    id 'java'
}
```

这告诉Gradle使用Java插件，**Java插件使Java应用程序的开发变得更加容易，并提供编译、代码测试、Javadoc任务等功能**。

此外，我们将向gradle.build文件中添加Javadoc任务的代码：

```groovy
javadoc {
    destinationDir = file("${buildDir}/docs/javadoc")
}
```

这将配置Javadoc任务并指定用于存储生成的文档的构建目录。

我们还可以配置Javadoc任务以在运行任务时包含和排除包：

```groovy
javadoc {
    destinationDir = file("${buildDir}/docs/javadoc")
    include 'cn/tuyucheng/taketoday/addition/**'
    exclude 'cn/tuyucheng/taketoday/subtraction/**'
}
```

这里，我们包含addition包，并排除subtraction包，**include和exclude属性允许我们在Javadoc任务中选择所需的包**。

最后，为了生成文档，让我们打开终端并切换到根文件夹。然后，运行Gradle构建命令：

```shell
./gradlew javadoc
```

此命令执行Javadoc任务并生成HTML格式的文档，HTML文件存储在指定的文件夹中。

以下是HTML文件文档的示例：

![](/assets/images/2025/gradle/javagradlejavadoc01.png)

## 5. 总结

在本文中，我们学习了如何使用Gradle构建系统生成Javadoc。此外，我们还学习了如何为两个Java类编写文档注释。此外，我们还学习了如何配置Javadoc以包含和排除包。