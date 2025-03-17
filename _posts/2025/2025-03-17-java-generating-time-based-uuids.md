---
layout: post
title:  生成基于时间的UUID
category: java
copyright: java
excerpt: Java UUID
---

## 1. 概述

在本文中，我们将了解[UUID](https://www.baeldung.com/java-uuid)和基于时间的UUID。

**我们将了解基于时间的UUID的优点和缺点以及何时选择它们**。

我们还将探索和比较一些可以帮助我们实现生成UUID的不同算法的库。

## 2. UUID和基于时间的UUID

**UUID代表通用唯一标识符**，它是一个128位标识符，每次生成时都应是唯一的。

我们用它们来唯一地标识某物，即使该物没有固有标识符。我们可以在各种需要唯一标识对象的环境中使用它们，例如计算机系统、数据库和分布式系统。

两个UUID相同的可能性非常小，从统计上来说是不可能的，这使得它们成为在分布式系统中识别对象的可靠方法。

基于时间的UUID(也称为版本1 UUID)是使用当前时间和生成UUID的计算机或网络特有的唯一标识符生成的。即使同时生成多个UUID，时间戳也能确保UUID是唯一的。

我们将在下面实现的库中找到与时间相关的两个新版本的标准(v6和v7)。

版本1具有几个优点-按时间排序的ID更适合作为表中的主键，并且包含创建时间戳有助于分析和调试。但它也有一些缺点-从同一主机生成多个ID时发生冲突的可能性略高，我们稍后会看看这是否是个问题。

此外，包含主机地址可能会存在一些安全漏洞，这就是标准第6版试图提高安全性的原因。

## 3. 基准

为了使我们的比较更加直接，让我们编写一个基准程序来比较碰撞的可能性和UUID的生成时间。我们首先初始化所有必要的变量：

```java
int threadCount = 128;
int iterationCount = 100_000; 
Map<UUID, Long> uuidMap = new ConcurrentHashMap<>();
AtomicLong collisionCount = new AtomicLong();
long startNanos = System.nanoTime();
CountDownLatch endLatch = new CountDownLatch(threadCount);
```

我们将在128个线程上运行基准测试，每个线程进行100,000次迭代。此外，我们将使用[ConcurrentHashMap](https://www.baeldung.com/java-concurrent-map)来存储生成的所有UUID。除此之外，我们将使用计数器来记录冲突。为了检查速度性能，我们在执行开始时存储当前时间戳，以便将其与最终时间戳进行比较。最后，我们声明一个CountDownLatch来等待所有线程完成。

初始化测试所需的所有变量后，我们将循环并启动每个线程：

```java
for (long i = 0; i < threadCount; i++) {
    long threadId = i;
    new Thread(() -> {
        for (long j = 0; j < iterationCount; j++) {
            UUID uuid = UUID.randomUUID();
            Long existingUUID = uuidMap.put(uuid, (threadId * iterationCount) + j);
            if (existingUUID != null) {
                collisionCount.incrementAndGet();
            }
        }
        endLatch.countDown();
    }).start();
}
```

对于每行执行，我们将再次集成并开始使用java.util.UUID类生成UUID。我们将所有ID及其对应的计数插入到Map中，UUID将是Map的键。

因此，如果我们尝试在Map中插入现有UUID，put()方法将返回已存在的键。当我们获得重复的UUID时，我们将增加冲突计数。在迭代结束时，我们将减少CountDownLatch计数。

```java
endLatch.await();
System.out.println(threadCount * iterationCount + " UUIDs generated, " + collisionCount + " collisions in "
        + TimeUnit.NANOSECONDS.toMillis(System.nanoTime() - startNanos) + "ms");
```

最后，我们将使用[CountDownLatch](https://www.baeldung.com/java-countdown-latch)类的await()方法等待所有线程完成。我们将打印基准测试的结果，其中包括生成的UUID数量、冲突数量和执行时间。

现在，让我们针对JDK的内置UUID生成器运行基准测试：

```text
12800000 UUIDs generated, 0 collisions in 4622ms
```

我们可以看到，所有ID的生成都没有发生冲突。在以下部分中，我们将与其他生成器进行比较。

## 4. UUID Creator

### 4.1 依赖

Java [UUIDCreator](https://github.com/f4b6a3/uuid-creator)库对于生成UUID非常有用且灵活，它提供了各种生成UUID的选项，其简单的API使其易于在各种应用程序中使用。我们可以将该[库](https://mvnrepository.com/artifact/com.github.f4b6a3/uuid-creator)添加到我们的项目中：

```xml
<dependency>
    <groupId>com.github.f4b6a3</groupId>
    <artifactId>uuid-creator</artifactId>
    <version>5.2.0</version>
</dependency>
```

### 4.2 使用

该库为我们提供了三种生成基于时间的UUID的方法：

- UuidCreator.getTimeBased()：RFC-4122中指定的基于公历纪元的时间版本
- UuidCreator.getTimeOrdered()：提议将公历时间排序的版本作为新的UUID格式
- UuidCreator.getTimeOrderedEpoch()：提议以Unix纪元作为新UUID格式的时间顺序版本

添加依赖项后，我们可以在代码中直接使用它们：

```java
System.out.println("UUID Version 1: " + UuidCreator.getTimeBased());
System.out.println("UUID Version 6: " + UuidCreator.getTimeOrdered());
System.out.println("UUID Version 7: " + UuidCreator.getTimeOrderedEpoch());
```

我们可以在输出中看到，这三个都具有相同的经典UUID格式：

```text
UUID Version 1: 0da151ed-c82d-11ed-a2f6-6748247d7506
UUID Version 6: 1edc82d0-da0e-654b-9a98-79d770c05a84
UUID Version 7: 01870603-f211-7b9a-a7ea-4a98f5320ff8
```

**本文将重点介绍使用传统版本1 UUID的getTimeBased()方法**。它包含三部分：时间戳、时钟序列和节点标识符。

```text
Time-based UUID structure

 00000000-0000-v000-m000-000000000000
|1-----------------|2---|3-----------|

1: timestamp
2: clock-sequence
3: node identifier
```

### 4.3 基准

在本节中，我们将运行上一节中的基准测试，但我们将使用UuidCreator.getTimeBased()方法生成UUID。之后，我们得到结果：

```text
12800000 UUIDs generated, 0 collisions in 2595ms
```

我们可以看到，该算法还成功生成了所有没有重复的UUID。除此之外，它甚至比JDK的运行时间更短。这只是一个基本的基准测试，不过还有[更详细的基准测试](https://github.com/f4b6a3/uuid-creator/wiki/5.0.-Benchmark)可用。

## 5. Java UUID Generator(JUG)

### 5.1 依赖

[Java UUID Generator(JUG)](https://github.com/cowtowncoder/java-uuid-generator)是一组用于处理UUID的Java类。它包括使用标准方法生成UUID、高效输出、排序等，它根据[UUID规范(RFC-4122)](https://tools.ietf.org/html/rfc4122)生成UUID。

要使用该库，我们应该添加[Maven依赖](https://mvnrepository.com/artifact/com.fasterxml.uuid/java-uuid-generator)：

```xml
<dependency>
    <groupId>com.fasterxml.uuid</groupId>
    <artifactId>java-uuid-generator</artifactId>
    <version>4.1.0</version>
</dependency>
```

### 5.2 使用

该库还提供了三种方法来创建基于时间的UUID(经典版本1以及新版本6和7)。我们可以通过选择一种生成器，然后调用其generate()方法来生成它们：

```java
System.out.println("UUID Version 1: " + Generators.timeBasedGenerator().generate());
System.out.println("UUID Version 6: " + Generators.timeBasedReorderedGenerator().generate());
System.out.println("UUID Version 7: " + Generators.timeBasedEpochGenerator().generate());
```

然后我们可以在控制台中检查UUID：

```text
UUID Version 1: e6e3422c-c82d-11ed-8761-3ff799965458
UUID Version 6: 1edc82de-6e34-622d-8761-dffbc0ff00e8
UUID Version 7: 01870609-81e5-793b-9e4f-011ee370187b
```

### 5.3 基准

与上一节一样，我们将重点介绍此库提供的第一个UUID变体。我们还可以通过将上一个示例中的UUID生成替换为以下内容来测试发生碰撞的可能性：

```java
UUID uuid = Generators.timeBasedGenerator().generate();
```

运行代码后我们可以看到结果：

```text
12800000 UUIDs generated, 0 collisions in 15795ms
```

从中我们可以看到，我们也没有得到重复的UUID，就像前面的例子一样。但同时，我们也看到了执行时间的差异。即使差异看起来很大，两个库都很快生成了许多ID。

该库的文档告诉我们生成速度不太可能成为瓶颈，并且根据包和API的稳定性进行选择更好。

## 6. 总结

在本教程中，我们了解了基于时间的UUID的结构、优点和缺点。我们使用两个最流行的UUID库在代码中实现了它们，然后对它们进行了比较。

我们发现，选择UUID或库的类型可能取决于我们的需求。