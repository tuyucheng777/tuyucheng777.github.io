---
layout: post
title:  JUnit 5中@AutoClose扩展指南
category: unittest
copyright: unittest
excerpt: JUnit
---

## 1. 概述

在这个简短的教程中，我们将探索新的@AutoClose [JUnit 5](https://www.baeldung.com/junit-5)注解，它可以帮助我们处理测试执行后需要特定方法调用的类。

之后，我们将学习如何使用此扩展来简化我们的测试并从@AfterAll块中删除样板代码。

## 2. @AutoClose扩展

在测试中，有些情况下某些类需要在测试完成后执行特定操作。例如，当我们有实现[AutoCloseable](https://www.baeldung.com/java-try-with-resources#custom)接口的测试依赖项时，通常就是这种情况。为了演示目的，让我们创建自定义AutoCloseable类：

```java
class DummyAutoCloseableResource implements AutoCloseable {
   
    // logger
   
    private boolean open = true;

    @Override
    public void close() {
        LOGGER.info("Closing Dummy Resource");
        open = false;
    }
}
```

当我们完成测试运行时，我们使用@AfterAll块关闭资源：

```java
class AutoCloseableExtensionUnitTest {

    static DummyAutoCloseableResource resource = new DummyAutoCloseableResource();

    @AfterAll
    static void afterAll() {
        resource.close();
    }

    // tests
}
```

**但是，从JUnit 5.11版本开始，我们可以使用@AutoClose扩展来消除样板代码**。该扩展集成到JUnit 5框架中，因此我们不需要在类级别添加任何特殊注解。相反，我们可以只用@AutoClose标注字段：

```java
class AutoCloseableExtensionUnitTest {

    @AutoClose
    DummyAutoCloseableResource resource = new DummyAutoCloseableResource();

    // tests
}
```

我们可以看到，这也消除了将字段声明为static的限制。在这种情况下，执行每个测试、工厂或模板方法后都会关闭该字段。**此外，注解字段不一定必须实现AutoCloseable接口**。默认情况下，扩展会在注解字段内部查找名为“close”的方法，但我们可以自定义并指向其他函数。

**让我们考虑另一个用例，我们想要在完成资源处理后调用clear()方法**：

```java
class DummyClearableResource {
   
    // logger

    public void clear() {
        LOGGER.info("Clear Dummy Resource");
    }
}
```

在这种情况下，我们可以使用注解的值来指示在所有测试之后需要调用哪个方法：

```java
class AutoCloseableExtensionUnitTest {

    @AutoClose
    DummyAutoCloseableResource resource = new DummyAutoCloseableResource();

    @AutoClose("clear")
    DummyClearableResource clearResource = new DummyClearableResource();

    // tests
}
```

## 3. 总结

在这篇简短的文章中，我们讨论了新的@AutoClose扩展并将其用于实际示例，我们探索了它如何帮助我们保持测试简洁并管理需要关闭的资源。