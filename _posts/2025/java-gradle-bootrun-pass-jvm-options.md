---
layout: post
title:  从Gradle bootRun传递JVM选项
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

[Gradle](https://www.baeldung.com/gradle)是一款多功能自动化构建工具，用于开发、编译和测试软件包。它支持多种语言，但我们主要将其用于基于[Java](https://www.baeldung.com/)的语言，例如[Kotlin](https://www.baeldung.com/kotlin/kotlin-overview)、[Groovy](https://www.baeldung.com/groovy-language)和[Scala](https://www.baeldung.com/scala/scala-intro)。

在使用Java时，我们可能需要自定义Java应用程序中的[JVM](https://www.baeldung.com/category/java/jvm)参数。由于我们使用Gradle构建Java应用程序，因此我们还可以通过调整Gradle配置来自定义应用程序的JVM参数。

在本教程中，我们将学习将JVM参数从Gradle bootRun传递到[Spring Boot](https://www.baeldung.com/spring-boot-start) Java应用程序。

## 2. 理解bootRun

**Gradle bootRun是默认[Spring Boot Gradle插件](https://www.baeldung.com/spring-boot-gradle-plugin)附带的Gradle指定任务，它帮助我们直接从Gradle运行Spring Boot应用程序**。执行bootRun命令会在开发环境中启动我们的应用程序，这对于测试和开发目的非常有用，它主要用于迭代开发，因为它不需要任何单独的构建或部署目的。

**简而言之，它提供了一种在开发环境中构建应用程序并执行与Spring Boot开发相关的任务的简化方法**。

## 3. 在build.gradle文件中使用jvmArgs

Gradle提供了一种直接的方法，使用[build.gradle](https://www.baeldung.com/gradle-build-settings-properties)文件将JVM参数添加到bootRun命令中。为了说明这一点，让我们看一下使用bootRun命令向Spring Boot应用程序添加JVM参数的命令：

```groovy
bootRun {
    jvmArgs([
        "-Xms256m",
        "-Xmx512m"
    ])
}
```

我们可以看到，使用jvmArgs选项修改了springboot应用程序的[最大/最小堆](https://www.baeldung.com/java-min-max-heap)[ps](https://www.baeldung.com/linux/ps-command)，现在，让我们使用命令来验证JVM对Spring Boot应用程序的更改：

```shell
$ ps -ef | grep java | grep spring
502  7870  7254   0  8:07PM ??  0:03.89 /Library/Java/JavaVirtualMachines/jdk-14.0.2.jdk/Contents/Home/bin/java
-XX:TieredStopAtLevel=1 -Xms256m -Xmx512m -Dfile.encoding=UTF-8 -Duser.country=IN 
-Duser.language=en com.example.demo.DemoApplication
```

在上面的bootRun任务中，我们使用jvmArgs选项更改了Spring Boot应用程序的最大和最小堆，**这样，JVM参数将动态附加到Spring Boot应用程序。此外，我们还可以使用-D选项向bootRun添加自定义属性**，为了演示，我们来看看bootRun任务：

```groovy
bootRun {
    jvmArgs(['-Dtuyucheng=test', '-Xmx512m'])
}
```

这样，我们就可以传递JVM选项和自定义属性了，为了说明这一点，我们来使用jvm参数验证一下自定义值：

```shell
$ ps -ef | grep java | grep spring
502  8423  7254   0  8:16PM ??  0:00.62 /Library/Java/JavaVirtualMachines/jdk-14.0.2.jdk/Contents/Home/bin/java 
-XX:TieredStopAtLevel=1  -Dtuyucheng=test -Xms256m -Xmx512m -Dfile.encoding=UTF-8 -Duser.country=IN 
-Duser.language=en com.example.demo.DemoApplication
```

另外，我们还可以将这些属性文件放入gradle.properties中，然后在build.gradle中使用它们：

```properties
tuyucheng=test
max.heap.size=512m
```

现在，我们可以在bootRun命令中使用它：

```groovy
bootRun {
    jvmArgs([
        "-Dtuyucheng=${project.findProperty('tuyucheng')}",
	"-Xmx${project.findProperty('max.heap.size')}"
    ])
}
```

使用上述方法我们可以将配置文件与主build.gradle文件分开。

## 4. 使用命令行参数

我们还可以直接向./gradlew bootRun命令提供JVM选项，在Gradle中，可以使用-D标志指定系统属性，使用-X标志指定JVM选项：

```shell
$ ./gradlew bootRun --args='--spring-boot.run.jvmArguments="-Xmx512m" --tuyucheng=test'
```

**我们可以使用此命令在运行时动态提供JVM选项，而无需修改Gradle构建文件**。为了演示，让我们使用ps命令验证JVM参数：

```shell
$ ps -ef | grep java | grep spring 
 502 58504 90399   0  7:21AM ?? 0:02.95 /Library/Java/JavaVirtualMachines/jdk-14.0.2.jdk/Contents/Home/bin/java 
 -XX:TieredStopAtLevel=1 -Xms256m -Xmx512m -Dfile.encoding=UTF-8 -Duser.country=IN -Duser.language=en 
 com.example.demo.DemoApplication --spring-boot.run.jvmArguments=-Xmx512m --tuyucheng=test
```

上述命令使用./gradlew bootRun命令直接设置jvm参数。

## 5. 总结

在本文中，我们学习了将JVM选项传递给bootRun命令的不同方法。

首先，我们了解了bootRun的重要性和基本用法。然后，我们探索了如何使用命令行参数和build.gradle文件为bootRun提供JVM选项。