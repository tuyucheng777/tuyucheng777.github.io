---
layout: post
title:  在Spring WebFlux中取消正在进行的Flux
category: springreactive
copyright: springreactive
excerpt: Spring WebFlux
---

## 1. 简介

在本文中，我们将讨论[Spring WebFlux](https://www.baeldung.com/spring-webflux)提供的用于取消正在进行的Flux的各种选项。首先，我们将在[响应式编程](https://www.baeldung.com/java-reactive-systems)的背景下快速概述Flux，接下来，我们将研究取消正在进行的Flux的必要性。

我们将研究Spring WebFlux提供的各种方法来显式和自动取消订阅，我们将使用JUnit测试来驱动我们的简单示例，以验证系统是否按预期运行。最后，我们将了解如何执行取消后清理，以便在取消后将系统重置到所需状态。

让我们首先快速概述一下Flux。

## 2. 什么是Flux？

**Spring WebFlux是一个响应式Web框架，它提供了用于构建异步、非阻塞应用程序的强大功能**。Spring WebFlux的主要功能之一是它能够处理Flux，[Flux](https://www.baeldung.com/java-reactor-flux-vs-mono)是一种可以发出0个或多个元素的响应式数据流。它可以从各种来源创建，例如数据库查询、网络调用或内存集合。

**在本文中，我们应该了解一个相关术语：订阅，它表示数据源(即发布者)和数据消费者(即订阅者)之间的连接。订阅会维护一个状态，该状态反映订阅是否处于活动状态。它可用于取消订阅，这将停止Flux的数据发射并释放发布者持有的任何资源**，我们可能需要取消正在进行的订阅的一些潜在场景包括用户取消请求或发生超时等。

## 3. 取消正在进行的Flux的好处

在Reactive Spring WebFlux中，取消正在进行的Flux非常重要，以确保高效利用系统资源并防止潜在的内存泄漏，原因如下：

- **背压**：响应式编程使用[背压](https://www.baeldung.com/spring-webflux-backpressure)来调节发布者和订阅者之间的数据流，如果订阅者无法跟上发布者的步伐，则使用背压来减慢或停止数据流。**如果正在进行的订阅未被取消，即使订阅者没有消费数据，它也会继续生成数据，从而导致背压累积并可能导致内存泄漏**。
- **资源管理**：它可以保存内存、CPU和网络连接等系统资源，如果不加以控制，可能会导致资源耗尽。**可以通过取消订阅来释放系统资源，以便稍后用于其他任务**。
- **性能**：**通过提前终止订阅，系统可以避免不必要的处理并减少响应时间，从而提高整体系统性能**。

## 4. Maven依赖

让我们举一个非常简单的例子，一些传感器数据以Flux的形式发送，我们希望使用WebFlux提供的各种选项根据订阅取消数据发射。

首先，我们需要添加以下关键依赖：

-  [spring-boot-starter-webflux](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-webflux)：它捆绑了使用Spring WebFlux开始构建响应式Web应用程序所需的所有依赖，包括用于响应式编程的Reactor库和作为默认嵌入式服务器的Netty。
- [react-spring](https://mvnrepository.com/artifact/org.projectreactor/reactor-spring)：是[Reactor项目](https://www.baeldung.com/reactor-core)中的一个模块，提供与Spring框架的集成。
- [react-test](https://mvnrepository.com/artifact/io.projectreactor/reactor-test)：为Reactive流提供测试支持。

现在，让我们在项目[POM](https://www.baeldung.com/maven)中声明这些依赖：
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    <dependency>
        <groupId>org.projectreactor</groupId>
        <artifactId>reactor-spring</artifactId>
        <version>${reactor-spring.version}</version>
    </dependency>
    <dependency>
        <groupId>io.projectreactor</groupId>
        <artifactId>reactor-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

## 5. 在WebFlux中取消正在进行的Flux

在Spring WebFlux中，我们可以使用dispose()显式取消，也可以使用在Subscription对象上调用cancel()的某些运算符时隐式取消，这些运算符包括：

- takeUntil()
- takeWhile()
- take(long n)
- take(Duration n)

**进一步研究，我们会发现这些运算符在内部调用Subscription对象上的cancel()方法，并将其作为参数传递给Subscription的OnSubscribe()方法**。

接下来我们来讨论一下这些运算符。

### 5.1 使用takeUntil()运算符取消

让我们以传感器数据为例，我们希望继续从数据流接收数据，直到遇到值8，此时我们希望取消发送任何数据：
```typescript
@Test
void givenOngoingFlux_whentakeUntil_thenFluxCancels() {
    Flux<Integer> sensorData = Flux.range(1, 10);
    List<Integer> result = new ArrayList<>();

    sensorData.takeUntil(reading -> reading == 8)
        .subscribe(result::add);
    assertThat(result).containsExactly(1, 2, 3, 4, 5, 6, 7, 8);
}
```

此代码片段使用Flux API创建整数流并使用各种运算符对其进行操作，首先，使用Flux.range()创建从1到10的整数序列。然后，应用takeUntil()运算符，该运算符需要一个谓词来指定Flux应持续发出整数，直到值达到8。

**最后，调用subscribe()方法，它会导致Flux发出值，直到takeUntil()谓词的计算结果为true**。在subscribe()方法中，发出的每个新整数都会添加到List<Integer\>中，从而可以捕获和操作发出的值。

**需要注意的是，subscribe()方法对于触发Flux的值的发出至关重要，如果没有它，Flux就不会发出任何值，因为Flux没有订阅。一旦takeUntil()运算符指定的条件为true，订阅就会自动取消，Flux也会停止发出值**。测试结果证实，结果列表仅包含最多8个整数值，这表明任何后续数据发出均已取消。

### 5.2 使用takeWhile()运算符取消

**接下来，让我们考虑这样一种情况：只要传感器读数保持小于8，我们希望订阅能够持续发送数据**。这时，我们可以利用takeWhile()运算符，它需要以下延续谓词：
```java
@Test
void givenOngoingFlux_whentakeWhile_thenFluxCancels() {
    List<Integer> result = new ArrayList<>();
    Flux<Integer> sensorData = Flux.range(1, 10)
        .takeWhile(reading -> reading < 8)
        .doOnNext(result::add);

    sensorData.subscribe();
    assertThat(result).containsExactly(1, 2, 3, 4, 5, 6, 7);
}
```

本质上，这里的takeWhile()运算符也需要一个谓词，只要谓词的计算结果为true，数据流就会发出数据。**一旦谓词的计算结果为false，订阅就会被取消，并且不会再发出任何数据**。请注意，在这里，我们在设置流时使用了doOnNext()方法将每个发出的值添加到列表中。

之后，我们调用sensorData.subscribe()。

### 5.3 使用take(long n)运算符取消

**接下来，让我们看一下take()运算符，它可以限制我们想要从可能无限的响应数据流序列中获取的元素数量**。让我们取一个从1到整数最大值的整数流，然后取前10个元素：
```java
@Test
void givenOngoingFlux_whentake_thenFluxCancels() {
    Flux<Integer> sensorData = Flux.range(1, Integer.MAX_VALUE);
    List<Integer> result = new ArrayList<>();

    sensorData.take(10)
        .subscribe(result::add);
    Assertions.assertThat(result).containsExactly(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
}
```

**这里，订阅再次在前10个元素之后被取消，我们的result列表证实了这一点**。

### 5.4 使用take(Duration d)运算符取消

**另一个我们可能想要取消任何进一步数据发射的潜在场景是，当一段时间过去后，我们不再对任何进一步的发射感兴趣**。在这种情况下，我们查看Flux的持续时间，然后停止接收持续时间之外的任何数据：
```java
@Test
void givenAnOnGoingFlux_whenTimeout_thenCancelsFlux() {
    Flux<Integer> sensorData = Flux.interval(Duration.ZERO, Duration.ofSeconds(2))
        .map(i -> i.intValue() + 10)
        .take(5);

    Flux<Integer> canceledByTimeout = sensorData.take(Duration.ofSeconds(3));

    StepVerifier.create(canceledByTimeout)
        .expectNext(10, 11)
        .expectComplete()
        .verify();
}
```

首先，我们使用interval()运算符创建一个整数Flux，该Flux从0开始以每2秒的间隔发出值。然后，我们通过将每个发出的值加10映射到一个整数。**接下来，我们使用take()运算符将发出的值的数量限制为5，这意味着Flux将仅发出前5个值，然后完成**。

**然后，我们通过应用take(Duration)运算符并以3秒的持续时间值创建一个名为canceledBytimeOut的新Flux**，这意味着canceledBytimeoutFlux将从传感器数据中发出前2个值，然后完成。

**这里我们使用StepVerifier，StepVerifier是Reactor Test库提供的一个实用程序，它通过设置对预期事件的期望，然后验证事件是否按预期顺序和预期值发出，来帮助验证Flux或Mono流的行为**。

在我们的例子中，预期的顺序和值是10和11，并且我们还使用expectComplete()验证Flux是否完成而没有发出任何其他值。

**需要注意的是，subscribe()方法没有被显式调用，因为它在我们调用verify()时被内部调用。这意味着事件仅在运行StepVerifier时才会发出，而不是在我们创建Flux流时发出**。

### 5.5 使用dispose()方法取消

接下来，让我们看看如何通过调用属于Disposable接口的dispose()来显式取消。**简而言之，Disposable是一个提供单向取消机制的接口**，它可以处理资源或取消订阅。

让我们举一个例子，其中我们有一个整数Flux，它以1秒的延迟发出从1到10的值，我们将订阅Flux以在控制台上打印值，然后我们将使线程休眠5毫秒，然后调用dispose()：
```java
@Test
void giveAnOnGoingFlux_whenDispose_thenCancelsFluxExplicitly() throws InterruptedException {
    Flux<Integer> flux = Flux.range(1, 10)
      .delayElements(Duration.ofSeconds(1));

    AtomicInteger count = new AtomicInteger(0);
    Disposable disposable = flux.subscribe(i -> {
        System.out.println("Received: " + i);
        count.incrementAndGet();
    }, e -> System.err.println("Error: " + e.getMessage())
    );

    Thread.sleep(5000);
    System.out.println("Will Dispose the Flux Next");
    disposable.dispose();
    if(disposable.isDisposed()) {
        System.out.println("Flux Disposed");
    }
    assertEquals(4, count.get());
}

```

在这里，我们让线程休眠5秒，然后调用dispose()，这将取消订阅。

## 6. 取消后的清理

**重要的是要理解，取消正在进行的订阅不会隐式释放任何相关资源**。但是，一旦流被取消或完成，就必须进行清理和状态重置，我们可以使用提供的doOnCancel()和doFinally()方法来实现这一点：

为了简化测试，我们将在取消flux后打印相应的消息。**但是，在实际场景中，此步骤可以执行任何资源清理操作，例如关闭连接**。

让我们快速测试一下，当Flux被取消时，我们想要的字符串是否会作为取消后清理的一部分被打印出来：
```java
@Test
void givenAFluxIsCanceled_whenDoOnCancelAndDoFinally_thenMessagePrinted() throws InterruptedException {

    List<Integer> result = new ArrayList<>();
    PrintStream mockPrintStream = mock(PrintStream.class);
    System.setOut(mockPrintStream);

    Flux<Integer> sensorData = Flux.interval(Duration.ofMillis(100))
            .doOnCancel(() -> System.out.println("Flux Canceled"))
            .doFinally(signalType -> {
                if (signalType == SignalType.CANCEL) {
                    System.out.println("Flux Completed due to Cancelation");
                } else {
                    System.out.println("Flux Completed due to Completion or Error");
                }
            })
            .map(i -> ThreadLocalRandom.current().nextInt(1, 1001))
            .doOnNext(result::add);

    Disposable subscription = sensorData.subscribe();

    Thread.sleep(1000);
    subscription.dispose();

    ArgumentCaptor<String> captor = ArgumentCaptor.forClass(String.class);
    Mockito.verify(mockPrintStream, times(2)).println(captor.capture());

    assertThat(captor.getAllValues()).contains("Flux Canceled", "Flux Completed due to Cancelation");
}
```

这段代码同时调用了doOnCancel()和doFinally()运算符，**需要注意的是，只有当Flux序列被明确取消时，doOnCancel()运算符才会执行。另一方面，无论被取消、成功完成还是出现错误，doFinally()运算符都会执行**。

此外，doFinally()运算符使用SignalType接口类型，它表示可能的信号类型，例如OnComplete、OnError和CANCEL。在本例中，SignalType为CANCEL，因此还会捕获“Flux Completed due to Cancelation”消息。

## 7. 总结

在本教程中，我们介绍了Webflux提供的取消正在进行的Flux的各种方法，我们快速回顾了Flux在响应式编程中的应用，分析了可能需要取消订阅的原因。然后，我们讨论了各种方便取消订阅的方法，此外，我们还研究了取消后的清理。