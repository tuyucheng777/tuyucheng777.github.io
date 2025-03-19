---
layout: post
title:  理解JMeter中的Ramp-up
category: load
copyright: load
excerpt: JMeter
---

## 1. 概述

[JMeter](https://www.baeldung.com/jmeter)是一种非常流行的JVM应用程序性能测试解决方案，**我们在JMeter中经常遇到的一个问题是如何控制给定时间内发出的请求数量**，这在应用程序刚刚启动并启动线程的预热阶段尤其重要。

**JMeter测试计划有一个称为ramp-up period的属性，它提供了一种配置用户向被测系统发出请求的频率的方法**。此外，我们可以使用此属性来调整每秒的请求数。

在本教程中，我们将简要介绍JMeter的一些主要概念，并重点介绍JMeter中ramp-up的使用。然后，我们将尝试清楚地了解它并学习如何正确计算/配置它。

## 2. JMeter概念

**JMeter是一个用Java编写的开源软件，用于对JVM应用程序进行性能测试**。最初，它是为了测试Web应用程序的性能而创建的，但现在它支持其他类型，例如FTP、JDBC、Java对象等。

JMeter的一些主要功能包括：

- [命令行模式](https://www.baeldung.com/java-jmeter-command-line)，建议用于更准确的负载测试
- 支持[分布式性能测试](https://www.baeldung.com/jmeter-distributed-testing)
- 丰富的报告，包括[仪表板报告](https://www.baeldung.com/jmeter-dashboard-report)
- 使用[线程组](https://www.baeldung.com/jmeter-run-multiple-thread-groups)同时进行多线程采样

**使用JMeter运行负载测试涉及三个主要阶段。首先，我们需要创建一个测试计划，其中包含我们想要Mock的场景的配置**。例如，测试执行周期、每秒操作数、并发操作等。**然后，我们执行测试计划，最后，我们分析结果**。

### 2.1 测试计划的线程组

为了理解JMeter中的ramp-up，我们首先需要了解测试计划的线程组。线程组控制测试计划的线程数，其主要元素包括：

- 线程数，即我们希望执行当前测试计划的不同用户的数量
- 启动期(ramp-up period，以秒为单位)是JMeter让所有用户启动并运行所需的时间
- 循环计数是重复的次数，换句话说，它设置用户(线程)执行当前测试计划的次数

![](/assets/images/2025/load/javajmeterrampup01.png)

## 3. JMeter中的Ramp-up

如前所述，**JMeter中的ramp-up是配置JMeter启动所有线程所需时间的属性，这也意味着启动每个线程之间的延迟是多少**。例如，如果我们有两个线程，ramp-up周期为1秒，那么JMeter需要一秒钟来启动所有用户，每个用户启动大约有半秒的延迟(1秒 / 2个线程 = 0.5秒)。

如果我们将值设置为0秒，那么JMeter会立即启动我们所有的用户。

需要注意的一点是，线程数指的是用户数。因此，使用JMeter中的ramp-up可以调整每秒用户数，这与每秒操作数不同。后者是性能测试中的一个关键概念，也是我们在设置性能测试场景时关注的一个概念。**ramp-up周期与其他一些属性相结合，也有助于我们设置每秒操作数值**。

### 3.1 使用JMeter中的Ramp-up配置每秒用户数

在以下示例中，我们将针对Spring Boot应用程序执行测试。我们要测试的端点是UUID生成器，它只创建一个UUID并返回响应：

```java
@RestController
public class RetrieveUuidController {
    @GetMapping("/api/uuid")
    public Response uuid() {
        return new Response(format("Test message... %s.", UUID.randomUUID()));
    }
}
```

现在，假设我们想以每秒30个用户的速度测试我们的服务。我们可以通过设置线程数为30并将ramp-up period设置为1来轻松实现这一点。但这对于性能测试来说并没有太大的价值，因此假设我们想要每秒30个用户，持续1分钟：

![](/assets/images/2025/load/javajmeterrampup02.png)

**我们将线程总数设置为1800，这意味着一分钟内每秒有30个用户(每秒30个用户 * 60秒 = 1800个用户)**。由于每个用户执行一次测试计划(循环计数设置为1)，因此吞吐量或每秒操作数也是每秒30次操作，如摘要报告所示：

![](/assets/images/2025/load/javajmeterrampup03.png)

此外，我们可以从JMeter图表中看到，测试确实执行了1分钟(〜21:58:42至21:59:42)：

![](/assets/images/2025/load/javajmeterrampup04.png)

### 3.2 使用JMeter中的Ramp-up配置每秒请求数

在以下示例中，我们将使用上一段中介绍的Spring Boot应用程序的相同端点。

在性能测试中，我们更关心系统吞吐量，而不是某个时间段内的用户数量。这是因为同一个用户可以在给定时间内发出多个请求，例如在购物时多次单击“Add to cart”按钮。**JMeter中的Ramp-up可以与循环计数结合使用，以配置我们想要实现的吞吐量，而与用户数量无关**。

让我们考虑这样一个场景：我们想要以每秒60次操作的速度测试我们的服务，持续一分钟。我们可以通过两种方式实现这一点：

1. 我们可以将线程总数设置为目标操作数 * 执行总秒数，并将循环计数设置为1。这与之前的方法相同，这样每个操作都有不同的用户。
2. 我们将线程总数设置为目标操作数 * 执行总秒数/循环数，并设置一些循环数，以便每个用户执行多个操作。

让我们来计算一下。在第一种情况下，与前面的示例类似，我们将线程总数设置为3600，这意味着一分钟内每秒有60个用户(每秒60个操作 * 60秒 = 3600个用户)：

![](/assets/images/2025/load/javajmeterrampup05.png)

这使得我们的吞吐量达到每秒约60次操作，如摘要报告所示：

![](/assets/images/2025/load/javajmeterrampup06.png)

从该图中，我们还可以看到总样本数(对我们端点的请求)是我们预期的数量，即3600。总执行时间(~21:58:42至21:59:42)和响应延迟可以在响应时间图中看到：

![](/assets/images/2025/load/javajmeterrampup07.png)

**在第二种情况下，我们的目标是一分钟内每秒执行60次操作，但我们希望每个用户在系统上执行两次操作。因此，我们将线程总数设置为1800，ramp-up为60秒，循环计数为2，(每秒60次操作 * 60秒 / 循环计数2 = 1800个用户)**：

![](/assets/images/2025/load/javajmeterrampup08.png)

从汇总结果中我们可以看到，总样本数再次达到预期的3600。吞吐量也符合预期，约为每秒60个：

![](/assets/images/2025/load/javajmeterrampup09.png)

最后，我们在响应时间图中验证执行时间为一分钟(~21:58:45至21:59:45)：

![](/assets/images/2025/load/javajmeterrampup10.png)

## 4. 总结

在本文中，我们研究了JMeter中的ramp-up属性。我们了解了ramp-up period的作用以及如何使用它来调整性能测试的两个主要方面，即每秒用户数和每秒请求数。最后，我们使用REST服务来演示如何在JMeter中使用ramp-up，并展示一些测试执行的结果。