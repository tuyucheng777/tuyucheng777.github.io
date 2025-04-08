---
layout: post
title:  用于数据库视图的Spring Data JPA Repository
category: springdata
copyright: springdata
excerpt: Spring Data JPA
---

## 1. 概述

数据库视图是关系型数据库系统中的类似表的结构，其中数据源来自一个或多个连接在一起的表。

虽然Spring Data [Repository](https://www.baeldung.com/spring-data-repositories)通常用于数据库表，但它们也可以有效地应用于数据库视图。在本教程中，我们将探讨如何采用Spring Data Repository来创建数据库视图。

## 2. 数据库表设置

在本教程中，我们将采用H2数据库系统进行数据定义，并使用两个示例表SHOP和SHOP_TRANSACTION演示数据库视图概念。

SHOP表存储商店信息：

```sql
CREATE TABLE SHOP
(
    shop_id             int             AUTO_INCREMENT,
    shop_location       varchar(100)    NOT NULL UNIQUE,
    PRIMARY KEY(shop_id)
);
```

SHOP_TRANSACTION表存储与商店相关的交易记录以及通过shop_id对SHOP表的引用：

```sql
CREATE TABLE SHOP_TRANSACTION
(
    transaction_id      bigint          AUTO_INCREMENT,
    transaction_date    date            NOT NULL,
    shop_id             int             NOT NULL,
    amount              decimal(8,2)    NOT NULL,
    PRIMARY KEY(transaction_id),
    FOREIGN KEY(shop_id) REFERENCES SHOP(shop_id)
);
```

在实体关系(ER)模型中，我们可以将其说明为一对多关系，其中一家商店可以有多笔交易。不过，每笔交易仅与一家商店相关联。我们可以使用[ER图](https://www.baeldung.com/cs/erd)直观地表示这一点：

![](/assets/images/2025/springdata/springdatajparepositoryview01.png)

## 3. 数据库视图

**数据库视图提供了一个虚拟表，用于从预定义查询的结果中收集数据**。使用数据库视图而不是使用连接查询有以下优势：

- 简单性：视图封装了复杂的连接，无需反复重写相同的连接查询
- 安全性：视图可能只包含基表中的一部分数据，从而降低了暴露基表中敏感信息的风险
- 可维护性：当基表结构发生变化时更新视图定义可以避免修改应用程序中引用已更改基表的查询

### 3.1 标准视图和物化视图

有两种常见类型的[数据库视图](https://www.baeldung.com/databases-views-simple-complex-materialized)，它们有不同的用途：

- 标准视图：这些视图是在查询时通过执行预定义的SQL查询生成的，它们本身不存储数据，所有数据都存储在底层基表中。
- 物化视图：这些视图类似于标准视图，也是从预定义的SQL查询生成的。不同之处在于，它们将查询结果到数据库中的物理表中，后续查询从此表中检索数据，而不是动态生成数据。

下面的比较表重点介绍了标准视图和物化视图的不同特点，有助于根据特定要求选择合适的视图类型：

|     |      标准视图       |                   物化视图                   |
|:---:|:---------------:| :------------------------------------------: |
| 数据源 | 通过预定义查询从基础表动态生成 |          包含预定义查询数据的物理表          |
| 性能  |  由于动态查询生成而速度较慢  |        由于从物理表检索数据，速度更快        |
| 过时  |    始终返回最新数据     |         可能会变得陈旧，需要定期刷新         |
| 用例  |     适用于实时数据     | 适用于数据新鲜度不是很重要时计算量很大的查询 |

### 3.2 标准视图示例

在我们的示例中，我们想定义一个视图来总结每个日历月的商店总销售额。物化视图被证明是合适的，因为前几个月的销售额保持不变。除非需要当月的数据，否则实时数据对于计算总销售额是不必要的。

但是，H2数据库不支持物化视图，我们将创建一个标准视图：

```sql
CREATE VIEW SHOP_SALE_VIEW AS
SELECT ROW_NUMBER() OVER () AS id, shop_id, shop_location, transaction_year, transaction_month, SUM(amount) AS total_amount
FROM (
    SELECT 
        shop.shop_id, shop.shop_location, trans.amount, 
        YEAR(transaction_date) AS transaction_year, MONTH(transaction_date) AS transaction_month
    FROM SHOP shop, SHOP_TRANSACTION trans
    WHERE shop.shop_id = trans.shop_id
) SHOP_MONTH_TRANSACTION
GROUP BY shop_id, transaction_year, transaction_month;
```

查询视图后，我们应该获得如下数据：

|  id  | shop_id | shop_location | transaction_year | transaction_month | amount |
| :--: | :-----: | :-----------: | :--------------: | :---------------: | :----: |
|  1   |    1    |    Ealing     |       2024       |         1         | 10.78  |
|  2   |    1    |    Ealing     |       2024       |         2         | 13.58  |
|  3   |    1    |    Ealing     |       2024       |         3         | 14.48  |
|  4   |    2    |   Richmond    |       2024       |         1         | 17.98  |
|  5   |    2    |   Richmond    |       2024       |         2         |  8.49  |
|  6   |    2    |   Richmond    |       2024       |         3         | 13.78  |

## 4. 实体Bean定义

现在我们可以为数据库视图SHOP_SALE_VIEW定义实体Bean，实际上，该定义与为普通数据库表定义实体Bean几乎相同。

在JPA中，实体Bean有一个要求，即必须具有主键，我们可以考虑两种策略来在数据库视图中定义主键。

### 4.1 物理主键

**在大多数情况下，我们可以在视图中选取一个或多个列来标识数据库视图中行的唯一性**。在我们的场景中，商店ID、年份和月份可以唯一地标识视图中的每一行。

因此，我们可以通过shop_id、transaction_year和transaction_month列派生出复合主键。在JPA中，我们必须首先定义一个单独的类来表示复合主键：

```java
public class ShopSaleCompositeId {
    private int shopId;
    private int year;
    private int month;
    // constructors, getters, setters
}
```

随后，我们使用[@EmbeddedId](https://www.baeldung.com/jpa-embedded-embeddable#attribute-overrides)将这个复合ID类嵌入到实体类中，并通过[@AttributeOverrides](https://www.baeldung.com/jpa-embedded-embeddable#attribute-overrides)标注复合ID来定义列映射：

```java
@Entity
@Table(name = "SHOP_SALE_VIEW")
public class ShopSale {
    @EmbeddedId
    @AttributeOverrides({
            @AttributeOverride( name = "shopId", column = @Column(name = "shop_id")),
            @AttributeOverride( name = "year", column = @Column(name = "transaction_year")),
            @AttributeOverride( name = "month", column = @Column(name = "transaction_month"))
    })
    private ShopSaleCompositeId id;

    @Column(name = "shop_location", length = 100)
    private String shopLocation;

    @Column(name = "total_amount")
    private BigDecimal totalAmount;

    // constructor, getters and setters
}
```

### 4.2 虚拟主键

**在某些情况下，由于缺少可以确保数据库视图中每行唯一性的列组合，因此定义物理主键是不可行的。作为一种解决方法，我们可以生成虚拟主键来模拟行唯一性**。

在我们的数据库视图定义中，我们有一个额外的列id，它利用ROW_NUMBER() OVER()生成行号作为标识符，这是我们采用虚拟主键策略时的实体类定义：

```java
@Entity
@Table(name = "SHOP_SALE_VIEW")
public class ShopSale {
    @Id
    @Column(name = "id")
    private Long id;

    @Column(name = "shop_id")
    private int shopId;

    @Column(name = "shop_location", length = 100)
    private String shopLocation;

    @Column(name = "transaction_year")
    private int year;

    @Column(name = "transaction_month")
    private int month;

    @Column(name = "total_amount")
    private BigDecimal totalAmount;

    // constructors, getters and setters
}
```

需要注意的是，**这些标识符特定于当前结果集**。重新查询时，分配给每行的行号可能会不同。因此，后续查询中的相同行号可能代表数据库视图中的不同行。

## 5. 视图Repository

根据数据库的不同，Oracle等系统可能支持可更新视图，允许在某些条件下更新数据。但是，数据库视图大多是只读的。

**对于只读数据库视图，没有必要在我们的Repository中公开数据修改方法，例如save()或delete()**。尝试调用这些方法将引发异常，因为数据库系统不支持此类操作：

```text
org.springframework.orm.jpa.JpaSystemException: could not execute statement [Feature not supported: "TableView.addRow"; SQL statement:
insert into shop_sale_view (transaction_month,shop_id,shop_location,total_amount,transaction_year,id) values (?,?,?,?,?,?) [50100-224]] [insert into shop_sale_view (transaction_month,shop_id,shop_location,total_amount,transaction_year,id) values (?,?,?,?,?,?)]
```

基于这种理由，在定义Spring Data JPA Repository时，我们将排除这些方法并仅公开数据检索方法。

### 5.1 物理主键

对于具有物理主键的视图，我们可以定义一个仅公开数据检索方法的新的基本Repository接口：

```java
@NoRepositoryBean
public interface ViewRepository<T, K> extends Repository<T, K> {
    long count();

    boolean existsById(K id);

    List<T> findAll();

    List<T> findAllById(Iterable<K> ids);

    Optional<T> findById(K id);
}
```

[@NoRepositoryBean](https://www.baeldung.com/spring-data-annotations#2-norepositorybean)注解表示此接口是基础Repository接口，并指示Spring Data JPA不要在运行时创建此接口的实例。在此Repository接口中，我们包括来自[ListCrudRepository](https://www.baeldung.com/spring-data-3-crud-repository-interfaces#1-list-based-repository-interfaces)的所有数据检索方法，并排除所有数据更改方法。

对于具有复合ID的实体Bean，我们扩展了ViewRepository并定义了一个额外方法来查询shopId的商店销售情况：

```java
public interface ShopSaleRepository extends ViewRepository<ShopSale, ShopSaleCompositeId> {
    List<ShopSale> findByIdShopId(Integer shopId);
}
```

我们将查询方法定义为findByIdShopId()而不是findByShopId()，因为它[派生自](https://www.baeldung.com/spring-data-derived-queries)ShopSale实体类中的属性id.shopId。

### 5.2 虚拟主键

当我们处理具有虚拟主键的数据库视图的Repository设计时，我们的方法略有不同，因为虚拟主键是人工的，无法真正识别数据行的唯一性。

由于这种性质，我们将定义另一个基本Repository接口，该接口也排除了通过主键查询的方法。这是因为我们使用的是虚拟主键，使用假主键检索数据是没有意义的：

```java
public interface ViewNoIdRepository<T, K> extends Repository<T, K> {
    long count();

    List<T> findAll();
}
```

随后，让我们通过将其扩展为ViewNoIdRepository来定义我们的Repository：

```java
public interface ShopSaleRepository extends ViewNoIdRepository<ShopSale, Long> {
    List<ShopSale> findByShopId(Integer shopId);
}
```

由于ShopSale实体类这次直接定义了shopId，因此我们可以在我们的Repository中使用findByShopId()。

## 6. 总结

本文对数据库视图进行了介绍，并对标准视图和物化视图进行了简要比较。

此外，我们还描述了根据数据性质在数据库视图上应用不同的主键策略。最后，我们根据所选的主键策略探讨了实体Bean和基本Repository接口的定义。