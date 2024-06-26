---
layout: post
title:  二进制信号量与可重入锁
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本教程中，我们将探讨二进制信号量和可重入锁。此外，我们会将它们相互比较，看看哪一个最适合常见情况。

## 2. 什么是二进制信号量？

二进制[信号量](https://www.baeldung.com/java-semaphore)提供了访问单个资源的信号机制。换句话说，**二进制信号量提供了一种互斥机制，一次只允许一个线程访问临界区**。

为此，它只保留一个可供访问的许可证。因此，**二进制信号量只有两种状态：一个可用许可或零个可用许可**。

让我们讨论使用Java中可用的[Semaphore](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/Semaphore.html)类的[二进制信号量的简单实现](https://www.baeldung.com/java-semaphore#mutex)：

```java
@Test
void givenBinarySemaphore_whenAcquireAndRelease_thenCheckAvailablePermits() {
    Semaphore binarySemaphore = new Semaphore(1);
    try {
        binarySemaphore.acquire();
        assertEquals(0, binarySemaphore.availablePermits());
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    } finally {
        binarySemaphore.release();
        assertEquals(1, binarySemaphore.availablePermits());
    }
}
```

在这里，我们可以观察到，调用acquire方法将可用许可减1。类似地，调用release方法将可用许可证加1。

此外，Semaphore构造函数还提供了fairness参数。当设置为true时，fairness参数确保请求线程获得许可的顺序(基于它们的等待时间)：

```java
Semaphore binarySemaphore = new Semaphore(1, true);
```

## 3. 什么是可重入锁？

[可重入锁是一种互斥机制](https://www.baeldung.com/java-concurrent-locks#lock-implementations)，**它允许线程重新进入资源上的锁(多次)，而不会出现死锁情况**。

进入锁的线程每次都会将持有计数加1。同样，当请求解锁时，持有计数会减少。因此，**资源被锁定直到计数器归零**。

例如，让我们看一下使用Java中可用的[ReentrantLock](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/concurrent/locks/ReentrantLock.html)类的简单实现：

```java
@Test
void givenReentrantLock_whenLockAndUnlock_thenCheckHoldCountAndIsLocked() {
    ReentrantLock reentrantLock = new ReentrantLock();
    try {
        reentrantLock.lock();
        assertEquals(1, reentrantLock.getHoldCount());
        assertTrue(reentrantLock.isLocked());
    } finally {
        reentrantLock.unlock();
        assertEquals(0, reentrantLock.getHoldCount());
        assertFalse(reentrantLock.isLocked());
    }
}
```

在这里，lock方法将持有计数增加1并锁定资源。类似地，unlock方法会减少持有计数并在持有计数为零时解锁资源。

当线程重新进入锁时，它必须请求解锁相同的次数才能释放资源：

```java
@Test
void givenReentrantLock_whenLockMultipleTimes_thenUnlockMultipleTimesToRelease() {
    ReentrantLock reentrantLock = new ReentrantLock();
    try {
        reentrantLock.lock();
        reentrantLock.lock();
        assertEquals(2, reentrantLock.getHoldCount());
        assertTrue(reentrantLock.isLocked());
    } finally {
        reentrantLock.unlock();
        assertEquals(1, reentrantLock.getHoldCount());
        assertTrue(reentrantLock.isLocked());

        reentrantLock.unlock();
        assertEquals(0, reentrantLock.getHoldCount());
        assertFalse(reentrantLock.isLocked());
    }
}
```

与Semaphore类类似，ReentrantLock类也支持fairness参数：

```java
ReentrantLock reentrantLock = new ReentrantLock(true);
```

## 4. 二进制信号量与可重入锁

### 4.1 机制

**二进制信号量是一种信号机制**，而可重入锁是一种锁定机制。

### 4.2 所有者

没有线程是二进制信号量的所有者。但是，**最后一个成功锁定资源的线程是可重入锁的所有者**。

### 4.3 本质

二进制信号量本质上是不可重入的，这意味着同一个线程不能重新获取临界区，否则会导致死锁情况。

另一方面，可重入锁本质上允许同一线程多次重新进入锁。

### 4.4 灵活性

**二进制信号量通过允许锁定机制和死锁恢复的自定义实现来提供更高级的同步机制**。因此，它为开发人员提供了更多控制权。

但是，**可重入锁是一种具有固定锁定机制的低级同步**。

### 4.5 修改

二进制信号量支持诸如等待和信号之类的操作(在Java的Semaphore类的情况下为acquire和release)，以允许任何线程修改可用的许可。

另一方面，只有锁定/解锁资源的同一线程才能修改可重入锁。

### 4.6 死锁恢复

**二进制信号量提供了一种非所有权释放机制**。因此，任何线程都可以释放二进制信号量的死锁恢复许可。

相反，在可重入锁的情况下，很难实现死锁恢复。例如，如果可重入锁的所有者线程进入休眠或无限等待状态，则无法释放资源，从而导致死锁情况。

## 5. 总结

在这篇简短的文章中，我们探讨了二进制信号量和可重入锁。

首先，我们讨论了二进制信号量和可重入锁的基本定义，以及Java中的基本实现。然后，我们根据机制、所有权和灵活性等几个参数将它们相互比较。

我们可以得出的总结是，**二进制信号量为互斥提供了一种基于非所有权的信号机制**。同时，它还可以进一步扩展以提供锁定能力，并易于死锁恢复。

另一方面，**可重入锁提供了具有基于所有者的锁定功能的可重入互斥**，并且可用作简单的互斥锁。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-advanced-4)上获得。