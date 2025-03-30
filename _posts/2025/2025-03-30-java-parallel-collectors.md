---
layout: post
title:  Java Parallel Collectors库指南
category: libraries
copyright: libraries
excerpt: Parallel Collectors
---

## 1. 简介

[Parallel-collectors](https://github.com/pivovarit/parallel-collectors)是一个小型库，它提供一组支持并行处理的Java Stream API收集器，同时规避了标准并行流的主要缺陷。

## 2. Maven依赖

如果我们想开始使用这个库，我们需要在Maven的pom.xml文件中添加一个条目：

```xml
<dependency>
    <groupId>com.pivovarit</groupId>
    <artifactId>parallel-collectors</artifactId>
    <version>1.1.0</version>
</dependency>
```

或者Gradle构建文件中的一行：

```groovy
compile 'com.pivovarit:parallel-collectors:1.1.0'
```

最新版本可以在[Maven Central](https://mvnrepository.com/artifact/com.pivovarit/parallel-collectors)上找到。

## 3. 并行流注意事项

并行流是Java 8的亮点之一，但事实证明它们只适用于繁重的CPU处理。

这样做的原因是，**并行流由JVM范围内共享的ForkJoinPool内部支持，它提供有限的并行性**，并由在单个JVM实例上运行的所有并行流使用。

例如，假设我们有一个ID列表，我们想用它们来获取用户列表。假设该操作很昂贵并且实际上[值得并行化](https://www.baeldung.com/java-when-to-use-parallel-stream)，我们可能希望使用并行流来实现我们的目的：

```java
List<Integer> ids = Arrays.asList(1, 2, 3); 
List<String> results = ids.parallelStream() 
    .map(i -> fetchById(i)) // each operation takes one second
    .collect(Collectors.toList()); 

System.out.println(results); // [user-1, user-2, user-3]
```

事实上，我们可以看到速度明显提升。但是如果我们开始并行运行多个阻塞操作，就会出现问题。这可能会很快使池饱和，并导致潜在的巨大延迟。这就是为什么通过创建单独的线程池来构建隔离层很重要的原因-以防止不相关的任务相互影响彼此的执行。

为了提供自定义的ForkJoinPool实例，我们可以利用[此处](https://www.baeldung.com/java-8-parallel-streams-custom-threadpool)描述的技巧，但这种方法依赖于未记录的hack，并且在JDK 10之前是错误的。我们可以在问题本身中阅读更多内容–[JDK8190974](https://bugs.openjdk.java.net/browse/JDK-8190974)。

## 4. 并行收集器的实际应用

**顾名思义，并行收集器只是标准的Stream API收集器，允许在collect()阶段并行执行其他操作**。

ParallelCollectors(它反映了Collectors类)类是一个门面，提供对库的全部功能的访问。

如果我们想重做上面的例子，我们可以简单地写：

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

List<Integer> ids = Arrays.asList(1, 2, 3);

CompletableFuture<List<String>> results = ids.stream()
    .collect(ParallelCollectors.parallelToList(i -> fetchById(i), executor, 4));

System.out.println(results.join()); // [user-1, user-2, user-3]
```

结果是一样的，但是，我们能够提供自定义线程池并指定自定义并行级别，并且结果包装在[CompletableFuture](https://www.baeldung.com/java-completablefuture)实例中，而不会阻塞当前线程。

另一方面，标准并行流无法实现任何这些功能。

## 4.1 使用标准Collectors并行收集

直观地说，如果我们想要并行处理Stream并收集结果，我们可以简单地使用ParallelCollectors.parallel并提供所需的Collector，就像标准Stream API一样：

```java
List ids = Arrays.asList(1, 2, 3);

CompletableFuture<List> results = ids.stream()
    .collect(parallel(i -> fetchById(i), toList(), executor, 4));

assertThat(results.join()).containsExactly("user-1", "user-2", "user-3");
```

## 4.2 使用Stream进行并行收集

如果前面提到的API方法不够灵活，我们可以将所有元素收集到Stream实例中，然后像CompletableFuture中的任何其他Stream实例一样处理它：

```java
List ids = Arrays.asList(1, 2, 3);

CompletableFuture<Stream> results = ids.stream()
    .collect(parallel(i -> fetchById(i), executor, 4));

assertThat(results.join()).containsExactly("user-1", "user-2", "user-3");
```

## 4.3 ParallelCollectors.parallelToStream()

上面的例子主要关注需要接收包装在CompletableFuture中的结果的用例，但如果我们只想阻塞调用线程并按完成顺序处理结果，我们可以使用parallelToStream()：

```java
List ids = Arrays.asList(1, 2, 3);

Stream result = ids.stream()
    .collect(parallelToStream(i -> fetchByIdWithRandomDelay(i), executor, 4));

assertThat(result).contains("user-1", "user-2", "user-3");
```

在这种情况下，由于我们引入了随机处理延迟，因此我们可以预期收集器每次都会返回不同的结果。因此，我们在测试中加入了contains()断言。

## 4.4 ParallelCollectors.parallelToOrderedStream()

如果我们想确保元素按照原始顺序进行处理，我们可以利用parallelToOrderedStream()：

```java
List ids = Arrays.asList(1, 2, 3);

Stream result = ids.stream()
    .collect(parallelToOrderedStream(ParallelCollectorsUnitTest::fetchByIdWithRandomDelay, executor, 4));

assertThat(result).containsExactly("user-1", "user-2", "user-3");
```

在这种情况下，收集器将始终保持顺序，但可能比上述速度慢。

## 5. 限制

在撰写本文时，**即使使用短路操作，并行收集器也无法处理无限流**-这是Stream API内部设计的限制。简而言之，Stream将收集器视为非短路操作，因此流需要在终止之前处理所有上游元素。

另一个限制是**短路操作不会中断短路后的剩余任务**-这是CompletableFuture不会将中断传播到正在执行的线程的限制之一。

## 6. 总结

我们看到了parallel-collectors库如何允许我们使用自定义Java Stream API收集器和CompletableFutures来执行并行处理，以利用自定义线程池、并行性和CompletableFutures的非阻塞风格。