---
layout: post
title:  Gradle：build.gradle与settings.gradle与gradle.properties
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本文中，我们将研究Gradle Java项目的不同配置文件。此外，我们还将了解实际构建的详细信息。

你可以查看[这篇文章](https://www.baeldung.com/gradle)来了解有关Gradle的一般介绍。

## 2. build.gradle

假设我们通过运行gradle init–type java-application来创建一个新的Java项目，这将为我们创建一个具有以下目录和文件结构的新项目：

```text
build.gradle
gradle    
    wrapper
        gradle-wrapper.jar
        gradle-wrapper.properties
gradlew
gradlew.bat
settings.gradle
src
    main
        java  
            App.java
    test      
        java
            AppTest.java
```

我们可以将build.gradle文件视为项目的心脏或大脑，本例中生成的文件如下所示：

```groovy
plugins {
    id 'java'
    id 'application'
}

mainClassName = 'App'

dependencies {
    compile 'com.google.guava:guava:23.0'

    testCompile 'junit:junit:4.12'
}

repositories {
    jcenter()
}
```

**它由Groovy代码组成，或者更准确地说，是基于Groovy的DSL(领域特定语言)，用于描述构建。我们可以在这里定义依赖，还可以添加用于依赖解析的Maven仓库等**。

Gradle的基本构建块是项目和任务，在本例中，由于应用了java插件，因此构建Java项目所需的所有任务都已隐式定义。这些任务包括assemble、check、build、jar、javadoc、clean等等。

这些任务也以这样的方式设置，它们描述了Java项目的有用依赖图，这意味着执行构建任务通常就足够了，而Gradle(和Java插件)将确保执行所有必要的任务。

如果我们需要额外的特殊任务，例如构建Docker镜像，那么它也应该被添加到build.gradle文件中，最简单的任务定义如下：

```groovy
task hello {
    doLast {
        println 'Hello Tuyucheng!'
    }
}
```

我们可以通过将任务指定为Gradle CLI的参数来运行它，如下所示：

```shell
$ gradle -q hello
Hello Tuyucheng!
```

它不会做任何有用的事情，但当然会打印出“Hello Tuyucheng!”。

如果是多项目构建，我们可能会有多个不同的build.gradle文件，每个项目一个。

build.gradle文件针对[Project](https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html)实例执行，每个子项目会创建一个Project实例。上述任务可以在build.gradle文件中定义，它们作为[Task](https://docs.gradle.org/current/javadoc/org/gradle/api/Task.html)对象集合的一部分驻留在Project实例中。Task对象本身由多个操作组成，并以有序列表的形式呈现。

在之前的示例中，我们添加了一个Groovy闭包，用于在列表末尾打印出“Hello Tuyucheng!”，方法是在hello Task对象上调用doLast(Closure action)。在Task执行过程中，Gradle通过调用Action.execute(T)方法按顺序执行其中的每个Action。

## 3. settings.gradle

Gradle还会生成一个settings.gradle文件：

```groovy
rootProject.name = 'gradle-example'
```

settings.gradle文件也是一个Groovy脚本。

与build.gradle文件不同，每次Gradle构建只会执行一个settings.gradle文件，我们可以用它来定义多项目构建的项目。

此外，我们还可以将代码注册为构建的不同生命周期钩子的一部分。

该框架要求在多项目构建中存在settings.gradle，而对于单项目构建它是可选的。

在创建构建的[Settings](https://docs.gradle.org/current/dsl/org.gradle.api.initialization.Settings.html)实例后，通过执行该文件来配置它，从而使用它。这意味着我们在settings.gradle文件中定义子项目，如下所示：

```groovy
include 'foo', 'bar'
```

并且Gradle在创建构建时在Settings实例上调用void include(String... projectPaths)方法。

## 4. gradle.properties

**Gradle默认不会创建gradle.properties文件，该文件可以位于不同的位置，例如项目根目录、GRADLE_USER_HOME内或-Dgradle.user.home命令行标志指定的位置**。

该文件由键值对组成，我们可以使用它来配置框架本身的行为，并且可以作为使用命令行标志进行配置的替代方案。

可能的键的示例如下：

- org.gradle.caching=(true,false)
- org.gradle.daemon=(true,false)
- org.gradle.parallel=(true,false)
- org.gradle.logging.level=(quiet,warn,lifecycle,info,debug)

此外，你可以使用此文件直接向Project对象添加属性，例如，带有其命名空间的属性：org.gradle.project.property_to_set

另一个用例是指定这样的JVM参数：

```shell
org.gradle.jvmargs=-Xmx2g -XX:MaxPermSize=256m -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8
```

请注意，解析gradle.properties文件需要启动一个JVM进程，这意味着这些JVM参数仅影响单独启动的JVM进程。

## 5. 构建概要

假设我们不将其作为守护进程运行，我们可以将Gradle构建的一般生命周期总结如下：

- 它作为一个新的JVM进程启动
- 它解析gradle.properties文件并相应地配置Gradle
- 接下来，它为构建创建一个Settings实例
- 然后，它根据Settings对象评估settings.gradle文件
- 它根据配置的Settings对象创建Projects层次结构
- 最后，它针对项目执行每个build.gradle文件

## 6. 总结

我们了解了不同的Gradle配置文件如何实现不同的开发目的，我们可以根据项目需求，使用它们来配置Gradle构建以及Gradle本身。