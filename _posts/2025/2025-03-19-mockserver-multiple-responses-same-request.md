---
layout: post
title:  如何模拟同一请求的多个响应
category: mock
copyright: mock
excerpt: MockServer
---

## 1. 概述

在本文中，我们将探讨如何使用MockServer Mock同一请求的多个响应。

**[MockServer](https://www.baeldung.com/mockserver)通过模仿真实API的行为来Mock它，允许我们无需后端服务即可测试应用程序**。

## 2. 应用程序设置

**让我们考虑一个支付处理API，它提供了一个处理支付请求的端点**。当发起支付时，此API会调用外部银行支付服务，银行的API会以引用paymentId进行响应。使用此ID，API会通过轮询银行的API定期检查支付状态，确保支付成功处理。

让我们首先定义支付请求模型，其中包括处理支付所需的卡详细信息：

```java
public record PaymentGatewayRequest(String cardNumber, String expiryMonth, String expiryYear, String currency, int amount, String cvv) {
}
```

同样的，我们来定义支付响应模型，其中包含支付状态：

```java
public record PaymentGatewayResponse(UUID id, PaymentStatus status) {
    public enum PaymentStatus {
        PENDING,
        AUTHORIZED,
        DECLINED,
        REJECTED
    }
}
```

现在，让我们添加控制器和实现，以便与银行的支付服务集成，以提交付款和状态轮询。当付款状态从待处理状态开始，然后更新为AUTHORIZED、DECLINED或REJECTED时，API将继续轮询：

```java
@PostMapping("payment/process")
public ResponseEntity<PaymentGatewayResponse> submitPayment(@RequestBody PaymentGatewayRequest paymentGatewayRequest) throws JSONException {
    String paymentSubmissionResponse = webClient.post()
            .uri("http://localhost:9090/payment/submit")
            .body(BodyInserters.fromValue(paymentGatewayRequest))
            .retrieve()
            .bodyToMono(String.class)
            .block();

    UUID paymentId = UUID.fromString(new JSONObject(paymentSubmissionResponse).getString("paymentId"));
    PaymentGatewayResponse.PaymentStatus paymentStatus = PaymentGatewayResponse.PaymentStatus.PENDING;
    while (paymentStatus.equals(PaymentGatewayResponse.PaymentStatus.PENDING)) {
        String paymentStatusResponse = webClient.get()
                .uri("http://localhost:9090/payment/status/%s".formatted(paymentId))
                .retrieve()
                .bodyToMono(String.class)
                .block();
        paymentStatus = PaymentGatewayResponse.PaymentStatus.
                valueOf(new JSONObject(paymentStatusResponse).getString("paymentStatus"));
        logger.info("Payment Status {}", paymentStatus);
    }
    return new ResponseEntity<>(new PaymentGatewayResponse(paymentId, paymentStatus), HttpStatus.OK);
}
```

**为了测试此API并确保它轮询付款状态直到达到终止状态，我们需要能够Mock付款状态API的多个响应**。Mock响应应最初返回PENDING状态几次，然后再更新为AUTHORIZED，使我们能够有效地验证轮询机制。

## 3. 如何Mock同一请求的多个响应

测试此API的第一步是在端口9090上[启动一个Mock服务器](https://www.baeldung.com/mockserver#2-launching-via-java-api)，我们的API使用此端口与银行的支付提交和状态服务进行交互：

```java
class PaymentControllerTest {
    private ClientAndServer clientAndServer;

    private final MockServerClient mockServerClient = new MockServerClient("localhost", 9090);

    @BeforeEach
    void setup() {
        clientAndServer = startClientAndServer(9090);
    }

    @AfterEach
    void tearDown() {
        clientAndServer.stop();
    }

    // ...
}
```

接下来，让我们为付款提交端点设置一个Mock，以返回paymentId：

```java
mockServerClient
    .when(request()
        .withMethod("POST")
        .withPath("/payment/submit"))
    .respond(response()
        .withStatusCode(200)
        .withBody("{\"paymentId\": \"%s\"}".formatted(paymentId))
        .withHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE));
```

为了Mock同一请求的多个响应，我们需要将[Times](https://5-2.mock-server.com/apidocs/org/mockserver/matchers/Times.html)类与[when()](https://mock-server.com/versions/5.11.0/apidocs/org/mockserver/client/MockServerClient.html#when-org.mockserver.model.RequestDefinition-org.mockserver.matchers.Times-)方法一起使用。

**when()方法使用Times参数来指定请求应匹配的次数，这使我们能够Mock重复请求的不同响应**。

接下来，让我们Mock支付状态端点返回4次PENDING状态：

```java
mockServerClient
    .when(request()
        .withMethod("GET")
        .withPath("/payment/status/%s".formatted(paymentId)), Times.exactly(4))
    .respond(response()
        .withStatusCode(200)
        .withHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
        .withBody("{\"paymentStatus\": \"%s\"}"
        .formatted(PaymentGatewayResponse.PaymentStatus.PENDING.toString())));
```

接下来，让我们Mock支付状态端点返回AUTHORIZED：

```java
mockServerClient
    .when(request()
        .withMethod("GET")
        .withPath("/payment/status/%s".formatted(paymentId)))
    .respond(response()
        .withStatusCode(200)
        .withHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
        .withBody("{\"paymentStatus\": \"%s\"}"
        .formatted(PaymentGatewayResponse.PaymentStatus.AUTHORIZED.toString())));
```

最后，让我们向支付处理API端点发送请求以接收AUTHORIZED结果：

```text
webTestClient.post()
    .uri("http://localhost:9000/api/payment/process")
    .bodyValue(new PaymentGatewayRequest("4111111111111111", "12", "2025", "USD", 10000, "123"))
    .exchange()
    .expectStatus()
    .isOk()
    .expectBody(PaymentGatewayResponse.class)
    .value(response -> {
        Assertions.assertNotNull(response);
        Assertions.assertEquals(PaymentGatewayResponse.PaymentStatus.AUTHORIZED, response.status());
    });
```

我们应该看到日志打印四次“Payment Status PENDING”，然后打印“Payment Status AUTHORIZED”。

## 4. 总结

在本教程中，我们探讨了如何Mock同一请求的多个响应，从而使用Times类灵活地测试API。

MockServerClient中的默认when()方法使用Times.unlimited()来一致地响应所有匹配的请求。要Mock特定数量请求的响应，我们可以使用Times.exactly()。