---
layout: post
title:  单例设计模式的缺点
category: designpattern
copyright: designpattern
excerpt: 单例模式
---

## 1. 概述

单例模式是GoF于1994年发表的[创建型设计模式](https://www.baeldung.com/creational-design-patterns)之一。

由于其实现简单，我们往往会过度使用它。因此，如今它被认为是一种反模式。在代码中引入它之前，我们应该问问自己，我们是否真的需要它提供的功能。

在本教程中，我们将讨论[单例](https://www.baeldung.com/java-singleton)设计模式的普遍缺点，并了解一些可以使用的替代方案。

## 2. 代码示例

首先，让我们创建一个将在示例中使用的类：

```java
public class Logger {
    private static Logger instance;

    private PrintWriter fileWriter;

    public static Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }

    private Logger() {
        try {
            fileWriter = new PrintWriter(new FileWriter("app.log", true));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    public void log(String message) {
        String log = String.format("[%s]- %s", LocalDateTime.now(), message);
        fileWriter.println(log);
        fileWriter.flush();
    }
}
```

上面的类代表了一个用于记录到文件的简化类，我们使用惰性初始化方法将其实现为单例。

## 3. 单例的缺点

根据定义，单例模式确保一个类只有一个实例，并且提供对该实例的全局访问。因此，我们应该在需要同时满足这两个条件的情况下使用它。

**查看其定义，我们可以注意到它违反了[单一职责原则](https://www.baeldung.com/java-single-responsibility-principle)**，该原则规定一个类应该只有一个职责。

然而，单例模式至少有两个职责-它确保类只有一个实例并且包含业务逻辑。

在接下来的部分中，我们将讨论这种设计模式的其他一些缺陷。

### 3.1 全局状态

我们知道[全局状态](https://www.baeldung.com/cs/global-variables)被认为是一种不好的做法，因此应该避免。

虽然可能不太明显，但单例在我们的代码中引入了全局变量，但它们被封装在一个类中。

**由于它们是全局的，每个类都可以访问和使用它们。此外，如果它们不是不可变的，那么每个类都可以更改它们**。

假设我们在代码中的几个地方使用了Logger类，每个地方都可以访问和修改它的值。

现在，如果我们在使用它的一种方法中遇到问题并发现问题出在单例本身，我们需要检查整个代码库和使用它的每个方法来找出问题的影响。

这很快就会成为我们应用程序的瓶颈。

### 3.2 代码灵活性

其次，就软件开发而言，唯一可以确定的是，我们的代码将来可能会发生变化。

当项目处于开发的早期阶段时，我们可以假设某些类不会有超过一个实例，并使用单例设计模式来定义它们。

**然而，如果需求发生变化并且我们的假设被证明是错误的，我们就需要付出巨大的努力来重构我们的代码**。

让我们在工作示例中讨论上述问题。

我们假设只需要一个Logger类的实例，如果将来我们觉得一个文件不够用怎么办？

例如，我们可能需要为错误和信息消息分别创建单独的文件。此外，一个类的实例已经不够用了。接下来，为了使修改成为可能，我们需要重构整个代码库并移除单例，这将需要大量的工作。

**使用单例，我们的代码就会变得紧密耦合，灵活性也会降低**。

### 3.3 依赖隐藏

进一步来说，单例模式促进了隐藏的依赖关系。

**换句话说，当我们在其他类中使用它们时，我们隐藏了这些类依赖于单例实例的事实**。

让我们考虑一下sum()方法：

```java
public static int sum(int a, int b){
    Logger logger = Logger.getInstance();
    logger.log("A simple message");
    return a + b;
}
```

如果我们不直接查看sum()方法的实现，我们就无法知道它使用了Logger类。

我们没有像往常一样将依赖项作为参数传递给构造函数或方法。

### 3.4 多线程

其次，在多线程环境中，单例的实现可能比较棘手。

**主要问题是全局变量对我们代码中的所有线程都是可见的**，此外，每个线程都无法感知其他线程在同一个实例上进行的活动。

因此，我们最终会面临不同的问题，例如[竞争条件](https://www.baeldung.com/cs/race-conditions)和其他同步问题。

我们之前实现的Logger类在多线程环境下无法正常工作，我们的方法中没有任何内容可以阻止多个线程同时访问getInstance()方法。因此，我们最终可能会得到多个Logger类的实例。

让我们用[synchronized](https://www.baeldung.com/java-synchronized)关键字修改getInstance()方法：

```java
public static Logger getInstance() {
    synchronized (Logger.class) {
        if (instance == null) {
            instance = new Logger();
        }
    }
    return instance;
}
```

我们现在强制每个线程等待轮到自己，但是，我们应该意识到同步的开销很大。此外，我们还会给方法带来额外开销。

如果有必要，解决问题的方法之一是应用[双重检查锁](https://www.baeldung.com/java-singleton-double-checked-locking)机制：

```java
private static volatile Logger instance;

public static Logger getInstance() {
    if (instance == null) {
        synchronized (Logger.class) {
            if (instance == null) {
                instance = new Logger();
            }
        }
    }
    return instance;
}
```

**但是，[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm)允许访问部分构造的对象，这可能会导致程序出现意外的行为**。因此，需要在实例变量中添加[volatile](https://www.baeldung.com/java-volatile)关键字。

我们可能考虑的其他替代方案包括：

- 急切创建的实例，而不是惰性创建的实例
- [枚举](https://www.baeldung.com/a-guide-to-java-enums)单例
- Bill Pugh单例

### 3.5 测试

进一步说，在测试代码时，我们可以注意到单例的缺点。

**单元测试应该只测试我们代码的一小部分，并且不应该依赖于可能失败的其他服务，从而导致我们的测试也失败**。

让我们测试一下sum()方法：

```java
@Test
void givenTwoValues_whenSum_thenReturnCorrectResult() {
    SingletonDemo singletonDemo = new SingletonDemo();
    int result = singletonDemo.sum(12, 4);
    assertEquals(16, result);
}
```

即使我们的测试通过，它也会创建一个包含日志的文件，因为sum()方法使用了Logger类。

如果我们的Logger类出了问题，测试就会失败。那么，我们应该如何防止日志记录失败呢？

如果适用，一种解决方案是使用[Mockito](https://www.baeldung.com/java-mockito-singleton) Mock静态getInstance()方法：

```java
@Test
void givenMockedLogger_whenSum_thenReturnCorrectResult() {
    Logger logger = mock(Logger.class);

    try (MockedStatic<Logger> loggerMockedStatic = mockStatic(Logger.class)) {
        loggerMockedStatic.when(Logger::getInstance).thenReturn(logger);
        doNothing().when(logger).log(any());

        SingletonDemo singletonDemo = new SingletonDemo();
        int result = singletonDemo.sum(12, 4);
        Assertions.assertEquals(16, result);
    }
}
```

## 4. 单例模式的替代方案

最后，让我们讨论一些替代方案。

**如果只需要一个实例，我们可以使用依赖注入。换句话说，我们可以只创建一个实例，并在需要时将其作为参数传递**。这样，我们就能更好地了解方法或其他类正常运行所需的依赖关系。

此外，如果我们将来需要多个实例，我们可以更轻松地更改代码。

此外，我们可以将[工厂模式](https://www.baeldung.com/java-factory-pattern)用于长寿命对象。

## 5. 总结

在本文中，我们研究了单例设计模式的主要缺点。

总而言之，我们应该只在真正需要时才使用此模式,过度使用它会在实际上不需要单个实例的情况下引入不必要的限制。作为替代方案，我们可以简单地使用依赖注入并将对象作为参数传递。