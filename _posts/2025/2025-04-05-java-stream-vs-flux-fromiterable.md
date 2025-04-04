---
layout: post
title:  比较Java Stream和Flux.fromIterable()
category: reactor
copyright: reactor
excerpt: Reactor
---

## 1. 简介

在本教程中，我们将比较Java [Stream](https://www.baeldung.com/java-8-streams-introduction)和[Flux.fromIterable()](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)。我们将首先探索它们的相似之处，然后深入研究它们的主要区别。

## 2. Java Stream概述

Stream表示支持函数式操作的元素序列，**Stream实例是同步、基于拉取的，并且仅在调用终端操作时才处理元素**。

让我们看看它的主要特点：

- 支持顺序和并行执行
- 使用[函数式](https://www.baeldung.com/java-functional-programming)操作(map()、filter()等)
- 惰性执行(应用终端操作时执行)
- 同步和基于拉取

## 3. Flux.fromIterable()概述

Flux.fromIterable()是[Project Reactor](https://www.baeldung.com/reactor-core)中的一个工厂方法，它从现有的Iterable(例如List、Set或Collection)创建[Flux](https://www.baeldung.com/java-reactor-flux-vs-mono#what-is-flux)。当我们有一个数据集合并想要响应式处理它时，它很有用。

以下是其主要特点：

- 支持[异步](https://www.baeldung.com/java-asynchronous-programming)和非阻塞处理
- 提供[响应式编程](https://www.baeldung.com/cs/reactive-programming)操作(map()、filter()、flatMap()等)
- 基于推送-数据一可用就会发出

## 4. Stream和Flux.fromIterable()之间的相似之处

尽管Stream和Flux.fromIterable()是基于不同的范式构建的，但它们也具有一些共同点。让我们仔细看看这些共同点，并了解它们在数据处理中是如何协调的。

### 4.1 函数式风格

两者都支持函数式编程风格，我们可以使用filter()、map()和reduce()等操作，也可以将多个操作链接在一起，以创建用于处理数据的声明式且可读的管道。

让我们看一个例子，我们过滤一串数字以仅保留偶数，然后将其值加倍：

```java
@Test
void givenList_whenProcessedWithStream_thenReturnDoubledEvenNumbers() {
    List<Integer> numbers = List.of(1, 2, 3, 4, 5);

    List<Integer> doubledEvenNumbers = numbers.stream()
            .filter(n -> n % 2 == 0)
            .map(n -> n * n)
            .toList();
    assertEquals(List.of(4, 16), doubledEvenNumbers);
}
```

现在，让我们使用Flux.fromIterable()执行相同的示例：

```java
@Test
void givenList_whenProcessedWithFlux_thenReturnDoubledEvenNumbers() {
    List<Integer> numbers = List.of(1, 2, 3, 4, 5);
    Flux<Integer> fluxPipeline = Flux.fromIterable(numbers)
            .filter(n -> n % 2 == 0)
            .map(n -> n * 2);

    StepVerifier.create(fluxPipeline)
            .expectNext(4, 16);
}
```

### 4.2 惰性求值

Stream和Flux实例都是惰性的，这意味着只有在真正需要结果时才执行操作。

对于Stream实例，需要一个终端运算符来执行管道：

```java
@Test
void givenList_whenNoTerminalOperator_thenNoResponse() {
    List<Integer> numbers = List.of(1, 2, 3, 4, 5);
    Function<Integer, Integer> mockMapper = mock(Function.class);
    Stream<Integer> streamPipeline = numbers.stream()
            .map(mockMapper);
    verifyNoInteractions(mockMapper);

    List<Integer> mappedList = streamPipeline.toList();
    verify(mockMapper, times(5));
}
```

在这个例子中，我们可以看到，在调用终端运算符toList()之前，没有与mockMapper函数的交互。

类似地，Flux仅在有订阅者时才开始处理：

```java
@Test
void givenList_whenFluxNotSubscribed_thenNoResponse() {
    Function<Integer, Integer> mockMapper = mock(Function.class);
    when(mockMapper.apply(anyInt())).thenAnswer(i -> i.getArgument(0));
    List<Integer> numbers = List.of(1, 2, 3, 4, 5);
    Flux<Integer> flux = Flux.fromIterable(numbers)
            .map(mockMapper);

    verifyNoInteractions(mockMapper);

    StepVerifier.create(flux)
            .expectNextCount(5)
            .verifyComplete();

    verify(mockMapper, times(5)).apply(anyInt());
}
```

在这个例子中，我们可以验证Flux最初没有与mockMapper交互。但是，一旦我们使用StepVerifier订阅它，我们就可以观察到Flux开始与其方法交互。

## 5. Stream和Flux.fromIterable()之间的主要区别

尽管Stream和Flux.fromIterable()之间有一些相似之处，但它们的目的、性质以及处理元素的方式有所不同，让我们探讨一下这两者之间的一些主要区别。

### 5.1 同步与异步

Stream和Flux.fromIterable()之间的一个主要区别是它们如何处理执行。

**Stream同步运行，这意味着计算在调用终端操作的同一线程上按顺序进行**。如果我们使用Stream处理大型数据集或执行耗时任务，它可能会阻塞线程直到完成。没有内置方法可以异步执行Stream实例，除非明确使用[parallelStream()](https://www.baeldung.com/java-when-to-use-parallel-stream)或手动管理线程。即使在并行模式下，流也以阻塞方式运行。

另一方面，Flux.fromIterable()侧重于同步而非异步。默认情况下，它的行为是同步的-在调用线程上发出元素，类似于顺序流。但是，Flux提供了对异步和非阻塞执行的内置支持，可以使用subscribeOn()、publishOn()和delayElements()等运算符启用。这允许Flux并发处理元素而不阻塞主线程，使其更适合响应式应用程序。

让我们检查一下Flux.fromIterable()的异步行为：

```java
@Test
void givenList_whenProcessingTakesLongerThanEmission_thenEmittedBeforeProcessing() {
    VirtualTimeScheduler.set(VirtualTimeScheduler.create());

    List<Integer> numbers = List.of(1, 2, 3, 4, 5);
    Flux<Integer> sourceFlux = Flux.fromIterable(numbers)
            .delayElements(Duration.ofMillis(500));

    Flux<Integer> processedFlux = sourceFlux.flatMap(n ->
            Flux.just(n * n)
                    .delayElements(Duration.ofSeconds(1))
    );

    StepVerifier.withVirtualTime(() -> Flux.merge(sourceFlux, processedFlux))
            .expectSubscription()
            .expectNoEvent(Duration.ofMillis(500))
            .thenAwait(Duration.ofMillis(500 * 5))
            .expectNextCount(7)
            .thenAwait(Duration.ofMillis(5000))
            .expectNextCount(3)
            .verifyComplete();
}
```

在上面的例子中，我们每500ms发出一次数字，并将处理延迟1秒。使用merge()，我们合并两个Flux实例的结果。在2500ms结束时，所有元素都已发出，并且已处理两个元素，因此我们预计总共有7个元素。5秒钟后，所有元素都已处理完毕，因此我们预计剩余3个元素。

这里，发射器不会等待处理器完成其操作，而是继续独立发射值。

### 5.2 异常处理

在Stream中，异常会导致管道立即终止。**如果某个元素在处理过程中抛出异常，则整个流将停止，并且不会再处理其他元素**。

这是因为Stream实例将异常视为终止事件，我们必须依靠map()、filter()中的try-catch块，或在每个阶段使用自定义异常处理逻辑来防止故障传播。

让我们看一下Stream管道在遇到除以0异常时的行为：

```java
@Test
void givenList_whenDividedByZeroInStream_thenThrowException() {
    List<Integer> numbers = List.of(1, 2, 0, 4, 5);
    assertThrows(ArithmeticException.class, () -> numbers.stream()
        .map(n -> 10 / n)
        .toList());
}
```

这里，异常立即终止管道，并且4和5永远不会被处理。

相比之下，**Flux将错误视为数据，这意味着错误将通过单独的错误通道传播，而不是直接终止管道**。这使我们能够使用Flux中提供的onErrorResume()、onErrorContinue()和onErrorReturn()等内置方法优雅地处理异常，确保即使个别元素失败，处理仍可继续。

现在，让我们看看如何使用Flux.fromIterable()进行相同的处理：

```java
@Test
void givenList_whenDividedByZeroInFlux_thenReturnFallbackValue() {
    List<Integer> numbers = List.of(1, 2, 0, 4, 5);
    Flux<Integer> flux = Flux.fromIterable(numbers)
        .map(n -> 10 / n)
        .onErrorResume(e -> Flux.just(-1));

    StepVerifier.create(flux)
        .expectNext(10, 5, -1)
        .verifyComplete();
}
```

这里捕获了异常，Flux并没有失败，而是发出了一个回退值(-1)。**这使Flux更具弹性，使其能够处理网络故障、数据库错误或意外输入等现实场景**。

### 5.3 单管道与多个订阅者

Stream的一个基本限制是它代表一次性管道，**一旦调用终端操作(如forEach()或collect())，Stream就会被消耗，无法重复使用**。每次我们需要处理相同的数据集时，都必须创建一个新的Stream：

```java
@Test
void givenStream_whenReused_thenThrowException() {
    List<Integer> numbers = List.of(1, 2, 3, 4, 5);
    Stream<Integer> doubleStream = numbers.stream()
        .map(n -> n * 2);

    assertEquals(List.of(2, 4, 6, 8, 10), doubleStream.toList());

    assertThrows(IllegalStateException.class, doubleStream::toList);
}
```

在上面的例子中，当我们第一次调用Stream上的终止操作toList()时，它返回了预期的结果。但是，当我们尝试再次调用它时，它会抛出IllegalStateException，因为Stream实例一旦被使用就无法重用。

另一方面，**Flux支持多个订阅者，允许不同的消费者独立地对同一数据做出响应**。这实现了事件驱动架构，其中系统中的多个组件可以响应同一数据流而无需重新加载源。

现在，让我们使用Flux执行相同的操作并看看它的行为：

```java
@Test
void givenFlux_whenMultipleSubscribers_thenEachReceivesData() {
    List<Integer> numbers = List.of(1, 2, 3, 4, 5);
    Flux<Integer> flux = Flux.fromIterable(numbers).map(n -> n * 2);
    StepVerifier.create(flux)
        .expectNext(2, 4, 6, 8, 10)
        .verifyComplete();

    StepVerifier.create(flux)
        .expectNext(2, 4, 6, 8, 10)
        .verifyComplete();
}
```

这里我们可以看到每个订阅者都收到了相同的数据，并且没有抛出任何异常。这使得Flux比Stream更加通用，特别是在多个组件需要监听和处理相同数据而又不需要重复源的场景中。

### 5.4 性能

**Stream实例针对高性能内存处理进行了优化，它们运行迅速，这意味着所有转换都在数据的一次传递中发生**。

此外，Stream支持使用parallelStream()进行并行处理，通过[ForkJoinPool](https://www.baeldung.com/java-fork-join)利用多个CPU核心进一步提高大数据集的执行速度。

另一方面，**Flux.fromIterable()是为响应式数据处理而设计的，这使得它更加灵活，但在内存计算方面本质上速度较慢**。

由于Flux本质上是异步的，每个数据元素都包裹在响应信号(onNext、onError、onComplete)中，从而增加了处理开销。

**Flux遵循非阻塞执行模型，这对于I/O密集型操作有益，但对于纯计算任务效率较低**。

## 6. 功能比较：Java Stream与Flux.fromIterable()

|  特征   |         Java Stream          |      Flux.fromIterable()       |
|:-----:|:----------------------------:|:------------------------------:|
| 执行模型  |       基于拉取-消费者在需要时请求数据       |       基于推送-生产者异步向消费者推送数据       |
| 处理风格  |           函数式、基于管道           |          函数式、响应式、基于事件          |
| 同步或异步 | 同步-可以使用parallelStream()进行并行化 |       异步-可以在多个线程上执行而不会阻塞       |
| 错误处理  |   没有内置支持，我们需要使用try-catch块    |      错误被视为数据并通过单独的错误通道传播       |
| 多个订阅者 |      不支持-一旦使用，流就无法重复使用       |        多个订阅者可以监听同一个Flux        |
|  用例   |      适用于快速、CPU密集型、内存转换       | 当我们需要异步和非阻塞等响应式功能、响应式地处理错误和批处理 |

## 7. 总结

在本文中，我们探讨了Java的Stream和Reactor的Flux.fromIterable()之间的相似之处和主要区别，了解这些区别使我们能够根据应用程序的需求选择正确的方法。