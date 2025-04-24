---
layout: post
title:  查看JAR文件的内容
category: saas
copyright: saas
excerpt: Jar
---

## 1. 概述

我们已经学习了如何[从JAR文件中获取类名](https://www.baeldung.com/jar-file-get-class-names)，此外，在该教程中，我们还将讨论如何在Java应用程序中获取JAR文件中的类名。

在本教程中，我们将学习另一种从命令行列出JAR文件内容的方法。

我们还将看到几个用于查看JAR文件的更详细内容的GUI工具-例如，Java源代码。

## 2. 示例JAR文件

本教程中我们依然以[stripe-0.0.1-SNAPSHOT.jar](https://github.com/eugenp/tutorials/tree/master/saas-modules/stripe)文件为例，讲解如何查看JAR文件中的内容：

![](/assets/images/2025/saas/javaviewjarcontents01.png)

## 3. 回顾jar命令

我们已经了解到可以[使用JDK附带的jar命令](https://www.baeldung.com/jar-file-get-class-names#using-thejar-command)来检查JAR文件的内容：

```shell
$ jar tf stripe-0.0.1-SNAPSHOT.jar 
META-INF/
META-INF/MANIFEST.MF
...
templates/result.html
templates/checkout.html
application.properties
cn/tuyucheng/taketoday/stripe/StripeApplication.class
cn/tuyucheng/taketoday/stripe/ChargeRequest.class
cn/tuyucheng/taketoday/stripe/StripeService.class
cn/tuyucheng/taketoday/stripe/ChargeRequest$Currency.class
```

如果我们想过滤输出以仅获取我们想要的信息，例如类名或属性文件，我们可以通过管道将输出传输到过滤工具，例如[grep](https://www.baeldung.com/linux/common-text-search)。

如果我们的系统安装了JDK，那么jar命令使用起来就非常方便。

然而，有时我们想在未安装JDK的系统上查看JAR文件的内容。在这种情况下，jar命令不可用。

接下来我们将看一下这一点。

## 4. 使用unzip命令

**JAR文件以ZIP文件格式打包**，换句话说，如果某个实用程序可以读取ZIP文件，那么我们也可以用它来查看JAR文件。

unzip命令是Linux命令行中处理ZIP文件的常用实用程序。

因此，我们可以使用[unzip](https://linux.die.net/man/1/unzip)命令的-l选项来列出JAR文件的内容，而无需解压缩它：

```shell
$ unzip -l stripe-0.0.1-SNAPSHOT.jar
Archive:  stripe-0.0.1-SNAPSHOT.jar
  Length      Date    Time    Name
---------  ---------- -----   ----
        0  2020-10-16 20:53   META-INF/
...
      137  2020-10-16 20:53   static/index.html
      677  2020-10-16 20:53   templates/result.html
     1323  2020-10-16 20:53   templates/checkout.html
       37  2020-10-16 20:53   application.properties
      715  2020-10-16 20:53   com/tuyucheng/stripe/StripeApplication.class
     3375  2020-10-16 20:53   cn/tuyucheng/taketoday/stripe/ChargeRequest.class
     2033  2020-10-16 20:53   cn/tuyucheng/taketoday/stripe/StripeService.class
     1146  2020-10-16 20:53   cn/tuyucheng/taketoday/stripe/ChargeRequest$Currency.class
     2510  2020-10-16 20:53   cn/tuyucheng/taketoday/stripe/ChargeController.class
     1304  2020-10-16 20:53   com/tuyucheng/stripe/CheckoutController.class
...
---------                     -------
    15394                     23 files
```

借助unzip命令，我们可以在没有JDK的情况下查看JAR文件的内容。

上面的输出非常清晰，它以表格形式列出了JAR文件中的文件。

## 5. 使用GUI实用程序探索JAR文件

jar和unzip命令都很方便，但它们只列出JAR文件中的文件名。

有时，我们想了解有关JAR文件中文件的更多信息，例如检查某个类的Java源代码。

在本节中，我们将介绍几个独立于平台的GUI工具来帮助我们查看JAR文件中的文件。

### 5.1 使用 JD-GUI

首先，我们来看一下[JD-GUI](http://java-decompiler.github.io/)。

JD-GUI是一个很好的开源GUI实用程序，用于探索由Java反编译器[JD-Core](https://github.com/java-decompiler/jd-core)反编译的Java源代码。

JD-GUI附带一个JAR文件，我们可以使用带有-jar选项的java命令来启动该实用程序，例如：

```shell
$ java -jar jd-gui-1.6.6.jar
```

当我们看到JD-GUI的主窗口时，我们可以通过导航菜单“File -> Open File...”来打开我们的JAR文件，或者直接将JAR文件拖放到窗口中。

一旦我们打开一个JAR文件，JAR文件中的所有类都会被反编译。

然后我们可以在左侧选择我们感兴趣的文件来检查它们的源代码：

[![20201209_232956](https://www.baeldung.com/wp-content/uploads/2020/12/20201209_232956.gif)](https://www.baeldung.com/wp-content/uploads/2020/12/20201209_232956.gif)

正如我们在上面的演示中看到的，**在左侧的大纲中，还列出了类以及每个类的成员(例如方法和字段)，就像我们通常在IDE中看到的那样**。

定位方法或字段非常方便，特别是当我们需要检查一些具有多行代码的类时。

当我们点击左侧的不同类时，每个类将在右侧的选项卡中打开。

如果我们需要在多个类别之间切换，则标签功能很有用。

### 5.2 使用Jar Explorer

[Jar Explorer](http://dst.in.ua/jarexp/index.html?l=en)是另一个用于查看JAR文件内容的开源GUI工具，它附带一个jar文件和一个启动脚本“Jar Explorer.sh”，它也支持拖放功能，使打开JAR文件变得非常容易。

Jar Explorer提供的另一个不错的功能是**它支持三种不同的Java反编译器：JD-Core、[Procyon](https://github.com/mstrobel/procyon)和[Fernflower](https://github.com/fesh0r/fernflower)**。

我们可以在检查源代码时在反编译器之间切换：

[![20201210_000351](https://www.baeldung.com/wp-content/uploads/2020/12/20201210_000351.gif)](https://www.baeldung.com/wp-content/uploads/2020/12/20201210_000351.gif)

Jar Explorer相当好用，它的反编译器切换功能也很棒，不过，左侧的概要只停留在类级别。

此外，由于Jar Explorer不提供选项卡功能，我们一次只能打开一个文件。

而且，我们每次在左侧选择一个类时，该类都会被当前选择的反编译器进行反编译。

### 5.3 使用Luyten

[Luyten](https://github.com/deathmarine/Luyten)是一款出色的Java反编译器Procyon开源GUI实用程序，它提供[不同平台的下载](https://github.com/deathmarine/Luyten/releases)，例如.exe格式和JAR格式。

下载JAR文件后，我们可以使用java-jar命令启动Luyten：

```shell
$ java -jar luyten-0.5.4.jar 
```

我们可以将JAR文件拖放到Luyten中并浏览JAR文件中的内容：

[![20201210_003959](https://www.baeldung.com/wp-content/uploads/2020/12/20201210_003959.gif)](https://www.baeldung.com/wp-content/uploads/2020/12/20201210_003959.gif)

使用Luyten时，我们无法选择不同的Java反编译器。但是，正如上面的演示所示，Luyten提供了多种反编译选项。此外，我们还可以在选项卡中打开多个文件。

除此之外，Luyten支持一个很好的主题系统，我们可以在检查源代码的同时选择一个舒适的主题。

但是，Luyten仅列出了JAR文件的结构到文件级别。

## 6. 总结

在本文中，我们学习了如何从命令行列出JAR文件中的文件。之后，我们还学习了三个GUI实用程序，用于查看JAR文件的更详细内容。

如果我们想要反编译类并检查JAR文件的源代码，选择GUI工具可能是最直接的方法。