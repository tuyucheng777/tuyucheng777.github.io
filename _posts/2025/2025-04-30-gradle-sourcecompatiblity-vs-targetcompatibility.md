---
layout: post
title:  Gradle：sourceCompatibility与targetCompatibility
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本文中，我们将研究sourceCompatibility和targetCompatibility Java配置之间的区别以及它们在Gradle中的用法。

你可以查看我们的[Gradle简介](https://www.baeldung.com/gradle)文章来了解更多基础知识。

## 2. Java中的版本处理

当我们使用javac编译Java程序时，我们可以提供用于版本处理的编译选项，有两个可用选项：

- -source的值应与Java版本匹配，**最高可达我们用于编译的JDK版本**(例如，JDK8的版本号为1.8)，我们提供的版本值会将我们在源代码中可以使用的语言功能限制在相应的Java版本中。
- -target类似，但控制生成的类文件的版本，这意味着我们提供的版本值将是**程序可以运行的最低Java版本**。

例如：

```shell
javac HelloWorld.java -source 1.6 -target 1.8
```

这将生成一个需要Java 8或更高版本才能运行的类文件，此外，**源代码不能包含Lambda表达式或Java 6中不可用的任何功能**。

## 3. 使用Gradle处理版本

Gradle和Java插件允许我们使用java任务的sourceCompatibility和targetCompatibility配置来设置source和target选项，同样，**我们使用的值与javac相同**。

让我们设置build.gradle文件：

```groovy
plugins {
    id 'java'
}

group 'cn.tuyucheng.taketoday'

java {
    sourceCompatibility = "1.6"
    targetCompatibility = "1.8"
}
```

## 4. HelloWorldApp示例编译

我们可以创建一个Hello World控制台应用程序，并通过使用上述脚本构建它来演示其功能。

让我们创建一个非常简单的类：

```java
public class HelloWorldApp {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```

当我们使用gradle build命令构建它时，Gradle将生成一个名为HelloWorldApp.class的类文件。

我们可以使用Java自带的javap命令行工具来检查该类文件生成的字节码版本：

```shell
javap -verbose HelloWorldApp.class
```

这会打印很多信息，但在前几行，我们可以看到：

```text
public class cn.tuyucheng.taketoday.helloworld.HelloWorldApp
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
```

major version字段的值为52，这是Java 8类文件的版本号，这意味着我们的HelloWorldApp.class只能使用Java 8及更高版本运行。

为了测试sourceCompatibility配置，我们可以更改源代码并引入Java 6中没有的功能。

让我们使用Lambda表达式：

```java
public class HelloWorldApp {

    public static void main(String[] args) {
        Runnable helloLambda = () -> {
            System.out.println("Hello World!");
        };
        helloLambda.run();
    }
}
```

如果我们尝试使用Gradle构建代码，我们将看到编译错误：

```shell
error: lambda expressions are not supported in -source 1.6
```

-source选项是Java版的Gradle配置sourceCompatibility的等效选项，它可以阻止我们的代码编译。基本上，它可以防止我们错误地使用我们不想引入的高版本特性-例如，我们可能希望我们的应用也能在Java 6运行时上运行。

## 5. 总结

在本文中，我们解释了如何使用-source和-target编译选项来处理Java源代码和目标运行时的版本。此外，我们还学习了这些选项如何与Gradle的sourceCompatibility和targetCompatibility配置以及Java插件进行映射，并在实践中演示了它们的功能。