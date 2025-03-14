---
layout: post
title:  如何在不使用Thread.sleep()的情况下对ExecutorService进行单元测试
category: java-concurrency
copyright: java-concurrency
excerpt: Java Concurrency
---

## 1. 概述

[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)对象在后台运行任务，对在另一个线程上运行的任务进行单元测试很有挑战性。**父线程必须等待任务结束才能断言其结果**。

此外，解决此问题的一个方法是使用[Thread.sleep()](https://www.baeldung.com/java-thread-sleep-vs-awaitility-await#plain-java)方法，此方法会在定义的时间范围内阻塞父线程。但是，如果任务超出了sleep()上设置的时间，则单元测试会在任务之前完成并失败。

在本教程中，我们将学习如何在不使用Thread.sleep()方法的情况下对ExecutorService实例进行单元测试。

## 2. 创建Runnable对象

在进行测试之前，让我们创建一个实现Runnable接口的类：

```java
public class MyRunnable implements Runnable {
    Long result;

    public Long getResult() {
        return result;
    }

    public void setResult(Long result) {
        this.result = result;
    }

    @Override
    public void run() {
        result = sum();
    }

    private Long sum() {
        Long result = 0L;
        for (int i = 0; i < Integer.MAX_VALUE; i++) {
            result += i;
        }
        return result;
    }
}
```

MyRunnable类执行需要大量时间的计算，然后将计算出的总和设置为result成员字段。因此，这将是我们提交给执行器的任务。

## 3. 问题

通常，**ExecutorService对象在后台线程中运行任务**，任务实现Callable或Runnable接口。

**如果父线程没有等待，它会在任务完成之前终止**。因此，测试总是失败。

让我们创建一个单元测试来验证该问题：

```java
ExecutorService executorService = Executors.newSingleThreadExecutor();
MyRunnable r = new MyRunnable();
executorService.submit(r);
assertNull(r.getResult());
```

在这个测试中，我们首先创建了一个单线程的ExecutorService实例。然后，我们创建并提交了一个任务。最后，我们断言了result字段的值。

在运行时，断言在任务结束之前运行。因此，getResult()返回null。

## 4. 使用Future类

**[Future](https://www.baeldung.com/java-future-vs-promise-comparison#understanding-future)类表示后台任务的结果**。同时，**它可以阻塞父线程直到任务完成**。

让我们修改测试以使用submit()方法返回的Future对象：

```java
Future<?> future = executorService.submit(r);
future.get();
assertEquals(2305843005992468481L, r.getResult());
```

这里，**Future实例的get()方法一直阻塞直到任务结束**。

此外，当任务是Callable实例时，get()可能返回一个值。**如果任务是Runnable实例，get()总是返回null**。

现在运行测试所需的时间比以前更长，这表明父线程正在等待任务完成。最后，测试成功了。

## 5. 关机并等待

**另一个选择是使用ExecutorService类的shutdown()和awaitTermination()方法**。

**shutdown()方法关闭executor**，executor不接收任何新任务，现有任务不会被终止。但是，它不会等待它们结束。

另一方面，我们可以**使用awaitTermination()方法来阻塞，直到所有提交的任务结束**。另外，我们应该在该方法上设置一个阻塞超时时间，超过超时时间意味着阻塞结束。

让我们修改前面的测试来使用这两种方法：

```java
executorService.shutdown();
executorService.awaitTermination(10000, TimeUnit.SECONDS);
assertEquals(2305843005992468481L, r.getResult());
```

可以看出，我们在提交任务后关闭了执行器。接下来，我们调用awaitTermination()来阻塞线程，直到任务完成。

此外，我们将最大超时时间设置为10000秒。因此，如果任务运行时间超过10000秒，即使任务尚未结束，该方法也会解除阻塞。换句话说，如果我们设置较小的超时值，awaitTermination()会像Thread.sleep()一样过早解除阻塞。

确实，当我们运行它时，测试是成功了。

## 6. 使用ThreadPoolExecutor

**另一个选择是创建一个ExecutorService对象，该对象接收一定数量的作业并阻塞直到它们完成**。

一种简单的方法是扩展ThreadPoolExecutor类：

```java
public class MyThreadPoolExecutor extends ThreadPoolExecutor {
    CountDownLatch doneSignal = null;

    public MyThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue,
                                int jobsNumberToWaitFor) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue);
        doneSignal = new CountDownLatch(jobsNumberToWaitFor);
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        doneSignal.countDown();
    }

    public void waitDone() throws InterruptedException {
        doneSignal.await();
    }
}
```

这里，我们创建了继承自ThreadPoolExecutor的MyThreadPoolExecutor类。在它的构造函数中，我们添加了jobsNumberToWaitFor参数，即我们计划提交的作业数量。

此外，该类使用doneSignal字段，它是CountDownLatch类的一个实例，doneSignal字段在构造函数中用要等待的作业数进行初始化。接下来，我们重写afterExecute()方法，将doneSignal减1。作业结束时将调用afterExecute()方法。

最后，我们有waitDone()方法，它使用doneSignal来阻塞，直到所有作业结束。

另外，我们可以用单元测试来测试上述实现：

```java
@Test
void whenUsingThreadPoolExecutor_thenTestSucceeds() throws InterruptedException {
    MyThreadPoolExecutor threadPoolExecutor = new MyThreadPoolExecutor(3, 6, 10L, TimeUnit.SECONDS, new LinkedBlockingQueue<Runnable>(), 20);
    List<MyRunnable> runnables = new ArrayList<MyRunnable>();
    for (int i = 0; i < 20; i++) {
        MyRunnable r = new MyRunnable();
        runnables.add(r);
        threadPoolExecutor.submit(r);
    }
    threadPoolExecutor.waitDone();
    for (int i = 0; i < 20; i++) {
        assertEquals(2305843005992468481L, runnables.get(i).result);
    }
}
```

在此单元测试中，我们向执行器提交了20个作业。之后，我们立即调用waitDone()方法，该方法会阻塞直至20个作业完成。最后，我们断言每个作业的结果。

## 7. 总结

**在本文中，我们学习了如何在不使用Thread.sleep()方法的情况下对ExecutorService实例进行单元测试**。也就是说，我们研究了三种方法：

- 获取Future对象并调用get()方法
- 关闭执行器并等待正在运行的任务完成
- 创建自定义ExecutorService