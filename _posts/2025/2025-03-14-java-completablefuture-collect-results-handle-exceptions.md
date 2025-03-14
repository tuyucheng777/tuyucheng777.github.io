---
layout: post
title:  如何使用CompletableFuture循环收集所有结果并处理异常
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

Java 8的[CompletableFuture](https://www.baeldung.com/java-completablefuture#CompletableFuture)非常适合处理异步计算，例如，Web客户端可以在进行服务器调用时使用CompletableFuture。入门和处理单个CompletableFuture响应很容易，**但是，目前还不清楚如何收集多个CompletableFuture执行的结果，同时处理异常**。

在本教程中，我们将开发一个返回CompletableFuture的简单模拟微服务客户端，并了解如何多次调用它来生成成功和失败的摘要。

## 2. 微服务客户端示例

为了我们的示例，让我们编写一个简单的微服务客户端，负责创建资源并返回该资源的标识符。

我们将声明一个简单的接口MicroserviceClient，这样可以在单元测试中Mock它(使用[Mockito)](https://www.baeldung.com/mockito-series)：

```java
interface MicroserviceClient {
    CompletableFuture<Long> createResource(String resourceName);
}
```

[CompletableFuture单元测试](https://www.baeldung.com/java-completablefuture-unit-test)有其自身的挑战，但测试对MicroserviceClient的单次调用则很简单，在这里我们不详细介绍这一点，而是继续处理可能引发异常的多个客户端调用。

## 3. 合并对微服务的多个调用

让我们首先创建一个单元测试并声明我们的MicroserviceClient的Mock，该Mock对于“Good Resource”的输入返回成功响应，对于“Bad Resource”的输入抛出异常：

```java
@ParameterizedTest
@MethodSource("clientData")
public void givenMicroserviceClient_whenMultipleCreateResource_thenCombineResults(List<String> inputs, int expectedSuccess, int expectedFailure) throws ExecutionException, InterruptedException {
    MicroserviceClient mockMicroservice = mock(MicroserviceClient.class);
    when(mockMicroservice.createResource("Good Resource"))
        .thenReturn(CompletableFuture.completedFuture(123L));
    when(mockMicroservice.createResource("Bad Resource"))
        .thenReturn(CompletableFuture.failedFuture(new IllegalArgumentException("Bad Resource")));
}
```

我们将使其成为一个[参数化测试](https://www.baeldung.com/parameterized-tests-junit-5)，并使用[MethodSource](https://www.baeldung.com/parameterized-tests-junit-5#6-method)传入不同的数据集。我们需要创建一个静态方法，为我们的测试提供JUnit Arguments流：

```java
private static Stream<Arguments> clientData() {
    return Stream.of(
        Arguments.of(List.of("Good Resource"), 1, 0),
        Arguments.of(List.of("Bad Resource"), 0, 1),
        Arguments.of(List.of("Good Resource", "Bad Resource"), 1, 1),
        Arguments.of(List.of("Good Resource", "Bad Resource", "Good Resource", "Bad Resource", "Good Resource"), 3, 2)
    );
}
```

这将创建四个测试执行，传递输入列表以及预期的成功和失败次数。

接下来，让我们回到单元测试并使用测试数据调用MicroserviceClient并将每个结果CompletableFuture收集到List中：

```java
List<CompletableFuture<Long>> clientCalls = new ArrayList<>();
for (String resource : inputs) {
    clientCalls.add(mockMicroservice.createResource(resource));
}
```

现在，我们有了问题的核心部分：**一个CompletableFuture对象列表，我们需要完成并收集其结果，同时处理遇到的任何异常**。

### 3.1 处理异常

在了解如何完成每个CompletableFuture之前，让我们定义一个用于处理异常的辅助方法，我们还将定义并模拟一个Logger来模仿现实世界的错误处理：

```java
private final Logger logger = mock(Logger.class);

private Long handleError(Throwable throwable) {
    logger.error("Encountered error: " + throwable);
    return -1L;
}

interface Logger {
    void error(String message);
}
```

辅助方法只是记录错误消息并返回-1，我们用它来指定无效资源。

### 3.2 使用异常处理完成CompletableFuture

现在，我们需要完成所有CompletableFuture并适当处理任何异常，**我们可以利用CompletableFuture提供的一些工具来实现这一点**：

- exceptionally()：如果CompletableFuture出现异常，则执行一个函数
- join()：CompletableFuture完成后返回其结果

然后，我们可以定义一个辅助方法来完成单个CompletableFuture：

```java
private Long handleFuture(CompletableFuture<Long> future) {
    return future
        .exceptionally(this::handleError)
        .join();
}
```

值得注意的是，我们使用exceptionally()来处理MicroserviceClient调用可能通过我们的handleError()辅助方法抛出的任何异常。最后，我们在CompletableFuture上调用join()以等待客户端调用完成并返回其资源标识符。

### 3.3 处理CompletableFuture列表

回到我们的单元测试，现在可以利用我们的辅助方法以及[Java的Stream API](https://www.baeldung.com/java-streams)来创建一个解决所有客户端调用的简单语句：

```java
Map<Boolean, List<Long>> resultsByValidity = clientCalls.stream()
    .map(this::handleFuture)
    .collect(Collectors.partitioningBy(resourceId -> resourceId != -1L));
```

让我们分解一下这段代码：

- 我们使用handleFuture()辅助方法将每个CompletableFuture映射到结果资源标识符中 
- 我们使用Java的Collectors.partitioningBy()实用程序根据有效性将生成的资源标识符拆分为单独的列表

我们可以通过对分区List的大小进行断言以及检查对我们模拟的Logger的调用来轻松验证我们的测试：

```java
List<Long> validResults = resultsByValidity.getOrDefault(true, List.of());
assertThat(validResults.size()).isEqualTo(successCount);

List<Long> invalidResults = resultsByValidity.getOrDefault(false, List.of());
assertThat(invalidResults.size()).isEqualTo(errorCount);
verify(logger, times(errorCount))
    .error(eq("Encountered error: java.lang.IllegalArgumentException: Bad Resource"));
```

运行测试，我们可以看到我们的分区列表与我们的预期相符。

## 4. 总结

在本文中，我们学习了如何处理CompletableFuture集合的完成，如有必要，可以轻松扩展我们的方法以使用更强大的错误处理或复杂的业务逻辑。