---
layout: post
title:  Spring Reactive中的Mono.fromCallable与Mono.justOrEmpty
category: spring-reactive
copyright: spring-reactive
excerpt: Spring Reactive
---

## 1. 概述

在[响应式编程](https://www.baeldung.com/reactor-core)中，处理和转换数据流对于构建响应式应用程序至关重要。创建Mono实例的两种常用方法是Mono.fromCallable和Mono.justOrEmpty，这两种方法都有其独特的用途，具体取决于我们想要如何处理流中的可空性和惰性求值。

在本教程中，我们将探讨这些方法之间的差异，展示Mono.fromCallable如何通过包装计算来延迟执行并优雅地处理错误，而Mono.justOrEmpty如何直接从可选值创建Mono实例，从而简化可能有空数据的情况。

## 2. Mono简介

Mono是Project Reactor中的一个[Publisher](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/package-summary.html)，表示最多发出一个值的流。它可以是完整的，带有值，可以为空，也可以因错误而终止。

Mono支持两种类型的发布者：[冷发布者(cold publisher)和热发布者(hot publisher)](https://projectreactor.io/docs/core/snapshot/reference/advancedFeatures/reactor-hotCold.html)。

**冷发布者只会在消费者订阅后才发布元素**，确保每个消费者从一开始就收到数据，而**热发布者则会在创建后立即发出数据，无论是否订阅**。

Mono的[fromCallable](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#fromCallable-java.util.concurrent.Callable-)是[冷发布者](https://www.baeldung.com/java-mono-defer)的一个示例：订阅后它会延迟返回该Mono。相反，[justOrEmpty](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html#justOrEmpty-java.util.Optional-)是一个表现为热发布者的Mono，它会立即发出数据而无需等待任何订阅。

![](/assets/images/2025/springreactive/springreactivemonofromcallablevsmonojustorempty01.png)

来源：projectreactor.io

## 3. Mono.fromCallable

fromCallable接收一个[Callable](https://www.baeldung.com/java-runnable-callable)接口并返回一个Mono，该Mono会延迟该Callable的执行，直到有对其的订阅。如果Callable解析为null，则生成的Mono为空。

让我们考虑一个示例用例，其中我们在每次方法调用之间以一致的5秒延迟来获取数据。我们将此逻辑设置为延迟，因此它仅在订阅时执行：

```java
public String fetchData() {
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        throw new RuntimeException(e);
    }
    return "Data Fetched";
}
```

接下来，我们将定义一个使用fromCallable创建Mono发布者的方法，timeTakenForCompletion属性测量从订阅开始到收到onComplete信号之间的持续时间：

```java
public void givenDataAvailable_whenCallingFromCallable_thenLazyEvaluation() {
    AtomicLong timeTakenForCompletion = new AtomicLong();
    Mono<String> dataFetched = Mono.fromCallable(this::fetchData)
            .doOnSubscribe(subscription -> timeTakenForCompletion.set(-1 * System.nanoTime()))
            .doFinally(consumer -> timeTakenForCompletion.addAndGet(System.nanoTime()));

    StepVerifier.create(dataFetched)
            .expectNext("Data Fetched")
            .verifyComplete();
}
```

最后，**断言验证从订阅到接收onComplete信号的时间与预期的5秒延迟非常接近**，从而确认fromCallable延迟执行直到订阅：

```java
assertThat(TimeUnit.NANOSECONDS.toMillis(timeTakenForCompletion.get())) 
    .isCloseTo(5000L, Offset.offset(50L));
```

### 3.1 内置错误处理

fromCallable还支持内置错误处理，**如果Callable抛出异常，fromCallable会捕获它，从而允许错误通过响应流传播**。

让我们考虑同一个例子来了解fromCallable的错误处理：

```java
public void givenExceptionThrown_whenCallingFromCallable_thenFromCallableCapturesError() {
    Mono<String> dataFetched = Mono.fromCallable(() -> {
                String data = fetchData();
                if (data.equals("Data Fetched")) {
                    throw new RuntimeException("ERROR");
                }
                return data;
            })
            .onErrorResume(error -> Mono.just("COMPLETED"));

    StepVerifier.create(dataFetched)
            .expectNext("COMPLETED")
            .verifyComplete();
}
```

## 4. Mono.justOrEmpty

justOrEmpty创建一个Mono，该Mono包含一个值，如果值为null则为空。与fromCallable不同，它不会推迟执行，而是在创建Mono后立即执行。

**justOrEmpty不会传播错误，因为它被设计用于处理可空值**。

让我们重新回顾一下之前模拟数据获取的用例，这次，我们将使用justOrEmpty方法来创建一个Mono类型的发布者。

timeTakenToReceiveOnCompleteSignalAfterSubscription属性跟踪从订阅到接收onComplete信号的时间，timeTakenForMethodCompletion属性测量该方法完成所需的总时间：

```java
public void givenDataAvailable_whenCallingJustOrEmpty_thenEagerEvaluation() {
    AtomicLong timeTakenToReceiveOnCompleteSignalAfterSubscription = new AtomicLong();
    AtomicLong timeTakenForMethodCompletion = new AtomicLong(-1 * System.nanoTime());
    Mono<String> dataFetched = Mono.justOrEmpty(fetchData())
            .doOnSubscribe(subscription -> timeTakenToReceiveOnCompleteSignalAfterSubscription
                    .set(-1 * System.nanoTime()))
            .doFinally(consumer -> timeTakenToReceiveOnCompleteSignalAfterSubscription
                    .addAndGet(System.nanoTime()));

    timeTakenForMethodCompletion.addAndGet(System.nanoTime());

    StepVerifier.create(dataFetched)
            .expectNext("Data Fetched")
            .verifyComplete();
}
```

让我们写一个断言来证明从订阅到收到onComplete信号的时间非常短，从而确认Mono是被急切创建的：

```java
assertThat(TimeUnit.NANOSECONDS.toMillis(timeTakenToReceiveOnCompleteSignalAfterSubscription
    .get())).isCloseTo(1L, Offset.offset(1L));
```

接下来，让我们确认5秒的延迟包含在方法的完成时间中，并且发生在订阅Mono之前：

```java
assertThat(TimeUnit.NANOSECONDS.toMillis(timeTakenForMethodCompletion.get()))
    .isCloseTo(5000L, Offset.offset(50L));
```

## 5. 何时使用fromCallable

让我们看看可以使用Mono.fromCallable()方法的用例：

- 当我们需要有条件地订阅发布者以节省资源时
- 每次订阅都可能带来不同的结果
- 当操作有可能引发异常时，我们希望它们通过响应流传播

### 5.1 示例用法

让我们来看一个有条件延迟执行有益的示例用例：

```java
public Optional<String> fetchLatestStatus() {
    List<String> activeStatusList = List.of("ARCHIVED", "ACTIVE");
    if (activeStatusList.contains("ARCHIVED")) {
        return Optional.empty();
    }
    return Optional.of(activeStatusList.get(0));
}
```

```java
public void givenLatestStatusIsEmpty_thenCallingFromCallableForEagerEvaluation() {
    Optional<String> latestStatus = fetchLatestStatus();
    String updatedStatus = "ACTIVE";
    Mono<String> currentStatus = Mono.justOrEmpty(latestStatus)
        .switchIfEmpty(Mono.fromCallable(()-> updatedStatus));

    StepVerifier.create(currentStatus)
        .expectNext(updatedStatus)
        .verifyComplete();
}
```

在此示例中，Mono发布者在switchIfEmpty方法中使用fromCallable进行定义，允许其有条件地执行。因此，只有当Mono被订阅时才会返回ACTIVE状态，使其成为惰性执行。

## 6. 总结

在本文中，我们讨论了Mono.fromCallable方法(充当冷发布者)和Mono.justOrEmpty(充当热发布者)，我们还探讨了何时使用fromCallable方法而不是justOrEmpty，强调了它们的区别并讨论了示例用例。