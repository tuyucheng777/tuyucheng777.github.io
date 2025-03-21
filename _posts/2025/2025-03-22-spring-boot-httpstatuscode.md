---
layout: post
title:  在Spring Boot 3中将HttpStatus迁移到HttpStatusCode
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 概述

在本文中，我们将介绍如何在Spring Boot应用程序中使用HttpStatusCode，重点介绍版本3.3.3中引入的最新增强功能。**通过这些增强功能，HttpStatusCode已合并到[HttpStatus](https://www.baeldung.com/spring-mvc-controller-custom-http-status-code)实现中，从而简化了我们使用HTTP状态码的方式**。

这些改进的主要目的是提供一种更灵活、更可靠的方法来处理标准和自定义HTTP状态代码，使我们在处理HTTP响应时具有更高的灵活性和可扩展性，同时保持向后兼容性。

## 2. HttpStatus枚举器

在Spring 3.3.3之前，HTTP状态码在HttpStatus中以[枚举](https://www.baeldung.com/a-guide-to-java-enums)形式表示。这限制了自定义或非标准HTTP状态码的使用，因为枚举是一组固定的预定义值。

**尽管HttpStatus类尚未被弃用，但一些返回原始整数状态码的枚举和方法(如getRawStatusCode()和rawStatusCode())现已被弃用**。不过，使用[@ResponseStatus](https://www.baeldung.com/spring-response-status)注解来提高代码的可读性仍然是推荐的方法。

我们还可以将HttpStatus与HttpStatusCode结合使用，以实现更灵活的HTTP响应管理：

```java
@GetMapping("/exception")
public ResponseEntity<String> resourceNotFound() {
    HttpStatus statusCode = HttpStatus.NOT_FOUND;
    if (statusCode.is4xxClientError()) {
        return new ResponseEntity<>("Resource not found", HttpStatusCode.valueOf(404));
    }
    return new ResponseEntity<>("Resource found", HttpStatusCode.valueOf(200));
}
```

## 3. HttpStatusCode接口

HttpStatusCode接口旨在支持除HttpStatus中预定义的状态码之外的自定义状态码，它有8个实例方法：

- is1xxInformational()
- is2xxSuccessful()
- is3xxRedirection()
- is4xxClientError()
- is5xxServerError()
- isError()
- isSameCodeAs(HttpStatusCode other)
- value()

这些方法不仅增加了处理不同HTTP状态的灵活性，而且简化了检查响应类别的流程，提高了状态码管理的清晰度和效率。

让我们看一个例子来了解我们如何在实践中使用它们：

```java
@GetMapping("/resource")
public ResponseEntity successStatusCode() {
    HttpStatusCode statusCode = HttpStatusCode.valueOf(200);
    if (statusCode.is2xxSuccessful()) {
        return new ResponseEntity("Success", statusCode);
    }

    return new ResponseEntity("Moved Permanently", HttpStatusCode.valueOf(301));
}
```

### 3.1 静态方法valueOf(int)

此方法返回给定int值的HttpStatusCode对象。**输入参数必须是3位正数，否则会引发IllegalArgumentException**。

valueOf()将状态码映射到HttpStatus中的相应枚举值。如果没有现有条目与提供的状态代码匹配，则该方法默认返回DefaultHttpStatusCode的实例。

DefaultHttpStatusCode类实现了HttpStatusCode，并提供了value()方法的直接实现，该方法返回用于初始化它的原始整数值。这种方法可确保所有HTTP状态码(无论是自定义还是非标准)都易于使用：

```java
@GetMapping("/custom-exception")
public ResponseEntity<String> goneStatusCode() {
    throw new ResourceGoneException("Resource Gone", HttpStatusCode.valueOf(410));
}
```

## 4. 在自定义异常中使用HttpStatusCode

接下来，让我们看看如何在[ExceptionHandler](https://www.baeldung.com/exception-handling-for-rest-with-spring)中使用带有HttpStatusCode的自定义异常。我们将使用@ControllerAdvice注解在所有控制器中全局处理异常，并使用@ExceptionHandler注解来管理自定义异常的实例。

**这种方法将错误处理集中在Spring MVC应用程序中，使代码更清晰、更易于维护**。


### 4.1 @ControllerAdvice和@ExceptionHandler

@ControllerAdvice全局处理异常，而@ExceptionHandler管理自定义异常实例以返回具有异常消息和状态码的一致HTTP响应。

让我们看看如何在实践中使用这两个注解：

```java
@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(CustomException.class)
    public ResponseEntity<String> handleGoneException(CustomException e) {
        return new ResponseEntity<>(e.getMessage(), e.getStatusCode());
    }
}
```

### 4.2 自定义异常

接下来，让我们定义一个扩展RuntimeException并包含HttpStatusCode字段的CustomException类，启用自定义消息和HTTP状态码，以便更精确地处理错误：

```java
public class CustomException extends RuntimeException {

    private final HttpStatusCode statusCode;

    public CustomException(String message, HttpStatusCode statusCode) {
        super(message);
        this.statusCode = statusCode;
    }

    public HttpStatusCode getStatusCode() {
        return statusCode;
    }
}
```

## 5. 总结

HttpStatus枚举包含一组有限的标准HTTP状态码，在旧版本的Spring中，这些代码在大多数情况下都能很好地工作。但是，它们在定义自定义状态码方面缺乏灵活性。

Spring Boot 3.3.3引入了HttpStatusCode，通过允许我们定义自定义状态码来解决此限制。这提供了一种更灵活的方式来处理HTTP状态码，并为常用状态码提供实例方法，例如is2xxSuccessful()和is3xxRedirection()，最终允许对响应处理进行更精细的控制。