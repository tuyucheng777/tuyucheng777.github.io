---
layout: post
title:  使用单独的线程在Java中读取和写入文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 简介

当谈到Java中的文件处理时，管理大文件而不引起性能问题可能是一项挑战，这就是使用单独线程的概念的由来。**通过使用单独的线程，我们可以高效地读取和写入文件而不会阻塞主线程**。在本教程中，我们将探讨如何使用单独的线程读取和写入文件。

## 2. 为什么要使用单独的线程

使用单独的线程进行文件操作可以提高性能，因为允许并发执行任务。在单线程程序中，文件操作是按顺序执行的。例如，我们先读取整个文件，然后写入另一个文件。这可能很耗时，尤其是对于大文件。

**通过使用单独的线程，可以同时执行多个文件操作，利用多核处理器并将I/O操作与计算重叠，这种并发性可以更好地利用系统资源并减少总体执行时间**。但是，必须注意的是，使用单独线程的有效性取决于任务的性质和所涉及的I/O操作。

## 3. 利用线程实现文件操作

可以使用单独的线程来读取和写入文件以提高性能，在本节中，我们将讨论如何使用线程实现文件操作。

### 3.1 在单独的线程中读取文件

要在单独的线程中读取文件，我们可以创建一个新线程并传递一个读取文件的Runnable对象，[FileReader](https://www.baeldung.com/java-filereader)类用于读取文件。此外，为了增强文件读取过程，我们使用BufferedReader，它允许我们高效地逐行读取文件：

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        try (BufferedReader bufferedReader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = bufferedReader.readLine()) != null) {
                System.out.println(line);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
});

thread.start();
```

### 3.2 在单独的线程中写入文件

我们创建另一个新线程并使用[FileWriter](https://www.baeldung.com/java-filewriter)类将数据写入文件：

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        try (FileWriter fileWriter = new FileWriter(filePath)) {
            fileWriter.write("Hello, world!");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
});

thread.start();
```

这种方法允许读取和写入同时运行，这意味着它们可以在不同的线程中同时发生。**当一个操作不依赖于另一个操作的完成时，这种方法尤其有用**。

## 4. 处理并发

多个线程同时访问文件时需要格外小心，以免数据损坏和出现意外行为。在前面的代码中，两个线程是同时启动的，这意味着它们可以同时执行，并且无法保证它们的操作交错顺序。**如果读取线程在写入操作仍在进行时尝试访问文件，则最终可能会读取不完整或部分写入的数据**。这可能会导致在处理过程中出现误导性信息或错误，从而可能影响依赖准确数据的下游操作。

此外，如果两个写入线程同时尝试将数据写入文件，它们的写入可能会交错并覆盖彼此的部分数据。如果没有适当的同步处理，这可能会导致信息损坏或不一致。

为了解决这个问题，一种常见的方法是使用生产者-消费者模型。**一个或多个生产者线程读取文件并将其添加到队列中，一个或多个消费者线程处理队列中的文件**。这种方法允许我们根据需要添加更多生产者或消费者，从而轻松扩展我们的应用程序。

## 5. 使用BlockingQueue进行并发文件处理

具有队列的生产者-消费者模型协调操作，确保读写顺序一致。为了实现此模型，我们可以使用线程安全的队列数据结构，例如[BlockingQueue](https://www.baeldung.com/java-blocking-queue)。生产者可以使用offer()方法将文件添加到队列，消费者可以使用poll()方法检索文件。

**每个BlockingQueue实例都有一个内部锁，用于管理对其内部数据结构(链表、数组等)的访问**。当线程尝试执行offer()或poll()等操作时，它首先获取此锁，这可确保一次只有一个线程可以访问队列，从而防止同时修改和数据损坏。

通过使用BlockingQueue，我们可以解耦生产者和消费者，让它们按照自己的节奏工作，而不是直接等待对方，这可以提高整体性能。

### 5.1 创建FileProducer

我们首先创建FileProducer类，表示负责从输入文件中读取行并将其添加到共享队列的生产者线程。该类利用BlockingQueue来协调生产者和消费者线程，它接收BlockingQueue作为行的同步存储，确保消费者线程可以访问它们。

以下是FileProducer类的一个示例：

```java
class FileProducer implements Runnable {
    private final BlockingQueue<String> queue;
    private final String inputFileName;

    public FileProducer(BlockingQueue<String> queue, String inputFileName) {
        this.queue = queue;
        this.inputFileName = inputFileName;
    }
    // ...
}
```

接下来，在run()方法中，我们使用BufferedReader打开文件，以便高效地读取行；我们还为文件操作期间可能发生的潜在IOException提供了错误处理。

```java
@Override
public void run() {
    try (BufferedReader reader = new BufferedReader(new FileReader(inputFileName))) {
        String line;
        // ...
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

打开文件后，代码进入循环，从文件中读取行并同时使用offer()方法将它们添加到队列中：

```java
while ((line = reader.readLine()) != null) {
    queue.offer(line);
}
```

### 5.2 创建FileConsumer

接下来，我们介绍FileConsumer类，它表示负责从队列中检索行并将其写入输出文件的消费者线程。此类接收BlockingQueue作为输入，用于从生产者线程接收行：

```java
class FileConsumer implements Runnable {
    private final BlockingQueue<String> queue;
    private final String outputFileName;

    public FileConsumer(BlockingQueue queue, String outputFileName) {
        this.queue = queue;
        this.outputFileName = outputFileName;
    }

    // ...
}
```

接下来，在run()方法中我们使用BufferedWriter来方便高效地写入输出文件：

```java
@Override
public void run() {
    try (BufferedWriter writer = new BufferedWriter(new FileWriter(outputFileName))) {
        String line;
        // ...
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

打开输出文件后，代码进入连续循环，使用poll()方法从队列中检索行。如果有行可用，它会将该行写入文件。当poll()返回null时，循环终止，表示生产者已完成行的写入，并且没有其他行需要处理：

```java
while ((line = queue.poll()) != null) {
    writer.write(line);
    writer.newLine();
}
```

### 5.3 线程协调器

最后，我们将主程序中的所有内容包装在一起。首先，我们创建一个LinkedBlockingQueue实例，作为生产者和消费者线程之间的中介。此队列建立了一个同步通道，用于通信和协调。

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>();
```

接下来，我们创建两个线程：一个FileProducer线程负责从输入文件中读取行并将其添加到队列。我们还创建一个FileConsumer线程，负责从队列中检索行并熟练地处理它们并输出到指定的输出文件：

```java
String fileName = "input.txt";
String outputFileName = "output.txt";

Thread producerThread = new Thread(new FileProducer(queue, fileName));
Thread consumerThread = new Thread(new FileConsumer(queue, outputFileName);
```

随后，我们使用start()方法启动它们的执行，我们利用join()方法确保两个线程在程序退出之前正常完成其工作：

```java
producerThread.start();
consumerThread.start();

try {
    producerThread.join();
    consumerThread1.join();
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

现在，让我们创建一个输入文件，然后运行该程序：

```text
Hello,
Tuyucheng!
Nice to meet you!
```

运行程序后，我们可以检查输出文件，应该看到输出文件包含与输入文件相同的行：

```text
Hello,
Tuyucheng!
Nice to meet you!
```

在提供的示例中，生产者循环将行添加到队列中，而消费者循环从队列中检索行，这意味着多行可以同时在队列中，即使生产者仍在添加更多行，消费者也可以从队列中处理行。

## 6. 总结

在本文中，我们探讨了如何使用单独的线程在Java中高效处理文件，我们还演示了如何使用BlockingQueue实现文件的同步和高效逐行处理。