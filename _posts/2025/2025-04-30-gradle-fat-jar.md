---
layout: post
title:  在Gradle中创建Fat Jar
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在这篇简短的文章中，我们将介绍如何在Gradle中创建“fat jar”。

基本上，**fat jar(也称为uber-jar)是一个自给自足的档案，其中包含运行应用程序所需的类和依赖**。

## 2. 初始设置

让我们从具有两个依赖的Java项目的简单build.gradle文件开始：

```groovy
plugins {
   id 'java'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation group: 'org.slf4j', name: 'slf4j-api', version: '1.7.25'
    implementation group: 'org.slf4j', name: 'slf4j-simple', version: '1.7.25'
}
```

## 3. 使用Java插件中的Jar任务

让我们从修改Java Gradle插件中的jar任务开始，默认情况下，此任务生成的jar文件不包含任何依赖。

我们可以通过添加几行代码来覆盖此行为，要使其正常工作，我们需要做两件事：

- 清单文件中的Main-Class属性
- 包含依赖jar

让我们对Gradle任务添加一些修改：

```groovy
jar {
    manifest {
        attributes "Main-Class": "cn.tuyucheng.taketoday.fatjar.Application"
    }

    from {
        configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
```

## 4. 创建单独的任务

如果我们想保留原始的jar任务，我们可以创建一个单独的任务来执行相同的工作。

以下代码将添加一个名为customFatJar的新任务：

```groovy
task customFatJar(type: Jar) {
    manifest {
        attributes 'Main-Class': 'cn.tuyucheng.taketoday.fatjar.Application'
    }
    archiveBaseName = 'all-in-one-jar'
    duplicatesStrategy = DuplicatesStrategy.EXCLUDE
    from { configurations.runtimeClasspath.collect { it.isDirectory() ? it : zipTree(it) } }
    with jar
}
```

## 5. 使用专用插件

我们还可以使用现有的Gradle插件来构建fat jar。

在此示例中，我们将使用[Shadow](https://github.com/johnrengelman/shadow)插件：

```groovy
buildscript {
    repositories {
        mavenCentral()
        gradlePluginPortal()
    }
    dependencies {
        classpath "gradle.plugin.com.github.johnrengelman:shadow:7.1.2"
    }
}
```

```groovy
plugins {
  id 'com.github.johnrengelman.shadow' version '7.1.2'
  id 'java'
}
```

一旦我们应用了Shadow插件，shadowJar任务就可以使用了。

## 6. 总结

在本教程中，我们介绍了几种在Gradle中创建Fat Jar文件的方法。我们覆盖了默认的jar任务，创建了一个单独的任务，并使用了Shadow插件。

在简单的项目中，覆盖默认的jar任务或创建一个新的jar任务就足够了。但随着项目规模的扩大，我们强烈建议使用插件，因为它们已经解决了更棘手的问题，例如与外部META-INF文件的冲突。