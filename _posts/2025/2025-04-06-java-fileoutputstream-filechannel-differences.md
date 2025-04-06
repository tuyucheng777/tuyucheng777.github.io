---
layout: post
title:  FileOutputStream与FileChannel指南
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

在Java中进行文件I/O操作时，[FileOutputStream](https://www.baeldung.com/java-outputstream#3-filteroutputstream)和[FileChannel](https://www.baeldung.com/java-filechannel)是将数据写入文件的两种常用方法。在本教程中，我们将探索它们的功能并了解它们之间的区别。

## 2. FileOutputStream

FileOutputStream是java.io包的一部分，是将二进制数据写入文件的最简单方法之一。**对于简单的写入操作，尤其是对于较小的文件，它是一个很好的选择**。它的简单性使其易于用于基本的文件写入任务。

以下代码片段演示了如何使用FileOutputStream将字节数组写入文件：

```java
byte[] data = "This is some data to write".getBytes();

try (FileOutputStream outputStream = new FileOutputStream("output.txt")) {
    outputStream.write(data);
} catch (IOException e) {
    // ...
}
```

在此示例中，我们首先创建一个包含要写入的数据的字节数组。接下来，我们初始化一个FileOutputStream对象，并指定文件名“output.txt”，try-with-resources语句确保自动关闭资源，FileOutputStream的write()方法将整个字节数组“data”写入文件。

## 3. FileChannel

FileChannel是java.nio.channels包的一部分，与FileOutputStream相比，它提供了更高级、更灵活的文件[I/O](https://www.baeldung.com/java-io)操作。**它特别适合处理较大的文件、随机访问和性能关键型应用程序，它使用缓冲区可以实现更高效的数据传输和操作**。

以下代码片段演示了如何使用FileChannel将字节数组写入文件：

```java
byte[] data = "This is some data to write".getBytes();
ByteBuffer buffer = ByteBuffer.wrap(data);

try (FileChannel fileChannel = FileChannel.open(Path.of("output.txt"), StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
    fileChannel.write(buffer);
} catch (IOException e) {
    // ...
}
```

在此示例中，我们创建一个ByteBuffer并将字节数组数据包装到其中，然后我们使用FileChannel.open()方法初始化FileChannel对象。**接下来，我们还指定文件名“output.txt”和必要的打开选项(StandardOpenOption.WRITE和StandardOpenOption.CREATE)**。

然后，FileChannel的write()方法将ByteBuffer的内容写入指定的文件。

## 4. 数据访问

在本节中，让我们深入探讨数据访问方面FileOutputStream和FileChannel之间的区别。

### 4.1 FileOutputStream

FileOutputStream按顺序写入数据，这意味着它按照给定的顺序从头到尾将字节写入文件，**它不支持跳转到文件内的特定位置来读取或写入数据**。

以下是使用FileOutputStream按顺序写入数据的示例：

```java
byte[] data1 = "This is the first line.\n".getBytes();
byte[] data2 = "This is the second line.\n".getBytes();

try (FileOutputStream outputStream = new FileOutputStream("output.txt")) {
    outputStream.write(data1);
    outputStream.write(data2);
} catch (IOException e) {
    // ...
}
```

在此代码中，将首先写入“This is the first line.”，然后在“output.txt”文件的新行中写入“This is the second line.”。**如果不从头开始重写所有内容，我们就无法在文件中间写入数据**。

### 4.2 FileChannel

另一方面，FileChannel允许我们在文件中的任何位置读取或写入数据，**这是因为FileChannel使用可以移动到文件中的任何位置的文件指针**，这是使用position()方法实现的，该方法设置文件中下一次读取或写入发生的位置。

下面的代码片段演示了FileChannel如何将数据写入文件内的特定位置：

```java
ByteBuffer buffer1 = ByteBuffer.wrap(data1);
ByteBuffer buffer2 = ByteBuffer.wrap(data2);

try (FileChannel fileChannel = FileChannel.open(Path.of("output.txt"), StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
    fileChannel.write(buffer1);

    fileChannel.position(10);
    fileChannel.write(buffer2);
} catch (IOException e) {
    // ...
}
```

在这个例子中，data1写入文件的开头。现在，我们想从位置10开始将data2插入到文件中。因此，我们使用fileChannel.position(10)将位置设置为10，然后从第10个字节开始写入data2。

## 5. 并发和线程安全

在本节中，我们将探讨FileOutputStream和FileChannel如何处理并发和线程安全。

### 5.1 FileOutputStream

FileOutputStream内部不处理同步，**如果两个[线程](https://www.baeldung.com/cs/multiprocessing-multithreading#multiprocessing-and-multithreading-basics)试图同时写入同一个FileOutputStream，则结果可能是输出文件中的数据交错不可预测**，因此我们需要同步来确保线程安全。

下面是使用带有外部同步的FileOutputStream的示例：

```java
final Object lock = new Object();

void writeToFile(String fileName, byte[] data) {
    synchronized (lock) {
        try (FileOutputStream outputStream = new FileOutputStream(fileName, true)) {
            outputStream.write(data);
            log.info("Data written by " + Thread.currentThread().getName());
        } catch (IOException e) {
            // ...
        }
    }
}
```

在这个例子中，我们使用一个通用的锁对象来同步对文件的访问，当多个线程顺序地向文件写入数据时，保证了线程安全：

```java
Thread thread1 = new Thread(() -> writeToFile("output.txt", data1));
Thread thread2 = new Thread(() -> writeToFile("output.txt", data2));

thread1.start();
thread2.start();
```

### 5.2 FileChannel

相比之下，FileChannel支持[文件锁定](https://www.baeldung.com/java-lock-files#file-locks-in-java)，允许我们锁定特定的文件部分，以防止其他线程或进程同时访问该数据。

以下是使用FileChannel和FileLock处理并发访问的示例：

```java
void writeToFileWithLock(String fileName, ByteBuffer buffer, int position) {
    try (FileChannel fileChannel = FileChannel.open(Path.of(fileName), StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
        // Acquire an exclusive lock on the file
        try (FileLock lock = fileChannel.lock(position, buffer.remaining(), false)) {
            fileChannel.position(position);
            fileChannel.write(buffer);
            log.info("Data written by " + Thread.currentThread().getName() + " at position " + position);
        } catch (IOException e) {
            // ...
        }
    } catch (IOException e) {
        // ...
    }
}
```

在此示例中，FileLock对象用于确保锁定要写入的文件部分，以防止其他线程同时访问它。当线程调用writeToFileWithLock()时，它首先获取文件特定部分的锁定：

```java
Thread thread1 = new Thread(() -> writeToFileWithLock("output.txt", buffer1, 0));
Thread thread2 = new Thread(() -> writeToFileWithLock("output.txt", buffer2, 20));

thread1.start();
thread2.start();
```

## 6. 性能

在本节中，我们将使用[JMH](https://www.baeldung.com/java-microbenchmark-harness)比较FileOutputStream和FileChannel的性能，我们将创建一个包含FileOutputStream和FileChannel基准测试的基准类，以评估它们处理大文件的性能：

```java
@Setup
public void setup() {
    largeData = new byte[1000 * 1024 * 1024]; // 1 GB of data
    Arrays.fill(largeData, (byte) 1);
}

@Benchmark
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public void testFileOutputStream() {
    try (FileOutputStream outputStream = new FileOutputStream("largeOutputStream.txt")) {
        outputStream.write(largeData);
    } catch (IOException e) {
        // ...
    }
}

@Benchmark
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
public void testFileChannel() {
    ByteBuffer buffer = ByteBuffer.wrap(largeData);
    try (FileChannel fileChannel = FileChannel.open(Path.of("largeFileChannel.txt"), StandardOpenOption.WRITE, StandardOpenOption.CREATE)) {
        fileChannel.write(buffer);
    } catch (IOException e) {
        // ...
    }
}
```

让我们执行基准测试并比较FileOutputStream和FileChannel的性能，结果显示每个操作所需的平均时间(以毫秒为单位)：

```java
Options opt = new OptionsBuilder()
    .include(FileIOBenchmark.class.getSimpleName())
    .forks(1)
    .build();

new Runner(opt).run();
```

运行基准测试后，我们获得了以下结果：

```text
Benchmark                             Mode  Cnt    Score    Error  Units
FileIOBenchmark.testFileChannel       avgt    5  431.414 ± 52.229  ms/op
FileIOBenchmark.testFileOutputStream  avgt    5  556.102 ± 91.512  ms/op
```

FileOutputStream的设计理念是简单易用，然而，在处理具有高频率I/O操作的大型文件时，它可能会带来一些开销。**这是因为FileOutputStream操作是阻塞的，这意味着每个写入操作都必须在下一个操作开始之前完成**。

另一方面，FileChannel支持内存映射I/O，可以将文件的一部分映射到内存中，**这使得数据操作可以直接在内存空间中进行，从而实现更快的传输**。

## 7. 总结

在本文中，我们探讨了两种文件I/O方法：FileOutputStream和FileChannel。FileOutputStream为基本文件写入任务提供了简单性和易用性，非常适合较小的文件和顺序数据写入。

另一方面，FileChannel提供了直接缓冲区访问等高级功能，以便获得大文件的更好性能。