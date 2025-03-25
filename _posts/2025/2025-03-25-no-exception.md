---
layout: post
title:  NoException简介
category: libraries
copyright: libraries
excerpt: NoException
---

## 1. 概述

有时try/catch块可能会导致冗长甚至不合理的代码结构。

在本文中，我们将重点介绍**提供简洁易用的异常处理程序的[NoException](https://noexception.machinezoo.com/)**。

## 2. Maven依赖

让我们将[NoException](https://mvnrepository.com/search?q=noexception)添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>com.machinezoo.noexception</groupId>
    <artifactId>noexception</artifactId>
    <version>1.1.0</version>
</dependency>
```

## 3. 标准异常处理

我们先从一个常见的习语开始：

```java
private static Logger logger = LoggerFactory.getLogger(NoExceptionUnitTest.class);

@Test
public void whenStdExceptionHandling_thenCatchAndLog() {
    try {
        logger.info("Result is " + Integer.parseInt("foobar"));
    } catch (Throwable exception) {
        logger.error("Caught exception:", exception);
    }
}
```

我们首先分配一个Logger，然后进入一个try块。如果抛出异常，我们记录它：

```text
09:29:28.140 [main] ERROR c.t.t.n.NoExceptionUnitTest 
  - Caught exception
j.l.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

## 4. 使用NoException处理异常

### 4.1 默认日志处理程序

让我们用NoException的标准异常处理程序替换它：

```java
@Test
public void whenDefaultNoException_thenCatchAndLog() {
    Exceptions 
        .log()
        .run(() -> System.out.println("Result is " + Integer.parseInt("foobar")));
}
```

此代码为我们提供了与上面几乎相同的输出：

```text
09:36:04.461 [main] ERROR c.m.n.Exceptions 
  - Caught exception
j.l.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

**在最基本的形式中，NoException为我们提供了一种用一行代码替换try/catch/exceptions的方法**。它执行我们传递给run()的Lambda，如果抛出异常，它会被记录下来。

### 4.2 添加自定义记录器

如果我们仔细观察输出，我们会发现异常被记录为日志记录类，而不是我们的。

我们可以通过提供记录器来解决这个问题：

```java
@Test
public void whenDefaultNoException_thenCatchAndLogWithClassName() {
    Exceptions
        .log(logger)
        .run(() -> System.out.println("Result is " + Integer.parseInt("foobar")));
}
```

这给了我们以下输出：

```text
09:55:23.724 [main] ERROR c.t.t.n.NoExceptionUnitTest 
  - Caught exception
j.l.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

### 4.3 提供自定义日志消息

我们可能希望使用不同于默认“Caught Exception”的消息，**我们可以通过传递一个Logger作为第一个参数和一个String消息作为第二个参数来做到这一点**：

```java
@Test
public void whenDefaultNoException_thenCatchAndLogWithMessage() {
    Exceptions
        .log(logger, "Something went wrong:")
        .run(() -> System.out.println("Result is " + Integer.parseInt("foobar")));
}
```

这给了我们以下输出：

```text
09:55:23.724 [main] ERROR c.t.t.n.NoExceptionUnitTest 
  - Something went wrong:
j.l.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

但是如果我们想要做的不仅仅是记录异常怎么办，比如在parseInt()失败时插入一个回退值？

### 4.4 指定默认值

异常可以返回包裹在Optional中的结果。让我们移动一下，以便在目标失败时使用它来提供默认值：

```java
@Test
public void
givenDefaultValue_whenDefaultNoException_thenCatchAndLogPrintDefault() {
    System.out.println("Result is " + Exceptions
        .log(logger, "Something went wrong:")
        .get(() -> Integer.parseInt("foobar"))
        .orElse(-1));
}
```

我们仍然看到异常：

```text
12:02:26.388 [main] ERROR c.t.t.n.NoExceptionUnitTest
  - Caught exception java.lang.NumberFormatException: For input string: "foobar"
at j.l.NumberFormatException.forInputString(NumberFormatException.java:65)
at j.l.Integer.parseInt(Integer.java:580)
...
```

但我们也看到我们的消息也打印到控制台：

```text
Result is -1
```

## 5. 创建自定义日志处理程序

到目前为止，我们已经有了一种很好的方法，可以在简单的try/catch/log场景中避免重复并使代码更具可读性。如果我们想重用具有不同行为的处理程序怎么办？

让我们扩展NoException的ExceptionHandler类并根据异常类型执行以下两项操作之一：

```java
public class CustomExceptionHandler extends ExceptionHandler {

    Logger logger = LoggerFactory.getLogger(CustomExceptionHandler.class);

    @Override
    public boolean handle(Throwable throwable) {
        if (throwable.getClass().isAssignableFrom(RuntimeException.class)
                || throwable.getClass().isAssignableFrom(Error.class)) {
            return false;
        } else {
            logger.error("Caught Exception", throwable);
            return true;
        }
    }
}
```

**当我们看到Error或RuntimeException时，通过返回false，我们告诉ExceptionHandler重新抛出。通过对其他所有情况返回true，我们表明异常已被处理**。

首先，我们将以标准异常运行它：

```java
@Test
public void givenCustomHandler_whenError_thenRethrowError() {
    CustomExceptionHandler customExceptionHandler = new CustomExceptionHandler();
    customExceptionHandler.run(() -> "foo".charAt(5));
}
```

我们将函数传递给继承自ExceptionHandler的自定义处理程序中的run()方法：

```text
18:35:26.374 [main] ERROR c.t.t.n.CustomExceptionHandler 
  - Caught Exception 
j.l.StringIndexOutOfBoundsException: String index out of range: 5
at j.l.String.charAt(String.java:658)
at c.t.t.n.CustomExceptionHandling.throwSomething(CustomExceptionHandling.java:20)
at c.t.t.n.CustomExceptionHandling.lambda$main$0(CustomExceptionHandling.java:10)
at c.m.n.ExceptionHandler.run(ExceptionHandler.java:1474)
at c.t.t.n.CustomExceptionHandling.main(CustomExceptionHandling.java:10)
```

记录此异常。让我们尝试使用Error：

```java
@Test(expected = Error.class)
public void givenCustomHandler_whenException_thenCatchAndLog() {
    CustomExceptionHandler customExceptionHandler = new CustomExceptionHandler();
    customExceptionHandler.run(() -> throwError());
}

private static void throwError() {
    throw new Error("This is very bad.");
}
```

我们看到Error被重新抛出到main()中，而不是被记录下来：

```text
Exception in thread "main" java.lang.Error: This is very bad.
at c.t.t.n.CustomExceptionHandling.throwSomething(CustomExceptionHandling.java:15)
at c.t.t.n.CustomExceptionHandling.lambda$main$0(CustomExceptionHandling.java:8)
at c.m.n.ExceptionHandler.run(ExceptionHandler.java:1474)
t c.t.t.n.CustomExceptionHandling.main(CustomExceptionHandling.java:8)
```

因此，我们有一个可重复使用的类，可以在整个项目中使用，以实现一致的异常处理。

## 6. 总结

借助NoException，我们可以通过一行代码简化逐个异常处理。