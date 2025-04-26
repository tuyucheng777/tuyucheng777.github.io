---
layout: post
title:  用于查找Java堆大小的命令行工具
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在本快速教程中，我们将熟悉几种获取正在运行的Java应用程序的堆大小的不同方法。

## 2. jcmd

**要查找正在运行的Java应用程序的堆和元空间相关信息，我们可以使用[jcmd](https://docs.oracle.com/en/java/javase/11/tools/jcmd.html)命令行实用程序**：

```shell
jcmd  GC.heap_info
```

首先，让我们使用[jps](https://docs.oracle.com/en/java/javase/11/tools/jps.html)命令找到特定Java应用程序的进程ID：

```shell
$ jps -l
73170 org.jetbrains.idea.maven.server.RemoteMavenServer36
4309  quarkus.jar
12070 sun.tools.jps.Jps
```

如上所示，我们的[Quarkus](https://www.baeldung.com/quarkus-io)应用程序的进程ID是4309，现在我们有了进程ID，让我们看看堆信息：

```shell
$ jcmd 4309 GC.heap_info
4309:
 garbage-first heap   total 206848K, used 43061K
  region size 1024K, 43 young (44032K), 3 survivors (3072K)
 Metaspace       used 12983K, capacity 13724K, committed 13824K, reserved 1060864K
  class space    used 1599K, capacity 1740K, committed 1792K, reserved 1048576K
```

此应用程序使用G1 [GC算法](https://www.baeldung.com/jvm-garbage-collectors)：

- 第一行报告当前堆大小为202MB(206848K)，并且已使用42MB(43061K)。
- G1区域为1MB，其中43个区域标记为年轻区域，3个区域标记为幸存者空间。
- 元空间的当前容量约为13.5MB(13724K)，其中，大约有12.5MB(12983K)已被使用。此外，我们最多可以拥有1GB的元空间(1048576K)。此外，Java虚拟机保证可以使用13842KB的内存，这也称为已提交内存。
- 最后一行显示了有多少元空间用于存储类信息。

**此输出可能会根据GC算法而变化**，例如，如果我们通过“-XX:+UnlockExperimentalVMOptions-XX:+UseZGC”使用[ZGC](https://www.baeldung.com/jvm-zgc-garbage-collector)运行相同的Quarkus应用程序：

```text
ZHeap           used 28M, capacity 200M, max capacity 1024M
Metaspace       used 21031K, capacity 21241K, committed 21504K, reserved 22528K
```

如上所示，我们使用了28MB的堆内存和大约20MB的元空间。截至撰写本文时，Intellij IDEA仍在使用CMS GC，其堆内存信息如下：

```text
par new generation   total 613440K, used 114299K
  eden space 545344K,  18% used
  from space 68096K,  16% used
  to   space 68096K,   0% used
 concurrent mark-sweep generation total 1415616K, used 213479K
 Metaspace       used 423107K, capacity 439976K, committed 440416K, reserved 1429504K
  class space    used 55889K, capacity 62488K, committed 62616K, reserved 1048576K
```

我们可以在堆配置中发现CMS GC的经典代际特性。

## 3. jstat

除了[jcmd](https://www.baeldung.com/java-heap-dump-capture#2-jcmd)之外，我们还可以使用[jstat](https://docs.oracle.com/en/java/javase/11/tools/jstat.html)从正在运行的应用程序中获取相同的信息，例如，我们可以使用jstat-gc查看堆统计信息：

```shell
$ jstat -gc 4309
S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     
0.0    0.0    0.0    0.0   129024.0  5120.0   75776.0    10134.6   20864.0
MU      CCSC   CCSU     YGC     YGCT    FGC    FGCT     CGC    CGCT     GCTGCT
19946.2 2688.0 2355.0    2      0.007    1      0.020    0     0.000     0.027
```

每列代表特定内存区域的内存容量或利用率：

- S0C：第一个幸存者空间的容量
- S1C：第二个幸存者空间的容量
- S0U：第一个幸存者的已用空间
- S1U：第二个幸存者的已用空间
- EC：Eden空间容量
- EU：Eden已使用空间
- OC：老年代容量
- OU：老年代已用空间
- MC：元空间容量
- MU：元空间已用空间
- CCSC：压缩类空间容量
- CCSU：压缩类空间已用空间
- YGC：Minor GC的数量
- YGCT：Minor GC所花费的时间
- FGC：Full GC的数量
- FGCT：Full GC所花费的时间
- CGC：Concurrent GC的数量
- CGCT：Concurrent GC所花费的时间
- GCT：所有GC所花费的时间

jstat还有其他与内存相关的选项，例如：

- -gccapacity报告不同内存区域的不同容量 
- -gcutil仅显示每个区域的利用率百分比
- -gccause与-gcutil相同，但添加了上次GC的原因以及可能的当前GC事件

## 4. 命令行参数

如果我们使用堆配置选项(例如[-Xms和-Xmx](https://www.baeldung.com/jvm-parameters#explicit-heap-memory---xms-and-xmx-options))运行Java应用程序，那么还有一些其他技巧可以找到指定的值。

例如，jps报告这些值的方式如下：

```shell
$ jps -lv
4309 quarkus.jar -Xms200m -Xmx1g
```

**使用这种方法，我们只能找到这些静态值，因此，我们无法了解当前已提交的内存**。

除了jps之外，其他一些工具也会报告同样的情况，例如，“jcmd <pid\> VM.command_line”也会报告以下详细信息：

```shell
$ jcmd 4309 VM.command_line
4309:
VM Arguments:
jvm_args: -Xms200m -Xmx1g
java_command: quarkus.jar
java_class_path (initial): quarkus.jar
Launcher Type: SUN_STANDARD
```

另外，在大多数基于Unix的系统上，我们可以使用procps包中的[ps](https://www.baeldung.com/linux/ps-command)：

```shell
$ ps -ef | grep quarkus
... java -Xms200m -Xmx1g -jar quarkus.jar
```

最后，在Linux上，我们可以使用/proc虚拟文件系统及其pid文件：

```shell
$ cat /proc/4309/cmdline
java -Xms200m -Xmx1g -jar quarkus.jar
```

cmdline文件位于以Quarkus pid命名的目录中，包含应用程序的命令行条目。

## 5. 总结

在本快速教程中，我们看到了几种获取正在运行的Java应用程序的堆大小的不同方法。