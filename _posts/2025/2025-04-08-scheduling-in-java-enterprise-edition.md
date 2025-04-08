---
layout: post
title:  Jakarta EE中的任务调度
category: webmodules
copyright: webmodules
excerpt: Jakarta EE
---

## 1. 概述

在[上一篇文章](https://www.baeldung.com/spring-scheduled-tasks)中，我们演示了如何使用@Scheduled注解在Spring中安排任务。在本文中，我们将针对上一篇文章中介绍的每个案例，演示如何通过在Jakarta EE应用程序中使用计时器服务来实现相同的目的。

## 2. 启用调度支持

在Jakarta EE应用程序中，无需启用对定时任务的支持。计时器服务是一种容器管理服务，允许应用程序调用为基于时间的事件安排的方法。例如，应用程序可能需要在某个时间运行一些每日报告才能生成统计数据。

有两种类型的计时器：

- **编程式定时器**：定时器服务可以注入到任何Bean(有状态会话Bean除外)中，业务逻辑应放在用@Timeout标注的方法中，定时器可以通过Bean中用@PostConstruct标注的方法初始化，也可以通过调用方法初始化。
- **自动计时器**：业务逻辑放在任何用@Schedule或@Schedules标注的方法中，这些计时器在应用程序启动时立即初始化。

## 3. 以固定延迟调度任务

在Spring中，只需使用@Scheduled(fixedDelay= 1000)注解即可完成此操作。在这种情况下，上一次执行结束和下一次执行开始之间的持续时间是固定的，任务始终等到上一个任务完成。

在Jakarta EE中实现完全相同的功能稍微困难一些，因为没有提供类似的内置机制，不过，只需一些额外的编码就可以实现类似的场景。让我们看看如何做到这一点：

```java
@Singleton
public class FixedTimerBean {

    @EJB
    private WorkerBean workerBean;

    @Lock(LockType.READ)
    @Schedule(second = "*/5", minute = "*", hour = "*", persistent = false)
    public void atSchedule() throws InterruptedException {
        workerBean.doTimerWork();
    }
}
```

```java
@Singleton
public class WorkerBean {

    private AtomicBoolean busy = new AtomicBoolean(false);

    @Lock(LockType.READ)
    public void doTimerWork() throws InterruptedException {
        if (!busy.compareAndSet(false, true)) {
            return;
        }
        try {
            Thread.sleep(20000L);
        } finally {
            busy.set(false);
        }
    }
}
```

如你所见，定时器计划每5秒触发一次。但是，在我们的例子中，触发的方法通过在当前线程上调用sleep()模拟了20秒的响应时间。

因此，容器将继续每5秒调用一次doTimerWork()，但如果前一次调用尚未完成，则方法开头设置的条件busy.compareAndSet(false, true)将立即返回。这样，我们确保只有在前一个任务完成后才会执行下一个任务。

## 4. 按固定速率调度任务

其中一种方法是使用[计时器服务](https://docs.oracle.com/javaee/6/api/javax/ejb/TimerService.html)，该服务通过@Resource注入并在标注为@PostConstruct的方法中配置，计时器到期时将调用标注为@Timeout的方法。

如上一篇文章所述，任务执行的开始不会等待上一次执行的完成；当任务的每次执行都是独立时，应使用此选项。以下代码片段创建一个每秒触发一次的计时器：

```java
@Startup
@Singleton
public class ProgrammaticAtFixedRateTimerBean {

    @Inject
    Event<TimerEvent> event;

    @Resource
    TimerService timerService;

    @PostConstruct
    public void initialize() {
        timerService.createTimer(0,1000, "Every second timer with no delay");
    }

    @Timeout
    public void programmaticTimout(Timer timer) {
        event.fire(new TimerEvent(timer.getInfo().toString()));
    }
}
```

另一种方法是使用@Scheduled注解，在下面的代码片段中，我们每5秒触发一次计时器：

```java
@Startup
@Singleton
public class ScheduleTimerBean {

    @Inject
    Event<TimerEvent> event;

    @Schedule(hour = "*", minute = "*", second = "*/5", info = "Every 5 seconds timer")
    public void automaticallyScheduled(Timer timer) {
        fireEvent(timer);
    }

    private void fireEvent(Timer timer) {
        event.fire(new TimerEvent(timer.getInfo().toString()));
    }
}
```

## 5. 调度初始延迟的任务

如果你的用例场景需要计时器延迟启动，我们也可以这样做。在这种情况下，Jakarta EE允许使用[计时器服务](https://docs.oracle.com/javaee/6/api/javax/ejb/TimerService.html)。让我们看一个例子，其中计时器的初始延迟为10秒，然后每5秒触发一次：

```java
@Startup
@Singleton
public class ProgrammaticWithInitialFixedDelayTimerBean {

    @Inject
    Event<TimerEvent> event;

    @Resource
    TimerService timerService;

    @PostConstruct
    public void initialize() {
        timerService.createTimer(10000, 5000, "Delay 10 seconds then every 5 seconds timer");
    }

    @Timeout
    public void programmaticTimout(Timer timer) {
        event.fire(new TimerEvent(timer.getInfo().toString()));
    }
}
```

我们的示例中使用的createTimer方法使用以下签名createTimer(long initialDuration, long intervalDuration, java.io.Serializable info)，其中initialDuration是第一次计时器到期通知之前必须经过的毫秒数，而intervalDuration是计时器到期通知之间必须经过的毫秒数。

在这个例子中，我们使用的initialDuration为10秒，intervalDuration为5秒；任务将在initialDuration值之后首次执行，并将继续根据intervalDuration执行。

## 6. 使用Cron表达式调度任务

我们看到的所有调度程序(无论是编程式的还是自动的)都允许使用Cron表达式，让我们看一个例子：

```java
@Schedules ({
    @Schedule(dayOfMonth="Last"),
    @Schedule(dayOfWeek="Fri", hour="23")
})
public void doPeriodicCleanup() { ... }
```

在此示例中，方法doPeriodicCleanup()将在每周星期五的23:00和每月的最后一天被调用。

## 7. 总结

在本文中，我们以之前使用Spring完成示例的文章为起点，研究了在Jakarta EE环境中调度任务的各种方法。