---
layout: post
title:  在Spring测试中禁用@EnableScheduling
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

在本教程中，我们将深入探讨测试使用[计划任务](https://www.baeldung.com/spring-scheduling-annotations)的Spring应用程序的主题。当我们尝试开发测试(尤其是[集成](https://www.baeldung.com/spring-boot-testing)测试)时，它们的广泛使用可能会引起麻烦。我们将讨论可能的选项，以确保它们尽可能稳定。

## 2. 示例

首先，我们来简单解释一下本文中将要使用的示例。假设有一个系统，允许公司代表向客户发送通知。有些通知对时间敏感，应立即发送，但有些通知应等到下一个工作日。因此，我们需要一种定期尝试发送通知的机制：

```java
public class DelayedNotificationScheduler {
    private NotificationService notificationService;

    @Scheduled(fixedDelayString = "${notification.send.out.delay}", initialDelayString = "${notification.send.out.initial.delay}")
    public void attemptSendingOutDelayedNotifications() {
        notificationService.sendOutDelayedNotifications();
    }
}
```

**我们可以在attemptSendingOutDelayedNotifications()方法上发现@Scheduled注解**，当initialDelayString配置的时间过去时，该方法将首次被调用。执行结束后，Spring会在fixedDelayString参数配置的时间之后再次调用它。该方法本身将实际逻辑委托给NotificationService。

当然，我们还需要启用调度。我们通过**在用@Configuration标注的类上应用@EnableScheduling注解来实现这一点**。虽然这很重要，但我们不会在这里深入讨论它，因为它与主要主题紧密相关。稍后，我们将看到几种方法，如何以不会对测试产生负面影响的方式来做到这一点。

## 3. 集成测试中的计划任务问题

首先，让我们为通知应用程序编写一个基本的集成测试：

```java
@SpringBootTest(
        classes = { ApplicationConfig.class, SchedulerTestConfiguration.class },
        properties = {
                "notification.send.out.delay: 10",
                "notification.send.out.initial.delay: 0"
        }
)
public class DelayedNotificationSchedulerIntegrationTest {
    @Autowired
    private Clock testClock;

    @Autowired
    private NotificationRepository repository;

    @Autowired
    private DelayedNotificationScheduler scheduler;

    @Test
    public void whenTimeIsOverNotificationSendOutTime_thenItShouldBeSent() {
        ZonedDateTime fiveMinutesAgo = ZonedDateTime.now(testClock).minusMinutes(5);
        Notification notification = new Notification(fiveMinutesAgo);
        repository.save(notification);

        scheduler.attemptSendingOutDelayedNotifications();

        Notification processedNotification = repository.findById(notification.getId());
        assertTrue(processedNotification.isSentOut());
    }
}

@TestConfiguration
class SchedulerTestConfiguration {
    @Bean
    @Primary
    public Clock testClock() {
        return Clock.fixed(Instant.parse("2024-03-10T10:15:30.00Z"), ZoneId.systemDefault());
    }
}
```

值得一提的是，@EnableScheduling注解只是应用于ApplicationConfig类，该类还负责创建我们在测试中自动装配的所有其他Bean。

让我们运行这个测试并查看生成的日志：

```text
2024-03-13T00:17:38.637+01:00  INFO 4728 --- [pool-1-thread-1] c.t.t.d.DelayedNotificationScheduler       : Scheduled notifications send out attempt
2024-03-13T00:17:38.637+01:00  INFO 4728 --- [pool-1-thread-1] c.t.t.d.NotificationService                : Sending out delayed notifications
2024-03-13T00:17:38.644+01:00  INFO 4728 --- [           main] c.t.t.d.DelayedNotificationScheduler       : Scheduled notifications send out attempt
2024-03-13T00:17:38.644+01:00  INFO 4728 --- [           main] c.t.t.d.NotificationService                : Sending out delayed notifications
2024-03-13T00:17:38.647+01:00  INFO 4728 --- [pool-1-thread-1] c.t.t.d.DelayedNotificationScheduler       : Scheduled notifications send out attempt
2024-03-13T00:17:38.647+01:00  INFO 4728 --- [pool-1-thread-1] c.t.t.d.NotificationService                : Sending out delayed notifications
```

分析输出，我们发现attemptSendingOutDelayedNotifications()方法已被调用多次。

一个调用来自主线程，其他调用来自pool-1-thread-1。

我们可以观察到这种行为，因为应用程序在启动期间初始化了计划任务，它们在属于单线程池的线程中定期调用我们的调度程序。这就是为什么我们可以看到来自pool-1-thread-1的方法调用。另一方面，来自主线程的调用是我们在集成测试中直接调用的。

**测试通过了，但该操作被调用了多次。这只是这里的代码异味，但在不太幸运的情况下可能会导致不稳定的测试**，我们的测试应该尽可能明确和隔离。因此，我们应该引入修复程序，让我们确信调用调度程序的唯一时间是我们直接调用它的时候。

## 4. 禁用集成测试的计划任务

让我们考虑一下我们可以做些什么来确保在测试期间只执行我们想要执行的代码。我们将要介绍的方法类似于允许我们[在Spring应用程序中有条件地启用计划作业](https://www.baeldung.com/spring-scheduled-enabled-conditionally)的方法，但针对集成测试进行了调整。

### 4.1 根据Profile启用@EnableScheduling注解的配置

**首先，我们可以将配置中启用调度的部分提取到另一个配置类中。然后，我们可以根据[激活的Profile](https://www.baeldung.com/spring-profiles)有条件地应用它**。在我们的例子中，我们希望在integrationTest Profile处于激活状态时禁用调度：

```java
@Configuration
@EnableScheduling
@Profile("!integrationTest")
public class SchedulingConfig {
}
```

在集成测试方面，我们唯一需要做的就是启用上述激活：

```java
@SpringBootTest(
    classes = { ApplicationConfig.class, SchedulingConfig.class, SchedulerTestConfiguration.class },
    properties = {
        "notification.send.out.delay: 10",
        "notification.send.out.initial.delay: 0"
    }
)
@ActiveProfiles("integrationTest")
```

此设置使我们能够确保在执行DelayedNotificationSchedulerIntegrationTest中定义的所有测试期间，调度被禁用，并且不会自动执行任何代码作为计划任务。

### 4.2 根据属性启用@EnableScheduling注解的配置

**另一种方法(但仍然类似)是根据[属性](https://www.baeldung.com/properties-with-spring)值启用应用程序的调度**，我们可以使用已经提取的配置类并根据不同的条件应用它：

```java
@Configuration
@EnableScheduling
@ConditionalOnProperty(value = "scheduling.enabled", havingValue = "true", matchIfMissing = true)
public class SchedulingConfig {
}
```

现在，调度取决于scheduling.enabled属性的值。如果我们有意识地将其设置为false，Spring将不会选择SchedulingConfig配置类。集成测试方面所需的更改很少：

```java
@SpringBootTest(
    classes = { ApplicationConfig.class, SchedulingConfig.class, SchedulerTestConfiguration.class },
    properties = {
        "notification.send.out.delay: 10",
        "notification.send.out.initial.delay: 0",
        "scheduling.enabled: false"
    }
)
```

其效果与我们按照前面的想法所实现的效果相同。

### 4.3 微调计划任务配置

我们可以采取的最后一种方法是仔细微调计划任务的配置。我们可以为它们设置一个非常长的初始延迟时间，以便在Spring尝试执行任何定期操作之前，集成测试有足够的时间执行：

```java
@SpringBootTest(
    classes = { ApplicationConfig.class, SchedulingConfig.class, SchedulerTestConfiguration.class },
    properties = {
        "notification.send.out.delay: 10",
        "notification.send.out.initial.delay: 60000"
    }
)
```

我们只是设置了60秒的初始延迟，集成测试应该有足够的时间通过，而不会受到Spring管理的计划任务的干扰。

**但是，我们需要注意，当无法引入前面显示的选项时，这是最后的手段，避免将任何与时间相关的依赖关系引入到代码中是一种很好的做法**。测试有时需要稍微多一点时间来执行的原因有很多，让我们考虑一个过度使用的CI服务器的简单示例。在这种情况下，我们冒着在项目中进行不稳定测试的风险。

## 5. 总结

在本文中，我们讨论了在测试使用计划任务机制的应用程序时配置集成测试的不同选项。

我们讨论了如何确保调度不会对我们的测试产生负面影响。一个好主意是通过将@EnableScheduling注解提取到有条件应用的单独配置中来禁用调度，该配置基于Profile或属性的值。当不可能时，我们始终可以为执行我们正在测试的逻辑的任务设置较高的初始延迟。