---
layout: post
title:  Quarkus中的负载削减
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

[quarkus-load-shedding扩展](https://quarkus.io/extensions/io.quarkus/quarkus-load-shedding/)提供了一种机制，用于在高流量条件下故意拒绝请求，以防止应用程序或服务内的系统过载。该库还公开了关键配置属性，可帮助自定义如何处理负载削减。

在本教程中，我们将研究如何将此扩展合并到我们的[Quarkus应用程序](https://www.baeldung.com/quarkus-io)中并定制其配置以满足我们的特定需求。

## 2. 设置

为了演示quarkus-load-shedding扩展的主要功能，我们将使用两个作为REST端点公开的Web资源：FibonacciResource和FactorialResource。在调用它们时，每个资源都会在返回结果之前引入1到15秒的随机响应延迟。

让我们首先将扩展作为[依赖](https://mvnrepository.com/artifact/io.quarkus/quarkus-load-shedding-parent)添加到我们的Quarkus项目中：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-load-shedding</artifactId>
</dependency>
```

当我们以开发模式启动应用程序时，我们应该能够在Dev UI上看到扩展：

```shell
./mvnw quarkus:dev
```

截至撰写本文时，此扩展仍处于实验状态：

![](/assets/images/2025/quarkus/quarkusloadshedding01.png)

## 3. 默认削减

添加扩展后，它将默认启用，它将使用默认配置开始删除请求。

让我们通过编辑application.properties文件来调整此配置，使其更具限制性：

```properties
quarkus.load-shedding.enabled=true
quarkus.load-shedding.max-limit=10
quarkus.load-shedding.initial-limit=5
```

此配置设置允许的并发请求的初始限制和最大限制，**max-limit设置并发请求的边界，initial-limit主要用于计算可接受的队列大小**。

现在，让我们通过使用[JMeter](https://www.baeldung.com/jmeter)调用两个端点来测试我们的配置，在JMeter中，我们设置了7个用户线程，以5秒的启动期运行两次：

![](/assets/images/2025/quarkus/quarkusloadshedding02.png)

运行此测试计划将得到以下结果，显示HTTP 503错误率在两个端点之间均匀分布。

|     标签     | #样本 | 平均 | 最小 | 最大  | 标准差| 错误％  |
|:----------:|:---:|:--:|:--:|:---:| :-----: | :------: |
| 斐波那契HTTP请求 | 14  |5735| 2  |12613|4517.63|28.571％ |
|  阶乘HTTP请求  | 14  |5195| 1  |11470|4133.69|28.571％ |
|     总共     | 28  |5465| 1  |12613|4338.33|28.571％ |

根据application.properties中的配置，quarkus-load-shedding扩展会无差别地拒绝请求。

## 4. 自定义削减

quarkus-load-shedding扩展提供了一些配置，使我们能够自定义负载削减行为，其中一个功能是能够在系统负载过重时确定应优先削减哪些请求，我们将在下一小节中讨论这一点。

### 4.1 请求优先级

首先，让我们在application.properties文件中启用优先级设置：

```properties
# ...
quarkus.load-shedding.priority.enabled=true
```

现在，让我们提供RequestPrioritizer的实现来指定我们的请求优先级：

```java
@Provider
public class LoadRequestPrioritizer implements RequestPrioritizer<HttpServerRequestWrapper> {
    @Override
    public boolean appliesTo(Object request) {
        return request instanceof HttpServerRequestWrapper;
    }

    // ...
}
```

我们的LoadRequestPrioritizer类使用@Provider标注为[CDI Bean](https://www.baeldung.com/java-ee-cdi#overview)，以便运行时可以自动发现它，我们还指定它只应处理HttpServerRequestWrapper类型的请求。

接下来，我们为特定的目标端点分配优先级：

```java
@Provider
public class LoadRequestPrioritizer implements RequestPrioritizer<HttpServerRequestWrapper> {
    //...
    @Override
    public RequestPriority priority(HttpServerRequestWrapper request) {
        String requestPath = request.path();
        if (requestPath.contains("fibonacci")) {
            return RequestPriority.CRITICAL;
        } else {
            return RequestPriority.NORMAL;
        }
    }
}
```

因此，/fibonacci端点将比其他端点具有更高的优先级，这意味着对该端点的请求比其他端点更不可能被拒绝。

### 4.2 触发负载检测

**由于quarkus-load-shedding扩展仅在检测到系统处于压力之下时才应用优先负载削减，接下来让我们模拟一个CPU负载**：

```java
@Path("/fibonacci")
public class FibonacciResource {
    //...
    @PostConstruct
    public void startLoad() {
        for (int i = 0; i < Runtime.getRuntime().availableProcessors(); i++) {
            new Thread(() -> {
                while (true) {
                    Math.pow(Math.random(), Math.random());
                }
            }).start();
        }
    }
}
```

首次调用时，FibonacciResource的@PostConstruct会触发一个CPU密集型任务，以确保CPU在测试期间保持负载。重新启动应用程序后，让我们向/fibonacci端点触发初始请求：

```shell
curl -X GET http://localhost:8080/api/fibonacci?iterations=9
```

接下来，我们使用相同的值再次执行JMeter测试计划，产生以下结果：

|     标签     | #样本 | 平均 | 最小 | 最大  | 标准差| 错误％  |
|:----------:|:---:|:--:|:--:|:---:| :-----: | :------: |
| 斐波那契HTTP请求 | 14  |5848| 9  |13355|4280.47|14.286%  |
|  阶乘HTTP请求  | 14  |6915| 10 |14819|5905.41|28.571％ |
|     总共     | 28  |6381| 9  |14819|5184.86|21.429％ |

如表所示，**由于优先级较高，/fibonacci端点的拒绝率较低**。

### 4.3 系统探测率

**ProbeFactor配置影响扩展探测或检查系统请求处理能力波动的频率**。

默认情况下，probeFactor设置为30，为了比较此设置的效果，我们首先仅对FibonacciResource运行JMeter测试，使用11个用户线程：

![](/assets/images/2025/quarkus/quarkusloadshedding03.png)

我们来看一下运行JMeter测试计划之后的结果：

|     标签     | #样本 | 平均 | 最小 | 最大  | 标准差| 错误％ |
|:----------:|:---:|:--:|:--:|:---:| :-----: | :-----: |
| 斐波那契HTTP请求 | 33  |8236| 3  |14484|4048.21|9.091％ |
|     总共     | 33  |8236| 3  |14484|4048.21|9.091％ |

正如预期的那样，只有三个请求(即33个样本 x 9.091％的错误率)被拒绝，因为我们在application.properties文件中将允许的最大并发请求数配置为10。

现在让我们在application.properties文件中将probeFactor从默认值30增加：

```properties
quarkus.load-shedding.probe-factor=70
```

然后，让我们使用与之前相同的设置重新运行JMeter测试并检查结果：

|     标签     |#样本 | 平均 | 最小 | 最大  | 标准差| 错误％ |
|:----------:|:--:|:--:|:--:|:---:| :-----: | :-----: |
| 斐波那契HTTP请求 | 33 |6515| 11 |13110|3850.50|6.061%  |
|     总共     | 33 |6515| 11 |13110|3850.50|6.061%  |

这次只有两个请求被拒绝，**设置较高的probeFactor会使系统对负载波动不那么敏感**。

### 4.4 队列管理

alphaFactor和betaFactor配置根据观察到的请求队列大小控制限制如何增加和减少(从初始限制到最大限制)，为了查看它们的效果，让我们将它们添加到application.properties文件中：

```properties
quarkus.load-shedding.alpha-factor=1
quarkus.load-shedding.beta-factor=5
```

使用上一节中的11个用户线程重新运行我们之前的JMeter测试，我们得到以下结果：

![](/assets/images/2025/quarkus/quarkusloadshedding04.png)

从结果中我们注意到，限制分配是渐进的，这导致第6个请求被拒绝，尽管尚未达到最大限制。

## 5. 总结

在本文中，我们了解了如何将quarkus-load-shedding扩展合并到我们的Quarkus应用程序中，以便我们的系统能够在负载下有效响应。我们还学习了如何通过调整扩展的配置属性来定制扩展以满足我们的需求，并更深入地了解了每个属性的含义。