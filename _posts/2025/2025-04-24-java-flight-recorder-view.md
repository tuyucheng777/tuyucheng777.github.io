---
layout: post
title:  Java中的JFR View命令
category: java-new
copyright: java-new
excerpt: Java 21
---

## 1. 简介

[Java Flight Recorder(JFR)](https://www.baeldung.com/java-flight-recorder-monitoring)是一款[性能分析](https://www.baeldung.com/java-profilers)和诊断工具，用于监控[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)及其上运行的程序，它是一款方便的分析工具，开发人员可以使用它来监控其应用程序的状态和性能。

本教程将重点介绍Java 21中为JFR引入的新view命令。

## 2. Java Flight Recorder(JFR)

Java Flight Recorder(JFR)是Java 7中作为一项实验性功能引入的低开销应用程序分析框架，它使我们能够分析和了解有关程序的重要指标，例如[垃圾回收](https://www.baeldung.com/jvm-garbage-collectors)模式、IO操作、[内存分配](https://www.baeldung.com/java-stack-heap)等。

### 2.1 什么是Java Flight Recorder？

JFR收集有关Java应用程序运行时JVM中的事件的信息，我们使用诊断工具分析结果。

JFR监视应用程序并将分析结果记录在文件中，我们可以通过两种方式指示JFR监视应用程序：

1. 使用命令行启动应用程序并启用JFR。
2. 在已运行的Java应用程序上使用[jcmd](https://www.baeldung.com/running-jvm-diagnose#what-is-jcmd)等诊断工具。

**记录通常生成为.jfr文件，然后可以通过Java Mission Control(JMC)等可视化工具或使用新的view命令进行分析，我们将在后面的部分中看到**。

### 2.2 从命令行录制

为了演示JFR，让我们编写一个小程序，将对象插入到ArrayList中，直到引发OutOfMemoryError：
```java
void insertToList(List<Object> list) {
    try {
        while (true) {
            list.add(new Object());
        }
    } catch (OutOfMemoryError e) {
        System.out.println("Out of Memory. Exiting");
    }
}
```

让我们使用标准javac编译器编译该程序：
```shell
javac -d out -sourcepath JFRExample.java
```

生成.class文件后，我们启动JFR，我们向java命令传递一些附加参数，即-XX:+FlightRecorder选项，以及一些附加参数来设置记录持续时间和存储记录的输出文件路径：
```shell
java -XX:StartFlightRecording=duration=200s,filename=flight.jfr -cp ./out/ cn.tuyucheng.taketoday.jfrview.JFRExample
```

我们的程序现在在启用JFR的情况下运行，以捕获事件和其他系统属性，并且JFR将结果写入flight.jfr文件中。

### 2.3 使用jcmd工具进行录制

**jcmd诊断工具提供了另一种方法来记录和分析应用程序和JVM的性能**，我们可以使用此工具将诊断事件注册到正在运行的虚拟机。

要使用jcmd工具，我们需要我们的应用程序正在运行，并且我们必须知道pid：
```shell
jcmd <pid|MainClass> <command> [parameters]
```

jcmd工具可以识别的几个命令是：

- **JFR.start**：开始新的JFR录制
- **JFR.check**：检查正在运行的JFR记录
- **JFR.stop**：停止特定的JFR记录
- **JFR.dump**：将JFR记录的内容复制到文件

每个命令都需要额外的参数。

现在让我们使用jcmd工具创建一个记录，我们需要启动应用程序并找到正在运行的进程的pid，获取到pid后，我们就可以开始记录：
```shell
jcmd 128263 JFR.start filename=recording.jfr
```

当我们捕获到相关事件后，我们可以停止录制：
```shell
jcmd 128263 JFR.stop filename=recording.jfr
```

## 3. 查看JFR录制文件

要查看和理解jfr文件的结果，我们可以使用Java Mission Control(JMC)工具，JMC工具具有多种功能来分析和监视Java应用程序，**其中包括一个读取JFR文件并显示结果可视化表示的诊断工具**：

![](/assets/images/2025/javanew/javaflightrecorderview01.png)

## 4. jfr命令

jfr命令解析并将Java Flight Recorder文件(.jfr)打印到标准输出，虽然我们之前使用Mission Control工具进行可视化表示，但jfr命令为我们提供了一种在控制台中过滤、汇总和生成JFR文件的人性化输出的方法。

### 4.1 使用jfr命令

jfr命令位于$JAVA_HOME的bin路径下，我们来看看它的语法：
```shell
$JAVA_HOME/bin/jfr [command] <path>
```

在以下部分中，我们将直接访问jfr。

### 4.2 jfr命令

jfr之前有5个命令，分别是print、summary、metadata、assemble和disassemble，**view命令是新引入的第6个jfr命令**。

print命令用于打印JFR记录的内容，它需要几个参数，包括输出格式(json/xml等)、一系列可能包括类别、事件和堆栈深度的过滤器：
```shell
jfr print [--xml|--json] [--categories <filters>] [--events <filters>] [--stack-depth <depth>] <file>
```

顾名思义，summary命令会生成记录摘要，其中包括发生的事件、磁盘空间使用情况等：
```shell
jfr summary <file>
```

metadata命令生成有关事件的详细信息，例如事件的名称和类别：
```shell
jfr metadata <file>
```

最后，assemble和disassemble命令用于将块文件组装成记录文件，反之亦然：
```shell
jfr assemble <repository> <file>
jfr disassemble [--max-chunks <chunks>] [--output <directory>] <file>
```

### 4.3 jfr命令示例

现在我们将查看一个示例jfr命令并生成我们的JFR文件的摘要：
```shell
$JAVA_HOME/bin/jfr summary recording.jfr

 Version: 2.1
 Chunks: 1
 Start: 2023-12-25 17:07:08 (UTC)
 Duration: 1 s

 Event Type                              Count  Size (bytes)
=============================================================
 jdk.NativeLibrary                         512         44522
 jdk.SystemProcess                         511         49553
 jdk.ModuleExport                          506          4921
 jdk.BooleanFlag                           495         15060
 jdk.ActiveSetting                         359         10376
 jdk.GCPhaseParallel                       162          4033
```

## 5. JDK 21中的view命令

Java 21引入了view命令，以方便从命令行分析JFR记录。**这个新的view命令消除了将记录下载转储到JMC工具中的使用，并附带了70多个预建选项**。这些选项(可能会随着时间的推移而增加)涵盖了应用程序的几乎所有重要方面，包括JVM、应用程序本身以及应用程序的环境。

### 5.1 view选项类别

我们可以将不同的view选项大致分为三类，与JMC工具中显示的类似：

1. Java虚拟机视图
2. 环境视图
3. 应用程序视图

### 5.2 JVM视图

Java虚拟机视图可以深入了解JVM属性，例如堆空间、垃圾回收、本机内存和其他与编译器相关的指标，一些常见的JVM视图包括：

- class-modifications
- gc-concurrent-phases
- compiler-configuration
- gc-configuration
- native-memory-committed
- gc-cpu-time
- compiler-statistics
- gc-pause-phases
- heap-configuration

### 5.3 环境视图

环境视图显示有关运行JVM的主机系统的信息，例如CPU信息、网络利用率、系统属性、进程和信息，一些常见的环境视图包括：

- cpu-information
- cpu-load
- jvm-flags
- container-configuration
- network-utilization
- system-processes
- system-properties

### 5.4 应用程序视图

应用程序视图可深入了解我们的应用程序代码，例如有关其线程使用情况、对象统计信息和内存利用率的信息，一些常见的应用程序视图包括：

- exception-count
- object-statistics
- allocation-by-thread
- memory-leaks-by-class
- thread-cpu-load
- thread-count
- thread-allocation

### 5.5 view的命令结构

**view命令需要视图名称和记录文件的路径以及相关参数**，可以使用jfr命令和jcmd命令激活它：
```shell
jfr view [view] <recording file>
```

```shell
jcmd <pid> JFR.view [view name]
```

然后，输出将显示在命令行或任何标准输出中。**此外，这些命令还提供输出格式、时间范围和自定义视图的自定义功能**。

## 6. JFR view命令用法

在本节中，我们将使用view命令和上一节中提到的一些视图来分析我们之前生成的JFR记录文件。

### 6.1 使用jfr命令

让我们使用jfr命令在我们的JFR记录上应用gc-configuration JVM视图并捕获输出：
```shell
jfr view gc-configuration flight.jfr
```

```text
GC Configuration
----------------
Young GC: G1New
Old GC: G1Old
Parallel GC Threads: 4
Concurrent GC Threads: 1
Dynamic GC Threads: true
Concurrent Explicit GC: false
Disable Explicit GC: false
Pause Target: N/A
GC Time Ratio: 12
```

正如其名称所示，该视图会生成有关正在使用的GC类型以及其他垃圾回收相关数据的信息。

### 6.2 使用jcmd工具

我们也可以将view命令与jcmd工具一起使用：
```shell
jcmd <pid> JFR.view [view name]
```

jcmd工具需要正在运行的pid来诊断，我们将请求环境视图system-processes：
```shell
jcmd 37417 JFR.view cell-height=3 truncate=beginning width=80 system-processes
                                                    System Processes
First Observed Last Observed PID  Command Line
-------------- ------------- ---- -------------------------------------------------------------------------------------
23:21:26       23:21:26      453  /Applications/Flycut.app/Contents/MacOS/Flycut
23:21:26       23:21:26      780  /Applications/Grammarly Desktop.app/Contents/Library/LaunchAgents/Grammarly Deskto...
23:21:26       23:21:26      455  /Applications/Grammarly Desktop.app/Contents/MacOS/Grammarly Desktop
23:21:26       23:21:26      431  /Applications/JetBrains Toolbox.app/Contents/MacOS/jetbrains-toolbox
23:21:26       23:21:26      624  /Applications/Safari.app/Contents/MacOS/Safari
```

### 6.3 格式化view命令输出

我们可以尝试一下view命令的输出，并根据需要进行调整。**view命令目前提供了几个选项来格式化输出**，包括修改列数和行数，以及其他选项，如详细标志和输出截断：
```shell
--width [number-of-columns]
--cell-height [number of rows]
--verbose [the query that makes up the view]
--truncate [--beginning|--end] [truncate content from beginning or end]
```

举个例子，让我们格式化上面生成的system-processes视图的输出，使得每行有两行，并使响应详细：
```shell
jfr view --cell-height 2 --width 100 --truncate beginning --verbose system-processes flight.jfr

                                          System Processes

First Observed Last Observed PID   Command Line
(startTime)    (startTime)   (pid) (commandLine)
-------------- ------------- ----- ----------------------------------------------------------------
23:33:47       23:33:47      453   /Applications/Flycut.app/Contents/MacOS/Flycut
23:33:47       23:33:47      780   ...ions/Grammarly Desktop.app/Contents/Library/LaunchAgents/Gram
                                   marly Desktop Helper.app/Contents/MacOS/Grammarly Desktop Helper
23:33:47       23:33:47      455   /Applications/Grammarly Desktop.app/Contents/MacOS/Grammarly Des
                                   ktop
23:33:47       23:33:47      431   /Applications/JetBrains Toolbox.app/Contents/MacOS/jetbrains-too
                                   lbox
23:33:47       23:33:47      624   /Applications/Safari.app/Contents/MacOS/Safari


COLUMN 'First Observed', 'Last Observed', 'PID', 'Command Line' SELECT FIRST(startTime),
LAST(startTime), FIRST(pid), FIRST(commandLine) FROM SystemProcess GROUP BY pid

Execution: query-validation=10.6 ms, aggregation=241 ms, formatting=121 ms
```

我们可以看到，verbose命令生成了更多关于系统进程的信息，包括一些元信息，例如查询的解释和执行统计信息，以查看执行花费了多长时间。

### 6.4 与JCM的结果比较

**JFR view命令旨在使分析JFR记录的过程更加简单，并且不依赖于任何其他工具**。JMC可视化输出和基于控制台的view命令输出非常相似，我们可以将system-processes视图的结果与可视化JMC工具的结果进行比较：

![](/assets/images/2025/javanew/javaflightrecorderview02.png)

## 7. 总结

在本文中，我们研究了新添加的JFR view命令，该命令有助于使用一组预定义的视图直接在命令行中显示JFR的结果，我们介绍了目前可用的不同类别的视图以及为JFR生成视图的方式。