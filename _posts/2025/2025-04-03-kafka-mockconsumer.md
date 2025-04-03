---
layout: post
title:  使用Kafka MockConsumer
category: kafka
copyright: kafka
excerpt: Kafka
---

## 1. 概述

在本教程中，我们将探索MockConsumer，它是[Kafka](https://www.baeldung.com/tag/kafka/)的Consumer实现之一。

首先，我们将讨论测试Kafka Consumer时需要考虑的主要事项。然后，我们将了解如何使用MockConsumer来实现测试。

## 2. 测试Kafka Consumer

从Kafka消费数据主要包括两个步骤。首先，我们必须手动订阅主题或分配主题分区。其次，我们使用poll方法轮询一批记录。

轮询通常以无限循环的方式进行，这是因为我们通常希望连续消费数据。

例如，让我们考虑仅由订阅和轮询循环组成的简单消费逻辑：

```java
void consume() {
    try {
        consumer.subscribe(Arrays.asList("foo", "bar"));
        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofSeconds(1));
            records.forEach(record -> processRecord(record));
        }
    } catch (WakeupException ex) {
        // ignore for shutdown
    } catch (RuntimeException ex) {
        // exception handling
    } finally {
        consumer.close();
    }
}
```

查看上面的代码，我们可以看到有几件事可以测试：

- 订阅
- 轮询循环
- 异常处理
- 如果Consumer被正确关闭

我们有多种选择来测试消费逻辑。

我们可以使用内存中的Kafka实例，但是，这种方法有一些缺点。一般来说，内存中的Kafka实例会使测试变得非常繁重和缓慢。此外，设置它不是一项简单的任务，并且可能导致测试不稳定。

或者，我们可以使用Mock框架来Mock Consumer。虽然使用这种方法可以使测试变得轻量，但设置起来可能有些棘手。

最后一个选项，也许是最好的选项，是使用MockConsumer，这是一个用于测试的Consumer实现。**它不仅可以帮助我们构建轻量级测试，而且设置起来也很容易**。

让我们看看它提供的功能。

## 3. 使用MockConsumer

**MockConsumer实现了kafka-clients库提供的Consumer接口。因此，它可以模拟真实Consumer的整个行为，而我们不需要编写大量代码**。

让我们看一些MockConsumer的使用示例。特别是，我们将介绍在测试消费者应用程序时可能遇到的一些常见场景，并使用MockConsumer来实现它们。

在我们的示例中，我们考虑一个应用程序，它消费来自Kafka主题的国家人口更新，更新仅包含国家名称及其当前人口：

```java
class CountryPopulation {

    private String country;
    private Integer population;

    // standard constructor, getters and setters
}
```

我们的消费者只是使用Kafka Consumer实例轮询更新、处理它们，最后使用commitSync方法提交偏移量：

```java
public class CountryPopulationConsumer {

    private Consumer<String, Integer> consumer;
    private java.util.function.Consumer<Throwable> exceptionConsumer;
    private java.util.function.Consumer<CountryPopulation> countryPopulationConsumer;

    // standard constructor

    void startBySubscribing(String topic) {
        consume(() -> consumer.subscribe(Collections.singleton(topic)));
    }

    void startByAssigning(String topic, int partition) {
        consume(() -> consumer.assign(Collections.singleton(new TopicPartition(topic, partition))));
    }

    private void consume(Runnable beforePollingTask) {
        try {
            beforePollingTask.run();
            while (true) {
                ConsumerRecords<String, Integer> records = consumer.poll(Duration.ofMillis(1000));
                StreamSupport.stream(records.spliterator(), false)
                        .map(record -> new CountryPopulation(record.key(), record.value()))
                        .forEach(countryPopulationConsumer);
                consumer.commitSync();
            }
        } catch (WakeupException e) {
            System.out.println("Shutting down...");
        } catch (RuntimeException ex) {
            exceptionConsumer.accept(ex);
        } finally {
            consumer.close();
        }
    }

    public void stop() {
        consumer.wakeup();
    }
}
```

### 3.1 创建MockConsumer实例

接下来，让我们看看如何创建MockConsumer的实例：

```java
@BeforeEach
void setUp() {
    consumer = new MockConsumer<>(OffsetResetStrategy.EARLIEST);
    updates = new ArrayList<>();
    countryPopulationConsumer = new CountryPopulationConsumer(consumer, ex -> this.pollException = ex, updates::add);
}
```

基本上，我们需要提供的只是偏移重置策略。

请注意，我们使用updates来收集countryPopulationConsumer将收到的记录，这将帮助我们断言预期结果。

同样的方法，我们使用pollException来收集和断言异常。

对于所有测试用例，我们将使用上述设置方法。现在，让我们看看消费者应用程序的几个测试用例。

### 3.2 分配主题分区

首先，让我们为startByAssigning方法创建一个测试：

```java
@Test
void whenStartingByAssigningTopicPartition_thenExpectUpdatesAreConsumedCorrectly() {
    // GIVEN
    consumer.schedulePollTask(() -> consumer.addRecord(record(TOPIC, PARTITION, "Romania", 19_410_000)));
    consumer.schedulePollTask(() -> countryPopulationConsumer.stop());

    HashMap<TopicPartition, Long> startOffsets = new HashMap<>();
    TopicPartition tp = new TopicPartition(TOPIC, PARTITION);
    startOffsets.put(tp, 0L);
    consumer.updateBeginningOffsets(startOffsets);

    // WHEN
    countryPopulationConsumer.startByAssigning(TOPIC, PARTITION);

    // THEN
    assertThat(updates).hasSize(1);
    assertThat(consumer.closed()).isTrue();
}
```

首先，我们设置MockConsumer。首先使用addRecord方法向消费者添加一条记录。

首先要记住的是，**我们不能在分配或订阅主题之前添加记录**。这就是我们使用schedulePollTask方法安排轮询任务的原因，我们安排的任务将在获取记录之前的第一次轮询上运行。因此，记录的添加将在分配之后进行。

同样重要的是，我们不能向MockConsumer添加不属于分配给它的主题和分区的记录。

然后，**为了确保消费者不会无限期地运行，我们将其配置为在第二次轮询时关闭**。

此外，我们必须设置起始偏移量，我们使用updateBeginningOffsets方法来执行此操作。

最后，我们检查是否正确消费了更新，并且消费者已关闭。

### 3.3 订阅主题

现在，让我们为startBySubscribing方法创建一个测试：

```java
@Test
void whenStartingBySubscribingToTopic_thenExpectUpdatesAreConsumedCorrectly() {
    // GIVEN
    consumer.schedulePollTask(() -> {
        consumer.rebalance(Collections.singletonList(new TopicPartition(TOPIC, 0)));
        consumer.addRecord(record("Romania", 1000, TOPIC, 0));
    });
    consumer.schedulePollTask(() -> countryPopulationConsumer.stop());

    HashMap<TopicPartition, Long> startOffsets = new HashMap<>();
    TopicPartition tp = new TopicPartition(TOPIC, 0);
    startOffsets.put(tp, 0L);
    consumer.updateBeginningOffsets(startOffsets);

    // WHEN
    countryPopulationConsumer.startBySubscribing(TOPIC);

    // THEN
    assertThat(updates).hasSize(1);
    assertThat(consumer.closed()).isTrue();
}
```

在这种情况下，**添加记录之前要做的第一件事是重新平衡**，我们通过调用rebalance方法来模拟重新平衡。

其余部分与startByAssigning测试用例相同。

### 3.4 控制轮询循环

我们可以通过多种方式控制轮询循环。

第一个选项是调度轮询任务，就像我们在上面的测试中所做的那样。我们通过schedulePollTask来执行此操作，它以Runnable为参数。**我们安排的每个任务都会在我们调用poll方法时运行**。

第二种选择是调用wakeup方法。通常，这就是我们中断长轮询调用的方法。实际上，这就是我们在CountryPopulationConsumer中实现stop方法的方法。

最后，**我们可以使用setPollException方法设置要抛出的异常**：

```java
@Test
void whenStartingBySubscribingToTopicAndExceptionOccurs_thenExpectExceptionIsHandledCorrectly() {
    // GIVEN
    consumer.schedulePollTask(() -> consumer.setPollException(new KafkaException("poll exception")));
    consumer.schedulePollTask(() -> countryPopulationConsumer.stop());

    HashMap<TopicPartition, Long> startOffsets = new HashMap<>();
    TopicPartition tp = new TopicPartition(TOPIC, 0);
    startOffsets.put(tp, 0L);
    consumer.updateBeginningOffsets(startOffsets);

    // WHEN
    countryPopulationConsumer.startBySubscribing(TOPIC);

    // THEN
    assertThat(pollException).isInstanceOf(KafkaException.class).hasMessage("poll exception");
    assertThat(consumer.closed()).isTrue();
}
```

### 3.5 模拟结束偏移和分区信息

如果我们的消费逻辑是基于结束偏移或分区信息，我们也可以使用MockConsumer来模拟这些。

当我们想要模拟结束偏移时，我们可以使用addEndOffsets和updateEndOffsets方法。

而且，如果我们想要模拟分区信息，我们可以使用updatePartitions方法。

## 4. 总结

在本文中，我们探讨了如何使用MockConsumer来测试Kafka消费者应用程序。

首先，我们查看了消费者逻辑的示例以及需要测试的必要部分。然后，我们使用MockConsumer测试了一个简单的Kafka消费者应用程序。

在此过程中，我们研究了MockConsumer的功能及其使用方法。