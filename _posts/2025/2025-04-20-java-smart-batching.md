---
layout: post
title:  Java中的智能批处理
category: designpattern
copyright: designpattern
excerpt: 智能批处理
---

## 1. 概述

在本教程中，我们将了解智能批处理模式。我们将首先了解微批处理及其优缺点，然后了解智能批处理如何缓解其问题。我们还将使用简单的Java数据结构来查看这两种模式的一些示例。

## 2. 微批处理

**我们可以将微批处理视为智能批处理模式的基础**，虽然它不如智能批处理，但它是我们构建智能批处理的基础。

### 2.1 什么是微批处理？

微批处理是一种优化技术，适用于工作负载由突发小任务组成的系统。虽然它们的计算开销较小，但它们会附带一些支持每秒少量请求的操作，例如写入I/O设备。

当我们采用微批处理模式时，我们避免单独处理传入的任务。相反，我们将它们聚合在一个批次中，**当批次足够大时，我们再一起处理它们**。

通过这种分组技术，我们可以优化资源利用率，尤其是在I/O设备方面，**这种方法有助于我们缓解逐个处理突发小任务所带来的延迟**。

### 2.2 它是如何工作的？

**实现微批处理最简单的方法是将传入的任务缓存在一个集合中，例如[Queue](https://www.baeldung.com/java-queue)**，一旦集合的大小超过目标系统属性规定的特定值，我们就会将所有达到该限制的任务集中起来，并将它们一起处理。

让我们创建一个最小的MicroBatcher类：

```java
class MicroBatcher {
    Queue<String> tasksQueue = new ConcurrentLinkedQueue<>();
    Thread batchThread;
    int executionThreshold;
    int timeoutThreshold;

    MicroBatcher(int executionThreshold, int timeoutThreshold, Consumer<List<String>> executionLogic) {
        batchThread = new Thread(batchHandling(executionLogic));
        batchThread.setDaemon(true);
        batchThread.start();
        this.executionThreshold = executionThreshold;
        this.timeoutThreshold = timeoutThreshold;
    }

    void submit(String task) {
        tasksQueue.add(task);
    }

    Runnable batchHandling(Consumer<List<String>> executionLogic) {
        return () -> {
            while (!batchThread.isInterrupted()) {
                long startTime = System.currentTimeMillis();
                while (tasksQueue.size() < executionThreshold && (System.currentTimeMillis() - startTime) < timeoutThreshold) {
                    Thread.sleep(100);
                }
                List<String> tasks = new ArrayList<>(executionThreshold);
                while (tasksQueue.size() > 0 && tasks.size() < executionThreshold) {
                    tasks.add(tasksQueue.poll());
                }
                executionLogic.accept(tasks);
            }
        };
    }
}
```

我们的batcher类有两个重要字段，tasksQueue和batchThread。

**我们选择ConcurrentLinkedQueue作为队列实现，因为它支持并发访问，并且可以根据需要扩展**，所有已提交的任务都驻留在队列中。在本例中，我们将它们表示为简单的String对象，并将其作为参数传递给我们外部定义的执行逻辑(executionLogic)。

此外，我们的MicroBatcher有一个专门的线程用于批处理。需要注意的是，**任务提交和处理必须在不同的线程中完成**。这种解耦是最小化延迟的关键，**这是因为我们只允许一个线程发出慢速请求，而其余线程则可以根据需要以最快的速度提交任务，因为它们不会被操作阻塞**。

最后，我们定义executionThreshold和timeoutThreshold。executionThreshold决定了在执行任务之前必须缓冲的任务数量，其值取决于目标操作。例如，如果我们正在写入网络设备，则阈值应等于最大数据包大小。timeoutThreshold是即使尚未达到executionThreshold，在处理任务之前我们等待任务缓冲的最长时间。

### 2.3 优点和缺点

使用微批处理模式有很多好处，首先，它提高了吞吐量，因为任务的提交与执行状态无关，这意味着我们的系统响应速度更快。

此外，通过调整微批处理器，我们可以实现底层资源(例如磁盘存储)的适当利用，并将其饱和到最佳水平。

最后，**它非常符合现实世界的流量状况，现实世界的流量很少是均匀的，而且通常是突发的**。

然而，这种实现方式最大的缺点之一是，当系统负载较低时(例如在夜间)，**即使是单个请求也必须等待timeoutThreshold才能被处理**。这会导致资源利用率低下，最重要的是，用户体验不佳。

## 3. 智能批处理

智能批处理是微批处理的改良版本，**区别在于，我们省略了timeoutThreshold，并且不再等待队列填满任务，而是立即执行任意数量的任务，直至达到executionThreshold为止**。

通过这个简单的更改，我们避免了上面提到的低流量延迟问题，同时仍然保留了微批次的所有优势。这是因为通常情况下，处理一批任务所需的时间足以让队列填满下一批任务。这样一来，我们就能优化资源利用率，避免阻塞单个任务的执行(万一只有单个任务待处理)。

让我们将MicroBatcher转换为SmartBatcher：

```java
class SmartBatcher {
    BlockingQueue<String> tasksQueue = new LinkedBlockingQueue<>();
    Thread batchThread;
    int executionThreshold;
    boolean working = false;
    SmartBatcher(int executionThreshold, Consumer<List<String>> executionLogic) {
        batchThread = new Thread(batchHandling(executionLogic));
        batchThread.setDaemon(true);
        batchThread.start();
        this.executionThreshold = executionThreshold;
    }

    Runnable batchHandling(Consumer<List<String>> executionLogic) {
        return () -> {
            while (!batchThread.isInterrupted()) {
                List<String> tasks = new ArrayList<>(executionThreshold);
                while(tasksQueue.drainTo(tasks, executionThreshold) == 0) {
                    Thread.sleep(100);
                }
                working = true;
                executionLogic.accept(tasks);
                working = false;
            }
        };
    }
}
```

我们在新的实现中做了三处改动，首先，我们移除了timeoutThreshold。其次，我们将Queue的实现改为[BlockingQueue](https://www.baeldung.com/java-blocking-queue)，这些都支持drainTo()方法，完全符合我们的需求。最后，我们利用这个方法简化了batchHandling()的逻辑。

## 4. 无批处理与批处理比较

让我们创建一个具有简单场景的应用程序类来测试直接方法与批处理方法：

```java
class BatchingApp {
    public static void main(String[] args) throws Exception {
        final Path testPath = Paths.get("./test.txt");
        testPath.toFile().createNewFile();
        ScheduledExecutorService executorService = Executors.newScheduledThreadPool(100);
        Set<Future> futures = new HashSet<>();
        for (int i = 0; i < 50000; i++) {
            futures.add(executorService.submit(() -> Files.write(testPath, Collections.singleton(Thread.currentThread().getName()), StandardOpenOption.APPEND)));
        }
        long start = System.currentTimeMillis();
        for (Future future : futures) {
            future.get();
        }
        System.out.println("Time: " + (System.currentTimeMillis() - start));
        executorService.shutdown();
    }
}
```

我们选择了一个简单的文件写入作为I/O操作，我们创建一个test.txt文件，并使用100个线程向其中写入50000行。虽然控制台中显示的时间取决于目标硬件，但以下是示例：

```text
Time (ms): 4968
```

即使尝试了不同的线程数，时间仍然在4500毫秒左右，看来我们已经达到了硬件的极限。

现在让我们切换到SmartBatcher：

```java
class BatchingApp {
    public static void main(String[] args) throws Exception {
        final Path testPath = Paths.get("./testio.txt");
        testPath.toFile().createNewFile();
        SmartBatcher batcher = new SmartBatcher(10, strings -> {
            List<String> content = new ArrayList<>(strings);
            content.add("-----Batch Operation-----");
            Files.write(testPath, content, StandardOpenOption.APPEND);
        });

        for (int i = 0; i < 50000; i++) {
            batcher.submit(Thread.currentThread().getName() + "-1");
        }
        long start = System.currentTimeMillis();
        while (!batcher.finished());
        System.out.println("Time: " + (System.currentTimeMillis() - start));
    }
}
```

我们向SmartBatcher添加了finish()方法来检查所有任务何时完成：

```java
boolean finished() {
    return tasksQueue.isEmpty() && !working;
}
```

以下是显示的新时间：

```text
Time (ms): 1053
```

即使执行阈值为10，我们也实现了5倍的提升。将阈值提高到100后，时间缩短至约150毫秒，比简单方法快了近50倍。

由此可见，采用一种利用底层硬件特性的简单技术，就能显著提升应用程序的性能。**我们应该时刻牢记系统正在做什么，以及它正在处理的流量**。

## 5. 总结

在本文中，我们概述了任务批处理技术，特别是微批处理和智能批处理。我们探讨了潜在的用例、微批处理的优缺点，以及智能批处理如何弥补其不足。最后，我们比较了简单任务执行和批处理执行。