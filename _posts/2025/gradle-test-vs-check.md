---
layout: post
title:  Gradle test和check之间的区别
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

近年来，[Gradle](https://www.baeldung.com/gradle-dependency-management)已成为非常流行的依赖管理工具，尤其是在Java开发者中。它易于学习，并且由于采用Groovy编写，因此扩展性也非常好。

在本快速教程中，**我们将了解Gradle test任务和check任务之间的区别**，我们将明确每个任务的作用以及何时使用它们。

## 2. Gradle概述

Gradle是一个依赖管理工具，它提供了强大的功能来定义软件项目中的依赖，以及各种功能来管理从测试到部署的整个构建生命周期。

Gradle使用任务来完成其工作，任务本质上是一组用于实现目标的独立步骤，以下是一些示例：

- 编译源代码
- 将源代码打包成模块
- 将模块部署到远程系统

虽然Gradle具有可扩展性，允许我们定义任何所需的任务，但Java开发人员最常用的任务默认都已提供，其中两个任务名为test和check。

虽然它们本质上相似，但在管理软件项目时，它们各自扮演的角色略有不同。在接下来的章节中，我们将更深入地探讨它们。

## 3. 何时使用Gradle Test

要运行test任务，我们只需运行：

```shell
gradle test
```

**test任务执行项目中的所有单元测试**，test任务有相当多的[属性](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.testing.Test.html)来控制其行为，我们不会在这里全部介绍它们。

默认情况下，test任务会自动检测项目中的所有[单元测试](https://www.baeldung.com/junit-5-gradle)，进行编译并执行。最后，它会生成一份报告，列出哪些测试通过了，哪些测试失败了。

test任务是一个非常通用的任务，所有开发者在使用Gradle时都应该熟悉它。通常，每个开发者在向主分支提交任何新代码之前都应该执行它。

但作为额外的保护层，**它通常也会作为任何正式软件构建过程的一部分自动执行**。例如，在编译源代码之后、组装最终工件之前，通常将test任务用作任何自动化软件构建的一部分。

## 4. 何时使用Gradle Check

要运行check任务，我们只需运行：

```shell
gradle check
```

与test任务不同，check任务被称为“生命周期”任务，**这意味着它本身不执行任何操作，相反，它会执行一个或多个其他任务**。

默认情况下，check任务仅执行test任务。这意味着，**在没有任何其他插件的情况下，这两个任务的行为完全相同，并生成相同的输出**。

check之所以重要，是因为它可以聚合一个或多个验证任务，这使我们能够将多个任务合并到一个步骤中，而不必逐个执行。

Gradle生态系统包含许多插件，例如[checkstyle](https://docs.gradle.org/current/userguide/checkstyle_plugin.html)插件，它为check任务提供了额外的功能。

一般来说，任何进行源代码验证的任务都应附加到check任务中，源代码风格强制执行、库漏洞扫描和集成测试等都适合包含在check任务中。

## 5. 总结

在本文中，我们仔细研究了Gradle的test和check任务，虽然它们很相似，有时可以互换使用，但它们的用途却有所不同。

大多数开发人员都应该熟悉test任务，因为它执行我们所有的单元测试并提供哪些通过和失败的报告。

相比之下，check任务将test任务与其他任务结合在一起。有许多插件可以向检查任务添加自己的步骤，但我们始终可以根据软件项目的特定需求创建自己的插件。