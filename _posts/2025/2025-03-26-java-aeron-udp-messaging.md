---
layout: post
title:  使用Aeron进行UDP消息传递
category: libraries
copyright: libraries
excerpt: Aeron
---

## 1. 简介

在本文中，我们将介绍[Aeron](https://github.com/real-logic/aeron?tab=readme-ov-file)，这是一个由Adaptive Financial Consulting维护的多语言库，旨在实现应用程序之间的高效UDP消息传递。它专为提高性能而设计，旨在实现高吞吐量、低延迟和容错。

## 2. 依赖

**在使用Aeron之前，我们需要在我们的构建中包含[最新版本](https://mvnrepository.com/artifact/io.aeron/aeron-all)，在撰写本文时版本为[1.44.1](https://mvnrepository.com/artifact/io.aeron/aeron-all/1.44.1)**。

如果我们使用Maven，我们可以在pom.xml中包含它的依赖：

```xml
<dependency>
    <groupId>io.aeron</groupId>
    <artifactId>aeron-all</artifactId>
    <version>1.44.1</version>
</dependency>
```

或者如果使用Gradle，我们可以将其包含在build.gradle中：

```groovy
implementation("io.aeron:aeron-all:1.44.1")
```

此时，我们已准备好开始在我们的应用程序中使用它。

**请注意，目前Aeron的某些部分无法与Java 16或更新版本兼容**，这是由于[JPMS](https://www.baeldung.com/java-illegal-reflective-access)阻止了某些交互。

## 3. 媒体驱动程序

**Aeron在应用程序和传输之间采用间接方式工作，这被称为媒体驱动程序，因为它是我们的应用程序和传输媒体之间的交互**。

每个Aeron进程都与一个媒体驱动程序交互，并通过该驱动程序与其他进程交互-无论是在同一台机器上还是远程。它通过文件系统执行此交互，我们需要将媒体驱动程序和所有应用程序指向磁盘上的同一目录，该目录存储了各个方面。请注意，我们只能同时为任何给定目录运行一个媒体驱动程序。尝试运行多个媒体驱动程序将失败。

当我们想要简单的时候，我们可以运行应用程序中嵌入的媒体驱动程序：

```java
MediaDriver mediaDriver = MediaDriver.launch();
```

这将启动具有所有默认设置的媒体驱动程序。具体来说，这将使用默认媒体驱动程序目录运行。

我们还有另一种专为嵌入式使用而设计的启动方法，它的作用与以前完全相同，只是它会生成一个随机目录，以确保同一台机器上的多个实例不会发生冲突：

```java
MediaDriver mediaDriver = MediaDriver.launchEmbedded();
```

在这两种情况下，我们还可以提供MediaDriver.Context对象来进一步配置媒体驱动程序：

```java
MediaDriver.Context context = new MediaDriver.Context();
context.threadingMode(ThreadingMode.SHARED);

MediaDriver mediaDriver = MediaDriver.launch(context);
```

执行此操作时，我们需要在使用完媒体驱动程序后将其关闭。该接口实现了AutoCloseable，因此我们可以使用[try-with-resources](https://www.baeldung.com/java-try-with-resources)模式来管理此操作。

**或者，我们可以将媒体驱动程序作为外部应用程序运行，我们可以使用作为依赖包含的aeron-all.jar JAR文件来执行此操作**：

```shell
$ java -cp aeron-all-1.44.1.jar io.aeron.driver.MediaDriver
```

其功能与上面的MediaDriver.launch()完全相同。

## 4. Aeron API客户端

**我们通过Aeron类使用Aeron执行所有API交互，我们需要创建一个新的实例并将其指向我们的媒体驱动程序**。只需创建一个新实例即可指向默认位置的媒体驱动程序-就像我们使用MediaDriver.launch()启动它一样：

```java
Aeron aeron = Aeron.connect();
```

或者，我们可以提供一个Aeron.Context对象来配置连接，包括指定媒体驱动程序正在运行的目录：

```java
Aeron.Context ctx = new Aeron.Context();
ctx.aeronDirectoryName(mediaDriver.aeronDirectoryName());

Aeron aeron = Aeron.connect(ctx);
```

如果我们的媒体驱动程序位于非标准目录中，包括我们使用MediaDriver.launchEmbedded()启动它，那么我们必须这样做。如果我们指向的目录没有正在运行的媒体驱动程序，则Aeron.connect()调用将阻塞，直到有正在运行的媒体驱动程序为止。

**我们可以将任意数量的Aeron客户端连接到同一个媒体驱动程序**。通常，这些客户端来自不同的应用程序，但如果需要，它们也可以来自同一个应用程序。但是，如果我们这样做，那么我们还需要使用Aeron.Context的新实例：

```java
Aeron.Context ctx1 = new Aeron.Context();
ctx1.aeronDirectoryName(mediaDriver.aeronDirectoryName());
aeron1 = Aeron.connect(ctx1);
System.out.println("Aeron 1 connected: " + aeron1);

Aeron.Context ctx2 = new Aeron.Context();
ctx2.aeronDirectoryName(mediaDriver.aeronDirectoryName());
aeron2 = Aeron.connect(ctx2);
System.out.println("Aeron 2 connected: " + aeron2);
```

与MediaDriver一样，Aeron实例也是AutoCloseable的，这意味着我们可以用try-with-resources模式包装它，以确保我们正确关闭它。

## 5. 发送和接收消息

现在我们已经有了Aeron API客户端，我们可以使用它来发送和接收消息。

### 5.1 缓冲区

**Aeron将所有消息(包括发送和接收)表示为DirectBuffer实例，归根结底，这些不过是一组字节，但它们为我们提供了一组方法来处理一组标准类型**。

当我们发送消息时，我们需要根据自己的数据自行构建缓冲区。为此，我们最好使用UnsafeBuffer实例-之所以这样命名是因为它使用[sun.misc.Unsafe](https://www.baeldung.com/java-unsafe)来读取和写入底层缓冲区的值。创建它需要一个字节数组或ByteBuffer实例，然后我们可以使用BufferUtil.allocateDirectAligned()来帮助最有效地完成此操作：

```java
UnsafeBuffer buffer = new UnsafeBuffer(BufferUtil.allocateDirectAligned(256, 64));
```

一旦我们获得了缓冲区，我们就可以使用一系列getXyz()和putXyz()方法来操作缓冲区中的数据：

```java
// Put a string into the buffer starting at index 0.
int length = buffer.putStringWithoutLengthUtf8(0, message); 

// Read a string of the given length from the buffer starting from the given offset.
String message = buffer.getStringWithoutLengthUtf8(offset, length); 
```

请注意，我们需要自己管理缓冲区中的偏移量。每当我们将数据放入缓冲区时，它都会返回写入数据的长度，以便我们计算下一个偏移量。当我们从缓冲区读取时，我们需要知道长度是多少。

### 5.2 通道和流

**使用通过特定通道传输的已识别流，可以使用Aeron发送和接收数据**。

我们将通道指定为特定格式的URI，告诉Aeron如何传输消息。然后，我们的媒体驱动程序使用它与传输媒体进行交互，确保它正确发送和接收消息。流仅以数字标识，唯一的要求是同一通道的两端使用相同的流ID。

最简单的此类通道是aeron:ipc，它使用媒体驱动程序中的共享内存进行传输和接收。请注意，这仅在双方使用相同的媒体驱动程序且不允许联网时才有效。

**更有用的是，我们可以使用aeron:udp使用[UDP](https://www.baeldung.com/cs/udp-vs-tcp#udp)发送和接收，这使我们能够与我们可以连接的任何地方的任何其他应用程序进行通信**。特别是，我们的应用程序将与媒体驱动程序通信，然后媒体驱动程序将相互通信：

![](/assets/images/2025/libraries/javaaeronudpmessaging01.png)

指定UDP通道时，我们至少需要包含主机和端口。在接收端，我们将在此监听；在发送端，我们将在此发送消息。例如，aeron:udp?endpoint=localhost:20121将通过localhost:20121上的UDP发送和接收消息。

### 5.3 订阅

一旦设置好媒体驱动程序和Aeron客户端，我们就可以接收消息了。我们通过创建对特定通道上特定流的订阅，然后轮询该流以获取消息来实现这一点。

**添加订阅足以让媒体驱动程序设置一切以便能够接收我们的消息，我们使用Aeron实例上的addSubscription()方法执行此操作**：

```java
Subscription subscription = aeron.addSubscription("aeron:udp?endpoint=localhost:20121", 1001);
```

与之前一样，当不再使用它时，我们需要关闭它，以便媒体驱动程序知道停止监听消息。与往常一样，它是AutoCloseable，因此我们可以使用try-with-resources来管理它。

**当我们订阅时，我们需要接收消息。Aeron使用轮询机制执行此操作，允许我们完全控制它何时处理消息**。要轮询消息，我们需要提供一个FragmentHandler来处理收到的消息。如果我们想将所有代码内联，我们可以使用Lambda来实现它；如果我们想重用它，我们可以使用单独的类来实现接口：

```java
FragmentHandler fragmentHandler = (buffer, offset, length, header) -> {
    String data = buffer.getStringWithoutLengthUtf8(offset, length);

    System.out.printf("Message from session %d (%d@%d) <<%s>>%n", header.sessionId(), length, offset, data);
};
```

Aeron使用缓冲区、数据起始偏移量以及接收数据的长度来调用此方法。然后，我们可以根据需要根据应用程序需要处理此缓冲区。

当我们准备轮询新消息时，我们使用Subscription.poll()方法：

```java
int fragmentsRead = subscription.poll(fragmentHandler, 10);
```

这里，我们提供了FragmentHandler实例以及尝试接收单个消息时要考虑的消息片段数量。请注意，即使媒体驱动程序中有许多消息可用，我们一次也最多只能接收一条消息。但是，如果没有消息可用，这将立即返回，如果收到的消息太大，我们可能只会收到其中的一部分。

### 5.4 发布

**消息传递的另一面是发送消息，我们使用Publication来实现这一点，它可以将消息发送到特定通道上的特定流**。

我们可以使用Aeron.addPublication()方法添加新发布，然后我们需要等待它连接，这要求接收端有一个订阅准备好接收消息：

```java
ConcurrentPublication publication = aeron.addPublication("aeron:udp?endpoint=localhost:20121", 1001);
while (!publication.isConnected()) {
    TimeUnit.MILLISECONDS.sleep(100);
}
```

如果没有连接，它将立即无法发送消息，而不是等待有人添加订阅。

与之前一样，当不再使用它时，我们需要将其关闭，以便媒体驱动程序可以释放任何分配的资源。与往常一样，这是AutoCloseable，因此我们可以使用try-with-resources来管理它。

**一旦我们有了连接的发布，我们就可以向其提供消息**。这些消息始终以填充缓冲区的形式提供，然后发送给连接的订阅者：

```java
UnsafeBuffer buffer = new UnsafeBuffer(BufferUtil.allocateDirectAligned(256, 64));
buffer.putStringWithoutLengthUtf8(0, message);

long result = publication.offer(buffer, 0, message.length());
```

如果消息已发送，我们将返回一个表示已传输字节数的值，如果缓冲区太大，该值可能小于我们预期发送的字节数。或者，它可能会向我们返回一组错误代码之一，所有错误代码都是负数，因此很容易与成功情况区分开来：

- Publication.NOT_CONNECTED：发布未连接到订阅者
- Publication.BACK_PRESSURED：来自订阅者的背压意味着我们现在无法再发送任何消息
- Publication.ADMIN_ACTION：某些管理操作(例如日志轮换)导致发送失败，在这种情况下，立即重试通常是安全的
- Publication.CLOSED：Publication实例已关闭
- Publication.MAX_POSITION_EXCEEDED：媒体驱动程序中的缓冲区已满，通常，我们可以通过关闭发布并创建新发布来解决这个问题

## 6. 总结

我们已经快速了解了Aeron、如何设置它以及如何使用它在应用程序之间进行消息传递。