---
layout: post
title:  Micronaut中的错误处理
category: micronaut
copyright: micronaut
excerpt: Micronaut
---

## 1. 概述

[错误处理](https://www.baeldung.com/java-global-exception-handler)是开发系统时的主要关注点之一。**在代码级别，错误处理处理我们编写的代码抛出的异常。在服务级别，错误是指我们返回的所有不成功的响应**。

**在大型系统中，以一致的方式处理类似的错误是一种很好的做法**。例如，在具有两个控制器的服务中，我们希望身份验证错误响应相似，以便我们更容易调试问题。退一步说，为了简单起见，我们可能希望系统的所有服务都具有相同的错误响应。我们可以通过使用全局异常处理程序来实现这种方法。

**在本教程中，我们将重点介绍[Micronaut](https://www.baeldung.com/micronaut)中的错误处理**。与大多数Java框架类似，Micronaut提供了一种常见的错误处理机制。我们将讨论这种机制，并在示例中演示它。

## 2. Micronaut中的错误处理

在编码中，我们唯一可以想当然的事情就是错误会发生。无论我们编写的代码有多好，测试和测试覆盖范围有多好，我们都无法避免错误。因此，如何在系统中处理它们应该是我们的主要关注点之一。通过使用状态处理程序和异常处理程序等一些框架功能，Micronaut中的错误处理变得更容易。

如果我们熟悉[Spring中的错误处理](https://www.baeldung.com/exception-handling-for-rest-with-spring)，那么使用Micronaut的方式就很容易了。**Micronaut提供了处理程序来处理抛出的异常，也提供了处理特定响应状态的处理程序**。在错误状态处理中，我们可以设置本地作用域或全局作用域。异常处理仅在全局作用域内进行。

值得一提的是，如果我们利用[Micronaut环境](https://www.baeldung.com/guide-to-micronaut-environments)功能，我们可以为不同的活动环境设置不同的全局错误处理程序。例如，如果我们有一个发布事件消息的错误处理程序，我们就可以利用活动环境并跳过本地环境上的消息发布功能。

## 3. 使用@Error注解在Micronaut中处理错误

**在Micronaut中，我们可以使用@Error注解定义错误处理程序**。此注解在方法级别定义，它应该位于@Controller注解的类内。它具有与其他控制器方法类似的一些功能，例如它可以在参数上使用请求绑定注解来访问请求标头、请求正文等。

**通过在Micronaut中使用@Error注解进行错误处理，我们可以处理异常或响应状态代码**。这与其他流行的Java框架不同，后者仅为每个异常提供处理程序。

**错误处理程序的一个特性是我们可以为它们设置一个作用域**，通过将作用域设置为全局，我们可以让一个处理程序处理整个服务的404响应。如果我们不设置作用域，则处理程序仅处理在同一个控制器中抛出的指定错误。

### 3.1 使用@Error注解处理响应错误代码

**@Error注解为我们提供了一种处理每个错误响应状态的错误的方法。这样，我们可以定义一种通用方法来处理所有HttpStatus.NOT_FOUND响应**。例如，我们可以处理的错误状态应该是在io.micronaut.http.HttpStatus枚举中定义的状态：

```java
@Controller("/notfound")
public class NotFoundController {
    @Error(status = HttpStatus.NOT_FOUND, global = true)
    public HttpResponse<JsonError> notFound(HttpRequest<?> request) {
        JsonError error = new JsonError("Page Not Found")
                .link(Link.SELF, Link.of(request.getUri()));

        return HttpResponse.<JsonError> notFound().body(error);
    }
}

public class CustomException extends RuntimeException {
    public CustomException(String message) {
        super(message);
    }
}
```

在这个控制器中，我们定义了一个用@Error注解的方法来处理HttpStatus.NOT_FOUND响应。范围设置为全局，因此所有 404 错误都应通过此方法。处理后，所有此类错误都应返回状态代码404，并带有修改后的主体，其中包含错误消息“页面未找到”和链接。

请注意，即使我们使用了@Controller注解，该控制器也没有指定任何HttpMethod，因此它不能完全像传统控制器一样工作，但它具有一些实现相似性，正如我们前面提到的。

现在假设我们有一个给出NOT_FOUND错误响应的端点：

```java
@Get("/not-found-error")
public HttpResponse<String> endpoint1() {
    return HttpResponse.notFound();
}
```

“/not-found-error”端点应该始终返回404。如果我们访问这个端点，应该触发NOT_FOUND错误处理程序：

```java
@Test
public void whenRequestThatThrows404_thenResponseIsHandled(
        RequestSpecification spec
) {
    spec.given()
            .basePath(ERRONEOUS_ENDPOINTS_PATH)
            .when()
            .get("/not-found-error")
            .then()
            .statusCode(404)
            .body(Matchers.containsString("\"message\":\"Page Not Found\",\"_links\":"));
}
```

此Micronaut测试向“/not-found-error”端点发出GET请求并返回预期的404状态代码。但是，通过断言响应主体，我们可以验证响应是否通过处理程序发送，因为错误消息是我们添加到处理程序中的消息。

需要澄清的一点是，如果我们将基本路径和路径更改为指向NotFoundController，因为此控制器中没有定义GET，只有错误，那么服务器就是抛出404器，并且处理程序仍会处理它。

### 3.2 使用@Error注解处理异常

**在Web服务中，如果未在任何地方捕获和处理异常，则控制器默认返回内部服务器错误。Micronaut中的错误处理为此类情况提供了@Error注解**。

让我们创建一个引发异常的端点和一个处理这些特定异常的处理程序：

```java
@Error(exception = UnsupportedOperationException.class)
public HttpResponse<JsonError> unsupportedOperationExceptions(HttpRequest<?> request) {
    log.info("Unsupported Operation Exception handled");
    JsonError error = new JsonError("Unsupported Operation")
            .link(Link.SELF, Link.of(request.getUri()));

    return HttpResponse.<JsonError> notFound().body(error);
}

@Get("/unsupported-operation")
public HttpResponse<String> endpoint5() {
    throw new UnsupportedOperationException();
}
```

“/unsupported-operation”端点仅抛出UnsupportedOperationException异常，unsupportedOperationExceptions方法使用@Error注解来处理这些异常。它返回404错误代码(因为不支持此资源)和带有消息“Unsupported Operation”的响应主体。请注意，此示例中的作用域是本地的，因为我们没有将其设置为全局。

如果我们访问这个端点，我们应该看到处理程序处理它并返回unsupportedOperationExceptions方法中定义的响应：

```java
@Test
public void whenRequestThatThrowsLocalUnsupportedOperationException_thenResponseIsHandled(
        RequestSpecification spec
) {
    spec.given()
            .basePath(ERRONEOUS_ENDPOINTS_PATH)
            .when()
            .get("/unsupported-operation")
            .then()
            .statusCode(404)
            .body(containsString("\"message\":\"Unsupported Operation\""));
}

@Test
public void whenRequestThatThrowsExceptionInOtherController_thenResponseIsNotHandled(
        RequestSpecification spec
) {
    spec.given()
            .basePath(PROBES_ENDPOINTS_PATH)
            .when()
            .get("/readiness")
            .then()
            .statusCode(500)
            .body(containsString("\"message\":\"Internal Server Error\""));
}
```

在第一个示例中，我们请求“/unsupported-operation”端点，该端点会抛出UnsupportedOperationException异常。由于本地处理程序位于同一个控制器中，因此我们会从处理程序获得我们期望的响应，其中包含修改后的响应错误消息“Unsupported Operation”。

在第二个示例中，我们从另一个控制器请求“/readiness”端点，该控制器也抛出了UnsupportedOperationException异常。由于此端点是在不同的控制器上定义的，因此本地处理程序不会处理该异常，因此我们得到的响应是默认的，错误代码为500。

## 4. 使用ExceptionHandler接口在Micronaut中处理错误 

**Micronaut还提供了实现ExceptionHandler接口的选项，以在全局作用域内处理特定异常**。这种方法要求每个异常一个类，这意味着默认情况下它们必须位于全局作用域内。

Micronaut提供了一些默认的异常处理程序，例如：

- jakarta.validation.ConstraintViolationException
- com.fasterxml.jackson.core.JsonProcessingException
- UnsupportedMediaException
- [以及更多](https://docs.micronaut.io/latest/guide/#builtInExceptionHandlers)

如果需要的话，这些处理程序当然可以在我们的服务上被重写。

需要考虑的一件事是异常层次结构，当我们为特定异常A创建处理程序时，扩展A的异常B也将属于同一处理程序，除非我们为该特定异常B实现另一个处理程序。有关此内容的更多详细信息请参阅以下部分。

### 4.1 处理异常

如前所述，我们可以使用ExceptionHandler接口来全局处理特定类型的异常：

```java
@Slf4j
@Produces
@Singleton
@Requires(classes = { CustomException.class, ExceptionHandler.class })
public class CustomExceptionHandler implements ExceptionHandler<CustomException, HttpResponse<String>> {
    @Override
    public HttpResponse<String> handle(HttpRequest request, CustomException exception) {
        log.info("handling CustomException: [{}]", exception.getMessage());

        return HttpResponse.ok("Custom Exception was handled");
    }
}
```

在这个类中，我们实现了接口，它使用泛型来定义我们将要处理的异常。在本例中，它是我们之前定义的CustomException。该类需要用@Requires标注，并包含异常类，也包括接口。handle方法将触发异常的请求以及异常对象作为参数，然后，我们只需在响应主体中添加自定义消息，返回200响应状态代码。

现在假设我们有一个抛出CustomException的端点：

```java
@Get("/custom-error")
public HttpResponse<String> endpoint3(@Nullable @Header("skip-error") String isErrorSkipped) {
    if (isErrorSkipped == null) {
        throw new CustomException("something else went wrong");
    }
    return HttpResponse.ok("Endpoint 3");
}
```

“/custom-error”端点接收isErrorSkipped标头，以启用/禁用抛出的异常。如果我们不包含标头，则会抛出异常：

```java
@Test
public void whenRequestThatThrowsCustomException_thenResponseIsHandled(
        RequestSpecification spec
) {
    spec.given()
            .basePath(ERRONEOUS_ENDPOINTS_PATH)
            .when()
            .get("/custom-error")
            .then()
            .statusCode(200)
            .body(is("Custom Exception was handled"));
}
```

在此测试中，我们请求“/custom-error”端点，但不包含标头。因此，会抛出CustomException异常。然后，我们通过断言我们期望从处理程序获得的响应代码和响应主体来验证处理程序是否已处理此异常。

### 4.2 根据层次结构处理异常

**对于未明确处理的异常，如果它们扩展了具有处理程序的异常，则它们将由同一处理程序隐式处理**。假设我们有一个扩展了CustomException的CustomChildException：

```java
public class CustomChildException extends CustomException {
    public CustomChildException(String message) {
        super(message);
    }
}
```

有一个端点抛出此异常：

```java
@Get("/custom-child-error")
public HttpResponse<String> endpoint4(@Nullable @Header("skip-error") String isErrorSkipped) {
    log.info("endpoint4");
    if (isErrorSkipped == null) {
        throw new CustomChildException("something else went wrong");
    }

    return HttpResponse.ok("Endpoint 4");
}
```

“/custom-child-error”端点接收isErrorSkipped标头，以启用/禁用抛出的异常。如果我们不包含标头，则会抛出异常：

```java
@Test
public void whenRequestThatThrowsCustomChildException_thenResponseIsHandled(
        RequestSpecification spec
) {
    spec.given()
            .basePath(ERRONEOUS_ENDPOINTS_PATH)
            .when()
            .get("/custom-child-error")
            .then()
            .statusCode(200)
            .body(is("Custom Exception was handled"));
}
```

此测试命中“/custom-child-error”端点并触发CustomChildException异常。从响应中，我们可以通过断言我们期望从处理程序获得的响应代码和响应主体来验证处理程序是否也处理了这个子异常。

## 5. 总结

在本文中，我们介绍了Micronaut中的错误处理。处理错误的方法有很多种，包括处理异常或处理错误响应状态代码，我们还了解了如何在不同的范围(本地和全局)上应用处理程序。最后，我们通过一些代码示例演示了讨论的所有选项，并使用Micronaut测试来验证结果。