---
layout: post
title:  FileWriter与BufferedWriter指南
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

**在本教程中，我们将研究用于写入文件的两个基本Java类：[FileWriter](https://www.baeldung.com/java-filewriter)和[BufferedWriter](https://www.baeldung.com/java-write-to-file#write-with-bufferedwriter)之间的性能差异**。虽然网上的传统观点通常认为BufferedWriter的性能通常优于FileWriter，但我们的目标是对这一假设进行测试。

在了解了使用类、其继承及其内部实现的基本信息之后，我们将使用[Java Microbenchmark Harness(JMH)](https://www.baeldung.com/java-microbenchmark-harness)来测试BufferedWriter是否真的具有优势。

我们将在Linux上使用JDK 17运行测试，但我们可以预期在任何操作系统上使用任何最新版本的JDK都会获得类似的结果。

## 2. 基本用法

FileWriter使用默认缓冲区将文本写入字符文件，该缓冲区的大小未在[Javadoc](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/io/FileWriter.html)中指定：

```java
FileWriter writer = new FileWriter("testFile.txt");
writer.write("Hello, Tuyucheng!");
writer.close();
```

BufferedWriter是另一种选择，它旨在包装其他[Writer](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/io/Writer.html)类，包括FileWriter：

```java
int BUFSIZE = 4194304; // 4MiB
BufferedWriter writer = new BufferedWriter(new FileWriter("testBufferedFile.txt"), BUFSIZE);
writer.write("Hello, Buffered Tuyucheng!");
writer.close();
```

在本例中，我们指定了一个4MiB的缓冲区。但是，如果我们不设置缓冲区的大小，则[Javadoc](https://docs.oracle.com/en/java/javase/22/docs/api/java.base/java/io/BufferedWriter.html)中不会指定其默认大小。

## 3. 继承

下面是一个UML图，说明了FileWriter和BufferedWriter的继承结构：

![](/assets/images/2025/javaio/javafilewritervsbufferedwriter01.png)

**了解FileWriter和BufferedWriter都扩展了Writer，并且FileWriter的操作基于OutputStreamWriter，这一点很有帮助**。不幸的是，继承层次结构的分析和Javadocs都没有告诉我们有关FileWriter和BufferedWriter的默认缓冲区大小的足够信息，因此我们将检查JDK源代码以了解更多信息。

## 4. 底层实现

**查看FileWriter的底层实现，我们发现从JDK 10到JDK 18，其默认缓冲区大小为8192字节，在更高版本中从512变为8192**。具体来说，FileWriter扩展了OutputStreamWriter，正如我们在UML图中看到的那样，OutputStreamWriter使用[StreamEncoder](https://github.com/openjdk/jdk22/blob/master/src/java.base/share/classes/sun/nio/cs/StreamEncoder.java)，其代码在JD K18之前包含DEFAULT_BYTE_BUFFER_SIZE = 8192，在更高版本中包含MAX_BYTE_BUFFER_CAPACITY = 8192。

StreamEncoder不是JDK API中的公共类，它是sun.nio.cs包中的一个内部类，在Java框架中用于处理字符流的编码。

缓冲区大小允许FileWriter通过最小化I/O操作次数来高效处理数据，由于Java中的默认字符编码通常是UTF-8，因此在大多数情况下，8192字节大约对应8192个字符。尽管缓冲效率很高，但由于文档过时，FileWriter仍被认为没有缓冲能力。

**BufferedWriter的默认缓冲区大小与FileWriter相同**，我们可以通过检查其[源代码](https://github.com/openjdk/jdk22/blob/master/src/java.base/share/classes/java/io/BufferedWriter.java)来验证这一点，从JDK 10到JDK 18，其中包含defaultCharBufferSize = 8192，在更高版本中包含DEFAULT_MAX_BUFFER_SIZE = 8192。但是，BufferedWriter允许我们指定不同的缓冲区大小，正如我们在前面的示例中看到的那样。

## 5. 性能比较

在这里，我们将FileWriter和BufferedWriter使用JMH进行比较，如果我们想在我们的机器上测试，并且如果我们使用Maven，我们需要在pom.xml上设置JMH依赖，将JMH注解处理器添加到Maven编译器插件配置，并确保所有必需的类和资源在执行期间都可用，我们的JMH教程的[入门](https://www.baeldung.com/java-microbenchmark-harness#start)部分涵盖了这些要点。

### 5.1 磁盘写同步

**要使用JHM执行磁盘写入基准测试，必须通过禁用操作系统缓存来实现磁盘操作的完全同步**。此步骤至关重要，因为异步磁盘写入会严重影响I/O操作测量的准确性。默认情况下，操作系统会将经常访问的数据存储在内存中，从而减少实际磁盘写入的次数，这会使基准测试结果无效。

在Linux系统上，我们可以使用[mount](https://www.baeldung.com/linux/mount-unmount-filesystems)的sync选项重新挂载文件系统以禁用缓存并确保所有写入操作立即同步到磁盘：

```shell
$ sudo mount -o remount,sync /path/to/mount
```

类似地，[macOS mount](https://ss64.com/mac/mount.html)有一个sync选项，可确保文件系统的所有I/O都是同步的。

在Windows上，我们打开设备管理器并展开驱动器部分，然后右键单击要配置的驱动器，选择属性，然后导航到策略选项卡。最后，我们禁用在设备上启用写入缓存选项。

### 5.2 我们的测试

我们的代码测量了FileWriter和BufferedWriter在各种写入条件下的性能，我们运行了几个基准测试，以测试对benchmark.txt文件的单次写入和重复写入(10、1000、10000和100000次)。

**我们使用[特定于JMH的注解](https://javadoc.io/doc/org.openjdk.jmh/jmh-core/latest/org/openjdk/jmh/annotations/package-summary.html)来配置基准测试参数**，例如@Benchmark、@State、@BenchmarkMode等，以设置范围、模式、预热迭代、测量迭代和分叉设置。

main方法在运行JMH基准测试套件之前，通过删除任何现有的benchmark.txt文件并调整类路径来设置环境：

```java
@State(Scope.Benchmark)
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@Warmup(iterations = 5, time = 1, timeUnit = TimeUnit.SECONDS)
@Measurement(iterations = 10, time = 10, timeUnit = TimeUnit.SECONDS)
@Fork(1)
public class BenchmarkWriters {
    private static final Logger log = LoggerFactory.getLogger(BenchmarkWriters.class);
    private static final String FILE_PATH = "benchmark.txt";
    private static final String CONTENT = "This is a test line.";
    private static final int BUFSIZE = 4194304; // 4MiB

    @Benchmark
    public void fileWriter1Write() {
        try (FileWriter writer = new FileWriter(FILE_PATH, true)) {
            writer.write(CONTENT);
            writer.close();
        } catch (IOException e) {
            log.error("Error in FileWriter 1 write", e);
        }
    }

    @Benchmark
    public void bufferedWriter1Write() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(FILE_PATH, true), BUFSIZE)) {
            writer.write(CONTENT);
            writer.close();
        } catch (IOException e) {
            log.error("Error in BufferedWriter 1 write", e);
        }
    }

    @Benchmark
    public void fileWriter10Writes() {
        try (FileWriter writer = new FileWriter(FILE_PATH, true)) {
            for (int i = 0; i < 10; i++) {
                writer.write(CONTENT);
            }
            writer.close();
        } catch (IOException e) {
            log.error("Error in FileWriter 10 writes", e);
        }
    }

    @Benchmark
    public void bufferedWriter10Writes() {
        try (BufferedWriter writer = new BufferedWriter(new FileWriter(FILE_PATH, true), BUFSIZE)) {
            for (int i = 0; i < 10; i++) {
                writer.write(CONTENT);
            }
            writer.close();
        } catch (IOException e) {
            log.error("Error in BufferedWriter 10 writes", e);
        }
    }

    @Benchmark
    public void fileWriter1000Writes() {
        [...]
    }

    @Benchmark
    public void bufferedWriter1000Writes() {
        [...]
    }

    @Benchmark
    public void fileWriter10000Writes() {
        [...]
    }

    @Benchmark
    public void bufferedWriter10000Writes() {
        [...]
    }

    @Benchmark
    public void fileWriter100000Writes() {
        [...]
    }

    @Benchmark
    public void bufferedWriter100000Writes() {
        [...]
    }

    [...]
}
```

在这些测试中，每个基准测试方法都会独立打开和关闭文件写入器。@Fork(1)注解表示只使用了一个fork，因此不会多次并行执行相同的基准测试方法。代码不会明确创建或管理线程，因此所有写入操作都在基准测试的主线程中完成。

**所有这些意味着写入确实是串行的而不是并发的**，这对于获得有效的测量是必要的。

### 5.3 结果

这些是代码中指定BufferedWriter缓冲区大小为4MiB的结果：

```text
Benchmark                                    Mode  Cnt     Score     Error  Units
BenchmarkWriters.bufferedWriter100000Writes  avgt   10  9170.583 ± 245.916  ms/op
BenchmarkWriters.bufferedWriter10000Writes   avgt   10   918.662 ±  15.105  ms/op
BenchmarkWriters.bufferedWriter1000Writes    avgt   10   114.261 ±   2.966  ms/op
BenchmarkWriters.bufferedWriter10Writes      avgt   10    37.999 ±   1.571  ms/op
BenchmarkWriters.bufferedWriter1Write        avgt   10    37.968 ±   2.219  ms/op
BenchmarkWriters.fileWriter100000Writes      avgt   10  9253.935 ± 261.032  ms/op
BenchmarkWriters.fileWriter10000Writes       avgt   10   951.684 ±  41.391  ms/op
BenchmarkWriters.fileWriter1000Writes        avgt   10   114.610 ±   4.366  ms/op
BenchmarkWriters.fileWriter10Writes          avgt   10    37.761 ±   1.836  ms/op
BenchmarkWriters.fileWriter1Write            avgt   10    37.912 ±   2.080  ms/op
```

相反，这些是没有为BufferedWriter指定缓冲区值的结果，即使用其默认缓冲区：

```text
Benchmark                                    Mode  Cnt     Score     Error  Units
BenchmarkWriters.bufferedWriter100000Writes  avgt   10  9117.021 ± 143.096  ms/op
BenchmarkWriters.bufferedWriter10000Writes   avgt   10   931.994 ±  34.986  ms/op
BenchmarkWriters.bufferedWriter1000Writes    avgt   10   113.186 ±   2.076  ms/op
BenchmarkWriters.bufferedWriter10Writes      avgt   10    40.038 ±   2.042  ms/op
BenchmarkWriters.bufferedWriter1Write        avgt   10    38.891 ±   0.684  ms/op
BenchmarkWriters.fileWriter100000Writes      avgt   10  9261.613 ± 305.692  ms/op
BenchmarkWriters.fileWriter10000Writes       avgt   10   932.001 ±  26.676  ms/op
BenchmarkWriters.fileWriter1000Writes        avgt   10   114.209 ±   5.988  ms/op
BenchmarkWriters.fileWriter10Writes          avgt   10    38.205 ±   1.361  ms/op
BenchmarkWriters.fileWriter1Write            avgt   10    37.490 ±   2.137  ms/op
```

本质上，**这些结果表明FileWriter和BufferedWriter的性能在所有测试条件下几乎相同**。此外，为BufferedWriter指定比默认缓冲区更大的缓冲区不会带来任何好处。

## 6. 总结

在本文中，我们探讨了使用JHM测试FileWriter和BufferedWriter之间的性能差异。我们首先研究了它们的基本用法和继承结构，从JDK 10到JDK 18，这两个类的默认缓冲区大小均为8192字节，在更高版本中则从512字节变为8192字节。

我们运行基准测试来比较它们在不同条件下的性能，并通过禁用操作系统缓存来确保测量准确，测试包括使用BufferedWriter的默认和指定的4MiB缓冲区进行单次和重复写入。

我们的结果表明，FileWriter和BufferedWriter在所有场景下的性能几乎相同。此外，增加BufferedWriter的缓冲区大小并不能显著提高性能。