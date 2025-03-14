---
layout: post
title:  如何有效地对CompletableFuture进行单元测试
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 简介

[CompletableFuture](https://www.baeldung.com/java-completablefuture)是Java中用于异步编程的强大工具，**它提供了一种将异步任务链接在一起并处理其结果的便捷方法**，通常用于需要执行异步操作并且其结果需要在稍后阶段使用或处理的情况。

**然而，由于CompletableFuture具有异步特性，因此对其进行单元测试可能具有挑战性**。传统的测试方法依赖于顺序执行，通常无法捕捉异步代码的细微差别。在本教程中，我们将讨论如何使用两种不同的方法有效地对CompletableFuture进行单元测试：黑盒测试和基于状态的测试。

## 2. 测试异步代码的挑战

**异步代码由于其非阻塞和并发执行而带来了挑战**，给传统的测试方法带来了困难，这些挑战包括：

- 时间问题：异步操作会将时间依赖性引入代码中，使得控制执行流程和验证代码在特定时间点的行为变得困难，**依赖顺序执行的传统测试方法可能不适合异步代码**。
- 异常处理：异步操作可能会引发异常，因此确保代码妥善处理这些异常且不会默默失败至关重要，单元测试应涵盖各种场景以验证异常处理机制。
- 竞争条件：异步代码可能导致竞争条件，其中多个线程或进程试图同时访问或修改共享数据，从而可能导致意外结果。
- 测试覆盖率：由于交互的复杂性和不确定结果的可能性，实现异步代码的全面测试覆盖率可能具有挑战性。

## 3. 黑盒测试

**[黑盒测试](https://www.baeldung.com/cs/testing-white-box-vs-black-box)专注于测试代码的外部行为，而无需了解其内部实现，这种方法适合从用户的角度验证异步代码行为**，测试人员只需知道代码的输入和预期输出。

当使用黑盒测试来测试CompletableFuture时，我们优先考虑以下方面：

- 成功完成：验证CompletableFuture是否成功完成，返回预期结果。
- 异常处理：验证CompletableFuture是否可以正常处理异常，防止出现无声故障。
- 超时：确保CompletableFuture在遇到超时时能够按预期运行。

我们可以使用[Mockito](https://www.baeldung.com/mockito-series)之类的Mock框架来模拟测试中的CompletableFuture的依赖，这将使我们能够隔离CompletableFuture并在受控环境中测试其行为。

### 3.1 被测系统

我们将测试一个名为processAsync()的方法，该方法封装了异步数据检索和组合过程，此方法接收Microservice对象列表作为输入并返回CompletableFuture<String\>，每个Microservice对象代表一个能够执行异步检索操作的微服务。

processAsync()使用两个辅助方法fetchDataAsync()和CombineResults()来处理异步数据检索和组合任务：

```java
CompletableFuture<String> processAsync(List<Microservice> microservices) {
    List<CompletableFuture<String>> dataFetchFutures = fetchDataAsync(microservices);
    return combineResults(dataFetchFutures);
}
```

fetchDataAsync()方法遍历Microservice列表，为每个Microservice调用retrieveAsync()，并返回CompletableFuture<String\>列表：

```java
private List<CompletableFuture<String>> fetchDataAsync(List<Microservice> microservices) {
    return microservices.stream()
        .map(client -> client.retrieveAsync(""))
        .collect(Collectors.toList());
}
```

combineResults()方法使用CompletableFuture.allOf()等待列表中的所有Future完成；完成后，它会映射Future、拼接结果并返回单个字符串：

```java
private CompletableFuture<String> combineResults(List<CompletableFuture<String>> dataFetchFutures) {
    return CompletableFuture.allOf(dataFetchFutures.toArray(new CompletableFuture[0]))
        .thenApply(v -> dataFetchFutures.stream()
            .map(future -> future.exceptionally(ex -> {
                throw new CompletionException(ex);
            })
            .join())
        .collect(Collectors.joining()));
}
```

### 3.2 测试用例：验证数据检索和组合是否成功

此测试用例验证processAsync()方法是否正确地从多个微服务中检索数据并将结果组合成一个字符串：

```java
@Test
public void givenAsyncTask_whenProcessingAsyncSucceed_thenReturnSuccess() throws ExecutionException, InterruptedException {
    Microservice mockMicroserviceA = mock(Microservice.class);
    Microservice mockMicroserviceB = mock(Microservice.class);

    when(mockMicroserviceA.retrieveAsync(any())).thenReturn(CompletableFuture.completedFuture("Hello"));
    when(mockMicroserviceB.retrieveAsync(any())).thenReturn(CompletableFuture.completedFuture("World"));

    CompletableFuture<String> resultFuture = processAsync(List.of(mockMicroserviceA, mockMicroserviceB));

    String result = resultFuture.get();
    assertEquals("HelloWorld", result);
}
```

### 3.3 测试用例：验证微服务抛出异常时的异常处理

此测试用例验证当其中一个微服务抛出异常时，processAsync()方法是否会抛出ExecutionException；它还断言异常消息与微服务抛出的异常相同：

```java
@Test
public void givenAsyncTask_whenProcessingAsyncWithException_thenReturnException() throws ExecutionException, InterruptedException {
    Microservice mockMicroserviceA = mock(Microservice.class);
    Microservice mockMicroserviceB = mock(Microservice.class);

    when(mockMicroserviceA.retrieveAsync(any())).thenReturn(CompletableFuture.completedFuture("Hello"));
    when(mockMicroserviceB.retrieveAsync(any()))
        .thenReturn(CompletableFuture.failedFuture(new RuntimeException("Simulated Exception")));
    CompletableFuture<String> resultFuture = processAsync(List.of(mockMicroserviceA, mockMicroserviceB));

    ExecutionException exception = assertThrows(ExecutionException.class, resultFuture::get);
    assertEquals("Simulated Exception", exception.getCause().getMessage());
}
```

### 3.4 测试用例：验证组合结果超时的超时处理

此测试用例尝试在指定的300毫秒超时时间内从processAsync()方法检索组合结果，它断言当超过超时时间时会抛出TimeoutException：

```java
@Test
public void givenAsyncTask_whenProcessingAsyncWithTimeout_thenHandleTimeoutException() throws ExecutionException, InterruptedException {
    Microservice mockMicroserviceA = mock(Microservice.class);
    Microservice mockMicroserviceB = mock(Microservice.class);

    Executor delayedExecutor = CompletableFuture.delayedExecutor(200, TimeUnit.MILLISECONDS);
    when(mockMicroserviceA.retrieveAsync(any()))
        .thenReturn(CompletableFuture.supplyAsync(() -> "Hello", delayedExecutor));
    Executor delayedExecutor2 = CompletableFuture.delayedExecutor(500, TimeUnit.MILLISECONDS);
    when(mockMicroserviceB.retrieveAsync(any()))
        .thenReturn(CompletableFuture.supplyAsync(() -> "World", delayedExecutor2));
    CompletableFuture<String> resultFuture = processAsync(List.of(mockMicroserviceA, mockMicroserviceB));

    assertThrows(TimeoutException.class, () -> resultFuture.get(300, TimeUnit.MILLISECONDS));
}
```

上述代码使用CompletableFuture.delayedExecutor()创建执行器，分别将retrieveAsync()调用的完成延迟200和500毫秒，这模拟了微服务造成的延迟，并允许测试验证processAsync()方法是否正确处理超时。

## 4. 基于状态的测试

**基于状态的测试侧重于验证代码执行时的状态转换**，这种方法对于测试异步代码特别有用，因为它允许测试人员跟踪代码在不同状态下的进度并确保其正确转换。

例如，我们可以验证当异步任务成功完成时，CompletableFuture是否转换为完成状态。或者，当发生异常或任务因中断而被取消时，是否转换为失败状态。

### 4.1 测试用例：成功完成后验证状态

此测试用例验证当CompletableFuture实例的所有组成部分都已成功完成后，CompletableFuture实例是否会转换为完成状态：

```java
@Test
public void givenCompletableFuture_whenCompleted_thenStateIsDone() {
    Executor delayedExecutor = CompletableFuture.delayedExecutor(200, TimeUnit.MILLISECONDS);
    CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> "Hello", delayedExecutor);
    CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> " World");
    CompletableFuture<String> cf3 = CompletableFuture.supplyAsync(() -> "!");
    CompletableFuture<String>[] cfs = new CompletableFuture[] { cf1, cf2, cf3 };

    CompletableFuture<Void> allCf = CompletableFuture.allOf(cfs);

    assertFalse(allCf.isDone());
    allCf.join();
    String result = Arrays.stream(cfs)
        .map(CompletableFuture::join)
        .collect(Collectors.joining());

    assertFalse(allCf.isCancelled());
    assertTrue(allCf.isDone());
    assertFalse(allCf.isCompletedExceptionally());
}
```

### 4.2 测试用例：验证异常完成后的状态

此测试用例验证当其中一个组成CompletableFuture实例cf2异常完成时，allCf CompletableFuture是否转换到异常状态：

```java
@Test
public void givenCompletableFuture_whenCompletedWithException_thenStateIsCompletedExceptionally() throws ExecutionException, InterruptedException {
    Executor delayedExecutor = CompletableFuture.delayedExecutor(200, TimeUnit.MILLISECONDS);
    CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> "Hello", delayedExecutor);
    CompletableFuture<String> cf2 = CompletableFuture.failedFuture(new RuntimeException("Simulated Exception"));
    CompletableFuture<String> cf3 = CompletableFuture.supplyAsync(() -> "!");
    CompletableFuture<String>[] cfs = new CompletableFuture[] { cf1, cf2, cf3 };

    CompletableFuture<Void> allCf = CompletableFuture.allOf(cfs);

    assertFalse(allCf.isDone());
    assertFalse(allCf.isCompletedExceptionally());

    assertThrows(CompletionException.class, allCf::join);

    assertTrue(allCf.isCompletedExceptionally());
    assertTrue(allCf.isDone());
    assertFalse(allCf.isCancelled());
}
```

### 4.3 测试用例：验证任务取消后的状态

此测试用例验证当使用cancel(true)方法取消allCf CompletableFuture时，它会转换到已取消状态：

```java
@Test
public void givenCompletableFuture_whenCancelled_thenStateIsCancelled() throws ExecutionException, InterruptedException {
    Executor delayedExecutor = CompletableFuture.delayedExecutor(200, TimeUnit.MILLISECONDS);
    CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> "Hello", delayedExecutor);
    CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> " World");
    CompletableFuture<String> cf3 = CompletableFuture.supplyAsync(() -> "!");
    CompletableFuture<String>[] cfs = new CompletableFuture[] { cf1, cf2, cf3 };

    CompletableFuture<Void> allCf = CompletableFuture.allOf(cfs);
    assertFalse(allCf.isDone());
    assertFalse(allCf.isCompletedExceptionally());

    allCf.cancel(true);

    assertTrue(allCf.isCancelled());
    assertTrue(allCf.isDone());
}
```

## 5. 总结

总之，由于CompletableFuture具有异步特性，因此对其进行单元测试可能具有挑战性。但是，这是编写可靠且可维护的异步代码的重要部分。通过使用黑盒和基于状态的测试方法，我们可以评估CompletableFuture代码在各种条件下的行为，确保它按预期运行并妥善处理潜在异常。