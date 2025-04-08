---
layout: post
title:  如何在一个地方记录所有请求、响应和异常
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

日志记录在构建Web应用程序中起着至关重要的作用，它能够实现高效的调试、性能监控和错误跟踪。然而，以干净、有条理的方式实现日志记录是一个常见的挑战，尤其是在以集中方式捕获每个请求、响应和异常时。

**在本教程中，我们将在Spring Boot应用程序中实现集中式日志记录**。我们将提供详细的分步指南，涵盖所有必要的配置，并通过实际的代码示例演示该过程。

## 2. Maven依赖

首先，确保我们的pom.xml中具有必要的依赖项。我们需要[Spring Web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)以及可选的[Spring Boot Actuator](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-actuator)来实现更好的监控：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.4.1</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
    <version>3.4.1</version>
</dependency>
```

一旦设置了依赖关系，我们就可以实现日志记录逻辑了。

## 3. 使用Spring Boot Actuator进行请求日志记录

在创建自定义逻辑之前，请考虑使用[Spring Boot Actuator](https://www.baeldung.com/spring-boot-actuators)，它可以开箱即用地记录HTTP请求。**Actuator模块包含一个端点/actuator/httpexchanges(适用于Spring Boot 2.0+)，它显示对应用程序发出的最后100个HTTP请求**。除了添加spring-boot-starter-actuator依赖项之外，我们还将配置应用程序属性以公开httpexchanges端点：

```yaml
management:
    endpoints:
        web:
            exposure:
                include: httpexchanges
```

我们还将添加一个内存存储来保存跟踪数据，这使我们能够临时存储跟踪数据，而不会影响主要应用程序逻辑：

```java
@Configuration
public class HttpTraceActuatorConfiguration {
    @Bean
    public InMemoryHttpExchangeRepository createTraceRepository() {
        return new InMemoryHttpExchangeRepository();
    }
}
```

现在我们可以运行我们的应用程序并访问/actuator/httpexchanges来查看记录：

![](/assets/images/2025/springboot/springbootcentralizehttplogging01.png)

## 4. 创建自定义日志过滤器

创建自定义日志过滤器允许我们根据自己的需求定制流程，虽然Spring Boot Actuator提供了一种记录HTTP请求和响应的便捷方法，但它可能无法涵盖详细或自定义日志记录要求的所有用例。自定义过滤器允许我们记录其他详细信息、以特定方式格式化日志或将日志记录与其他监视工具集成。此外，记录Actuator等工具默认不捕获的敏感数据也很有用。例如，我们可以以任何格式记录请求标头、正文内容和响应详细信息。

### 4.1 实现自定义过滤器

此过滤器将成为所有传入HTTP请求和传出HTTP响应的集中拦截器，通过实现Filter接口，我们可以记录通过应用程序的每个请求和响应的详细信息，从而使调试和监控更加高效：

```java
@Override
public void doFilter(jakarta.servlet.ServletRequest request, jakarta.servlet.ServletResponse response, FilterChain chain) throws IOException, ServletException {
    if (request instanceof HttpServletRequest && response instanceof HttpServletResponse) {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;

        logRequest(httpRequest);

        ResponseWrapper responseWrapper = new ResponseWrapper(httpResponse);

        chain.doFilter(request, responseWrapper);

        logResponse(httpRequest, responseWrapper);
    } else {
        chain.doFilter(request, response);
    }
}
```

在我们的自定义过滤器中，我们使用两种附加方法来记录请求和响应：

```java
private void logRequest(HttpServletRequest request) {
    logger.info("Incoming Request: [{}] {}", request.getMethod(), request.getRequestURI());
    request.getHeaderNames().asIterator().forEachRemaining(header ->
        logger.info("Header: {} = {}", header, request.getHeader(header))
    );
}

private void logResponse(HttpServletRequest request, ResponseWrapper responseWrapper) throws IOException {
    logger.info("Outgoing Response for [{}] {}: Status = {}", request.getMethod(), request.getRequestURI(), responseWrapper.getStatus());
    logger.info("Response Body: {}", responseWrapper.getBodyAsString());
}
```

### 4.2  自定义响应包装器

**我们将实现一个自定义的ResponseWrapper，它允许我们在基于Servlet的Web应用程序中捕获和操作HTTP响应的响应主体**。此包装器非常方便，因为默认的HttpServletResponse在写入响应主体后不提供对响应主体的直接访问。通过拦截和存储响应内容，我们可以在将其发送到客户端之前记录或修改它：

```java
public class ResponseWrapper extends HttpServletResponseWrapper {

    private final ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    private final PrintWriter writer = new PrintWriter(new OutputStreamWriter(outputStream));

    public ResponseWrapper(HttpServletResponse response) {
        super(response);
    }

    @Override
    public ServletOutputStream getOutputStream() {
        return new ServletOutputStream() {
            @Override
            public boolean isReady() {
                return true;
            }

            @Override
            public void setWriteListener(WriteListener writeListener) {
            }

            @Override
            public void write(int b) {
                outputStream.write(b);
            }
        };
    }

    @Override
    public PrintWriter getWriter() {
        return writer;
    }

    @Override
    public void flushBuffer() throws IOException {
        super.flushBuffer();
        writer.flush();
    }

    public String getBodyAsString() {
        writer.flush();
        return outputStream.toString();
    }
}
```

### 4.3 全局处理异常

Spring Boot通过@ControllerAdvice注解提供了一种方便的方法来管理异常，该注解定义了一个全局异常处理程序。此处理程序将捕获请求处理期间发生的任何异常并记录有关它的有用信息：

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(Exception.class)
    public ResponseEntity<String> handleException(Exception ex) {
        logger.error("Exception caught: {}", ex.getMessage(), ex);
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body("An error occurred");
    }
}
```

我们还使用了ExceptionHandler注解，此注解指定该方法将处理特定类型的异常，在本例中为Exception.class，这意味着此处理程序将捕获所有异常(除非它们在应用程序的其他地方处理)。我们捕获所有异常，记录它们，并向客户端返回通用错误响应。通过记录堆栈跟踪，我们确保不会遗漏任何细节。

## 5. 测试实现

为了测试日志设置，我们可以创建一个简单的REST控制器：

```java
@RestController
@RequestMapping("/api")
public class TestController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello, World!";
    }

    @GetMapping("/error")
    public String error() {
        throw new RuntimeException("This is a test exception");
    }
}
```

访问/api/hello将记录请求和响应：

```text
INFO 19561 --- [log-all-requests] [nio-8080-exec-3] c.t.taketoday.logallrequests.LoggingFilter  : Incoming Request: [GET] /api/hello
NFO 19561 --- [log-all-requests] [nio-8080-exec-3] c.t.taketoday.logallrequests.LoggingFilter  : Header: host = localhost:8080
INFO 19561 --- [log-all-requests] [nio-8080-exec-3] c.t.taketoday.logallrequests.LoggingFilter  : Header: connection = keep-alive
…
INFO 19561 --- [log-all-requests] [nio-8080-exec-3] c.t.taketoday.logallrequests.LoggingFilter  : Outgoing Response for [GET] /api/hello: Status = 200
INFO 19561 --- [log-all-requests] [nio-8080-exec-3] c.t.taketoday.logallrequests.LoggingFilter  : Response Body: Hello, World!
```

访问/api/error将会触发异常，并将其记录在流程中：

```text
INFO 19561 --- [log-all-requests] [nio-8080-exec-7] c.t.taketoday.logallrequests.LoggingFilter  : Outgoing Response for [GET] /api/error: Status = 500
```

## 6. 总结

在本文中，我们成功实现了请求、响应和异常的集中日志记录机制。通过利用Spring Boot Actuator或使用Filter和ControllerAdvice创建自定义日志记录逻辑，我们确保我们的应用程序保持干净且可维护。

这有助于我们监控应用程序，并使我们能够在问题出现时快速解决问题。