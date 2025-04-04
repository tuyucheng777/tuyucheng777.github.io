---
layout: post
title:  如何在Project Reactor中将List转换为Flux
category: reactor
copyright: reactor
excerpt: Reactor
---

## 1. 概述

在[响应式编程](https://www.baeldung.com/reactor-core)中，通常需要将集合转换为响应式流(称为Flux)。在将现有数据结构集成到响应式管道时，这是至关重要的一步。

在本教程中，我们将探讨如何将元素集合转换为元素流。

## 2. 问题定义

Project Reactor中[Publisher](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/package-summary.html)的两种主要类型是[Flux](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html)和[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)。Mono最多可以发出一个值，而Flux可以发出任意数量的值。

当我们获取List<T\>时，我们可以将其包装在Mono<List<T\>\>中，也可以将其转换为Flux<T\>。返回List<T\>的阻塞调用可以包装在Mono中，从而一次性发出整个列表。

但是，如果我们将如此大的列表放入Flux中，它允许订阅者以可管理的块形式请求数据，这使订阅者可以逐个或小批量地处理元素：

![](/assets/images/2025/springreactive/javareactorconvertlistflux01.png)

来源：projectreactor.io

我们将探索转换已包含类型T元素的List的不同方法。对于我们的用例，我们将考虑使用Publisher类型Flux的运算符[fromIterable](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#fromIterable-java.lang.Iterable-)和[create](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Flux.html#create-java.util.function.Consumer-)将List<T\>转换为Flux<T\>。

## 3. fromIterable

让我们首先创建一个整数类型的列表并向其中添加一些值：

```java
List<Integer> list = List.of(1, 2, 3);
```

fromIterable是FluxPublisher上的运算符，它发出所提供集合中包含的元素。

我们使用log()运算符来记录发布的每个元素：

```java
private <T> Flux<T> listToFluxUsingFromIterableOperator(List<T> list) {
    return Flux
        .fromIterable(list)
        .log();
}
```

然后我们可以将fromIterable运算符应用到我们的整数列表并观察其行为：

```java
@Test
public void givenList_whenCallingFromIterableOperator_thenListItemsTransformedAsFluxAndEmitted(){

    List<Integer> list = List.of(1, 2, 3);
    Flux<Integer> flux = listToFluxUsingFromIterableOperator(list);

    StepVerifier.create(flux)
        .expectNext(1)
        .expectNext(2)
        .expectNext(3)
        .expectComplete()
        .verify();
}
```

最后，我们使用[StepVerifier](https://www.baeldung.com/reactive-streams-step-verifier-test-publisher) API来验证Flux发出的元素与List中的元素是否一致。在完成正在测试的Flux源后，我们使用expectNext方法来交叉引用Flux发出的元素和List中的元素是否相同且遵循相同的顺序。

## 4. create

Flux类型Publisher上的[create](https://www.baeldung.com/java-flux-create-generate)运算符使我们能够以编程方式使用[FluxSink](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/FluxSink.html) API创建Flux。

虽然fromIterable在大多数情况下都是一个不错的选择，但当列表由回调生成时，它并不好用。在这种情况下，使用create运算符更合适。

让我们创建一个回调接口：

```java
public interface Callback<T>  {
    void onTrigger(T element);
}
```

接下来，让我们想象一个从异步API调用返回的List<T\>：

```java
private void asynchronousApiCall(Callback<List<Integer>> callback) {
    Thread thread = new Thread(()-> {
        List<Integer> list = List.of(1, 2,3);
        callback.onTrigger(list);
    });
    thread.start();
}
```

现在，让我们在回调中使用FluxSink而不是fromIterable来将每个元素添加到列表中：

```java
@Test
public void givenList_whenCallingCreateOperator_thenListItemsTransformedAsFluxAndEmitted() {

    Flux<Integer> flux = Flux.create(sink -> {
        Callback<List<Integer>> callback = list -> {
            list.forEach(sink::next);
            sink.complete();
        };
      asynchronousApiCall(callback);
    });

    StepVerifier.create(flux)
        .expectNext(1)
        .expectNext(2)
        .expectNext(3)
        .expectComplete()
        .verify();
}
```

## 5. 总结

在本文中，我们探索了使用Publisher类型Flux中的运算符fromIterable和create将List<T\>转换为Flux<T\>的不同方法，fromIterable运算符可以与类型List<T\>一起使用，也可以与包装在Mono中的List<T\>一起使用。create运算符最适合从回调创建的List<T\>。