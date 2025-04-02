---
layout: post
title:  Java应用程序远程调试
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在多种情况下，调试远程Java应用程序都非常方便。

在本教程中，我们将了解如何使用JDK的工具来做到这一点。

## 2. 用程序

让我们从编写一个应用程序开始，我们将在远程位置运行它，并通过本文在本地进行调试：

```java
public class OurApplication {
    private static String staticString = "Static String";
    private String instanceString;

    public static void main(String[] args) {
        for (int i = 0; i < 1_000_000_000; i++) {
            OurApplication app = new OurApplication(i);
            System.out.println(app.instanceString);
        }
    }

    public OurApplication(int index) {
        this.instanceString = buildInstanceString(index);
    }

    public String buildInstanceString(int number) {
        return number + ". Instance String !";
    }
}
```

然后，我们使用-g标志编译它以包含所有调试信息：

```shell
javac -g OurApplication.java
```

## 3. JDWP：Java调试线协议

**Java Debug Wire Protocol是Java中用于被调试者和调试器之间通信的协议**，被调试者是被调试的应用程序，而调试器是一个应用程序或连接到被调试应用程序的进程。

这两个应用程序可以在同一台机器上运行，也可以在不同的机器上运行，我们将重点讨论后者。

### 3.1 JDWP选项

启动被调试应用程序时，我们将在JVM命令行参数中使用JDWP。

它的调用需要一个选项列表：

-   transport是唯一完全必需的选项，它定义了要使用的传输机制。**dt_shmem仅适用于Windows，并且两个进程在同一台计算机上运行，而dt_socket与所有平台兼容并允许进程在不同机器上运行**。
-   server不是强制选项，此标志启用时，定义它附加到调试器的方式。它要么通过address选项中定义的地址公开进程，否则，JDWP公开一个默认的。
-   suspend定义JVM是否应该挂起并等待调试器连接。
-   address是包含调试器公开的地址(通常是端口)的选项，它还可以表示转换为字符串的地址(如果我们在Windows上使用server=y而不提供地址，则为javadebug)。

### 3.2 启动命令

让我们首先启动远程应用程序，我们将提供前面列出的所有选项：

```shell
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=8000 OurApplication
```

在Java 5之前，JVM参数runjdwp必须与其他选项debug一起使用：

```shell
java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8000
```

这种使用JDWP的方式仍然受支持，但在未来的版本中将被删除。如果可能，我们更倾向于使用较新的符号。

### 3.3 从Java 9开始

最后，随着Java版本9的发布，JDWP的一个选项发生了变化。这是一个相当小的变化，因为它只涉及一个选项，但如果我们尝试调试远程应用程序，它会有所不同。

此更改会影响远程应用程序的address行为方式，旧表示法address=8000仅适用于localhost。为了实现旧行为，我们将使用带冒号的星号作为地址的前缀(例如address=\*:8000)。

根据文档，这并不安全，建议尽可能指定调试器的IP地址：

```shell
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=127.0.0.1:8000
```

## 4. JDB：Java调试器

**JDB(即Java调试器)，是JDK中包含的一个工具，旨在从命令行提供方便的调试器客户端**。

要启动JDB，我们将使用attach模式，此模式将JDB附加到正在运行的JVM。还有其他运行模式，例如listen或run，但在调试本地运行的应用程序时最方便：

```shell
jdb -attach 127.0.0.1:8000
> Initializing jdb ...
```

### 4.1 断点

让我们继续在第1节中介绍的应用程序中放置一些断点。

我们将在构造函数上设置一个断点：

```shell
> stop in OurApplication.<init>
```

我们将在静态方法main中设置另一个，使用String类的完全限定名称：

```shell
> stop in OurApplication.main(java.lang.String[])
```

最后，我们将在实例方法buildInstanceString上设置最后一个：

```shell
> stop in OurApplication.buildInstanceString(int)
```

我们现在应该注意到服务器应用程序停止了，并且在我们的调试器控制台中打印了以下内容：

```text
> Breakpoint hit: "thread=main", OurApplication.<init>(), line=11 bci=0
```

现在让我们在特定行上添加一个断点，即打印变量app.instanceString的行：

```bash
> stop at OurApplication:7
```

我们注意到当在特定行上定义断点时，在stop之后使用at而不是in。

### 4.2 导航和评估

现在我们已经设置了断点，让我们使用cont继续执行线程，直到到达第7行的断点。

我们应该在控制台中看到以下内容：

```shell
> Breakpoint hit: "thread=main", OurApplication.main(), line=7 bci=17
```

提醒一下，我们在包含以下代码的行上停止了：

```java
System.out.println(app.instanceString);
```

也可以通过在main方法上停止并输入2次step来在此行上停止，step执行当前代码行并直接在下一行停止调试器。

现在我们已经停止了，被调试者正在评估我们的staticString、应用程序的instanceString、局部变量i，最后看看如何评估其他表达式。

让我们将staticField打印到控制台：

```shell
> eval OurApplication.staticString
OurApplication.staticString = "Static String"
```

我们明确地将类名放在静态字段之前。

现在让我们打印app的实例字段：

```shell
> eval app.instanceString
app.instanceString = "68741. Instance String !"
```

接下来，让我们看看变量i：

```shell
> print i
i = 68741
```

与其他变量不同，局部变量不需要指定类或实例。我们还可以看到print与eval具有完全相同的行为：它们都对表达式或变量求值。

我们将评估OurApplication的一个新实例，并为其传递一个整数作为构造函数参数：

```shell
> print new OurApplication(10).instanceString
new OurApplication(10).instanceString = "10. Instance String !"
```

现在我们已经评估了所有需要的变量，我们需要删除之前设置的断点，让线程继续处理。为此，我们将使用命令clear，后跟断点的标识符。

该标识符与之前在命令stop中使用的标识符完全相同：

```shell
> clear OurApplication:7
Removed: breakpoint OurApplication:7
```

为了验证断点是否已正确删除，我们将使用不带参数的clear。这将显示现有断点列表，不包括我们刚刚删除的断点：

```shell
> clear
Breakpoints set:
        breakpoint OurApplication.<init>
        breakpoint OurApplication.buildInstanceString(int)
        breakpoint OurApplication.main(java.lang.String[])
```

## 5. 总结

在这篇简短的文章中，我们了解了如何将JDWP与JDB这两种JDK工具一起使用。

当然，有关工具的更多信息可以在它们各自的参考资料[JDWP](https://docs.oracle.com/en/java/javase/11/docs/specs/jdwp/jdwp-spec.html)和[JDB](https://docs.oracle.com/en/java/javase/11/tools/jdb.html)中找到，以更深入地了解工具。