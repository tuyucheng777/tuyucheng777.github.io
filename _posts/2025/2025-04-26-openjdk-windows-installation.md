---
layout: post
title:  在Windows上安装OpenJDK
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

Java在现代软件开发中扮演着关键角色，为许多应用程序和系统提供支持。为了在我们的机器上充分利用Java的强大功能，我们需要安装Java开发工具包(JDK)。虽然Oracle JDK是一个受欢迎的选择，但[OpenJDK](https://openjdk.org/)提供了一个具有类似功能的开源替代方案。

**在本文中，我们将探讨在Windows环境中安装OpenJDK的各种方法，以满足不同的偏好和要求**。

## 2. 手动安装

此方法涉及直接从[官方网站](https://jdk.java.net/)或受信任的仓库(例如[AdoptOpenJDK](https://adoptopenjdk.net/releases.html))下载OpenJDK发行版。

下载完成后，将存档内容解压到我们机器上的首选位置，必须配置环境变量(例如PATH和JAVA_HOME)以指向OpenJDK的安装目录，让我们继续访问控制面板并导航到“系统设置”：

![](/assets/images/2025/javajvm/openjdkwindowsinstallation01.png)

选择高级系统设置将提示出现一个对话框：

![](/assets/images/2025/javajvm/openjdkwindowsinstallation02.png)

现在，我们点击“环境变量”来查看系统变量和用户变量。在这里，我们将修改PATH变量并添加JAVA_HOME变量，JAVA_HOME变量应指向OpenJDK的安装目录，而PATH变量应指向JDK的bin目录。

![](/assets/images/2025/javajvm/openjdkwindowsinstallation03.png)

在我们的例子中，JAVA_HOME将是C:\\Program Files\\Java\\jdk-21.0.2，PATH将是C:\\Program Files\\Java\\jdk-21.0.2\\bin。

最后，我们可以通过在命令提示符中运行以下命令来确认安装是否成功：

```shell
> java -version
```

运行上述命令后，命令提示符中将显示类似的输出：

![](/assets/images/2025/javajvm/openjdkwindowsinstallation04.png)

## 3. Chocolatey包管理器

[Chocolatey](https://chocolatey.org/)是一款流行的Windows软件包管理器，可简化软件包的安装和管理。**它提供了一个命令行界面(CLI)，允许用户轻松搜索、安装和卸载软件包，类似于Ubuntu上的[apt](https://www.baeldung.com/linux/yum-and-apt)或macOS上的[Homebrew](https://www.baeldung.com/linux/homebrew)等软件包管理器**。

在继续之前，我们需要先在机器上安装Chocolatey，让我们打开一个提升权限的命令行并运行以下命令：

```shell
> Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

安装Chocolatey后，我们可以使用它来安装OpenJDK，运行以下命令将安装Java：

```shell
> choco install openjdk
```

## 4. Scoop包管理器

与Chocolatey类似，[Scoop](https://scoop.sh/)是另一个专为Windows设计的软件包管理器。**Scoop面向个人用户，而非系统范围的安装**，它将软件包安装在用户的主目录中，无需管理员权限。

要开始使用Scoop，我们必须首先安装它：

```shell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser 
> Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

现在，要使用Scoop安装OpenJDK，我们需要以管理员身份打开PowerShell并执行以下命令：

```shell
> scoop bucket add java
> scoop install openjdk
```

## 5. 使用第三方安装程序

一些第三方工具和实用程序简化了Windows上OpenJDK的安装过程，例如，像[SDKMAN](https://sdkman.io/)和[WinGet](https://winget.run/)这样的工具提供了易于使用的界面来管理软件安装，包括OpenJDK。

如果我们更喜欢具有附加功能和自定义选项的更简化的安装过程，我们可以探索这些选项。

## 6. 总结

在本文中，我们探讨了在Windows机器上安装OpenJDK的不同方法。我们可以选择手动安装、使用Chocolatey或Scoop等包管理器，或者使用第三方安装程序，每种方法在简单性、可定制性和自动化方面都有其优势。