---
layout: post
title:  在Spring Boot中使用ProblemDetail返回错误
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本文中，我们将探讨如何使用ProblemDetail在[Spring Boot应用程序](https://www.baeldung.com/spring-boot)中返回错误。无论我们处理的是[REST API](https://www.baeldung.com/rest-with-spring-series)还是[响应流](https://www.baeldung.com/java-reactive-systems)，它都提供了一种标准化的方式将错误传达给客户端。

让我们深入探讨一下我们为什么要关心它。我们将探索在引入它之前错误处理是如何进行的，然后，我们还将讨论这个强大工具背后的规范。最后，我们将学习如何使用它来准备错误响应。

## 2. 为什么我们应该关心ProblemDetail？

**对于任何API来说，使用ProblemDetail来标准化错误响应都至关重要**。

它可以帮助客户理解和处理错误，提高API的可用性和可调试性，这将带来更好的开发人员体验和更强大的应用程序。

采用它还可以帮助提供更多信息性的错误消息，这对于维护和排除我们的服务故障至关重要。

## 3. 传统的错误处理方法
 
在ProblemDetail之前，我们经常实现自定义异常处理程序和响应实体来处理Spring Boot中的错误。我们会创建自定义错误响应结构，这导致不同API之间的不一致。

此外，这种方法需要大量的样板代码。此外，它缺乏标准化的错误表示方式，导致客户端难以统一地解析和理解错误消息。

## 4. ProblemDetail规范

ProblemDetail规范是[RFC 7807标准](https://datatracker.ietf.org/doc/html/rfc7807)的一部分。**它为错误响应定义了一致的结构，包括type、title、status、detail和instance等字段，此标准化通过提供错误信息的通用格式来帮助API开发人员和消费者**。

**实现ProblemDetail可确保我们的错误响应可预测且易于理解**，这反过来又改善了我们的API与其客户端之间的整体沟通。

接下来，我们将研究如何在Spring Boot应用程序中实现它，从基本设置和配置开始。

## 5. 在Spring Boot中实现ProblemDetail

Spring Boot中有多种方法来实现ProblemDetail。

### 5.1 使用应用程序属性启用ProblemDetail

首先，我们可以添加一个属性来启用它。对于RESTful服务，我们向application.properties添加以下属性：

```properties
spring.mvc.problemdetails.enabled=true
```

此属性允许自动使用ProblemDetail在基于MVC(Servlet堆栈)的应用程序中处理错误。

对于响应式应用程序，我们添加以下属性：

```properties
spring.webflux.problemdetails.enabled=true
```

一旦启用，Spring将使用ProblemDetail报告错误：

```json
{
    "type": "about:blank",
    "title": "Bad Request",
    "status": 400,
    "detail": "Invalid request content.",
    "instance": "/sales/calculate"
}
```

此属性在错误处理中自动提供ProblemDetail。另外，如果不需要，我们可以将其关闭。

### 5.2 在异常处理程序中实现ProblemDetail

[全局异常处理程序](https://www.baeldung.com/exception-handling-for-rest-with-spring)在Spring Boot REST应用程序中实现集中错误处理。

让我们考虑一个简单的REST服务来计算折扣价格。

它接收操作请求并返回结果。此外，它还执行[输入校验](https://www.baeldung.com/spring-boot-bean-validation)并执行业务规则。

我们来看看请求的实现：

```java
public record OperationRequest(
        @NotNull(message = "Base price should be greater than zero.")
        @Positive(message = "Base price should be greater than zero.")
        Double basePrice,
        @Nullable @Positive(message = "Discount should be greater than zero when provided.")
        Double discount) {}
```

以下是结果的实现：

```java
public record OperationResult(
        @Positive(message = "Base price should be greater than zero.") Double basePrice,
        @Nullable @Positive(message = "Discount should be greater than zero when provided.")
        Double discount,
        @Nullable @Positive(message = "Selling price should be greater than zero.")
        Double sellingPrice) {}
```

以下是无效操作异常的实现：

```java
public class InvalidInputException extends RuntimeException {

    public InvalidInputException(String s) {
        super(s);
    }
}
```

现在，让我们实现[REST控制器](https://www.baeldung.com/spring-controllers#Controller-1)：

```java
@RestController
@RequestMapping("sales")
public class SalesController {

    @PostMapping("/calculate")
    public ResponseEntity<OperationResult> calculate(@Validated @RequestBody OperationRequest operationRequest) {
        OperationResult operationResult = null;
        Double discount = operationRequest.discount();
        if (discount == null) {
            operationResult =
                    new OperationResult(operationRequest.basePrice(), null, operationRequest.basePrice());
        } else {
            if (discount.intValue() >= 100) {
                throw new InvalidInputException("Free sale is not allowed.");
            } else if (discount.intValue() > 30) {
                throw new IllegalArgumentException("Discount greater than 30% not allowed.");
            } else {
                operationResult = new OperationResult(operationRequest.basePrice(),
                        discount,
                        operationRequest.basePrice() * (100 - discount) / 100);
            }
        }
        return ResponseEntity.ok(operationResult);
    }
}
```

SalesController类在“/sales/calculate”端点处理HTTP POST请求。

它检查并验证OperationRequest对象，如果请求有效，它会计算销售价格，并考虑可选的折扣。如果折扣无效(超过100%或超过30%)，它会抛出异常。如果折扣有效，它会应用折扣计算最终价格并返回包装在[ResponseEntity](https://www.baeldung.com/spring-response-entity)中的OperationResult。

现在让我们看看如何在全局异常处理程序中实现ProblemDetail：

```java
@RestControllerAdvice
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(InvalidInputException.class)
    public ProblemDetail handleInvalidInputException(InvalidInputException e, WebRequest request) {
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, e.getMessage());
        problemDetail.setInstance(URI.create("discount"));
        return problemDetail;
    }
}
```

用@RestControllerAdvice标注的GlobalExceptionHandler类扩展了ResponseEntityExceptionHandler以在Spring Boot应用程序中提供集中异常处理。

它定义了一个方法来处理InvalidInputException异常，当发生此异常时，它会创建一个带有BAD_REQUEST状态和异常消息的ProblemDetail对象。此外，它将instance设置为URI(“discount”)以指示错误的具体上下文。

这种标准化的错误响应为客户提供了有关出了什么问题的清晰详细的信息。

**[ResponseEntityExceptionHandler](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/servlet/mvc/method/annotation/ResponseEntityExceptionHandler.html)是一个便于以标准化方式跨应用程序处理异常的类**。因此，将异常转换为有意义的HTTP响应的过程得到了简化。此外，它还提供了使用ProblemDetail开箱即用地处理常见Spring MVC异常(如MissingServletRequestParameterException、MethodArgumentNotValidException等)的方法。

### 5.3 测试ProblemDetail实现

现在让我们测试我们的功能：

```java
@Test
void givenFreeSale_whenSellingPriceIsCalculated_thenReturnError() throws Exception {
    OperationRequest operationRequest = new OperationRequest(100.0, 140.0);
    mockMvc
            .perform(MockMvcRequestBuilders.post("/sales/calculate")
                    .content(toJson(operationRequest))
                    .contentType(MediaType.APPLICATION_JSON))
            .andDo(print())
            .andExpectAll(status().isBadRequest(),
                    jsonPath("$.title").value(HttpStatus.BAD_REQUEST.getReasonPhrase()),
                    jsonPath("$.status").value(HttpStatus.BAD_REQUEST.value()),
                    jsonPath("$.detail").value("Free sale is not allowed."),
                    jsonPath("$.instance").value("discount"))
            .andReturn();
}
```

在此SalesControllerUnitTest中，我们[自动注入](https://www.baeldung.com/spring-autowire)了[MockMvc](https://www.baeldung.com/integration-testing-in-spring#3-mocking-web-context-beans)和[ObjectMapper](https://www.baeldung.com/jackson-object-mapper-tutorial)来测试SalesController。

测试方法givenFreeSale_whenSellingPriceIsCalculated_thenReturnError()模拟对“/sales/calculate”端点的POST请求，其OperationRequest包含基本价格100.0和折扣140.0。因此，这应该会触发控制器中的InvalidOperandException。

最后，我们使用ProblemDetail验证类型为BadRequest的响应，表明“Free sale is not allowed.”。

## 6. 总结

在本教程中，我们探讨了ProblemDetails、其规范以及它在Spring Boot REST应用程序中的实现。然后，我们讨论了与传统错误处理相比的优势，以及如何在Servlet和反应堆栈中使用它。