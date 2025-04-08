---
layout: post
title:  Spring Data Cassandra中使用IN子句进行查询
category: springdata
copyright: springdata
excerpt: Spring Data Cassandra
---

## 1. 概述

在本教程中，我们将学习**如何使用[Spring Data Cassandra](https://www.baeldung.com/spring-data-cassandra-tutorial)实现查询以获取多条记录**。

我们将使用IN子句来实现查询，以便为一列指定多个值。在测试此查询时，我们还会看到一个意外错误。

最后，我们将了解根本原因并解决问题。

## 2. 在Spring Data Cassandra中使用IN运算符实现查询

假设我们需要构建一个简单的应用程序来查询Cassandra数据库以获取一条或多条记录。

我们可以在WHERE子句中使用相等条件运算符IN为一列指定多个可能的值。

### 2.1 了解IN运算符的用法

在构建应用程序之前，让我们了解该运算符的用法。

**仅当我们查询所有前面的键列是否相等时，才允许在[分区键](https://www.baeldung.com/cassandra-keys#1-partition-key)的最后一列上使用IN条件**。同样，我们可以按照相同的规则在任何聚类键列中使用它。

我们将通过product表上的示例来了解这一点：

```text
CREATE TABLE mykeyspace.product (
    product_id uuid,
    product_name text,
    description text,
    price float,
    PRIMARY KEY (product_id, product_name)
)
```

假设我们尝试查找具有相同product_id集的产品：

```text
cqlsh:mykeyspace> select * from product where product_id in (2c11bbcd-4587-4d15-bb57-4b23a546bd7e, 2c11bbcd-4587-4d15-bb57-4b23a546bd22);

 product_id                           | product_name | description     | price
--------------------------------------+--------------+-----------------+-------
 2c11bbcd-4587-4d15-bb57-4b23a546bd22 |       banana |    banana |  6.05
 2c11bbcd-4587-4d15-bb57-4b23a546bd22 |    banana v2 | banana v2 |  8.05
 2c11bbcd-4587-4d15-bb57-4b23a546bd22 |    banana v3 | banana v3 |  6.25
 2c11bbcd-4587-4d15-bb57-4b23a546bd7e |    banana chips | banana chips | 10.05
```

在上面的查询中，我们在product_id列上应用了IN子句，并且没有其他可包含的前置主键。

类似地，我们找到所有具有相同产品名称的产品：

```text
cqlsh:mykeyspace> select * from product where product_id = 2c11bbcd-4587-4d15-bb57-4b23a546bd22 and product_name in ('banana', 'banana v2');

 product_id                           | product_name | description     | price
--------------------------------------+--------------+-----------------+-------
 2c11bbcd-4587-4d15-bb57-4b23a546bd22 |       banana |    banana |  6.05
 2c11bbcd-4587-4d15-bb57-4b23a546bd22 |    banana v2 | banana v2 |  8.05
```

在上面的查询中，我们对所有前置主键(即product_id)应用了相等性检查。

我们应该注意，**where子句中包含的列的顺序应该和[主键](https://www.baeldung.com/cassandra-keys#primary-key)子句中定义的顺序相同**。

接下来，我们将在Spring Data应用程序中实现此查询。

### 2.2 Maven依赖

我们将添加[spring-boot-starter-data-cassandra](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-cassandra)依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-cassandra</artifactId>
    <version>3.1.5</version>
</dependency>
```

### 2.3 实现Spring Data Repository

让我们通过扩展CassandraRepository接口来实现查询。

首先，我们将使用一些属性来实现上述product表：

```java
@Table
public class Product {

    @PrimaryKeyColumn(name = "product_id", ordinal = 0, type = PrimaryKeyType.PARTITIONED)
    private UUID productId;

    @PrimaryKeyColumn(name = "product_name", ordinal = 1, type = PrimaryKeyType.CLUSTERED)
    private String productName;

    @Column("description")
    private String description;

    @Column("price")
    private double price;
}
```

在上面的Product类中，我们将productId标注为分区键，将productName标注为聚簇键，这两列一起构成主键。

现在，假设我们尝试查找与单个productId和多个productName匹配的所有产品。

我们将使用IN查询实现ProductRepository接口：

```java
@Repository
public interface ProductRepository extends CassandraRepository<Product, UUID> {
    @Query("select * from product where product_id = :productId and product_name in :productNames")
    List<Product> findByProductIdAndNames(@Param("productId") UUID productId, @Param("productNames") String[] productNames);
}
```

在上面的查询中，我们将productId作为UUID传递，将productNames作为数组类型传递以获取匹配的产品。

当未包含所有主键时，Cassandra不允许查询非主键列，这是因为在多个节点上执行此类查询时性能不可预测。

或者，**我们可以使用ALLOW FILTERING选项在任何列上使用IN或任何其他条件**： 

```text
cqlsh:mykeyspace> select * from product where product_name in ('banana', 'apple') and price=6.05 ALLOW FILTERING;
```

ALLOW FILTERING选项可能会对性能产生潜在的影响，因此我们应该谨慎使用它。

## 3. 实现ProductRepository的测试

现在让我们使用Cassandra容器实例为ProductRepository实现一个测试用例。

### 3.1 设置测试容器

为了进行测试，我们需要一个测试容器来运行Cassandra，我们将使用[Testcontainers](https://www.baeldung.com/spring-data-cassandra-test-containers)库设置该容器。

我们应该注意，Testcontainers库需要正在运行的[Docker](https://www.baeldung.com/ops/docker-guide)环境才能运行。

让我们添加[testcontainers](https://mvnrepository.com/artifact/org.testcontainers/testcontainers)和[testcontainers-cassandra](https://mvnrepository.com/artifact/org.testcontainers/cassandra)依赖：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>cassandra</artifactId>
    <version>1.19.0</version>
    <scope>test</scope>
</dependency>
```

### 3.2 启动测试容器

首先，我们将使用Testcontainers注解设置测试类：

```java
@Testcontainers
@SpringBootTest
class ProductRepositoryIntegrationTest { }
```

接下来，我们将定义Cassandra容器对象并将其公开在指定的端口上：

```java
@Container
private static final CassandraContainer cassandra = new CassandraContainer("cassandra:3.11.2")
    .withExposedPorts(9042);
```

最后，让我们配置一些与连接相关的属性并创建Keyspace：

```java
@BeforeAll
static void setupCassandraConnectionProperties() {
    System.setProperty("spring.cassandra.keyspace-name", "mykeyspace");
    System.setProperty("spring.cassandra.contact-points", cassandra.getHost());
    System.setProperty("spring.cassandra.port", String.valueOf(cassandra.getMappedPort(9042)));
    createKeyspace(cassandra.getCluster());
}

static void createKeyspace(Cluster cluster) {
    try (Session session = cluster.connect()) {
        session.execute("CREATE KEYSPACE IF NOT EXISTS " + KEYSPACE_NAME + " WITH replication = \n" +
                "{'class':'SimpleStrategy','replication_factor':'1'};");
    }
}
```

### 3.3 实现集成测试

为了测试，我们将使用上述ProductRepository查询检索一些现有产品。

现在，让我们完成测试并验证检索功能：

```java
UUID productId1 = UUIDs.timeBased();
Product product1 = new Product(productId1, "Apple", "Apple v1", 12.5);
Product product2 = new Product(productId1, "Apple v2", "Apple v2", 15.5);
UUID productId2 = UUIDs.timeBased();
Product product3 = new Product(productId2, "Banana", "Banana v1", 5.5);
Product product4 = new Product(productId2, "Banana v2", "Banana v2", 15.5);
productRepository.saveAll(List.of(product1, product2, product3, product4));

List<Product> existingProducts = productRepository.findByProductIdAndNames(productId1, new String[] {"Apple", "Apple v2"});
assertEquals(2, existingProducts.size());
assertTrue(existingProducts.contains(product1));
assertTrue(existingProducts.contains(product2));
```

上述测试应该会通过，然而，我们却从ProductRepository中得到了一个意外的错误：

```text
com.datastax.oss.driver.api.core.type.codec.CodecNotFoundException: Codec not found for requested operation: [List(TEXT, not frozen]
<-> [Ljava.lang.String;]
	at com.datastax.oss.driver.internal.core.type.codec.registry.CachingCodecRegistry.createCodec(CachingCodecRegistry.java:609)
	at com.datastax.oss.driver.internal.core.type.codec.registry.DefaultCodecRegistry$1.load(DefaultCodecRegistry.java:95)
	at com.datastax.oss.driver.internal.core.type.codec.registry.DefaultCodecRegistry$1.load(DefaultCodecRegistry.java:92)
	at com.datastax.oss.driver.shaded.guava.common.cache.LocalCache$LoadingValueReference.loadFuture(LocalCache.java:3527)
	....
	at com.datastax.oss.driver.internal.core.data.ValuesHelper.encodePreparedValues(ValuesHelper.java:112)
	at com.datastax.oss.driver.internal.core.cql.DefaultPreparedStatement.boundStatementBuilder(DefaultPreparedStatement.java:187)
	at org.springframework.data.cassandra.core.PreparedStatementDelegate.bind(PreparedStatementDelegate.java:59)
	at org.springframework.data.cassandra.core.CassandraTemplate$PreparedStatementHandler.bindValues(CassandraTemplate.java:1117)
	at org.springframework.data.cassandra.core.cql.CqlTemplate.query(CqlTemplate.java:541)
	at org.springframework.data.cassandra.core.cql.CqlTemplate.query(CqlTemplate.java:571)...
	at com.sun.proxy.$Proxy90.findByProductIdAndNames(Unknown Source)
	at cn.tuyucheng.taketoday.inquery.ProductRepositoryIntegrationTest$ProductRepositoryLiveTest.givenExistingProducts_whenFindByProductIdAndNames_thenProductsIsFetched(ProductRepositoryNestedLiveTest.java:113)
```

接下来，让我们详细调查一下这个错误。

### 3.4 错误根源

上述日志表明测试未能获取产品，并出现内部CodecNotFoundException异常，CodecNotFoundException异常表示未找到请求操作的查询参数类型。

异常类表明未找到cqlType及其对应javaType的编解码器：

```java
public CodecNotFoundException(@Nullable DataType cqlType, @Nullable GenericType<?> javaType) {
    this(String.format("Codec not found for requested operation: [%s <-> %s]", cqlType, javaType), (Throwable)null, cqlType, javaType);
}
```

**[CQL](https://docs.datastax.com/en/cql-oss/3.x/cql/cql_reference/cql_data_types_c.html)数据类型包括所有常见的原始类型、集合和用户定义类型**，但不允许使用数组。在Spring Data Cassandra的一些早期版本(例如1.3.x)中，也不支持List类型。

## 4. 修复查询

**为了修复该错误，我们将在ProductRepository接口中添加一个有效的查询参数类型**。

我们将请求参数类型从数组更改为List：

```java
@Query("select * from product where product_id = :productId and product_name in :productNames")
List<Product> findByProductIdAndNames(@Param("productId") UUID productId, @Param("productNames") List<String> productNames);
```

最后，我们将重新运行测试并验证查询是否有效：

```text
givenExistingProducts_whenFindByIdAndNamesIsCalled_thenProductIsReturned: 1 total, 1 passed
```

## 5. 总结

在本文中，我们学习了如何使用Spring Data Cassandra在Cassandra中实现IN查询子句。我们在测试时还遇到了一个意外错误，并了解了根本原因，我们了解了如何在方法参数中使用有效的Collection类型来解决问题。