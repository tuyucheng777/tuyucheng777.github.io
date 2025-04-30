---
layout: post
title:  Gradle包装器指南
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

[Gradle](https://www.baeldung.com/gradle)是开发者常用来管理项目构建生命周期的工具，它是所有新Android项目的默认构建工具。

在本教程中，我们将了解Gradle Wrapper，这是一个可以更轻松地分发项目的附带实用程序。

## 2. Gradle包装器

要构建基于Gradle的项目，我们需要在计算机上安装Gradle。但是，如果我们安装的版本与项目的版本不匹配，我们可能会遇到许多不兼容问题。

Gradle Wrapper(简称Wrapper)解决了这个问题，**它是一个脚本，使用声明的版本运行Gradle任务**。如果未安装声明的版本，Wrapper会安装所需的版本。

Wrapper的主要好处是我们可以：

- **在任何机器上使用Wrapper构建项目，无需先安装Gradle**
- 拥有固定的Gradle版本，这可以在CI管道上实现可重用且更强大的构建
- 通过更改Wrapper定义轻松升级到新的Gradle版本

在接下来的部分中，我们将运行需要[在本地安装Gradle](https://gradle.org/install/)的Gradle任务。

### 2.1 生成包装文件

要使用Wrapper，我们需要生成一些特定的文件，**我们将使用名为wrapper的内置Gradle任务来生成这些文件**，请注意，这些文件只需生成一次。

现在，让我们在项目目录中运行wrapper任务：

```shell
$ gradle wrapper
```

让我们看看这个命令的输出：

![](/assets/images/2025/gradle/gradlewrapper01.png)

让我们看看这些文件是什么：

- gradle-wrapper.jar包含下载gradle-wrapper.properties文件中指定的Gradle发行版的代码
- gradle-wrapper.properties包含Wrapper运行时属性-最重要的是，与当前项目兼容的Gradle发行版的版本
- gradlew是使用Wrapper执行Gradle任务的脚本
- gradlew.bat是适用于Windows机器的gradlew等效批处理脚本

默认情况下，wrapper任务会使用当前机器上安装的Gradle版本生成Wrapper文件。如果需要，我们可以指定其他版本：

```shell
$ gradle wrapper --gradle-version 6.3
```

**我们建议将Wrapper文件检入GitHub等源代码控制系统**，这样可以确保其他开发者无需安装Gradle即可运行该项目。

### 2.2 使用Wrapper运行Gradle命令

**我们可以通过用gradlew替换gradle来使用Wrapper运行任何Gradle任务**。

要列出可用的任务，我们可以使用gradlew tasks命令：

```shell
$ gradlew tasks
```

让我们看一下输出：

```text
Help tasks
----------
buildEnvironment - Displays all buildscript dependencies declared in root project 'gradle-wrapper'.
components - Displays the components produced by root project 'gradle-wrapper'. [incubating]
dependencies - Displays all dependencies declared in root project 'gradle-wrapper'.
dependencyInsight - Displays the insight into a specific dependency in root project 'gradle-wrapper'.
dependentComponents - Displays the dependent components of components in root project 'gradle-wrapper'. [incubating]
help - Displays a help message.
model - Displays the configuration model of root project 'gradle-wrapper'. [incubating]
outgoingVariants - Displays the outgoing variants of root project 'gradle-wrapper'.
projects - Displays the sub-projects of root project 'gradle-wrapper'.
properties - Displays the properties of root project 'gradle-wrapper'.
tasks - Displays the tasks runnable from root project 'gradle-wrapper'.
```

我们可以看到，输出与使用gradle命令运行此任务时获得的输出相同。

## 3. 常见问题

现在，让我们看看使用Wrapper时可能遇到的一些常见问题。

### 3.1 全局.gitignore忽略所有Jar文件

某些组织不允许开发人员将jar文件签入其源代码控制系统，通常，此类项目在全局.gitignore文件中设置了一条规则，用于忽略所有jar文件。因此，gradle-wrapper.jar文件不会被签入到Git仓库中，因此，Wrapper任务无法在其他机器上运行。在这种情况下，**我们需要将gradle-wrapper.jar文件强制添加到Git**：

```shell
git add -f gradle/wrapper/gradle-wrapper.jar
```

类似地，我们可能有一个项目特定的.gitignore文件，它会忽略jar文件，我们可以通过放宽.gitignore规则或强制添加包装器jar文件来解决这个问题，如上所示。

### 3.2 缺少包装器文件夹

在检入基于Wrapper的项目时，我们可能会忘记包含gradle文件夹中的wrapper文件夹。但正如我们上面所见，wrapper文件夹包含两个关键文件：gradle-wrapper.jar和gradle-wrapper.properties。

如果没有这些文件，使用Wrapper运行Gradle任务时就会出错，因此，**我们必须将Wrapper文件夹检入源代码管理系统**。

### 3.3 删除的包装文件

基于Gradle的项目包含一个.gradle文件夹，用于存储用于加速Gradle任务的缓存。有时，我们需要清除缓存以解决Gradle构建问题。通常，我们会删除整个.gradle文件夹，但我们可能会将Wrapper的gradle文件夹误认为.gradle文件夹，并将其也删除。之后，当我们尝试使用Wrapper运行Gradle任务时，肯定会遇到问题。

**我们可以通过从源中提取最新的更改来解决这个问题**，或者，我们可以重新生成Wrapper文件。

## 4. 总结

在本教程中，我们学习了Gradle Wrapper及其基本用法，并了解了使用Gradle Wrapper时可能遇到的一些常见问题。