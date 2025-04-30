---
layout: post
title:  Gradle工具链对JVM项目的支持
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

在本教程中，我们将探索Gradle工具链对JVM项目的支持。

我们首先来了解一下此功能背后的动机，然后，我们将对其进行定义，并通过实际示例进行尝试。

## 2. 工具链背后的推理

在讨论工具链是什么之前，我们需要先了解一下它存在的原因。假设我们要编写一个Java项目，我们的Java项目可能包含一些测试。因此，我们至少需要编译代码并运行测试，我们添加内置的Gradle插件，并指定所需的字节码版本：

```groovy
plugins {
    id 'java'
}

java {
    sourceCompatibility = JavaVersion.VERSION_1_8
    targetCompatibility = JavaVersion.VERSION_1_8
}
```

此外，如果需要，我们可以告诉Gradle将[我们的测试类编译成不同的字节码版本](https://www.baeldung.com/gradle-sourcecompatiblity-vs-targetcompatibility)：

```groovy
tasks {
    compileTestJava {
        sourceCompatibility = JavaVersion.VERSION_1_7
        targetCompatibility = JavaVersion.VERSION_1_7
    }
}
```

到目前为止一切顺利，**唯一的细微差别是，为了编译我们的源代码/测试类，Gradle使用了它自己的JDK，也就是它运行的JDK**。不过，我们可以通过指定要使用的确切可执行文件来解决这个问题：

```groovy
compileTestJava.getOptions().setFork(true)
compileTestJava.getOptions().getForkOptions().setExecutable('/home/mpolivaha/.jdks/corretto-17.0.4.1/bin/javac')

compileJava.getOptions().setFork(true)
compileJava.getOptions().getForkOptions().setExecutable('/home/mpolivaha/.jdks/corretto-17.0.4.1/bin/javac')
```

**然而如果我们在构建过程中使用各种JDK，就会出现问题**。

例如，假设我们必须在发布之前在客户的JDK上测试我们的Java应用，这些JDK可能来自不同的供应商，虽然符合规范，但在细节上可能存在差异。理论上，我们可以不用工具链来解决这个问题，但这会是一个更加复杂的解决方案。工具链可以简化构建的配置，因为构建需要不同的JDK来实现不同的目的。

## 3. 工具链定义

从6.7版本开始，Gradle引入了JVM工具链(JVM Toolchains)功能。不过，“工具链”这个概念并不新鲜，它在Maven中已经存在了[相当长一段时间](https://maven.apache.org/guides/mini/guide-using-toolchains.html)。通常，工具链是构建、测试和运行软件所需的一组工具和二进制文件。因此，在Java中，我们可以说JDK就是Java工具链，因为它允许编译、测试和运行Java程序。

我们可以在项目级别定义工具链，因此在这种情况下，它看起来像这样：

```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(17)
        vendor = JvmVendorSpec.AMAZON
        implementation = JvmImplementation.VENDOR_SPECIFIC
    }
}
```

因此，我们可以指定所需的Java版本、JDK供应商以及该供应商的具体JVM实现。为了确保工具链规范正确，我们至少必须[设置版本](https://docs.gradle.org/current/userguide/toolchains.html#sec:configuring_toolchain_specifications)。

Gradle处理工具链的方式很简单，首先，它会尝试在本地查找所需的工具链；这有一套特定的算法。如果Gradle在本地找不到所需的工具链，它会尝试在远程查找并下载，如果Gradle无法在远程找到所需的工具链，构建就会失败。

还值得一提的是，有时我们可能想要禁用自动配置功能，我们可以通过将-Porg.gradle.java.installations.auto-download=false传递给gradle可执行文件来实现。在这种情况下，如果在本地找不到工具链，Gradle构建将会失败。

## 4. 任务级别的工具链

工具链的真正威力在于能够按任务方式指定JDK安装：

```groovy
tasks.named('compileJava').get().configure {
    javaCompiler = javaToolchains.compilerFor {
        languageVersion = JavaLanguageVersion.of(17)
        vendor = JvmVendorSpec.AMAZON
        implementation = JvmImplementation.VENDOR_SPECIFIC
    }
}

tasks.register("testOnAmazonJdk", Test.class, {
    javaLauncher = javaToolchains.launcherFor {
        languageVersion = JavaLanguageVersion.of(17)
        vendor = JvmVendorSpec.AMAZON
    }
})

tasks.named("testClasses").get().finalizedBy("testOnAmazonJdk")
```

在上面的示例中，我们将compileJava任务配置为在Oracle JDK 15上运行，我们还创建了testOnAmazonJdk任务，它将在testClasses任务之后立即运行。请注意，这个新任务也是在单独的JDK上执行的。

## 5. 本地工具链识别

最后，Gradle允许我们使用以下命令查看当前项目可用的工具链的本地安装：

```shell
gradle javaToolchains
```

首先，Gradle会在当前位置搜索构建文件。然后，它会根据构建文件中指定的位置/规则列出找到的工具链。

## 6. 总结

在本快速教程中，我们回顾了Gradle工具链功能，此功能简化了在构建过程中使用不同JDK的流程(如果适用)。该功能从Gradle 6.7开始提供，我们可以在任务级别使用它，这使得它非常有价值。