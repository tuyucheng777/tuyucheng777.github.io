---
layout: post
title:  java.util.concurrent.Locks指南
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

简单地说，锁是一种比标准同步块更灵活、更复杂的线程同步机制。

Lock接口自Java 1.5开始出现，它定义在java.util.concurrent.lock包中，提供广泛的锁定操作。

在本教程中，我们将探索Lock接口的不同实现及其应用。

## 2. Lock和同步块的区别

使用同步块和使用Lock API之间存在一些差异：

- **同步块完全包含在方法中**，而我们可以在单独的方法中使用Lock API lock()和unlock()操作。
- 同步块不支持公平性，任何线程都可以在释放后获取锁，并且无法指定偏好。**我们可以通过指定fairness属性在Lock API中实现公平性**，它确保等待时间最长的线程可以访问锁。
- 如果线程无法访问同步块，则它会被阻塞。**Lock API提供了tryLock()方法，只有当锁可用且未被其他线程持有时，线程才会获取锁**，这减少了等待锁的线程的阻塞时间。
- 处于“等待”状态以获取同步块访问权限的线程无法被中断。**Lock API提供了一种方法lockInterruptibly()，可用于在线程等待锁时中断该线程**。

## 3. Lock API

我们来看一下Lock接口中的方法：

- **void lock()**：如果可用，则获取锁。如果锁不可用，则线程将被阻塞，直到锁被释放。
- **void lockInterruptibly()**：这与lock()类似，但它允许中断被阻塞的线程并通过抛出java.lang.InterruptedException恢复执行。
- **boolean tryLock()**：这是lock()方法的非阻塞版本，它尝试立即获取锁，如果锁定成功则返回true。
- **boolean tryLock(long timeout, TimeUnit timeUnit)**：这与tryLock()类似，不同之处在于它会等待给定的超时时间，然后放弃尝试获取Lock。
- **void unlock()**：释放Lock实例。

锁定的实例应始终处于解锁状态以避免出现死锁情况。

建议使用锁的代码块应该包含try/catch和finally块：

```java
Lock lock = ...; 
lock.lock();
try {
    // access to the shared resource
} finally {
    lock.unlock();
}
```

除了Lock接口之外，我们还有一个ReadWriteLock接口，它维护一对锁，一个用于只读操作，一个用于写操作。只要没有写操作，读锁可以同时被多个线程持有。

ReadWriteLock声明了获取读锁或写锁的方法：

- **Lock readLock()**返回用于读取的锁。
- **Lock writeLock()**返回用于写入的锁。

## 4. 锁的实现

### 4.1 ReentrantLock

ReentrantLock类实现了Lock接口，它提供与使用同步方法和语句访问的隐式监视器锁相同的并发性和内存语义，并具有扩展功能。

让我们看看如何使用ReentrantLock进行同步：

```java
public class SharedObjectWithLock {
    //...
    ReentrantLock lock = new ReentrantLock();
    int counter = 0;

    public void perform() {
        lock.lock();
        try {
            // Critical section here
            count++;
        } finally {
            lock.unlock();
        }
    }
    //...
}
```

我们需要确保在try-finally块中包装lock()和unlock()调用，以避免死锁情况。

让我们看看tryLock()是如何工作的：

```java
public void performTryLock(){
    //...
    boolean isLockAcquired = lock.tryLock(1, TimeUnit.SECONDS);
    
    if(isLockAcquired) {
        try {
            //Critical section here
        } finally {
            lock.unlock();
        }
    }
    //...
}
```

在这种情况下，调用tryLock()的线程将等待一秒钟，如果锁不可用，则放弃等待。

### 4.2 ReentrantReadWriteLock

ReentrantReadWriteLock类实现了ReadWriteLock接口。

我们来看一下线程获取ReadLock或者WriteLock的规则：

- **读锁**：如果没有线程获取写锁或请求写锁，则多个线程可以获取读锁。
- **写锁**：如果没有线程正在读取或写入，则只有一个线程可以获取写锁。

让我们看看如何使用ReadWriteLock：

```java
public class SynchronizedHashMapWithReadWriteLock {

    Map<String,String> syncHashMap = new HashMap<>();
    ReadWriteLock lock = new ReentrantReadWriteLock();
    // ...
    Lock writeLock = lock.writeLock();

    public void put(String key, String value) {
        try {
            writeLock.lock();
            syncHashMap.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }
    // ...
    public String remove(String key){
        try {
            writeLock.lock();
            return syncHashMap.remove(key);
        } finally {
            writeLock.unlock();
        }
    }
    //...
}
```

对于这两种写入方法，我们都需要用写锁包围临界区-只有一个线程可以访问它：

```java
Lock readLock = lock.readLock();
//...
public String get(String key){
    try {
        readLock.lock();
        return syncHashMap.get(key);
    } finally {
        readLock.unlock();
    }
}

public boolean containsKey(String key) {
    try {
        readLock.lock();
        return syncHashMap.containsKey(key);
    } finally {
        readLock.unlock();
    }
}
```

对于这两种读取方法，我们都需要用读取锁包围临界区。如果没有正在进行的写入操作，则多个线程可以访问此部分。

### 4.3 StampedLock

StampedLock是Java 8中引入的，它也支持读锁和写锁。

但是，锁获取方法会返回一个用于释放锁或检查锁是否仍然有效的戳记：

```java
public class StampedLockDemo {
    Map<String, String> map = new HashMap<>();
    private StampedLock lock = new StampedLock();

    public void put(String key, String value) {
        long stamp = lock.writeLock();
        try {
            map.put(key, value);
        } finally {
            lock.unlockWrite(stamp);
        }
    }

    public String get(String key) throws InterruptedException {
        long stamp = lock.readLock();
        try {
            return map.get(key);
        } finally {
            lock.unlockRead(stamp);
        }
    }
}
```

StampedLock提供的另一个功能是乐观锁定，大多数情况下，读取操作不需要等待写入操作完成，因此不需要完整的读取锁定。

相反，我们可以升级到读锁：

```java
public String readWithOptimisticLock(String key) {
    long stamp = lock.tryOptimisticRead();
    String value = map.get(key);

    if (!lock.validate(stamp)) {
        stamp = lock.readLock();
        try {
            return map.get(key);
        } finally {
            lock.unlock(stamp);
        }
    }
    return value;
}
```

## 5. 使用Condition

Condition类提供了使线程在执行临界区时等待某些条件发生的功能。

当线程获取了对临界区的访问权限但没有执行其操作的必要条件时，就会发生这种情况。例如，读取线程可以访问共享队列的锁，但该队列仍然没有任何数据可供使用。

传统上，Java提供wait()、notify()和notifyAll()方法用于线程间通信。

Condition有类似的机制，但是我们也可以指定多个条件：

```java
public class ReentrantLockWithCondition {

    Stack<String> stack = new Stack<>();
    int CAPACITY = 5;

    ReentrantLock lock = new ReentrantLock();
    Condition stackEmptyCondition = lock.newCondition();
    Condition stackFullCondition = lock.newCondition();

    public void pushToStack(String item) {
        try {
            lock.lock();
            while (stack.size() == CAPACITY) {
                stackFullCondition.await();
            }
            stack.push(item);
            stackEmptyCondition.signalAll();
        } finally {
            lock.unlock();
        }
    }

    public String popFromStack() {
        try {
            lock.lock();
            while (stack.size() == 0) {
                stackEmptyCondition.await();
            }
            return stack.pop();
        } finally {
            stackFullCondition.signalAll();
            lock.unlock();
        }
    }
}
```

## 6. 总结

在本文中，我们看到了Lock接口和新引入的StampedLock类的不同实现。

我们还探讨了如何利用Condition类来处理多种条件。