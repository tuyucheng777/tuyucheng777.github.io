---
layout: post
title:  Resilience4j指南
category: libraries
copyright: libraries
excerpt: Resilience4j
---

## 1. 概述

在本教程中，我们将讨论[Resilience4j](http://resilience4j.github.io/resilience4j/)库。

**该库通过管理远程通信的容错能力来帮助实现弹性系统**。

该库受到[Hystrix](https://www.baeldung.com/introduction-to-hystrix)的启发，但提供了更方便的API和许多其他功能，如Rate Limiter(阻止过于频繁的请求)、Bulkhead(避免过多的并发请求)等。

## 2. Maven设置

首先，我们需要将目标模块添加到我们的pom.xml中(例如，这里我们添加了断路器)：

```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-circuitbreaker</artifactId>
    <version>2.1.0</version>
</dependency>
```

这里我们使用resilience4j-circuitbreaker模块，所有模块及其最新版本都可以在[Maven Central](https://mvnrepository.com/artifact/io.github.resilience4j)上找到。

在接下来的部分中，我们将介绍该库中最常用的模块。

## 3. 断路器

请注意，对于此模块，我们需要上面显示的resilience4j-circuitbreaker依赖。

当远程服务发生故障时，[断路器模式](https://martinfowler.com/bliki/CircuitBreaker.html)可以帮助我们防止发生一系列故障。

**经过多次失败尝试后，我们可以认为该服务不可用/过载，并急切地拒绝所有后续请求**。这样，我们可以为可能失败的调用节省系统资源。

让我们看看如何使用Resilience4j实现这一点。

首先，我们需要定义要使用的设置，最简单的方法是使用默认设置：

```java
CircuitBreakerRegistry circuitBreakerRegistry = CircuitBreakerRegistry.ofDefaults();
```

也可以使用自定义参数：

```java
CircuitBreakerConfig config = CircuitBreakerConfig.custom()
    .failureRateThreshold(20)
    .withSlidingWindow(5)
    .build();
```

在这里，我们将速率阈值设置为20%，并将最低调用尝试次数设置为5次。

然后，我们创建一个CircuitBreaker对象并通过它调用远程服务：

```java
interface RemoteService {
    int process(int i);
}

CircuitBreakerRegistry registry = CircuitBreakerRegistry.of(config);
CircuitBreaker circuitBreaker = registry.circuitBreaker("my");
Function<Integer, Integer> decorated = CircuitBreaker
    .decorateFunction(circuitBreaker, service::process);
```

最后，让我们通过JUnit测试看看它是如何工作的。

我们将尝试调用该服务10次，我们应该能够验证该调用至少尝试了5次，然后在20%的调用失败时立即停止：

```java
when(service.process(any(Integer.class))).thenThrow(new RuntimeException());

for (int i = 0; i < 10; i++) {
    try {
        decorated.apply(i);
    } catch (Exception ignore) {}
}

verify(service, times(5)).process(any(Integer.class));
```

### 3.1 断路器的状态和设置

CircuitBreaker可以处于以下三种状态之一：

- CLOSED：一切正常，无短路
- OPEN：远程服务器已关闭，对它的所有请求均被短路
- HALF_OPEN：自进入OPEN状态以来已过了配置的时间，并且CircuitBreaker允许请求检查远程服务是否重新上线

我们可以配置以下设置：

- 故障率阈值，超过该阈值，断路器将打开并开始短路调用
- 等待时长，定义断路器在切换到半开状态之前应保持打开状态的时间
- CircuitBreaker半开或半闭时环形缓冲区的大小
- 自定义CircuitBreakerEventListener处理CircuitBreaker事件
- 自定义谓词，用于评估异常是否应计入失败，从而增加失败率

## 4. 速率限制器

与上一节类似，此功能需要[resilience4j-ratelimiter](https://mvnrepository.com/search?q=resilience4j-ratelimiter)依赖。

顾名思义，**此功能允许限制对某些服务的访问**。它的API与CircuitBreaker非常相似-有Registry、Config和Limiter类。

以下是一个示例：

```java
RateLimiterConfig config = RateLimiterConfig.custom().limitForPeriod(2).build();
RateLimiterRegistry registry = RateLimiterRegistry.of(config);
RateLimiter rateLimiter = registry.rateLimiter("my");
Function<Integer, Integer> decorated = RateLimiter.decorateFunction(rateLimiter, service::process);
```

现在，如果需要的话，对装饰服务块的所有调用都符合速率限制器配置。

我们可以配置如下参数：

- 限制刷新周期
- 刷新周期的权限限制
- 默认等待权限时长

## 5. 隔板

这里，我们首先需要[resilience4j-bulkhead](https://mvnrepository.com/search?q=resilience4j-bulkhead)依赖。

**可以限制对特定服务的并发调用数量**。

让我们看一个使用Bulkhead API配置最大并发调用数量的示例：

```java
BulkheadConfig config = BulkheadConfig.custom().maxConcurrentCalls(1).build();
BulkheadRegistry registry = BulkheadRegistry.of(config);
Bulkhead bulkhead = registry.bulkhead("my");
Function<Integer, Integer> decorated = Bulkhead.decorateFunction(bulkhead, service::process);
```

为了测试此配置，我们将调用Mock服务方法。

然后，我们确保Bulkhead不允许任何其他调用：

```java
CountDownLatch latch = new CountDownLatch(1);
when(service.process(anyInt())).thenAnswer(invocation -> {
    latch.countDown();
    Thread.currentThread().join();
    return null;
});

ForkJoinTask<?> task = ForkJoinPool.commonPool().submit(() -> {
    try {
        decorated.apply(1);
    } finally {
        bulkhead.onComplete();
    }
});
latch.await();
assertThat(bulkhead.tryAcquirePermission()).isFalse();
```

我们可以配置以下设置：

- 隔板允许的最大并行执行数量
- 线程尝试进入饱和隔板时将等待的最长时间

## 6. 重试

对于此功能，我们需要将[resilience4j-retry](https://mvnrepository.com/search?q=resilience4j-retry)库添加到项目中。

我们可以**使用Retry API自动重试失败的调用**：

```java
RetryConfig config = RetryConfig.custom().maxAttempts(2).build();
RetryRegistry registry = RetryRegistry.of(config);
Retry retry = registry.retry("my");
Function<Integer, Void> decorated = Retry.decorateFunction(retry, (Integer s) -> {
        service.process(s);
        return null;
    });
```

现在让我们模拟远程服务调用期间抛出异常的情况，并确保库自动重试失败的调用：

```java
when(service.process(anyInt())).thenThrow(new RuntimeException());
try {
    decorated.apply(1);
    fail("Expected an exception to be thrown if all retries failed");
} catch (Exception e) {
    verify(service, times(2)).process(any(Integer.class));
}
```

我们还可以配置以下内容：

- 最大重试次数
- 重试前的等待时长
- 用于修改故障后的等待间隔的自定义函数
- 自定义谓词，用于评估异常是否应导致重试调用

## 7. 缓存

缓存模块需要[resilience4j-cache](https://mvnrepository.com/search?q=resilience4j-cache)依赖。

初始化看起来与其他模块略有不同：

```java
javax.cache.Cache cache = ...; // Use appropriate cache here
Cache<Integer, Integer> cacheContext = Cache.of(cache);
Function<Integer, Integer> decorated = Cache.decorateSupplier(cacheContext, () -> service.process(1));
```

这里的缓存是由使用的[JSR-107 Cache](https://www.baeldung.com/jcache)实现完成的，并且Resilience4j提供了一种应用它的方法。

请注意，没有用于装饰器函数的API(例如Cache.decorateFunction(Function))，API仅支持Supplier和Callable类型。

## 8. TimeLimiter

对于这个模块，我们必须添加[resilience4j-timelimiter](https://mvnrepository.com/search?q=resilience4j-timelimiter)依赖。

**可以使用TimeLimiter来限制调用远程服务所花费的时间**。

为了演示，让我们设置一个TimeLimiter，并将超时时间配置为1毫秒：

```java
long ttl = 1;
TimeLimiterConfig config = TimeLimiterConfig.custom().timeoutDuration(Duration.ofMillis(ttl)).build();
TimeLimiter timeLimiter = TimeLimiter.of(config);
```

接下来，让我们验证Resilience4j是否以预期的超时时间调用Future.get()：

```java
Future futureMock = mock(Future.class);
Callable restrictedCall = TimeLimiter.decorateFutureSupplier(timeLimiter, () -> futureMock);
restrictedCall.call();

verify(futureMock).get(ttl, TimeUnit.MILLISECONDS);
```

我们还可以将其与CircuitBreaker结合使用：

```java
Callable chainedCallable = CircuitBreaker.decorateCallable(circuitBreaker, restrictedCall);
```

## 9. 其他模块

Resilience4j还提供了许多额外模块，以便与其他流行框架和库的集成。

一些较为知名的集成包括：

- Spring Boot：resilience4j-spring-boot模块
- Ratpack：resilience4j-ratpack模块
- Retrofit：resilience4j-retrofit模块
- Vertx：resilience4j-vertx模块
- Dropwizard：resilience4j-metrics模块
- Prometheus：resilience4j-prometheus模块

## 10. 总结

在本文中，我们介绍了Resilience4j库的不同方面，并学习了如何使用它来解决服务器间通信中的各种容错问题。