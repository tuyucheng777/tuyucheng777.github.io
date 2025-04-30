---
layout: post
title:  在Gradle中传递命令行参数
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

有时，我们想从[Gradle](https://www.baeldung.com/gradle)执行各种需要输入参数的程序。

在本快速教程中，我们将了解如何从Gradle传递命令行参数。

## 2. 输入参数的类型

当我们想从Gradle CLI传递输入参数时，我们有两个选择：

- 使用-D标志设置系统属性
- 使用-P标志设置项目属性

一般来说，**我们应该使用项目属性，除非我们想在JVM中自定义设置**。

尽管可以劫持系统属性来传递我们的输入，但我们应该避免这样做。

让我们看看这些属性的作用，首先，我们配置build.gradle：

```groovy
apply plugin: "java"
description = "Gradle Command Line Arguments examples"

task propertyTypes(){
    doLast{
        if (project.hasProperty("args")) {
            println "Our input argument with project property ["+project.getProperty("args")+"]"
        }
        println "Our input argument with system property ["+System.getProperty("args")+"]"
    }
}

```

请注意，我们在任务中对它们的读取有所不同。

我们这样做是因为**如果属性未定义，project.getProperty()会抛出MissingPropertyException**。

与项目属性不同，如果未定义属性，System.getProperty()将返回空值。

接下来，让我们运行任务并查看其输出：

```shell
$ ./gradlew propertyTypes -Dargs=lorem -Pargs=ipsum

> Task :cmd-line-args:propertyTypes
Our input argument with project property [ipsum]
Our input argument with system property [lorem]
```

## 3. 传递命令行参数

到目前为止，我们只了解了如何读取属性，实际上，我们需要将这些属性作为参数传递给我们选择的程序。

### 3.1 向Java应用程序传递参数

在上一篇教程中，我们讲解了如何[从Gradle运行Java主类](https://www.baeldung.com/gradle-run-java-main)，在此基础上，我们来看一下如何传递参数。

首先，让我们**在build.gradle中使用application插件**：

```groovy
apply plugin: "java"
apply plugin: "application"
description = "Gradle Command Line Arguments examples"
 
// previous declarations
 
ext.javaMainClass = "cn.tuyucheng.taketoday.cmd.MainClass"
 
application {
    mainClassName = javaMainClass
}
```

现在，让我们看一下主类：

```java
public class MainClass {
    public static void main(String[] args) {
        System.out.println("Gradle command line arguments example");
        for (String arg : args) {
            System.out.println("Got argument [" + arg + "]");
        }
    }
}
```

接下来，让我们用一些参数来运行它：

```shell
$ ./gradlew :cmd-line-args:run --args="lorem ipsum dolor"

> Task :cmd-line-args:run
Gradle command line arguments example
Got argument [lorem]
Got argument [ipsum]
Got argument [dolor]
```

这里我们不使用属性来传递参数，而是**传递–args标志和相应的输入**。

这是application插件提供的一个很好的包装器，但是，**此功能仅在Gradle 4.9及更高版本中可用**。

让我们看看使用JavaExec任务会是什么样子。

首先，我们需要在build.gradle中定义它：

```groovy
ext.javaMainClass = "cn.tuyucheng.taketoday.cmd.MainClass"
if (project.hasProperty("args")) {
    ext.cmdargs = project.getProperty("args")
} else { 
    ext.cmdargs = ""
}
task cmdLineJavaExec(type: JavaExec) {
    group = "Execution"
    description = "Run the main class with JavaExecTask"
    classpath = sourceSets.main.runtimeClasspath
    main = javaMainClass
    args cmdargs.split()
}
```

我们首先从项目属性中读取参数，由于它包含所有参数作为一个字符串，因此我们使用split方法来获取一个参数数组。

接下来，**我们将这个数组传递给JavaExec任务的args属性**。

让我们看看当我们运行此任务时会发生什么，并使用-P选项传递项目属性：

```shell
$ ./gradlew cmdLineJavaExec -Pargs="lorem ipsum dolor"

> Task :cmd-line-args:cmdLineJavaExec
Gradle command line arguments example
Got argument [lorem]
Got argument [ipsum]
Got argument [dolor]
```

### 3.2 将参数传递给其他应用程序

在某些情况下，我们可能希望将一些参数从Gradle传递给第三方应用程序。

幸运的是，我们可以使用更通用的Exec任务来执行此操作：

```groovy
if (project.hasProperty("args")) {
    ext.cmdargs = project.getProperty("args")
} else { 
    ext.cmdargs = "ls"
}
 
task cmdLineExec(type: Exec) {
    group = "Execution"
    description = "Run an external program with ExecTask"
    commandLine cmdargs.split()
}
```

在这里，**我们使用任务的commandLine属性来传递可执行文件及其参数**。同样，我们根据空格来拆分输入。

让我们看看如何运行ls命令：

```shell
$ ./gradlew cmdLineExec -Pargs="ls -ll"

> Task :cmd-line-args:cmdLineExec
total 4
drwxr-xr-x 1 user 1049089    0 Sep  1 17:59 bin
drwxr-xr-x 1 user 1049089    0 Sep  1 18:30 build
-rw-r--r-- 1 user 1049089 1016 Sep  3 15:32 build.gradle
drwxr-xr-x 1 user 1049089    0 Sep  1 17:52 src
```

如果我们不想在任务中对可执行文件进行硬编码，这将非常有用。

## 4. 总结

在本快速教程中，我们了解了如何从Gradle传递输入参数。

首先，我们解释了可以使用的属性类型。虽然我们可以使用系统属性来传递输入参数，但我们应该更倾向项目属性。

然后，我们探索了将命令行参数传递给Java或外部应用程序的不同方法。