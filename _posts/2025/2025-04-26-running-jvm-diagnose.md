---
layout: post
title:  诊断正在运行的JVM
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

**Java虚拟机(JVM)是使计算机能够运行Java程序的虚拟机**，在本文中，我们将了解如何轻松诊断正在运行的JVM。

JDK本身就有很多[工具](https://docs.oracle.com/en/java/javase/11/tools/jconsole.html)可用于各种开发、监控和故障排除活动。我们来看一下jcmd，它非常易于使用，并且可以提供有关正在运行的JVM的各种信息。**此外，从JDK 7开始，jcmd是推荐的工具，它可以增强JVM诊断功能，并且不会或最大限度地减少性能开销**。

## 2. 什么是jcmd？

这是一个向正在运行的JVM发送诊断命令请求的实用程序，**但是，它必须在运行JVM的同一台计算机上使用**。更多详细信息，请参阅其[文档](https://docs.oracle.com/en/java/javase/11/tools/jcmd.html#GUID-59153599-875E-447D-8D98-0078A5778F05)。

让我们看看如何将此实用程序与服务器上运行的示例Java应用程序一起使用。

## 3. 如何使用jcmd？

让我们使用JDK 11上的[Spring Initializr](https://start.spring.io/)创建一个快速演示Web应用程序，现在，让我们启动服务器并使用jcmd对其进行诊断。

### 3.1 获取PID

我们知道每个进程都有一个关联的进程ID，称为PID。因此，为了获取应用程序的关联PID，我们可以使用jcmd，它将列出所有适用的Java进程，如下所示：

```shell
root@c6b47b129071:/# jcmd
65 jdk.jcmd/sun.tools.jcmd.JCmd
18 /home/pgm/demo-0.0.1-SNAPSHOT.jar
root@c6b47b129071:/# 
```

**在这里，我们可以看到正在运行的应用程序的PID是18**。

### 3.2 获取可能的jcmd用途列表

首先让我们了解一下jcmd PID help命令可用的选项：

```shell
root@c6b47b129071:/# jcmd 18 help
18:
The following commands are available:
Compiler.CodeHeap_Analytics
Compiler.codecache
Compiler.codelist
Compiler.directives_add
Compiler.directives_clear
Compiler.directives_print
Compiler.directives_remove
Compiler.queue
GC.class_histogram
GC.class_stats
GC.finalizer_info
GC.heap_dump
GC.heap_info
GC.run
GC.run_finalization
JFR.check
JFR.configure
JFR.dump
JFR.start
JFR.stop
JVMTI.agent_load
JVMTI.data_dump
ManagementAgent.start
ManagementAgent.start_local
ManagementAgent.status
ManagementAgent.stop
Thread.print
VM.class_hierarchy
VM.classloader_stats
VM.classloaders
VM.command_line
VM.dynlibs
VM.flags
VM.info
VM.log
VM.metaspace
VM.native_memory
VM.print_touched_methods
VM.set_flag
VM.stringtable
VM.symboltable
VM.system_properties
VM.systemdictionary
VM.uptime
VM.version
help
```

不同版本的HotSpot VM可用的诊断命令可能不同。

## 4. jcmd命令

让我们探索一些最有用的jcmd命令选项来诊断我们正在运行的JVM。

### 4.1 VM.version

这是为了获取JVM基本详细信息，如下所示：

```shell
root@c6b47b129071:/# jcmd 18 VM.version
18:
OpenJDK 64-Bit Server VM version 11.0.11+9-Ubuntu-0ubuntu2.20.04
JDK 11.0.11
root@c6b47b129071:/# 
```

在这里我们可以看到正在使用OpenJDK 11用于示例应用程序。

### 4.2 VM.system_properties

这将打印虚拟机的所有系统属性设置，可能会显示数百行信息：

```shell
root@c6b47b129071:/# jcmd 18 VM.system_properties
18:
#Thu Jul 22 10:56:13 IST 2021
awt.toolkit=sun.awt.X11.XToolkit
java.specification.version=11
sun.cpu.isalist=
sun.jnu.encoding=ANSI_X3.4-1968
java.class.path=/home/pgm/demo-0.0.1-SNAPSHOT.jar
java.vm.vendor=Ubuntu
sun.arch.data.model=64
catalina.useNaming=false
java.vendor.url=https\://ubuntu.com/
user.timezone=Asia/Kolkata
java.vm.specification.version=11
...
```

### 4.3 VM.flags

对于我们的示例应用程序，这将打印所有使用的VM参数，这些参数要么是我们指定的，要么是JVM默认使用的。在这里，我们可以注意到各种默认的VM参数，如下所示：

```shell
root@c6b47b129071:/# jcmd 18 VM.flags            
18:
-XX:CICompilerCount=3 -XX:CompressedClassSpaceSize=260046848 -XX:ConcGCThreads=1 -XX:G1ConcRefinementThreads=4 -XX:G1HeapRegionSize=1048576 -XX:GCDrainStackTargetSize=64 -XX:InitialHeapSize=536870912 -XX:MarkStackSize=4194304 -XX:MaxHeapSize=536870912 -XX:MaxMetaspaceSize=268435456 -XX:MaxNewSize=321912832 -XX:MinHeapDeltaBytes=1048576 -XX:NonNMethodCodeHeapSize=5830732 -XX:NonProfiledCodeHeapSize=122913754 -XX:ProfiledCodeHeapSize=122913754 -XX:ReservedCodeCacheSize=251658240 -XX:+SegmentedCodeCache -XX:ThreadStackSize=256 -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:+UseFastUnorderedTimeStamps -XX:+UseG1GC 
root@c6b47b129071:/#
```

类似地，其他命令，如VM.command_line、VM.uptime、VM.dynlibs也提供了有关所使用的各种其他属性的其他基本和有用的详细信息。

以上所有命令主要是为了获取不同的JVM相关详细信息，现在，让我们再看看一些有助于排除JVM相关故障的命令。

### 4.4 Thread.print

此命令用于获取即时线程转储，因此，它将打印所有正在运行的线程的堆栈跟踪。以下是使用方法，根据正在使用的线程数量，输出可能会比较长：

```shell
root@c6b47b129071:/# jcmd 18 Thread.print
18:
2021-07-22 10:58:08
Full thread dump OpenJDK 64-Bit Server VM (11.0.11+9-Ubuntu-0ubuntu2.20.04 mixed mode, sharing):

Threads class SMR info:
_java_thread_list=0x00007f21cc0028d0, length=25, elements={
0x00007f2210244800, 0x00007f2210246800, 0x00007f221024b800, 0x00007f221024d800,
0x00007f221024f800, 0x00007f2210251800, 0x00007f2210253800, 0x00007f22102ae800,
0x00007f22114ef000, 0x00007f21a44ce000, 0x00007f22114e3800, 0x00007f221159d000,
0x00007f22113ce800, 0x00007f2210e78800, 0x00007f2210e7a000, 0x00007f2210f20800,
0x00007f2210f22800, 0x00007f2210f24800, 0x00007f2211065000, 0x00007f2211067000,
0x00007f2211069000, 0x00007f22110d7800, 0x00007f221122f800, 0x00007f2210016000,
0x00007f21cc001000
}

"Reference Handler" #2 daemon prio=10 os_prio=0 cpu=2.32ms elapsed=874.34s tid=0x00007f2210244800 nid=0x1a waiting on condition  [0x00007f221452a000]
   java.lang.Thread.State: RUNNABLE
	at java.lang.ref.Reference.waitForReferencePendingList(java.base@11.0.11/Native Method)
	at java.lang.ref.Reference.processPendingReferences(java.base@11.0.11/Reference.java:241)
	at java.lang.ref.Reference$ReferenceHandler.run(java.base@11.0.11/Reference.java:213)

"Finalizer" #3 daemon prio=8 os_prio=0 cpu=0.32ms elapsed=874.34s tid=0x00007f2210246800 nid=0x1b in Object.wait()  [0x00007f22144e9000]
   java.lang.Thread.State: WAITING (on object monitor)
	at java.lang.Object.wait(java.base@11.0.11/Native Method)
	- waiting on <0x00000000f7330898> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@11.0.11/ReferenceQueue.java:155)
	- waiting to re-lock in wait() <0x00000000f7330898> (a java.lang.ref.ReferenceQueue$Lock)
	at java.lang.ref.ReferenceQueue.remove(java.base@11.0.11/ReferenceQueue.java:176)
	at java.lang.ref.Finalizer$FinalizerThread.run(java.base@11.0.11/Finalizer.java:170)

"Signal Dispatcher" #4 daemon prio=9 os_prio=0 cpu=0.40ms elapsed=874.33s tid=0x00007f221024b800 nid=0x1c runnable  [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE
```

有关使用其他选项捕获线程转储的详细讨论可以在[这里](https://www.baeldung.com/java-thread-dump)找到。

### 4.5 GC.class_histogram

让我们使用另一个jcmd命令，它将提供有关堆使用情况的重要信息，此外，它还会列出所有具有多个实例的类(外部类或特定于应用程序的类)。同样，根据正在使用的类的数量，列表可能会有数百行：

```shell
root@c6b47b129071:/# jcmd 18 GC.class_histogram
18:
 num     #instances         #bytes  class name (module)
-------------------------------------------------------
   1:         41457        2466648  [B (java.base@11.0.11)
   2:         38656         927744  java.lang.String (java.base@11.0.11)
   3:          6489         769520  java.lang.Class (java.base@11.0.11)
   4:         21497         687904  java.util.concurrent.ConcurrentHashMap$Node (java.base@11.0.11)
   5:          6570         578160  java.lang.reflect.Method (java.base@11.0.11)
   6:          6384         360688  [Ljava.lang.Object; (java.base@11.0.11)
   7:          9668         309376  java.util.HashMap$Node (java.base@11.0.11)
   8:          7101         284040  java.util.LinkedHashMap$Entry (java.base@11.0.11)
   9:          3033         283008  [Ljava.util.HashMap$Node; (java.base@11.0.11)
  10:          2919         257000  [I (java.base@11.0.11)
  11:           212         236096  [Ljava.util.concurrent.ConcurrentHashMap$Node; (java.base@11.0.11)
```

但是，如果这还不能提供清晰的信息，我们可以获取堆转储，接下来我们来看一下。

### 4.6 GC.heap_dump

此命令将立即提供[JVM堆转储](https://www.baeldung.com/java-heap-dump-capture)，因此，我们可以将堆转储提取到文件中，以便稍后进行分析，如下所示：

```shell
root@c6b47b129071:/# jcmd 18 GC.heap_dump ./demo_heap_dump
18:
Heap dump file created
root@c6b47b129071:/# 
```

这里，demo_heap_dump是堆转储文件名。此外，它将创建在应用程序jar所在的同一位置。

### 4.7 JFR命令选项

在我们之前的[文章](https://www.baeldung.com/java-flight-recorder-monitoring)中，我们讨论了使用JFR和JMC进行Java应用程序监控。现在，让我们研究一下可以用来分析应用程序性能问题的jcmd命令。

[JFR(Java Flight Recorder)](https://docs.oracle.com/en/java/javase/11/troubleshoot/diagnostic-tools.html#GUID-D38849B6-61C7-4ED6-A395-EA4BC32A9FD6)是JDK内置的分析和事件收集框架，JFR使我们能够收集有关JVM和Java应用程序行为的详细底层信息。此外，我们还可以使用[JMC](https://docs.oracle.com/en/java/javase/11/troubleshoot/diagnostic-tools.html#GUID-3FA1CF76-96FA-41A6-8F38-DC83171EE834)来可视化JFR收集的数据。因此，JFR和JMC共同构成了一个完整的工具链，用于持续收集底层和详细的运行时信息。

虽然如何使用JMC不在本文讨论范围内，但我们将了解如何使用jcmd创建JFR文件。JFR是一项商业功能，因此，默认情况下，它是禁用的。但是，可以使用“jcmd PID VM.unlock_commercial_features”启用它。

但是，我们在本文中使用的是OpenJDK，因此，我们已启用JFR。现在，让我们使用jcmd命令生成一个JFR文件，如下所示：

```shell
root@c6b47b129071:/# jcmd 18 JFR.start name=demo_recording settings=profile delay=10s duration=20s filename=./demorecording.jfr
18:
Recording 1 scheduled to start in 10 s. The result will be written to:

/demorecording.jfr
root@c6b47b129071:/# jcmd 18 JFR.check
18:
Recording 1: name=demo_recording duration=20s (delayed)
root@c6b47b129071:/# jcmd 18 JFR.check
18:
Recording 1: name=demo_recording duration=20s (running)
root@c6b47b129071:/# jcmd 18 JFR.check
18:
Recording 1: name=demo_recording duration=20s (stopped)
```

我们在jar应用程序所在的位置创建了一个名为demorecording.jfr的示例JFR录制文件，此外，此录制时长为20秒，并根据需求进行了配置。

**此外，我们可以使用JFR.check命令检查JFR录制的状态。此外，我们可以使用JFR.stop命令立即停止并丢弃录制。另一方面，我们可以使用JFR.dump命令立即停止并转储录制**。

### 4.8 VM.native_memory

这是最好的命令之一，它可以提供有关JVM上堆和非堆内存的大量有用详细信息，因此，它可以用于调整内存使用情况并检测任何内存泄漏。众所周知，JVM内存大致可以分为堆内存和非堆内存，为了获取完整的JVM内存使用情况详细信息，我们可以使用此实用程序。此外，它还有助于定义基于容器的应用程序的内存大小。

**要使用此功能，我们需要重启应用程序并添加额外的虚拟机参数，例如XX:NativeMemoryTracking=summary或-XX:NativeMemoryTracking=detail。请注意，启用NMT会导致5%-10%的性能开销**。

这将为我们提供一个新的PID来诊断：

```shell
root@c6b47b129071:/# jcmd 19 VM.native_memory
19:

Native Memory Tracking:

Total: reserved=1159598KB, committed=657786KB
-                 Java Heap (reserved=524288KB, committed=524288KB)
                            (mmap: reserved=524288KB, committed=524288KB) 
 
-                     Class (reserved=279652KB, committed=29460KB)
                            (classes #6425)
                            (  instance classes #5960, array classes #465)
                            (malloc=1124KB #15883) 
                            (mmap: reserved=278528KB, committed=28336KB) 
                            (  Metadata:   )
                            (    reserved=24576KB, committed=24496KB)
                            (    used=23824KB)
                            (    free=672KB)
                            (    waste=0KB =0.00%)
                            (  Class space:)
                            (    reserved=253952KB, committed=3840KB)
                            (    used=3370KB)
                            (    free=470KB)
                            (    waste=0KB =0.00%)
 
-                    Thread (reserved=18439KB, committed=2699KB)
                            (thread #35)
                            (stack: reserved=18276KB, committed=2536KB)
                            (malloc=123KB #212) 
                            (arena=39KB #68)
 
-                      Code (reserved=248370KB, committed=12490KB)
                            (malloc=682KB #3839) 
                            (mmap: reserved=247688KB, committed=11808KB) 
 
-                        GC (reserved=62483KB, committed=62483KB)
                            (malloc=10187KB #7071) 
                            (mmap: reserved=52296KB, committed=52296KB) 
 
-                  Compiler (reserved=146KB, committed=146KB)
                            (malloc=13KB #307) 
                            (arena=133KB #5)
 
-                  Internal (reserved=460KB, committed=460KB)
                            (malloc=428KB #1421) 
                            (mmap: reserved=32KB, committed=32KB) 
 
-                     Other (reserved=16KB, committed=16KB)
                            (malloc=16KB #3) 
 
-                    Symbol (reserved=6593KB, committed=6593KB)
                            (malloc=6042KB #72520) 
                            (arena=552KB #1)
 
-    Native Memory Tracking (reserved=1646KB, committed=1646KB)
                            (malloc=9KB #113) 
                            (tracking overhead=1637KB)
 
-        Shared class space (reserved=17036KB, committed=17036KB)
                            (mmap: reserved=17036KB, committed=17036KB) 
 
-               Arena Chunk (reserved=185KB, committed=185KB)
                            (malloc=185KB) 
 
-                   Logging (reserved=4KB, committed=4KB)
                            (malloc=4KB #191) 
 
-                 Arguments (reserved=18KB, committed=18KB)
                            (malloc=18KB #489) 
 
-                    Module (reserved=124KB, committed=124KB)
                            (malloc=124KB #1521) 
 
-              Synchronizer (reserved=129KB, committed=129KB)
                            (malloc=129KB #1089) 
 
-                 Safepoint (reserved=8KB, committed=8KB)
                            (mmap: reserved=8KB, committed=8KB) 
```

在这里，除了Java Heap Memory之外，我们还可以注意到有关不同内存类型的细节，Class定义了用于存储类元数据的JVM内存。类似地，Thread定义了我们的应用程序线程正在使用的内存。Code提供了用于存储JIT生成代码的内存，Compiler本身占用了一些空间，GC也占用了一些空间。

此外，reserved可以估算出应用程序所需的内存，而commit则显示了已分配的最小内存。

## 5. 诊断内存泄漏

让我们看看如何识别JVM中是否存在内存泄漏；因此，首先我们需要有一个基准，然后需要监控一段时间，以了解上述任何内存类型的内存是否存在持续增长。

我们首先来了解一下JVM内存使用情况，如下所示：

```shell
root@c6b47b129071:/# jcmd 19 VM.native_memory baseline
19:
Baseline succeeded
```

现在，请将该应用程序正常或频繁使用一段时间。最后，使用diff来识别自基线以来的变化，如下所示：

```shell
root@c6b47b129071:/# jcmd 19 VM.native_memory summary.diff
19:

Native Memory Tracking:

Total: reserved=1162150KB +2540KB, committed=660930KB +3068KB

-                 Java Heap (reserved=524288KB, committed=524288KB)
                            (mmap: reserved=524288KB, committed=524288KB)
 
-                     Class (reserved=281737KB +2085KB, committed=31801KB +2341KB)
                            (classes #6821 +395)
                            (  instance classes #6315 +355, array classes #506 +40)
                            (malloc=1161KB +37KB #16648 +750)
                            (mmap: reserved=280576KB +2048KB, committed=30640KB +2304KB)
                            (  Metadata:   )
                            (    reserved=26624KB +2048KB, committed=26544KB +2048KB)
                            (    used=25790KB +1947KB)
                            (    free=754KB +101KB)
                            (    waste=0KB =0.00%)
                            (  Class space:)
                            (    reserved=253952KB, committed=4096KB +256KB)
                            (    used=3615KB +245KB)
                            (    free=481KB +11KB)
                            (    waste=0KB =0.00%)
 
-                    Thread (reserved=18439KB, committed=2779KB +80KB)
                            (thread #35)
                            (stack: reserved=18276KB, committed=2616KB +80KB)
                            (malloc=123KB #212)
                            (arena=39KB #68)
 
-                      Code (reserved=248396KB +21KB, committed=12772KB +213KB)
                            (malloc=708KB +21KB #3979 +110)
                            (mmap: reserved=247688KB, committed=12064KB +192KB)
 
-                        GC (reserved=62501KB +16KB, committed=62501KB +16KB)
                            (malloc=10205KB +16KB #7256 +146)
                            (mmap: reserved=52296KB, committed=52296KB)
 
-                  Compiler (reserved=161KB +15KB, committed=161KB +15KB)
                            (malloc=29KB +15KB #341 +34)
                            (arena=133KB #5)
 
-                  Internal (reserved=495KB +35KB, committed=495KB +35KB)
                            (malloc=463KB +35KB #1429 +8)
                            (mmap: reserved=32KB, committed=32KB)
 
-                     Other (reserved=52KB +36KB, committed=52KB +36KB)
                            (malloc=52KB +36KB #9 +6)
 
-                    Symbol (reserved=6846KB +252KB, committed=6846KB +252KB)
                            (malloc=6294KB +252KB #76359 +3839)
                            (arena=552KB #1)
 
-    Native Memory Tracking (reserved=1727KB +77KB, committed=1727KB +77KB)
                            (malloc=11KB #150 +2)
                            (tracking overhead=1716KB +77KB)
 
-        Shared class space (reserved=17036KB, committed=17036KB)
                            (mmap: reserved=17036KB, committed=17036KB)
 
-               Arena Chunk (reserved=186KB, committed=186KB)
                            (malloc=186KB)
 
-                   Logging (reserved=4KB, committed=4KB)
                            (malloc=4KB #191)
 
-                 Arguments (reserved=18KB, committed=18KB)
                            (malloc=18KB #489)
 
-                    Module (reserved=124KB, committed=124KB)
                            (malloc=124KB #1528 +7)
 
-              Synchronizer (reserved=132KB +3KB, committed=132KB +3KB)
                            (malloc=132KB +3KB #1111 +22)
 
-                 Safepoint (reserved=8KB, committed=8KB)
                            (mmap: reserved=8KB, committed=8KB)
```

随着GC运行时间的推移，我们会注意到内存使用量时增时减。但是，如果内存使用量出现不受控制的增长，则可能存在内存泄漏问题。因此，我们可以从这些统计数据中识别内存泄漏区域，例如Heap、Thread、Code、Class等。如果我们的应用程序需要更多内存，我们可以分别调整相应的虚拟机参数。

**如果内存泄漏发生在堆(Heap)中，我们可以进行堆转储(如前所述)或者仅仅调整Xmx。同样，如果内存泄漏发生在线程(Thread)中，我们可以查找未处理的递归指令或调整Xss**。

## 6. 总结

在本文中，我们介绍了一种用于诊断不同场景的JVM的实用程序。

我们还介绍了jcmd命令及其各种用法，用于获取堆转储、线程转储和JFR记录，以进行各种性能相关的分析。最后，我们还研究了一种使用jcmd诊断内存泄漏的方法。