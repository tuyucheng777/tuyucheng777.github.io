---
layout: post
title:  DDD有界上下文和Java模块
category: ddd
copyright: ddd
excerpt: DDD
---

## 1. 概述

**领域驱动设计(DDD)是一套原则和工具，可帮助我们设计有效的软件架构，从而实现更高的业务价值**。有界上下文是将架构从“泥球”中解救出来的核心和必要模式之一，它通过将整个应用程序域划分为多个语义一致的部分。

同时，借助[Java 9模块系统](https://www.baeldung.com/java-9-modularity)，我们可以创建强封装的模块。

在本教程中，我们将创建一个简单的商店应用程序，并了解如何利用Java 9模块，同时为有界上下文定义明确的边界。

## 2. DDD有界上下文

如今，软件系统已不再是简单的[CRUD应用程序](https://www.baeldung.com/spring-boot-crud-thymeleaf)。实际上，典型的单体企业系统由一些遗留代码库和新增功能组成。然而，随着每次变更，维护这样的系统变得越来越困难。最终，它可能变得完全无法维护。

### 2.1 有界上下文和通用语言

为了解决这个问题，DDD提出了“有界上下文”的概念。**有界上下文是某个领域的逻辑边界，特定术语和规则在该领域中始终适用。在此边界内，所有术语、定义和概念构成了通用语言**。

具体来说，通用语言的主要好处是将来自不同区域的项目成员聚集在特定业务领域周围。

此外，同一个事物可能在多个上下文中起作用。然而，它在各个上下文中可能具有不同的含义。

![](/assets/images/2025/ddd/javamodulesdddboundedcontexts01.png)

### 2.2 订单上下文

让我们通过定义订单上下文来实现我们的应用程序，此上下文包含两个实体：OrderItem和CustomerOrder。

![](/assets/images/2025/ddd/javamodulesdddboundedcontexts02.png)

CustomerOrder实体是一个[聚合根](https://www.baeldung.com/spring-persisting-ddd-aggregates)：

```java
public class CustomerOrder {
    private int orderId;
    private String paymentMethod;
    private String address;
    private List<OrderItem> orderItems;

    public float calculateTotalPrice() {
        return orderItems.stream().map(OrderItem::getTotalPrice)
            .reduce(0F, Float::sum);
    }
}
```

我们可以看到，这个类包含一个叫calculateTotalPrice的业务方法，但是在实际项目中，它可能会复杂得多-例如，在最终价格中包含折扣和税费。

接下来，让我们创建OrderItem类：

```java
public class OrderItem {
    private int productId;
    private int quantity;
    private float unitPrice;
    private float unitWeight;
}
```

我们已经定义了实体，但还需要向应用程序的其他部分公开一些API，让我们创建CustomerOrderService类：

```java
public class CustomerOrderService implements OrderService {
    public static final String EVENT_ORDER_READY_FOR_SHIPMENT = "OrderReadyForShipmentEvent";

    private CustomerOrderRepository orderRepository;
    private EventBus eventBus;

    @Override
    public void placeOrder(CustomerOrder order) {
        this.orderRepository.saveCustomerOrder(order);
        Map<String, String> payload = new HashMap<>();
        payload.put("order_id", String.valueOf(order.getOrderId()));
        ApplicationEvent event = new ApplicationEvent(payload) {
            @Override
            public String getType() {
                return EVENT_ORDER_READY_FOR_SHIPMENT;
            }
        };
        this.eventBus.publish(event);
    }
}
```

这里，我们需要强调一些要点。placeOrder方法负责处理客户订单，**订单处理完成后，事件会发布到EventBus**，我们将在下一章讨论事件驱动的通信。此服务提供了OrderService接口的默认实现：

```java
public interface OrderService extends ApplicationService {
    void placeOrder(CustomerOrder order);

    void setOrderRepository(CustomerOrderRepository orderRepository);
}
```

此外，此服务需要CustomerOrderRepository来保存订单：

```java
public interface CustomerOrderRepository {
    void saveCustomerOrder(CustomerOrder order);
}
```

重要的是，**这个接口不是在这个上下文中实现的，而是由基础设施模块提供的**，我们稍后会看到。

### 2.3 运输上下文

现在，让我们定义运输上下文。它也很简单，包含三个实体：Parcel、PackageItem和ShippableOrder。

![](/assets/images/2025/ddd/javamodulesdddboundedcontexts03.png)

让我们从ShippableOrder实体开始：

```java
public class ShippableOrder {
    private int orderId;
    private String address;
    private List<PackageItem> packageItems;
}
```

在这种情况下，实体不包含paymentMethod字段，这是因为在我们的运输上下文中，我们不关心使用哪种付款方式，运输上下文只负责处理订单的发货。

此外，Parcel实体特定于运输环境：

```java
public class Parcel {
    private int orderId;
    private String address;
    private String trackingId;
    private List<PackageItem> packageItems;

    public float calculateTotalWeight() {
        return packageItems.stream().map(PackageItem::getWeight)
                .reduce(0F, Float::sum);
    }

    public boolean isTaxable() {
        return calculateEstimatedValue() > 100;
    }

    public float calculateEstimatedValue() {
        return packageItems.stream().map(PackageItem::getWeight)
                .reduce(0F, Float::sum);
    }
}
```

我们可以看到，它还包含具体的业务方法，并充当聚合根。

最后，让我们定义ParcelShippingService：

```java
public class ParcelShippingService implements ShippingService {
    public static final String EVENT_ORDER_READY_FOR_SHIPMENT = "OrderReadyForShipmentEvent";
    private ShippingOrderRepository orderRepository;
    private EventBus eventBus;
    private Map<Integer, Parcel> shippedParcels = new HashMap<>();

    @Override
    public void shipOrder(int orderId) {
        Optional<ShippableOrder> order = this.orderRepository.findShippableOrder(orderId);
        order.ifPresent(completedOrder -> {
            Parcel parcel = new Parcel(completedOrder.getOrderId(), completedOrder.getAddress(),
                    completedOrder.getPackageItems());
            if (parcel.isTaxable()) {
                // Calculate additional taxes
            }
            // Ship parcel
            this.shippedParcels.put(completedOrder.getOrderId(), parcel);
        });
    }

    @Override
    public void listenToOrderEvents() {
        this.eventBus.subscribe(EVENT_ORDER_READY_FOR_SHIPMENT, new EventSubscriber() {
            @Override
            public <E extends ApplicationEvent> void onEvent(E event) {
                shipOrder(Integer.parseInt(event.getPayloadValue("order_id")));
            }
        });
    }

    @Override
    public Optional<Parcel> getParcelByOrderId(int orderId) {
        return Optional.ofNullable(this.shippedParcels.get(orderId));
    }
}
```

该服务同样使用ShippingOrderRepository通过ID获取订单，**更重要的是，它订阅了由另一个上下文发布的OrderReadyForShipmentEvent事件**。当此事件发生时，服务会应用一些规则并发送订单。为了简单起见，我们将已发货的订单存储在[HashMap](https://www.baeldung.com/java-hashmap)中。

## 3. 上下文映射

到目前为止，我们定义了两个上下文，但是，我们尚未在它们之间建立任何明确的关系。为此，DDD提出了“上下文映射”的概念。**上下文映射是对系统中不同上下文之间关系的可视化描述**，该映射展示了不同部分如何共存并构成领域。

有界上下文之间主要有5种关系：

- 伙伴关系：两种环境之间的一种关系，通过合作使两个团队朝着相互依赖的目标保持一致
- 共享内核：一种将多个上下文的公共部分提取到另一个上下文/模块以减少代码重复的关系
- 客户-供应商：两种环境之间的连接，其中一个环境(上游)产生数据，另一个环境(下游)消费数据。在这种关系中，双方都希望建立尽可能最佳的沟通
- 顺从者：这种关系也存在上游和下游，但下游始终遵循上游的API
- 防腐层：这种关系广泛用于遗留系统，使其适应新的架构，并逐步从遗留代码库迁移。防腐层充当适配器，[转换](https://www.baeldung.com/hexagonal-architecture-ddd-spring)来自上游的数据，并防止意外更改。

在我们的具体示例中，我们将使用共享内核关系。我们不会以纯粹的形式定义它，但它主要充当系统中事件的中介。

因此，SharedKernel模块不会包含任何具体的实现，只包含接口。

让我们从EventBus接口开始：

```java
public interface EventBus {
    <E extends ApplicationEvent> void publish(E event);

    <E extends ApplicationEvent> void subscribe(String eventType, EventSubscriber subscriber);

    <E extends ApplicationEvent> void unsubscribe(String eventType, EventSubscriber subscriber);
}
```

该接口稍后将在我们的基础设施模块中实现。

接下来，我们创建一个具有默认方法的基本服务接口来支持事件驱动的通信：

```java
public interface ApplicationService {

    default <E extends ApplicationEvent> void publishEvent(E event) {
        EventBus eventBus = getEventBus();
        if (eventBus != null) {
            eventBus.publish(event);
        }
    }

    default <E extends ApplicationEvent> void subscribe(String eventType, EventSubscriber subscriber) {
        EventBus eventBus = getEventBus();
        if (eventBus != null) {
            eventBus.subscribe(eventType, subscriber);
        }
    }

    default <E extends ApplicationEvent> void unsubscribe(String eventType, EventSubscriber subscriber) {
        EventBus eventBus = getEventBus();
        if (eventBus != null) {
            eventBus.unsubscribe(eventType, subscriber);
        }
    }

    EventBus getEventBus();

    void setEventBus(EventBus eventBus);
}
```

因此，有界上下文中的服务接口扩展了此接口以具有常见的事件相关功能。

## 4. Java 9模块化

现在，是时候探索Java 9模块系统如何支持定义的应用程序结构了。

**Java平台模块系统(JPMS)鼓励构建更可靠、封装性更强的模块**。因此，这些特性有助于隔离上下文并建立清晰的边界。

让我们看看最终的模块图：

![](/assets/images/2025/ddd/javamodulesdddboundedcontexts04.png)

### 4.1 共享内核模块

让我们从SharedKernel模块开始，它不依赖其他模块。因此，module-info.java如下所示：

```java
module cn.tuyucheng.taketoday.dddmodules.sharedkernel {
    exports cn.tuyucheng.taketoday.dddmodules.sharedkernel.events;
    exports cn.tuyucheng.taketoday.dddmodules.sharedkernel.service;
}
```

我们导出模块接口，以便其他模块可以使用它们。

### 4.2 OrderContext模块

接下来，让我们将焦点转移到OrderContext模块，它只需要在SharedKernel模块中定义的接口：

```java
module cn.tuyucheng.taketoday.dddmodules.ordercontext {
    requires cn.tuyucheng.taketoday.dddmodules.sharedkernel;
    exports cn.tuyucheng.taketoday.dddmodules.ordercontext.service;
    exports cn.tuyucheng.taketoday.dddmodules.ordercontext.model;
    exports cn.tuyucheng.taketoday.dddmodules.ordercontext.repository;
    provides cn.tuyucheng.taketoday.dddmodules.ordercontext.service.OrderService with cn.tuyucheng.taketoday.dddmodules.ordercontext.service.CustomerOrderService;
}
```

另外，我们可以看到该模块导出了OrderService接口的默认实现。

### 4.3 ShippingContext模块

与上一个模块类似，让我们创建ShippingContext模块定义文件：

```java
module cn.tuyucheng.taketoday.dddmodules.shippingcontext {
    requires cn.tuyucheng.taketoday.dddmodules.sharedkernel;
    exports cn.tuyucheng.taketoday.dddmodules.shippingcontext.service;
    exports cn.tuyucheng.taketoday.dddmodules.shippingcontext.model;
    exports cn.tuyucheng.taketoday.dddmodules.shippingcontext.repository;
    provides cn.tuyucheng.taketoday.dddmodules.shippingcontext.service.ShippingService with cn.tuyucheng.taketoday.dddmodules.shippingcontext.service.ParcelShippingService;
}
```

以同样的方式，我们导出ShippingService接口的默认实现。

### 4.4 基础设施模块

现在是时候描述基础设施模块了，该模块包含已定义接口的实现细节，我们将首先为EventBus接口创建一个简单的实现：

```java
public class SimpleEventBus implements EventBus {
    private final Map<String, Set<EventSubscriber>> subscribers = new ConcurrentHashMap<>();

    @Override
    public <E extends ApplicationEvent> void publish(E event) {
        if (subscribers.containsKey(event.getType())) {
            subscribers.get(event.getType())
                    .forEach(subscriber -> subscriber.onEvent(event));
        }
    }

    @Override
    public <E extends ApplicationEvent> void subscribe(String eventType, EventSubscriber subscriber) {
        Set<EventSubscriber> eventSubscribers = subscribers.get(eventType);
        if (eventSubscribers == null) {
            eventSubscribers = new CopyOnWriteArraySet<>();
            subscribers.put(eventType, eventSubscribers);
        }
        eventSubscribers.add(subscriber);
    }

    @Override
    public <E extends ApplicationEvent> void unsubscribe(String eventType, EventSubscriber subscriber) {
        if (subscribers.containsKey(eventType)) {
            subscribers.get(eventType).remove(subscriber);
        }
    }
}
```

接下来，我们需要实现CustomerOrderRepository和ShippingOrderRepository接口。**大多数情况下，Order实体会存储在同一张表中，但在有界上下文中用作不同的实体模型**。

单个实体包含来自业务领域不同部分或低级数据库映射的混合代码是很常见的，在我们的实现中，我们根据有界上下文拆分了实体：CustomerOrder和ShippableOrder。

首先，让我们创建一个代表整个持久模型的类：

```java
public static class PersistenceOrder {
    public int orderId;
    public String paymentMethod;
    public String address;
    public List<OrderItem> orderItems;

    public static class OrderItem {
        public int productId;
        public float unitPrice;
        public float itemWeight;
        public int quantity;
    }
}
```

我们可以看到此类包含CustomerOrder和ShippableOrder实体的所有字段。

为了简单起见，让我们模拟一个内存数据库：

```java
public class InMemoryOrderStore implements CustomerOrderRepository, ShippingOrderRepository {
    private Map<Integer, PersistenceOrder> ordersDb = new HashMap<>();

    @Override
    public void saveCustomerOrder(CustomerOrder order) {
        this.ordersDb.put(order.getOrderId(), new PersistenceOrder(order.getOrderId(),
                order.getPaymentMethod(),
                order.getAddress(),
                order
                        .getOrderItems()
                        .stream()
                        .map(orderItem ->
                                new PersistenceOrder.OrderItem(orderItem.getProductId(),
                                        orderItem.getQuantity(),
                                        orderItem.getUnitWeight(),
                                        orderItem.getUnitPrice()))
                        .collect(Collectors.toList())
        ));
    }

    @Override
    public Optional<ShippableOrder> findShippableOrder(int orderId) {
        if (!this.ordersDb.containsKey(orderId)) return Optional.empty();
        PersistenceOrder orderRecord = this.ordersDb.get(orderId);
        return Optional.of(
                new ShippableOrder(orderRecord.orderId, orderRecord.orderItems
                        .stream().map(orderItem -> new PackageItem(orderItem.productId,
                                orderItem.itemWeight,
                                orderItem.quantity * orderItem.unitPrice)
                        ).collect(Collectors.toList())));
    }
}
```

在这里，我们通过将持久模型转换为适当的类型或从适当的类型转换来持久化和检索不同类型的实体。

最后，让我们创建模块定义：

```java
module cn.tuyucheng.taketoday.dddmodules.infrastructure {
    requires transitive cn.tuyucheng.taketoday.dddmodules.sharedkernel;
    requires transitive cn.tuyucheng.taketoday.dddmodules.ordercontext;
    requires transitive cn.tuyucheng.taketoday.dddmodules.shippingcontext;
    provides cn.tuyucheng.taketoday.dddmodules.sharedkernel.events.EventBus with cn.tuyucheng.taketoday.dddmodules.infrastructure.events.SimpleEventBus;
    provides cn.tuyucheng.taketoday.dddmodules.ordercontext.repository.CustomerOrderRepository with cn.tuyucheng.taketoday.dddmodules.infrastructure.db.InMemoryOrderStore;
    provides cn.tuyucheng.taketoday.dddmodules.shippingcontext.repository.ShippingOrderRepository with cn.tuyucheng.taketoday.dddmodules.infrastructure.db.InMemoryOrderStore;
}
```

使用provides子句，我们提供在其他模块中定义的一些接口的实现。

此外，该模块充当依赖项的聚合器，因此我们使用了传递关键字require。**这样，需要基础设施模块的模块将传递性地获取所有这些依赖项**。

### 4.5 主模块

最后，让我们定义一个模块作为我们应用程序的入口点：

```java
module cn.tuyucheng.taketoday.dddmodules.mainapp {
    uses cn.tuyucheng.taketoday.dddmodules.sharedkernel.events.EventBus;
    uses cn.tuyucheng.taketoday.dddmodules.ordercontext.service.OrderService;
    uses cn.tuyucheng.taketoday.dddmodules.ordercontext.repository.CustomerOrderRepository;
    uses cn.tuyucheng.taketoday.dddmodules.shippingcontext.repository.ShippingOrderRepository;
    uses cn.tuyucheng.taketoday.dddmodules.shippingcontext.service.ShippingService;
    requires transitive cn.tuyucheng.taketoday.dddmodules.infrastructure;
}
```

由于我们刚刚在基础设施模块上设置了传递依赖关系，因此我们不需要在这里明确要求它们。

另一方面，我们使用uses关键字列出这些依赖项。uses子句指示ServiceLoader(我们将在下一章中学习)该模块想要使用这些接口，但是，**它并不要求在编译时就提供实现**。

## 5. 运行应用程序

最后，我们将利用[Maven](https://www.baeldung.com/maven)来构建我们的项目，这使得模块的使用更加容易。

### 5.1 项目结构

我们的项目包含[五个模块和一个父模块](https://www.baeldung.com/maven-multi-module-project-java-jpms)，我们来看看我们的项目结构：

```text
ddd-modules (the root directory)
pom.xml
|-- infrastructure
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- cn.tuyucheng.taketoday.dddmodules.infrastructure
    pom.xml
|-- mainapp
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- cn.tuyucheng.taketoday.dddmodules.mainapp
    pom.xml
|-- ordercontext
    |-- src
        |-- main
            | -- java
            module-info.java
            |--cn.tuyucheng.taketoday.dddmodules.ordercontext
    pom.xml
|-- sharedkernel
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- cn.tuyucheng.taketoday.dddmodules.sharedkernel
    pom.xml
|-- shippingcontext
    |-- src
        |-- main
            | -- java
            module-info.java
            |-- cn.tuyucheng.taketoday.dddmodules.shippingcontext
    pom.xml
```

### 5.2 主应用

到目前为止，除了主应用程序之外，我们已经有了所有内容，因此让我们定义main方法：

```java
public static void main(String[] args) {
    Map<Class<?>, Object> container = createContainer();
    OrderService orderService = (OrderService) container.get(OrderService.class);
    ShippingService shippingService = (ShippingService) container.get(ShippingService.class);
    shippingService.listenToOrderEvents();

    CustomerOrder customerOrder = new CustomerOrder();
    int orderId = 1;
    customerOrder.setOrderId(orderId);
    List<OrderItem> orderItems = new ArrayList<OrderItem>();
    orderItems.add(new OrderItem(1, 2, 3, 1));
    orderItems.add(new OrderItem(2, 1, 1, 1));
    orderItems.add(new OrderItem(3, 4, 11, 21));
    customerOrder.setOrderItems(orderItems);
    customerOrder.setPaymentMethod("PayPal");
    customerOrder.setAddress("Full address here");
    orderService.placeOrder(customerOrder);

    if (orderId == shippingService.getParcelByOrderId(orderId).get().getOrderId()) {
        System.out.println("Order has been processed and shipped successfully");
    }
}
```

让我们简要讨论一下main方法，在此方法中，我们使用先前定义的服务模拟了一个简单的客户订单流程。首先，我们创建了包含三件商品的订单，并提供了必要的配送和付款信息。接下来，我们提交了订单，最后检查订单是否已发货并成功处理。

但是我们是如何获取所有依赖项的？为什么createContainer方法会返回Map<Class<?\>,Object\>？让我们仔细看看这个方法。

### 5.3 使用ServiceLoader进行依赖注入

在这个项目中，我们没有任何[Spring IoC](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)依赖，因此，我们将使用[ServiceLoader API](https://www.baeldung.com/java-spi#4-serviceloader)来发现服务的实现。这并不是一个新功能-ServiceLoader API本身自Java 6以来就已经存在了。

我们可以通过调用ServiceLoader类的静态load方法来获取加载器实例，**load方法返回Iterable类型，以便我们可以迭代已发现的实现**。

现在，让我们应用加载器来解决我们的依赖关系：

```java
public static Map<Class<?>, Object> createContainer() {
    EventBus eventBus = ServiceLoader.load(EventBus.class).findFirst().get();

    CustomerOrderRepository customerOrderRepository = ServiceLoader.load(CustomerOrderRepository.class)
            .findFirst().get();
    ShippingOrderRepository shippingOrderRepository = ServiceLoader.load(ShippingOrderRepository.class)
            .findFirst().get();

    ShippingService shippingService = ServiceLoader.load(ShippingService.class).findFirst().get();
    shippingService.setEventBus(eventBus);
    shippingService.setOrderRepository(shippingOrderRepository);
    OrderService orderService = ServiceLoader.load(OrderService.class).findFirst().get();
    orderService.setEventBus(eventBus);
    orderService.setOrderRepository(customerOrderRepository);

    HashMap<Class<?>, Object> container = new HashMap<>();
    container.put(OrderService.class, orderService);
    container.put(ShippingService.class, shippingService);

    return container;
}
```

这里，**我们为每个需要的接口调用静态load方法，该方法每次都会创建一个新的加载器实例**。因此，它不会缓存已经解析的依赖项，而是每次都会创建新的实例。

通常，服务实例可以通过两种方式创建，要么服务实现类必须具有一个公共的无参数构造函数，要么必须使用静态提供程序方法。

因此，我们的大多数服务都提供了无参数的构造函数和依赖项的Setter方法。但是，正如我们已经看到的，InMemoryOrderStore类实现了两个接口：CustomerOrderRepository和ShippingOrderRepository。

但是，如果我们使用load方法请求每个接口，我们将获得不同的InMemoryOrderStore实例。这不是我们想要的行为，所以让我们使用提供程序方法来缓存实例：

```java
public class InMemoryOrderStore implements CustomerOrderRepository, ShippingOrderRepository {
    private volatile static InMemoryOrderStore instance = new InMemoryOrderStore();

    public static InMemoryOrderStore provider() {
        return instance;
    }
}
```

我们应用了[Singleton模式](https://www.baeldung.com/java-singleton)来缓存InMemoryOrderStore类的单个实例并从提供程序方法返回它。

如果服务提供者声明了provider方法，则ServiceLoader会调用此方法来获取服务实例。否则，它将尝试通过[反射](https://www.baeldung.com/java-reflection)使用无参数构造函数来创建实例。因此，我们可以更改服务提供者机制，而不会影响createContainer方法。

最后，我们通过Setter为服务提供已解析的依赖关系并返回已配置的服务。

最后，我们可以运行该应用程序。

## 6. 总结

在本文中，我们讨论了一些关键的DDD概念：有界上下文、通用语言和上下文映射。虽然将系统划分为有界上下文有很多好处，但同时也没有必要在任何地方都应用这种方法。

接下来，我们了解了如何使用Java 9模块系统以及有界上下文来创建强封装模块。

此外，我们还介绍了用于发现依赖项的默认ServiceLoader机制。