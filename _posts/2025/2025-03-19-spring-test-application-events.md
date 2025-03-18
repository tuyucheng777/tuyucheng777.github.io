---
layout: post
title:  如何测试Spring应用程序事件
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

在本教程中，我们将讨论使用[Spring应用程序事件](https://www.baeldung.com/spring-events)的测试代码。我们将首先手动创建测试实用程序，以帮助我们发布和收集用于测试目的的应用程序事件。

之后，我们将探索[Spring Modulith](https://www.baeldung.com/spring-modulith)的测试库，并使用其流式的Scenario API来讨论常见的测试用例。使用这种声明式DSL，我们将编写能够轻松生成和消费应用程序事件的富有表现力的测试。

## 2. 应用程序事件

**[Spring框架](https://www.baeldung.com/spring-intro)提供应用程序事件，允许组件在保持松散耦合的同时相互通信**。我们可以使用ApplicationEventPublisher Bean发布内部事件，这些事件是普通的Java对象。因此，所有已注册的监听器都会收到通知。

例如，当订单下达成功时，OrderService组件可以发布OrderCompletedEvent：

```java
@Service
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    // constructor

    public void placeOrder(String customerId, String... productIds) {
        Order order = new Order(customerId, Arrays.asList(productIds));
        // business logic to validate and place the order

        OrderCompletedEvent event = new OrderCompletedEvent(savedOrder.id(), savedOrder.customerId(), savedOrder.timestamp());
        eventPublisher.publishEvent(event);
    }
}
```

我们可以看到，已完成的订单现在已作为应用程序事件发布。因此，不同模块的组件现在可以监听这些事件并做出相应的反应。

我们假设LoyaltyPointsService对这些事件做出反应，以忠诚度积分奖励客户。为了实现这一点，我们可以利用Spring的@EventListener注解：

```java
@Service
public class LoyaltyPointsService {

    private static final int ORDER_COMPLETED_POINTS = 60;

    private final LoyalCustomersRepository loyalCustomers;

    // constructor

    @EventListener
    public void onOrderCompleted(OrderCompletedEvent event) {
        // business logic to reward customers
        loyalCustomers.awardPoints(event.customerId(), ORDER_COMPLETED_POINTS);
    }
}
```

**使用应用程序事件而不是直接方法调用使我们能够保持更松散的耦合并反转两个模块之间的依赖关系**。换句话说，“订单”模块对“奖励”模块中的类没有源代码依赖关系。

## 3. 测试事件监听器

**我们可以通过在测试内部发布应用程序事件来测试使用@EventListener的组件**。

为了测试LoyaltyPointsService，我们需要创建一个@SpringBootTest，注入ApplicationEventPublisher Bean，并使用它来发布OrderCompletedEvent：

```java
@SpringBootTest
class EventListenerUnitTest {

    @Autowired
    private LoyalCustomersRepository customers;

    @Autowired
    private ApplicationEventPublisher testEventPublisher;

    @Test
    void whenPublishingOrderCompletedEvent_thenRewardCustomerWithLoyaltyPoints() {
        OrderCompletedEvent event = new OrderCompletedEvent("order-1", "customer-1", Instant.now());
        testEventPublisher.publishEvent(event);

        // assertions
    }
}
```

最后，我们需要断言LoyaltyPointsService消费了该事件并向客户奖励了正确的积分数。让我们使用LoyalCustomersRepository来查看向该客户奖励了多少忠诚度积分：

```java
@Test
void whenPublishingOrderCompletedEvent_thenRewardCustomerWithLoyaltyPoints() {
    OrderCompletedEvent event = new OrderCompletedEvent("order-1", "customer-1", Instant.now());
    testEventPublisher.publishEvent(event);

    assertThat(customers.find("customer-1"))
        .isPresent().get()
        .hasFieldOrPropertyWithValue("customerId", "customer-1")
        .hasFieldOrPropertyWithValue("points", 60);
}
```

正如预期的那样，测试通过：事件被“奖励”模块接收并处理，并且奖励被应用。

## 4. 测试事件发布者

**我们可以通过在测试包中创建自定义事件监听器来测试发布应用程序事件的组件**。此监听器也将使用@EventHandler标注，类似于生产者实现。但是，这次我们将所有传入事件收集到一个列表中，该列表将通过Getter公开：

```java
@Component
class TestEventListener {

    final List<OrderCompletedEvent> events = new ArrayList<>();
    // getter

    @EventListener
    void onEvent(OrderCompletedEvent event) {
        events.add(event);
    }

    void reset() {
        events.clear();
    }
}
```

我们可以观察到，我们还可以添加实用程序reset()。我们可以在每次测试之前调用它，以清除前一个测试产生的事件。让我们创建Spring Boot测试并@Autowire我们的TestEventListener组件：

```java
@SpringBootTest
class EventPublisherUnitTest {

    @Autowired
    OrderService orderService;

    @Autowired
    TestEventListener testEventListener;

    @BeforeEach
    void beforeEach() {
        testEventListener.reset();
    }

    @Test
    void whenPlacingOrder_thenPublishApplicationEvent() {
        // place an order

        assertThat(testEventListener.getEvents())
        // verify the published events
    }
}
```

要完成测试，我们需要使用OrderService组件下订单。之后，我们将断言testEventListener只收到一个应用程序事件，并具有足够的属性：

```java
@Test
void whenPlacingOrder_thenPublishApplicationEvent() {
    orderService.placeOrder("customer1", "product1", "product2");

    assertThat(testEventListener.getEvents())
        .hasSize(1).first()
        .hasFieldOrPropertyWithValue("customerId", "customer1")
        .hasFieldOrProperty("orderId")
        .hasFieldOrProperty("timestamp");
}
```

如果我们仔细观察，就会发现这两个测试的设置和验证是相辅相成的。这个测试模拟方法调用并监听已发布的事件，而前一个测试发布事件并验证状态变化。**换句话说，我们仅使用两个测试就测试了整个过程：每个测试覆盖一个不同的部分，在逻辑模块边界处分割**。

## 5. Spring Modulith的测试支持

Spring Modulith提供了一组可以独立使用的工件，这些库提供了一系列功能，主要目的是在应用程序内的逻辑模块之间建立清晰的界限。

### 5.1 Scenario API

这种架构风格通过利用应用程序事件促进模块之间的灵活交互。因此，**Spring Modulith中的一项工件提供了对涉及应用程序事件的测试流程的支持**。

让我们将[spring-modulith-starter-test](https://mvnrepository.com/artifact/org.springframework.modulith/spring-modulith-starter-test)依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.modulith</groupId>
    <artifactId>spring-modulith-starter-test</artifactId>
    <version>1.1.3</version>
</dependency>
```

这使我们能够使用Scenario API以声明方式编写测试。首先，我们将创建一个测试类并用@ApplcationModuleTest对其进行标注。这样，我们就能够在任何测试方法中注入Scenario对象：

```java
@ApplicationModuleTest
class SpringModulithScenarioApiUnitTest {
 
    @Test
    void test(Scenario scenario) {
        // ...
    }
}
```

简而言之，此功能提供了一种方便的DSL，使我们能够测试最常见的用例。例如，它可以通过以下方式轻松启动测试并评估其结果：

- 执行方法调用
- 发布应用程序事件
- 验证状态变化
- 捕获并验证传出的事件

此外，该API还提供一些其他实用程序，例如：

- 轮询并等待异步应用程序事件
- 定义超时
- 对捕获的事件进行过滤和映射
- 创建自定义断言

### 5.2 使用Scenario API测试事件监听器

要使用@EventListener方法测试组件，我们必须注入ApplicationEventPublisher Bean并发布OrderCompletedEvent。但是，Spring Modulith的测试DSL通过scene.publish()提供了更直接的解决方案：

```java
@Test
void whenReceivingPublishOrderCompletedEvent_thenRewardCustomerWithLoyaltyPoints(Scenario scenario) {
    scenario.publish(new OrderCompletedEvent("order-1", "customer-1", Instant.now()))
        .andWaitForStateChange(() -> loyalCustomers.find("customer-1"))
        .andVerify(it -> assertThat(it)
            .isPresent().get()
            .hasFieldOrPropertyWithValue("customerId", "customer-1")
            .hasFieldOrPropertyWithValue("points", 60));
}
```

andWaitforStateChange()方法接收一个Lambda表达式，并不断重试执行，直到返回一个非null对象或非空Optional。此机制对于异步方法调用特别有用。

总而言之，**我们定义了一个场景，发布一个事件，等待状态改变，然后验证系统的最终状态**。 

### 5.3 使用Scenario API测试事件发布者 

**我们还可以使用Scenario API来模拟方法调用，并拦截和验证传出的应用程序事件**。让我们使用DSL编写一个测试来验证“order”模块的行为： 

```java
@Test
void whenPlacingOrder_thenPublishOrderCompletedEvent(Scenario scenario) {
    scenario.stimulate(() -> orderService.placeOrder("customer-1", "product-1", "product-2"))
        .andWaitForEventOfType(OrderCompletedEvent.class)
        .toArriveAndVerify(evt -> assertThat(evt)
            .hasFieldOrPropertyWithValue("customerId", "customer-1")
            .hasFieldOrProperty("orderId")
            .hasFieldOrProperty("timestamp"));
}
```

我们可以看到，andWaitforEventOfType()方法允许我们声明想要捕获的事件类型。接下来，toArriveAndVerify()用于等待事件并执行相关断言。

## 6. 总结

在本文中，我们了解了使用Spring应用程序事件测试代码的各种方法。在我们的第一个测试中，我们使用ApplicationEventPublisher手动发布应用程序事件。

类似地，我们创建了一个自定义的TestEventListener，它使用@EventHandler注解来捕获所有传出的事件。我们使用这个辅助组件来捕获和验证我们的应用程序在测试期间产生的事件。

之后，我们了解了Spring Modulith的测试支持，并使用Scenario API以声明式方式编写相同的测试。流式的DSL使我们能够发布和捕获应用程序事件、模拟方法调用并等待状态更改。