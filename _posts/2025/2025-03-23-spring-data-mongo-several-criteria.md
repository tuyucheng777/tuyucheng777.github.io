---
layout: post
title:  Spring Data MongoDB查询中的多个条件
category: spring-data
copyright: spring-data
excerpt: Spring Data MongoDB
---

## 1. 简介

在本教程中，我们将探讨如何使用Spring Data JPA在[MongoDB](https://www.baeldung.com/spring-data-mongodb-tutorial)中创建具有多个条件的查询。

## 2. 设置项目

首先，我们需要在项目中包含必要的依赖，我们将[Spring Data MongoDB Starter](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-mongodb/)依赖添加到pom.xml文件中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
    <version>3.3.1</version>
</dependency>
```

此依赖允许我们在Spring Boot项目中使用Spring Data MongoDB功能。

### 2.1 定义MongoDB文档和Repository

**接下来，我们定义一个MongoDB文档，它是一个用@Document标注的Java类**，此类映射到MongoDB中的集合。例如，让我们创建一个Product文档：

```java
@Document(collection = "products")
public class Product {
    @Id
    private String id;
    private String name;
    private String category;
    private double price;
    private boolean available;

    // Getters and setters
}
```

在Spring Data MongoDB中，我们可以创建一个自定义Repository来定义我们自己的查询方法。**通过注入[MongoTemplate](https://www.baeldung.com/spring-data-mongodb-tutorial#mongotemplate-and-mongorepository)，我们可以对MongoDB数据库执行高级操作**。此类提供了一组丰富的方法来有效地执行查询、聚合数据和处理CRUD操作：

```java
@Repository
public class CustomProductRepositoryImpl implements CustomProductRepository {
    @Autowired
    private MongoTemplate mongoTemplate;

    @Override
    public List find(Query query, Class entityClass) {
        return mongoTemplate.find(query, entityClass);
    }
}
```

### 2.2 MongoDB中的示例数据

在开始编写查询之前，假设我们的MongoDB products集合中有以下示例数据：

```json
[
    {
        "name": "MacBook Pro M3",
        "price": 1500,
        "category": "Laptop",
        "available": true
    },
    {
        "name": "MacBook Air M2",
        "price": 1000,
        "category": "Laptop",
        "available": false
    },
    {
        "name": "iPhone 13",
        "price": 800,
        "category": "Phone",
        "available": true
    }
]
```

这些数据将帮助我们有效地测试我们的查询。

## 3. 构建MongoDB查询

在Spring Data MongoDB中构建复杂查询时，我们利用andOperator()和orOperator()等方法来有效地组合多个条件，**这些方法对于创建需要文档同时或交替满足多个条件的查询至关重要**。

### 3.1 使用addOperator()

andOperator()方法用于将多个条件与AND运算符组合在一起，**这意味着，文档的所有条件都必须为true，才能与查询匹配**。当我们需要强制满足多个条件时，这很有用。

以下是我们使用andOperator()构造此查询的方法：

```java
List<Product> findProductsUsingAndOperator(String name, int minPrice, String category, boolean available) {
    Query query = new Query();
    query.addCriteria(new Criteria().andOperator(Criteria.where("name")
        .is(name), Criteria.where("price")
        .gt(minPrice), Criteria.where("category")
        .is(category), Criteria.where("available")
        .is(available)));
   return customProductRepository.find(query, Product.class);
}
```

假设我们想要检索一台名为“MacBook Pro M3”、价格高于1000美元的笔记本电脑，并确保有现货：

```java
List<Product> actualProducts = productService.findProductsUsingAndOperator("MacBook Pro M3", 1000, "Laptop", true);

assertThat(actualProducts).hasSize(1);
assertThat(actualProducts.get(0).getName()).isEqualTo("MacBook Pro M3");
```

### 3.2 使用orOperator()

相反，orOperator()方法使用OR运算符将多个条件组合在一起，**这意味着，指定的任何一个条件为true，文档就能与查询匹配**。这在检索至少符合多个条件之一的文档时非常有用。

以下是我们使用orOperator()构造此查询的方法：

```java
List<Product> findProductsUsingOrOperator(String category, int minPrice) {
    Query query = new Query();
    query.addCriteria(new Criteria().orOperator(Criteria.where("category")
        .is(category), Criteria.where("price")
        .gt(minPrice)));

    return customProductRepository.find(query, Product.class);
}
```

如果我们想要检索属于“Laptop”类别或价格高于1000美元的产品，我们可以调用该方法：

```java
actualProducts = productService.findProductsUsingOrOperator("Laptop", 1000);
assertThat(actualProducts).hasSize(2);
```

### 3.3 结合andOperator()和orOperator()

我们可以通过结合andOperator()和orOperator()方法来创建复杂的查询：

```java
List<Product> findProductsUsingAndOperatorAndOrOperator(String category1, int price1, String name1, boolean available1) {
    Query query = new Query();
    query.addCriteria(new Criteria().orOperator(
            new Criteria().andOperator(
                    Criteria.where("category").is(category1),
                    Criteria.where("price").gt(price1)),
            new Criteria().andOperator(
                    Criteria.where("name").is(name1),
                    Criteria.where("available").is(available1)
            )
    ));

    return customProductRepository.find(query, Product.class);
}
```

在此方法中，我们创建一个Query对象并使用orOperator()来定义条件的主要结构。**在此过程中，我们使用andOperator()指定两个条件**。例如，我们可以检索属于“Laptop”类别且价格高于1000美元的产品，或者名为“MacBook Pro M3”且有现货的产品：

```java
actualProducts = productService.findProductsUsingAndOperatorAndOrOperator("Laptop", 1000, "MacBook Pro M3", true);

assertThat(actualProducts).hasSize(1);
assertThat(actualProducts.get(0).getName()).isEqualTo("MacBook Pro M3");
```

### 3.4 使用链式方法

此外，我们可以利用Criteria类通过使用and()方法将多个条件链接在一起，以流式的方式构建查询。**这种方法提供了一种清晰简洁的方式来定义复杂的查询，而不会失去可读性**：

```java
List<Product> findProductsUsingChainMethod(String name1, int price1, String category1, boolean available1) {
    Criteria criteria = Criteria.where("name").is(name1)
        .and("price").gt(price1)
        .and("category").is(category1)
        .and("available").is(available1);
    return customProductRepository.find(new Query(criteria), Product.class);
}
```

当调用此方法时，我们期望找到一款名为“MacBook Pro M3”、价格超过1000美元且有现货的产品：

```java
actualProducts = productService.findProductsUsingChainMethod("MacBook Pro M3", 1000, "Laptop", true);

assertThat(actualProducts).hasSize(1);
assertThat(actualProducts.get(0).getName()).isEqualTo("MacBook Pro M3");
```

## 4. 多条件@Query注解

除了使用MongoTemplate自定义Repository之外，我们还可以创建一个扩展MongoRepository的新Repository接口，以利用[@Query](https://www.baeldung.com/spring-data-jpa-query)注解进行多条件查询。**这种方法允许我们直接在Repository中定义复杂查询，而无需以编程方式构建它们**。

我们可以在ProductRepository接口中定义一个自定义方法：

```java
public interface ProductRepository extends MongoRepository<Product, String> {
    @Query("{ 'name': ?0, 'price': { $gt: ?1 }, 'category': ?2, 'available': ?3 }")
    List<Product> findProductsByNamePriceCategoryAndAvailability(String name, double minPrice, String category, boolean available);

    @Query("{ $or: [{ 'category': ?0, 'available': ?1 }, { 'price': { $gt: ?2 } } ] }")
    List<Product> findProductsByCategoryAndAvailabilityOrPrice(String category, boolean available, double minPrice);
}
```

第一种方法findProductsByNamePriceCategoryAndAvailability()检索符合所有指定条件的产品，这包括产品的确切名称、高于指定最低限度的价格、产品所属的类别以及产品是否有库存：

```java
actualProducts = productRepository.findProductsByNamePriceCategoryAndAvailability("MacBook Pro M3", 1000, "Laptop",  true);

assertThat(actualProducts).hasSize(1);
assertThat(actualProducts.get(0).getName()).isEqualTo("MacBook Pro M3");
```

另一方面，第二种方法findProductsByCategoryAndAvailabilityOrPrice()提供了更灵活的方法，它查找属于特定类别且可用或价格高于指定最低限度的产品：

```java
actualProducts = productRepository.findProductsByCategoryAndAvailabilityOrPrice("Laptop", false, 600);

assertThat(actualProducts).hasSize(3);
```

## 5. 使用QueryDSL

**[QueryDSL](https://www.baeldung.com/querydsl-with-jpa-tutorial)是一个框架，它允许我们以编程方式构建类型安全的查询**。让我们逐步了解如何在Spring Data MongoDB项目中设置和使用QueryDSL来处理多条件查询。

### 5.1 添加QueryDSL依赖

首先，我们需要在项目中包含[QueryDSL](https://mvnrepository.com/artifact/com.querydsl/querydsl-mongodb/)，可以通过在pom.xml文件中添加以下依赖来实现这一点：

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-mongodb</artifactId>
    <version>5.1.0</version>
</dependency>
```

### 5.2 生成Q类

QueryDSL需要为我们的域对象生成辅助类，**这些类通常以“Q”前缀命名(例如QProduct)，为我们的实体字段提供类型安全的访问**。我们可以使用Maven插件自动执行此生成过程：

```xml
<plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>org.springframework.data.mongodb.repository.support.MongoAnnotationProcessor</processor>
            </configuration>
        </execution>
    </executions>
</plugin>
```

当构建过程运行此配置时，注解处理器会为我们的每个MongoDB文档生成Q类。**例如，如果我们有一个Product类，它会生成一个QProduct类**。此QProduct类提供对Product实体字段的类型安全访问，使我们能够使用QueryDSL以更结构化且无错误的方式构建查询。

接下来，我们需要修改我们的Repository以扩展QuerydslPredicateExecutor：

```java
public interface ProductRepository extends MongoRepository<Product, String>, QuerydslPredicateExecutor<Product> {
}
```

### 5.3 使用AND和QueryDSL

**在QueryDSL中，我们可以使用[Predicate](https://www.baeldung.com/jpa-and-or-criteria-predicates)接口构造复杂查询，该接口表示布尔表达式**。and()方法允许我们组合多个条件，确保文档满足所有指定的条件才能匹配查询：

```java
List<Product> findProductsUsingQueryDSLWithAndCondition(String category, boolean available, String name, double minPrice) {
    QProduct qProduct = QProduct.product;
    Predicate predicate = qProduct.category.eq(category)
        .and(qProduct.available.eq(available))
        .and(qProduct.name.eq(name))
        .and(qProduct.price.gt(minPrice));

    return StreamSupport.stream(productRepository.findAll(predicate).spliterator(), false)
        .collect(Collectors.toList());
}
```

在这个方法中，我们首先创建一个QProduct实例。然后，我们使用and()方法构造一个结合多个条件的Predicate。**最后，我们使用productRepository.findAll(predicate)执行查询，该查询根据构造的谓词检索所有匹配的产品**：

```java
actualProducts = productService.findProductsUsingQueryDSLWithAndCondition("Laptop", true, "MacBook Pro M3", 1000);

assertThat(actualProducts).hasSize(1);
assertThat(actualProducts.get(0).getName()).isEqualTo("MacBook Pro M3");
```

### 5.4 使用QueryDSL进行OR运算

我们还可以使用or()方法构造查询，该方法允许我们使用逻辑OR运算符组合多个条件。**这意味着，如果满足任何指定的条件，则文档与查询匹配**。

让我们创建一个使用带有OR条件的QueryDSL查找产品的方法：

```java
List<Product> findProductsUsingQueryDSLWithOrCondition(String category, String name, double minPrice) {
    QProduct qProduct = QProduct.product;
    Predicate predicate = qProduct.category.eq(category)
        .or(qProduct.name.eq(name))
        .or(qProduct.price.gt(minPrice));

    return StreamSupport.stream(productRepository.findAll(predicate).spliterator(), false)
        .collect(Collectors.toList());
}
```

or()方法可确保如果以下任何条件成立，产品就与查询匹配：

```java
actualProducts = productService.findProductsUsingQueryDSLWithOrCondition("Laptop", "MacBook", 800);

assertThat(actualProducts).hasSize(2);
```

### 5.5 使用QueryDSL结合AND和OR

我们还可以在谓词中组合使用and()和or()方法，**这种灵活性使我们能够指定某些条件，其中某些条件必须为true，而其他条件可以是替代条件**。以下是如何在单个查询中组合and()和or()的示例：

```java
List<Product> findProductsUsingQueryDSLWithAndOrCondition(String category, boolean available, String name, double minPrice) {
    QProduct qProduct = QProduct.product;
    Predicate predicate = qProduct.category.eq(category)
        .and(qProduct.available.eq(available))
        .or(qProduct.name.eq(name).and(qProduct.price.gt(minPrice)));

    return StreamSupport.stream(productRepository.findAll(predicate).spliterator(), false)
        .collect(Collectors.toList());
}
```

在此方法中，我们通过将条件与and()和or()组合来构造查询。这使我们能够构建一个查询，该查询将匹配特定类别中价格大于指定金额的产品或具有特定名称的可用产品：

```java
actualProducts = productService.findProductsUsingQueryDSLWithAndOrCondition("Laptop", true, "MacBook Pro M3", 1000);
assertThat(actualProducts).hasSize(3);
```

## 6. 总结

在本文中，我们探讨了在Spring Data MongoDB中构建具有多个条件的查询的各种方法。**对于具有几个条件的简单查询，由于其简单性，Criteria或链式方法可能就足够了**。但是，如果查询涉及具有多个条件和嵌套的复杂逻辑，则通常建议使用@Query注解或QueryDSL，因为它们具有更好的可读性和可维护性。