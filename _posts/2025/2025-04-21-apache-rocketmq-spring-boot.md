---
layout: post
title: Apache RocketMQ与Spring Boot
category: messaging
copyright: messaging
excerpt: Apache RocketMQ
---

## 1. 简介

在本教程中，我们将使用Spring Boot和Apache RocketMQ(一个开源分布式消息传递和流数据平台)创建消息生产者和消费者。

## 2. 依赖

对于Maven项目，我们需要添加[RocketMQ Spring Boot Starter](https://mvnrepository.com/artifact/org.apache.rocketmq/rocketmq-spring-boot-starter)依赖：

```xml
<dependency>
    <groupId>org.apache.rocketmq</groupId>
    <artifactId>rocketmq-spring-boot-starter</artifactId>
    <version>2.0.4</version>
</dependency>
```

## 3. 生成消息

在我们的示例中，我们将创建一个基本的消息生成器，每当用户在购物车中添加或删除商品时，该生成器就会发送事件。

首先，让我们在application.properties中设置服务器位置和组名：

```properties
rocketmq.name-server=127.0.0.1:9876
rocketmq.producer.group=cart-producer-group
```

请注意，如果我们有多个名称服务器，我们可以像host:port;host:port一样列出它们。

现在，为了简单起见，我们将创建一个CommandLineRunner应用程序并在应用程序启动期间生成一些事件：

```java
@SpringBootApplication
public class CartEventProducer implements CommandLineRunner {

    @Autowired
    private RocketMQTemplate rocketMQTemplate;

    public static void main(String[] args) {
        SpringApplication.run(CartEventProducer.class, args);
    }

    public void run(String... args) throws Exception {
        rocketMQTemplate.convertAndSend("cart-item-add-topic", new CartItemEvent("bike", 1));
        rocketMQTemplate.convertAndSend("cart-item-add-topic", new CartItemEvent("computer", 2));
        rocketMQTemplate.convertAndSend("cart-item-removed-topic", new CartItemEvent("bike", 1));
    }
}
```

CartItemEvent仅包含两个属性-商品ID和数量：

```java
class CartItemEvent {
    private String itemId;
    private int quantity;

    // constructor, getters and setters
}
```

在上面的示例中，我们使用了convertAndSend()方法(AbstractMessageSendingTemplate抽象类定义的一个泛型方法)来发送购物车事件，该方法接收两个参数：目的地(在本例中为主题名称)和消息负载。

## 4. 消息消费者

消费RocketMQ消息非常简单，只需创建一个用@RocketMQMessageListener标注的Spring组件并实现RocketMQListener接口即可：

```java
@SpringBootApplication
public class CartEventConsumer {

    public static void main(String[] args) {
        SpringApplication.run(CartEventConsumer.class, args);
    }

    @Service
    @RocketMQMessageListener(
            topic = "cart-item-add-topic",
            consumerGroup = "cart-consumer_cart-item-add-topic"
    )
    public class CardItemAddConsumer implements RocketMQListener<CartItemEvent> {
        public void onMessage(CartItemEvent addItemEvent) {
            log.info("Adding item: {}", addItemEvent);
            // additional logic
        }
    }

    @Service
    @RocketMQMessageListener(
            topic = "cart-item-removed-topic",
            consumerGroup = "cart-consumer_cart-item-removed-topic"
    )
    public class CardItemRemoveConsumer implements RocketMQListener<CartItemEvent> {
        public void onMessage(CartItemEvent removeItemEvent) {
            log.info("Removing item: {}", removeItemEvent);
            // additional logic
        }
    }
}
```

我们需要为每个监听的消息主题创建一个单独的组件，在每个监听器中，我们通过@RocketMQMessageListener注解定义主题名称和消费者组名称。

## 5. 同步和异步传输

在前面的例子中，我们使用了convertAndSend方法发送消息。不过，我们还有其他选择。

例如，我们可以调用syncSend，它与convertAndSend不同，因为它返回SendResult对象。

例如，它可以用于验证我们的消息是否已成功发送或获取其ID：

```java
public void run(String... args) throws Exception { 
    SendResult addBikeResult = rocketMQTemplate.syncSend("cart-item-add-topic", new CartItemEvent("bike", 1)); 
    SendResult addComputerResult = rocketMQTemplate.syncSend("cart-item-add-topic", new CartItemEvent("computer", 2)); 
    SendResult removeBikeResult = rocketMQTemplate.syncSend("cart-item-removed-topic", new CartItemEvent("bike", 1)); 
}
```

与convertAndSend类似，仅当发送过程完成时才返回此方法。

对于可靠性要求较高的情况，比如重要的通知消息或者短信通知，应该使用同步传输。

另一方面，我们可能希望异步发送消息，并在发送完成时收到通知。

我们可以使用asyncSend来实现这一点，它接收SendCallback作为参数并立即返回：

```java
rocketMQTemplate.asyncSend("cart-item-add-topic", new CartItemEvent("bike", 1), new SendCallback() {
    @Override
    public void onSuccess(SendResult sendResult) {
        log.error("Successfully sent cart item");
    }

    @Override
    public void onException(Throwable throwable) {
        log.error("Exception during cart item sending", throwable);
    }
});
```

在需要高吞吐量的情况下，我们使用异步传输。

最后，对于吞吐量要求非常高的情况，我们可以使用sendOneWay代替asyncSend，sendOneWay与asyncSend的不同之处在于它不能保证消息被发送。

单向传输也可以用于普通的可靠性情况，例如收集日志。

## 6. 在事务中发送消息 

RocketMQ为我们提供了在事务内发送消息的功能，我们可以通过使用sendInTransaction()方法来实现：

```java
MessageBuilder.withPayload(new CartItemEvent("bike", 1)).build();
rocketMQTemplate.sendMessageInTransaction("test-transaction", "topic-name", msg, null);
```

另外，我们必须实现RocketMQLocalTransactionListener接口：

```java
@RocketMQTransactionListener(txProducerGroup="test-transaction")
class TransactionListenerImpl implements RocketMQLocalTransactionListener {
    @Override
    public RocketMQLocalTransactionState executeLocalTransaction(Message msg, Object arg) {
        // ... local transaction process, return ROLLBACK, COMMIT or UNKNOWN
        return RocketMQLocalTransactionState.UNKNOWN;
    }

    @Override
    public RocketMQLocalTransactionState checkLocalTransaction(Message msg) {
        // ... check transaction status and return ROLLBACK, COMMIT or UNKNOWN
        return RocketMQLocalTransactionState.COMMIT;
    }
}
```

sendMessageInTransaction()中第一个参数是事务名称，必须与@RocketMQTransactionListener的成员字段txProducerGroup一致。

## 7. 消息生产者配置

我们还可以配置消息生产者本身的各个方面：

- rocketmq.producer.send-message-timeout：消息发送超时时间(毫秒)，默认值为3000。
- rocketmq.producer.compress-message-body-threshold：超过此阈值时，RocketMQ将压缩消息，默认值为1024。
- rocketmq.producer.max-message-size：最大消息大小(以字节为单位)，默认值为4096。
- rocketmq.producer.retry-times-when-send-async-failed：发送失败前在异步模式下内部执行的最大重试次数，默认值为2。
- rocketmq.producer.retry-next-server：指示在内部发送失败时是否重试另一个Broker，默认值为false。
- rocketmq.producer.retry-times-when-send-failed：发送失败前在异步模式下内部执行的最大重试次数，默认值为2。

## 8. 总结

在本文中，我们学习了如何使用Apache RocketMQ和Spring Boot发送和消费消息。