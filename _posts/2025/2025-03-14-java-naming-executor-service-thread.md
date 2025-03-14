---
layout: post
title:  在Java中命名ExecutorService线程和线程池
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)提供了一种在Java中管理线程和执行并发任务的便捷方法，**使用ExecutorService时，为线程和线程池分配有意义的名称有助于改善线程的调试、监控和理解**。在本文中，我们将了解在Java的ExecutorService中命名线程和线程池的不同方法。

首先，我们将了解如何在ExecutorService中设置线程的默认名称。然后，我们将看到使用自定义ThreadFactory、Apache Commons的BasicThreadFactory和Guava库的ThreadFactoryBuilder来自定义线程名称的不同方法。

## 2. 命名线程

如果我们不使用ExecutorService，则可以轻松地[在Java中设置线程名称](https://www.baeldung.com/java-set-thread-name)。虽然**ExecutorService使用默认线程池和线程名称(例如“pool-1-thread-1”、“pool-1-thread-2”等)**，但可以为ExecutorService管理的线程指定自定义线程名称。

首先，我们来创建一个简单的程序来运行ExecutorService。稍后，我们将看到它如何显示默认线程和线程池名称：

```java
ExecutorService executorService = Executors.newFixedThreadPool(3);
for (int i = 0; i < 5; i++) {
    executorService.execute(() -> System.out.println(Thread.currentThread().getName()));
}
```

现在我们运行一下程序，可以看到打印出来的默认线程名：

```text
pool-1-thread-1
pool-1-thread-2
pool-1-thread-1
pool-1-thread-3
pool-1-thread-2
```

### 2.1 使用自定义ThreadFactory

在ExecutorService中，使用[ThreadFactory](https://download.java.net/java/early_access/jdk24/docs/api/java.base/java/util/concurrent/ThreadFactory.html)创建新线程，ExecutorService使用[Executors.defaultThreadFactory](https://download.java.net/java/early_access/jdk24/docs/api/java.base/java/util/concurrent/Executors.html#defaultThreadFactory())创建线程来执行任务。

**通过向ExecutorService提供不同的自定义ThreadFactory，我们可以改变线程的名称、优先级等**。

首先，让我们创建ThreadFactory自己的实现的MyThreadFactory。然后，我们将为使用MyThreadFactory创建的任何新线程创建一个自定义名称：

```java
public class MyThreadFactory implements ThreadFactory {
    private AtomicInteger threadNumber = new AtomicInteger(1);
    private String threadlNamePrefix = "";

    public MyThreadFactory(String threadlNamePrefix) {
        this.threadlNamePrefix = threadlNamePrefix;
    }

    public Thread newThread(Runnable runnable) {
        return new Thread(runnable, threadlNamePrefix + threadNumber.getAndIncrement());
    }
}
```

现在，我们将使用自定义工厂MyThreadFactory来设置线程名称并将其传递给ExecutorService：

```java
MyThreadFactory myThreadFactory = new MyThreadFactory("MyCustomThread-");
ExecutorService executorService = Executors.newFixedThreadPool(3, myThreadFactory);
for (int i = 0; i < 5; i++) {
    executorService.execute(() -> System.out.println(Thread.currentThread().getName()));
}
```

最后，当我们运行程序时，我们可以看到ExecutorService的线程打印的自定义线程名称：

```text
MyCustomThread-1
MyCustomThread-2
MyCustomThread-2
MyCustomThread-3
MyCustomThread-1
```

### 2.2 使用Apache Commons的BasicThreadFactory

**commons-lang3中的BasicThreadFactory实现了ThreadFactory接口，为线程提供了配置选项，有助于设置线程名称**。

首先，让我们将[commons-lang3](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)依赖项添加到我们的项目中：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

接下来，我们使用自定义名称创建BasicThreadFactory。之后，我们使用工厂创建ExecutorService：

```java
BasicThreadFactory factory = new BasicThreadFactory.Builder()
    .namingPattern("MyCustomThread-%d").priority(Thread.MAX_PRIORITY).build();
ExecutorService executorService = Executors.newFixedThreadPool(3, factory);
for (int i = 0; i < 5; i++) {
    executorService.execute(() -> System.out.println(Thread.currentThread().getName()));
}
```

在这里，我们可以看到namingPattern()方法采用线程名称的名称模式。

最后，让我们运行程序来查看打印的自定义线程名称：

```text
MyCustomThread-1
MyCustomThread-2
MyCustomThread-2
MyCustomThread-3
MyCustomThread-1
```

### 2.3 使用Guava中的ThreadFactoryBuilder

**Guava的ThreadFactoryBuilder也提供了自定义其创建的线程的选项**。

首先，让我们将[Guava](https://mvnrepository.com/artifact/com.google.guava/guava)依赖项添加到我们的项目中：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>33.2.0-jre</version>
</dependency>
```

接下来，我们使用自定义名称创建ThreadFactory，并将其传递给ExecutorService：

```java
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
  .setNameFormat("MyCustomThread-%d").build();
ExecutorService executorService = Executors.newFixedThreadPool(3, namedThreadFactory);
for (int i = 0; i < 5; i++) {
    executorService.execute(() -> System.out.println(Thread.currentThread().getName()));
}
```

在这里，我们可以看到setNameFormat()采用线程名称的名称模式。

最后，当我们运行程序时，我们可以看到打印的自定义线程名称：

```text
MyCustomThread-0
MyCustomThread-1
MyCustomThread-2
MyCustomThread-2
MyCustomThread-1
```

这些是我们在使用Java中的ExecutorService时命名线程的一些方法，可根据应用程序的要求提供灵活性。

## 3. 总结

在本文中，我们了解了Java的ExecutorService中命名线程和线程池的不同方式。

首先，我们了解了如何设置默认名称。然后，我们使用自定义的ThreadFactory以及Apache Commons和Guava等不同API的ThreadFactory自定义了线程名称。