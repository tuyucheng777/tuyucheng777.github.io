---
layout: post
title:  在Spring Data中按嵌套对象的属性查找
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

在Spring Data中，通常使用基于方法名称的派生查询来查询实体。在处理实体之间的关系(例如嵌套对象)时，Spring Data提供了各种机制来从这些嵌套对象中检索数据。

在本教程中，我们将探讨如何使用查询派生和JPQL(Java持久化查询语言)通过嵌套对象的属性进行查询。

## 2. 场景概述

让我们考虑一个有两个实体的简单场景：Customer和Order每个Order通过ManyToOne关系链接到Customer。

我们希望找到属于具有特定email的客户的所有订单。在这种情况下，email是Customer实体的属性，而我们的主要查询将在Order实体上执行。

以下是我们的示例实体：

```java
@Entity
@Table(name = "customers")
public class Customer {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    private String email;

    // getters and setters
}

@Entity
@Table(name = "orders")
public class Order {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private Date orderDate;

    @ManyToOne
    @JoinColumn(name = "customer_id")
    private Customer customer;

    // getters and setters
}
```

## 3. 使用查询派生

**Spring Data JPA通过允许开发人员从Repository接口中的方法签名[派生查询](https://www.baeldung.com/spring-data-derived-queries)来简化查询创建**。

### 3.1 正常用例

在正常情况下，如果我们想通过关联的Customer的email查找Order实体，我们可以简单地这样做：

```java
public interface OrderRepository extends JpaRepository<Order, Long> {
    List<Order> findByCustomerEmail(String email);
}
```

生成的SQL将如下所示：

```sql
select
    o.id,
    o.customer_id,
    o.order_date
from
    orders o
left outer join
    customers c
        on o.customer_id = c.id
where
    c.email = ?
```

### 3.2 关键字大小写冲突

现在，假设除了嵌套的Customer对象之外，Order类本身还有一个名为customerEmail的字段。在这种情况下，Spring Data JPA不会像我们预期的那样仅在customers表上生成查询：

```sql
select
    o.id,
    o.customer_id,
    o.customer_email,
    o.order_date
from
    orders o
where
    o.customer_email = ?
```

在这种情况下，**我们可以使用下划线字符来定义JPA应该尝试拆分关键字的位置**：

```java
List<Order> findByCustomer_Email(String email);
```

这里，下划线帮助Spring Data JPA正确解析查询方法。

## 4. 使用JPQL查询

**如果我们想要更好地控制查询逻辑或执行更复杂的操作，[JPQL](https://www.baeldung.com/jpql-hql-criteria-query)是一个不错的选择**。要使用JPQL查询按Customer的email查询Order，我们可以编写如下代码：

```java
@Query("SELECT o FROM Order o WHERE o.customer.email = ?1")
List<Order> findByCustomerEmailAndJPQL(String email);
```

这使我们可以灵活地编写更具定制性的查询，而不依赖于方法名称约定。

## 5. 总结

在本文中，我们探讨了如何通过Spring Data中嵌套对象的属性查询数据。我们介绍了查询派生和自定义JPQL查询，通过利用这些技术，我们可以轻松处理许多用例。