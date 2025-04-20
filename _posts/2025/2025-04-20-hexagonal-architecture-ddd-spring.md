---
layout: post
title:  使用六边形架构、DDD和Spring组织层
category: ddd
copyright: ddd
excerpt: DDD
---

## 1. 概述

在本教程中，我们将使用DDD实现一个Spring应用程序。此外，我们将借助六边形架构来组织层。

通过这种方法，我们可以轻松地交换应用程序的不同层。

## 2. 六边形架构

**六边形架构是一种围绕领域逻辑设计软件应用程序以将其与外部因素隔离的模型**。

领域逻辑在业务核心中指定，我们将其称为内部部分，其余部分称为外部部分；外部可以通过端口和适配器访问领域逻辑。 

## 3. 原则

首先，我们应该定义代码划分的原则。正如之前简要解释的那样，**六边形架构定义了内部和外部部分**。

我们在这里要做的是将应用程序分为三层：**应用程序(外部)、域(内部)和基础设施(外部)**：

![](/assets/images/2025/ddd/hexagonalarchitecturedddspring01.png)

通过应用层，**用户或任何其他程序与应用程序交互**。此区域应包含用户接口、RESTful控制器和JSON序列化库等内容。**它包含向应用程序公开入口的所有内容，并协调域逻辑的执行**。

**在领域层，我们保留涉及并实现业务逻辑的代码**，这是我们应用程序的核心，该层应该与应用程序部分和基础架构部分隔离。此外，它还应该包含定义API的接口，以便与领域层需要交互的外部部分(例如数据库)进行通信。

最后，**基础架构层包含应用程序运行所需的所有内容**，例如数据库配置或Spring配置。它还实现了来自领域层的、与基础架构相关的接口。

## 4. 领域层

让我们从实现核心层(即领域层)开始。

首先，我们应该创建Order类：

```java
public class Order {
    private UUID id;
    private OrderStatus status;
    private List<OrderItem> orderItems;
    private BigDecimal price;

    public Order(UUID id, Product product) {
        this.id = id;
        this.orderItems = new ArrayList<>(Arrays.astList(new OrderItem(product)));
        this.status = OrderStatus.CREATED;
        this.price = product.getPrice();
    }

    public void complete() {
        validateState();
        this.status = OrderStatus.COMPLETED;
    }

    public void addOrder(Product product) {
        validateState();
        validateProduct(product);
        orderItems.add(new OrderItem(product));
        price = price.add(product.getPrice());
    }

    public void removeOrder(UUID id) {
        validateState();
        final OrderItem orderItem = getOrderItem(id);
        orderItems.remove(orderItem);

        price = price.subtract(orderItem.getPrice());
    }

    // getters
}
```

这是我们的[聚合根](https://www.baeldung.com/spring-persisting-ddd-aggregates)，任何与业务逻辑相关的内容都会经过这个类。此外，Order还负责保持自身处于正确的状态：

- 订单只能使用给定的ID并基于一个Product来创建；构造函数本身也会以CREATED状态初始化订单。
- 一旦订单完成，就无法更改OrderItem。
- 不可能像使用Setter那样从域对象外部更改Order。

此外，Order类还负责创建其OrderItem。

因此让我们创建OrderItem类：

```java
public class OrderItem {
    private UUID productId;
    private BigDecimal price;

    public OrderItem(Product product) {
        this.productId = product.getId();
        this.price = product.getPrice();
    }

    // getters
}
```

我们可以看到，OrderItem是基于Product创建的。它保存了对Product的引用，并存储了Product的当前价格。

接下来，我们将创建一个Repository接口(六边形架构中的一个端口)，该接口的实现将位于基础架构层：

```java
public interface OrderRepository {
    Optional<Order> findById(UUID id);

    void save(Order order);
}
```

最后，我们应该确保每次操作后订单都会被保存。**为此，我们将定义一个领域服务，它通常包含不能作为根服务一部分的逻辑**：

```java
public class DomainOrderService implements OrderService {

    private final OrderRepository orderRepository;

    public DomainOrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Override
    public UUID createOrder(Product product) {
        Order order = new Order(UUID.randomUUID(), product);
        orderRepository.save(order);

        return order.getId();
    }

    @Override
    public void addProduct(UUID id, Product product) {
        Order order = getOrder(id);
        order.addOrder(product);

        orderRepository.save(order);
    }

    @Override
    public void completeOrder(UUID id) {
        Order order = getOrder(id);
        order.complete();

        orderRepository.save(order);
    }

    @Override
    public void deleteProduct(UUID id, UUID productId) {
        Order order = getOrder(id);
        order.removeOrder(productId);

        orderRepository.save(order);
    }

    private Order getOrder(UUID id) {
        return orderRepository
                .findById(id)
                .orElseThrow(RuntimeException::new);
    }
}
```

在六边形架构中，此服务是实现端口的适配器。**我们不会将其注册为Spring Bean，因为从领域角度来看，它位于内部，而Spring配置位于外部**。稍后我们将在基础架构层手动将其与Spring连接。

由于领域层与应用层和基础设施层完全解耦，我们也可以独立地对其进行测试：

```java
class DomainOrderServiceUnitTest {

    private OrderRepository orderRepository;
    private DomainOrderService tested;
    @BeforeEach
    void setUp() {
        orderRepository = mock(OrderRepository.class);
        tested = new DomainOrderService(orderRepository);
    }

    @Test
    void shouldCreateOrder_thenSaveIt() {
        final Product product = new Product(UUID.randomUUID(), BigDecimal.TEN, "productName");

        final UUID id = tested.createOrder(product);

        verify(orderRepository).save(any(Order.class));
        assertNotNull(id);
    }
}
```

## 5. 应用层

在本节中，我们将实现应用层，我们将允许用户通过RESTful API与我们的应用程序进行通信。

因此让我们创建OrderController：

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private OrderService orderService;

    @Autowired
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    CreateOrderResponse createOrder(@RequestBody CreateOrderRequest request) {
        UUID id = orderService.createOrder(request.getProduct());

        return new CreateOrderResponse(id);
    }

    @PostMapping(value = "/{id}/products")
    void addProduct(@PathVariable UUID id, @RequestBody AddProductRequest request) {
        orderService.addProduct(id, request.getProduct());
    }

    @DeleteMapping(value = "/{id}/products")
    void deleteProduct(@PathVariable UUID id, @RequestParam UUID productId) {
        orderService.deleteProduct(id, productId);
    }

    @PostMapping("/{id}/complete")
    void completeOrder(@PathVariable UUID id) {
        orderService.completeOrder(id);
    }
}
```

**这个简单的[Spring REST控制器](https://www.baeldung.com/building-a-restful-web-service-with-spring-and-java-based-configuration)负责协调域逻辑的执行**。

该控制器将外部RESTful接口适配到我们的领域，它通过调用OrderService(端口)中的相应方法来实现。

## 6. 基础设施层

基础设施层包含运行应用程序所需的逻辑。

我们将从创建配置类开始；首先，我们将实现一个类，将OrderService注册为Spring Bean：

```java
@Configuration
public class BeanConfiguration {

    @Bean
    OrderService orderService(OrderRepository orderRepository) {
        return new DomainOrderService(orderRepository);
    }
}
```

接下来，我们将创建负责启用我们将使用的[Spring Data](https://www.baeldung.com/spring-data-mongodb-tutorial) Repository的配置：

```java
@EnableMongoRepositories(basePackageClasses = SpringDataMongoOrderRepository.class)
public class MongoDBConfiguration {
}
```

我们使用了basePackageClasses属性，因为这些Repository只能位于基础架构层。因此，Spring无需扫描整个应用程序。此外，此类可以包含与在MongoDB和我们的应用程序之间建立连接相关的所有内容。

最后，我们将从领域层实现OrderRepository，我们将在实现中使用SpringDataMongoOrderRepository：

```java
@Component
public class MongoDbOrderRepository implements OrderRepository {

    private SpringDataMongoOrderRepository orderRepository;

    @Autowired
    public MongoDbOrderRepository(SpringDataMongoOrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Override
    public Optional<Order> findById(UUID id) {
        return orderRepository.findById(id);
    }

    @Override
    public void save(Order order) {
        orderRepository.save(order);
    }
}
```

此实现将订单存储在MongoDB中，在六边形架构中，此实现也是一个适配器。

## 7. 优点

这种方法的第一个优点是我们将工作分为各个层，我们可以专注于某一层而不影响其他层。

此外，由于它们各自注重其逻辑，因此自然更容易理解。

另一个巨大的优势是，我们将领域逻辑与其他部分隔离开来。**领域部分只包含业务逻辑，可以轻松迁移到其他环境**。

事实上，让我们改变基础设施层以使用[Cassandra](https://www.baeldung.com/spring-data-cassandra-tutorial)作为数据库：

```java
@Component
public class CassandraDbOrderRepository implements OrderRepository {

    private final SpringDataCassandraOrderRepository orderRepository;

    @Autowired
    public CassandraDbOrderRepository(SpringDataCassandraOrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    @Override
    public Optional<Order> findById(UUID id) {
        Optional<OrderEntity> orderEntity = orderRepository.findById(id);
        if (orderEntity.isPresent()) {
            return Optional.of(orderEntity.get()
                    .toOrder());
        } else {
            return Optional.empty();
        }
    }

    @Override
    public void save(Order order) {
        orderRepository.save(new OrderEntity(order));
    }
}
```

与MongoDB不同，我们现在使用OrderEntity将域保存在数据库中。

**如果我们向Order域对象添加特定于技术的注解，那么我们就违反了基础设施层和域层之间的解耦**。

Repository使域适应我们的持久化需求。

让我们更进一步，将RESTful应用程序转换为命令行应用程序：

```java
@Component
public class CliOrderController {

    private static final Logger LOG = LoggerFactory.getLogger(CliOrderController.class);

    private final OrderService orderService;

    @Autowired
    public CliOrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    public void createCompleteOrder() {
        LOG.info("<<Create complete order>>");
        UUID orderId = createOrder();
        orderService.completeOrder(orderId);
    }

    public void createIncompleteOrder() {
        LOG.info("<<Create incomplete order>>");
        UUID orderId = createOrder();
    }

    private UUID createOrder() {
        LOG.info("Placing a new order with two products");
        Product mobilePhone = new Product(UUID.randomUUID(), BigDecimal.valueOf(200), "mobile");
        Product razor = new Product(UUID.randomUUID(), BigDecimal.valueOf(50), "razor");
        LOG.info("Creating order with mobile phone");
        UUID orderId = orderService.createOrder(mobilePhone);
        LOG.info("Adding a razor to the order");
        orderService.addProduct(orderId, razor);
        return orderId;
    }
}
```

与之前不同，我们现在硬编码了一组与领域交互的预定义操作。例如，我们可以用它来填充模拟数据。

尽管我们完全改变了应用程序的目的，但我们还没有触及领域层。

## 8. 总结

在本文中，我们学习了如何将与我们的应用程序相关的逻辑分离到特定的层中。

首先，我们定义了三个主要层：应用程序层、领域层和基础设施层。然后，我们描述了如何填充这三个层，并解释了其优势。

接下来，我们提出了每一层的实现：

![](/assets/images/2025/ddd/hexagonalarchitecturedddspring02.png)

最后，我们在不影响域的情况下交换了应用程序层和基础设施层。