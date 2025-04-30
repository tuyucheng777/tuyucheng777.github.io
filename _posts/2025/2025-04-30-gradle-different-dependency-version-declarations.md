---
layout: post
title:  Gradle中不同的依赖版本声明
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

Gradle是JVM项目最流行的构建工具之一，它提供了多种方法来声明和控制依赖版本。

在这个简短的教程中，我们将了解如何在Gradle中定义最新的依赖版本。

首先，我们将研究定义依赖版本的不同方式。然后，我们将创建一个小型Gradle项目，并在其中指定一些第三方库。最后，我们将分析Gradle的依赖树，并了解Gradle如何处理不同的版本声明。

## 2. 声明版本

### 2.1 精确版本

这是声明依赖版本的最直接的方法，我们所要做的就是指定我们想要在应用中使用的确切版本，例如1.0。

大多数官方依赖通常只使用数字版本号，例如3.2.1。但是，依赖版本号是一个字符串，因此它也可以包含字符。例如，Spring有一个5.2.22.RELEASE版本号。

### 2.2 Maven风格的版本范围

如果我们不想指定确切的依赖版本，我们可以使用Maven风格的版本范围，例如[1.0,2.0)，(1.0,2.0\]。

方括号[和\]表示包含边界，相反，圆括号(和)表示排除边界。此外，**我们可以在版本声明中混合使用不同的括号**。

### 2.3 前缀/通配符版本范围

我们可以使用+通配符来指定依赖版本范围，例如1.+，**Gradle将查找版本与+通配符之前的部分完全匹配的依赖**。

### 2.4 使用latest版本的关键字

Gradle提供了两个特殊的版本关键字，我们可以将其用于任何依赖：

- latest.integration将匹配最高版本的SNAPSHOT模块
- latest.release将匹配最高版本的非SNAPSHOT模块

## 3. 版本范围优缺点

使用Maven风格或通配符版本范围作为依赖版本可能很有用，但是，我们必须考虑这种方法的优缺点。

版本范围的最大优势在于我们始终拥有最新的依赖可用，我们不必在每次构建应用程序时都搜索新的依赖版本。

然而，始终拥有最新的依赖也可能是一个很大的缺点。我们的应用程序在不同版本之间可能会表现不同，仅仅是因为我们在应用程序中使用的某个第三方依赖发布了新版本，而这些依赖在我们没有意识到的情况下在不同版本之间发生了行为变化。

根据我们的需求，为依赖指定版本范围可能会有所帮助。但是，在使用依赖之前，我们必须谨慎并确定其遵循的版本控制算法。

## 4. 创建测试应用程序

让我们创建一个简单的Gradle应用程序来尝试一些具有不同版本声明的依赖。

我们的应用程序只包含一个文件build.gradle：

```groovy
plugins {
    id 'java'
}

group = "cn.tuyucheng.taketoday.gradle"
version = "1.0.0-SNAPSHOT"
sourceCompatibility = JavaVersion.VERSION_17

repositories {
    mavenLocal()
    mavenCentral()
}

dependencies {
}
```

接下来，我们将添加一些具有不同版本声明的[org.apache.commons依赖](https://mvnrepository.com/artifact/org.apache.commons)。

让我们从最简单的、精确的版本开始：

```groovy
implementation group: 'org.apache.commons', name: 'commons-lang3', version: '3.12.0'
```

现在，让我们使用Maven风格的版本范围：

```groovy
implementation group: 'org.apache.commons', name: 'commons-math3', version: '[3.4, 3.5)'
```

这里我们指定，我们需要一个commons-math3依赖，其版本介于3.4和3.5之间。

下一个依赖将使用通配符版本范围：

```groovy
implementation group: 'org.apache.commons', name: 'commons-collections4', version: '4.+'
```

我们希望拥有最新的commons-collections4依赖，其版本与4.前缀匹配。

最后，让我们使用Gradle latest.release关键字：

```groovy
implementation group: 'org.apache.commons', name: 'commons-text', version: 'latest.release'
```

这是build.gradle文件中的完整依赖部分：

```groovy
dependencies {
    implementation group: 'org.apache.commons', name: 'commons-lang3', version: '3.12.0'
    implementation group: 'org.apache.commons', name: 'commons-math3', version: '[3.4, 3.5)'
    implementation group: 'org.apache.commons', name: 'commons-collections4', version: '4.+'
    implementation group: 'org.apache.commons', name: 'commons-text', version: 'latest.release'
}
```

现在，让我们看看Gradle如何解析我们的版本声明。

## 5. 显示依赖关系树

让我们使用Gradle依赖任务来查看依赖报告：

```shell
$ gradle dependencies

compileClasspath - Compile classpath for source set 'main'.
+--- org.apache.commons:commons-lang3:3.12.0
+--- org.apache.commons:commons-collections4:4.+ -> 4.5.0-M2
+--- org.apache.commons:commons-math3:[3.4, 3.5) -> 3.4.1
\--- org.apache.commons:commons-text:latest.release -> 1.10.0
```

在撰写本文时，符合指定范围的最新commons-math3依赖版本为3.4.1，我们可以看到Gradle使用了该版本。

此外，4.5.0-M2是与4.+通配符匹配的最新commons-collections4版本。

同样，1.10.0是commons-text依赖的最新发布版本。

## 6. 总结

在本文中，我们学习了如何以各种方式在Gradle中声明依赖版本。

首先，我们了解了如何在Gradle构建脚本中指定确切的依赖版本以及版本范围。然后，我们尝试了几种表达依赖关系的方法。

最后，我们检查了Gradle如何解析这些版本。