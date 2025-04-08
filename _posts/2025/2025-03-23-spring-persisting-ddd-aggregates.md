---
layout: post
title:  持久化DDD聚合
category: springdata
copyright: springdata
excerpt: DDD
---

## 1. 概述

在本教程中，我们将探讨使用不同技术持久化[DDD聚合](https://martinfowler.com/bliki/DDD_Aggregate.html)的可能性。

## 2. 聚合介绍

**[聚合](https://www.baeldung.com/cs/aggregate-root-ddd)是一组始终需要保持一致的业务对象**。因此，我们在事务中将聚合作为一个整体进行保存和更新。

聚合是DDD中的一个重要战术模式，它有助于维护业务对象的一致性。然而，聚合的概念在DDD上下文之外也很有用。

在许多业务案例中，此模式都很有用。**根据经验，当同一事务中有多个对象发生更改时，我们应该考虑使用聚合**。

让我们看看在订单建模时如何应用这一点。

### 2.1 采购订单示例

因此，假设我们想要建立采购订单模型：

```java
class Order {
    private Collection<OrderLine> orderLines;
    private Money totalCost;
    // ...
}
```

```java
class OrderLine {
    private Product product;
    private int quantity;
    // ...
}
```

```java
class Product {
    private Money price;
    // ...
}
```

**这些类构成了一个简单的聚合**，Order的orderLines和totalCost字段必须始终保持一致，也就是说totalCost的值应始终等于所有orderLines的值之和。

**现在，我们可能都想将所有这些转变为成熟的Java Bean**。但是，请注意，在Order中引入简单的Getter和Setter很容易破坏我们模型的封装并违反业务约束。

让我们看看可能出现什么问题。

### 2.2 简单的聚合设计

让我们想象一下，如果我们决定天真地向Order类的所有属性添加Getter和Setter，包括setOrderTotal，会发生什么。

没有什么可以阻止我们执行以下代码：

```java
Order order = new Order();
order.setOrderLines(Arrays.asList(orderLine0, orderLine1));
order.setTotalCost(Money.zero(CurrencyUnit.USD)); // this doesn't look good...
```

在这段代码中，我们手动将totalCost属性设置为0，违反了一条重要的业务规则：总成本绝对不应该是0美元。

**我们需要一种方法来保护我们的业务规则，让我们看看聚合根如何提供帮助**。

### 2.3 聚合根

聚合根是一个作为聚合入口点的类，**所有业务操作都应通过该根**。这样，聚合根就可以负责保持聚合处于一致状态。

**根在于处理我们所有业务的不变量**。

在我们的示例中，Order类是聚合根的合适候选者，我们只需要进行一些修改即可确保聚合始终一致：

```java
class Order {
    private final List<OrderLine> orderLines;
    private Money totalCost;

    Order(List<OrderLine> orderLines) {
        checkNotNull(orderLines);
        if (orderLines.isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one order line item");
        }
        this.orderLines = new ArrayList<>(orderLines);
        totalCost = calculateTotalCost();
    }

    void addLineItem(OrderLine orderLine) {
        checkNotNull(orderLine);
        orderLines.add(orderLine);
        totalCost = totalCost.plus(orderLine.cost());
    }

    void removeLineItem(int line) {
        OrderLine removedLine = orderLines.remove(line);
        totalCost = totalCost.minus(removedLine.cost());
    }

    Money totalCost() {
        return totalCost;
    }

    // ...
}
```

使用聚合根现在可以让我们更轻松地将Product和OrderLine转换为不可变对象，其中所有属性都是final的。

我们可以看到，这是一个非常简单的聚合。

而且，我们可以简单地计算每次的总成本而不使用字段。

但是，目前我们只讨论聚合持久化，而不是聚合设计。请继续关注，因为这个特定领域稍后会派上用场。

这与持久化技术配合得如何？让我们来看看。**最终，这将帮助我们为下一个项目选择正确的持久化工具**。

## 3. JPA和Hibernate

在本节中，让我们尝试使用JPA和Hibernate持久化我们的订单聚合，我们将使用Spring Boot和[JPA Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

对于我们大多数人来说，这似乎是最自然的选择。毕竟，我们已经与关系型系统打交道多年，并且我们都知道流行的ORM框架。

**使用ORM框架时最大的问题可能是模型设计的简化**，有时也称为[对象关系阻抗不匹配](https://en.wikipedia.org/wiki/Object-relational_impedance_mismatch)。让我们想想如果我们想要持久化我们的订单聚合会发生什么：

```java
@DisplayName("given order with two line items, when persist, then order is saved")
@Test
public void test() throws Exception {
    // given
    JpaOrder order = prepareTestOrderWithTwoLineItems();

    // when
    JpaOrder savedOrder = repository.save(order);

    // then
    JpaOrder foundOrder = repository.findById(savedOrder.getId())
            .get();
    assertThat(foundOrder.getOrderLines()).hasSize(2);
}
```

此时，此测试将引发异常：

> java.lang.IllegalArgumentException: Unknown entity: cn.tuyucheng.taketoday.ddd.order.Order。

**显然，我们缺少一些JPA要求**：

1. 添加映射注解
2. OrderLine和Product类必须是实体或@Embeddable类，而不是简单的值对象
3. 为每个实体或@Embeddable类添加一个空的构造函数
4. 用简单类型替换Money属性

**嗯，我们需要修改Order聚合的设计才能使用JPA。虽然添加注解不是什么大问题，但其他要求可能会带来很多问题**。

### 3.1 值对象的变更

尝试将聚合放入JPA的第一个难题是我们需要打破值对象的设计：它们的属性不再是final的，我们需要打破封装。

**我们需要为OrderLine和Product添加人工ID，即使这些类从未设计为具有标识符**。我们希望它们是简单的值对象。

可以使用@Embedded和@ElementCollection注解，但是在使用复杂对象图(例如@Embeddable对象具有另一个@Embedded属性等)时，这种方法会使事情变得非常复杂。

使用@Embedded注解只是向父表添加平面属性，除此之外，基本属性(例如String类型)仍然需要Setter方法，这违反了所需的值对象设计。

**空构造函数要求强制值对象属性不再是final，从而破坏了我们原始设计的一个重要方面**。说实话，Hibernate可以使用私有无参数构造函数，这可以稍微缓解这个问题，但它还远远不够完美。

即使使用私有默认构造函数，我们也不能将我们的属性标记为final，或者我们需要在默认构造函数中使用默认值(通常为null)初始化它们。

但是，如果我们想要完全符合JPA标准，我们必须至少对默认构造函数使用protected可见性，这意味着同一个包中的其他类可以创建值对象而无需指定其属性的值。

### 3.2 复杂类型

**不幸的是，我们不能指望JPA自动将第三方复杂类型映射到表中，看看我们在上一节中必须引入多少更改**！

例如，当处理我们的订单聚合时，我们会遇到持久化Joda Money字段的困难。

在这种情况下，我们最终可能会编写JPA 2.1中提供的自定义类型@Converter。不过，这可能需要一些额外的工作。

或者，我们也可以将Money属性拆分为两个基本属性。例如，String表示货币单位，BigDecimal表示实际值。

虽然我们可以隐藏实现细节并仍然通过公共方法API使用Money类，但实践表明，大多数开发人员无法证明额外的工作是合理的，而只会简单地使模型退化以符合JPA规范。

### 3.3 总结

虽然JPA是世界上采用最广泛的规范之一，但它可能不是持久化我们的订单聚合的最佳选择。

**如果我们希望我们的模型反映真正的业务规则，我们就不应该将其设计为底层表的简单1:1表示**。

基本上，我们这里有三个选择：

1. 创建一组简单的数据类，并使用它们来持久化和重新创建丰富的业务模型。不幸的是，这可能需要大量额外的工作
2. 接受JPA的限制并选择正确的折衷方案
3. 考虑另一种技术

第一种方案的潜力最大，实际开发中大部分项目都是采用第二种方案。

现在，让我们考虑另一种持久聚合的技术。

## 4. 文档存储

文档存储是存储数据的另一种方式，我们保存的是整个对象，而不是关系和表，**这使得文档存储成为持久化聚合的潜在完美候选者**。

为了满足本教程的需要，我们将重点关注类似JSON的文档。

让我们仔细看看我们的订单持久化问题在像MongoDB这样的文档存储中是怎样的。

### 4.1 使用MongoDB持久化聚合

**现在有不少数据库可以存储JSON数据，其中比较流行的是MongoDB**，MongoDB实际上以二进制形式存储BSON或JSON。

**感谢MongoDB，我们可以按原样存储订单示例聚合**。

在继续之前，让我们添加[Spring Boot MongoDB Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-mongodb)：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

现在我们可以运行与JPA示例类似的测试用例，但这次使用MongoDB：

```java
@DisplayName("given order with two line items, when persist using mongo repository, then order is saved")
@Test
void test() throws Exception {
    // given
    Order order = prepareTestOrderWithTwoLineItems();

    // when
    repo.save(order);

    // then
    List<Order> foundOrders = repo.findAll();
    assertThat(foundOrders).hasSize(1);
    List<OrderLine> foundOrderLines = foundOrders.iterator()
            .next()
            .getOrderLines();
    assertThat(foundOrderLines).hasSize(2);
    assertThat(foundOrderLines).containsOnlyElementsOf(order.getOrderLines());
}
```

**重要的是-我们根本没有改变原始的Order聚合类**；不需要为Money类创建默认构造函数、Setter或自定义转换器。

以下是我们的订单聚合在存储中显示的内容：

```json
{
    "_id": ObjectId("5bd8535c81c04529f54acd14"),
    "orderLines": [
        {
            "product": {
                "price": {
                    "money": {
                        "currency": {
                            "code": "USD",
                            "numericCode": 840,
                            "decimalPlaces": 2
                        },
                        "amount": "10.00"
                    }
                }
            },
            "quantity": 2
        },
        {
            "product": {
                "price": {
                    "money": {
                        "currency": {
                            "code": "USD",
                            "numericCode": 840,
                            "decimalPlaces": 2
                        },
                        "amount": "5.00"
                    }
                }
            },
            "quantity": 10
        }
    ],
    "totalCost": {
        "money": {
            "currency": {
                "code": "USD",
                "numericCode": 840,
                "decimalPlaces": 2
            },
            "amount": "70.00"
        }
    },
    "_class": "cn.tuyucheng.taketoday.ddd.order.mongo.Order"
}
```

这个简单的BSON文档包含整个订单聚合，与我们最初的想法(所有这些都应该是联合一致的)很好地匹配。

请注意，BSON文档中的复杂对象仅被序列化为一组常规JSON属性。得益于此，即使是第三方类(如Joda Money)也可以轻松序列化，而无需简化模型。

### 4.2 总结

使用MongoDB持久化聚合比使用JPA更简单。

**这绝对不意味着MongoDB优于传统数据库**，在很多合理的情况下，我们甚至不应该尝试将我们的类建模为聚合，而应该使用SQL数据库。

**不过，当我们确定了一组应该根据复杂要求始终保持一致的对象时，使用文档存储可能是一个非常有吸引力的选择**。

## 5. 总结

在DDD中，聚合通常包含系统中最复杂的对象，使用它们需要与大多数CRUD应用程序截然不同的方法。

使用流行的ORM解决方案可能会导致域模型过于简单或过度暴露，通常无法表达或执行复杂的业务规则。

**文档存储可以更容易地持久化聚合，而不会牺牲模型的复杂性**。