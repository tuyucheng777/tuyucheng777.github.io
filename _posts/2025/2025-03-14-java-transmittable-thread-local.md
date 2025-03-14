---
layout: post
title:  transmissiontable-thread-local简介
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在代码执行期间存储上下文是一项常见的挑战。例如，我们可能会在Web请求期间存储安全属性，或者保留可追溯性字段(如事务ID)以供记录或在整个系统中共享。为了处理这个问题，我们可以使用[ThreadLocal](https://www.baeldung.com/java-threadlocal)或InheritableThreadLocal字段，这些类为我们的上下文提供了一个强大的容器，同时确保了线程隔离。但是，这些类有局限性。

在本文中，我们将探讨如何使用[transmissiontable-thread-local](https://github.com/alibaba/transmittable-thread-local/blob/master/README-EN.md)库中的TransmittableThreadLocal来克服多线程问题并安全地管理上下文。

## 2. ThreadLocal问题

我们可以使用ThreadLocal来存储调用上下文。**但是，如果我们尝试从另一个线程访问它，我们将无法获取该值**。让我们看一个简单的例子来说明这个问题：

```java
@Test
void givenThreadLocal_whenTryingToGetValueFromAnotherThread_thenNullIsExpected() {
    ThreadLocal<String> transactionID = new ThreadLocal<>();
    transactionID.set(UUID.randomUUID().toString());
    new Thread(() -> assertNull(transactionID.get())).start();
}
```

我们在主线程中设置了UUID，并在新线程中检索它。正如预期的那样，我们没有得到该值。

## 3. InheritableThreadLocal问题

通过使用[InheritableThreadLocal](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/InheritableThreadLocal.html)，我们可以避免多线程访问上下文的问题。我们可以从主线程下创建的任何线程访问存储的值。**但是，我们在这里仍然可能存在限制。如果我们在过程中修改上下文，更新的值将不会出现在并行线程中**。

让我们看看它是如何工作的：

```java
@Test
void givenInheritableThreadLocal_whenChangeTheTransactionIdAfterSubmissionToThreadPool_thenNewValueWillNotBeAvailableInParallelThread() {
    String firstTransactionIDValue = UUID.randomUUID().toString();
    InheritableThreadLocal<String> transactionID = new InheritableThreadLocal<>();
    transactionID.set(firstTransactionIDValue);
    Runnable task = () -> assertEquals(firstTransactionIDValue, transactionID.get());

    ExecutorService executorService = Executors.newFixedThreadPool(1);
    executorService.submit(task);

    String secondTransactionIDValue = UUID.randomUUID().toString();
    Runnable task2 = () -> assertNotEquals(secondTransactionIDValue, transactionID.get());
    transactionID.set(secondTransactionIDValue);
    executorService.submit(task2);

    executorService.shutdown();
}
```

我们创建一个UUID值并将其设置在InheritableThreadLocal变量中，然后，我们在线程池中运行的单独线程中检查该值，我们确认线程池中的值与主线程中设置的值相匹配。接下来，我们更新变量并再次在线程池中检查该值。这次我们检索了之前的值，并且我们的更新被忽略。

## 4. 使用transmittable-thread-local库

TransmittableThreadLocal是阿里开源的transmissiontable-thread-local库中的一个类，它扩展了InheritableThreadLocal。它支持跨线程共享值，甚至使用线程池也是如此。**我们可以使用它来确保在执行期间上下文更改在所有线程中保持同步**。

### 4.1 依赖

让我们首先添加必要的[依赖](https://mvnrepository.com/artifact/com.alibaba/transmittable-thread-local)：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>transmittable-thread-local</artifactId>
    <version>2.14.5</version>
</dependency>
```

添加此依赖后，我们可以使用TransmittableThreadLocal类。

### 4.2 单个并行线程示例

在第一个例子中，**我们将检查TransmittableThreadLocal变量是否可以跨线程存储值**：

```java
@Test
void givenTransmittableThreadLocal_whenTryingToGetValueFromAnotherThread_thenValueIsPresent() {
    TransmittableThreadLocal<String> transactionID = new TransmittableThreadLocal<>();
    transactionID.set(UUID.randomUUID().toString());
    new Thread(() -> assertNotNull(transactionID.get())).start();
}
```

我们创建一个事务ID并在另一个线程中成功检索其值。

### 4.3 ExecutorService示例

在下一个示例中，**我们将创建一个TransmittableThreadLocal变量，其具有事务ID**。然后，我们将它提交到线程池并在过程中对其进行修改：

```java
@Test
void givenTransmittableThreadLocal_whenChangeTheTransactionIdAfterSubmissionToThreadPool_thenNewValueWillBeAvailableInParallelThread() {
    String firstTransactionIDValue = UUID.randomUUID().toString();
    String secondTransactionIDValue = UUID.randomUUID().toString();

    TransmittableThreadLocal<String> transactionID = new TransmittableThreadLocal<>();
    transactionID.set(firstTransactionIDValue);

    Runnable task = () -> assertEquals(firstTransactionIDValue, transactionID.get());
    Runnable task2 = () -> assertEquals(secondTransactionIDValue, transactionID.get());

    ExecutorService executorService = Executors.newFixedThreadPool(1);
    executorService.submit(TtlRunnable.get(task));

    transactionID.set(secondTransactionIDValue);
    executorService.submit(TtlRunnable.get(task2));

    executorService.shutdown();
}
```

我们可以看到成功检索了初始值和修改值。**我们在这里使用[TtlRunnable](https://github.com/alibaba/transmittable-thread-local/blob/master/ttl-core/src/main/java/com/alibaba/ttl3/TtlRunnable.java)，此类允许我们在线程池中的线程之间传输线程本地状态**。

### 4.4 并行流示例

使用TransmittableThreadLocal变量的另一个有趣情况涉及并行流，**当我们的流中有多个元素时，它可能会在[ForkJoinPool](https://www.baeldung.com/java-fork-join#forkJoinPool)上执行**，这可能会导致池中所有线程共享上下文的问题。让我们看看如何使用TransmittableThreadLocal解决这个挑战：

```java
@Test
void givenTransmittableThreadLocal_whenChangeTheTransactionIdAfterParallelStreamAlreadyProcessed_thenNewValueWillBeAvailableInTheSecondParallelStream() {
    String firstTransactionIDValue = UUID.randomUUID().toString();
    String secondTransactionIDValue = UUID.randomUUID().toString();

    TransmittableThreadLocal<String> transactionID = new TransmittableThreadLocal<>();
    transactionID.set(firstTransactionIDValue);

    TtlExecutors.getTtlExecutorService(new ForkJoinPool(4))
        .submit(
            () -> List.of(1, 2, 3, 4, 5)
                .parallelStream()
                .forEach(i -> assertEquals(firstTransactionIDValue, transactionID.get())));

    transactionID.set(secondTransactionIDValue);
    TtlExecutors.getTtlExecutorService(new ForkJoinPool(4))
        .submit(
            () -> List.of(1, 2, 3, 4, 5)
                .parallelStream()
                .forEach(i -> assertEquals(secondTransactionIDValue, transactionID.get())));
}
```

**由于我们无法修改用于所有并行线程的共享线程池，因此我们需要在单独的ThreadPoolExecutor中运行流。我们使用[TtlExecutors](https://github.com/noseew/transmittable-thread-local-fork/blob/master/ttl-core/src/main/java/com/alibaba/ttl3/executor/TtlExecutors.java)包装器来同步主线程和并行流执行期间使用的所有线程之间的上下文**。

在我们的实验中，我们在主线程中创建并修改了事务ID。此外，我们从并行流中访问了此事务ID，并成功检索了初始值和修改后的值。

## 5. 总结

在本教程中，我们探索了线程局部变量的不同实现。

简单的ThreadLocal变量对于具有特定上下文的单线程执行很有用。当我们需要在多个继承线程之间共享上下文时，我们使用InheritableThreadLocal。最后，我们可以从transmissiontable-thread-local库中选择TransmittableThreadLocal来同步线程池内线程之间的上下文更改。