---
layout: post
title:  使用Gradle运行Java main方法
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

在本教程中，我们将探讨使用Gradle执行[Java main方法](https://www.baeldung.com/java-main-method)的不同方法。

## 2. Java main方法

我们可以通过多种方式在Gradle中运行Java的main方法，让我们使用一个将消息打印到标准输出的简单程序来仔细看看它们：

```java
public class MainClass {
    public static void main(String[] args) {
        System.out.println("Goodbye cruel world ...");
    }
}
```

## 3. 使用application插件运行

application插件是核心Gradle插件，它定义了一组可立即使用的任务，帮助我们打包和分发我们的应用程序。

让我们首先在build.gradle文件中插入以下内容：

```groovy
plugins {
    id "application"
}
apply plugin : "java" 
ext {
   javaMainClass = "cn.tuyucheng.taketoday.gradle.exec.MainClass"
}

application {
    mainClassName = javaMainClass
}
```

该插件会自动生成一个名为run的任务，只需要我们将其指向主类即可，第9行的闭包就是用来触发该任务的：

```shell
~/work/tuyucheng/tutorials/gradle-java-exec> ./gradlew run

> Task :run
Goodbye cruel world ...

BUILD SUCCESSFUL in 531ms
2 actionable tasks: 1 executed, 1 up-to-date
```

## 4. 使用JavaExec任务运行

接下来，让我们借助JavaExec任务类型实现一个运行main方法的自定义任务：

```groovy
task runWithJavaExec(type: JavaExec) {
    group = "Execution"
    description = "Run the main class with JavaExecTask"
    classpath = sourceSets.main.runtimeClasspath
    main = javaMainClass
}
```

我们需要在第5行定义主类，并指定类路径，类路径是根据构建输出的默认属性计算得出的，包含编译后类的实际位置。

请注意，在每种情况下，**我们都使用主类的完全限定名称，包括包**。

让我们使用JavaExec运行示例：

```shell
~/work/tuyucheng/tutorials/gradle-java-exec> ./gradlew runWithJavaExec

> Task :runWithJavaExec
Goodbye cruel world ...

BUILD SUCCESSFUL in 526ms
2 actionable tasks: 1 executed, 1 up-to-date
```

## 5. 使用Exec任务运行

最后，我们可以使用基础的Exec任务类型执行主类，由于此选项为我们提供了以多种方式配置执行的可能性，因此让我们实现三个自定义任务并分别讨论它们。

### 5.1 从编译后的构建输出运行

首先，我们创建一个自定义的Exec任务，其行为类似于JavaExec：

```groovy
task runWithExec(type: Exec) {
    dependsOn build
    group = "Execution"
    description = "Run the main class with ExecTask"
    commandLine "java", "-classpath", sourceSets.main.runtimeClasspath.getAsPath(), javaMainClass
}
```

我们可以运行任何可执行文件(在本例中为java)并传递其运行所需的参数。

我们在第5行配置类路径并指向我们的主类，并且我们还在第2行向构建任务添加了依赖。这是必要的，因为我们只能在编译后才能运行主类：

```shell
~/work/tuyucheng/tutorials/gradle-java-exec> ./gradlew runWithExec

> Task :runWithExec
Goodbye cruel world ...

BUILD SUCCESSFUL in 666ms
6 actionable tasks: 6 executed

```

### 5.2 从输出Jar运行

第二种方法依赖于我们的小应用程序的jar打包：

```groovy
task runWithExecJarOnClassPath(type: Exec) {
    dependsOn jar
    group = "Execution"
    description = "Run the mainClass from the output jar in classpath with ExecTask"
    commandLine "java", "-classpath", jar.archiveFile.get(), javaMainClass
}
```

注意第2行对jar任务的依赖以及第5行对java可执行文件的第二个参数，**我们使用普通的jar，因此需要使用第四个参数指定入口点**：

```shell
~/work/tuyucheng/tutorials/gradle-java-exec> ./gradlew runWithExecJarOnClassPath

> Task :runWithExecJarOnClassPath
Goodbye cruel world ...

BUILD SUCCESSFUL in 555ms
3 actionable tasks: 3 executed
```

### 5.3 从可执行输出Jar运行

第三种方式也依赖于jar打包，但是我们借助manifest属性来定义入口点：

```groovy
jar {
    manifest {
        attributes(
            "Main-Class": javaMainClass
        )
    }
}

task runWithExecJarExecutable(type: Exec) {
    dependsOn jar
    group = "Execution"
    description = "Run the output executable jar with ExecTask"
    commandLine "java", "-jar", jar.archiveFile.get()
}
```

**这里我们不再需要指定classpath**，直接运行jar即可：

```shell
~/work/tuyucheng/tutorials/gradle-java-exec> ./gradlew runWithExecJarExecutable

> Task :runWithExecJarExecutable
Goodbye cruel world ...

BUILD SUCCESSFUL in 572ms
3 actionable tasks: 3 executed
```

## 6. 总结

在本文中，我们探讨了使用Gradle运行Java main方法的各种方法。

开箱即用，Application插件提供了一个可配置的最小任务来运行我们的方法，JavaExec任务类型允许我们在不指定任何插件的情况下运行main方法。

最后，通用Exec任务类型可以与Java可执行文件以各种组合使用以实现相同的结果，但需要依赖于其他任务。