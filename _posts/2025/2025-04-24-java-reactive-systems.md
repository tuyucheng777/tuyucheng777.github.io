---
layout: post
title:  Java中的响应式系统
category: reactor
copyright: reactor
excerpt: Reactor
---

## 1. 简介

在本教程中，我们将学习使用Spring和其他工具和框架在Java中创建响应式系统的基础知识。

在此过程中，我们将讨论响应式编程如何成为创建响应式系统的驱动力，这将帮助我们理解创建响应式系统的基本原理以及它在此过程中启发的不同规范、库和标准。

## 2. 什么是响应式系统？

在过去的几十年里，技术领域经历了数次颠覆，彻底改变了我们对科技价值的认知，互联网出现之前的计算机世界永远无法想象它将改变我们今天的方式方法。

随着互联网逐渐普及到大众，以及它所承诺的不断发展的体验，应用程序架构师需要时刻保持警惕以满足需求。

从根本上说，这意味着我们再也不能像以前那样设计应用程序了，**高度响应的应用程序不再是奢侈品，而是必需品**。

这也是在面临随机故障和不可预测负载的情况下，当务之急不仅仅是获得正确的结果，更要快速获得！实现我们承诺的卓越用户体验至关重要。

这就是为什么我们需要一种能够提供响应式系统的架构风格。

### 2.1 响应式宣言

**早在2013年，由Jonas Boner领导的开发团队就共同制定了一套核心原则，并将其写入名为[响应式宣言(Reactive Manifesto)](https://www.reactivemanifesto.org/)的文档中**，这为创建响应式系统的架构风格奠定了基础。从那时起，这份宣言就引起了开发者社区的广泛关注。

基本上，本文档规定了响应式系统应**具有灵活性、松耦合性和可扩展性**。这使得此类系统易于开发、容错性强，最重要的是响应速度快，而这正是卓越用户体验的基础。

那么这个秘诀是什么呢？好吧，它并不是什么秘密，这份宣言定义了响应式系统的基本特征或原则：

-   响应式：响应式系统应提供快速且一致的响应时间，从而提供一致的服务质量。
-   弹性：响应式系统应该通过复制和隔离在随机故障的情况下保持响应。
-   伸缩性：这样的系统应该通过具有成本效益的可扩展性在不可预测的工作负载下保持响应。
-   消息驱动：它应该依赖于系统组件之间的异步消息传递。

这些原则听起来简单合理，但在复杂的企业架构中实现起来却并非易事。在本文中，我们将基于这些原则，用Java开发一个示例系统。

## 3. 什么是响应式编程？

在继续之前，我们有必要先了解一下响应式编程和响应式系统之间的区别。我们经常使用这两个术语，但很容易混淆。正如我们之前所见，响应式系统是特定架构风格的产物。

相比之下，**响应式编程是一种专注于开发异步和非阻塞组件的编程范式**。响应式编程的核心是一个我们可以观察、响应甚至施加背压的数据流，这带来了非阻塞执行，以及在更少的执行线程下实现更好的可扩展性。

当然，这并不意味着响应式系统和响应式编程是互相排斥的。事实上，响应式编程是实现响应式系统的重要一步，但它并非一切。

### 3.1 响应流

[Reactive Streams](http://www.reactive-streams.org/)是一项社区倡议，始于2013年，旨在**为具有非阻塞背压的异步流处理提供标准**，目标是定义一组可以描述必要操作和实体的接口、方法和协议。

从那时起，多种编程语言中出现了符合响应式流规范的实现，其中包括Akka Streams、Ratpack和Vert.x等等。

### 3.2 Java的响应式库

响应式流背后的最初目标之一是最终作为官方Java标准库包含在内，因此，响应流规范在语义上等同于Java 9中引入的Java Flow库。

除此之外，还有一些流行的选择可以在Java中实现响应式编程：

-   [Reactive Extensions](http://reactivex.io/)：通常称为ReactiveX，它们提供使用可观察流进行异步编程的API，它们适用于多种编程语言和平台，包括Java，在Java中被称为[RxJava](https://github.com/ReactiveX/RxJava)。
-   [Project Reactor](https://projectreactor.io/)：这是另一个响应式库，基于响应式流规范，旨在在JVM上构建非应用程序，它也恰好是[Spring生态系统](https://spring.io/reactive)中响应式堆栈的基础。

## 4. 一个简单的应用程序

在本教程中，我们将基于微服务架构开发一个前端极简的简单应用程序，该应用程序架构应包含足够的元素来构建一个响应式系统。

对于我们的应用程序，我们将采用端到端响应式编程以及其他模式和工具来实现响应式系统的基本特征。

### 4.1 架构

**我们将首先定义一个简单的应用程序架构，该架构不一定具有响应式系统的特征**。在此基础上，我们将进行必要的更改，以逐一实现这些特征。

首先，让我们从定义一个简单的架构开始：

![](/assets/images/2025/reactor/javareactivesystems01.png)

这是一个相当简单的架构，包含一系列微服务，用于实现一个可以下单的商业用例。它还提供了一个用户体验的前端，所有通信都以REST HTTP的方式进行。此外，每个微服务都在各自的数据库中管理其数据，这种做法被称为“每个服务一个数据库”。

我们将在接下来的小节中创建这个简单的应用程序，这将成为我们理解这种架构的谬误的基础，以及如何采用相关原则和实践，将其转变为响应式系统。

### 4.2 库存微服务

**库存微服务将负责管理产品列表及其当前库存，它还允许在处理订单时更改库存**，我们将结合使用[Spring Boot](https://www.baeldung.com/spring-boot)和MongoDB来开发此服务。

我们首先定义一个控制器来公开一些端点：

```java
@GetMapping
public List<Product> getAllProducts() {
    return productService.getProducts();
}
 
@PostMapping
public Order processOrder(@RequestBody Order order) {
    return productService.handleOrder(order);
}
 
@DeleteMapping
public Order revertOrder(@RequestBody Order order) {
    return productService.revertOrder(order);
}
```

并将定义一个服务来封装我们的业务逻辑：

```java
@Transactional
public Order handleOrder(Order order) {
    order.getLineItems()
            .forEach(l -> {
                Product> p = productRepository.findById(l.getProductId())
                        .orElseThrow(() -> new RuntimeException("Could not find the product: " + l.getProductId()));
                if (p.getStock() >= l.getQuantity()) {
                    p.setStock(p.getStock() - l.getQuantity());
                    productRepository.save(p);
                } else {
                    throw new RuntimeException("Product is out of stock: " + l.getProductId());
                }
            });
    return order.setOrderStatus(OrderStatus.SUCCESS);
}

@Transactional
public Order revertOrder(Order order) {
    order.getLineItems()
            .forEach(l -> {
                Product p = productRepository.findById(l.getProductId())
                        .orElseThrow(() -> new RuntimeException("Could not find the product: " + l.getProductId()));
                p.setStock(p.getStock() + l.getQuantity());
                productRepository.save(p);
            });
    return order.setOrderStatus(OrderStatus.SUCCESS);
}
```

请注意，**我们在事务中将实体保存**，这可确保在出现异常时不会出现不一致的状态。

除此之外，我们还必须定义域实体、Repository接口以及一切正常运行所需的一系列配置类。

但由于这些大多是样板文件，我们将避免对它们进行讨论，并且可以在本文最后一节提供的GitHub仓库中参考它们。

### 4.3 运输微服务

运输微服务也不会有太大的不同，它将**负责检查订单是否可以生成发货，并在可能的情况下创建发货**。

和以前一样，我们将定义一个控制器来公开一个端点：

```java
@PostMapping
public Order process(@RequestBody Order order) {
    return shippingService.handleOrder(order);
}
```

以及封装与订单发货相关的业务逻辑的服务：

```java
public Order handleOrder(Order order) {
    LocalDate shippingDate = null;
    if (LocalTime.now().isAfter(LocalTime.parse("10:00"))
            && LocalTime.now().isBefore(LocalTime.parse("18:00"))) {
        shippingDate = LocalDate.now().plusDays(1);
    } else {
        throw new RuntimeException("The current time is off the limits to place order.");
    }
    shipmentRepository.save(new Shipment()
            .setAddress(order.getShippingAddress())
            .setShippingDate(shippingDate));
    return order.setShippingDate(shippingDate)
            .setOrderStatus(OrderStatus.SUCCESS);
}
```

我们简单的运输服务只是检查下单的有效时间范围，和之前一样，我们避免讨论其余的样板代码。

### 4.4 订单微服务

最后，我们将定义一个订单微服务，除了其他功能外，它还将**负责创建新订单**。有趣的是，它还将充当协调器服务，与订单的库存服务和运输服务进行通信。

让我们用所需的端点定义我们的控制器：

```java
@PostMapping
public Order create(@RequestBody Order order) {
    Order processedOrder = orderService.createOrder(order);
    if (OrderStatus.FAILURE.equals(processedOrder.getOrderStatus())) {
        throw new RuntimeException("Order processing failed, please try again later.");
    }
    return processedOrder;
}

@GetMapping
public List<Order> getAll() {
    return orderService.getOrders();
}
```

并且，封装与订单相关的业务逻辑的服务：

```java
public Order createOrder(Order order) {
    boolean success = true;
    Order savedOrder = orderRepository.save(order);
    Order inventoryResponse = null;
    try {
        inventoryResponse = restTemplate.postForObject(
                inventoryServiceUrl, order, Order.class);
    } catch (Exception ex) {
        success = false;
    }
    Order shippingResponse = null;
    try {
        shippingResponse = restTemplate.postForObject(
                shippingServiceUrl, order, Order.class);
    } catch (Exception ex) {
        success = false;
        HttpEntity<Order> deleteRequest = new HttpEntity<>(order);
        ResponseEntity<Order> deleteResponse = restTemplate.exchange(
                inventoryServiceUrl, HttpMethod.DELETE, deleteRequest, Order.class);
    }
    if (success) {
        savedOrder.setOrderStatus(OrderStatus.SUCCESS);
        savedOrder.setShippingDate(shippingResponse.getShippingDate());
    } else {
        savedOrder.setOrderStatus(OrderStatus.FAILURE);
    }
    return orderRepository.save(savedOrder);
}

public List<Order> getOrders() {
    return orderRepository.findAll();
}
```

订单处理中，我们协调库存和配送服务的调用，但效果远非理想。**多微服务的分布式事务本身就是一个复杂的话题，超出了本文的讨论范围**。

但是，我们将在本文后面看到响应式系统如何在一定程度上避免对分布式事务的需求。

和以前一样，我们不会介绍其余的样板代码；但是，可以在GitHub仓库中引用这些代码。

### 4.5 前端

我们还将添加一个用户界面，该用户界面将基于Angular，是一个简单的单页应用程序。

我们需要**在Angular中创建一个简单的组件来处理订单的创建和获取**，特别重要的是调用API来创建订单的部分：

```javascript
createOrder() {
    let headers = new HttpHeaders({'Content-Type': 'application/json'});
    let options = {headers: headers}
    this.http.post('http://localhost:8080/api/orders', this.form.value, options)
        .subscribe(
            (response) => {
                this.response = response
            },
            (error) => {
                this.error = error
            }
        )
}
```

上面的代码片段**期望订单数据以表单形式捕获并在组件范围内可用**，Angular为使用[响应式和模板驱动的表单](https://angular.io/guide/forms-overview)创建简单到复杂的表单提供了出色的支持。

同样重要的是我们获取先前创建的订单的部分：

```javascript
getOrders() {
    this.previousOrders = this.http.get(''http://localhost:8080/api/orders'')
}
```

请注意**[Angular HTTP模块](https://angular.io/guide/http)本质上是异步的，因此返回RxJS Observables**，我们可以通过异步管道传递响应，在视图中处理它们：

```html
<div class="container" ngIf="previousOrders !== null">
    <h2>Your orders placed so far:</h2>
    <ul>
        <li ngFor="let order of previousOrders | async">
            <p>Order ID: {{ order.id }}, Order Status: {{order.orderStatus}}, Order Message: {{order.responseMessage}}</p>
        </li>
    </ul>
</div>
```

当然，Angular需要模板、样式和配置才能正常工作，但这些可以在GitHub仓库中找到。请注意，我们在这里将所有内容都打包在一个组件中，理想情况下我们不应该这样做。

但对于本文而言，这些问题不在讨论范围内。

### 4.6 部署应用程序

现在我们已经创建了应用程序的所有各个部分，那么我们应该如何部署它们呢？我们可以手动完成，但要注意，这很快就会变得繁琐。

在本文中，我们将使用[Docker Compose](https://www.baeldung.com/docker-compose)在Docker上构建和部署我们的应用程序，这需要我们在每个服务中添加一个标准的Dockerfile，并为整个应用程序创建一个Docker Compose文件。在运行**docker-compose up**之前，请确保所有模块都已构建，要构建模块，请使用以下命令：**mvn clean package spring-boot:repackage**。在docker-compose中，我们将拥有运行整个项目所需的所有依赖服务，例如Kafka和MongoDB。

下面是docker-compose.yml文件的样子：

```yaml
version: '3'
services:
    frontend:
        build: ./frontend
        ports:
            - "80:80"
    zookeeper:
        image: confluentinc/cp-zookeeper:latest
        environment:
            ZOOKEEPER_CLIENT_PORT: 2181
            ZOOKEEPER_TICK_TIME: 2000
        ports:
            - 22181:2181
    kafka:
        image: confluentinc/cp-kafka:latest
        container_name: kafka-broker
        depends_on:
            - zookeeper
        ports:
            - 29092:29092
        environment:
            KAFKA_BROKER_ID: 1
            KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
            KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://kafka-broker:9092,PLAINTEXT_HOST://localhost:29092
            KAFKA_LISTENER_SECURITY_PROTOCOL_MAP: PLAINTEXT:PLAINTEXT,PLAINTEXT_HOST:PLAINTEXT
            KAFKA_INTER_BROKER_LISTENER_NAME: PLAINTEXT
            KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    mongodb:
        container_name: mongo-db
        image: mongo:6.0
        volumes:
            - ~/mongo:/data/db
        ports:
            - "27017:27017"
        healthcheck:
            test: exit 0
    order-service:
        build: ./order-service
        ports:
            - "8080:8080"
        depends_on:
            mongodb:
                condition: service_healthy
    inventory-service:
        build: ./inventory-service
        ports:
            - "8081:8081"
        depends_on:
            mongodb:
                condition: service_healthy
    shipping-service:
        build: ./shipping-service
        ports:
            - "8082:8082"
        depends_on:
            mongodb:
                condition: service_healthy
```

这是Docker Compose中相当标准的服务定义，不需要特别注意。

### 4.7 这种架构的问题

现在我们已经有了一个包含多个相互交互的服务的简单应用程序，我们可以讨论一下这种架构中存在的问题。我们将在接下来的章节中尝试解决这些问题，最终将应用程序转变为响应式系统。

虽然此应用程序远非生产级软件，并且存在一些问题，但我们将**重点关注与响应式系统动机相关的问题**：

-   库存服务或运输服务的失败会产生连锁反应
-   对外部系统和数据库的调用本质上都是阻塞的
-   部署无法自动处理故障和负载波动

## 5. 响应式编程

**任何程序中的阻塞调用通常都会导致关键资源处于等待状态，等待结果发生**。这些情况包括数据库调用、Web服务调用以及文件系统调用。如果我们能够将执行线程从这种等待状态中解放出来，并提供一种机制，在结果可用时立即返回，那么资源利用率将会大大提高。

这就是采用响应式编程范式为我们带来的益处，虽然许多调用可以切换到响应式库，但并非所有调用都能做到这一点。幸运的是，Spring使我们更容易地使用MongoDB和REST API进行响应式编程：

![](/assets/images/2025/reactor/javareactivesystems02.png)

[Spring Data Mongo](https://www.baeldung.com/spring-data-mongodb-tutorial)支持通过MongoDB Reactive Streams Java驱动程序进行响应式访问，它提供ReactiveMongoTemplate和ReactiveMongoRepository，两者都具有广泛的映射功能。

[Spring WebFlux](https://www.baeldung.com/spring-webflux)为Spring提供响应式堆栈Web框架，支持非阻塞代码和Reactive Stream背压，它利用Reactor作为其响应式库。此外，它还提供WebClient，用于在响应式流背压下执行HTTP请求，它使用Reactor Netty作为HTTP客户端库。

### 5.1 库存服务

我们首先将更改端点以发出响应式发布者：

```java
@GetMapping
public Flux<Product> getAllProducts() {
    return productService.getProducts();
}
```

```java
@PostMapping
public Mono<Order> processOrder(@RequestBody Order order) {
    return productService.handleOrder(order);
}

@DeleteMapping
public Mono<Order> revertOrder(@RequestBody Order order) {
    return productService.revertOrder(order);
}
```

显然，我们还必须对服务进行必要的更改：

```java
@Transactional
public Mono<Order> handleOrder(Order order) {
    return Flux.fromIterable(order.getLineItems())
            .flatMap(l -> productRepository.findById(l.getProductId()))
            .flatMap(p -> {
                int q = order.getLineItems().stream()
                        .filter(l -> l.getProductId().equals(p.getId()))
                        .findAny().get()
                        .getQuantity();
                if (p.getStock() >= q) {
                    p.setStock(p.getStock() - q);
                    return productRepository.save(p);
                } else {
                    return Mono.error(new RuntimeException("Product is out of stock: " + p.getId()));
                }
            })
            .then(Mono.just(order.setOrderStatus("SUCCESS")));
}

@Transactional
public Mono<Order> revertOrder(Order order) {
    return Flux.fromIterable(order.getLineItems())
            .flatMap(l -> productRepository.findById(l.getProductId()))
            .flatMap(p -> {
                int q = order.getLineItems().stream()
                        .filter(l -> l.getProductId().equals(p.getId()))
                        .findAny().get()
                        .getQuantity();
                p.setStock(p.getStock() + q);
                return productRepository.save(p);
            })
            .then(Mono.just(order.setOrderStatus("SUCCESS")));
}
```

### 5.2 运输服务

同样，我们将更改运输服务的端点：

```java
@PostMapping
public Mono<Order> process(@RequestBody Order order) {
    return shippingService.handleOrder(order);
}
```

我们还将在服务中做出相应的更改以利用响应式编程：

```java
public Mono<Order> handleOrder(Order order) {
    return Mono.just(order)
            .flatMap(o -> {
                LocalDate shippingDate = null;
                if (LocalTime.now().isAfter(LocalTime.parse("10:00"))
                        && LocalTime.now().isBefore(LocalTime.parse("18:00"))) {
                    shippingDate = LocalDate.now().plusDays(1);
                } else {
                    return Mono.error(new RuntimeException("The current time is off the limits to place order."));
                }
                return shipmentRepository.save(new Shipment()
                        .setAddress(order.getShippingAddress())
                        .setShippingDate(shippingDate));
            })
            .map(s -> order.setShippingDate(s.getShippingDate())
                    .setOrderStatus(OrderStatus.SUCCESS));
}
```

### 5.3 订单服务

我们必须在订单服务的端点做出类似的更改：

```java
@PostMapping
public Mono<Order> create(@RequestBody Order order) {
    return orderService.createOrder(order)
            .flatMap(o -> {
                if (OrderStatus.FAILURE.equals(o.getOrderStatus())) {
                    return Mono.error(new RuntimeException("Order processing failed, please try again later. " + o.getResponseMessage()));
                } else {
                    return Mono.just(o);
                }
            });
}

@GetMapping
public Flux<Order> getAll() {
    return orderService.getOrders();
}
```

对服务的更改将更加复杂，因为我们必须使用Spring WebClient来调用库存和运输服务响应式端点：

```java
public Mono<Order> createOrder(Order order) {
    return Mono.just(order)
            .flatMap(orderRepository::save)
            .flatMap(o -> {
                return webClient.method(HttpMethod.POST)
                        .uri(inventoryServiceUrl)
                        .body(BodyInserters.fromValue(o))
                        .exchange();
            })
            .onErrorResume(err -> {
                return Mono.just(order.setOrderStatus(OrderStatus.FAILURE)
                        .setResponseMessage(err.getMessage()));
            })
            .flatMap(o -> {
                if (!OrderStatus.FAILURE.equals(o.getOrderStatus())) {
                    return webClient.method(HttpMethod.POST)
                            .uri(shippingServiceUrl)
                            .body(BodyInserters.fromValue(o))
                            .exchange();
                } else {
                    return Mono.just(o);
                }
            })
            .onErrorResume(err -> {
                return webClient.method(HttpMethod.POST)
                        .uri(inventoryServiceUrl)
                        .body(BodyInserters.fromValue(order))
                        .retrieve()
                        .bodyToMono(Order.class)
                        .map(o -> o.setOrderStatus(OrderStatus.FAILURE)
                                .setResponseMessage(err.getMessage()));
            })
            .map(o -> {
                if (!OrderStatus.FAILURE.equals(o.getOrderStatus())) {
                    return order.setShippingDate(o.getShippingDate())
                            .setOrderStatus(OrderStatus.SUCCESS);
                } else {
                    return order.setOrderStatus(OrderStatus.FAILURE)
                            .setResponseMessage(o.getResponseMessage());
                }
            })
            .flatMap(orderRepository::save);
}

public Flux<Order> getOrders() {
    return orderRepository.findAll();
}
```

**这种使用响应式API的编排并不容易，而且往往容易出错且难以调试**，下一节我们将探讨如何简化这一过程。

### 5.4 前端

既然我们的API能够实时处理事件，那么很自然地，我们也应该能够在前端利用这一点。幸运的是，Angular支持[EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)，**它是服务器发送事件(SSE)的接口**。

让我们看看如何将所有以前的订单作为事件流来提取和处理：

```javascript
getOrderStream() {
    return Observable.create((observer) => {
        let eventSource = new EventSource('http://localhost:8080/api/orders')
        eventSource.onmessage = (event) => {
            let json = JSON.parse(event.data)
            this.orders.push(json)
            this._zone.run(() => {
                observer.next(this.orders)
            })
        }
        eventSource.onerror = (error) => {
            if(eventSource.readyState === 0) {
                eventSource.close()
                this._zone.run(() => {
                    observer.complete()
                })
            } else {
                this._zone.run(() => {
                    observer.error('EventSource error: ' + error)
                })
            }
        }
    })
}
```

## 6. 消息驱动架构

我们要解决的第一个问题与服务间通信有关，目前，**这些通信是同步的，这带来了一些问题**，包括级联故障、复杂的编排和分布式事务等等。

解决这个问题的一个显而易见的方法是使这些通信异步化，**一个用于促进所有服务间通信的消息代理可以帮我们解决这个问题**。我们将使用Kafka作为消息代理，并使用[Spring Kafka](https://www.baeldung.com/spring-kafka)来生成和消费消息：

![](/assets/images/2025/reactor/javareactivesystems03.png)

我们将使用单个主题来生成和消费具有不同订单状态的订单消息，以便服务做出响应。

让我们看看每个服务需要如何改变。

### 6.1 库存服务

让我们首先定义库存服务的消息生产者：

```java
@Autowired
private KafkaTemplate<String, Order> kafkaTemplate;

public void sendMessage(Order order) {
    this.kafkaTemplate.send("orders", order);
}
```

接下来，我们必须为库存服务定义一个消息消费者，以对主题上的不同消息做出响应：

```java
@KafkaListener(topics = "orders", groupId = "inventory")
public void consume(Order order) throws IOException {
    if (OrderStatus.RESERVE_INVENTORY.equals(order.getOrderStatus())) {
        productService.handleOrder(order)
                .doOnSuccess(o -> {
                    orderProducer.sendMessage(order.setOrderStatus(OrderStatus.INVENTORY_SUCCESS));
                })
                .doOnError(e -> {
                    orderProducer.sendMessage(order.setOrderStatus(OrderStatus.INVENTORY_FAILURE)
                            .setResponseMessage(e.getMessage()));
                }).subscribe();
    } else if (OrderStatus.REVERT_INVENTORY.equals(order.getOrderStatus())) {
        productService.revertOrder(order)
                .doOnSuccess(o -> {
                    orderProducer.sendMessage(order.setOrderStatus(OrderStatus.INVENTORY_REVERT_SUCCESS));
                })
                .doOnError(e -> {
                    orderProducer.sendMessage(order.setOrderStatus(OrderStatus.INVENTORY_REVERT_FAILURE)
                            .setResponseMessage(e.getMessage()));
                }).subscribe();
    }
}
```

这也意味着我们现在可以安全地从控制器中删除一些冗余端点，这些更改足以在我们的应用程序中实现异步通信。

### 6.2 运输服务

运输服务的变化与我们之前对库存服务所做的改动相对相似，消息生产者相同，而消息消费者则特定于配送逻辑：

```java
@KafkaListener(topics = "orders", groupId = "shipping")
public void consume(Order order) throws IOException {
    if (OrderStatus.PREPARE_SHIPPING.equals(order.getOrderStatus())) {
        shippingService.handleOrder(order)
                .doOnSuccess(o -> {
                    orderProducer.sendMessage(order.setOrderStatus(OrderStatus.SHIPPING_SUCCESS)
                            .setShippingDate(o.getShippingDate()));
                })
                .doOnError(e -> {
                    orderProducer.sendMessage(order.setOrderStatus(OrderStatus.SHIPPING_FAILURE)
                            .setResponseMessage(e.getMessage()));
                }).subscribe();
    }
}
```

我们现在可以安全地删除控制器中的所有端点，因为我们不再需要它们。

### 6.3 订单服务

订单服务的变化会稍微复杂一些，因为这是我们之前进行所有编排的地方。

尽管如此，消息生产者保持不变，消息消费者则承担订单服务特定的逻辑：

```java
@KafkaListener(topics = "orders", groupId = "orders")
public void consume(Order order) throws IOException {
    if (OrderStatus.INITIATION_SUCCESS.equals(order.getOrderStatus())) {
        orderRepository.findById(order.getId())
                .map(o -> {
                    orderProducer.sendMessage(o.setOrderStatus(OrderStatus.RESERVE_INVENTORY));
                    return o.setOrderStatus(order.getOrderStatus())
                            .setResponseMessage(order.getResponseMessage());
                })
                .flatMap(orderRepository::save)
                .subscribe();
    } else if ("INVENTORY-SUCCESS".equals(order.getOrderStatus())) {
        orderRepository.findById(order.getId())
                .map(o -> {
                    orderProducer.sendMessage(o.setOrderStatus(OrderStatus.PREPARE_SHIPPING));
                    return o.setOrderStatus(order.getOrderStatus())
                            .setResponseMessage(order.getResponseMessage());
                })
                .flatMap(orderRepository::save)
                .subscribe();
    } else if ("SHIPPING-FAILURE".equals(order.getOrderStatus())) {
        orderRepository.findById(order.getId())
                .map(o -> {
                    orderProducer.sendMessage(o.setOrderStatus(OrderStatus.REVERT_INVENTORY));
                    return o.setOrderStatus(order.getOrderStatus())
                            .setResponseMessage(order.getResponseMessage());
                })
                .flatMap(orderRepository::save)
                .subscribe();
    } else {
        orderRepository.findById(order.getId())
                .map(o -> {
                    return o.setOrderStatus(order.getOrderStatus())
                            .setResponseMessage(order.getResponseMessage());
                })
                .flatMap(orderRepository::save)
                .subscribe();
    }
}
```

**这里的消费者只是对具有不同订单状态的订单消息做出反应**，这为我们提供了不同服务之间的编排。

最后，我们的订单服务也必须改变以支持这种编排：

```java
public Mono<Order> createOrder(Order order) {
    return Mono.just(order)
            .flatMap(orderRepository::save)
            .map(o -> {
                orderProducer.sendMessage(o.setOrderStatus(OrderStatus.INITIATION_SUCCESS));
                return o;
            })
            .onErrorResume(err -> {
                return Mono.just(order.setOrderStatus(OrderStatus.FAILURE)
                        .setResponseMessage(err.getMessage()));
            })
            .flatMap(orderRepository::save);
}
```

请注意，这比上一节中我们必须使用响应式端点编写的服务要简单得多，**异步编排通常会使代码更加简洁，尽管它牺牲了最终一致性以及复杂的调试和监控**。正如我们可能猜到的那样，我们的前端将不再立即获得订单的最终状态。

## 7. 容器编排服务

我们要解决的最后一个难题与部署有关。

我们希望应用程序具有充足的冗余度，并且能够根据需要自动扩大或缩小规模。

我们已经通过Docker实现了服务的容器化，并通过Docker Compose管理它们之间的依赖关系，虽然这些工具本身就很棒，但它们并不能帮助我们实现目标。

因此，**我们需要一个容器编排服务来处理应用程序中的冗余和可扩展性**。虽然有很多选择，但Kubernetes是其中一种流行的选择。Kubernetes为我们提供了一种与云供应商无关的方法，以实现容器化工作负载的高度可扩展部署。

[Kubernetes](https://www.baeldung.com/kubernetes)将Docker等容器包装到Pod中，Pod是部署的最小单位。此外，我们可以使用[Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)以声明式的方式描述所需状态。

Deployment会创建ReplicaSet，它在内部负责启动Pod，我们可以定义在任何时间点应该运行的相同Pod的最小数量，这提供了冗余，从而实现了高可用性。

让我们看看如何为我们的应用程序定义Kubernetes部署：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: inventory-deployment
spec:
    replicas: 3
    selector:
        matchLabels:
            name: inventory-deployment
    template:
        metadata:
            labels:
                name: inventory-deployment
        spec:
            containers:
                - name: inventory
                  image: inventory-service-async:latest
                  ports:
                      - containerPort: 8081
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: shipping-deployment
spec:
    replicas: 3
    selector:
        matchLabels:
            name: shipping-deployment
    template:
        metadata:
            labels:
                name: shipping-deployment
        spec:
            containers:
                - name: shipping
                  image: shipping-service-async:latest
                  ports:
                      - containerPort: 8082
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: order-deployment
spec:
    replicas: 3
    selector:
        matchLabels:
            name: order-deployment
    template:
        metadata:
            labels:
                name: order-deployment
        spec:
            containers:
                - name: order
                  image: order-service-async:latest
                  ports:
                      - containerPort: 8080
```

这里我们声明我们的部署在任何时候都维护3个相同的Pod副本，虽然这是增加冗余的好方法，但对于变化的负载来说可能还不够。Kubernetes提供了另一种称为[Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)的资源，**它可以根据观察到的指标(如CPU利用率)来扩展部署中的Pod数量**。

请注意，我们刚刚介绍了托管在Kubernetes集群上的应用程序的可扩展性，这并不一定意味着底层集群本身也具备可扩展性。创建高可用性Kubernetes集群并非易事，超出了本文的讨论范围。

## 8. 最终的响应式系统

现在我们已经对架构进行了一些改进，也许是时候根据响应式系统的定义来评估它了，我们将根据之前讨论过的响应式系统的4个特征进行评估：

-   响应式：采用响应式编程范式应该可以帮助我们实现端到端的非阻塞，从而实现响应式应用程序。
-   弹性：具有所需数量Pod的ReplicaSet的Kubernetes部署应该能够抵御随机故障。
-   可伸缩性：Kubernetes集群和资源应该为我们提供必要的支持，以便在面对不可预测的负载时具有弹性。
-   消息驱动：通过Kafka代理异步处理所有服务到服务的通信应该对我们有所帮助。

虽然这看起来很有希望，但远未结束。说实话，**对真正响应式系统的追求应该是一个持续改进的过程**，我们永远无法预知所有可能在高度复杂的基础设施中发生故障的情况，而我们的应用程序只是其中的一小部分。

一个响应式系统需要**构成整体的每个部分都具备可靠性**，从物理网络到DNS等基础设施服务，它们都应该协调一致，以帮助我们实现最终目标。

通常，我们可能无法管理所有这些部分并提供必要的保障。这时，托管云基础设施就能帮我们减轻负担，我们可以选择一系列服务，例如IaaS(基础设施即服务)、BaaS(后端即服务)和PaaS(平台即服务)，将责任委托给外部各方。这样，我们就能尽可能地承担应用程序的责任。

## 9. 总结

在本文中，我们介绍了响应式系统的基础知识，并比较了它与响应式编程的区别。我们创建了一个包含多个微服务的简单应用程序，并重点介绍了我们打算用响应式系统解决的问题。

然后我们在架构中引入了响应式编程、基于消息的架构、以及容器编排服务来实现响应式系统。

最后，我们讨论了最终的架构，以及如何继续构建响应式系统。