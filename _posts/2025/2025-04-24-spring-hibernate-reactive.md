---
layout: post
title:  Hibernate Reactive简介
category: springreactive
copyright: springreactive
excerpt: Hibernate Reactive
---

## 1. 概述

[响应式编程](https://www.baeldung.com/cs/reactive-programming)是一种强调异步数据流和非阻塞操作原则的编程范式，其主要目标是构建能够处理多个并发事件并实时处理的应用程序。

传统上，在命令式编程中，我们按顺序执行代码，一次执行一条指令。然而，在响应式编程中，我们可以同时处理多个事件，这使我们能够创建响应更快、可扩展性更强的应用程序。

本教程将介绍[Hibernate](https://www.baeldung.com/spring-boot-hibernate) Reactive编程，包括基础知识、它与传统命令式编程的区别，以及如何将Hibernate Reactive与[Spring Boot](https://www.baeldung.com/spring-boot-start)结合使用的分步指南。

## 2. 什么是Hibernate Reactive？

Reactive Hibernate是[Hibernate ORM框架](https://www.baeldung.com/learn-jpa-hibernate)的扩展，广泛用于将面向对象编程模型映射到关系数据库。此扩展将响应式编程概念融入Hibernate，使Java应用程序能够更高效、更灵敏地与关系数据库交互。通过集成非阻塞I/O和异步数据处理等响应式原则，Reactive Hibernate允许开发人员在其Java应用程序中创建高度可扩展且响应迅速的数据库交互。

**Hibernate Reactive扩展了流行的Hibernate ORM框架，以支持响应式编程范式，此扩展使开发人员能够构建能够处理大型数据集和高流量负载的响应式应用程序**。Hibernate Reactive的一个显著优势是它能够促进异步数据库访问，确保应用程序可以同时处理多个请求而不会造成瓶颈。

## 3. 特殊之处

在传统的数据库交互中，当程序向数据库发送请求时，它必须等待响应才能继续执行下一个任务，这种等待时间可能会累积起来，尤其是在严重依赖数据库的应用程序中，Hibernate Reactive引入了一种异步处理数据库交互的新方法。

**这意味着程序不必等待每个数据库操作完成后再继续执行，而是可以在等待数据库响应的同时执行其他任务**。

这个概念类似于在收银员处理付款时能够继续购物，Hibernate Reactive允许程序在等待数据库响应时执行其他任务，从而显著提高应用程序的整体效率、性能、资源利用率和响应能力。

**这在高流量电子商务网站等场景中尤为重要，因为应用程序必须处理许多并发用户或同时执行多个数据库操作**。

在这种情况下，Hibernate Reactive在等待数据库响应的同时继续执行其他任务的能力可以极大地提高应用程序的性能和用户体验。Hibernate Reactive为开发人员提供了构建高度可扩展和响应迅速的应用程序的工具，它使这些应用程序能够在不牺牲性能的情况下处理繁重的数据库工作负载，这证明了Hibernate Reactive在熟练的开发人员手中的潜力。解释这些要点有助于理解Hibernate Reactive与传统数据库交互的不同之处，以及它为何有利于创建现代Java应用程序。但是，需要注意的是，Hibernate Reactive可能并不适合所有用例，尤其是那些需要严格事务一致性或具有复杂数据访问模式的用例。

## 4. Maven依赖

在开始之前，我们需要将[Hibernate Reactive Core](https://mvnrepository.com/artifact/org.hibernate.reactive/hibernate-reactive-core)和[Reactive Relational Database Connectivity(R2DBC)](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-r2dbc)依赖添加到pom.xml文件中：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>org.hibernate.reactive</groupId>
    <artifactId>hibernate-reactive-core</artifactId>
</dependency>
```

## 5. 添加响应式Repository

在传统的Spring Data中，Repository同步处理数据库交互，响应式Repository[异步](https://www.baeldung.com/java-asynchronous-programming)执行这些操作，使其响应更快。

### 5.1 实体

实体与传统应用程序中使用的传统实体相同：
```java
@Entity
public class Product {
    @Id
    private Long id;
    private String name;
}
```

### 5.2 响应式Repository接口

**响应式Repository是扩展R2dbcRepository的专用接口，旨在支持使用关系型数据库([R2DBC](https://www.baeldung.com/r2dbc))进行响应式编程**。[Hibernate](https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate)中的这些Repository提供了一系列针对异步操作量身定制的方法，包括保存、查找、更新和删除。这种异步方法允许与数据库进行非阻塞交互，使其非常适合高并发和高吞吐量应用程序：
```java
@Repository
public interface ProductRepository extends R2dbcRepository<Product, Long> {
}
```

响应式Repository返回响应式类型，例如[Mono](https://www.baeldung.com/java-reactor-flux-vs-mono)(用于单个结果)或[Flux](https://www.baeldung.com/reactor-core)(用于多个结果)，允许处理异步数据库交互。

## 6. 添加响应式服务

**在Spring Boot中，[响应式服务](https://www.baeldung.com/java-reactive-systems)旨在利用响应式编程原则异步处理业务逻辑，从而提高应用程序的响应能力和可扩展性**。与传统的Spring应用程序(服务类同步执行业务逻辑)不同，响应式应用程序具有返回响应类型的服务方法，以有效地管理异步操作，这种方法可以更有效地利用资源并改善并发请求的处理：
```java
@Service
public class ProductService {
    private final ProductRepository productRepository;
    
    @Autowired
    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    public Flux<Product> findAll() {
        return productRepository.findAll();
    }

    public Mono<Product> save(Product product) {
        return productRepository.save(product);
    }
}
```

与Repository类似，服务方法返回[Mono](https://www.baeldung.com/java-reactor-flux-vs-mono)或Flux等响应类型，允许它们执行异步操作而不阻塞应用程序。

## 7. 单元测试

[响应式单元测试](https://www.baeldung.com/reactive-streams-step-verifier-test-publisher)是软件开发中必不可少的实践，专注于单独测试各个应用程序组件以确保其正常运行。特别是在响应式应用程序中，单元测试在验证控制器、服务和Repository等响应式组件的行为方面起着至关重要的作用。**对于响应式服务，单元测试对于确保服务方法表现出预期的行为非常重要，包括管理异步操作和正确处理错误条件**，这些测试有助于保证应用程序内响应式组件的可靠性和有效性：
```java
public class ProductServiceUnitTest {
    @Autowired
    private ProductService productService;

    @Autowired
    private ProductRepository productRepository;
    @BeforeEach
    void setUp() {
        productRepository.deleteAll()
                .then(productRepository.save(new Product(1L, "Product 1", "Category 1", 10.0)))
                .then(productRepository.save(new Product(2L, "Product 2", "Category 2", 15.0)))
                .then(productRepository.save(new Product(3L, "Product 3", "Category 3", 20.0)))
                .block();
    }

    @Test
    void testSave() {
        Product newProduct = new Product(4L, "Product 4", "Category 4", 24.0);

        StepVerifier.create(productService.save(newProduct))
                .assertNext(product -> {
                    assertNotNull(product.getId());
                    assertEquals("Product 4", product.getName());
                })
                .verifyComplete();
        StepVerifier.create(productService.findAll())
                .expectNextCount(4)
                .verifyComplete();
    }

    @Test
    void testFindAll() {
        StepVerifier.create(productService.findAll())
                .expectNextCount(3)
                .verifyComplete();
    }
}
```

## 8. 总结

在本教程中，我们介绍了使用Hibernate Reactive和Spring Boot构建响应式应用程序的基础知识，我们还讨论了响应式组件的优点以及如何定义和实现响应式组件，并讨论了响应式组件的单元测试，强调了创建现代、高效且可扩展的应用程序的能力。