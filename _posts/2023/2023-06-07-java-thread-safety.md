---
layout: post
title:  什么是线程安全以及如何实现它？
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

Java支持开箱即用的多线程。这意味着，通过在单独的工作线程中并发运行字节码，[JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk)能够提高应用程序的性能。

尽管多线程是一个强大的功能，但它是有代价的。在多线程环境中，我们需要以线程安全的方式编写实现。这意味着不同的线程可以访问相同的资源，而不会暴露错误行为或产生不可预测的结果。这种编程方法称为“线程安全”。

在本教程中，我们将探讨实现这一目标的不同方法。

## 2. 无状态实现

在大多数情况下，多线程应用程序中的错误是多个线程之间错误共享状态的结果。

因此，我们将研究的第一种方法是**使用无状态实现**来实现线程安全。

为了更好地理解这种方法，让我们考虑一个简单的工具类，它包含一个静态方法，可以计算数字的阶乘：

```java
public class MathUtils {

    public static BigInteger factorial(int number) {
        BigInteger f = new BigInteger("1");
        for (int i = 2; i <= number; i++)
            f = f.multiply(BigInteger.valueOf(i));
        return f;
    }
}
```

**factorial()方法是一个无状态的确定性函数**，即给定一个特定的输入，它总是产生相同的输出。

该方法**既不依赖于外部状态，也完全不维护状态**。因此，它被认为是线程安全的，可以被多个线程同时安全地调用。

所有线程都可以安全地调用factorial()方法，并且可以得到预期的结果，而不会相互干扰，也不会改变该方法为其他线程生成的输出。

因此，**无状态实现是实现线程安全的最简单方法**。

## 3. 不可变实现

**如果我们需要在不同的线程之间共享状态，我们可以通过使它们不可变来创建线程安全类**。

不可变性是一个强大的、与语言无关的概念，在Java中很容易实现。

简单地说，**当类实例的内部状态(也称为属性，字段)在构造后无法修改时，它是不可变的**。

在Java中创建不可变类的最简单方法是声明所有字段为private和final，并且不提供setter方法：

```java
public class MessageService {
    private final String message;

    public MessageService(String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
}
```

在JDK 17中，我们也可以使用另一种Java类型Record来创建不可变类：

```java
public record MessageService(String message) {

    public String getMessage() {
        return message;
    }
}
```

MessageService对象实际上是不可变的，因为它的状态在构造后无法更改。所以，它是线程安全的。

此外，如果MessageService实际上是可变的，但多个线程只能对它进行只读访问，那么它也是线程安全的。

正如我们所见，**不可变性只是实现线程安全的另一种方式**。

## 4. ThreadLocal

在面向对象编程(OOP)中，对象实际上需要通过字段来维护状态，并通过一个或多个方法实现行为。

如果我们真的需要维护状态，**我们可以创建线程安全的类，通过将它们的字段设为线程本地(thread-local)，在线程之间不共享状态**。

通过简单地在[Thread](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Thread.html)类中定义私有字段，我们可以轻松创建线程本地的字段。

例如，我们可以定义一个存储整数数组的线程类：

```java
public class ThreadA extends Thread {
    private final List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

    @Override
    public void run() {
        numbers.forEach(System.out::println);
    }
}
```

同时，另一个可能存储一个字符串数组：

```java
public class ThreadB extends Thread {
    private final List<String> letters = Arrays.asList("a", "b", "c", "d", "e", "f");

    @Override
    public void run() {
        letters.forEach(System.out::println);
    }
}
```

**在这两种实现中，类都有自己的状态，但不会与其他线程共享。因此，这些类是线程安全的**。

类似地，我们可以通过将[ThreadLocal](https://www.baeldung.com/java-threadlocal)实例分配给字段来创建线程本地字段。

让我们考虑以下StateHolder类：

```java
public class StateHolder {
    private final String state;

    public StateHolder(String state) {
        this.state = state;
    }

    public String getState() {
        return state;
    }
}
```

我们可以很容易地使其成为线程本地变量：

```java
public class ThreadState {
    public static final ThreadLocal<StateHolder> statePerThread = ThreadLocal.withInitial(
            () -> new StateHolder("active")
    );

    public static StateHolder getState() {
        return statePerThread.get();
    }
}
```

线程本地字段与普通类字段非常相似，不同之处在于，通过setter/getter访问它们的每个线程都会获得该字段的独立初始化副本，这样每个线程都有自己的状态。

## 5. 同步集合

通过使用[集合框架](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html)中包含的一组同步包装器，我们可以轻松创建线程安全的集合。

例如，我们可以使用其中一个[同步包装器](https://www.baeldung.com/java-synchronized-collections)来创建线程安全的集合：

```java
Collection<Integer> syncCollection = Collections.synchronizedCollection(new ArrayList<>());
Thread thread1 = new Thread(() -> syncCollection.addAll(Arrays.asList(1, 2, 3, 4, 5, 6)));
Thread thread2 = new Thread(() -> syncCollection.addAll(Arrays.asList(7, 8, 9, 10, 11, 12)));
thread1.start();
thread2.start();
```

记住，同步集合在每个方法中都使用内部锁(稍后我们将介绍内部锁)。

**这意味着一次只能由一个线程访问这些方法，而其他线程将被阻塞，直到第一个线程解锁该方法**。

因此，由于同步访问的底层逻辑，同步集合在性能上会受到影响。

## 6. 并发集合

作为同步集合的替代方案，我们还可以使用并发集合来创建线程安全的集合。

Java提供了[java.util.concurrent](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/package-summary.html)包，其中包含多个并发集合，例如[ConcurrentHashMap](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html)：

```java
Map<String,String> concurrentMap = new ConcurrentHashMap<>();
concurrentMap.put("1", "one");
concurrentMap.put("2", "two");
concurrentMap.put("3", "three");
```

与同步集合不同，**并发集合通过将数据划分为段来实现线程安全**。例如，在ConcurrentHashMap中，多个线程可以获取不同map段上的锁，因此多个线程可以同时访问该map。

**由于并发线程访问的固有优势，并发集合的性能比同步集合高得多**。

值得一提的是，**同步和并发集合只会使集合本身具有线程安全性，而不会使其内容安全**。

## 7. 原子对象

还可以使用Java提供的一组[原子类](https://www.baeldung.com/java-atomic-variables)来实现线程安全，包括[AtomicInteger](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicInteger.html)、[AtomicLong](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicLong.html)、[AtomicBoolean](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicBoolean.html)和[AtomicReference](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/atomic/AtomicReference.html)。

**原子类允许我们在不使用同步的情况下执行线程安全的原子操作**，原子操作在单个机器级别的操作中执行。

为了理解它解决的问题，让我们看看下面的Counter类：

```java
public class Counter {
    private int counter = 0;

    public void incrementCounter() {
        counter += 1;
    }

    public int getCounter() {
        return counter;
    }
}
```

**假设在**[竞争条件](https://www.baeldung.com/cs/race-conditions)**下，两个线程同时访问incrementCounter()方法**。

理论上，counter字段的最终值应该为2。但是我们无法确定结果，因为线程在同一时间执行相同的代码块，并且+=操作不是原子的。

我们可以通过使用AtomicInteger对象创建Counter类的线程安全实现：

```java
public class AtomicCounter {
    private final AtomicInteger counter = new AtomicInteger(0);

    public AtomicCounter() {
    }

    public void incrementCounter() {
        counter.incrementAndGet();
    }

    public int getCounter() {
        return counter.get();
    }
}
```

**这是线程安全的，虽然++需要多个操作，但incrementAndGet()是原子的**。

## 8. 同步方法

早期的方法非常适用于集合和原始数据类型，但我们有时需要更多的控制。

因此，我们可以用来实现线程安全的另一种常见方法是实现同步方法。

简单地说，**一次只有一个线程可以访问一个同步方法，同时阻止其他线程访问该方法**。其他线程将保持阻塞状态，直到第一个线程完成或该方法抛出异常。

我们可以通过另一种方式创建incrementCounter()的线程安全版本，方法是将其设置为同步方法：

```java
public class Counter {
    private int counter;

    public Counter() {
        this.counter = 0;
    }

    public synchronized void incrementCounter() {
        counter += 1;
    }

    public int getCounter() {
        return counter;
    }
}
```

通过在方法签名前加上[synchronized](https://www.baeldung.com/java-synchronized)关键字，我们声明了一个同步方法。

由于一次只有一个线程可以访问同步方法，因此一个线程将执行incrementCounter()方法，而其他线程将执行同样的操作。不会出现任何重叠执行。

**同步方法依赖于“内部锁”或“监视器锁”的使用**，内部锁是与特定类实例关联的隐式内部实体。

在多线程上下文中，监视器monitor一词只是对锁在关联对象上执行的角色的引用，因为它强制对一组指定的方法或语句进行独占访问。

**当一个线程调用一个同步方法时，它会获取内部锁**。在线程完成方法的执行后，它会释放锁，这允许其他线程获取锁并访问该方法。

我们可以在实例方法、静态方法和语句(同步语句块)中实现同步。

## 9. 同步语句块

有时，如果我们只需要使方法的一部分线程安全，那么同步整个方法可能不是很有必要。

为了举例说明这个用例，让我们重构incrementCounter()方法：

```java
public void incrementCounter() {
    // additional unsync operations
    synchronized(this) {
        counter += 1; 
    }
}
```

这个例子很简单，它展示了如何创建同步语句块。假设该方法现在执行一些不需要同步的额外操作，我们只通过将相关状态修改部分包装在同步块中来同步它。

与同步方法不同，同步语句块必须指定提供内部锁的对象，通常是[this](https://www.baeldung.com/java-this)引用。

**同步的成本很高，因此使用此方法，我们只能同步方法的关键部分**。

### 9.1 其他对象作为锁

我们可以通过利用另一个对象作为监视锁(而不是this)来稍微改进Counter类的线程安全实现。

这不仅可以在多线程环境中提供对共享资源的协调访问，**还可以使用外部实体强制执行对资源的独占访问**：

```java
public class ObjectLockCounter {
    private final Object lock = new Object();
    private int counter;

    public ObjectLockCounter() {
        this.counter = 0;
    }

    public void incrementCounter() {
        synchronized (lock) {
            counter += 1;
        }
    }

    public int getCounter() {
        synchronized (lock) {
            return counter;
        }
    }
}
```

我们使用一个普通的[Object](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/Object.html)实例来强制互斥，这种实现稍微好一点，因为它在锁级别提高了安全性。

**当将this用于内部锁时，攻击者可能会通过获取内部锁并触发拒绝服务(denial of service - DoS)条件而导致**[死锁](https://www.baeldung.com/cs/deadlock-livelock-starvation)。

相反，当使用其他对象时，**无法从外部访问该私有实体**。这使得攻击者更难获得锁从而导致死锁。

### 9.2 注意事项

即使我们可以使用任何Java对象作为内部锁，我们也应该避免使用字符串：

```java
public class Class1 {
    private static final String LOCK = "Lock";

    // uses the LOCK as the intrinsic lock
}

public class Class2 {
    private static final String LOCK = "Lock";

    // uses the LOCK as the intrinsic lock
}
```

乍一看，这两个类似乎使用两个不同的对象作为它们的锁。然而，**由于**[字符串的特殊性](https://www.baeldung.com/string/intern)，**这两个“Lock”值实际上可能引用**[字符串池](https://www.baeldung.com/java-string-pool)**中的同一个对象**。也就是说，Class1和Class2共享同一个锁。

反过来，这可能会导致并发上下文中的一些意外行为。

除了字符串，**我们应该避免使用任何**[可缓存或可重用](https://mail.openjdk.java.net/pipermail/valhalla-spec-observers/2020-February/001199.html)**的对象作为内部锁**。例如，[Integer.valueOf()](https://github.com/openjdk/jdk/blob/8c647801fce4d6efcb3780570192973d16e4e6dc/src/java.base/share/classes/java/lang/Integer.java#L1062)方法会缓存较小的数字。因此，即使在不同的类中，调用Integer.valueOf(1)也会返回相同的对象。

## 10. volatile

同步方法和同步代码块对于解决线程之间的可变可见性问题非常方便。即便如此，常规类字段的值可能会被CPU缓存。因此，对特定字段的后续更新，即使它们是同步的，也可能对其他线程不可见。

为了防止这种情况，我们可以使用[volatile](https://www.baeldung.com/java-volatile)关键字：

```java
public class Counter {
    private volatile int counter;
    // ...
}
```

**使用volatile关键字，我们指示JVM和编译器将counter变量存储在主内存中**。这样，我们可以确保每次JVM读取counter变量的值时，它实际上都会从主内存中读取，而不是从CPU缓存中读取。同样，JVM每次写入counter变量时，该值都会写入主内存。

此外，**使用volatile变量可以确保对给定线程可见的所有变量也将从主内存中读取**。

考虑以下示例：

```java
public class User {
    private String name;
    private volatile int age;
    // standard constructors / getters
}
```

在这种情况下，JVM每次将volatile变量age写入主存时，它也会将非volatile变量name写入主存。这可以确保两个变量的最新值都存储在主内存中，因此对变量的后续更新将自动对其他线程可见。

类似地，如果一个线程读取一个volatile变量的值，那么线程可见的所有变量也将从主内存中读取。

**volatile变量提供的这种扩展保证称为**[完全volatile可见性保证](http://tutorials.jenkov.com/java-concurrency/volatile.html)。

## 11. 可重入锁

Java提供了一组改进的[Lock](https://www.baeldung.com/java-concurrent-locks)实现，其行为比上面讨论的内部锁稍微复杂一些。

**对于内部锁，锁的获取模型相当严格**：一个线程获取锁，然后执行一个方法或代码块，最后释放锁，以便其他线程可以获取锁并访问该方法。

没有一种底层机制检查排队的线程，并为等待时间最长的线程提供优先访问权。

ReentrantLock实例允许我们做到这一点，**防止排队的线程遭受某些类型的**[资源匮乏](https://en.wikipedia.org/wiki/Starvation_(computer_science))：

```java
public class ReentrantLockCounter {
    private final ReentrantLock lock = new ReentrantLock(true);
    private int counter;

    public ReentrantLockCounter() {
        this.counter = 0;
    }

    public void incrementCounter() {
        lock.lock();
        try {
            counter += 1;
        } finally {
            lock.unlock();
        }
    }

    public int getCounter() {
        return counter;
    }
}
```

ReentrantLock构造函数接收可选的fair布尔参数。当该参数设置为true，且多个线程试图获取锁时，**JVM将优先考虑等待时间最长的线程，并授予对锁的访问权**。

## 12. 读写锁

我们可以用来实现线程安全的另一个强大机制是使用[ReadWriteLock](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/locks/ReadWriteLock.html)实现。

读写锁实际上使用一对关联的锁，一个用于只读操作，另一个用于写操作。

因此，**只要没有线程写入资源，就有可能有多个线程读取资源。此外，写入资源的线程将阻止其他线程读取它**。

下面是使用读写锁的例子：

```java
public class ReentrantReadWriteLockCounter {
    private final ReentrantReadWriteLock readWriteLock = new ReentrantReadWriteLock();
    private final Lock readLock = readWriteLock.readLock();
    private final Lock writeLock = readWriteLock.writeLock();
    private int counter;

    public ReentrantReadWriteLockCounter(int counter) {
        this.counter = counter;
    }

    public void incrementCounter() {
        writeLock.lock();
        try {
            counter += 1;
        } finally {
            writeLock.unlock();
        }
    }

    public int getCounter() {
        readLock.lock();
        try {
            return counter;
        } finally {
            readLock.unlock();
        }
    }
}
```

## 13. 总结

在本文中，**我们学习了Java中的线程安全，并深入研究了实现线程安全的不同方法**。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-basic-1)上获得。