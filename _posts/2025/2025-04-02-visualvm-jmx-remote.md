---
layout: post
title:  使用VisualVM和JMX进行远程监控
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本文中，我们将学习如何使用VisualVM和Java Management Extensions(JMX)来远程监控Java应用程序。

## 2. JMX

**JMX是用于管理和监视JVM应用程序的标准API**，JVM具有JMX可用于此目的的内置工具。因此，我们通常将这些实用程序称为“开箱即用的管理工具”，或者在本例中称为“JMX代理”。

## 3. VisualVM

VisualVM是一种可视化工具，可为JVM提供轻量级分析功能。还有许多其他主流[分析工具](https://www.baeldung.com/java-profilers)，但是，VisualVM是免费的，并且与JDK 6U7版本捆绑在一起，直到JDK 8的早期更新。对于其他版本，[Java VisualVM](https://visualvm.github.io/)可作为独立应用程序使用。

**VisualVM允许我们连接到本地和远程JVM应用程序以进行监控**。

当在任何机器上启动时，**它会自动发现并开始监视本地运行的所有JVM应用程序**。但是，我们需要显式连接远程应用程序。

### 3.1 JVM连接模式

JVM通过[jstatd](https://docs.oracle.com/en/java/javase/11/tools/jstatd.html)或[JMX](https://docs.oracle.com/javase/8/docs/technotes/guides/visualvm/jmx_connections.html)等工具将自身暴露给监控者，这些工具反过来为VisualVM等工具提供API以获取分析数据。

jstatd程序是一个与JDK捆绑在一起的守护进程，但是，它的功能有限。例如，我们无法监控CPU使用率，也无法进行线程转储。

另一方面，JMX技术不需要在JVM上运行任何守护进程。此外，它可用于分析本地和远程JVM应用程序。但是，我们确实需要使用特殊属性启动JVM以启用开箱即用的监控功能。在本文中，我们将只关注JMX模式。

### 3.2 启动

正如我们之前看到的，我们的JDK版本可能与VisualVM捆绑在一起，也可能不与VisualVM捆绑在一起。无论哪种情况，我们都可以通过执行适当的二进制文件来启动它：

```shell
./jvisualvm
```

如果二进制文件存在于$JAVA_HOME/bin文件夹中，则上述命令将打开VisualVM界面，如果单独安装，它可能位于不同的文件夹中。

默认情况下，VisualVM将启动并加载所有在本地运行的Java应用程序：

![](/assets/images/2025/javajvm/visualvmjmxremote01.png)

### 3.3 功能

VisualVM提供了几个有用的功能：

-   显示本地和远程Java应用程序进程
-   监控CPU使用率、GC活动、已加载类的数量和其他指标方面的进程性能
-   可视化所有进程中的线程以及它们处于不同状态(例如睡眠和等待)的时间
-   获取并显示线程转储，以便立即了解被监控进程中发生的情况

VisualVM[功能页面](https://visualvm.github.io/features.html)有一个更全面的可用功能列表。与所有设计良好的软件一样，VisualVM可以通过安装Plugins选项卡上可用的第三方插件来[扩展](https://visualvm.github.io/plugins.html)以访问更高级和独特的功能。

## 4. 远程监控

在本节中，我们将演示如何使用VisualVM和JMX远程监控Java应用程序，我们还将有机会探索所有必要的配置和JVM启动选项。

### 4.1 应用程序配置

我们使用启动脚本启动大多数Java应用程序，在此脚本中，启动命令通常将基本参数传递给JVM以指定应用程序的需求，例如最大和最小内存要求。

假设我们有一个打包为MyApp.jar的应用程序，让我们看一个包含主要JMX配置参数的示例启动命令：

```shell
java -Dcom.sun.management.jmxremote.port=8080 
-Dcom.sun.management.jmxremote.ssl=false 
-Dcom.sun.management.jmxremote.authenticate=false
-Xms1024m -Xmx1024m -jar MyApp.jar
```

在上面的命令中，MyApp.jar通过端口8080配置的开箱即用的监控功能启动。此外，为简单起见，我们停用了SSL加密和密码身份验证。

在生产环境中，理想情况下，我们应该在公共网络中保护VisualVM和JVM应用程序之间的通信。

### 4.2 VisualVM配置

现在我们已经在本地运行了VisualVM并且在远程服务器上运行了MyApp.jar，我们可以开始远程监控会话了。

右键单击左侧面板并选择“Add JMX Connection”：

![](/assets/images/2025/javajvm/visualvmjmxremote02.png)

在出现的对话框的“Connection”字段中输入host:port组合，然后单击“OK”。

如果成功，我们现在应该能够通过双击左侧面板中的新连接来看到监控窗口：

![](/assets/images/2025/javajvm/visualvmjmxremote03.png)

## 5. 总结

在本文中，我们探讨了使用VisualVM和JMX对Java应用程序进行远程监控。