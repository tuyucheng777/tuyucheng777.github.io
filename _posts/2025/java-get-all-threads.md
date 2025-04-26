---
layout: post
title:  获取所有正在运行的JVM线程
category: java-jvm
copyright: java-jvm
excerpt: JVM
---

## 1. 概述

在这个简短的教程中，我们将学习如何获取当前JVM中所有正在运行的线程，包括未由我们的类启动的线程。

## 2. 使用Thread类

Thread类的getAllStackTrace()方法返回所有正在运行的线程的堆栈跟踪，它返回一个Map，其键是Thread对象，因此我们可以获取该键集，并简单地循环遍历其元素以获取有关线程的信息。

让我们使用[printf()](https://www.baeldung.com/java-printstream-printf)方法使输出更具可读性：

```java
Set<Thread> threads = Thread.getAllStackTraces().keySet();
System.out.printf("%-15s \t %-15s \t %-15s \t %s\n", "Name", "State", "Priority", "isDaemon");
for (Thread t : threads) {
    System.out.printf("%-15s \t %-15s \t %-15d \t %s\n", t.getName(), t.getState(), t.getPriority(), t.isDaemon());
}
```

输出将如下所示：

```text
Name            	 State           	 Priority        	 isDaemon
main            	 RUNNABLE        	 5               	 false
Signal Dispatcher 	 RUNNABLE        	 9               	 true
Finalizer       	 WAITING         	 8               	 true
Reference Handler 	 WAITING         	 10              	 true
```

我们看到，除了运行主程序的main线程之外，还有另外3个线程，不同Java版本的结果可能会有所不同。

让我们进一步了解一下这些其他线程：

- Signal Dispatcher：此线程处理操作系统发送给JVM的信号。
- Finalizer：此线程对不再需要释放系统资源的对象执行终结处理。
- Reference Handler：此线程将不再需要的对象放入队列中，以便由Finalizer线程处理。

如果主程序退出，所有这些线程都将终止。

## 3. 使用Apache Commons中的ThreadUtils类

我们还可以使用[Apache Commons Lang](https://mvnrepository.com/artifact/org.apache.commons/commons-lang3)库中的ThreadUtils类来实现相同的目标：

让我们向pom.xml文件添加一个依赖：

```xml
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.14.0</version>
</dependency>
```

只需使用getAllThreads()方法即可获取所有正在运行的线程：

```java
System.out.printf("%-15s \t %-15s \t %-15s \t %s\n", "Name", "State", "Priority", "isDaemon");
for (Thread t : ThreadUtils.getAllThreads()) {
    System.out.printf("%-15s \t %-15s \t %-15d \t %s\n", t.getName(), t.getState(), t.getPriority(), t.isDaemon());
}
```

输出与上面相同。

## 4. 总结

综上所述，我们学习了两种获取当前JVM中所有正在运行的线程的方法。