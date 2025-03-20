---
layout: post
title:  Micronaut中基于注解的HTTP过滤器
category: micronaut
copyright: micronaut
excerpt: Micronaut
---

## 1. 概述

在本教程中，我们将介绍[Micronaut框架](https://www.baeldung.com/micronaut)提供的使用注解的HTTP过滤器。最初，Micronaut中的HTTP过滤器更接近Java EE Filter接口和[Spring Boot过滤器](https://www.baeldung.com/spring-boot-add-filter)方法。但随着最新主要版本的发布，过滤器现在可以基于注解，将请求和响应的过滤器分开。

**在本教程中，我们将研究Micronaut中的HTTP过滤器。更具体地说，我们将重点介绍版本4中引入的服务器过滤器，即基于注解的过滤器方法**。

## 2. HTTP过滤器

HTTP过滤器是作为Java EE中的接口引入的，它是所有Java Web框架中实现的“规范”，如文档所述：

> **过滤器是一个对象，它对资源(Servlet或静态内容)的请求或资源的响应(或两者)执行过滤任务**。

实现Java EE接口的过滤器有一个doFilter()方法，该方法包含3个参数：ServletRequest、ServletResponse和FilterChain，这使我们能够访问请求对象和响应，并使用链将请求和响应传递给下一个组件。时至今日，即使是较新的框架仍可能使用相同或相似的名称和参数。

过滤器在一些常见的实际应用中非常有用：

- 身份验证过滤器
- 标头过滤器(从请求中检索值或在响应中添加值)
- 指标过滤器(例如，记录请求执行时间时)
- 日志过滤器

## 3. Micronaut中的HTTP过滤器

Micronaut中的HTTP过滤器在某种程度上遵循Java EE Filter规范。例如，Micronaut的HttpFilter接口提供了一个doFilter()方法，该方法带有一个用于请求对象的参数和一个用于链对象的参数。请求参数允许我们过滤请求，然后使用链对象来处理它并返回响应。最后，如果需要，可以对响应对象进行更改。

在Micronaut 4中，引入了一些针对过滤器的新注解，这些注解仅针对请求、仅针对响应或两者提供了过滤方法。

**Micronaut使用@ServerFilter为我们的服务器接收请求和发送响应提供过滤器，但它还使用@ClientFilter为我们的REST客户端提供针对第三方系统和微服务的请求的过滤器**。

服务器过滤器具有一些使其非常灵活和有用的概念：

- 接收一些模式来匹配我们想要过滤的路径
- 可以排序，因为一些过滤器需要在其他过滤器之前执行(例如，身份验证检查过滤器应该始终放在第一个)
- 提供有关过滤可能属于错误类型的响应的选项(例如过滤Throwable对象)

在接下来的段落中，我们将更详细地讨论其中一些概念。

## 4. 过滤模式

**Micronaut中的HTTP过滤器根据路径特定于端点，要配置过滤器应用于哪个端点，我们可以设置一个模式来匹配路径**。模式可以是不同的风格，如ANT或REGEX，值是实际模式，如/endpoint*。

模式风格有不同的选项，但默认为AntPathMatcher，因为它在性能方面更高效。使用模式匹配时，Regex是一种更强大的风格，但它比Ant慢得多。因此，当Ant不支持我们所需的风格时，我们应仅将其用作最后的选择。

使用过滤器时我们需要的一些风格示例包括：

- /**将匹配任何路径
- /filters-annotations/**将匹配`filters-annotations`下的所有路径，例如/filters-annotations/endpoint1和/filters-annotations/endpoint2
- /filters-annotations/*1将匹配`filters-annotations`下的所有路径，但仅当以“1”结尾时
- **/endpoint1将匹配所有以`endpoint1`结尾的路径
- *\*/endpoint\*将匹配所有以`endpoint`结尾的路径以及末尾的任何额外内容

其中，在默认的FilterPatternStyle.ANT风格中：

- *匹配零个或多个字符
- **匹配路径中的零个或多个子目录

## 5. Micronaut中基于注解的服务器过滤器

Micronaut中基于注解的HTTP过滤器是在Micronaut主版本4中添加的，也称为过滤器方法，**过滤器方法允许我们将特定过滤器与请求或响应区分开来**。在使用基于注解的过滤器之前，我们只有一种方法可以定义过滤器，并且过滤请求或响应的方法也一样。这样我们就可以分离关注点，从而使我们的代码更简洁、更易读。

**过滤方法仍然允许我们定义一个过滤器，该过滤器既可以访问请求，也可以修改响应(如果需要)，使用FilterContinuation**。

### 5.1 过滤方法

**根据我们是否要过滤请求或响应，我们可以使用@RequestFilter或@ResponseFilter注解**。在类级别，我们仍然需要一个注解来定义过滤器，即@ServerFilter。过滤的路径和过滤器的顺序是在类级别定义的。我们还可以选择按过滤方法应用路径模式。

让我们将所有这些信息结合起来创建一个ServerFilter，它有一个过滤请求的方法和另一个过滤响应的方法：

```java
@Slf4j
@ServerFilter(patterns = { "**/endpoint*" })
public class CustomFilter implements Ordered {
    @RequestFilter
    @ExecuteOn(TaskExecutors.BLOCKING)
    public void filterRequest(HttpRequest<?> request) {
        String customRequestHeader = request.getHeaders()
                .get(CUSTOM_HEADER_KEY);
        log.info("request header: {}", customRequestHeader);
    }

    @ResponseFilter
    public void filterResponse(MutableHttpResponse<?> res) {
        res.getHeaders()
                .add(X_TRACE_HEADER_KEY, "true");
    }
}
```

filterRequest()方法带有@RequestFilter注解，并接收HTTPRequest参数，这使我们能够访问请求。然后，它读取并记录请求中的标头。在实际示例中，这可能会做更多事情，例如根据传递的标头值拒绝请求。

filterResponse()方法带有@ResponseFilter注解，并接收MutableHttpResponse参数，该参数是我们将要返回给客户端的响应对象。不过，在响应之前，此方法会在响应中添加一个标头。

请记住，请求可能已被我们拥有的另一个具有较低优先级的过滤器处理，并且可能接下来被另一个具有较高优先级的过滤器处理。同样，响应可能已被具有较高优先级的过滤器处理，并且随后将应用具有较低优先级的过滤器。有关更多信息，请参阅“过滤器优先级”段落。

### 5.2 Continuation

过滤方法是一个很好的功能，可以让我们的代码保持干净整洁。但是，**仍然需要有过滤相同请求和响应的方法。Micronaut提供了Continuation来满足这一要求**。方法上的注解与请求中的注解@RequestFilter相同，但参数不同。我们还必须在类上使用@ServerFilter注解。

我们需要访问请求并在响应中使用值的一个典型示例是分布式系统中分布式跟踪模式的跟踪标头。从高层次上讲，我们使用标头来跟踪请求，以便我们了解如果返回错误，它在哪个步骤失败了。为此，我们需要在每个请求/消息中传递“request-id”或“trace-id”，如果服务与另一个服务通信，它会传递相同的值：

```java
@Slf4j
@ServerFilter(patterns = { "**/endpoint*" })
@Order(1)
public class RequestIDFilter implements Ordered {
    @RequestFilter
    @ExecuteOn(TaskExecutors.BLOCKING)
    public void filterRequestIDHeader(
            HttpRequest<?> request,
            FilterContinuation<MutableHttpResponse<?>> continuation
    ) {
        String requestIdHeader = request.getHeaders().get(REQUEST_ID_HEADER_KEY);
        if (requestIdHeader == null || requestIdHeader.trim().isEmpty()) {
            requestIdHeader = UUID.randomUUID().toString();
            log.info(
                    "request ID not received. Created and will return one with value: [{}]",
                    requestIdHeader
            );
        } else {
            log.info("request ID received. Request ID: [{}]", requestIdHeader);
        }

        MutableHttpResponse<?> res = continuation.proceed();

        res.getHeaders().add(REQUEST_ID_HEADER_KEY, requestIdHeader);
    }
}
```

filterRequestIDHeader()方法带有@RequestFilter注解，并具有一个HttpRequest和一个FilterContinuation参数。我们从请求参数获取对请求的访问权限，并检查“Request-ID”标头是否有值。如果没有，我们将创建一个值，并在任何情况下记录该值。

通过使用continuation.proceed()方法，我们可以访问响应对象。然后，我们在响应中添加与“Request-ID”标头相同的标头和值，以传播到客户端。

### 5.3 过滤顺序

在许多用例中，让特定过滤器在其他过滤器之前或之后执行是有意义的。Micronaut中的HTTP过滤器提供了两种方法来处理过滤器执行的顺序。一种是@Order注解，另一种是实现Ordered接口，两者都是在类级别。

排序的工作方式是，我们提供一个int值，它是过滤器的执行顺序。对于请求过滤器，它很简单。顺序-5将在顺序2之前执行，顺序2将在顺序4之前执行。对于响应过滤器，情况正好相反。顺序4将首先应用，然后是顺序2，最后是顺序-5。

当我们实现接口时，我们需要手动重写getOrder()方法，它默认为0：

```java
@Filter(patterns = { "**/*1" })
public class PrivilegedUsersEndpointFilter implements HttpServerFilter, Ordered {
    // filter methods ommited

    @Override
    public int getOrder() {
        return 3;
    }
}
```

当我们使用注解时，我们只需要设置值：

```java
@ServerFilter(patterns = { "**/endpoint*" })
@Order(1)
public class RequestIDFilter implements Ordered {
    // filter methods ommited
}
```

请注意，测试@Order注解和实现Ordered接口的组合会导致不当行为，因此选择两种方法中的一种并将其应用于任何地方是一种很好的做法。

## 6. 总结

在本教程中，我们研究了过滤器的一般概念以及Micronaut中的HTTP过滤器，我们看到了实现过滤器的不同选项以及一些实际用例。然后，我们展示了基于注解的过滤器的示例，包括仅请求过滤器、仅响应过滤器以及两者。最后，我们花了一些时间讨论路径模式和过滤器顺序等关键概念。