---
layout: post
title:  在IntelliJ IDEA中设置Gradle JVM
category: gradle
copyright: gradle
excerpt: Gradle
---

## 1. 概述

在本教程中，我们将学习如何[在IntelliJ](https://www.jetbrains.com/help/idea/settings-build-tools.html)Gradle项目中更改JVM版本，该操作适用于IntelliJ社区版和旗舰版。

## 2. IntelliJ中的Gradle JVM设置

[IntelliJ](https://www.baeldung.com/intellij-basics)将Gradle使用的JVM版本存储在其Build Tools中，有两种方法可以找到它：

- 通过菜单导航：导航至File -> Settings
- 通过键盘快捷键：对于Windows，我们按Ctrl + Alt + S和 f或OS X，我们按⌘Cmd + ,

结果，它会打开“Settings”菜单，要更新Gradle设置，请按照以下路径操作：Build, Execution, Deployment -> Build Tools -> Gradle。

然后我们会看到一个类似这样的弹出对话框：

![](/assets/images/2025/gradle/intellijsetgradlejvm01.png)

在最后的[Gradle](https://www.baeldung.com/gradle-series)部分下，我们可以选择Gradle JVM，这将设置所有Gradle操作期间使用的JVM。因此，更新到新版本的Java后，项目将开始重新索引其源文件和库，这确保了构建、编译和其他IDE功能同步。

**更新JVM时，我们需要记住，这仅适用于在IntelliJ中构建项目时**。如果我们尝试从命令行工具构建它，它仍然会从[JAVA_HOME](https://www.baeldung.com/java-home-on-windows-mac-os-x-linux)环境变量中引用JVM。

此外，这仅设置Gradle使用的JVM，但不会更改[项目JDK](https://www.baeldung.com/intellij-change-java-version)版本。

## 3. 总结

在本文中，我们学习了如何更改IntelliJ Gradle构建工具中使用的JVM版本。**我们还强调，更改IntelliJ中的JVM版本既不会更新项目JDK，也不会影响IntelliJ之外的任何设置(例如命令行工具或任何其他IDE)**。