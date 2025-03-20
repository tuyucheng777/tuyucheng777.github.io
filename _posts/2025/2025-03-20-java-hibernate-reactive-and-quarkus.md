---
layout: post
title:  Hibernate Reactive和Quarkus
category: quarkus
copyright: quarkus
excerpt: Quarkus
---

## 1. 概述

在Java开发中，为大规模、高性能系统创建[响应式](https://www.baeldung.com/cs/reactive-programming)应用程序已变得越来越重要。Hibernate Reactive和[Quarkus](https://www.baeldung.com/quarkus-io)是功能强大的工具，可帮助开发人员高效地构建响应式应用程序。**Hibernate Reactive是Hibernate ORM的响应式扩展，旨在与非阻塞数据库驱动程序无缝协作**。

另一方面，Quarkus是一个针对GraalVM和OpenJDK HotSpot优化的Kubernetes原生Java框架，专为构建响应式应用程序而量身定制。它们共同提供了一个强大的平台，用于创建高性能、可扩展且响应式的Java应用程序。

在本教程中，我们将通过从头开始构建响应式银行存款应用程序来深入探索[Hibernate Reactive](https://www.baeldung.com/spring-hibernate-reactive)和Quarkus。此外，我们将结合集成测试来确保应用程序的正确性和可靠性。

## 2. Quarkus中的响应式编程

**Quarkus以响应式框架而闻名，它从一开始就将响应式作为其架构的基本元素**。该框架拥有众多响应式功能，并由强大的生态系统提供支持。

值得注意的是，Quarkus通过Mutiny提供的Uni和Multi类型利用了响应式概念，展示了对异步和事件驱动编程范式的坚定承诺。

## 3. Quarkus Mutiny

**Mutiny是Quarkus中用于处理响应式功能的主要API**，大多数扩展都通过提供返回Uni和Multi的API来支持Mutiny，该API以非阻塞背压处理异步数据流。

我们的应用程序通过Quarkus提供的Uni和Multi类型利用了响应式概念。Multi表示可以异步发出多个元素的类型，类似于java.util.stream.Stream，但具有背压处理功能。

**在处理可能不受限制的数据流时，我们会使用Multi，例如实时流式传输多个银行存款**。

Uni表示最多发出一个元素或一个错误的类型，类似于java.util.concurrent.CompletableFuture，但具有更强大的组合运算符。**Uni用于我们期望单个结果或错误的场景，例如从数据库中提取单个银行存款**。

## 4. 了解PanacheEntity

当我们将Quarkus与Hibernate Reactive结合使用时，**[PanacheEntity](https://quarkus.io/guides/hibernate-orm-panache)类提供了一种简化的方法来使用最少的样板代码定义JPA实体**。通过从Hibernate的PanacheEntityBase扩展，PanacheEntity获得了响应式能力，从而能够以非阻塞方式管理实体。

这样可以有效地处理数据库操作而不会阻塞应用程序的执行，从而提高整体性能。

## 5. Maven依赖

首先，我们将[quarkus-hibernate-reactive-panache](https://mvnrepository.com/artifact/io.quarkus/quarkus-hibernate-reactive-panache)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-hibernate-reactive-panache</artifactId>
    <version>3.11.0</version>
</dependency>
```

现在我们已经配置了依赖，可以继续在示例实现中使用它。

## 6. 真实世界示例代码

银行系统通常要求较高，我们可以使用Quarkus、Hibernate和响应式编程等技术来实现关键服务，以解决这一问题。

在这个例子中，我们将重点实现两项特定的服务：创建银行存款以及列出和流式传输所有银行存款。

### 6.1 创建银行存款实体

[ORM(对象关系映射)](https://www.baeldung.com/cs/object-relational-mapping)实体对于每个基于CRUD的系统都至关重要，此实体允许将数据库对象映射到软件中的对象模型，从而方便数据操作。此外，正确定义存款实体对于确保系统平稳运行和准确的数据管理至关重要：

```java
@Entity
public class Deposit extends PanacheEntity {
    public String depositCod;
    public String currency;
    public String amount;  
    // standard setters and getters
}
```

在这个特定的例子中，Deposit类扩展了PanacheEntity类，有效地使其成为由Hibernate Reactive管理的响应式实体。

通过此扩展，Deposit类继承了CRUD(创建、读取、更新、删除)操作的方法并获得了查询功能，从而大大减少了应用程序内对手动SQL或JPQL查询的需求。这种方法简化了数据库操作管理并提高了系统的整体效率。

### 6.2 实现Repository

在大多数情况下，我们通常利用存款实体进行所有CRUD操作。但是，在这个特定场景中，我们创建一个专用的DepositRepository：

```java
@ApplicationScoped
public class DepositRepository {

    @Inject
    Mutiny.SessionFactory sessionFactory;

    @Inject
    JDBCPool client;

    public Uni<Deposit> save(Deposit deposit) {
        return sessionFactory.withTransaction((session, transaction) -> session.persist(deposit)
                .replaceWith(deposit));
    }

    public Multi<Deposit> streamAll() {
        return client.query("SELECT depositCode, currency,amount FROM Deposit ")
                .execute()
                .onItem()
                .transformToMulti(set -> Multi.createFrom()
                        .iterable(set))
                .onItem()
                .transform(Deposit::from);
    }
}
```

该Repository创建了一个自定义的save()方法和一个streamAll()方法，允许我们以Multi<Deposit\>格式检索所有存款。

### 6.3 实现REST端点

现在是时候使用[REST](https://www.baeldung.com/cs/rest-vs-soap)端点公开我们的响应方法了：

```java
@Path("/deposits")
@Produces(MediaType.APPLICATION_JSON)
@Consumes(MediaType.APPLICATION_JSON)
public class DepositResource {

    @Inject
    DepositRepository repository;

    @GET
    public Uni<Response> getAllDeposits() {
        return Deposit.listAll()
                .map(deposits -> Response.ok(deposits)
                        .build());
    }

    @POST
    public Uni<Response> createDeposit(Deposit deposit) {
        return deposit.persistAndFlush()
                .map(v -> Response.status(Response.Status.CREATED)
                        .build());
    }

    @GET
    @Path("stream")
    public Multi<Deposit> streamDeposits() {
        return repository.streamAll();
    }
}
```

我们可以看到，REST服务有三种响应式方法：getAllDeposits()，以Uni<Response\>类型返回所有存款，还有createDeposit()方法，用于创建存款。这两种方法的返回类型都是Uni<Response\>，而streamDeposits()返回的是Multi<Deposit\>。

## 7. 测试

为了确保应用程序的准确性和可靠性，我们将使用[JUnit和@QuarkusTest](https://www.baeldung.com/java-quarkus-testing)进行集成测试。此方法涉及创建测试来验证软件的各个代码或组件，以验证其是否具有正确的功能和性能。这些测试可帮助我们在开发早期识别和纠正任何问题，最终打造出更强大、更可靠的应用程序：

```java
@QuarkusTest
public class DepositResourceIntegrationTest {

    @Inject
    DepositRepository repository;

    @Test
    public void givenAccountWithDeposits_whenGetDeposits_thenReturnAllDeposits() {
        given().when()
                .get("/deposits")
                .then()
                .statusCode(200);
    }
}
```

我们讨论的测试仅侧重于验证与REST端点的成功连接并创建存款。但是，必须注意并非所有测试都如此简单。与测试常规Panache实体相比，在@QuarkusTest中测试响应式Panache实体会增加复杂性。

**这种复杂性源于API的异步特性以及所有操作必须在Vert.x事件循环中执行的严格要求**。

首先，我们将具有测试作用域的[quarkus-test-hibernate-reactive-panache](https://mvnrepository.com/artifact/io.quarkus/quarkus-test-hibernate-reactive-panache/)依赖添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-test-hibernate-reactive-panache</artifactId>
    <version>3.3.3</version>
    <scope>test</scope>
</dependency>
```

**集成测试方法应该用@RunOnVertxContext标注，这允许它们在Vert.x线程而不是主线程上运行**。

此注解对于测试必须在事件循环上执行的组件特别有用，可以更准确地模拟真实世界的条件。

**TransactionalUniAsserter是单元测试方法的预期输入类型，它的功能类似于拦截器，将每个断言方法包装在其自己的响应式事务中**。这允许更精确地管理测试环境，并确保每个断言方法在其自己的隔离上下文中运行：

```java
@Test
@RunOnVertxContext
public void givenDeposit_whenSaveDepositCalled_ThenCheckCount(TransactionalUniAsserter asserter){
    asserter.execute(() -> repository.save(new Deposit("DEP20230201","10","USD")));
    asserter.assertEquals(() -> Deposit.count(), 2L);
}
```

现在，我们需要为streamDeposit()编写一个测试，它返回Multi<Deposit\>：

```java
@Test
public void givenDepositsInDatabase_whenStreamAllDeposits_thenDepositsAreStreamedWithDelay() {
    Deposit deposit1 = new Deposit("67890", "USD", "200.0");
    Deposit deposit2 = new Deposit("67891", "USD", "300.0");
    repository.save(deposit1)
            .await()
            .indefinitely();
    repository.save(deposit2)
            .await()
            .indefinitely();
    Response response = RestAssured.get("/deposits/stream")
            .then()
            .extract()
            .response();

    // Then: the response contains the streamed books with a delay
    response.then()
            .statusCode(200);
    response.then()
            .body("$", hasSize(2));
    response.then()
            .body("[0].depositCode", equalTo("67890"));
    response.then()
            .body("[1].depositCode", equalTo("67891"));
}
```

此测试的目的是验证流式存款的功能，以被动方式使用Multi<Deposit\>类型检索所有账户。

## 8. 总结

在本文中，**我们探讨了使用Hibernate Reactive和Quarkus进行响应式编程的概念。我们讨论了Uni和Multi的基础知识，并进行了集成测试以验证代码的正确性**。

使用Hibernate Reactive和Quarkus进行响应式编程可实现高效、无阻塞的数据库操作，使应用程序更具响应性和可扩展性。通过利用这些工具，我们可以构建满足当今高性能环境需求的现代云原生应用程序。