---
layout: post
title:  Reactor WebFlux与虚拟线程
category: spring-reactive
copyright: spring-reactive
excerpt: Spring Reactive
---

## 1. 概述

在本教程中，我们将比较Java 19的[虚拟线程](https://www.baeldung.com/java-virtual-thread-vs-thread)与Project Reactor的[Webflux](https://www.baeldung.com/spring-webflux)。我们首先回顾每种方法的基本工作原理，然后分析它们的优缺点。

我们将首先探索响应式框架的优势，并了解WebFlux为何仍然有价值。之后，我们将讨论每个请求一个线程的方法，并重点介绍虚拟线程可能是更好选择的场景。

## 2. 代码示例

对于本文中的代码示例，我们假设我们正在开发电子商务应用程序的后端，我们将重点介绍负责计算和发布添加到购物车中的商品价格的函数：

```java
class ProductService {
    private final String PRODUCT_ADDED_TO_CART_TOPIC = "product-added-to-cart";

    private final ProductRepository repository;
    private final DiscountService discountService;
    private final KafkaTemplate<String, ProductAddedToCartEvent> kafkaTemplate;

    // constructor

    public void addProductToCart(String productId, String cartId) {
        Product product = repository.findById(productId)
                .orElseThrow(() -> new IllegalArgumentException("not found!"));

        Price price = product.basePrice();
        if (product.category().isEligibleForDiscount()) {
            BigDecimal discount = discountService.discountForProduct(productId);
            price.setValue(price.getValue().subtract(discount));
        }

        var event = new ProductAddedToCartEvent(productId, price.getValue(), price.getCurrency(), cartId);
        kafkaTemplate.send(PRODUCT_ADDED_TO_CART_TOPIC, cartId, event);
    }
}
```

如我们所见，我们首先使用MongoRepository从MongoDB数据库中检索产品。检索后，我们确定产品是否符合折扣条件。如果符合条件，我们使用DiscountService执行HTTP请求以确定产品是否有任何可用折扣。

最后，我们计算产品的最终价格。完成后，我们会发送一条[Kafka](https://www.baeldung.com/spring-kafka)消息，其中包含productId、cartId和计算出的价格。

## 3. WebFlux

**WebFlux是一个用于构建异步、非阻塞和事件驱动应用程序的框架**。它采用响应式编程原则，利用Flux和Mono类型来处理异步通信的复杂性。这些类型实现了[发布者-订阅者](https://www.baeldung.com/cs/publisher-subscriber-model)设计模式，将数据的消费者和生产者解耦。

### 3.1 响应式库

**Spring生态系统中的许多模块都与WebFlux集成，以实现响应式编程**。让我们使用其中一些模块，同时将代码重构为响应式范式。

例如，我们可以将MongoRepository切换到[ReactiveMongoRepository](https://www.baeldung.com/spring-data-mongodb-reactive)，此更改意味着我们必须使用Mono<Product\>而不是Optional<Product\>：

```java
Mono<Product> product = repository.findById(productId)
    .switchIfEmpty(Mono.error(() -> new IllegalArgumentException("not found!")));
```

类似地，我们可以将ProductService改为异步和非阻塞的。例如，我们可以让它使用[WebClient](https://www.baeldung.com/spring-5-webclient)执行HTTP请求，然后以Mono<BigDecimal\>的形式返回折扣：

```java
Mono<BigDecimal> discount = discountService.discountForProduct(productId);
```

### 3.2 不变性

**在函数式和响应式编程范式中，不变性始终优于可变数据**。我们的初始方法涉及使用Setter更改Price的值。但是，随着我们转向响应式方法，让我们重构Price对象并使其不可变。

例如，我们可以引入一个专用的方法来应用折扣并生成一个新的Price实例，而不是修改现有的实例：

```java
record Price(BigDecimal value, String currency) {
    public Price applyDiscount(BigDecimal discount) {
        return new Price(value.subtract(discount), currency);
    }
}
```

现在，我们可以使用WebFlux的map()方法根据折扣计算新价格：

```java
Mono<Price> price = discountService.discountForProduct(productId)
    .map(discount -> price.applyDiscount(discount));
```

此外，我们甚至可以在这里使用方法引用，以保持代码紧凑：

```java
Mono<Price> price = discountService.discountForProduct(productId).map(price::applyDiscount);
```

### 3.3 函数管道

**Mono和Flux通过map()和flatMap()等方法遵循函子和[monad](https://www.baeldung.com/java-monads)模式，这使我们能够将用例描述为对不可变数据进行转换的管道**。

让我们尝试确定我们的用例所需的转换：

- productId开始
- 将其转化为Product
- 使用Product来计算Price
- 使用Price来创建一个event
- 最后，将event发布到消息队列中

现在，让我们重构代码来反映这个函数链：

```java
void addProductToCart(String productId, String cartId) {
    Mono<Product> productMono = repository.findById(productId)
            .switchIfEmpty(Mono.error(() -> new IllegalArgumentException("not found!")));

    Mono<Price> priceMono = productMono.flatMap(product -> {
        if (product.category().isEligibleForDiscount()) {
            return discountService.discountForProduct(productId)
                    .map(product.basePrice()::applyDiscount);
        }
        return Mono.just(product.basePrice());
    });

    Mono<ProductAddedToCartEvent> eventMono = priceMono.map(
            price -> new ProductAddedToCartEvent(productId, price.value(), price.currency(), cartId));

    eventMono.subscribe(event -> kafkaTemplate.send(PRODUCT_ADDED_TO_CART_TOPIC, cartId, event));
}
```

现在，让我们内联局部变量以保持代码紧凑。此外，让我们提取一个用于计算价格的函数，并在flatMap()内部使用它：

```java
void addProductToCart(String productId, String cartId) {
    repository.findById(productId)
            .switchIfEmpty(Mono.error(() -> new IllegalArgumentException("not found!")))
            .flatMap(this::computePrice)
            .map(price -> new ProductAddedToCartEvent(productId, price.value(), price.currency(), cartId))
            .subscribe(event -> kafkaTemplate.send(PRODUCT_ADDED_TO_CART_TOPIC, cartId, event));
}

Mono<Price> computePrice(Product product) {
    if (product.category().isEligibleForDiscount()) {
        return discountService.discountForProduct(product.id())
                .map(product.basePrice()::applyDiscount);
    }
    return Mono.just(product.basePrice());
}
```

## 4. 虚拟线程

**虚拟线程是通过[Project Loom](https://www.baeldung.com/openjdk-project-loom)在Java中引入的，作为并行处理的替代解决方案。它们是由Java虚拟机([JVM](https://www.baeldung.com/jvm-vs-jre-vs-jdk#jvm))管理的轻量级用户模式线程**。因此，它们特别适合I/O操作，传统线程可能会花费大量时间等待外部资源。

与异步或响应式解决方案相比，虚拟线程使我们能够继续使用每个请求一个线程的处理模型。换句话说，我们可以继续按顺序编写代码，而无需混合业务逻辑和响应式API。

### 4.1 虚拟线程

有几种方法可以利用虚拟线程来执行我们的代码，**对于单个方法(例如上例中演示的方法)，我们可以使用startVirtualThread()**。此静态方法最近添加到Thread API中，并在新的虚拟线程上执行Runnable：

```java
public void addProductToCart(String productId, String cartId) {
    Thread.startVirtualThread(() -> computePriceAndPublishMessage(productId, cartId));
}

private void computePriceAndPublishMessage(String productId, String cartId) {
    // ...
}
```

**或者，我们可以使用新的静态工厂方法Executors.newVirtualThreadPerTaskExecutor()创建依赖于虚拟线程的[ExecutorService](https://www.baeldung.com/java-executor-service-tutorial)**。此外，对于使用Spring 6和Spring Boot 3的应用程序，我们可以利用新的Executor并[将Spring配置为支持虚拟线程](https://www.baeldung.com/spring-6-virtual-threads)而不是平台线程。

### 4.2 兼容性

虚拟线程通过使用更传统的同步编程模型来简化代码。因此，我们可以按顺序编写代码，类似于阻塞I/O操作，而不必担心显式的响应式构造。

**此外，我们可以无缝地从常规单线程代码切换到虚拟线程，几乎不需要任何改动**。例如，在我们前面的例子中，我们只需要使用静态工厂方法startVirtualThread()创建一个虚拟线程并在其中执行逻辑：

```java
void addProductToCart(String productId, String cartId) {
    Thread.startVirtualThread(() -> computePriceAndPublishMessage(productId, cartId));
}

void computePriceAndPublishMessage(String productId, String cartId) {
    Product product = repository.findById(productId)
            .orElseThrow(() -> new IllegalArgumentException("not found!"));

    Price price = computePrice(productId, product);

    var event = new ProductAddedToCartEvent(productId, price.value(), price.currency(), cartId);
    kafkaTemplate.send(PRODUCT_ADDED_TO_CART_TOPIC, cartId, event);
}

Price computePrice(String productId, Product product) {
    if (product.category().isEligibleForDiscount()) {
        BigDecimal discount = discountService.discountForProduct(productId);
        return product.basePrice().applyDiscount(discount);
    }
    return product.basePrice();
}
```

### 4.3 可读性

**使用每个请求一个线程的处理模型，可以更轻松地理解和推理业务逻辑，这可以减少与响应式编程范式相关的认知负担**。

换句话说，虚拟线程使我们能够将技术问题与业务逻辑完全分开。因此，它消除了我们在实现业务用例时对外部API的需求。

## 5. 总结

在本文中，我们比较了两种不同的并发和异步处理方法。我们首先分析了Reactor的WebFlux项目和响应式编程范式，我们发现这种方法有利于不可变对象和函数管道。

之后，我们讨论了虚拟线程及其与旧代码库的出色兼容性，从而可以顺利过渡到非阻塞代码。此外，它们还具有将业务逻辑与基础架构代码和其他技术问题清晰地分开的额外优势。