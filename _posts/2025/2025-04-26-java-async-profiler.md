---
layout: post
title:  使用Spring
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

[Java采样分析器](https://www.baeldung.com/java-profilers)通常使用JVM工具接口(JVMTI)设计，并在安全点收集堆栈跟踪。因此，这些采样分析器可能会受到[安全点偏差问题](https://psy-lob-saw.blogspot.com/2016/02/why-most-sampling-java-profilers-are.html)的影响。

为了全面了解应用程序，**我们需要一个采样分析器，它不需要线程处于安全点，并且可以随时收集堆栈跟踪，以避免安全点偏差问题**。

在本教程中，我们将探索[async-profiler](https://github.com/jvm-profiling-tools/async-profiler)及其提供的各种分析技术。

## 2. async-profiler

async-profiler是一个适用于任何基于[HotSpot JVM](https://en.wikipedia.org/wiki/HotSpot)的JDK的采样分析器，它开销低，并且不依赖于JVMTI。

**它通过使用HotSpot JVM提供的AsyncGetCallTrace API来分析Java代码路径，以及使用Linux的perf_events来分析本机代码路径，从而避免了安全点偏差问题**。

换句话说，分析器匹配Java代码和本机代码路径的调用堆栈以产生准确的结果。

## 3. 设置

### 3.1 安装

首先，我们需要根据平台下载最新版本的[async-profiler](https://github.com/jvm-profiling-tools/async-profiler/releases)，目前，它仅支持Linux和macOS平台。

下载完成后，我们可以检查它是否在我们的平台上运行：

```shell
$ ./profiler.sh --version
```

```text
Async-profiler 1.7.1 built on May 14 2020
Copyright 2016-2020 Andrei Pangin
```

事先检查async-profiler的所有可用选项始终是一个好主意：

```shell
$ ./profiler.sh
```

```text
Usage: ./profiler.sh [action] [options] 
Actions:
  start             start profiling and return immediately
  resume            resume profiling without resetting collected data
  stop              stop profiling
  check             check if the specified profiling event is available
  status            print profiling status
  list              list profiling events supported by the target JVM
  collect           collect profile for the specified period of time
                    and then stop (default action)
Options:
  -e event          profiling event: cpu|alloc|lock|cache-misses etc.
  -d duration       run profiling for  seconds
  -f filename       dump output to 
  -i interval       sampling interval in nanoseconds
  -j jstackdepth    maximum Java stack depth
  -b bufsize        frame buffer size
  -t                profile different threads separately
  -s                simple class names instead of FQN
  -g                print method signatures
  -a                annotate Java method names
  -o fmt            output format: summary|traces|flat|collapsed|svg|tree|jfr
  -I include        output only stack traces containing the specified pattern
  -X exclude        exclude stack traces with the specified pattern
  -v, --version     display version string

  --title string    SVG title
  --width px        SVG width
  --height px       SVG frame height
  --minwidth px     skip frames smaller than px
  --reverse         generate stack-reversed FlameGraph / Call tree

  --all-kernel      only include kernel-mode events
  --all-user        only include user-mode events
  --cstack mode     how to traverse C stack: fp|lbr|no

 is a numeric process ID of the target JVM
      or 'jps' keyword to find running JVM automatically
```

所示的许多选项在后面的部分中将会派上用场。

### 3.2 内核配置

在Linux平台上使用async-profiler时，我们应该确保配置内核以使用所有用户的perf_events捕获调用堆栈：

首先，我们将perf_event_paranoid设置为1，这将允许分析器收集性能信息：

```shell
$ sudo sh -c 'echo 1 >/proc/sys/kernel/perf_event_paranoid'
```

然后，我们将kptr_restrict设置为0，以消除对暴露内核地址的限制：

```shell
$ sudo sh -c 'echo 0 >/proc/sys/kernel/kptr_restrict'
```

但是，async-profiler可以在macOS平台上自行运行。

现在我们的平台已经准备就绪，我们可以构建我们的分析应用程序并使用Java命令运行它：

```shell
$ java -XX:+UnlockDiagnosticVMOptions -XX:+DebugNonSafepoints -jar path-to-jar-file
```

在这里，**我们使用[-XX:+UnlockDiagnosticVMOptions](https://www.baeldung.com/jvm-tuning-flags#1-diagnostic-flags) -XX:+DebugNonSafepoints JVM标志启动了我们的分析应用程序，为了获得准确的结果，强烈建议使用这些标志**。

现在我们已经准备好分析我们的应用程序了，让我们探索一下async-profiler支持的各种类型的分析。

## 4. CPU分析

async-profiler在分析CPU时收集Java方法的示例堆栈跟踪，包括JVM代码、本机类和内核函数。

让我们使用其PID来分析我们的应用程序：

```shell
$ ./profiler.sh -e cpu -d 30 -o summary 66959
Started [cpu] profiling
--- Execution profile --- 
Total samples       : 28

Frame buffer usage  : 0.069%
```

这里，我们使用-e选项定义了CPU分析事件。然后，我们使用-d <duration\>选项收集了30秒的样本。

最后，**-o选项可用于定义输出格式，如摘要、HTML、跟踪、SVG和树**。

让我们在CPU分析应用程序时创建HTML输出：

```shell
$ ./profiler.sh -e cpu -d 30 -f cpu_profile.html 66959
```

![](/assets/images/2025/javajvm/javaasyncprofiler01.png)

在这里，我们可以看到HTML输出允许我们展开、折叠和搜索样本。

此外，**async-profiler开箱即用地支持火焰图**。

让我们使用.svg文件扩展名来生成应用程序CPU配置文件的火焰图：

```shell
$ ./profiler.sh -e cpu -d 30 -f cpu_profile.svg 66959
```

![](/assets/images/2025/javajvm/javaasyncprofiler02.png)

这里，生成的火焰图显示Java代码路径为绿色，C++为黄色，系统代码路径为红色。

## 5. 分配分析

类似地，我们可以收集内存分配的样本，而无需使用字节码检测等侵入技术。

async-profiler使用基于[TLAB](https://alidg.me/blog/2019/6/21/tlab-jvm)(线程本地分配缓冲区)的采样技术来收集高于TLAB平均大小的堆分配样本。

通过使用alloc事件，我们可以让分析器收集分析应用程序的堆分配：

```shell
$ ./profiler.sh -e alloc -d 30 -f alloc_profile.svg 66255
```

![](/assets/images/2025/javajvm/javaasyncprofiler03.png)

这里我们可以看到对象克隆分配了很大一部分内存，这在查看代码时很难察觉。

## 6. 挂钟分析

此外，async-profiler可以使用挂钟分析对所有线程进行采样，无论其状态如何-例如运行、休眠或阻塞。

这在解决应用程序启动时的问题时非常有用。

通过定义wall事件，我们可以配置分析器来收集所有线程的样本：

```shell
$ ./profiler.sh -e wall -t -d 30 -f wall_clock_profile.svg 66959
```

![](/assets/images/2025/javajvm/javaasyncprofiler04.png)

在这里，我们使用了-t选项在每个线程模式下使用挂钟分析器，在分析所有线程时强烈建议使用此选项。

此外，我们可以使用list选项检查JVM支持的所有分析事件：

```shell
$ ./profiler.sh list 66959
```

```text
Basic events:
  cpu
  alloc
  lock
  wall
  itimer
Java method calls:
  ClassName.methodName
```

## 7. IntelliJ IDEA中的async-profiler

**IntelliJ IDEA具有与async-profiler集成的功能，可作为Java的分析工具**。

### 7.1 分析器配置

我们可以通过选择“Settings/Preferences > Build, Execution, Deployment”中的“Java Profiler”菜单选项来在IntelliJ IDEA中配置async-profiler：

![](/assets/images/2025/javajvm/javaasyncprofiler05.png)

此外，为了快速使用，**我们可以选择任何预定义的配置，例如IntelliJ IDEA提供的CPU Profiler和Allocation Profiler**。

类似地，我们可以复制分析器模板并针对特定用例编辑Agent选项。

### 7.2 使用IntelliJ IDEA进行应用程序分析

有几种方法可以使用分析器来分析我们的应用程序。

例如，我们可以选择应用程序并选择使用Run <application name\> with <profiler configuration name\>选项：

![](/assets/images/2025/javajvm/javaasyncprofiler06.png)

或者，我们可以单击工具栏并选择使用Run <application name\> with <profiler configuration name\>选项：

![](/assets/images/2025/javajvm/javaasyncprofiler07.png)

或者，通过选择“Run”菜单下的“Run with Profiler”选项，然后选择<profiler configuration name\>：

![](/assets/images/2025/javajvm/javaasyncprofiler08.png)

此外，我们可以在“Run”菜单下看到“Attach Profiler to Process”选项，它会打开一个对话框，让我们选择要附加的进程：

![](/assets/images/2025/javajvm/javaasyncprofiler09.png)

一旦我们的应用程序被分析，我们就可以使用IDE底部的Profiler工具窗口栏来分析分析结果。

我们的应用程序的分析结果将如下所示：

![](/assets/images/2025/javajvm/javaasyncprofiler10.png)

它以不同的输出格式(如火焰图、调用树和方法列表)显示线程结果。

或者，我们可以选择View > Tool Windows菜单下的“Profiler”选项来查看结果：

![](/assets/images/2025/javajvm/javaasyncprofiler11.png)

## 8. 总结

在本文中，我们探讨了async-profiler以及一些分析技术。

首先，我们了解了如何在使用Linux平台时配置内核，以及一些推荐的JVM标志，以便开始分析我们的应用程序以获得准确的结果。

然后，我们研究了各种类型的分析技术，如CPU、分配和挂钟。

最后，我们使用IntelliJ IDEA通过async-profiler对应用程序进行了分析。