---
layout: post
title:  使用Jersey进行异常处理
category: webmodules
copyright: webmodules
excerpt: Jersey
---

## 1. 简介

在本教程中，我们将了解使用[Jersey](https://www.baeldung.com/jersey-rest-api-with-spring)(一种[JAX-RS实现](https://www.baeldung.com/jax-rs-response))处理[异常](https://www.baeldung.com/java-exceptions)的不同方法。

**JAX-RS为我们提供了许多处理异常的机制，我们可以选择并组合这些机制**。处理REST异常是构建更好的API的重要一步，在我们的用例中，我们将构建一个用于购买股票的API。

## 2. 场景设置

我们的最小设置包括创建一个[Repository](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)、几个Bean和一些端点。首先是我们的资源配置，在那里，我们将使用@ApplicationPath和我们的端点包定义起始URL：

```java
@ApplicationPath("/exception-handling/*")
public class ExceptionHandlingConfig extends ResourceConfig {
    public ExceptionHandlingConfig() {
        packages("cn.tuyucheng.taketoday.jersey.exceptionhandling.rest");
    }
}
```

### 2.1 Bean类

我们只需要两个Bean：Stock和Wallet，这样我们就可以保存Stock并购买它们。对于我们的Stock，我们只需要一个price属性来帮助验证。更重要的是，我们的Wallet类将具有验证方法来帮助构建我们的场景：

```java
public class Wallet {
    private String id;
    private Double balance = 0.0;

    // getters and setters

    public Double addBalance(Double amount) {
        return balance += amount;
    }

    public boolean hasFunds(Double amount) {
        return (balance - amount) >= 0;
    }
}
```

### 2.2 端点

类似地，我们的API将有两个端点，它们将定义保存和检索Bean的标准方法：

```java
@Path("/stocks")
public class StocksResource {
    // POST and GET methods
}

@Path("/wallets")
public class WalletsResource {
    // POST and GET methods
}
```

例如，让我们看看StocksResource中的GET方法：

```java
@GET
@Path("/{ticker}")
@Produces(MediaType.APPLICATION_JSON)
public Response get(@PathParam("ticker") String id) {
    Optional<Stock> stock = stocksRepository.findById(id);
    stock.orElseThrow(() -> new IllegalArgumentException("ticker"));

    return Response.ok(stock.get())
            .build();
}
```

在GET方法中，我们抛出了第一个异常。我们稍后再处理它，以便我们能够看到它的影响。

## 3. 当抛出异常时会发生什么？

当发生未处理的异常时，我们可能会暴露有关应用程序内部的敏感信息。如果我们尝试使用不存在的StocksResource中的GET方法，我们会得到类似这样的页面：

![](/assets/images/2025/webmodules/javaexceptionhandlingjersey01.png)

此页面显示应用程序服务器和版本，这可能有助于潜在攻击者利用漏洞。此外，还有关于我们的类名和行号的信息，这也可能对攻击者有帮助。最重要的是，**这些信息大部分对API用户来说都是无用的，给人留下了不好的印象**。

为了帮助控制异常响应，JAX-RS提供了类ExceptionMapper和WebApplicationException，让我们看看它们是如何工作的。

## 4. 使用WebApplicationException自定义异常

使用WebApplicationException，我们可以创建自定义异常，**这种特殊类型的RuntimeException允许我们定义响应状态和实体**，我们首先创建一个设置消息和状态的InvalidTradeException：

```java
public class InvalidTradeException extends WebApplicationException {
    public InvalidTradeException() {
        super("invalid trade operation", Response.Status.NOT_ACCEPTABLE);
    }
}
```

另外值得一提的是，JAX-RS为常见的HTTP状态码定义了WebApplicationException的子类，其中包括NotAllowedException、BadRequestException等有用的异常。**但是，当我们需要更复杂的错误消息时，我们可以返回JSON响应**。

### 4.1 JSON异常

我们可以创建简单的Java类并将它们包含在我们的Response中，在我们的示例中，我们有一个subject属性，我们将使用它来包装上下文数据：

```java
public class RestErrorResponse {
    private Object subject;
    private String message;

    // getters and setters
}
```

由于此异常并非用来操纵的，因此我们不必担心subject的类型。

### 4.2 充分利用一切

为了了解如何使用自定义异常，让我们定义一种购买股票的方法：

```java
@POST
@Path("/{wallet}/buy/{ticker}")
@Produces(MediaType.APPLICATION_JSON)
public Response postBuyStock(@PathParam("wallet") String walletId, @PathParam("ticker") String id) {
    Optional<Stock> stock = stocksRepository.findById(id);
    stock.orElseThrow(InvalidTradeException::new);

    Optional<Wallet> w = walletsRepository.findById(walletId);
    w.orElseThrow(InvalidTradeException::new);

    Wallet wallet = w.get();
    Double price = stock.get()
            .getPrice();

    if (!wallet.hasFunds(price)) {
        RestErrorResponse response = new RestErrorResponse();
        response.setSubject(wallet);
        response.setMessage("insufficient balance");
        throw new WebApplicationException(Response.status(Status.NOT_ACCEPTABLE)
                .entity(response)
                .build());
    }

    wallet.addBalance(-price);
    walletsRepository.save(wallet);

    return Response.ok(wallet)
            .build();
}
```

在这个方法中，我们使用了目前创建的所有东西。如果股票或钱包不存在，我们会抛出InvalidTradeException。如果资金不足，我们会构建一个包含Wallet的RestErrorResponse，并将其作为WebApplicationException抛出。

### 4.3 用例示例

首先，让我们创建一个股票：

```shell
$ curl 'http://localhost:8080/jersey/exception-handling/stocks' -H 'Content-Type: application/json' -d '{
    "id": "STOCK",
    "price": 51.57
}'

{"id": "STOCK", "price": 51.57}
```

然后使用钱包购买：

```shell
$ curl 'http://localhost:8080/jersey/exception-handling/wallets' -H 'Content-Type: application/json' -d '{
    "id": "WALLET",
    "balance": 100.0
}'

{"balance": 100.0, "id": "WALLET"}
```

之后，我们将使用钱包购买股票：

```shell
$ curl -X POST 'http://localhost:8080/jersey/exception-handling/wallets/WALLET/buy/STOCK'

{"balance": 48.43, "id": "WALLET"}
```

我们将在响应中获得更新后的余额。此外，如果我们尝试再次购买，我们将获得详细的RestErrorResponse：

```json
{
    "message": "insufficient balance",
    "subject": {
        "balance": 48.43,
        "id": "WALLET"
    }
}
```

## 5. 使用ExceptionMapper处理未处理的异常

需要澄清的是，抛出WebApplicationException不足以摆脱默认错误页面，我们必须为Response指定一个实体，而InvalidTradeException则不需要。通常，尽管我们尽力处理所有情况，但仍可能会发生未处理的异常；因此，最好先处理这些情况。**使用ExceptionMapper，我们为特定类型的异常定义捕获点，并在提交之前修改Response**：

```java
public class ServerExceptionMapper implements ExceptionMapper<WebApplicationException> {
    @Override
    public Response toResponse(WebApplicationException exception) {
        String message = exception.getMessage();
        Response response = exception.getResponse();
        Status status = response.getStatusInfo().toEnum();

        return Response.status(status)
                .entity(status + ": " + message)
                .type(MediaType.TEXT_PLAIN)
                .build();
    }
}
```

例如，我们只是将异常信息重新传递到我们的Response中，它将准确显示我们返回的内容。随后，我们可以在构建Response之前通过检查状态码来做进一步的操作：

```java
switch (status) {
    case METHOD_NOT_ALLOWED:
        message = "HTTP METHOD NOT ALLOWED";
        break;
    case INTERNAL_SERVER_ERROR:
        message = "internal validation - " + exception;
        break;
    default:
        message = "[unhandled response code] " + exception;
}
```

### 5.1 处理特定异常

如果存在经常抛出的特定异常，我们也可以为其创建一个ExceptionMapper。**在我们的端点中，我们抛出一个IllegalArgumentException来进行简单的验证，因此让我们从它的映射器开始**。这次，使用JSON响应：

```java
public class IllegalArgumentExceptionMapper implements ExceptionMapper<IllegalArgumentException> {
    @Override
    public Response toResponse(IllegalArgumentException exception) {
        return Response.status(Response.Status.EXPECTATION_FAILED)
                .entity(build(exception.getMessage()))
                .type(MediaType.APPLICATION_JSON)
                .build();
    }

    private RestErrorResponse build(String message) {
        RestErrorResponse response = new RestErrorResponse();
        response.setMessage("an illegal argument was provided: " + message);
        return response;
    }
}
```

现在，每次我们的应用程序中出现未处理的IllegalArgumentException时，IllegalArgumentExceptionMapper都会处理它。

### 5.2 配置

为了激活我们的异常映射器，我们必须回到Jersey资源配置并注册它们：

```java
public ExceptionHandlingConfig() {
    // packages ...
    register(IllegalArgumentExceptionMapper.class);
    register(ServerExceptionMapper.class);
}
```

这足以摆脱默认错误页面；**然后，根据抛出的内容，当发生未处理的异常时，Jersey将使用我们的一个异常映射器**。例如，当尝试获取不存在的股票时，将使用IllegalArgumentExceptionMapper：

```shell
$ curl 'http://localhost:8080/jersey/exception-handling/stocks/NONEXISTENT'

{"message": "an illegal argument was provided: ticker"}
```

同样，对于其他未处理的异常，将使用更广泛的ServerExceptionMapper。例如，当我们使用错误的HTTP方法时：

```shell
$ curl -X POST 'http://localhost:8080/jersey/exception-handling/stocks/STOCK'

Method Not Allowed: HTTP 405 Method Not Allowed
```

## 6. 总结

在本文中，我们了解了使用Jersey处理异常的多种方式。此外，还了解了它的重要性以及如何配置它。之后，我们构建了一个可以应用它们的简单场景。因此，我们现在拥有一个更友好、更安全的API。