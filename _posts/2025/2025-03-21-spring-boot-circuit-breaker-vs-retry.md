---
layout: post
title:  Spring Boot中断路器和重试的区别
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

在分布式系统和微服务架构中，妥善处理故障对于保持系统可靠性和性能至关重要，有助于实现这一点的两种基本弹性模式是断路器和重试。虽然这两种模式都旨在提高系统稳定性和可靠性，但它们的用途截然不同，适用于不同的场景。

在本文中，我们将深入探讨这些模式，包括它们的机制、用例以及在[Spring Boot](https://www.baeldung.com/spring-boot-resilience4j)中使用[Resilience4j](https://www.baeldung.com/resilience4j)的实现细节。

## 2. 什么是重试？

重试模式是一种简单但功能强大的机制，用于**处理分布式系统中的瞬时故障**。当操作失败时，重试模式会尝试多次执行相同的操作，希望临时问题能够自行解决。

### 2.1 重试的关键特征

重试机制围绕特定属性展开，这些属性使其能够有效处理瞬态问题，确保临时故障不会升级为严重问题：

- **重复尝试**：核心思想是重新执行失败的操作指定的次数
- **退避策略**：这是一种高级重试机制，包括退避策略，例如指数退避，有助于避免系统过载
- **非常适合临时故障**：最适合间歇性网络问题、临时服务不可用或瞬时资源限制

### 2.2 重试实现示例

我们来看一个使用Resilience4j实现重试机制的简单示例：

```java
@Test
public void whenRetryWithExponentialBackoffIsUsed_thenItRetriesAndSucceeds() {
    IntervalFunction intervalFn = IntervalFunction.ofExponentialBackoff(1000, 2);
    RetryConfig retryConfig = RetryConfig.custom()
        .maxAttempts(5)
        .intervalFunction(intervalFn)
        .build();

    Retry retry = Retry.of("paymentRetry", retryConfig);

    when(paymentService.process(1)).thenThrow(new RuntimeException("First Failure"))
          .thenThrow(new RuntimeException("Second Failure"))
          .thenReturn("Success");

    Callable<String> decoratedCallable = Retry.decorateCallable(
        retry, () -> paymentService.processPayment(1)
    );

    try {
        String result = decoratedCallable.call();
        assertEquals("Success", result);
    } catch (Exception ignored) {
    }

    verify(paymentService, times(3)).processPayment(1);
}
```

在此示例中：

- 重试机制最多尝试该操作5次
- 它采用指数退避策略在尝试之间引入延迟，从而降低系统过载的风险
- 重试两次后操作成功

## 3. 什么是断路器模式？

[断路器模式](https://www.baeldung.com/cs/microservices-circuit-breaker-pattern)是一种更高级的故障处理方法，它可以防止应用程序反复尝试执行可能失败的操作，从而防止级联故障并提供系统稳定性。

### 3.1 断路器的主要特性

断路器模式专注于**防止故障服务承受过大负载并缓解级联故障**，让我们回顾一下它的关键属性：

- **状态管理**：断路器有三个主要状态：

  - **已关闭**：正常运营，允许请求继续进行
  - **打开**：阻止所有请求以防止进一步失败
  - **半开放**：允许有限数量的测试请求来检查系统是否已恢复

- **故障阈值**：监控滑动窗口内失败请求的百分比，当故障率超过配置的阈值时“跳闸”

- **防止级联故障**：停止对故障服务的重复调用，保护整个系统免于降级

### 3.2 断路器实现示例

以下是断路器实现的一个简单示例，展示了状态转换：

```java
@Test
public void whenCircuitBreakerTransitionsThroughStates_thenBehaviorIsVerified() {
    CircuitBreakerConfig circuitBreakerConfig = CircuitBreakerConfig.custom()
        .failureRateThreshold(50)
        .slidingWindowSize(5)
        .permittedNumberOfCallsInHalfOpenState(3)
        .build();

    CircuitBreaker circuitBreaker = CircuitBreaker.of("paymentCircuitBreaker", circuitBreakerConfig);

    AtomicInteger callCount = new AtomicInteger(0);

    when(paymentService.processPayment(anyInt())).thenAnswer(invocationOnMock -> {
        callCount.incrementAndGet();
        throw new RuntimeException("Service Failure");
    });

    Callable<String> decoratedCallable = CircuitBreaker.decorateCallable(
        circuitBreaker, () -> paymentService.processPayment(1)
    );

    for (int i = 0; i < 10; i++) {
        try {
            decoratedCallable.call();
        } catch (Exception ignored) {
        }
    }

    assertEquals(5, callCount.get());
    assertEquals(CircuitBreaker.State.OPEN, circuitBreaker.getState());

    callCount.set(0);
    circuitBreaker.transitionToHalfOpenState();

    assertEquals(CircuitBreaker.State.HALF_OPEN, circuitBreaker.getState());
    reset(paymentService);
    when(paymentService.processPayment(anyInt())).thenAnswer(invocationOnMock -> {
        callCount.incrementAndGet();
        return "Success";
    });

    for (int i = 0; i < 3; i++) {
        try {
            decoratedCallable.call();
        } catch (Exception ignored) {
        }
    }

    assertEquals(3, callCount.get());
    assertEquals(CircuitBreaker.State.CLOSED, circuitBreaker.getState());
}
```

在此示例中：

- 50%的故障率阈值和5次调用的滑动窗口决定了断路器何时“跳闸”
- 经过5次尝试失败后，电路断开，立即拒绝进一步调用
- 等待1秒后，电路转换为半开状态
- 在半开状态下，3次成功调用，导致断路器转换到关闭状态，恢复正常运行

## 4. 主要区别：重试与断路器

|      方面      |    重试模式    |         断路器模式         |
| :------------: |:----------:| :------------------------: |
|  主要目标  |   尝试操作多次   |   防止重复调用失败的服务   |
|  故障处理  |   假设瞬时故障   |     假设潜在的系统故障     |
|  状态管理  |  无状态，一直尝试  | 保持状态(关闭/打开/半开)|
| 最适合用于 | 间歇性、可恢复的错误 |     持续性或系统性故障     |

## 5. 何时使用每种模式

决定何时使用重试或断路器取决于我们的系统遇到的故障类型，这些模式相辅相成，了解它们的应用可以帮助我们构建有效处理错误的弹性系统。

- **在以下情况下使用重试**：
  - 处理暂时的网络问题
  - 预计服务将暂时不可用
  - 重试几次后即可快速恢复
- **在以下情况下使用断路器**：
  - 防止长期服务故障
  - 防止微服务中的级联故障
  - 实施自我修复系统架构

在实际应用中，这些模式经常一起使用。例如，重试机制可以在断路器的范围内工作，确保仅在断路器闭合或半开时尝试重试。

## 6. 最佳实践

为了最大程度地发挥这些模式的有效性：

- **监控指标**：持续监控故障率、重试尝试次数和电路状态以微调配置
- **组合模式**：对瞬态错误使用重试，对系统故障使用断路器
- **设置现实的阈值**：过于激进的阈值可能会阻碍恢复或延迟故障检测
- **利用库**：使用像Resilience4j或Spring Cloud Circuit Breaker这样的强健的库，它在底层实现Resilience4j和Spring Retry，以简化实现

## 7. Spring Boot集成

Spring Boot通过其生态系统为断路器和重试模式提供全面支持，这种集成主要**通过Spring Cloud Circuit Breaker项目和Spring Retry模块实现**。

[Spring Cloud Circuit Breaker项目](https://www.baeldung.com/spring-cloud-circuit-breaker)提供了一个抽象层，使我们能够实现断路器，而无需绑定到特定实现。这意味着我们可以根据需要在不同的断路器实现(如Resilience4j、Hysterix、Sentinel或Spring Retry)之间切换，而无需更改应用程序代码。**该项目使用Spring Boot的自动配置机制**，当它在类路径中检测到适当的启动器时，它会自动配置必要的断路器Bean。

对于重试功能，Spring Boot与[Spring Retry](https://www.baeldung.com/spring-retry)集成，提供基于注解和编程的方法来实现重试逻辑。该框架通过属性文件和Java配置提供灵活的配置选项，允许我们自定义重试尝试、退避策略和恢复策略。

让我们看一下Spring Boot与这些模式集成的一些特性，这些特性使其特别强大：

- **自动配置支持**：Spring Boot根据我们类路径中的依赖项自动配置断路器和重试Bean，从而减少样板配置代码。
- **可插入式架构**：抽象层允许我们在不同的断路器实现之间切换，而无需修改我们的业务逻辑。
- **配置灵活性**：两种模式都可以通过应用程序属性或Java配置进行配置，支持不同服务的全局配置和特定配置。
- **与Spring生态系统的集成**：这些模式可以与其他Spring组件(如RestTemplate、WebClient和各种Spring Cloud组件)无缝协作。
- **监控和指标**：Spring Boot的执行器集成为断路器和重试尝试提供了内置监控功能，帮助我们跟踪弹性机制的健康和行为。

这种集成方法**符合Spring Boot的约定优于配置理念**，同时保持了在需要时自定义行为的灵活性。该框架对这些模式的支持使构建弹性微服务变得更加容易，这些微服务可以优雅地处理故障并保持系统稳定性。

## 8. 总结

重试和断路器都是分布式系统中必不可少的弹性模式。重试侧重于立即恢复，而断路器则提供针对级联故障的强大保护。通过了解它们的区别和用例，我们可以设计出既可靠又容错的系统。

借助Resilience4j和Spring Cloud Circuit Breaker等库，Spring Boot提供了一个强大的平台来轻松实现这些模式。通过采用这些弹性策略，我们可以构建能够优雅地承受故障的应用程序，确保即使在恶劣条件下也能提供无缝的用户体验。