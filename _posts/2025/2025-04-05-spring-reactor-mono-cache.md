---
layout: post
title:  使用Reactor Mono.cache()进行缓存
category: reactor
copyright: reactor
excerpt: Reactor
---

## 1. 概述

优化代码性能是编程的关键部分，尤其是在处理昂贵的操作或数据检索过程时。提高性能的一种有效方法是缓存。

**[Project Reactor](https://www.baeldung.com/reactor-core)库提供了cache()方法来缓存昂贵的操作或几乎不变的数据，以避免重复操作并提高性能**。

在本教程中，我们将探索一种缓存形式-内存化，并演示如何使用Project Reactor库中的Mono.cache()将HTTP GET请求的结果缓存到[JSONPlaceholder](https://jsonplaceholder.typicode.com/) API。此外，我们将通过图了解Mono.cache()方法的内部原理。

## 2. 理解内存化

**内存化是一种缓存形式，用于存储昂贵的函数调用的输出。然后，当再次发生相同的函数调用时，它会返回缓存的结果**。

在涉及[递归函数](https://www.baeldung.com/java-recursion)或计算的情况下，它很有用，这些函数或计算对于给定的输入总是产生相同的输出。

让我们看一个使用斐波那契数列演示Java中的内存化的示例。首先，让我们创建一个[Map](https://www.baeldung.com/java-hashmap)对象来存储缓存结果：

```java
private static final Map<Integer, Long> cache = new HashMap<>();
```

接下来，我们定义一个计算斐波那契数列的方法：

```java
long fibonacci(int n) {
    if (n <= 1) {
        return n;
    }

    if (cache.containsKey(n)) {
        return cache.get(n);
    }

    long result = fibonacci(n - 1) + fibonacci(n - 2);
    logger.info("First occurrence of " + n);
    cache.put(n, result);

    return result;
}
```

在上面的代码中，我们在进一步计算之前检查整数n是否已存储在Map对象中。如果它已存储在Map对象中，我们将返回缓存的值。否则，我们递归计算结果并将其存储在Map对象中以供将来使用。

该方法通过避免冗余计算显著提高了斐波那契计算的性能。

让我们为该方法编写一个单元测试：

```java
@Test
void givenFibonacciNumber_whenFirstOccurenceIsCache_thenReturnCacheResultOnSecondCall() {
    assertEquals(5, FibonacciMemoization.fibonacci(5));
    assertEquals(2, FibonacciMemoization.fibonacci(3));
    assertEquals(55, FibonacciMemoization.fibonacci(10));
    assertEquals(21, FibonacciMemoization.fibonacci(8));
}
```

在上面的测试中，我们调用fibonacci()来计算序列。

## 3. 使用图描述Mono.cache()

Mono.cache()运算符有助于缓存Mono发布者的结果并返回缓存值以供后续订阅。

图有助于理解响应式类的内部细节及其工作原理，以下弹珠图说明了cache()运算符的行为：

![](/assets/images/2025/reactor/springreactormonocache01.png)

**在上图中，对[Mono](https://www.baeldung.com/java-reactor-flux-vs-mono#what-is-mono)发布者的第一个订阅发出数据并缓存。后续订阅将检索缓存的数据，而不会触发新的计算或数据提取**。

## 4. 示例设置

为了演示Mono.cache()的用法，我们将[react-core](https://mvnrepository.com/artifact/io.projectreactor/reactor-core)添加到pom.xml中：

```xml
<dependency>
    <groupId>io.projectreactor</groupId>
    <artifactId>reactor-core</artifactId>
    <version>3.6.5</version>
</dependency>
```

该库提供了Mono、Flux等运算符来实现Java中的响应式编程。

另外，让我们将[spring-boot-starter-webflux](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-webflux)添加到pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
    <version>3.2.5</version>
</dependency>
```

上述依赖提供了WebClient类来调用API。

另外，让我们看一下当我们收到对https://jsonplaceholder.typicode.com/users/2的GET请求时的示例响应：

```json
{
    "id": 2,
    "name": "Ervin Howell",
    "username": "Antonette"
    // ...
}
```

接下来，让我们创建一个名为User的[POJO类](https://www.baeldung.com/java-pojo-class)来反序列化来自GET请求的JSON响应：

```java
public class User {
    private int id;
    private String name;

    // standard constructor, getter and setter
}
```

此外，让我们创建一个[WebClient](https://www.baeldung.com/spring-5-webclient)对象并设置API的基本URL：

```java
WebClient client = WebClient.create("https://jsonplaceholder.typicode.com/users");
```

这将作为使用cache()方法缓存的HTTP响应的基本URL。

最后，让我们创建一个[AtomicInteger](https://www.baeldung.com/java-atomic-variables)对象：

```java
AtomicInteger counter = new AtomicInteger(0);
```

上述对象有助于跟踪我们向API发出GET请求的次数。

## 5. 不使用内存法获取数据

让我们首先定义一个从WebClient对象获取用户的方法：

```java
Mono<User> retrieveOneUser(int id) {
    return client.get()
        .uri("/{id}", id)
        .retrieve()
        .bodyToMono(User.class)
        .doOnSubscribe(i -> counter.incrementAndGet())
        .onErrorResume(Mono::error);
}
```

在上面的代码中，我们检索具有特定ID的用户，并将响应主体映射到User对象。此外，我们在每个订阅时增加counter。

这是一个测试用例，演示了如何在没有缓存的情况下获取用户：

```java
@Test
void givenRetrievedUser_whenTheCallToRemoteIsNotCache_thenReturnInvocationCountAndCompareResult() {
    MemoizationWithMonoCache memoizationWithMonoCache = new MemoizationWithMonoCache();

    Mono<User> retrieveOneUser = MemoizationWithMonoCache.retrieveOneUser(1);
    AtomicReference<User> firstUser = new AtomicReference<>();
    AtomicReference<User> secondUser = new AtomicReference<>();

    Disposable firstUserCall = retrieveOneUser.map(user -> {
                firstUser.set(user);
                return user.getName();
            })
            .subscribe();

    Disposable secondUserCall = retrieveOneUser.map(user -> {
                secondUser.set(user);
                return user.getName();
            })
            .subscribe();

    assertEquals(2, memoizationWithMonoCache.getCounter());
    assertEquals(firstUser.get(), secondUser.get());
}
```

**这里，我们订阅了retrieveOneUser Mono两次，每次订阅都会向WebClient对象触发单独的GET请求**，我们断言计数器增加两次。

## 6. 使用内存法获取数据

现在，让我们修改前面的示例以利用Mono.cache()并缓存第一个GET请求的结果：

```java
@Test
void givenRetrievedUser_whenTheCallToRemoteIsCache_thenReturnInvocationCountAndCompareResult() {
    MemoizationWithMonoCache memoizationWithMonoCache = new MemoizationWithMonoCache();

    Mono<User> retrieveOneUser = MemoizationWithMonoCache.retrieveOneUser(1).cache();
    AtomicReference<User> firstUser = new AtomicReference<>();
    AtomicReference<User> secondUser = new AtomicReference<>();

    Disposable firstUserCall = retrieveOneUser.map(user -> {
                firstUser.set(user);
                return user.getName();
            })
            .subscribe();

    Disposable secondUserCall = retrieveOneUser.map(user -> {
                secondUser.set(user);
                return user.getName();
            })
            .subscribe();

    assertEquals(1, memoizationWithMonoCache.getCounter());
    assertEquals(firstUser.get(), secondUser.get());
}
```

与上一个示例的主要区别在于，我们在订阅之前对retrieveOneUser对象调用cache()运算符。**这会缓存第一个GET请求的结果，后续订阅将接收缓存的结果，而不是触发新的请求**。

在测试用例中，我们断言counter增加一次，因为第二个订阅使用了缓存值。

## 7. 设置缓存时长

默认情况下，Mono.Cache()会无限期地缓存结果。但是，在数据可能随着时间的推移而变得陈旧的情况下，设置缓存持续时间至关重要：

```java
// ... 
Mono<User> retrieveOneUser = memoizationWithMonoCache.retrieveOneUser(1)
    .cache(Duration.ofMinutes(5));
// ...
```

在上面的代码中，cache()方法接收[Duration](https://www.baeldung.com/java-period-duration#duration-class)的一个实例作为参数。缓存的值将在5分钟后过期，此后的任何后续订阅都将触发新的GET请求。

## 8. 总结

在本文中，我们使用斐波那契数列示例学习了内存化的关键概念及其在Java中的实现。然后，我们深入研究了Project Reactor库中Mono.cache()的用法，并演示了如何缓存HTTP GET请求的结果。

缓存是提高性能的一项强大技术，然而，必须考虑缓存失效策略，以确保不会无限期地提供过时的数据。