---
layout: post
title:  什么导致java.lang.reflect.InvocationTargetException？
category: java-reflect
copyright: java-reflect
excerpt: Java反射
---

## 1. 概述

使用[Java反射API](https://www.baeldung.com/java-reflection)时，经常会遇到java.lang.reflect.InvocationTargetException。

在本教程中，我们将通过一个简单的示例来了解如何处理它。

## 2. InvocationTargetException的原因

它主要发生在我们使用反射层并尝试调用引发底层异常的方法或构造函数时。

**反射层使用InvocationTargetException包装方法抛出的实际异常**。

让我们尝试通过一个例子来理解它。

我们将编写一个类，其中的方法故意抛出异常：

```java
public class InvocationTargetExample {
    public int divideByZeroExample() {
        return 1 / 0;
    }
}
```

让我们在一个简单的JUnit 5测试中使用反射来调用上述方法：

```java
InvocationTargetExample targetExample = new InvocationTargetExample(); 
Method method = InvocationTargetExample.class.getMethod("divideByZeroExample");
 
Exception exception = assertThrows(InvocationTargetException.class, () -> method.invoke(targetExample));
```

在上面的代码中，我们断言了InvocationTargetException，它在调用方法时抛出。为了更容易调查异常，我们通常检查其[堆栈跟踪](https://www.baeldung.com/java-get-current-stack-trace)。所以，让我们打印这个异常的堆栈跟踪：

```java
LOG.error("InvocationTargetException Thrown:", exception);
```

然后我们可以在控制台中看到错误日志：

```text
18:08:28.662 [main] ERROR ...InvocationTargetUnitTest -- InvocationTargetException Thrown
java.lang.reflect.InvocationTargetException: null
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:118)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	at cn.tuyucheng.taketoday.reflection.exception.invocationtarget.InvocationTargetUnitTest.lambda$whenCallingMethodThrowsException_thenAssertCauseOfInvocationTargetException$0(InvocationTargetUnitTest.java:23)
	at org.junit.jupiter.api.AssertThrows.assertThrows(AssertThrows.java:53)
	at org.junit.jupiter.api.AssertThrows.assertThrows(AssertThrows.java:35)
	at org.junit.jupiter.api.Assertions.assertThrows(Assertions.java:3128)
	at cn.tuyucheng.taketoday.reflection.exception.invocationtarget.InvocationTargetUnitTest.whenCallingMethodThrowsException_thenAssertCauseOfInvocationTargetException(InvocationTargetUnitTest.java:23)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	...
Caused by: java.lang.ArithmeticException: / by zero
	at cn.tuyucheng.taketoday.reflection.exception.invocationtarget.InvocationTargetExample.divideByZeroExample(InvocationTargetExample.java:5)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	... 74 common frames omitted
```

值得注意的是，实际的异常(本例中为ArithmeticException)被包装到InvocationTargetException中。因此，有人可能会问，为什么反射一开始不抛出实际的异常？

这是因为**InvocationTargetException的目的是提供一种方法来区分反射调用本身期间抛出的异常和被调用的方法抛出的异常**。

## 3. 如何处理InvocationTargetException？

由于InvocationTargetException包装了实际原因，因此当我们调查堆栈跟踪时，我们必须搜索实际的底层异常。这可能很不方便。此外，我们可能需要以编程方式访问底层异常，以便在代码中适当地处理它。

为了实现这一点，**我们可以使用Throwable.getCause()来获取InvocationTargetException的实际原因**。

接下来，让我们使用上面相同的示例来看看在我们的应用程序中通常是如何做到这一点的：

```java
try {
    InvocationTargetExample targetExample = new InvocationTargetExample();
    Method method = InvocationTargetExample.class.getMethod("divideByZeroExample");
    method.invoke(targetExample);
} catch (InvocationTargetException e) {
    Throwable cause = e.getCause();
    if (cause instanceof ArithmeticException) {
        // ... handle ArithmeticException
        LOG.error("InvocationTargetException caused by ArithmeticException:", cause);
    } else {
        // ... handle other exceptions
        LOG.error("The underlying exception of InvocationTargetException:", cause);
    }
} catch (NoSuchMethodException | IllegalAccessException e) {
    // for simplicity, rethrow reflection exceptions as RuntimeException
    throw new RuntimeException(e);
}
```

在这个例子中，在InvocationTargetException catch块中，getCause()为我们提供了底层异常。

一旦我们得到了底层异常，我们就可以处理它或将其包装在一些自定义异常中，这使我们能够**以更具体和可控的方式处理异常**。

在示例中，如果出现ArithmeticException，我们希望记录cause的堆栈跟踪。因此，如果我们运行代码，我们将看到底层异常的堆栈跟踪：

```text
18:39:10.946 [main] ERROR ...InvocationTargetUnitTest -- InvocationTargetException caused By ArithmeticException:
java.lang.ArithmeticException: / by zero
	at cn.tuyucheng.taketoday.reflection.exception.invocationtarget.InvocationTargetExample.divideByZeroExample(InvocationTargetExample.java:5)
	at java.base/jdk.internal.reflect.DirectMethodHandleAccessor.invoke(DirectMethodHandleAccessor.java:103)
	at java.base/java.lang.reflect.Method.invoke(Method.java:580)
	...
```

我们可以看到，输出清楚地显示了实际的异常类型、消息以及哪个类中的哪一行代码导致了该异常。

## 4. 总结

InvocationTargetException是在Java中使用反射时遇到的常见异常。

在这篇简短的文章中，我们讨论了什么是InvocationTargetException，并通过一个简单的示例探讨了如何确定其根本原因以及如何处理这种情况。

了解如何有效地管理InvocationTargetException是调试或处理应用程序中的错误的基本技能。