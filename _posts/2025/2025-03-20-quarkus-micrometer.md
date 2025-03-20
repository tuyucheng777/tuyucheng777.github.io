---
layout: post
title:  Quarkus中的Micrometer指南
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 简介

监控和可观察性是现代应用程序开发不可或缺的方面，尤其是在云原生和微服务架构中。

[Quarkus](https://www.baeldung.com/quarkus-io)已成为构建基于Java的应用程序的热门选择，并以其轻量级和快速的特性而闻名。**将[Micrometer](https://www.baeldung.com/micrometer)集成到我们的Quarkus应用程序中，为监控应用程序性能和行为的各个方面提供了强大的解决方案**。

在本教程中，我们将探索在Quarkus中使用Micrometer的高级监控技术。

## 2. Maven依赖

要将Micrometer与Quarkus一起使用，我们需要包含[quarkus-micrometer-registry-prometheus](https://mvnrepository.com/artifact/io.quarkus/quarkus-micrometer-registry-prometheus)依赖：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-micrometer-registry-prometheus</artifactId>
    <version>3.11.0</version>
</dependency>
```

**此依赖提供了用于检测我们代码的必要接口和类，并包含特定的注册表实现**。具体来说，micrometer-registry-prometheus是一种流行的选择，它实现了Prometheus REST端点以在我们的Quarkus应用程序中公开指标。
这还间接包括核心[quarkus-micrometer](https://mvnrepository.com/artifact/io.quarkus/quarkus-micrometer)依赖，除了我们将用于自定义指标的指标注册表之外，它还提供了来自JVM、线程池和HTTP请求的开箱即用指标。

## 3. 计数器

现在我们已经了解了如何在Quarkus应用程序中包含Micrometer，让我们实现自定义指标。首先，我们将研究在应用程序中添加基本计数器以跟踪各种操作的使用情况。

我们的Quarkus应用程序实现了一个简单的端点来确定给定的字符串是否为回文，[回文](https://en.wikipedia.org/wiki/Palindrome)是从后往前读和从前往后读都相同的字符串，例如“radar”或“level”。我们特别希望计算每次调用此回文检查的次数。

让我们创建一个[Micrometer计数器](https://docs.micrometer.io/micrometer/reference/concepts/counters.html)：

```java
@Path("/palindrome")
@Produces("text/plain")
public class PalindromeResource {

    private final MeterRegistry registry;

    public PalindromeResource(MeterRegistry registry) {
        this.registry = registry;
    }

    @GET
    @Path("check/{input}")
    public boolean checkPalindrome(String input) {
        registry.counter("palindrome.counter").increment();
        boolean result = internalCheckPalindrome(input);
        return result;
    }

    private boolean internalCheckPalindrome(String input) {
        int left = 0;
        int right = input.length() - 1;

        while (left < right) {
            if (input.charAt(left) != input.charAt(right)) {
                return false;
            }
            left++;
            right--;
        }
        return true;
    }
}
```

我们可以通过对'/palindrome/check/{input}'发送GET请求来执行回文检查，其中input是我们要检查的单词。

**为了实现我们的计数器，我们将MeterRegistry注入到PalindromeResource中**。值得注意的是，我们在每次回文检查之前都会自增计数器。最后，在多次调用端点之后，我们可以调用“/q/metrics“端点来查看计数器指标。我们将在palindrome_counter_total条目中找到我们调用操作的次数。

## 4. 计时器

我们还可以跟踪回文检查的持续时间。**因此，为了实现这一点，我们将向PalindromeResource添加一个Micrometer Timer**：

```java
@GET
@Path("check/{input}")
public boolean checkPalindrome(String input) {
    Timer.Sample sample = Timer.start(registry);
    boolean result = internalCheckPalindrome(input);
    sample.stop(registry.timer("palindrome.timer"));
    return result;
}
```

首先，我们启动计时器，它会创建一个Timer.Sample来跟踪操作持续时间。然后我们在启动计时器后调用我们的internalCheckPalindrome()方法。最后，我们停止计时器并记录经过的时间。**通过合并此计时器，我们可以监视每次回文检查的持续时间，这也使我们能够识别性能瓶颈并优化应用程序的效率**。

Micrometer遵循Prometheus的计时器指标约定，将测量的持续时间转换为秒，并将此单位包含在指标名称中。

多次调用端点后，我们可以在同一个指标端点看到以下指标：

- palindrome_timer_seconds_count：计数器被调用的次数
- palindrome_timer_seconds_sum：所有方法调用的总时长
- palindrome_timer_seconds_max：衰减间隔内观察到的最大持续时间

最后，查看计时器生成的数据，我们可以使用总和和计数来计算确定字符串是否为回文所需的平均时间。

## 5. 仪表

**仪表是一种指标，表示可以任意上下波动的单个数值**。仪表使我们能够监控实时指标，深入了解动态值并帮助我们快速响应不断变化的条件。它们对于跟踪经常波动的值(例如队列大小和线程数)特别有用。

假设我们想将所有选中的单词保留在程序内存中，并将它们保存到数据库或发送到另一个服务，我们需要跟踪元素的数量以监控程序的内存。让我们为此实现一个仪表。

注入注册表后，我们将在构造函数中初始化一个仪表，并声明一个空列表来存储输入：

```java
private final LinkedList<String> list = new LinkedList<>();

public PalindromeResource(MeterRegistry registry) {
    this.registry = registry;
    registry.gaugeCollectionSize("palindrome.list.size", Tags.empty(), list);
}
```

现在，每当我们收到输入时，我们就会将元素添加到列表中，并检查palindrome_list_size值以查看仪表的大小：

```java
list.add(input);
```

该仪表有效地为我们提供了当前程序状态的快照。

我们还可以模拟清空列表并重置我们的仪表：

```java
@DELETE
@Path("empty-list")
public void emptyList() {
    list.clear();
}
```

这表明仪表是实时测量的。清除列表后，我们的palindrome_list_size仪表将重置为0，直到我们检查更多回文。

## 6. 总结

在Quarkus中使用Micrometer的过程中，我们学会了使用计数器跟踪执行特定操作的频率、使用计时器跟踪操作的持续时间以及使用仪表监控实时指标。这些工具为我们的应用程序性能提供了宝贵的见解，使我们能够做出明智的优化决策。