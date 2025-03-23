---
layout: post
title:  在JPA中保存后刷新并获取实体
category: spring-data
copyright: spring-data
excerpt: Spring Data JPA
---

## 1. 简介

[Java Persistence API(JPA)](https://www.baeldung.com/learn-jpa-hibernate)充当Java对象和关系型数据库之间的桥梁，使我们能够无缝地持久化和检索数据。在本教程中，我们将探索各种策略和技术，以便在JPA中保存操作后有效地刷新和获取实体。

## 2. 了解Spring Data JPA中的实体管理

在[Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)中，实体管理围绕[JpaRepository](https://docs.spring.io/spring-data/jpa/docs/2.7.9/api/org/springframework/data/jpa/repository/JpaRepository.html)接口进行，该接口是与数据库交互的主要机制。通过扩展[CrudRepository](https://docs.spring.io/spring-data/data-commons/docs/current/api/org/springframework/data/repository/CrudRepository.html)的JpaRepository接口，Spring Data JPA提供了一组强大的方法用于实体持久化、检索、更新和删除。

此外，**EntityManager会被Spring容器自动注入到这些Repository接口中**。该组件是Spring Data JPA中嵌入的JPA基础架构的组成部分，有助于与底层持久化上下文进行交互以及执行JPA查询。

### 2.1 持久化上下文

JPA中的一个关键组件是持久化上下文，**可以将此上下文想象为一个临时保存区域，JPA在其中管理已检索或已创建的实体的状态**。

它确保：

- 实体是唯一的：在任何给定时间，上下文中只存在一个具有特定主键的实体实例。
- 跟踪更改：EntityManager跟踪上下文中对实体属性所做的任何修改。
- 保持数据一致性：EntityManager在事务期间将上下文中所做的更改与底层数据库同步。

### 2.2 JPA实体的生命周期

JPA实体有四个不同的生命周期阶段：新建、托管、删除和分离。

当我们使用实体的构造函数创建一个新的实体实例时，它处于“新建”状态。我们可以通过检查实体的ID(主键)是否为空来验证这一点：

```java
Order order = new Order();
if (order.getId() == null) {
    // Entity is in the "New" state
}
```

使用Repository的save()方法持久化实体后，它将转换为“托管”状态。我们可以通过检查Repository中是否存在已保存的实体来验证这一点：

```java
Order savedOrder = repository.save(order);
if (repository.findById(savedOrder.getId()).isPresent()) {
    // Entity is in the "Managed" state
}
```

当我们在托管实体上调用Repository的delete()方法时，它会转换为“删除”状态。我们可以通过检查删除后实体是否不再存在于数据库中来验证这一点：

```java
repository.delete(savedOrder);
if (!repository.findById(savedOrder.getId()).isPresent()) {
    // Entity is in the "Removed" state
}
```

最后，一旦使用Repository的detach()方法分离实体，该实体就不再与持久化上下文相关联。**对分离实体所做的更改不会反映在数据库中，除非明确合并回托管状态**。我们可以通过在分离实体后尝试修改它来验证这一点：

```java
repository.detach(savedOrder);
// Modify the entity
savedOrder.setName("New Order Name");
```

如果我们在一个分离的实体上调用save()，它会将该实体重新附加到持久化上下文，并在刷新持久化上下文时将更改持久保存到数据库中。

## 3. 使用Spring Data JPA保存实体

当我们调用save()时，Spring Data JPA安排在事务提交时将实体插入数据库。它将实体添加到持久化上下文，并将其标记为托管。

下面是一个简单的代码片段，演示如何使用Spring Data JPA中的save()方法来持久化一个实体：

```java
Order order = new Order();
order.setName("New Order Name");

repository.save(order);
```

但是，**需要注意的是，调用save()不会立即触发数据库插入操作**。相反，它只是将实体转换为持久化上下文中的托管状态。因此，如果其他事务在我们的事务提交之前从数据库读取数据，它们可能会检索到过时的数据，这些数据不包括我们已做但尚未提交的更改。

为了确保数据保持最新，我们可以采用两种方法：获取和刷新。

## 4. 在Spring Data JPA中获取实体

当我们获取一个实体时，我们不会丢弃在持久化上下文中对它所做的任何修改。**相反，我们只是从数据库中检索实体的数据并将其添加到持久化上下文中以供进一步处理**。

### 4.1 使用findById()

Spring Data JPA Repository提供了诸如findById()之类的便捷方法来检索实体，**这些方法始终从数据库获取最新数据，而不管持久化上下文中的实体状态如何**。这种方法简化了实体检索，并且无需直接管理持久化上下文。

```java
Order order = repository.findById(1L).get();
```

### 4.2 急切获取与惰性获取

**在[急切获取](https://www.baeldung.com/hibernate-lazy-eager-loading#Eager)中，与主实体相关联的所有相关实体都与主实体同时从数据库检索**。通过在orderItems集合上设置fetch = FetchType.EAGER，我们指示JPA在检索Order时急切获取所有关联的OrderItem实体：

```java
@Entity
public class Order {
    @Id
    private Long id;

    @OneToMany(mappedBy = "order", fetch = FetchType.EAGER)
    private List<OrderItem> orderItems;
}
```

这意味着在调用findById()之后，我们可以直接访问order对象中的orderItems列表并遍历关联的OrderItem实体，而无需任何额外的数据库查询：

```java
Order order = repository.findById(1L).get();

// Accessing OrderItems directly after fetching the Order
if (order != null) {
    for (OrderItem item : order.getOrderItems()) {
        System.out.println("Order Item: " + item.getName() + ", Quantity: " + item.getQuantity());
    }
}
```

另一方面，通过设置fetch = FetchType.LAZY，相关实体将不会从数据库中检索，直到在代码中明确访问它们：

```java
@Entity
public class Order {
    @Id
    private Long id;

    @OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
    private List<OrderItem> orderItems;
}
```

**当我们调用order.getOrderItems()时，将执行单独的数据库查询来获取与该order相关的OrderItem实体**。仅因为我们明确访问了orderItems列表，才会触发此额外查询：

```java
Order order = repository.findById(1L).get();

if (order != null) {
    List<OrderItem> items = order.getOrderItems(); // This triggers a separate query to fetch OrderItems
    for (OrderItem item : items) {
        System.out.println("Order Item: " + item.getName() + ", Quantity: " + item.getQuantity());
    }
}
```

### 4.3 使用JPQL获取

**[Java持久化查询语言(JPQL)](https://www.baeldung.com/spring-data-jpa-query)允许我们编写针对实体而非表的类似SQL的查询**，它可以根据各种条件灵活地检索特定数据或实体。

让我们看一个按客户名称获取订单且订单日期在指定范围内的示例：

```java
@Query("SELECT o FROM Order o WHERE o.customerName = :customerName AND o.orderDate BETWEEN :startDate AND :endDate")
List<Order> findOrdersByCustomerAndDateRange(@Param("customerName") String customerName, @Param("startDate") LocalDate startDate, @Param("endDate") LocalDate endDate);
```

### 4.4 使用Criteria API获取

Spring Data JPA中的[Criteria API](https://www.baeldung.com/hibernate-criteria-queries)提供了一种可靠且灵活的方法来动态创建查询，**它允许我们使用方法链和条件表达式安全地构建复杂查询，确保我们的查询在编译时没有错误**。

让我们考虑一个示例，其中我们使用Criteria API根据多种条件(例如客户姓名和订单日期范围)组合来获取订单：

```java
CriteriaBuilder criteriaBuilder = entityManager.getCriteriaBuilder();
CriteriaQuery<Order> criteriaQuery = criteriaBuilder.createQuery(Order.class);
Root<Order> root = criteriaQuery.from(Order.class);

Predicate customerPredicate = criteriaBuilder.equal(root.get("customerName"), customerName);
Predicate dateRangePredicate = criteriaBuilder.between(root.get("orderDate"), startDate, endDate);

criteriaQuery.where(customerPredicate, dateRangePredicate);

return entityManager.createQuery(criteriaQuery).getResultList();
```

## 5. 使用Spring Data JPA刷新实体

JPA中的刷新实体可确保应用程序中实体的内存表示与数据库中存储的最新数据保持同步，**当其他事务修改或更新实体时，持久化上下文中的数据可能会过时**。刷新实体使我们能够从数据库中检索最新数据，从而防止出现不一致并保持数据准确性。

### 5.1 使用refresh()

在JPA中，我们使用EntityManager提供的refresh()方法实现实体刷新。**对托管实体调用refresh()会丢弃持久化上下文中对该实体所做的任何修改**，它会从数据库重新加载实体的状态，从而有效地替换自实体上次与数据库同步以来所做的任何修改。

但需要注意的是，Spring Data JPA Repository不提供内置的refresh()方法。

以下是使用EntityManager刷新实体的方法：

```java
@Autowired
private EntityManager entityManager;

entityManager.refresh(order);
```

### 5.2 处理OptimisticLockException

**Spring Data JPA中的@Version注解用于实现乐观锁，当多个事务可能尝试同时更新同一实体时，它有助于确保数据一致性**。当我们使用@Version时，JPA会自动在我们的实体类上创建一个特殊字段(通常名为version)。

该字段存储一个整数值，表示数据库中实体的版本：

```java
@Entity
public class Order {
    @Id
    @GeneratedValue
    private Long id;
    
    @Version
    private Long version;
}
```

从数据库检索实体时，JPA会主动获取其版本。**更新实体时，JPA会将持久化上下文中的实体版本与数据库中存储的版本进行比较**。如果实体的版本不同，则表明另一个事务已修改该实体，这可能会导致数据不一致。

**在这种情况下，JPA会抛出异常(通常是OptimisticLockException)，以指示潜在的冲突**。因此，我们可以在catch块中调用refresh()方法从数据库重新加载实体的状态。

让我们简单演示一下这种方法的工作原理：

```java
Order order = orderRepository.findById(orderId)
    .map(existingOrder -> {
        existingOrder.setName(newName);
        return existingOrder;
    })
    .orElseGet(() -> {
        return null;
    });

if (order != null) {
    try {
        orderRepository.save(order);
    } catch (OptimisticLockException e) {
        // Refresh the entity and potentially retry the update
        entityManager.refresh(order);
        // Consider adding logic to handle retries or notify the user about the conflict
    }
}
```

此外，值得注意的是，如果自上次检索后刷新的实体已被另一个事务从数据库中删除，则refresh()可能抛出javax.persistence.EntityNotFoundException。

## 6. 总结

在本文中，我们了解了Spring Data JPA中刷新和获取实体之间的区别。获取涉及在需要时从数据库检索最新数据，刷新涉及使用数据库中的最新数据更新持久化上下文中的实体状态。

通过策略性地利用这些方法，我们可以保持数据一致性并确保所有事务都基于最新数据进行操作。