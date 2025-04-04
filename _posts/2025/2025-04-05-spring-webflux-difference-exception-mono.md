---
layout: post
title:  Spring Webflux中抛出异常和Mono.error()的区别
category: spring-reactive
copyright: spring-reactive
excerpt: Spring Reactive
---

## 1. 概述

[错误处理](https://www.baeldung.com/spring-webflux-errors)是使用[Spring WebFlux](https://www.baeldung.com/spring-webflux)进行响应式编程的一个关键方面，开发人员通常依靠两种主要方法进行错误处理：抛出异常或使用[Project Reactor](https://www.baeldung.com/reactor-core)提供的Mono.error()方法。这两种方法都用于发出错误信号，但它们具有不同的特征和用例。

**在本教程中，我们将解释在Spring WebFlux中抛出异常和Mono.error()之间的区别**。我们将提供说明性的Java代码示例，使其更易于理解。

## 2. 传统方法：抛出异常

多年来，抛出异常一直是管理Java应用程序中错误的可靠方法，这是一种中断程序常规流程并将错误传达到应用程序更高层的简单方法。**Spring WebFlux与这种传统的错误处理方法顺利集成，使开发人员能够在其响应端点中抛出异常**，以下代码代表了传统方法的一个示例：

```java
public Mono<User> getUserByIdThrowingException(String id) {
    User user = userRepository.findById(id);
    if (user == null) {
       throw new NotFoundException("User Not Found");
    }
    return Mono.justOrEmpty(user);
}
```

在这个特定场景中，getUserByIdThrowingException()方法尝试根据UserRepository提供的ID检索用户数据。如果找不到用户，该方法将抛出NotFoundException，表示响应管道内出现错误。

为了执行单元测试，我们从org.junit.jupiter.api.Assertions导入了assertThrows方法。这将测试getUserByIdThrowingException()是否会对数据库中未找到的用户抛出NotFoundException，我们使用带有Lambda的assertThrows来执行应该抛出异常的方法调用。

如果引发异常，代码将验证引发的异常是否属于预期类型。但是，如果该方法未引发异常，则测试失败：

```java
@Test
public void givenNonExistUser_whenFailureCall_then_Throws_exception() {
    assertThrows(
            NotFoundException.class,
            () -> userService.getUserByIdThrowingException("3")
    );
}
```

## 3. 拥抱响应式：Mono.error()

与传统的抛出异常的方法不同，Project Reactor通过Mono.error()方法引入了一种响应式替代方案。**此方法会生成一个Mono，它会立即终止并发出错误信号，与响应式编程范式无缝契合**。

让我们使用Mono.error()检查一下修改后的示例：

```java
public Mono<User> getUserByIdUsingMonoError(String id) {
    User user = userRepository.findById(id);
    return (user != null)
            ? Mono.justOrEmpty(user)
            : Mono.error(new NotFoundException("User Not Found"));
}
```

为了保持流式的用户体验和一致的响应流程，我们使用Mono.error()，而不是直接为数据库中未找到的用户抛出异常。

以下是此方法的单元测试：

```java
@Test
 public void givenNonExistUser_whenFailureCall_then_returnMonoError() {
    Mono result = userService.getUserByIdUsingMonoError("3");
    StepVerifier.create(result)
        .expectError(NotFoundException.class)
        .verify();
}
```

## 4. 了解主要差异和用例

### 4.1 控制流中断

当Mono.error()发出信号时，我们使用try-catch或响应式运算符(如onErrorResume，onErrorReturn或onErrorMap)来处理异常。

### 4.2 惰性

Mono.error()现在支持异常的惰性实例化，这在构造异常涉及资源密集型操作的场景中非常有用。

### 4.3 响应式错误处理

Mono.error()与响应式编程范式非常契合，有助于在响应流中进行响应式错误处理。

## 5. 总结

在本文中，我们讨论了在响应式应用程序中抛出异常和在Spring WebFlux中使用Mono.error()进行错误处理之间的根本区别；尽管两种方法都用于发送错误信号的相同目的，但它们的控制流和与响应式管道的集成存在显著差异。

**抛出异常会中断执行流程并将控制权转移到最近的异常处理程序，使其适合命令式代码路径。相反，Mono.error()可与响应流无缝集成，无需停止执行流程即可实现异步错误信号发送**。

使用Spring WebFlux开发响应式应用程序时，根据上下文和需求选择正确的错误处理机制至关重要。我们在响应式管道中使用Mono.error()来保持其响应性，并对命令式代码路径使用异常。