---
layout: post
title:  在Spring中设置异步重试机制
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

有时，我们要求代码执行异步，以获得更好的应用程序性能和响应能力。此外，我们可能希望在出现任何异常时自动重新调用代码，因为我们预计会遇到偶尔的故障，例如网络故障。

在本教程中，我们将学习在Spring应用程序中实现带有自动重试的异步执行。

我们将探讨Spring对异步和重试操作的支持。

## 2. Spring Boot中的示例应用

假设我们需要构建一个简单的微服务来调用下游服务来处理一些数据。

### 2.1 Maven依赖

首先，我们需要包含[spring-boot-starter-web](https://central.sonatype.com/artifact/org.springframework.boot/spring-boot-starter-web) Maven依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### 2.2 实施Spring服务

现在，我们将实现EventService类调用另一个服务的方法：

```java
public String processEvents(List<String> events) {
    downstreamService.publishEvents(events);
    return "Completed";
}
```

然后，让我们定义DownstreamService接口：

```java
public interface DownstreamService {
    boolean publishEvents(List<String> events);
}
```

## 3. 通过重试实现异步执行

为了实现带重试的异步执行，我们将使用Spring的实现。

我们需要为应用程序配置异步和重试支持。

### 3.1 添加Retry Maven依赖

让我们将[spring-retry](https://central.sonatype.com/artifact/org.springframework.retry/spring-retry)添加到Maven依赖项中：

```xml
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
    <version>2.0.4</version>
</dependency>
```

### 3.2 @EnableAsync和@EnableRetry配置

接下来**我们需要包含[@EnableAsync](https://www.baeldung.com/spring-async#enable-async-support)和[@EnableRetry](https://www.baeldung.com/spring-retry#enabling-spring-retry)注解**：

```java
@Configuration
@ComponentScan("cn.tuyucheng.taketoday.asyncwithretry")
@EnableRetry
@EnableAsync
public class AsyncConfig {
}
```

### 3.3 包含@Async和@Retryable注解

要异步执行方法，我们需要使用[@Async](https://www.baeldung.com/spring-async#the-async-annotation)注解。同样，我们将使用[@Retryable](https://www.baeldung.com/spring-retry#1-retryable-without-recovery)注解对该方法进行标注，以便重试执行。

我们在上面的EventService方法中配置上面的注解：

```java
@Async
@Retryable(retryFor = RuntimeException.class, maxAttempts = 4, backoff = @Backoff(delay = 100))
public Future<String> processEvents(List<String> events) {
    LOGGER.info("Processing asynchronously with Thread {}", Thread.currentThread().getName());
    downstreamService.publishEvents(events);
    CompletableFuture<String> future = new CompletableFuture<>();
    future.complete("Completed");
    LOGGER.info("Completed async method with Thread {}", Thread.currentThread().getName());
    return future;
}
```

在上面的代码中，我们在出现RuntimeException的情况下重试该方法，并将结果作为Future对象。

我们应该注意**我们应该使用[Future](https://www.baeldung.com/java-future#1-using-isdone-and-get-to-obtain-results)来包装来自任何异步方法的响应**。

我们应该注意，**@Async注解仅适用于公共方法**，不应该在同一个类中自调用。自调用方法将绕过Spring代理调用并在同一线程中运行。

## 4. 对@Async和@Retryable实施测试

让我们测试一下EventService方法，并通过几个测试用例验证其异步和重试行为。

首先，当DownstreamService调用没有错误时，我们将实现一个测试用例：

```java
@Test
void givenAsyncMethodHasNoRuntimeException_whenAsyncMethodIscalled_thenReturnSuccess_WithoutAnyRetry() throws Exception {
    LOGGER.info("Testing for async with retry execution with thread " + Thread.currentThread().getName());
    when(downstreamService.publishEvents(anyList())).thenReturn(true);
    Future<String> resultFuture = eventService.processEvents(List.of("test1"));
    while (!resultFuture.isDone() && !resultFuture.isCancelled()) {
        TimeUnit.MILLISECONDS.sleep(5);
    }
    assertTrue(resultFuture.isDone());
    assertEquals("Completed", resultFuture.get());
    verify(downstreamService, times(1)).publishEvents(anyList());
}
```

在上面的测试中，我们等待Future完成，然后断言结果。

然后，让我们运行上面的测试并验证测试日志：

```text
18:59:24.064 [main] INFO cn.tuyucheng.taketoday.asyncwithretry.EventServiceIntegrationTest - Testing for async with retry execution with thread main
18:59:24.078 [SimpleAsyncTaskExecutor-1] INFO cn.tuyucheng.taketoday.asyncwithretry.EventService - Processing asynchronously with Thread SimpleAsyncTaskExecutor-1
18:59:24.080 [SimpleAsyncTaskExecutor-1] INFO cn.tuyucheng.taketoday.asyncwithretry.EventService - Completed async method with Thread SimpleAsyncTaskExecutor-1
```

从上面的日志中，我们确认服务方法在单独的线程中运行。

接下来，我们将实现另一个测试用例，其中DownstreamService方法抛出RuntimeException：

```java
@Test
void givenAsyncMethodHasRuntimeException_whenAsyncMethodIsCalled_thenReturnFailure_With_MultipleRetries() throws InterruptedException {
    LOGGER.info("Testing for async with retry execution with thread " + Thread.currentThread().getName()); 
    when(downstreamService.publishEvents(anyList())).thenThrow(RuntimeException.class);
    Future<String> resultFuture = eventService.processEvents(List.of("test1"));
    while (!resultFuture.isDone() && !resultFuture.isCancelled()) {
        TimeUnit.MILLISECONDS.sleep(5);
    }
    assertTrue(resultFuture.isDone());
    assertThrows(ExecutionException.class, resultFuture::get);
    verify(downstreamService, times(4)).publishEvents(anyList());
}
```

最后，我们用输出日志来验证一下上面的测试用例：

```text
19:01:32.307 [main] INFO cn.tuyucheng.taketoday.asyncwithretry.EventServiceIntegrationTest - Testing for async with retry execution with thread main
19:01:32.318 [SimpleAsyncTaskExecutor-1] INFO cn.tuyucheng.taketoday.asyncwithretry.EventService - Processing asynchronously with Thread SimpleAsyncTaskExecutor-1
19:01:32.425 [SimpleAsyncTaskExecutor-1] INFO cn.tuyucheng.taketoday.asyncwithretry.EventService - Processing asynchronously with Thread SimpleAsyncTaskExecutor-1
.....
```

从上面的日志中，我们确认服务方法被异步重新执行了4次。

## 5. 总结

在本文中，我们学习了如何使用Spring中的重试机制实现异步方法。

我们在示例应用程序中实现了这一点，并尝试了一些测试来了解它如何处理不同的用例。我们已经了解了异步代码如何在自己的线程上运行并可以自动重试。