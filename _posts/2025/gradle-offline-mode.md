---
layout: post
title:  Gradle离线模式
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

[Gradle](https://www.baeldung.com/gradle)是全球数百万开发人员的首选构建工具，也是Android应用程序的官方构建工具。

我们通常使用Gradle从网络下载依赖，但有时我们无法访问网络，在这种情况下，Gradle的离线模式将会很有用。

在这个简短的教程中，我们将讨论如何在Gradle中实现离线模式。

## 2. 准备

在进入离线模式之前，我们需要先安装Gradle。然后，我们需要构建应用程序并下载所有依赖，否则，尝试使用离线模式时将会失败。

## 3. 离线模式

我们通常在命令行工具或IDE(如JetBrains IntelliJ IDEA和Eclipse)中使用Gradle，因此我们主要学习如何在这些工具中使用离线模式。

### 3.1 命令行

一旦我们在系统中安装了Gradle，我们就可以下载依赖并构建我们的应用程序：

```shell
gradle build
```

现在，我们可以通过添加–offline选项来实现离线模式：

```shell
gradle --offline build
```

### 3.2 JetBrains IntelliJ IDEA

当我们使用IntelliJ时，我们可以将Gradle与其集成并配置，然后我们将看到Gradle窗口。

如果我们需要使用离线模式，只需转到Gradle窗口并单击Toggle Offline Mode按钮：

![](/assets/images/2025/gradle/gradleofflinemode01.png)

当我们点击启用离线模式的按钮之后，我们可以重新加载所有依赖，并发现离线模式已经起作用了。

### 3.3 Eclipse

最后，让我们看看如何在Eclipse中实现离线模式，我们可以在Eclipse的“Preferences” -> “Gradle”部分找到Gradle配置，我们可以看到离线模式的配置并将其关闭：

![](/assets/images/2025/gradle/gradleofflinemode02.png)

然后，我们会发现离线模式在Eclipse中起作用了。

## 4. 总结

在本快速教程中，我们讨论了Gradle中的离线模式，我们学习了如何从命令行以及两个流行的IDE(Eclipse和IntelliJ)启用离线模式。