---
layout: post
title:  Maven命令的Gradle等效项
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 简介

Maven和[Gradle](https://www.baeldung.com/gradle)是两种最流行的构建自动化和依赖管理工具，它们简化了开发人员的工作。Maven是一种广泛使用的构建自动化工具，而Gradle则更加现代化且灵活。

**作为从Maven过渡到Gradle的人，了解Gradle中的等效命令至关重要**。

在本教程中，我们将探索Maven命令及其在Gradle中的等效命令。我们还将了解如何将一个项目的构件发布到Maven本地仓库，以便其他项目可以使用。

## 2. Maven命令及其Gradle等效命令

在本节中，我们将探讨一些最流行的Maven命令及其在Gradle中的等效命令。

### 2.1 更新依赖

Maven和Gradle都允许我们更新项目依赖，以确保我们拥有它们的最新版本。

在Maven中，以下是用于更新项目依赖的典型命令：

```shell
mvn clean install -U
```

带有-U(大写)选项的[mvn clean install](https://www.baeldung.com/maven-install-versus-verify)将强制Maven检查依赖的更新版本并从远程仓库下载它们(即使它们已经存在于本地仓库中)。

但是此命令不会更新非快照依赖或发布版本，如果要更新非快照依赖，仍然需要修改pom.xml文件，或者也可以使用以下命令：

```shell
mvn versions:use-latest-releases
```

**此命令将使用最新的发布版本替换任何非快照且没有年-月-日后缀的发布版本**。

在Gradle中，我们使用以下命令强制刷新来自远程仓库的依赖。

```shell
gradle build --refresh-dependencies
```

此命令会忽略本地缓存的依赖版本，并强制Gradle在已配置的仓库中重新查找。**需要注意的是，Gradle会检查远程仓库中是否有动态版本(例如[SNAPSHOT](https://www.baeldung.com/maven-snapshot-release-repository)、latest.release和1.+)的更新，如果有更新版本可用，则会下载**。

然而，即使Gradle忽略了缓存，它也不会盲目地重新下载每个工件。相反，它会比较本地和远程仓库中文件的哈希值，如果匹配，则使用现有文件而不是重新下载。

### 2.2 构建和安装项目

在Maven中，为了清理、编译、测试、打包和安装项目，我们使用以下命令：

```shell
mvn clean install
```

我们可以使用以下命令在Gradle中实现相同的功能：

```shell
gradle clean build
```

此命令删除build/目录，然后编译、测试和打包项目。

### 2.3 其他常用命令

现在让我们看一下Maven中一些常用的其他命令以及Gradle中的等效命令：

|Maven命令| Gradle命令              | 目的|
| ---------------------------- |-----------------------| ----------------------------------------- |
|mvn clean| gradle clean          | 删除构建目录|
|mvn compile| gradle compileJava    | 编译Java代码|
|mvn test| gradle test           | 运行测试|
|mvn package                     | gradle jar/gradle war | 创建JAR/WAR文件|
|mvn verify| gradle check          | 运行所有验证任务|
|mvn deploy| gradle publish        | 将工件发布到远程仓库|
|mvn site| 没有直接等价项               | Maven生成项目站点；Gradle没有内置此功能|
|mvn dependency:tree| gradle dependencies              | 显示项目依赖关系|
|mvn dependency:purge-local-repository| gradle –refresh-dependencies          | 强制重新下载依赖|

## 3. 处理多个项目

在处理多个项目时，我们经常会遇到相互依赖的项目。我们需要将一个项目的工件发布到本地仓库中，以供另一个项目使用。

在Maven中，我们运行命令mvn install来将工件添加到本地仓库。但是在Gradle中，我们该如何实现同样的功能呢？我们来看一下。

对于Gradle 7之前的版本，我们在build.gradle文件中按如下方式使用maven插件：

```groovy
apply plugin: "java"
apply plugin: "maven"
group = 'com.example'
version = '1.0.0'
```

然后运行：

```shell
gradle install
```

此命令将工件安装到~/.m2/repository中，使其可供依赖目使用。

对于Gradle 7及更高版本，我们使用maven-publish插件将工件发布到本地仓库：

```groovy
plugins {
    id 'java'
    id 'maven-publish'
}
group = 'com.example'
version = '1.0.0'
publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
        }
    }
    repositories {
        mavenLocal()
    }
}
```

上述脚本定义了两个插件，**Java插件用于执行标准的Java编译和打包，Maven-publish插件用于将构件发布到本地仓库**。

接下来，我们有组和版本属性，它们定义项目的唯一标识符和版本。

publishing块帮助我们定义项目构件的发布方式，mavenJava发布，其类型为MavenPublication，是使用from components.java语句创建的。

**这将确保已编译的Java组件(包括JAR文件和其他元数据)包含在发布中**。

最后，作为仓库的一部分，我们定义了mavenlocal()，它指的是Maven本地仓库。

定义好这些之后，我们就可以运行mvn install来将工件安装到本地Maven仓库中。现在，该工件就可以供其他本地项目使用了。

## 4. 总结

在本文中，我们比较了Maven和Gradle的命令。正如我们所见，它们非常相似。

Maven能做到的事情Gradle也能做到；但是，两者在配置和需要运行的命令方面有所不同。虽然习惯Gradle需要一些时间，但如果我们比较并理解两者之间的相似之处，过渡就会更容易。