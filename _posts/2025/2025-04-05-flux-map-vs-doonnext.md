---
layout: post
title:  Flux.map()和Flux.doOnNext()之间的比较
category: reactor
copyright: reactor
excerpt: Reactor
---

## 1. 概述

在[Reactor库](https://www.baeldung.com/reactor-core)中，Flux.map()和Flux.doOnNext()运算符在处理流数据元素时扮演着不同的角色。

Flux.map()运算符有助于转换[Flux](https://www.baeldung.com/reactor-core#1-flux)发出的每个元素；Flux.doOnNext()运算符是一个生命周期钩子，它允许我们在发出每个元素时对其执行副作用。

在本教程中，我们将深入探讨这些运算符的细节，探索它们的内部实现和实际用例。此外，我们还将了解如何一起使用这两个运算符。

## 2. Maven依赖

为了使用Flux发布者和其他响应式运算符，我们需要将[react-core](https://mvnrepository.com/artifact/io.projectreactor/reactor-core)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.6.5</version>
</dependency>
```

此依赖提供了Flux、[Mono](https://www.baeldung.com/reactor-core#2-mono)等核心类。

另外，让我们添加[react-test](https://mvnrepository.com/artifact/io.projectreactor/reactor-test)依赖来帮助我们的单元测试：

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-test</artifactId>
    <version>3.6.5</version>
    <scope>test</scope>
</dependency>
```

上面的依赖提供了像StepVerifier这样的类，它允许我们创建测试场景并断言响应式管道的预期行为。

## 3. 理解Flux.map()运算符

Flux.map()运算符的工作方式与Java内置的[Stream.map()](https://www.baeldung.com/java-8-streams-introduction#3-mapping)类似，但它对响应流进行操作。

### 3.1 图示

让我们通过一张图来了解Flux.map()运算符的内部结构：

![](/assets/images/2025/reactor/fluxmapvsdoonnext01.png)

在上图中，我们有一个Flux发布者，它可以无错误地发出数据流。此外，它还显示了map()运算符对发出的数据的影响。该运算符将数据从圆形转换为正方形并返回转换后的数据。**订阅后，将发出转换后的数据，而不是原始数据**。

### 3.2 方法定义

Flux.map()运算符以[Function](https://www.baeldung.com/java-8-functional-interfaces#Functions)作为参数，并返回具有转换后元素的新Flux。

这是方法签名：

```java
public final <V> Flux<V> map(Function<? super T,? extends V> mapper)
```

在这种情况下，输入是来自Flux发布者的数据流。**mapper函数同步应用于Flux发出的每个元素**，输出是一个新的Flux，其中包含基于提供的mapper函数转换的元素。

### 3.3 示例代码

我们将一些数据转换为一个新序列，将每个值乘以10：

```java
Flux<Integer> numbersFlux = Flux.just(50, 51, 52, 53, 54, 55, 56, 57, 58, 59)
    .map(i -> i * 10)
    .onErrorResume(Flux::error);
```

然后，让我们断言发出的新数据序列等于预期的数字：

```java
StepVerifier.create(numbersFlux)
    .expectNext(500, 510, 520, 530, 540, 550, 560, 570, 580, 590)
    .verifyComplete();
```

map()运算符按照弹珠图和函数定义所述对数据进行操作，产生一个新的输出，每个值乘以10。

## 4. 理解doOnNext()运算符

Flux.doOnNext()运算符是一个生命周期钩子，它有助于查看已发出的数据流。**它类似于[Stream.peek()](https://www.baeldung.com/java-streams-peek-api#usage)，提供了一种在不改变原始数据流的情况下对发出的每个元素执行副作用的方法**。

### 4.1 图示

让我们通过一张图来了解Flux.doOnNext()方法的内部结构：

![](/assets/images/2025/reactor/fluxmapvsdoonnext02.png)

上图显示了Flux发出的数据流以及doOnNext()运算符对该数据的操作。

### 4.2 方法定义

我们来看一下doOnNext()运算符的方法定义：

```java
public final Flux<T> doOnNext(Consumer<? super T> onNext)
```

该方法接收[Consumer](https://www.baeldung.com/java-8-functional-interfaces#Consumers)作为参数，**Consumer是一个表示副作用操作的函数接口**，它消费输入但不产生任何输出，因此适合执行副作用操作。

### 4.3 示例代码

让我们应用doOnNext()运算符在订阅时将数据流中的元素记录到控制台：

```java
Flux<Integer> numberFlux = Flux.just(1, 2, 3, 4, 5)
    .doOnNext(number -> {
        LOGGER.info(String.valueOf(number));
    })
    .onErrorResume(Flux::error);
```

在上面的代码中，doOnNext()运算符记录Flux发出的每个数字，而不修改实际的数字。

## 5. 同时使用两个运算符

由于Flux.map()和Flux.doOnNext()有不同的用途，**它们可以组合在一个响应管道中来实现数据转换和副作用**。

让我们通过将元素记录到控制台并将原始数据转换为新数据来查看发出的数据流的元素：

```java
Flux numbersFlux = Flux.just(10, 11, 12, 13, 14)
    .doOnNext(number -> {
        LOGGER.info("Number: " + number);
    })
    .map(i -> i * 5)
    .doOnNext(number -> {
        LOGGER.info("Transformed Number: " + number);
    })
    .onErrorResume(Flux::error);
```

在上面的代码中，我们首先使用doOnNext()运算符来记录Flux发出的每个原始数字。接下来，我们应用map()运算符通过将每个数字乘以5来转换数字。然后，我们使用另一个doOnNext()运算符来记录转换后的数字。

最后，让我们断言发出的数据是预期的数据：

```java
StepVerifier.create(numbersFlux)
    .expectNext(50, 55, 60, 65, 70)
    .verifyComplete();
```

这种组合使用有助于我们转换数据流，同时还可以通过日志记录提供原始元素和转换后元素的可见性。

## 6. 主要区别

我们知道，这两个运算符作用于发出的数据。但是，Flux.map()运算符是一个转换运算符，它通过将提供的函数应用于每个元素来改变原始发出的数据流。**当我们想要对流的元素执行计算、数据转换或操作时，此运算符很有用**。

另一方面，Flux.doOnNext()运算符是一个生命周期钩子，它允许我们检查并对每个发出的元素执行操作。它不能修改数据。**此运算符在日志记录、调试等情况下很有用**。

## 7. 总结

在本文中，我们详细了解了Project Reactor库中的Flux.map()和Flux.doOnNext()运算符。我们通过研究弹珠图、类型定义和实际示例深入介绍了它们的内部工作原理。

这两个运算符适用于不同的用例，并且可以一起使用来构建强大而健壮的响应式系统。