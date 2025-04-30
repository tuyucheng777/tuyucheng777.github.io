---
layout: post
title:  Gradle中的BuildScripts块
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本教程中，我们将了解[Gradle](https://www.baeldung.com/gradle)中的构建脚本块([build.gradle](https://www.baeldung.com/gradle-build-settings-properties)文件中的脚本)，并详细了解buildScript块的用途。

## 2. 简介

### 2.1 什么是Gradle？

它是一个自动化构建工具，可以执行编译、打包、测试、部署、发布、[依赖解析](https://www.baeldung.com/gradle)等任务。如果没有它，我们就必须手动完成这些任务，这非常复杂且耗时。在当今的软件开发中，如果没有这样的构建工具，工作将非常困难。

### 2.2 Gradle常用构建脚本块

在本节中，我们将简要了解最常见的构建脚本块，**allProjects、subProjects、plugins、dependency、repositories、publication和buildScript**是最常见的构建脚本块，以下列表介绍了这些块的概念：

- [allProjects](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:allprojects(groovy.lang.Closure))块配置根项目和每个子项目。
- [subProjects](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:subprojects(groovy.lang.Closure))块与allProjects不同，它仅配置子项目。
- [plugins](https://docs.gradle.org/current/userguide/plugins.html)通过提供一系列实用的功能扩展了Gradle的功能，例如，java插件添加了诸如assemble、build、clean、jar、documentation等任务，以及更多其他功能。
- [dependencies](https://www.baeldung.com/gradle-dependency-management)是声明项目所需的所有jar的地方。
- [repositories](https://docs.gradle.org/current/userguide/declaring_repositories.html#header)块包含Gradle下载dependencies块中声明的jar文件的位置，可以声明多个位置，并按声明顺序执行。
- 当我们开发一个库并希望发布它时，会声明一个[publishing](https://docs.gradle.org/current/dsl/org.gradle.api.Project.html#org.gradle.api.Project:publishing(groovy.lang.Closure))块，该块包含一些详细信息，例如库jar文件的坐标，以及包含要发布位置的repositories块。

现在，我们考虑一个需要在构建脚本中使用库的用例。在这种情况下，我们不能使用dependencies块，因为它**包含项目classpath中所需的jar文件**。

由于我们想在构建脚本本身中使用该库，因此需要将该库添加到脚本的类路径中，这就是buildScript的作用所在，下一节将结合此用例深入讨论buildScript块。

## 3. BuildScript块的用途

考虑到上面定义的用例，假设在一个[Spring Boot](https://www.baeldung.com/spring-boot)应用中，我们想要在构建脚本中读取application.yml文件中定义的属性。为了实现这一点，我们可以使用一个名为[snakeyaml](https://www.baeldung.com/java-snake-yaml)的库，它可以解析YAML文件并轻松读取属性。

正如我们在上一节中讨论的那样，我们需要将此库添加到脚本类路径中，解决方案是**在buildScript块中将其添加为依赖**。

该脚本显示如何读取application.yml文件的属性temp.files.path，buildScript块包含[snakeyaml](https://www.baeldung.com/java-snake-yaml)库的依赖和下载它的仓库位置：

```groovy
import org.yaml.snakeyaml.Yaml

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath group: 'org.yaml', name: 'snakeyaml', version: '1.19'
    }
}

plugins {
    //plugins
}

def prop = new Yaml().loadAll(new File("$projectDir/src/main/resources/application.yml")
    .newInputStream()).first()
var path = prop.temp.files.path
```

path变量包含temp.files.path的值。 

有关buildScript块的更多信息：

- 它可以包含除项目类型依赖之外的任何类型的依赖。
- 对于多项目构建，声明的依赖可用于其所有子项目的构建脚本。
- 要将可作为外部jar使用的二进制插件添加到项目，我们应该将它们添加到构建脚本类路径，然后应用该插件。

## 4. 总结

在本教程中，我们了解了Gradle的使用、构建脚本最常见块的用途，并通过用例深入研究了buildScript块。