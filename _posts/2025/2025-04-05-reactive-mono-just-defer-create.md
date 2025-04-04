---
layout: post
title:  响应式编程中的Mono just()、defer()和create()
category: reactor
copyright: reactor
excerpt: Reactor
---

## 1. 简介

[Mono](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/Mono.html)是[响应式编程](https://www.baeldung.com/cs/reactive-programming)(尤其是在Project Reactor中)的核心概念，它表示最多发出一个元素然后成功或错误完成的流。Mono用于返回单个结果或根本不返回结果的异步操作。

Mono特别适合于表示异步计算的结果，例如数据库查询、HTTP请求或任何返回单个值或完成而不发出任何值的操作。

在本文中，我们将探讨创建Mono的三种常见方法之间的区别：just()、defer()和create()。

## 2. Mono.just()

Mono.just()方法是创建Mono的最简单方法，它获取现有值并将其包装在Mono中，从而有效地使其立即可供订阅者使用。当数据可用且不需要复杂的计算或延迟执行时，我们会使用此方法。

**当我们使用Mono.just()时，值在创建Mono时被评估和存储，而不是等到它被订阅**。一旦创建，值就无法更改，即使在订阅之前修改了底层数据源。这确保了订阅期间发出的值始终是Mono创建时可用的值。

让我们通过一个例子看看这个行为是如何运作的：

```java
@Test
void whenUsingMonoJust_thenValueIsCreatedEagerly() {
    String[] value = {"Hello"};
    Mono<String> mono = Mono.just(value[0]);

    value[0] = "world";

    mono.subscribe(actualValue -> assertEquals("Hello", actualValue));
}
```

在这个例子中，我们可以看到Mono.just()在创建Mono时立即创建了值。Mono使用数组中的值“Hello”进行初始化，即使在订阅Mono之前数组元素被修改为“world”，发出的值仍为“Hello”。

**让我们探索Mono.just()的一些常见用例**：

- 当我们有一个已知的、静态的值可以发出时
- 它非常适合不涉及任何计算或副作用的简单用例

## 3. Mono.defer()

**[Mono.defer()](https://www.baeldung.com/java-mono-defer)启用惰性执行，这意味着Mono直到被订阅时才会被创建**。当我们想要避免不必要的资源分配或计算，直到真正需要Mono时，这尤其有用。

让我们检查一个例子来验证Mono是在订阅时创建的，而不是在最初定义时创建的：

```java
@Test
void whenUsingMonoDefer_thenValueIsCreatedLazily() {
    String[] value = {"Hello"};
    Mono<String> mono = Mono.defer(() -> Mono.just(value[0]));

    value[0] = "World";

    mono.subscribe(actualValue -> assertEquals("World", actualValue));
}
```

在此示例中，我们可以看到数组中的值在创建Mono后发生了更改。由于Mono.defer()以惰性方式创建Mono，因此它不会在创建时捕获值，而是等到Mono被订阅。因此，当我们订阅时，我们收到更新后的值“World”，而不是原始值“Hello”，这表明Mono.defer()将求值推迟到订阅时。

**Mono.defer()在需要为每个订阅者创建一个新的、不同的Mono实例的情况下特别有用**。这使我们能够生成一个单独的实例，以反映订阅时的当前状态或数据。当我们需要根据订阅之间可能发生变化的条件动态得出值时，这种方法至关重要。

我们来看一个基于方法参数创建延迟Mono的场景：

```java
public Mono<String> getGreetingMono(String name) {
    return Mono.defer(() -> {
        String greeting = "Hello, " + name;
        return Mono.just(greeting);
    });
}
```

这里，Mono.defer()方法使用传递给getGreetingMono()方法的名称将Mono的创建推迟到订阅：

```java
@Test
void givenNameIsAlice_whenMonoSubscribed_thenShouldReturnGreetingForAlice() {
    Mono<String> mono = generator.getGreetingMono("Alice");

    StepVerifier.create(mono)
            .expectNext("Hello, Alice")
            .verifyComplete();
}

@Test
void givenNameIsBob_whenMonoSubscribed_thenShouldReturnGreetingForBob() {
    Mono<String> mono = generator.getGreetingMono("Bob");

    StepVerifier.create(mono)
            .expectNext("Hello, Bob")
            .verifyComplete();
}
```

当使用参数“Alice”调用时，该方法生成一个Mono，发出“Hello, Alice”。类似地，当使用“Bob”调用时，它生成一个Mono，发出“Hello, Bob”。

**让我们看看Mono.defer()的一些常见用例**：

- 当我们的Mono创建涉及数据库查询或网络调用等昂贵的操作时，使用defer()，我们可以通过以下方式阻塞这些操作的执行，除非确实需要此结果
- 它对于动态生成值或计算依赖于外部因素或用户输入时很有用

## 4. Mono.create()

Mono.create()是最灵活、最强大的方法，它让我们可以完全控制Mono的发射过程。它允许我们根据自定义逻辑以编程方式生成值、信号和错误。

**它的主要功能是提供[MonoSink](https://projectreactor.io/docs/core/release/api/reactor/core/publisher/MonoSink.html)，我们可以使用Sink通过sink.success()发出一个值。如果出现错误，我们使用sink.error()。要发送不带值的完成信号，我们调用不带参数的sink.success()**。

让我们探索一个执行外部操作(例如查询远程服务)的示例，在这种情况下，我们将使用Mono.create()来封装处理响应的逻辑。根据服务调用是成功还是导致错误，我们将向订阅者发出有效值或错误信号：

```java
public Mono<String> performOperation(boolean success) {
    return Mono.create(sink -> {
        if (success) {
            sink.success("Operation Success");
        } else {
            sink.error(new RuntimeException("Operation Failed"));
        }
    });
}
```

让我们验证一下当操作成功时这个Mono是否会发出成功消息：

```java
@Test
void givenSuccessScenario_whenMonoSubscribed_thenShouldReturnSuccessValue() {
    Mono<String> mono = generator.performOperation(true);

    StepVerifier.create(mono)
            .expectNext("Operation Success")
            .verifyComplete();
}
```

当操作失败时，Mono将发出错误消息“Operation Failed”：

```java
@Test
void givenErrorScenario_whenMonoSubscribed_thenShouldReturnError() {
    Mono<String> mono = generator.performOperation(false);

    StepVerifier.create(mono)
            .expectErrorMatches(throwable -> throwable instanceof RuntimeException
                    && throwable.getMessage()
                    .equals("Operation Failed"))
            .verify();
}
```

**让我们探索一下Mono.create()的一些常见场景**：

- 当我们需要实现用于发出值、处理错误或管理背压的复杂逻辑时，此方法提供了必要的灵活性
- 它非常适合与遗留系统集成或执行需要对发射和完成进行细粒度控制的复杂异步任务

## 5. 主要区别

现在我们已经分别探讨了这三种方法，让我们总结一下它们的主要区别：

|  特征   | Mono.just()  |  Mono.defer()   | Mono.create() |
|:-----:|:------------:|:---------------:|:-------------:|
| **执行时间**  |   急切(创建时)    |     惰性(需订阅)     |   惰性(手动发射)    |
|  **值状态**  |   静态/预定义值    |    每个订阅的动态值     |    人工生成的值     |
|  **用例**   | 当数据可用且不会改变时  |   当需要按需生成数据时    | 与复杂逻辑或外部源集成时  |
| **错误处理**  |   没有错误处理功能   | 可以处理Mono创建期间的错误 |  对成功和错误的明确控制  |
|  **表现**   | 对于静态、已知的值很有效 |   适用于动态或昂贵的操作   |   适合复杂或异步代码   |

## 6. 总结

在本教程中，我们研究了创建Mono的三种方法，包括每种方法最适合的场景。

理解Mono.just()、Mono.defer()和Mono.create()之间的区别是有效使用Java中的响应式编程的关键。通过选择正确的方法，我们可以使我们的响应式代码更高效、更易于维护，并适合我们的特定用例。