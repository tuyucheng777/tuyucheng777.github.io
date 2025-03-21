---
layout: post
title:  如何让Spring Boot应用程序只执行一次计划任务
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 简介

在本教程中，我们将学习如何安排任务仅运行一次。计划任务通常用于自动化报告或发送通知等流程，通常，我们将这些任务设置为定期运行。不过，在某些情况下，我们可能希望安排任务在未来某个时间仅执行一次，例如初始化资源或执行数据迁移。

我们将探索在Spring Boot应用程序中安排任务仅运行一次的几种方法，从使用带有初始延迟的[@Scheduled注解](https://www.baeldung.com/spring-scheduled-tasks)到TaskScheduler](https://www.baeldung.com/spring-task-scheduler)和自定义触发器等更灵活的方法，我们将学习如何确保我们的任务只执行一次，而不会出现意外重复。

## 2. 仅具有启动时间的TaskScheduler

虽然@Scheduled注解提供了一种直接的方法来安排任务，但它在灵活性方面受到限制。当我们需要对任务规划进行更多控制(尤其是一次性执行)时，Spring的[TaskScheduler](https://www.baeldung.com/spring-task-scheduler)接口提供了一种更通用的替代方案。**使用TaskScheduler，我们可以以编程方式安排具有指定开始时间的任务，从而为动态调度场景提供更大的灵活性**。

TaskScheduler中最简单的方法允许我们定义一个[Runnable](https://www.baeldung.com/java-runnable-vs-extending-thread)任务和一个[Instant](https://www.baeldung.com/current-date-time-and-timestamp-in-java-8#Timestamp)，表示我们希望它执行的确切时间。这种方法使我们能够动态安排任务，而无需依赖固定的注解。让我们编写一个方法来安排任务在未来的特定时间点运行：

```java
private TaskScheduler scheduler = new SimpleAsyncTaskScheduler();

public void schedule(Runnable task, Instant when) {
    scheduler.schedule(task, when);
}
```

**TaskScheduler中的所有其他方法都是用于定期执行的，因此此方法对于一次性任务很有帮助**。最重要的是，我们使用SimpleAsyncTaskScheduler进行演示，但我们可以切换到适合我们需要运行的任务的任何其他实现。

计划任务很难测试，但我们可以使用[CountDownLatch](https://www.baeldung.com/java-countdown-latch)等待我们选择的执行时间并确保它只执行一次。让我们使用任务调用latch的countDown()，并将其安排在未来一秒钟：

```java
@Test
void whenScheduleAtInstant_thenExecutesOnce() throws InterruptedException {
    CountDownLatch latch = new CountDownLatch(1);

    scheduler.schedule(latch::countDown,
            Instant.now().plus(Duration.ofSeconds(1)));

    boolean executed = latch.await(5, TimeUnit.SECONDS);
    assertTrue(executed);
}
```

我们使用的是接收超时的版本的latch.await()，因此我们永远不会无限期地等待。如果它返回true，我们断言任务已成功完成，并且我们的latch只有一个countDown()调用。

## 3. 仅在初始延迟时使用@Scheduled

在Spring中安排一次性任务的最简单方法之一是使用带有初始延迟的[@Scheduled](https://www.baeldung.com/spring-scheduled-tasks)注解并省略fixedDelay或fixedRate属性。**通常，我们使用@Scheduled定期运行任务，但是当我们仅指定initialDelay时，任务将在指定的延迟后执行一次，而不会重复**：

```java
@Scheduled(initialDelay = 5000)
public void doTaskWithInitialDelayOnly() {
    // ...
}
```

在这种情况下，我们的方法将在包含此方法的组件初始化后5秒(5000毫秒)运行。由于我们没有指定任何速率属性，因此该方法在首次执行后不会重复。当我们需要在应用程序启动后仅运行一次任务或出于某种原因想要延迟执行任务时，这种方法很有用。

**例如，这对于在应用程序启动后几秒钟运行CPU密集型任务非常方便，允许其他服务和组件在消耗资源之前正确初始化**。但是，这种方法的一个限制是调度是静态的。我们无法在运行时动态调整延迟或执行时间。还值得注意的是，@Scheduled注解要求该方法是Spring管理的组件或服务的一部分。

### 3.1 Spring 6之前

在Spring 6之前，不可能省略延迟或速率属性，因此我们唯一的选择是指定理论上无法达到的延迟：

```java
@Scheduled(initialDelay = 5000, fixedDelay = Long.MAX_VALUE)
public void doTaskWithIndefiniteDelay() {
    // ...
}
```

在此示例中，任务将在初始5秒延迟后执行，后续执行要等到数百万年才会发生，这实际上使其成为一次性任务。虽然这种方法有效，但如果我们需要灵活性或更简洁的代码，它并不理想。

## 4. 创建没有下一次执行的PeriodicTrigger

我们的最后一个选择是实现[PeriodicTrigger](https://www.baeldung.com/spring-task-scheduler#scheduling-with-periodictrigger)，在需要更多可重用、更复杂的调度逻辑的情况下，使用它而不是TaskScheduler对我们很有帮助。**我们可以覆盖nextExecution()以仅在尚未触发时返回下一次执行时间**。

让我们首先定义一个周期和初始延迟：

```java
public class OneOffTrigger extends PeriodicTrigger {
    public OneOffTrigger(Instant when) {
        super(Duration.ofSeconds(0));
        Duration difference = Duration.between(Instant.now(), when);
        setInitialDelay(difference);
    }

    // ...
}
```

由于我们希望只执行一次，因此我们可以将任何内容设置为间隔。由于我们必须传递一个值，因此我们将传递一个0。**最后，我们计算出我们希望任务执行的期望时刻与当前时间之间的差值，因为我们需要将Duration传递给我们的初始延迟**。

然后，为了覆盖nextExecution()，我们检查上下文中的最后完成时间：

```java
@Override
public Instant nextExecution(TriggerContext context) {
    if (context.lastCompletion() == null) {
        return super.nextExecution(context);
    }

    return null;
}
```

null完成意味着它尚未触发，因此我们让它调用默认实现。否则，我们返回null，这使其成为仅执行一次的触发器。最后，让我们创建一个方法来使用它：

```java
public void schedule(Runnable task, PeriodicTrigger trigger) {
    scheduler.schedule(task, trigger);
}
```

### 4.1 测试PeriodicTrigger

最后，我们可以编写一个简单的测试来确保触发器的行为符合预期。在此测试中，我们使用CountDownLatch来跟踪任务是否执行，我们使用OneOffTrigger安排任务并验证它是否只运行一次：

```java
@Test
void whenScheduleWithRunOnceTrigger_thenExecutesOnce() throws InterruptedException {
    CountDownLatch latch = new CountDownLatch(1);

    scheduler.schedule(latch::countDown, new OneOffTrigger(Instant.now().plus(Duration.ofSeconds(1))));

    boolean executed = latch.await(5, TimeUnit.SECONDS);
    assertTrue(executed);
}
```

## 5. 总结

在本文中，我们探讨了在Spring Boot应用程序中安排任务仅运行一次的解决方案。我们从最简单的选项开始，使用不带固定速率的@Scheduled注解。然后，我们转向更灵活的解决方案，例如使用TaskScheduler进行动态调度和创建确保任务仅执行一次的自定义触发器。

每种方法都提供不同级别的控制，因此我们选择最适合我们用例的方法。