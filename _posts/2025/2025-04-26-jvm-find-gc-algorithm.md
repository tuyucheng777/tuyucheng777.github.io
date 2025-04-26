---
layout: post
title:  查找JVM实例使用的GC算法
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

除了编译器和运行时等典型的开发工具外，每个JDK版本都附带了许多其他工具，其中一些工具可以帮助我们深入了解正在运行的应用程序。

在本文中，我们将了解如何使用这些工具来了解有关特定JVM实例使用的[GC算法](https://www.baeldung.com/jvm-garbage-collectors)的更多信息。

## 2. 示例应用程序

在本文中，我们将使用一个非常简单的应用程序：

```java
public class App {
    public static void main(String[] args) throws IOException {
        System.out.println("Waiting for stdin");
        int read = System.in.read();
        System.out.println("I'm done: " + read);
    }
}
```

显然，这个应用程序会等待并持续运行，直到收到来自标准输入的消息,这种暂停机制有助于我们模拟长时间运行的JVM应用程序的行为。

为了使用这个应用程序，我们必须用javac编译App.java文件，然后使用java工具运行它。

## 3. 查找JVM进程

**要查找JVM进程使用的GC，首先，我们应该确定该JVM实例的进程ID**，假设我们使用以下命令运行应用程序：

```shell
>> java App
Waiting for stdin
```

如果安装了JDK，查找JVM实例的进程ID的最佳方法是使用[jps](https://docs.oracle.com/en/java/javase/11/tools/jps.html)工具，例如：

```shell
>> jps -l
69569 
48347 App
48351 jdk.jcmd/sun.tools.jps.Jps
```

如上所示，系统中运行着三个JVM实例。显然，第二个JVM实例的描述(“App”)与我们的应用程序名称匹配，因此，我们要查找的进程ID是48347。

除了jps之外，我们还可以使用其他通用实用程序来过滤正在运行的进程。例如，[procps](https://gitlab.com/procps-ng/procps)包中著名的[ps](https://www.baeldung.com/linux/ps-command)工具也可以使用：

```shell
>> ps -ef | grep java
502 48347 36213   0  1:28AM ttys037    0:00.28 java App
```

然而，jps使用起来更简单，需要的过滤也更少。

## 4. 使用的GC

现在我们知道如何找到进程ID，让我们找到已经运行的JVM应用程序使用的GC算法。

### 4.1 Java 8及更早版本

**如果使用的是Java 8，我们可以使用[jmap](https://docs.oracle.com/en/java/javase/11/tools/jmap.html)实用程序打印堆摘要、堆直方图，甚至生成堆转储**。为了找到GC算法，我们可以使用-heap选项，如下所示：

```shell
>> jmap -heap <pid>
```

因此在我们的特定情况下，我们使用CMS GC：

```shell
>> jmap -heap 48347 | grep GC
Concurrent Mark-Sweep GC
```

对于其他GC算法，输出几乎相同：

```shell
>> jmap -heap 48347 | grep GC
Parallel GC with 8 thread(s)
```

### 4.2 Java 9+：jhsdb jmap

**从Java 9开始，我们可以使用[jhsdb和jmap](https://docs.oracle.com/en/java/javase/11/tools/jps.html)组合来打印一些关于JVM堆的信息**，更具体地说，这个命令等同于上一个命令：

```shell
>> jhsdb jmap --heap --pid <pid>
```

例如，我们的应用程序现在使用G1 GC运行：

```shell
>> jhsdb jmap --heap --pid 48347 | grep GC
Garbage-First (G1) GC with 8 thread(s)
```

### 4.3 Java 9+：jcmd

在现代JVM中，jcmd命令非常灵活。例如，**我们可以使用它来获取有关堆的一些常规信息**：

```shell
>> jcmd <pid> VM.info
```

因此，如果我们传递应用程序的进程ID，我们可以看到这个JVM实例正在使用Serial GC：

```shell
>> jcmd 48347 VM.info | grep gc
# Java VM: OpenJDK 64-Bit Server VM (15+36-1562, mixed mode, sharing, tiered, compressed oops, serial gc, bsd-amd64)
// omitted
```

G1或ZGC的输出类似：

```shell
// ZGC
# Java VM: OpenJDK 64-Bit Server VM (15+36-1562, mixed mode, sharing, tiered, z gc, bsd-amd64)
// G1GC
# Java VM: OpenJDK 64-Bit Server VM (15+36-1562, mixed mode, sharing, tiered, compressed oops, g1 gc, bsd-amd64)
```

通过一点[grep](https://www.baeldung.com/linux/common-text-search)魔法，我们还可以删除所有这些噪音并只获取GC名称：

```shell
>> jcmd 48347 VM.info | grep -ohE "[^\s^,]+\sgc"
g1 gc
```

### 4.4 命令行参数

有时，我们会在启动JVM应用程序时明确指定GC算法，例如，我们在这里选择使用ZGC：

```shell
>> java -XX:+UseZGC App
```

在这种情况下，有更简单的方法来查找已使用的GC。基本上，**我们要做的就是以某种方式找到应用程序执行的命令**。

例如，在基于UNIX的平台上，我们可以再次使用ps命令：

```shell
>> ps -p 48347 -o command=
java -XX:+UseZGC App
```

从上面的输出可以看出，JVM正在使用ZGC。同样，**jcmd命令也可以打印命令行参数**：

```shell
>> jcmd 48347 VM.flags
84020:
-XX:CICompilerCount=4 -XX:-UseCompressedOops -XX:-UseNUMA -XX:-UseNUMAInterleaving -XX:+UseZGC // omitted
```

**令人惊讶的是，如上所示，此命令将打印隐式和显式参数及可调参数**，因此，即使我们没有明确指定GC算法，它也将显示所选的默认算法：

```shell
>> jcmd 48347 VM.flags | grep -ohE '\S*GC\s'
-XX:+UseG1GC
```

更令人惊讶的是，这也适用于Java 8：

```shell
>> jcmd 48347 VM.flags | grep -ohE '\S*GC\s'
-XX:+UseParallelGC
```

## 5. 总结

在本文中，我们了解了查找特定JVM实例所使用的GC算法的不同方法，其中一些方法与特定的Java版本相关，而另一些方法则具有可移植性。

此外，我们还看到了几种查找进程ID的方法，这是始终需要的。