---
layout: post
title:  如何使用Mockito延迟存根方法响应
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 简介

[Mockito](https://www.baeldung.com/mockito-series)允许开发人员对方法进行存根处理以返回特定值或执行某些操作。在某些测试场景中，我们可能需要在存根方法的响应中引入延迟，以模拟真实情况，例如网络延迟或缓慢的数据库查询。

在本教程中，我们将探索使用Mockito引入此类延迟的不同方法。

## 2. 理解存根方法中延迟的必要性

在存根方法中引入延迟在以下几种情况下可能会有所帮助：

- 模拟真实情况：测试应用程序如何处理延迟和超时
- 性能测试：确保应用程序能够妥善处理缓慢的响应
- 调试：重现和诊断与时序和同步相关的问题

在我们的测试中添加延迟可以帮助我们创建更加强大和有弹性的应用程序，从而更好地适应现实世界的情况。

## 3. Maven依赖

让我们将[mockito-core](https://mvnrepository.com/artifact/org.mockito/mockito-core)和[awaitility](https://mvnrepository.com/artifact/org.awaitility/awaitility)的Maven依赖项添加到我们的项目中：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.11.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>4.2.1</version>
    <scope>test</scope>
</dependency>
```

## 4. 设置

让我们创建一个PaymentService来模拟处理付款：

```java
public class PaymentService {
    public String processPayment() {
        // simulate processing payment and return completion status
        return "SUCCESS";
    }
}
```

我们还将引入延迟来模拟以各种方式处理付款。

## 5. 使用Thread.sleep()引入延迟

引入[延迟](https://www.baeldung.com/java-delay-code-execution)的最简单方法是在存根方法中使用Thread.sleep()，此方法会暂停当前线程的执行指定的毫秒数：

```java
@Test
public void whenProcessingPayment_thenDelayResponseUsingThreadSleep() {
    when(paymentService.processPayment()).thenAnswer(invocation -> {
        Thread.sleep(1000); // Delay of one second
        return "SUCCESS";
    });

    long startTime = System.currentTimeMillis();
    String result = paymentService.processPayment();
    long endTime = System.currentTimeMillis();

    assertEquals("SUCCESS", result);
    assertTrue((endTime - startTime) >= 1000); // Verify the delay
}
```

在此示例中，PaymentService mock的processPayment()方法将等待两秒钟，然后返回“SUCCESS”。**此方法很简单，但会阻塞线程，可能不适合测试异步操作或在生产环境中使用**。

## 6. 使用Mockito的Answer引入延迟

Mockito的[Answer](https://www.baeldung.com/mockito-additionalanswers)接口提供了一种更灵活的存根方法，我们可以使用它来引入延迟以及其他自定义行为：

```java
@Test
public void whenProcessingPayment_thenDelayResponseUsingAnswersWithDelay() throws Exception {
    when(paymentService.processPayment()).thenAnswer(AdditionalAnswers.answersWithDelay(1000, invocation -> "SUCCESS"));

    long startTime = System.currentTimeMillis();
    String result = paymentService.processPayment();
    long endTime = System.currentTimeMillis();

    assertEquals("SUCCESS", result);
    assertTrue((endTime - startTime) >= 1000); // Verify the delay
}
```

在此示例中，使用Mockito的AddedAnswers类中的answersWithDelay()方法在返回“SUCCESS”之前引入2秒的延迟。这种方法抽象了延迟逻辑，使代码更简洁、更易于维护。

## 7. 使用Awaitility引入延迟

[Awaitility](https://www.baeldung.com/awaitility-testing)是一种用于在测试中同步异步操作的DSL，它可用于等待条件满足，但也可以导致延迟。这使得它特别适合测试异步代码：

```java
@Test
public void whenProcessingPayment_thenDelayResponseUsingAwaitility() {
    when(paymentService.processPayment()).thenAnswer(invocation -> {
        Awaitility.await().pollDelay(1, TimeUnit.SECONDS).until(()->true);
        return "SUCCESS";
    });

    long startTime = System.currentTimeMillis();
    String result = paymentService.processPayment();
    long endTime = System.currentTimeMillis();

    assertEquals("SUCCESS", result);
    assertTrue((endTime - startTime) >= 1000); // Verify the delay
}
```

在此示例中，Awaitility.await().pollDelay(1, TimeUnit.SECONDS).until(() -> true)至少延迟一秒才返回“SUCCESS”。Awaitility流式的API使其易于阅读和理解，并且它提供了用于等待异步条件的强大功能。

## 8. 确保测试稳定性

为了确保引入延迟时测试套件的稳定性，请考虑以下最佳实践：

- 设置适当的超时：确保超时足够宽裕以适应延迟，但不要太长以致影响测试执行时间，这可以防止出现问题时测试无限期挂起。
- 模拟外部依赖：如果可能，模拟外部依赖以可靠地控制和模拟延迟。
- 隔离延迟测试：隔离延迟测试，以防止它们影响整体测试套件执行时间，可以通过将此类测试分组到单独的类别中或在不同的环境中运行它们来实现。

## 9. 总结

延迟Mockito中存根方法的响应对于模拟真实世界条件、测试性能和调试很有用。

在本文中，我们通过使用Thread.sleep()、Awaitility和Mockito的Answer有效地引入了这些延迟。此外，确保测试稳定性对于维护可靠、强大的测试至关重要，结合这些技术可以帮助创建更具弹性的应用程序以应对实际场景。