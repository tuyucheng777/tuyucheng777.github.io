---
layout: post
title:  如何获取正在执行的方法的名称
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

有时我们需要知道当前正在执行的Java方法的名称。

这篇简短的文章介绍了几种获取当前执行堆栈中方法名称的简单方法。

## 2. Java 9：StackWalker API

**Java 9引入了[StackWalking API](https://www.baeldung.com/java-9-stackwalking-api)，以惰性且高效的方式遍历JVM堆栈帧**。为了使用此API找到当前正在执行的方法，我们可以编写一个简单的测试：

```java
public void givenJava9_whenWalkingTheStack_thenFindMethod() {
    StackWalker walker = StackWalker.getInstance();
    Optional<String> methodName = walker.walk(frames -> frames
        .findFirst()
        .map(StackWalker.StackFrame::getMethodName));

    assertTrue(methodName.isPresent());
    assertEquals("givenJava9_whenWalkingTheStack_thenFindMethod", methodName.get());
}
```

首先，我们使用[getInstance()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StackWalker.html#getInstance())工厂方法获取[StackWalker](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StackWalker.html)实例。**然后我们使用[walk()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StackWalker.html#walk(java.util.function.Function))方法从顶部到底部遍历堆栈帧**： 

- walk()方法可以将堆栈帧流(Stream\<[StackFrame](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StackWalker.StackFrame.html)\>)转换为任何内容
- 给定流中的第一个元素是堆栈上的顶部帧
- 堆栈顶部的帧始终代表当前正在执行的方法

因此，如果我们从流中获取第一个元素，我们就会知道当前正在执行的方法的详细信息。更具体地说，**我们可以使用[StackFrame.getMethodName()](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/StackWalker.StackFrame.html#getMethodName())来查找方法名称**。

### 2.1 优点

与其他方法(稍后会详细介绍)相比，StackWalking API具有以下优点：

- 无需创建虚拟匿名内部类实例：new Object().getClass(){}
- 不需要创建虚拟异常：new Throwable()
- 无需急切地捕获整个堆栈跟踪，这可能会很昂贵

相反，StackWalker只会以惰性方式逐个遍历堆栈。在这种情况下，它只会获取顶部的帧，而不像stacktrace方法那样会急切地捕获所有帧。

最重要的是，如果你使用的是Java 9+，请使用StackWalking API。

## 3. 使用getEnclosingMethod

我们可以使用getEnclosingMethod() API找到正在执行的方法的名称：

```java
public void givenObject_whenGetEnclosingMethod_thenFindMethod() {
    String methodName = new Object() {}
        .getClass()
        .getEnclosingMethod()
        .getName();
       
    assertEquals("givenObject_whenGetEnclosingMethod_thenFindMethod", methodName);
}
```

## 4. 使用Throwable堆栈跟踪

使用Throwable堆栈跟踪可以为我们提供当前正在执行的方法的堆栈跟踪：

```java
public void givenThrowable_whenGetStacktrace_thenFindMethod() {
    StackTraceElement[] stackTrace = new Throwable().getStackTrace();
 
    assertEquals(
        "givenThrowable_whenGetStacktrace_thenFindMethod",
        stackTrace[0].getMethodName());
}
```

## 5. 使用线程堆栈跟踪

此外，当前线程的堆栈跟踪(自JDK 1.5起)通常包含正在执行的方法的名称：

```java
public void givenCurrentThread_whenGetStackTrace_thenFindMethod() {
    StackTraceElement[] stackTrace = Thread.currentThread()
        .getStackTrace();
 
    assertEquals(
        "givenCurrentThread_whenGetStackTrace_thenFindMethod",
        stackTrace[1].getMethodName()); 
}
```

但是，我们需要记住，这个解决方案有一个明显的缺点。一些虚拟机可能会跳过一个或多个堆栈帧，虽然这种情况并不常见，但我们应该意识到这种情况可能会发生。

## 6. 总结

在本教程中，我们提供了一些如何获取当前执行的方法名称的示例，示例基于堆栈跟踪和getEnclosingMethod()。