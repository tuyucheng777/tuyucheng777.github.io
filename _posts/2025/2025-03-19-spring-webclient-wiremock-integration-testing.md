---
layout: post
title:  使用WireMock进行Spring WebClient集成测试
category: spring-reactive
copyright: spring-reactive
excerpt: Spring WebClient
---

## 1. 简介

[Spring WebClient](https://www.baeldung.com/spring-5-webclient)是一个用于执行HTTP请求的非阻塞、[响应式](https://www.baeldung.com/java-reactive-systems)客户端，而[WireMock](https://www.baeldung.com/introduction-to-wiremock)是一个用于Mock基于HTTP的API的强大工具。

在本教程中，我们将了解如何在使用WebClient时利用WireMock API来存根基于HTTP的客户端请求。通过Mock外部服务的行为，我们可以确保我们的应用程序能够按预期处理外部API响应。

我们将添加所需的依赖项，然后给出一个简单的示例。最后，我们将利用WireMock API为某些情况编写一些集成测试。

## 2. 依赖和示例

首先，让我们确保我们的Spring Boot项目中有必要的依赖。

我们需要[spring-boot-starter-flux](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-webflux)用于WebClient和[spring-cloud-starter-wiremock](https://mvnrepository.com/artifact/org.springframework.cloud/spring-cloud-contract-wiremock)用于WireMock服务器。 让我们将它们添加到pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-contract-wiremock</artifactId>
    <version>4.1.2</version>
    <scope>test</scope>
</dependency>
```

现在，让我们介绍一个简单的示例，我们将与外部天气API通信以获取给定城市的天气数据。接下来让我们定义WeatherData POJO：

```java
public class WeatherData {
    private String city;
    private int temperature;
    private String description;
    ....
   //constructor
   //setters and getters
}
```

我们想使用WebClient和WireMock进行集成测试来测试此功能。

## 3. 使用WireMock API进行集成测试

**让我们首先使用WireMock和WebClient设置Spring Boot测试类**：

```kotlin
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureWireMock(port = 0)
public class WeatherServiceIntegrationTest {

    @Autowired
    private WebClient.Builder webClientBuilder;

    @Value("${wiremock.server.port}")
    private int wireMockPort;

    // Create WebClient instance with WireMock base URL
    WebClient webClient = webClientBuilder.baseUrl("http://localhost:" + wireMockPort).build();
    // ....
    // ....
}
```

**值得注意的是，@AutoConfigureWireMock会自动在随机端口上启动WireMock服务器**。此外，我们正在使用WireMock服务器的基本URL创建WebClient实例。现在，通过WebClient发出的任何请求都会转到WireMock服务器实例，如果存在正确的存根，则会发送相应的响应。

### 3.1 存根响应，包含成功信息和JSON主体

**让我们首先使用JSON请求存根HTTP调用，服务器返回200 OK**：

```java
@Test
public void  givenWebClientBaseURLConfiguredToWireMock_whenGetRequestForACity_thenWebClientRecievesSuccessResponse() {
    // Stubbing response for a successful weather data retrieval
    stubFor(get(urlEqualTo("/weather?city=London"))
            .willReturn(aResponse()
                    .withStatus(200)
                    .withHeader("Content-Type", "application/json")
                    .withBody("{\"city\": \"London\", \"temperature\": 20, \"description\": \"Cloudy\"}")));

    // Create WebClient instance with WireMock base URL
    WebClient webClient = webClientBuilder.baseUrl("http://localhost:" + wireMockPort).build();

    // Fetch weather data for London
    WeatherData weatherData = webClient.get()
            .uri("/weather?city=London")
            .retrieve()
            .bodyToMono(WeatherData.class)
            .block();
    assertNotNull(weatherData);
    assertEquals("London", weatherData.getCity());
    assertEquals(20, weatherData.getTemperature());
    assertEquals("Cloudy", weatherData.getDescription());
}
```

当通过WebClient发送对/weather?city=London的请求并使用指向WireMock端口的基本URL时，将返回存根响应，然后根据需要在我们的系统中使用该响应。

### 3.2 模拟自定义标头

有时HTTP请求需要自定义标头，**WireMock可以匹配自定义标头以提供适当的响应**。

让我们创建一个包含两个标头的存根，一个是Content-Type，另一个是X-Custom-Header，其值为“tuyucheng-header”：

```java
@Test
public void givenWebClientBaseURLConfiguredToWireMock_whenGetRequest_theCustomHeaderIsReturned() {
    //Stubbing response with custom headers
    stubFor(get(urlEqualTo("/weather?city=London"))
            .willReturn(aResponse()
                    .withStatus(200)
                    .withHeader("Content-Type", "application/json")
                    .withHeader("X-Custom-Header", "tuyucheng-header")
                    .withBody("{\"city\": \"London\", \"temperature\": 20, \"description\": \"Cloudy\"}")));

    //Create WebClient instance with WireMock base URL
    WebClient webClient = webClientBuilder.baseUrl("http://localhost:" + wireMockPort).build();

    //Fetch weather data for London
    WeatherData weatherData = webClient.get()
            .uri("/weather?city=London")
            .retrieve()
            .bodyToMono(WeatherData.class)
            .block();

    //Assert the custom header
    HttpHeaders headers = webClient.get()
            .uri("/weather?city=London")
            .exchange()
            .block()
            .headers();
    assertEquals("tuyucheng-header", headers.getFirst("X-Custom-Header"));
}
```

WireMock服务器以伦敦的存根天气数据(包括自定义标头)进行响应。

### 3.3 模拟异常

**另一个有用的测试案例是外部服务返回异常时**。WireMock服务器允许我们Mock这些异常场景，以查看系统在此类条件下的行为：

```java
@Test
public void givenWebClientBaseURLConfiguredToWireMock_whenGetRequestWithInvalidCity_thenExceptionReturnedFromWireMock() {
    //Stubbing response for an invalid city
    stubFor(get(urlEqualTo("/weather?city=InvalidCity"))
        .willReturn(aResponse()
            .withStatus(404)
            .withHeader("Content-Type", "application/json")
            .withBody("{\"error\": \"City not found\"}")));

    // Create WebClient instance with WireMock base URL
    WebClient webClient = webClientBuilder.baseUrl("http://localhost:" + wireMockPort).build();

   // Fetch weather data for an invalid city
    WebClientResponseException exception = assertThrows(WebClientResponseException.class, () -> {
        webClient.get()
        .uri("/weather?city=InvalidCity")
        .retrieve()
        .bodyToMono(WeatherData.class)
        .block();
});
```

重要的是，我们在这里测试WebClient在查询无效城市的天气数据时是否正确处理来自服务器的错误响应。**它验证在向/weather?city=InvalidCity发出请求时是否抛出了WebClientResponseException，以确保应用程序中正确处理错误**。

### 3.4 使用查询参数模拟响应

我们经常需要发送带有查询参数的请求，接下来让我们为此创建一个存根：

```java
@Test
public void givenWebClientWithBaseURLConfiguredToWireMock_whenGetWithQueryParameter_thenWireMockReturnsResponse() {
    // Stubbing response with specific query parameters
    stubFor(get(urlPathEqualTo("/weather"))
            .withQueryParam("city", equalTo("London"))
            .willReturn(aResponse()
                    .withStatus(200)
                    .withHeader("Content-Type", "application/json")
                    .withBody("{\"city\": \"London\", \"temperature\": 20, \"description\": \"Cloudy\"}")));

    //Create WebClient instance with WireMock base URL
    WebClient webClient = webClientBuilder.baseUrl("http://localhost:" + wireMockPort).build();

    WeatherData londonWeatherData = webClient.get()
            .uri(uriBuilder -> uriBuilder.path("/weather").queryParam("city", "London").build())
            .retrieve()
            .bodyToMono(WeatherData.class)
            .block();
    assertEquals("London", londonWeatherData.getCity());
}
```

### 3.5 模拟动态响应

我们来看一个例子，我们在响应主体中生成一个介于10到30度之间的随机温度值：

```java
@Test
public void givenWebClientBaseURLConfiguredToWireMock_whenGetRequest_theDynamicResponseIsSent() {
    //Stubbing response with dynamic temperature
    stubFor(get(urlEqualTo("/weather?city=London"))
            .willReturn(aResponse()
                    .withStatus(200)
                    .withHeader("Content-Type", "application/json")
                    .withBody("{\"city\": \"London\", \"temperature\": ${randomValue|10|30}, \"description\": \"Cloudy\"}")));

    //Create WebClient instance with WireMock base URL
    WebClient webClient = webClientBuilder.baseUrl("http://localhost:" + wireMockPort).build();

    //Fetch weather data for London
    WeatherData weatherData = webClient.get()
            .uri("/weather?city=London")
            .retrieve()
            .bodyToMono(WeatherData.class)
            .block();

    //Assert temperature is within the expected range
    assertNotNull(weatherData);
    assertTrue(weatherData.getTemperature() >= 10 && weatherData.getTemperature() <= 30);
}
```

### 3.6 模拟异步行为

在这里，我们将尝试通过在响应中引入一秒的模拟延迟来模拟服务可能遇到延迟或网络延迟的真实场景：

```java
@Test
public void  givenWebClientBaseURLConfiguredToWireMock_whenGetRequest_thenResponseReturnedWithDelay() {
    //Stubbing response with a delay
    stubFor(get(urlEqualTo("/weather?city=London"))
            .willReturn(aResponse()
                    .withStatus(200)
                    .withFixedDelay(1000) // 1 second delay
                    .withHeader("Content-Type", "application/json")
                    .withBody("{\"city\": \"London\", \"temperature\": 20, \"description\": \"Cloudy\"}")));

    //Create WebClient instance with WireMock base URL
    WebClient webClient = webClientBuilder.baseUrl("http://localhost:" + wireMockPort).build();

    //Fetch weather data for London
    long startTime = System.currentTimeMillis();
    WeatherData weatherData = webClient.get()
            .uri("/weather?city=London")
            .retrieve()
            .bodyToMono(WeatherData.class)
            .block();
    long endTime = System.currentTimeMillis();

    assertNotNull(weatherData);
    assertTrue(endTime - startTime >= 1000); // Assert the delay
}
```

**本质上，我们希望确保应用程序能够正常处理延迟响应，而不会超时或遇到意外错误**。

### 3.7 模拟有状态行为

接下来，**让我们结合使用WireMock场景来模拟有状态行为**。API允许我们配置存根，使其根据状态在多次调用时做出不同的响应：

```java
@Test
public void givenWebClientBaseURLConfiguredToWireMock_whenMulitpleGet_thenWireMockReturnsMultipleResponsesBasedOnState() {
    //Stubbing response for the first call
    stubFor(get(urlEqualTo("/weather?city=London"))
            .inScenario("Weather Scenario")
            .whenScenarioStateIs("started")
            .willReturn(aResponse()
                    .withStatus(200)
                    .withHeader("Content-Type", "application/json")
                    .withBody("{\"city\": \"London\", \"temperature\": 20, \"description\": \"Cloudy\"}"))
            .willSetStateTo("Weather Found"));

    // Stubbing response for the second call
    stubFor(get(urlEqualTo("/weather?city=London"))
            .inScenario("Weather Scenario")
            .whenScenarioStateIs("Weather Found")
            .willReturn(aResponse()
                    .withStatus(200)
                    .withHeader("Content-Type", "application/json")
                    .withBody("{\"city\": \"London\", \"temperature\": 25, \"description\": \"Sunny\"}")));

    //Create WebClient instance with WireMock base URL
    WebClient webClient = webClientBuilder.baseUrl("http://localhost:" + wireMockPort).build();

    //Fetch weather data for London
    WeatherData firstWeatherData = webClient.get()
            .uri("/weather?city=London")
            .retrieve()
            .bodyToMono(WeatherData.class)
            .block();

    //Assert the first response
    assertNotNull(firstWeatherData);
    assertEquals("London", firstWeatherData.getCity());
    assertEquals(20, firstWeatherData.getTemperature());
    assertEquals("Cloudy", firstWeatherData.getDescription());

    // Fetch weather data for London again
    WeatherData secondWeatherData = webClient.get()
            .uri("/weather?city=London")
            .retrieve()
            .bodyToMono(WeatherData.class)
            .block();

    // Assert the second response
    assertNotNull(secondWeatherData);
    assertEquals("London", secondWeatherData.getCity());
    assertEquals(25, secondWeatherData.getTemperature());
    assertEquals("Sunny", secondWeatherData.getDescription());
}
```

本质上，我们在名为“Weather Scenario”的同一场景中为同一URL定义了两个存根映射。但是，我们已将第一个存根配置为在场景处于“started”状态时响应伦敦的天气数据，温度为20°C，描述为“Cloudy”。

响应后，它将场景状态转换为“Weather Found”。第二个存根配置为在场景处于“Weather Found”状态时响应不同的天气数据，温度为25°C，描述为“Sunny”。

## 4. 总结

在本文中，我们讨论了使用Spring WebClient和WireMock进行集成测试的基础知识，WireMock提供了广泛的功能来存根HTTP响应以模拟各种场景。

我们快速浏览了一下结合WebClient Mock HTTP响应的一些常见场景。