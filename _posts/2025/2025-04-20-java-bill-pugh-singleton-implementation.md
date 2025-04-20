---
layout: post
title:  Bill Pugh单例实现
category: designpattern
copyright: designpattern
excerpt: Bill Pugh单例
---

## 1. 概述

在本教程中，我们将讨论Bill Pugh的单例模式实现。单例模式有多种实现，其中，延迟加载单例模式和预先加载单例模式最为突出。此外，它们也支持同步和非同步版本。

Bill Pugh的单例模式实现支持延迟加载的单例对象，在接下来的章节中，我们将探索它的实现，并看看它如何解决其他实现所面临的挑战。

## 2. 单例实现的基本原理

**[单例模式](https://www.baeldung.com/java-singleton)是一种[创建型设计模式](https://www.baeldung.com/creational-design-patterns)，顾名思义，这种设计模式用于创建类的单个实例，该实例将在整个应用程序中使用**。它通常用于创建成本高且耗时的类，例如连接工厂、REST适配器、DAO等。

在继续之前我们先来了解一下Java类单例实现的基本原理：

- **私有构造函数以防止使用new运算符进行实例化**
- **最好使用名为getInstance()的公共静态方法，以返回类的单个实例**
- **私有静态变量，用于存储类的唯一实例**

此外，在多线程环境中限制类的单个实例，并将实例的初始化推迟到被引用时，可能会面临挑战。因此，这就是一种实现比其他实现更胜一筹的地方。考虑到这些挑战，我们将看看Bill Pugh的单例实现如何脱颖而出。

## 3. Bill Pugh Singleton实现

大多数情况下，Singleton实现面临以下一个或两个挑战：

- **预先加载**
- **同步造成的开销**

**Bill Pugh或Holder Singleton模式借助私有静态内部类解决了这两个问题**：

```java
public class BillPughSingleton {
    private BillPughSingleton() {
    }

    private static class SingletonHelper {
        private static final BillPughSingleton BILL_PUGH_SINGLETON_INSTANCE = new BillPughSingleton();
    }

    public static BillPughSingleton getInstance() {
        return SingletonHelper.BILL_PUGH_SINGLETON_INSTANCE;
    }
}
```

Java应用程序中的类加载器只会在内存中加载一次静态内部类SingletonHelper，即使多个线程调用getInstance()也是如此。值得注意的是，我们也没有使用同步方法，这消除了访问同步方法时锁定和释放对象的开销。因此，这种方法解决了其他实现所面临的缺陷。

现在，让我们看看它是如何工作的：

```java
@Test
void givenSynchronizedLazyLoadedImpl_whenCallgetInstance_thenReturnSingleton() {
    Set<BillPughSingleton> setHoldingSingletonObj = new HashSet<>();
    List<Future<BillPughSingleton>> futures = new ArrayList<>();

    ExecutorService executorService = Executors.newFixedThreadPool(10);
    Callable<BillPughSingleton> runnableTask = () -> {
        logger.info("run called for:" + Thread.currentThread().getName());
        return BillPughSingleton.getInstance();
    };

    int count = 0;
    while(count < 10) {
        futures.add(executorService.submit(runnableTask));
        count++;
    }
    futures.forEach(e -> {
        try {
            setHoldingSingletonObj.add(e.get());
        } catch (Exception ex) {
            throw new RuntimeException(ex);
        }
    });
    executorService.shutdown();
    assertEquals(1, setHoldingSingletonObj.size());
}
```

在上述方法中，多个线程并发调用getInstance()。然而，它总是返回同一个对象引用。

## 4. Bill Pugh与非同步实现

让我们在单线程和多线程环境中实现单例模式，在多线程环境中，我们将避免使用synchronized关键字。

### 4.1 延迟加载单例实现

根据前面描述的基本原则，我们来实现一个单例类：

```java
public class LazyLoadedSingleton {
    private static LazyLoadedSingleton lazyLoadedSingletonObj;

    private LazyLoadedSingleton() {
    }

    public static LazyLoadedSingleton getInstance() {
        if (null == lazyLoadedSingletonObj) {
            lazyLoadedSingletonObj = new LazyLoadedSingleton();
        }
        return lazyLoadedSingletonObj;
    }
}
```

**LazyLoadedSingleton对象仅在调用getInstance()方法时才会创建，这被称为惰性初始化。然而，当多个线程并发调用getInstance()方法时，由于脏读，会导致初始化失败。Bill Pugh的实现即使没有使用synchronized，也不会出现这种情况**。

让我们看看LazyLoadedSingleton类是否只创建一个对象：

```java
@Test
void givenLazyLoadedImpl_whenCallGetInstance_thenReturnSingleInstance() throws ClassNotFoundException {
    Class bs = Class.forName("cn.tuyucheng.taketoday.billpugh.LazyLoadedSingleton");
    assertThrows(IllegalAccessException.class, () -> bs.getDeclaredConstructor().newInstance());

    LazyLoadedSingleton lazyLoadedSingletonObj1 = LazyLoadedSingleton.getInstance();
    LazyLoadedSingleton lazyLoadedSingletonObj2 = LazyLoadedSingleton.getInstance();
    assertEquals(lazyLoadedSingletonObj1.hashCode(), lazyLoadedSingletonObj2.hashCode());
}
```

上述方法尝试借助[反射API](https://www.baeldung.com/java-reflection)并调用getInstance()方法来实例化LazyLoadedSingleton，然而，使用反射实例化会失败，并且getInstance()始终返回LazyLoadedSingleton类的单个实例。

### 4.2 预先加载单例实现

上一节讨论的实现仅适用于单线程环境，然而，对于多线程环境，我们可以考虑使用类级静态变量的另一种方法：

```java
public class EagerLoadedSingleton {
    private static final EagerLoadedSingleton EAGER_LOADED_SINGLETON = new EagerLoadedSingleton();

    private EagerLoadedSingleton() {
    }

    public static EagerLoadedSingleton getInstance() {
        return EAGER_LOADED_SINGLETON;
    }
}
```

**类级别变量EAGER_LOADED_SINGLETON是静态的，因此，应用程序启动时，即使不需要它也会立即加载。不过，正如前面所讨论的，Bill Pugh的实现在单线程和多线程环境中都支持延迟加载**。

让我们看看EagerLoadedSingleton类的实际作用：

```java
@Test
void givenEagerLoadedImpl_whenCallgetInstance_thenReturnSingleton() {
    Set<EagerLoadedSingleton> set = new HashSet<>();
    List<Future<EagerLoadedSingleton>> futures = new ArrayList<>();

    ExecutorService executorService = Executors.newFixedThreadPool(10);

    Callable<EagerLoadedSingleton> runnableTask = () -> {
        return EagerLoadedSingleton.getInstance();
    };

    int count = 0;
    while(count < 10) {
        futures.add(executorService.submit(runnableTask));
        count++;
    }

    futures.forEach(e -> {
        try {
            set.add(e.get());
        } catch (Exception ex) {
            throw new RuntimeException(ex);
        }
    });
    executorService.shutdown();

    assertEquals(1, set.size());
}
```

在上述方法中，多个线程调用runnableTask。然后，run()方法调用getInstance()来获取EagerLoadedSingleton的实例。然而，每次getInstance()都返回该对象的单个实例。

**上述代码在多线程环境中有效，但它表现出急切加载，这显然是一个缺点**。

## 5. Bill Pugh与同步单例实现

之前，我们在单线程环境中看到了LazyLoadedSingleton，让我们对其进行修改，使其在多线程环境中支持单例模式：

```java
public class SynchronizedLazyLoadedSingleton {
    private static SynchronizedLazyLoadedSingleton synchronizedLazyLoadedSingleton;

    private SynchronizedLazyLoadedSingleton() {
    }

    public static synchronized SynchronizedLazyLoadedSingleton getInstance() {
        if (null == synchronizedLazyLoadedSingleton) {
            synchronizedLazyLoadedSingleton = new SynchronizedLazyLoadedSingleton();
        }
        return synchronizedLazyLoadedSingleton;
    }
}
```

有趣的是，通过在getInstance()方法上使用[synchronized](https://www.baeldung.com/java-synchronized)关键字，我们可以限制线程并发访问它。我们可以使用[双重检查锁方法](https://www.baeldung.com/java-singleton-double-checked-locking)来实现更高性能的变体。

**然而，Bill Pugh的实现显然是赢家，因为它可以在多线程环境中使用，而没有同步的开销**。

让我们确认这在多线程环境中是否有效：

```java
@Test
void givenSynchronizedLazyLoadedImpl_whenCallgetInstance_thenReturnSingleton() {
    Set<SynchronizedLazyLoadedSingleton> setHoldingSingletonObj = new HashSet<>();
    List<Future<SynchronizedLazyLoadedSingleton>> futures = new ArrayList<>();

    ExecutorService executorService = Executors.newFixedThreadPool(10);
    Callable<SynchronizedLazyLoadedSingleton> runnableTask = () -> {
        logger.info("run called for:" + Thread.currentThread().getName());
        return SynchronizedLazyLoadedSingleton.getInstance();
    };

    int count = 0;
    while(count < 10) {
        futures.add(executorService.submit(runnableTask));
        count++;
    }
    futures.forEach(e -> {
        try {
            setHoldingSingletonObj.add(e.get());
        } catch (Exception ex) {
            throw new RuntimeException(ex);
        }
    });
    executorService.shutdown();
    assertEquals(1, setHoldingSingletonObj.size());
}
```

与EagerLoadedSingleton类似，SynchronizedLazyLoadedSingleton类在多线程设置中也返回单个对象，但这次程序以惰性加载的方式加载单例对象。然而，由于同步机制，它会带来一些开销。

## 6. 总结

在本文中，我们将Bill Pugh单例实现与其他流行的单例实现进行了比较。Bill Pugh单例实现性能更佳，并且支持延迟加载。因此，许多应用程序和库都广泛使用它。