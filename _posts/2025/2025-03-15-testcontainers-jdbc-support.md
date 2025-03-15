---
layout: post
title:  Testcontainers JDBC支持
category: test-lib
copyright: test-lib
excerpt: Testcontainers
---

## 1. 概述

在这篇短文中，我们将了解Testcontainers JDBC支持，并比较在测试中启动[Docker容器](https://www.baeldung.com/docker-java-api)的两种不同方式。

最初，我们将以编程方式管理Testcontainer的生命周期。之后，我们将通过单一配置属性简化此设置，并利用框架的JDBC支持。

## 2. 手动管理测试容器生命周期

[Testcontainers](https://www.baeldung.com/spring-boot-testcontainers-integration-test)是一个提供轻量级一次性Docker容器用于测试的框架，我们可以使用它对真实服务(如数据库、消息队列或Web服务)运行测试，而无需模拟或外部依赖。

假设我们想要使用Testcontainers来验证与PostgreSQL数据库的交互，首先，我们将[testcontainers](https://mvnrepository.com/artifact/org.testcontainers)依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>testcontainers</artifactId>
    <version>1.19.8</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <version>1.19.8</version>
    <scope>test</scope>
</dependency>
```

之后，我们必须管理容器的生命周期，按照几个简单的步骤：

- 创建容器对象
- 在所有测试之前启动容器
- 配置应用程序以连接容器
- 测试结束时停止容器

我们可以使用JUnit 5和Spring Boot注解(例如@BeforeAll、@AfterAll和@DynamicPropertyRegistry)自己实现这些步骤：

```java
@SpringBootTest
class FullTestcontainersLifecycleLiveTest {
    static PostgreSQLContainer postgres = new PostgreSQLContainer("postgres:16-alpine").withDatabaseName("test-db");

    @BeforeAll
    static void beforeAll() {
        postgres.start();
    }

    @DynamicPropertySource
    static void setProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @AfterAll
    static void afterAll() {
        postgres.stop();
    }

    // tests
}
```

**尽管此解决方案允许我们自定义特定的生命周期阶段，但它需要复杂的设置**。幸运的是，该框架提供了一种方便的解决方案，可以使用最少的配置启动容器并通过JDBC与它们通信。

## 3. 使用Testcontainers JDBC驱动程序

**当我们使用JDBC驱动程序时，Testcontainers将自动启动托管我们数据库的Docker容器**。为此，我们需要更新测试执行的JDBC URL，并使用以下模式：“jdbc:tc:<docker-image-name\>:<image-tag\>:///<database-name\>”。

让我们使用此语法在测试中更新spring.datasource.url：

```yaml
spring.datasource.url: jdbc:tc:postgresql:16-alpine:///test-db
```

不用说，这个属性可以在专用的配置文件中定义，也可以通过@SpringBootTest注解在测试本身中定义：

```java
@SpringBootTest(properties = "spring.datasource.url: jdbc:tc:postgresql:16-alpine:///test-db")
class CustomTestcontainersDriverLiveTest {
    @Autowired
    HobbitRepository theShire;

    @Test
    void whenCallingSave_thenEntityIsPersistedToDb() {
        theShire.save(new Hobbit("Frodo Baggins"));

        assertThat(theShire.findAll())
            .hasSize(1).first()
            .extracting(Hobbit::getName)
            .isEqualTo("Frodo Baggins");
    }
}
```

我们可以注意到，我们不再需要手动处理PostgreSQL容器的生命周期。**Testcontainers处理了这种复杂性，使我们能够专注于手头的测试**。

## 4. 总结

在这个简短的教程中，我们探索了启动Docker容器并通过JDBC连接到它的不同方法。首先，我们手动创建并启动容器，并将其与应用程序连接。此解决方案需要更多样板代码，但允许特定自定义。另一方面，当我们使用Testcontainers的自定义JDBC驱动程序时，我们仅用一行配置即可实现相同的设置。