---
layout: post
title:  消费者确认和发布者向RabbitMQ确认
category: messaging
copyright: messaging
excerpt: RabbitMQ
---

## 1. 概述

在本教程中，我们将学习如何通过发布者确认来确保消息已发布到[RabbitMQ](https://www.baeldung.com/rabbitmq)代理。然后，我们将学习如何通过消费者确认来告知代理我们已成功消费了一条消息。

## 2. 场景

在简单的应用中，我们经常在使用RabbitMQ时忽略显式的确认机制，而是依赖于基本的向队列发布消息并在消费时自动确认消息。**然而，尽管RabbitMQ拥有强大的基础架构，但仍可能出现错误**，因此需要采取某种方式来仔细检查消息是否已成功送达Broker，并确认消息消费是否成功。这时，发布者确认和消费者确认就派上用场了，它们共同构成了一道安全网。

## 3. 等待发布者确认

即使我们的应用程序没有错误，已发布的消息也可能会丢失。例如，由于不明原因的网络错误，消息可能会在传输过程中丢失。为了避免这种情况，**AMQP提供了[事务语义](https://www.rabbitmq.com/docs/semantics)来保证消息不会丢失**。但是，这需要付出巨大的代价，由于事务量很大，处理消息的时间可能会显著增加，尤其是在大量事务的情况下。

相反，我们将采用确认模式，尽管会引入一些开销，但比事务更快。此模式指示客户端和代理启动消息计数，随后，客户端使用代理返回的带有相应编号的投递标签来验证此计数。此过程可确保消息的安全存储，以便后续分发给消费者。

要启用确认模式，我们需要在通道上调用confirmSelect：

```java
channel.confirmSelect();
```

**确认可能需要一些时间，尤其是对于持久队列**，因为存在IO延迟。因此，RabbitMQ异步等待确认，但提供了同步方法供我们在应用程序中使用：

- Channel.waitForConfirms()：阻塞执行，直到自上次调用以来的所有消息都被代理ACK(确认)或NACK(拒绝)。
- Channel.waitForConfirms(timeout)：与上面类似，但我们可以将等待时间限制为毫秒；否则，将抛出TimeoutException。
- Channel.waitForConfirmsOrDie()：如果自上次调用以来有任何消息被NACK，则会抛出异常；**如果我们无法容忍任何消息丢失，这个方法就很有用**。
- Channel.waitForConfirmsOrDie(timeout)：与上面相同，但有超时。

### 3.1 发布者设置

让我们从一个常规的发布消息的类开始，我们只需要接收一个[Channel](https://www.baeldung.com/java-rabbitmq-channels-connections)和一个需要连接的[Queue](https://www.baeldung.com/java-rabbitmq-exchanges-queues-bindings)：

```java
class UuidPublisher {
    private Channel channel;
    private String queue;

    public UuidPublisher(Channel channel, String queue) {
        this.channel = channel;
        this.queue = queue;
    }
}
```

然后，我们将添加一个发布字符串消息的方法：

```java
public void send(String message) throws IOException {
    channel.basicPublish("", queue, null, message.getBytes());
}
```

**当我们以这种方式发送消息时，我们有可能在传输过程中丢失它们**，因此让我们包含一些代码以确保代理安全地接收我们的消息。

### 3.2 在通道上启动确认模式

我们首先修改构造函数，使其在最后调用channel的ConfirmSelect()方法。这是必要的，这样我们才能在channel上使用“wait”方法：

```java
public UuidPublisher(Channel channel, String queue) throws IOException {
    // ...

    this.channel.confirmSelect();
}
```

**如果我们尝试在未进入确认模式的情况下等待确认，则会引发IllegalStateException**。然后，我们将选择一个同步wait()方法，并在使用send()方法发布消息后调用它。让我们设置一个超时等待时间，这样就可以确保我们不会永远等待：

```java
public boolean send(String message) throws Exception {
    channel.basicPublish("", queue, null, message.getBytes());
    return channel.waitForConfirms(1000);
}
```

返回true表示代理已成功接收消息，如果我们只发送少量消息，这种方法很有效。

### 3.3 批量确认已发布的消息

由于确认消息需要时间，我们不应该在每次发布后都等待确认。相反，我们应该先发送一批消息，然后再等待确认。让我们修改一下方法，让它接收一个消息列表，并且只在发送完所有消息后才等待：

```java
public void sendAllOrDie(List<String> messages) throws Exception {
    for (String message : messages) {
        channel.basicPublish("", queue, null, message.getBytes());
    }

    channel.waitForConfirmsOrDie(1000);
}
```

这次我们使用waitForConfirmsOrDie()，因为waitForConfirms()返回false意味着Broker拒绝了未知数量的消息。虽然这确保了如果有任何消息被拒绝，我们都会收到异常，但我们无法判断哪些消息失败了。

## 4. 利用确认模式保证批量发布

使用确认模式时，也可以在Channel上注册一个[ConfirmListener](https://rabbitmq.github.io/rabbitmq-java-client/api/4.x.x/com/rabbitmq/client/ConfirmListener.html)，此监听器接收两个回调处理程序：一个用于成功传送，另一个用于代理失败。这样，我们可以实现一种机制来确保不会遗漏任何消息，我们将从将此监听器添加到channel的方法开始：

```java
private void createConfirmListener() {
    this.channel.addConfirmListener(
            (tag, multiple) -> {
                // ...
            },
            (tag, multiple) -> {
                // ...
            }
    );
}
```

在回调中，tag参数指的是消息的顺序投递标签，而multiple参数则表示此回调是否确认了多条消息。在这种情况下，tag参数将指向最新确认的标签。相反，如果上一次回调为NACK，则所有投递标签大于最新NACK回调标签的消息也将被确认。

**为了协调这些回调，我们将未确认的消息保存在[ConcurrentSkipListMap](https://www.baeldung.com/java-concurrent-map#concurrentskiplistmap)中**。我们将待处理的消息放在那里，并使用其标签号作为键。这样，我们可以调用headMap()并获取到当前收到的标签之前的所有消息的视图：

```java
private ConcurrentNavigableMap<Long, PendingMessage> pendingDelivery = new ConcurrentSkipListMap<>();
```

已确认消息的回调将从我们的Map中删除所有tag的消息：

```java
(tag, multiple) -> {
    ConcurrentNavigableMap<Long, PendingMessage> confirmed = pendingDelivery.headMap(tag, true);
    confirmed.clear();
}
```

如果multiple为false，headMap()将包含单个元素，否则包含多个元素。因此，我们不需要检查是否收到了多条消息的确认。

### 4.1 实现被拒绝消息的重试机制

我们将为被拒消息的回调实现重试机制，此外，我们还将设置最大重试次数，以避免无限重试的情况。我们先来创建一个类，用于保存当前消息的尝试次数，并创建一个简单的方法来递增该计数器：

```java
public class PendingMessage {
    private int tries;
    private String body;

    public PendingMessage(String body) {
        this.body = body;
    }

    public int incrementTries() {
        return ++this.tries;
    }

    // standard getters
}
```

现在，让我们用它来实现回调，**我们首先获取被拒绝的消息视图，然后[删除所有](https://www.baeldung.com/java-collection-remove-elements#java-8-andcollectionremoveif)超过最大尝试次数的消息**：

```java
(tag, multiple) -> {
    ConcurrentNavigableMap<Long, PendingMessage> failed = pendingDelivery.headMap(tag, true);

    failed.values().removeIf(pending -> {
        return pending.incrementTries() >= MAX_TRIES;
    });

    // ...
}
```

然后，如果仍有待处理的消息，我们会再次发送。这次，如果应用发生意外错误，我们还会删除该消息：

```java
if (!pendingDelivery.isEmpty()) {
    pendingDelivery.values().removeIf(message -> {
        try {
            channel.basicPublish("", queue, null, message.getBody().getBytes());
            return false;
        } catch (IOException e) {
            return true;
        }
    });
}
```

### 4.2 整合

最后，我们可以创建一个新方法，用于批量发送消息，但可以检测被拒绝的消息并尝试再次发送，**我们需要在通道上调用getNextPublishSeqNo()来获取消息标签**：

```java
public void sendOrRetry(List<String> messages) throws IOException {
    createConfirmListener();

    for (String message : messages) {
        long tag = channel.getNextPublishSeqNo();
        pendingDelivery.put(tag, new PendingMessage(message));

        channel.basicPublish("", queue, null, message.getBytes());
    }
}
```

**我们在发布消息之前创建监听器；否则，我们将无法收到确认**。这将创建一个接收回调的循环，直到我们成功发送或重试所有消息为止。

## 5. 发送消费者发货确认

在探讨手动确认之前，我们先来看一个没有手动确认的示例。使用自动确认时，只要Broker将消息发送给消费者，消息即被视为已成功送达，我们来看一个简单的示例：

```java
public class UuidConsumer {
    private String queue;
    private Channel channel;

    // all-args constructor

    public void consume() throws IOException {
        channel.basicConsume(queue, true, (consumerTag, delivery) -> {
            // processing...
        }, cancelledTag -> {
            // logging...
        });
    }
}
```

通过autoAck参数将true传递给basicConsume()时，将激活自动确认，**尽管这快速且直接，但它并不安全，因为代理会在我们处理消息之前丢弃它**。因此，最安全的选择是停用它，并在channel上使用basickAck()发送手动确认，以确保消息在退出队列之前被成功处理：

```java
channel.basicConsume(queue, false, (consumerTag, delivery) -> {
    long deliveryTag = delivery.getEnvelope().getDeliveryTag();

    // processing...

    channel.basicAck(deliveryTag, false);
}, cancelledTag -> {
    // logging...
});
```

最简单的形式是，我们在处理完每条消息后进行确认，我们使用收到的相同投递标签来确认消费。**最重要的是，为了发出单独的确认信号，我们必须将false传递给basicAck()**，这可能会非常慢，所以让我们看看如何改进它。

### 5.1 定义通道的基本QoS

通常，RabbitMQ会在消息可用时立即推送，为了避免这种情况，我们将在通道上设置必要的[服务质量(QoS)](https://www.rabbitmq.com/docs/consumer-prefetch)设置。因此，我们在构造函数中添加一个batchSize参数，并将其传递给通道上的basicQos()函数，这样只会预取以下数量的消息：

```java
public class UuidConsumer {
    // ...
    private int batchSize;

    public UuidConsumer(Channel channel, String queue, int batchSize) throws IOException {
        // ...

        this.batchSize = batchSize;
        channel.basicQos(batchSize);
    }
}
```

**这有助于在我们处理所能处理的消息的同时，保持其他消费者能够获取消息**。

### 5.2 定义确认策略

我们不必对每条处理的消息都发送ACK，而是在每次达到批量大小时发送一个ACK，这样可以提高性能。为了更完整地描述场景，我们引入一个简单的处理方法，如果我们可以将消息解析为UUID，则认为该消息已处理完毕：

```java
private boolean process(String message) {
    try {
        UUID.fromString(message);
        return true;
    } catch (IllegalArgumentException e) {
        return false;
    }
}
```

现在，让我们用发送批量确认的基本框架修改我们的consume()方法：

```java
channel.basicConsume(queue, false, (consumerTag, delivery) -> {
    String message = new String(delivery.getBody(), "UTF-8");
    long deliveryTag = delivery.getEnvelope().getDeliveryTag();

    if (!process(message)) {
        // ...
    } else if (deliveryTag % batchSize == 0) {
        // ...
    } else {
        // ...
    }
}
```

如果无法处理该消息，我们将对其进行NACK处理，并检查是否已达到批处理大小以确认待处理的消息。否则，我们将存储待处理ACK消息的送达标签，以便在后续迭代中发送，我们将该标签存储在一个类变量中：

```java
private AtomicLong pendingTag = new AtomicLong();
```

### 5.3 拒绝消息

如果我们不想要或无法处理消息，我们会拒绝它们；拒绝后，我们可以重新排队。重新排队很有用，例如，当我们超出容量，并且希望其他消费者接收它而不是告诉代理丢弃它时；我们有两种方法可以实现这一点：

- channel.basicReject(deliveryTag, requeue)：拒绝单条消息，并可选择重新排队或丢弃。
- channel.basicNack(deliveryTag, multiple, requeue)：与上面相同，但可以选择批量拒绝，**将true传递给multiple会拒绝自上次ACK到当前投递标签的所有消息**。

由于我们要逐条拒绝消息，因此我们将使用第一个选项，我们将发送该消息，并在有待处理的ACK时重置变量。最后，我们拒绝该消息：

```java
if (!process(message, deliveryTag)) {
    if (pendingTag.get() != 0) {
        channel.basicAck(pendingTag.get(), true);
        pendingTag.set(0);
    }

    channel.basicReject(deliveryTag, false);
}
```

### 5.4 批量确认消息

由于投递标签是连续的，我们可以使用[取模运算符](https://www.baeldung.com/modulo-java)来检查是否已达到批次大小。如果已达到，则发送ACK并重置pendingTag。这次，将true传递给“multiple”参数至关重要，以便代理知道我们已成功处理了当前投递标签之前的所有消息：

```java
else if (deliveryTag % batchSize == 0) {
    channel.basicAck(deliveryTag, true);
    pendingTag.set(0);
} else {
    pendingTag.set(deliveryTag);
}
```

否则，我们只需设置pendingTag即可在另一次迭代中检查它。此外，针对同一标签发送多个确认将导致RabbitMQ出现“PRECONDITION_FAILED – unknown delivery tag”错误。

需要注意的是，在使用多标志发送ACK时，**我们必须考虑一些情况，例如由于没有更多消息需要处理，批次大小永远无法达到**。一种方案是保留一个观察线程，定期检查是否有待处理的ACK需要发送。

## 6. 总结

在本文中，我们探讨了RabbitMQ中的发布者确认和消费者确认的功能，这些功能对于确保分布式系统中的数据安全性和稳健性至关重要。

发布者确认使我们能够验证消息是否已成功传输至RabbitMQ代理，从而降低消息丢失的风险。消费者确认则通过确认消息消费，实现了可控且弹性的消息处理。

通过实际的代码示例，我们了解了如何有效地实现这些功能，为构建可靠的消息传递系统奠定了基础。