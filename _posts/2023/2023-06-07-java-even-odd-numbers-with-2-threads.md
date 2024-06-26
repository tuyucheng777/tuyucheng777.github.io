---
layout: post
title:  使用2个线程打印偶数和奇数
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

在本教程中，我们将了解如何使用两个线程打印偶数和奇数。

我们的目标是按顺序打印数字，一个线程只打印偶数，而另一个线程只打印奇数。我们将使用线程同步和线程间通信的概念来解决这个问题。

## 2. Java中的线程

线程是可以并发执行的轻量级进程。并发执行多个线程可以提高性能和CPU利用率，因为我们可以通过并行运行的不同线程同时处理多个任务。

有关Java中线程的更多信息，请参阅[本文](https://www.baeldung.com/java-thread-lifecycle)。

**在Java中，我们可以通过扩展Thread类或实现Runnable接口来创建线程**。在这两种情况下，我们都覆盖了run方法并在其中编写线程的实现逻辑。

有关如何使用这些方法创建线程的更多信息，请参见[此处](https://www.baeldung.com/java-runnable-vs-extending-thread)。

## 3. 线程同步

在多线程环境中，可能有两个或多个线程同时访问同一资源。这可能是致命的并导致错误的结果。为了防止这种情况，我们需要确保在给定的时间点只有一个线程访问资源。

我们可以使用线程同步来实现这一点。

**在Java中，我们可以将一个方法或代码块标记为同步，这意味着在给定的时间点只有一个线程能够进入该方法或代码块**。

有关Java中线程同步的更多详细信息，请参阅[此处](https://www.baeldung.com/java-synchronized)。

## 4. 线程间通信

线程间通信允许同步线程使用一组方法相互通信。

使用的方法有wait、notify和notifyAll，它们都是在Object类定义的。

**wait()使当前线程无限期地等待，直到某个其他线程在同一对象上调用notify()或notifyAll()**。我们可以调用notify()来唤醒等待访问此对象监视器的线程。

有关这些方法的使用的更多详细信息，请参见[此处](https://www.baeldung.com/java-wait-notify)。

## 5. 交替打印奇数和偶数

### 5.1 使用wait()和notify()

我们将使用所讨论的同步和线程间通信概念，使用两个不同的线程按顺序打印奇数和偶数。

**在第一步中，我们将实现Runnable接口来定义两个线程的逻辑**。在run方法中，我们检查数字是偶数还是奇数。

如果数字是偶数，我们调用Printer类的printEven()方法，否则我们调用printOdd()方法：

```java
static class TaskEvenOdd implements Runnable {
    private final int max;
    private final Printer print;
    private final boolean isEvenNumber;

    // standard constructors

    @Override
    public void run() {
        int number = isEvenNumber ? 2 : 1;
        while (number <= max) {
            if (isEvenNumber) {
                print.printEven(number);
            } else {
                print.printOdd(number);
            }
            number += 2;
        }
    }
}
```

我们定义Printer类如下：

```java
static class Printer {
    private volatile boolean isOdd;

    synchronized void printEven(int number) {
        while (!isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        System.out.println(Thread.currentThread().getName() + ":" + number);
        isOdd = false;
        notify();
    }

    synchronized void printOdd(int number) {
        while (isOdd) {
            try {
                wait();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
        System.out.println(Thread.currentThread().getName() + ":" + number);
        isOdd = true;
        notify();
    }
}
```

**在main方法中，我们使用定义的TaskEvenOdd类来创建两个线程**。我们创建一个Printer类的对象并将其作为参数传递给TaskEvenOdd构造函数：

```java
public static void main(String[] args) {
    Printer printer = new Printer();
    Thread t1 = new Thread(new TaskEvenOdd(printer, 10, false), "Odd");
    Thread t2 = new Thread(new TaskEvenOdd(printer, 10, true), "Even");
    t1.start();
    t2.start();
}
```

第一个线程将是打印奇数的线程，因此我们传递false作为参数isEvenNumber的值。对于第二个线程，我们改为传递true。我们将两个线程的maxValue设置为10，以便只打印从1到10的数字。

然后我们通过调用start()方法来启动这两个线程。这将调用上面定义的两个线程的run()方法，其中我们检查数字是奇数还是偶数并打印它们。

当奇数线程开始运行时，变量number的值将为1。由于它小于maxValue并且标志isEventNumber为false，因此调用printOdd()方法。在该方法中，我们检查标志isOdd是否为true，如果为true，我们调用wait()。由于isOdd最初为false，因此不会调用wait()，而是打印值。

**然后我们将isOdd的值设置为true，以便奇数线程进入等待状态并调用notify()唤醒偶数线程**。由于isOdd标志为true，偶数线程随后被唤醒并打印偶数。然后它调用notify()来唤醒奇数线程。

执行相同的过程，直到变量number的值大于maxValue。

### 5.2 使用信号量

信号量通过使用计数器控制对共享资源的访问。**如果计数器大于零，则允许访问**。如果为零，则拒绝访问。

Java在java.util.concurrent包中提供了Semaphore类，我们可以使用它来实现所上面解释的机制。有关信号量的更多详细信息，请参阅[此处](https://www.baeldung.com/java-semaphore)。

我们创建两个线程，一个奇数线程和一个偶数线程。奇数线程将打印从1开始的奇数，偶数线程将打印从2开始的偶数。

这两个线程都有一个SharedPrinter类的对象。**SharedPrinter类将有两个信号量，semOdd和semEven，这两个信号量分别有1个和0个许可**。这可以确保首先打印奇数。

我们有两个方法printEvenNumber()printOddNumber()。奇数线程调用printOddNumber()方法，偶数线程调用printEvenNumber()方法。

要打印奇数，在semOdd上调用acquire()方法，由于初始许可为1，因此它成功获取访问权，打印奇数并在evenSemaphore上调用release()。

调用release()会将semEven的许可增加1，然后偶数线程可以成功获取访问权限并打印偶数。

这是上述工作流程的代码：

```java
public class PrintEvenOddSemaphore {

    public static void main(String[] args) {
        SharedPrinter sp = new SharedPrinter();
        Thread odd = new Thread(new Odd(sp, 10), "Odd");
        Thread even = new Thread(new Even(sp, 10), "Even");

        odd.start();
        even.start();
    }

    static class SharedPrinter {
        private final Semaphore semEven = new Semaphore(0);
        private final Semaphore semOdd = new Semaphore(1);

        void printEvenNum(int num) {
            try {
                semEven.acquire();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println(Thread.currentThread().getName() + ":" + num);
            semOdd.release();
        }

        void printOddNum(int num) {
            try {
                semOdd.acquire();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
            System.out.println(Thread.currentThread().getName() + ":" + num);
            semEven.release();
        }
    }

    static class Even implements Runnable {
        private final SharedPrinter printer;
        private final int max;

        Even(SharedPrinter printer, int max) {
            this.printer = printer;
            this.max = max;
        }

        @Override
        public void run() {
            for (int i = 2; i <= max; i = i + 2) {
                printer.printEvenNum(i);
            }
        }
    }

    static class Odd implements Runnable {
        private final SharedPrinter printer;
        private final int max;

        Odd(SharedPrinter printer, int max) {
            this.printer = printer;
            this.max = max;
        }

        @Override
        public void run() {
            for (int i = 1; i <= max; i = i + 2) {
                printer.printOddNum(i);
            }
        }
    }
}
```

## 6. 总结

在本教程中，我们了解了如何在Java中使用两个线程交替打印奇数和偶数。我们查看了两种实现相同结果的方法：**使用wait()和notify()**以及**使用Semaphore**。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-concurrency-advanced-2)上获得。