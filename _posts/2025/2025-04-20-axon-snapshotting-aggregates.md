---
layout: post
title:  Axon中的快照聚合
category: ddd
copyright: ddd
excerpt: Axon
---

## 1. 概述

在本文中，我们将研究[Axon](https://www.baeldung.com/axon-cqrs-event-sourcing)如何支持聚合快照。

我们认为本文是对[Axon](https://www.baeldung.com/axon-cqrs-event-sourcing)主指南的扩展，因此，我们将再次使用[Axon Framework](https://axoniq.io/product-overview/axon-framework)和[Axon Server](https://axoniq.io/product-overview/axon-server)。本文的实现中使用前者，后者则用作事件存储和消息路由器。

## 2. 聚合快照

首先，我们来了解一下快照聚合的含义。**当我们在应用程序中开始使用[事件溯源时](https://martinfowler.com/eaaDev/EventSourcing.html)，一个自然而然的问题是如何在应用程序中保持聚合溯源的高性能**？虽然有多种优化选项，但最直接的方法是引入快照。

**聚合快照是存储聚合状态快照以改进加载的过程**，加入快照后，在命令处理之前加载聚合将变成两个步骤：

1. 检索最新的快照(如果有)，并将其作为聚合的来源。快照带有一个序列号，用于定义它代表聚合状态的时间点。
2. 从快照序列开始检索剩余事件，并获取其余聚合。

如果要启用快照功能，则需要一个触发快照创建的流程，快照创建过程应确保快照与其创建时的整体聚合状态相似。最后，聚合加载机制(即Repository)应首先加载快照，然后再加载所有剩余事件。

## 3. Axon中的聚合快照

Axon框架支持聚合快照，如需完整了解此过程，请参阅Axon参考指南的[此部分](https://docs.axoniq.io/reference-guide/axon-framework/tuning/event-snapshots)。

**在该框架内，快照过程由两个主要部分组成**：

- [Snapshotter](https://apidocs.axoniq.io/latest/org/axonframework/eventsourcing/Snapshotter.html)
- [SnapshotTriggerDefinition](https://apidocs.axoniq.io/latest/org/axonframework/eventsourcing/SnapshotTriggerDefinition.html)

**Snapshotter是为聚合实例构建快照的组件**，默认情况下，框架将使用整个聚合的状态作为快照。

SnapshotTriggerDefinition定义了Snapshotter构造快照的触发器，触发器可以是：

- 在一定数量的事件之后，或者
- 一旦加载达到一定量，或者
- 在设定的时间点

快照的存储和检索由事件存储和聚合的Repository负责，**为此，事件存储包含一个单独的部分来存储快照**。在Axon Server中，一个单独的快照文件反映了此部分。

快照加载由Repository完成，并咨询事件存储。**因此，聚合的加载和快照的合并完全由框架负责**。

## 4. 配置快照

我们将研究上一篇文章中介绍的[订单域](https://github.com/eugenp/tutorials/tree/master/patterns-modules/axon)，快照的构建、存储和加载已由Snapshotter、事件存储和Repository负责。

**因此，要将快照引入OrderAggregate，我们只需配置SnapshotTriggerDefinition**。

### 4.1 定义快照触发器

 由于应用程序使用Spring，我们可以向应用程序上下文添加SnapshotTriggerDefinition。为此，我们添加一个Configuration类：

```java
@Configuration
public class OrderApplicationConfiguration {
    @Bean
    public SnapshotTriggerDefinition orderAggregateSnapshotTriggerDefinition(
            Snapshotter snapshotter,
            @Value("${axon.aggregate.order.snapshot-threshold:250}") int threshold) {
        return new EventCountSnapshotTriggerDefinition(snapshotter, threshold);
    }
}
```

在本例中，我们选择了[EventCountSnapshotTriggerDefinition](https://apidocs.axoniq.io/latest/org/axonframework/eventsourcing/EventCountSnapshotTriggerDefinition.html)。一旦聚合的事件计数达到“阈值”，此定义就会触发快照的创建。请注意，阈值可以通过属性进行配置。

该定义还需要Snapshotter，Axon会自动将其添加到应用程序上下文中。因此，在构建触发器定义时，可以将其作为参数注入。

我们可以使用的另一个实现是[AggregateLoadTimeSnapshotTriggerDefinition](https://apidocs.axoniq.io/latest/org/axonframework/eventsourcing/AggregateLoadTimeSnapshotTriggerDefinition.html)，如果聚合的加载时间超过loadTimeMillisThreshold，此定义将触发快照的创建。最后，由于它是一个快照触发器，因此它还需要Snapshotter来构建快照。

### 4.2 使用快照触发器

现在SnapshotTriggerDefinition已经成为应用程序的一部分，我们需要将其设置为OrderAggregate，Axon的[Aggregate](https://apidocs.axoniq.io/latest/org/axonframework/spring/stereotype/Aggregate.html)注解允许我们指定快照触发器的Bean名称。 

在注解上设置Bean名称将自动为聚合配置触发器定义：

```java
@Aggregate(snapshotTriggerDefinition = "orderAggregateSnapshotTriggerDefinition")
public class OrderAggregate {
    // state, command handlers and event sourcing handlers omitted
}
```

**通过将snapshotTriggerDefinition设置为等于构造定义的Bean名称，我们指示框架为该聚合配置它**。

## 5. 快照操作

该配置将触发器定义阈值设置为“250”，**此设置意味着框架在发布250个事件后构建快照**。虽然这对于大多数应用程序来说是一个合理的默认值，但这会延长我们的测试时间。

因此，为了进行测试，我们将axon.aggregate.order.snapshot-threshold属性调整为“5”。现在，我们可以更轻松地测试快照是否有效。

为此，我们启动Axon Server和Order应用程序，向OrderAggregate发出足够的命令以生成5个事件后，我们可以通过在Axon Server仪表板中搜索来检查应用程序是否存储了快照。

要搜索快照，我们需要点击左侧标签页中的“Search”按钮，选择左上角的“Snapshots”，然后点击右侧的橙色“Search”按钮，下表应显示如下单个条目：

![](/assets/images/2025/ddd/axonsnapshottingaggregates01.png)

## 6. 总结

在本文中，我们研究了什么是聚合快照以及Axon Framework如何支持这一概念。

启用快照功能只需在聚合上配置SnapshotTriggerDefinition即可，快照的创建、存储和检索工作都已由我们处理。