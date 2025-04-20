---
layout: post
title:  在Java中仅执行一次方法
category: designpattern
copyright: designpattern
excerpt: Java
---

## 1. 概述

在本教程中，我们将探索仅执行一次方法的方法，这在多种情况下非常有用。例如，初始化[单例](https://www.baeldung.com/java-singleton)实例的方法，或执行一次性设置操作的方法。

我们将探索各种技术来确保某个方法仅被调用一次，这些技术包括使用布尔变量和synchronized关键字、AtomicBoolean以及静态初始化块。此外，某些单元测试框架(例如[JUnit](https://www.baeldung.com/junit-5)和[TestNG)](https://www.baeldung.com/testng)提供了注解，可以帮助确保某个方法仅被执行一次。

## 2. 使用Boolean和Synchronized

**我们的第一种方法是将布尔标志与synchronized关键字结合使用**，让我们看看如何实现它：

```java
class SynchronizedInitializer {

    static volatile boolean isInitialized = false;
    int callCount = 0;

    synchronized void initialize() {
        if (!isInitialized) {
            initializationLogic();
            isInitialized = true;
        }
    }

    private void initializationLogic() {
        callCount++;
    }
}
```

在此实现中，我们最初将isInitialized标志设置为false。首次调用initialize()方法时，它会检查该标志是否为false。如果是，则执行一次性初始化逻辑，并将标志设置为true。后续调用initialize()方法时，将发现该标志已为true，因此不会执行初始化逻辑。

**同步确保每次只有一个线程可以执行初始化方法**，这可以防止多个线程同时执行初始化逻辑，从而避免潜在的竞争条件，我们需要使用volatile关键字来确保每个线程都能读取更新后的布尔值。

我们可以通过以下测试来测试我们只执行了一次初始化：

```java
void givenSynchronizedInitializer_whenRepeatedlyCallingInitialize_thenCallCountIsOne() {
    SynchronizedInitializer synchronizedInitializer = new SynchronizedInitializer();
    assertEquals(0, synchronizedInitializer.callCount);

    synchronizedInitializer.initialize();
    synchronizedInitializer.initialize();
    synchronizedInitializer.initialize();

    assertEquals(1, synchronizedInitializer.callCount);
}
```

首先，我们创建一个SynchronizedInitializer实例，并断言callCount变量为0。在多次调用initialize()方法后，callCount会增加到1。

## 3. 使用AtomicBoolean

**另一种只执行一次方法是使用[AtomicBoolean](https://www.baeldung.com/java-atomic-variables)类型的[原子变量](https://www.baeldung.com/java-atomic-variables)**，我们来看一个实现示例：

```java
class AtomicBooleanInitializer {

    AtomicBoolean isInitialized = new AtomicBoolean(false);
    int callCount = 0;

    void initialize() {
        if (isInitialized.compareAndSet(false, true)) {
            initializationLogic();
        }
    }

    private void initializationLogic() {
        callCount++;
    }
}
```

在此实现中，isInitialized变量最初使用AtomicBoolean构造函数设置为false。首次调用initialize()方法时，我们会使用预期值false和新值true调用compareAndSet()方法。如果isInitialized的当前值为false，则该方法会将其设置为true并返回true。后续调用initialize()方法时，会发现isInitialized变量已经为true，因此不会执行初始化逻辑。

**使用AtomicBoolean可以确保compareAndSet()方法是原子操作，这意味着同一时刻只有一个线程可以修改isInitialized的值**。这样可以避免出现竞争条件，并确保initialize()方法是[线程安全](https://www.baeldung.com/java-thread-safety)的。

我们可以通过以下测试验证我们只执行了一次AtomicBooleanInitializer的initializationLogic()方法：

```java
void givenAtomicBooleanInitializer_whenRepeatedlyCallingInitialize_thenCallCountIsOne() {
    AtomicBooleanInitializer atomicBooleanInitializer = new AtomicBooleanInitializer();
    assertEquals(0, atomicBooleanInitializer.callCount);

    atomicBooleanInitializer.initialize();
    atomicBooleanInitializer.initialize();
    atomicBooleanInitializer.initialize();

    assertEquals(1, atomicBooleanInitializer.callCount);
}
```

这个测试与我们之前看到的测试非常相似。

## 4. 使用静态初始化

[静态初始化](https://www.baeldung.com/java-static-instance-initializer-blocks)是仅执行一次方法的另一种方法：

```java
final class StaticInitializer {

    public static int CALL_COUNT = 0;

    static {
        initializationLogic();
    }

    private static void initializationLogic() {
        CALL_COUNT++;
    }
}
```

**静态初始化块在类加载期间仅执行一次，它不需要额外的锁定**。

我们可以通过以下测试来测试我们只调用了一次StaticInitializer的初始化方法：

```java
void whenLoadingStaticInitializer_thenCallCountIsOne() {
    assertEquals(1, StaticInitializer.CALL_COUNT);
}
```

因为在类加载期间已经调用了静态初始化块，所以CALL_COUNT变量已经设置为1。

```java
void whenInitializingStaticInitializer_thenCallCountStaysOne() {
    StaticInitializer staticInitializer = new StaticInitializer();
    assertEquals(1, StaticInitializer.CALL_COUNT);
}
```

当创建StaticInitializer的新实例时，CALL_COUNT仍然为1，我们不能再次调用静态初始化块。

## 5. 使用单元测试框架

JUnit和TestNG提供了注解，用于仅运行一次设置方法。在JUnit中，使用@BeforeAll注解；在TestNG或更早的JUnit版本中，我们可以使用@BeforeClass注解来仅执行一次方法。

以下是JUnit设置方法的示例：

```java
@BeforeAll
static void setup() {
    log.info("@BeforeAll - executes once before all test methods in this class");
}
```

## 6. 总结

在本文中，我们学习了如何确保某个方法只执行一次。我们在不同的场景中都需要这样做，例如初始化数据库连接。

我们看到的方法使用了锁定、AtomicBoolean和静态初始化器，也可以使用单元测试框架来仅运行一次方法。