---
layout: post
title:  Gradle中的依赖管理
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本教程中，我们将学习如何在Gradle构建脚本中声明依赖，我们将使用[Gradle 6.7](https://gradle.org/)作为示例。

## 2. 典型结构

让我们从[Java项目](https://www.baeldung.com/gradle-building-a-java-app)的简单Gradle脚本开始：

```groovy
plugins {
    id 'java'
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter:2.3.4.RELEASE'
    testImplementation 'org.springframework.boot:spring-boot-starter-test:2.3.4.RELEASE'
}
```

如上所示，我们有3个代码块：plugins、repositories和dependencies。

首先，plugins代码块告诉我们这是一个Java项目。其次，dependencies代码块声明了编译项目生产源代码所需的spring-boot-starter依赖的版本号为2.3.4.RELEASE。此外，它还声明了项目的测试套件需要spring-boot-starter-test才能进行编译。

Gradle构建从Maven Central仓库中提取所有依赖，如repositories块所定义。

让我们关注如何定义依赖关系。

## 3. 依赖配置

我们可以在不同的配置中声明依赖，在这方面，我们可以选择更精确或更不精确的配置，稍后我们将会看到。

### 3.1 如何声明依赖关系

首先，配置分为4 个部分：

- **group**：组织、公司或项目的标识符
- **name**：依赖标识符
- **version**：我们要导入的版本
- **classifier**：用于区分具有相同组、名称和版本的依赖

我们可以用两种格式声明依赖，约定格式允许我们将依赖声明为String：

```groovy
implementation 'org.springframework.boot:spring-boot-starter:2.3.4.RELEASE'
```

相反，扩展格式允许我们将其写为Map：

```groovy
implementation group:``'org.springframework.boot', name: 'spring-boot-starter', version: '2.3.4.RELEASE'
```

### 3.2 配置类型

此外，Gradle还提供了许多依赖配置类型：

- **api**：用于明确依赖，并在类路径中公开它们，例如，当实现一个库时，使其对库使用者透明
- **implementation**：编译生产源代码所需，并且纯内部使用，它们不会暴露在包外部
- **compileOnly**：适用于仅需在编译时声明的情况，例如仅源代码注解或注解处理器，它们不会出现在运行时类路径或测试类路径中。
- **compileOnlyApi**：在编译时需要并且需要在类路径中对消费者可见时使用
- **RuntimeOnly**：用于声明仅在运行时需要且在编译时不可用的依赖
- **testImplementation**：编译测试所需
- **testCompileOnly**：仅在测试编译时需要
- **testRuntimeOnly**：仅在测试运行时需要

需要注意的是，Gradle的最新版本已弃用一些配置，例如compile、testCompile、runtime和testRuntime，但在撰写本文时，它们仍然可用。

## 4. 外部依赖的类型

让我们深入研究一下Gradle构建脚本中遇到的外部依赖类型。

### 4.1 模块依赖关系

基本上，声明依赖的最常见方式是引用仓库，Gradle仓库是按组、名称和版本组织的模块集合。

事实上，Gradle从仓库块内的指定仓库中提取依赖：

```groovy
repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter:2.3.4.RELEASE'
}
```

### 4.2 文件依赖关系

由于项目并不总是使用自动依赖管理，有些项目会将依赖组织为源代码或本地文件系统的一部分。因此，我们需要指定依赖的确切位置。

为此，我们可以使用文件来包含依赖集合：

```groovy
dependencies {
    runtimeOnly files('libs/lib1.jar', 'libs/lib2.jar')
}
```

类似地，我们可以使用filetree将jar文件的层次结构包含在目录中：

```groovy
dependencies {
    runtimeOnly fileTree('libs') { include '*.jar' }
}
```

### 4.3 项目依赖关系

由于一个项目可以依赖另一个项目来重用代码，Gradle为我们提供了这样做的机会。

假设要声明我们的项目依赖于共享项目：

```groovy
dependencies { 
    implementation project(':shared') 
}
```

### 4.4 Gradle依赖

在某些情况下，例如开发任务或插件，我们可以定义属于使用的Gradle版本的依赖：

```groovy
dependencies {
    implementation gradleApi()
}
```

## 5. buildScript

正如我们之前所见，我们可以在dependency块中声明源代码和测试的外部依赖。同样，buildScript块允许我们声明Gradle构建的依赖，例如第三方插件和任务类。需要注意的是，如果没有buildScript块，我们只能使用Gradle的开箱即用功能。

下面我们声明想要通过从Maven Central下载来使用[Spring Boot插件](https://www.baeldung.com/spring-boot-gradle-plugin)：

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.springframework.boot:spring-boot-gradle-plugin:2.3.4.RELEASE' 
    }
}
apply plugin: 'org.springframework.boot'
```

因此，我们需要指定下载外部依赖的来源，因为没有默认来源。

以上内容与旧版本的Gradle有关，在较新版本中，可以使用更简洁的形式：

```groovy
plugins {
    id 'org.springframework.boot' version '2.3.4.RELEASE'
}
```

## 6. 总结

在本文中，我们研究了Gradle依赖、如何声明它们以及不同的配置类型。