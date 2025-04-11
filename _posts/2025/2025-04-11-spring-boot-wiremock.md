---
layout: post
title:  将WireMock与Spring Boot集成
category: springboot
copyright: springboot
excerpt: WireMock
---

## 1. 简介

在开发Web应用程序时，测试REST API等外部依赖项可能具有挑战性，由于第三方服务可能不可用或返回意外数据，网络调用速度缓慢且不可靠。我们必须找到一种可靠的方法来模拟外部服务，以确保应用程序测试的一致性和可靠性。这就是[WireMock](https://wiremock.org/)的作用所在。

**WireMock是一款功能强大的HTTP Mock服务器，允许我们存根并验证HTTP请求**。它提供了一个可控的测试环境，确保我们的集成测试快速、可重复且独立于外部系统。

在本教程中，我们将探讨如何将WireMock集成到Spring Boot项目中并使用它来编写全面的测试。

## 2. Maven依赖

要将WireMock与Spring Boot一起使用，我们需要在pom.xml中包含[wiremock-spring-boot](https://mvnrepository.com/artifact/org.wiremock.integrations/wiremock-spring-boot)依赖：

```xml
<dependency>
    <groupId>org.wiremock.integrations</groupId>
    <artifactId>wiremock-spring-boot</artifactId>
    <version>3.9.0</version>
    <scope>test</scope>
</dependency>
```

该依赖提供了WireMock和Spring Boot测试框架之间的无缝集成。

## 3. 编写基本的WireMock测试

在处理更复杂的场景之前，我们先编写一个简单的WireMock测试，我们必须保证我们的Spring Boot应用程序能够与外部API正确交互。通过使用@SpringBootTest和@EnableWireMock注解，WireMock在我们的测试环境中启用。然后，我们可以定义一个简单的测试用例来验证API行为：

```java
@SpringBootTest(classes = SimpleWiremockTest.AppConfiguration.class, webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@EnableWireMock
class SimpleWiremockTest {
    @Value("${wiremock.server.baseUrl}")
    private String wireMockUrl;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void givenWireMockStub_whenGetPing_thenReturnsPong() {
        stubFor(get("/ping").willReturn(ok("pong")));

        ResponseEntity<String> response = restTemplate.getForEntity(wireMockUrl + "/ping", String.class);

        Assertions.assertEquals("pong", response.getBody());
    }

    @SpringBootApplication
    static class AppConfiguration {}
}
```

**在此测试中，我们使用@EnableWireMock注解为测试环境启动嵌入式WireMock服务器**，@Value("${wiremock.server.baseUrl}")注解从属性文件中检索WireMock的基本URL，测试方法存根端点/ping以返回带有HTTP 200状态码的“pong”。然后，我们使用TestRestTemplate发出实际HTTP请求并验证响应主体是否与预期值匹配，这可确保我们的应用程序正确与Mock的外部服务通信。

## 4. 让测试更加复杂

现在我们已经有了基本的测试，让我们扩展示例以Mock返回JSON响应并处理各种状态代码的REST API，这将帮助我们验证应用程序如何处理不同的API行为。

### 4.1 对JSON响应进行存根

REST API中的一个常见场景是返回结构化的JSON响应，我们也可以使用Wiremock存根来模拟这种情况：

```java
@Test
void givenWireMockStub_whenGetGreeting_thenReturnsMockedJsonResponse() {
    String mockResponse = "{\"message\": \"Hello, Tuyucheng!\"}";
    stubFor(get("/api/greeting")
        .willReturn(okJson(mockResponse)));

    ResponseEntity<String> response = restTemplate.getForEntity(wireMockUrl + "/api/greeting", String.class);

    Assertions.assertEquals(HttpStatus.OK, response.getStatusCode());
    Assertions.assertEquals(mockResponse, response.getBody());
}
```

在此测试中，我们存根一个到/api/greeting的GET请求，该请求返回包含问候消息的JSON响应。然后，我们请求WireMock服务器验证响应状态码是否为200 OK，以及响应主体是否符合预期的JSON结构。

### 4.2 模拟错误响应

我们都知道，事情并不总是按预期进行，尤其是在Web开发中，一些外部调用可能会返回错误。**为了做好准备，我们还可以模拟错误消息，以便我们的应用程序能够对此做出适当的响应**：

```java
@Test
void givenWireMockStub_whenGetUnknownResource_thenReturnsNotFound() {
    stubFor(get("/api/unknown").willReturn(aResponse().withStatus(404)));

    ResponseEntity<String> response = restTemplate.getForEntity(wireMockUrl + "/api/unknown", String.class);

    Assertions.assertEquals(HttpStatus.NOT_FOUND, response.getStatusCode());
}
```

## 5. 注入WireMock服务器

在更复杂的场景中，我们可能需要管理多个WireMock实例或使用特定的设置配置它们，**WireMock允许我们使用@InjectWireMock注解注入和配置多个WireMock服务器**，当我们的应用程序与众多外部服务交互并且我们希望独立Mock每个服务时，这特别有用。

### 5.1 注入单个WireMock服务器

首先将单个WireMock服务器注入到测试类中，此方法在Mock单个外部服务时很有用：

```java
@SpringBootTest(classes = SimpleWiremockTest.AppConfiguration.class,
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@EnableWireMock({
        @ConfigureWireMock(name = "user-service", port = 8081),
})
public class InjectedWiremockTest {
    @InjectWireMock("user-service")
    WireMockServer mockUserService;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void givenEmptyUserList_whenFetchingUsers_thenReturnsEmptyList() {
        mockUserService.stubFor(get("/users").willReturn(okJson("[]")));

        ResponseEntity<String> response = restTemplate.getForEntity(
                "http://localhost:8081/users",
                String.class);

        Assertions.assertEquals(HttpStatus.OK, response.getStatusCode());
        Assertions.assertEquals("[]", response.getBody());
    }
}
```

与之前使用@EnableWireMock在测试类级别启用WireMock且无需显式注入的方法不同，此方法通过注入命名的WireMock服务器实例来实现更精细的控制，@ConfigureWireMock注解明确定义了WireMock实例的名称和端口，从而可以轻松管理不同测试用例中的多个外部服务。

@InjectWireMock("user-service")允许我们直接访问WireMockServer实例，以便在我们的测试方法中动态配置和管理其行为。

### 5.2 注入多个WireMock服务器

如果我们的应用程序与多个外部服务交互，我们可能需要使用单独的WireMock实例Mock多个API，WireMock允许我们为每个实例配置和指定不同的名称和端口：

```java
@SpringBootTest(classes = SimpleWiremockTest.AppConfiguration.class,
        webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@EnableWireMock({
        @ConfigureWireMock(name = "user-service", port = 8081),
        @ConfigureWireMock(name = "product-service", port = 8082)
})
public class InjectedWiremockTest {
    @InjectWireMock("user-service")
    WireMockServer mockUserService;

    @InjectWireMock("product-service")
    WireMockServer mockProductService;

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void givenUserAndProductLists_whenFetchingUsersAndProducts_thenReturnsMockedData() {
        mockUserService.stubFor(get("/users")
                .willReturn(okJson("[{\"id\": 1, \"name\": \"John\"}]")));
        mockProductService.stubFor(get("/products")
                .willReturn(okJson("[{\"id\": 101, \"name\": \"Laptop\"}]")));

        ResponseEntity<String> userResponse = restTemplate
                .getForEntity("http://localhost:8081/users", String.class);
        ResponseEntity<String> productResponse = restTemplate
                .getForEntity("http://localhost:8082/products", String.class);

        Assertions.assertEquals(HttpStatus.OK, userResponse.getStatusCode());
        Assertions.assertEquals("[{\"id\": 1, \"name\": \"John\"}]", userResponse.getBody());

        Assertions.assertEquals(HttpStatus.OK, productResponse.getStatusCode());
        Assertions.assertEquals("[{\"id\": 101, \"name\": \"Laptop\"}]", productResponse.getBody());
    }
}
```

我们隔离了各个服务，确保对一个Mock服务器的更改不会干扰其他服务器。通过注入多个WireMock实例，我们可以完全模拟复杂的服务交互，从而提高测试的准确性和可靠性，这种方法在微服务架构中特别有用，因为微服务架构中的不同组件与各种外部服务进行通信。

## 6. 总结

WireMock是一款功能强大的工具，可用于测试Spring Boot应用程序中的外部依赖项。在本文中，我们创建了可靠、可重复且独立的测试，而无需依赖实际的第三方服务。我们从一个简单的测试开始，逐步扩展到更高级的场景，包括注入多个WireMock服务器。

借助这些技术，我们可以确保我们的应用程序正确处理外部API响应，无论它们返回的是预期数据还是错误。