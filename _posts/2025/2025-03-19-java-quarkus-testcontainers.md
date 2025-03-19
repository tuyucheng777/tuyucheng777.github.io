---
layout: post
title:  使用Testcontainers测试Quarkus应用程序的服务依赖关系
category: quarkus
copyright: quarkus
excerpt: Quarkus Testcontainers
---

## 1. 概述

在本文中，我们将看到使用[Testcontainers](https://www.baeldung.com/docker-test-containers)的强大功能，以帮助使用“实时”服务到服务测试来测试[Quarkus](https://www.baeldung.com/quarkus-io)应用程序。

在微服务架构中进行测试非常重要，而且很难重现类似生产的系统。启用测试的常见选项包括使用手动API测试工具(例如[Postman)](https://www.baeldung.com/postman-testing-collections)、[Mock服务](https://www.baeldung.com/spring-mocking-webclient)或信任内存数据库。在运行CI管道或部署到上层环境之前，我们可能不会执行真正的服务到服务交互。如果不彻底测试服务交互，这可能会带来问题并延迟交付。

**Testcontainers提供了一种解决方案，它允许启动依赖项并直接在本地环境中对其进行测试。这使我们能够利用任何容器化应用程序、服务或依赖项，并将其用于我们的测试用例中**。

## 2. 解决方案架构

我们的解决方案架构相当简单，但却使我们能够专注于强大的设置，以证明Testcontainers在服务到服务测试中的价值：

![](/assets/images/2025/quarkus/javaquarkustestcontainers01.png)

客户调用客户服务(我们正在测试的服务)，该服务返回客户数据。客户服务依赖订单服务来查询特定客户的订单。PostgreSQL数据库支持我们的每项服务。

## 3. Testcontainers解决方案

首先，让我们确保我们有适当的依赖项，[org.testcontainers.core](https://mvnrepository.com/artifact/org.testcontainers/testcontainers)和[org.testcontainers.postgresql](https://mvnrepository.com/artifact/org.testcontainers/postgresql)：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.19.6</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.6</version>
    <scope>test</scope>
</dependency>
```

Quarkus为我们提供了一些非常重要的类，我们将利用这些类针对特定的依赖项编写测试。摘自官方[Quarkus指南](https://quarkus.io/guides/getting-started-testing#quarkus-test-resource)：

> 一个非常常见的需求是在Quarkus应用程序启动进行测试之前启动Quarkus应用程序所依赖的一些服务。为了满足这一需求，Quarkus提供了@io.quarkus.test.common.QuarkusTestResource和io.quarkus.test.common.QuarkusTestResourceLifecycleManager。

接下来，让我们声明一个实现QuarkusTestResourceLifecycleManager的类，它将负责配置依赖服务及其PostgreSQL数据库：

```java
public class CustomerServiceTestcontainersManager implements QuarkusTestResourceLifecycleManager {
}
```

**现在，让我们使用Testcontainers API和模块(它们是各种依赖项的预配置实现)将它们连接起来进行测试**：

```java
private PostgreSQLContainer<?> postgreSQLContainer;
private GenericContainer<?> orderService;

```

然后，我们将配置连接和运行依赖项所需的一切：

```java
@Override
public Map<String, String> start() {
    Network network = Network.newNetwork();
    String networkAlias = "baeldung";

    postgreSQLContainer = new PostgreSQLContainer<>(DockerImageName.parse("postgres:14")).withExposedPorts(5432)
        .withDatabaseName("quarkus")
        .withUsername("quarkus")
        .withPassword("quarkus")
        .withNetwork(network)
        .withNetworkAliases(networkAlias);
    postgreSQLContainer.start();

    String jdbcUrl = String.format("jdbc:postgresql://%s:5432/quarkus", networkAlias);
    orderService = new GenericContainer<>(DockerImageName.parse("quarkus/order-service-jvm:latest")).withExposedPorts(8080)
        .withEnv("quarkus.datasource.jdbc.url", jdbcUrl)
        .withEnv("quarkus.datasource.username", postgreSQLContainer.getUsername())
        .withEnv("quarkus.datasource.password", postgreSQLContainer.getPassword())
        .withEnv("quarkus.hibernate-orm.database.generation", "drop-and-create")
        .withNetwork(network)
        .dependsOn(postgreSQLContainer)
        .waitingFor(Wait.forListeningPort());
    orderService.start();

    String orderInfoUrl = String.format("http://%s:%s/orderapi/v1", orderService.getHost(), orderService.getMappedPort(8080));
    return Map.of("quarkus.rest-client.order-api.url", orderInfoUrl);
}
```

之后，我们需要在测试完成后停止我们的服务：

```java
@Override
public void stop() {
    if (orderService != null) {
        orderService.stop();
    }
    if (postgreSQLContainer != null) {
        postgreSQLContainer.stop();
    }
}
```

这里我们可以利用Quarkus来管理测试类中的依赖生命周期：

```java
@QuarkusTestResource(CustomerServiceTestcontainersManager.class)
class CustomerResourceLiveTest {
}
```

接下来，我们编写一个参数化测试，检查我们是否从依赖服务返回了客户的数据及其订单：

```java
@ParameterizedTest
@MethodSource(value = "customerDataProvider")
void givenCustomer_whenFindById_thenReturnOrders(long customerId, String customerName, int orderSize) {
    Customer response = RestAssured.given()
            .pathParam("id", customerId)
            .get()
            .thenReturn()
            .as(Customer.class);

    Assertions.assertEquals(customerId, response.id);
    Assertions.assertEquals(customerName, response.name);
    Assertions.assertEquals(orderSize, response.orders.size());
}

private static Stream<Arguments> customerDataProvider() {
    return Stream.of(Arguments.of(1, "Customer 1", 3), Arguments.of(2, "Customer 2", 1), Arguments.of(3, "Customer 3", 0));
}
```

因此，我们运行测试时的输出表明订单服务容器已启动：

```text
Creating container for image: quarkus/order-service-jvm:latest
Container quarkus/order-service-jvm:latest is starting: 02ae38053012336ac577860997f74391eef3d4d5cd07cfffba5e27c66f520d9a
Container quarkus/order-service-jvm:latest started in PT1.199365S
```

**因此，我们成功地使用实时依赖项执行了类似生产的测试，部署了验证端到端服务行为所需的一切**。

## 4. 总结

在本教程中，我们展示了Testcontainers作为使用容器化依赖项通过网络测试Quarkus应用程序的解决方案。

测试容器通过与那些真实服务对话并为我们的测试代码提供编程API来帮助我们执行可靠且可重复的测试。