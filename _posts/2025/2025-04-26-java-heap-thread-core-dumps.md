---
layout: post
title:  堆转储、线程转储和核心转储之间的区别
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

**转储是从存储介质中查询的数据，并存储在某个地方以供进一步分析**。[Java虚拟机(JVM)](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)负责管理Java中的内存，如果出现错误，我们可以从JVM获取转储文件来诊断错误。

在本教程中，我们将探讨三种常见的Java转储文件-堆转储、线程转储和核心转储，并了解它们的用例。

## 2. 堆转储

在运行时，JVM会创建堆，其中包含正在运行的Java应用程序中正在使用的对象的引用，**堆转储包含运行时正在使用的所有对象的当前状态的[保存副本](https://www.baeldung.com/java-heap-dump-capture)**。

此外，**它还用于分析Java中的[OutOfMemoryError](https://www.baeldung.com/java-memory-leaks)错误**。

此外，**堆转储可以采用两种格式-经典格式和可移植堆格式(PHD)**。

经典格式易于理解，而PHD格式是二进制文件，需要使用工具进行进一步分析。此外，PHD格式是堆转储的默认格式。

此外，现代堆转储还包含一些线程信息。从Java 6 Update 14开始，堆转储还包含线程的堆栈跟踪，**堆转储中的堆栈跟踪将对象与使用它们的线程联系起来**。

Eclipse Memory Analyzer等分析工具支持检索此信息。

### 2.1 用例

**在分析Java应用程序中的OutOfMemoryError时，堆转储可以提供帮助**。

让我们看一些抛出OutOfMemoryError的示例代码：

```java
public class HeapDump {
    public static void main(String[] args) {
        List numbers = new ArrayList<>();
        try {
            while (true) {
                numbers.add(10);
            }
        } catch (OutOfMemoryError e) {
            System.out.println("Out of memory error occurred!");
        }
    }
}
```

在上面的示例代码中，我们创建了一个无限循环的场景，直到堆内存已满。众所周知，new关键字有助于在Java中在堆上分配内存。

为了捕获上述代码的堆转储，我们需要一个工具，**最常用的工具之一是[jmap](https://www.baeldung.com/java-heap-dump-capture#1-jmap)**。

首先，我们需要通过运行jps命令来获取机器上所有正在运行的Java进程的进程ID：

```shell
$ jps
```

上述命令将所有正在运行的Java进程输出到控制台：

```text
12789 Launcher
13302 Jps
7517 HeapDump
```

这里我们感兴趣的进程是HeapDump，因此，让我们运行jmap命令，并传入HeapDump进程ID来捕获堆转储：

```shell
 $ jmap -dump:live,file=hdump.hprof 7517
```

上述命令在项目根目录中生成hdump.hprof文件。

最后，**我们可以使用Eclipse Memory Analyzer等工具来分析转储文件**。

## 3. 线程转储

**[线程转储](https://www.baeldung.com/java-thread-dump#:~:text=A%20thread%20dump%20is%20a,it%20displays%20the%20thread's%20activity.)包含特定时刻正在运行的Java程序中所有线程的快照**。

线程是进程的最小部分，它通过同时运行多个任务来帮助程序高效运行。

此外，**线程转储可以帮助诊断Java应用程序中的效率问题**。因此，它是分析性能问题的重要工具，尤其是在应用程序运行缓慢时。

此外，**它还可以帮助检测陷入无限循环的线程，还可以帮助识别[死锁](https://www.baeldung.com/java-deadlock-livelock#deadlock)(即多个线程等待另一个线程释放资源的情况)**。

此外，它还可以识别某些线程没有获得足够CPU时间的情况，这有助于识别性能瓶颈。

### 3.1 用例

这是一个示例程序，由于长时间运行的任务，其性能可能会很慢：

```java
public class ThreadDump {
    public static void main(String[] args) {
        longRunningTask();
    }
    
    private static void longRunningTask() {
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            if (Thread.currentThread().isInterrupted()) {
                System.out.println("Interrupted!");
                break;
            }
            System.out.println(i);
        }
    }
}
```

在上面的示例代码中，我们创建了一个循环到Integer.MAX_VALUE的方法，并将该值输出到控制台。**这是一个长时间运行的操作，可能会造成性能问题**。

**为了分析性能，我们可以捕获线程转储**。首先，让我们找到所有正在运行的Java程序的进程ID：

```shell
$ jps
```

jps命令将所有Java进程输出到控制台：

```text
3042 ThreadDump
964 Main
3032 Launcher
3119 Jps
```

我们对ThreadDump进程ID很感兴趣，接下来，**让我们使用[jstack](https://www.baeldung.com/java-thread-dump#1-jstack)命令并结合进程ID来获取线程转储**：

```shell
$ jstack -l 3042 > slow-running-task-thread-dump.txt
```

上述命令捕获线程转储并将其保存在txt文件中以供进一步[分析](https://www.baeldung.com/java-analyze-thread-dumps)。

## 4. 核心转储

**核心转储(也称为崩溃转储)包含程序崩溃或突然终止时的快照**。

JVM运行的是字节码而不是本机代码，因此，Java代码不会引起核心转储。

然而，有些Java程序使用[Java原生接口(JNI)](https://www.baeldung.com/jni)直接运行原生代码。由于外部库可能会崩溃，JNI可能会导致JVM崩溃，我们可以在崩溃时获取核心转储并进行分析。

此外，**核心转储是操作系统级别的转储，可用于在JVM崩溃时查找本机调用的详细信息**。

### 4.1 用例

让我们看一个使用JNI生成核心转储的示例。

首先，让我们创建一个名为CoreDump的类来加载本机库：

```java
public class CoreDump {
    private native void core();
    public static void main(String[] args) {
        new CoreDump().core();
    }
    static {
        System.loadLibrary("nativelib");
    }
}
```

接下来，让我们使用javac命令编译Java代码：

```shell
$ CoreDump.java
```

然后，让我们通过运行javac-h命令来生成本机方法实现的标头：

```shell
$ javac -h . CoreDump.java

```

最后，让我们用C实现一个会导致JVM崩溃的本机方法：

```c
#include <jni.h>
#include "CoreDump.h"
    
void core() {
    int *p = NULL;
    *p = 0;
}
JNIEXPORT void JNICALL Java_CoreDump_core (JNIEnv *env, jobject obj) {
    core();
};
void main() {
}
```

让我们通过运行gcc命令来编译本机代码：

```shell
$ gcc -fPIC -I"/usr/lib/jvm/java-17-graalvm/include" -I"/usr/lib/jvm/java-17-graalvm/include/linux" -shared -o libnativelib.so CoreDump.c
```

这将生成名为libnativelib.so的共享库，接下来，让我们使用共享库编译Java代码：

```shell
$ java -Djava.library.path=. CoreDump
```

**本机方法导致JVM崩溃并在项目目录中生成核心转储**：

```text
// ...
# A fatal error has been detected by the Java Runtime Environment:
# SIGSEGV (0xb) at pc=0x00007f9c48878119, pid=65743, tid=65744
# C  [libnativelib.so+0x1119]  core+0x10
# Core dump will be written. Default location: Core dumps may be processed with 
# "/usr/lib/systemd/systemd-coredump %P %u %g %s %t %c %h" (or dumping to /core-java-perf/core.65743)
# An error report file with more information is saved as:
# ~/core-java-perf/hs_err_pid65743.log
// ...
```

上述输出显示了崩溃信息和转储文件的位置。

## 5. 主要区别

下面是一个汇总表，显示了三种类型的Java转储文件之间的主要区别：

| 转储类型|                 用例|      内容       |
| :------: | :----------------------------------: |:-------------:|
|  堆转储| 诊断内存问题，例如OutOfMemoryError|  Java堆中的对象快照  |
| 线程转储|   解决性能问题、线程死锁和无限循环| JVM中所有线程状态的快照 |
| 核心转储|         本机库导致的调试崩溃|  JVM崩溃时的进程状态  |

## 6. 总结

在本文中，我们通过了解堆转储、线程转储和核心转储的用途，了解了它们之间的区别。此外，我们还查看了包含不同问题的示例代码，并生成了一个转储文件以供进一步分析，每个转储文件在Java应用程序故障排除中都有不同的用途。
