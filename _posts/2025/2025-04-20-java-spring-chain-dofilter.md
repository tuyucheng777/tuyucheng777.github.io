---
layout: post
title:  chain.doFilter()在Spring Filter中起什么作用？
category: designpattern
copyright: designpattern
excerpt: Spring Filter
---

## 1. 简介

在本教程中，主要重点是了解Spring框架中chain.doFilter()方法的用途。

**为了更好地理解，我们首先会探讨什么是过滤器、什么是过滤器链，以及一些使用过滤器的良好用例**。然后，我们会讨论chain.doFilter()方法的用途和重要性。此外，我们还会学习如何在Spring中创建自定义过滤器。

最后，我们将探讨与行为型设计模式之一[责任链](https://www.baeldung.com/chain-of-responsibility-pattern)的关联。

## 2. Spring中的过滤器是什么？

在Spring应用程序中，过滤器基于Java Servlet过滤器，后者表示拦截请求和响应的对象。**过滤器是Java Servlet API的一部分，在Web应用程序中发挥着重要作用，因为它们位于客户端和服务器处理逻辑之间**。

使用它们，我们可以在请求到达Servlet之前或生成响应之后执行任务。过滤器的常见用例包括：

- 身份验证和授权
- 审计和日志记录
- 请求/响应修改

虽然过滤器并非Spring框架的一部分，但它与Spring框架完全兼容。我们可以将它们注册为[Spring Bean](https://www.baeldung.com/spring-bean)，并在应用程序中使用它们。Spring提供了一些过滤器的实现，其中一些常见的包括[OncePerRequestFilter](https://www.baeldung.com/spring-onceperrequestfilter)和[CorsFilter](https://www.baeldung.com/spring-cors#cors-with-spring-security)。

## 3. 理解chain.doFilter()方法

在我们了解chain.doFilter()方法之前，首先了解过滤器链的概念及其在过滤过程中的作用非常重要。

**过滤器链表示应用于传入请求或传出响应的顺序处理逻辑流**，换句话说，它是用于预处理请求或后处理响应的过滤器集合。这些过滤器按明确定义的顺序排列，确保每个过滤器都能在将请求或响应传递到链中的下一阶段之前执行其处理逻辑。

为了定义过滤器链，我们使用Java Servlet API中的[FilterChain](https://jakarta.ee/specifications/platform/11/apidocs/jakarta/servlet/filterchain)接口，其中包含我们感兴趣的方法。检查该方法的签名，我们可以看到请求和响应对象被定义为输入参数：

```java
void doFilter(ServletRequest request, ServletResponse response) throws IOException, ServletException;
```

**chain.doFilter()方法将请求和响应传递给过滤器链中的下一个过滤器**，如果过滤器链中没有剩余的过滤器，则请求将被转发到目标资源(通常是一个Servlet)，并将响应发送给客户端。

### 3.1 为什么调用chain.doFilter()很重要？

由此可见，方法chain.doFilter()至关重要，因为它确保请求能够继续通过链中的所有过滤器。**如果我们忽略该方法调用，请求将无法继续执行后续过滤器或Servlet**，这可能会导致应用程序出现意外行为，因为后续过滤器处理的任何逻辑都无法到达。

另一方面，在身份验证或授权失败的情况下，跳过方法调用并中断链可能会对我们有利。

## 4. 责任链模式

现在我们了解了chain.doFilter()的功能，接下来我们简单看一下它与责任链模式的联系。我们不会深入探讨细节，而是描述一下责任链模式是什么以及它与我们主题的关系。

责任链模式是一种专注于对象间交互的[设计模式](https://www.baeldung.com/design-patterns-series)，**它解释了如何按顺序组织处理程序(即独立的处理组件)来处理请求**。每个处理程序执行特定的任务，然后决定是否将请求传递给下一个处理程序进行进一步处理。

**如果我们将模式逻辑与Servlet过滤器进行比较，我们会发现过滤器是责任链模式的一个真实示例**，每个过滤器都充当一个独立的处理程序，负责部分处理逻辑。

使用此模式的好处包括灵活性，因为我们可以在不修改其他组件的情况下添加、删除或重新排序过滤器。此外，责任链模式允许过滤器专注于单个任务，从而改善了关注点分离。

## 5. 实现自定义过滤器

实现自定义过滤器相当简单，只需遵循几个步骤即可。在我们的示例中，我们将创建两个带有日志语句的简单过滤器，并展示chain.doFilter()方法的实际用法。

### 5.1 创建过滤器

首先，我们必须实现[Filter](https://jakarta.ee/specifications/platform/11/apidocs/jakarta/servlet/filter)接口并重写方法doFilter()。

需要注意的是，此方法与过滤器链中的doFilter()方法不同。**过滤器的doFilter()方法是过滤器具体处理逻辑的入口点，而chain.doFilter()用于将请求和响应传递给链中的下一个过滤器**。

以下是我们如何实现过滤器并使用[@Order](https://www.baeldung.com/spring-order)注解来排列它们：

```java
@Order(1)
@Component
public class FirstFilter implements Filter {

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        LOG.info("Processing the First Filter");
        // Omit chain.doFilter() on purpose
    }
}

@Order(2)
@Component
public class SecondFilter implements Filter {

    @Override
    public void doFilter( ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
        LOG.info("Processing the Second Filter");
        chain.doFilter(request, response);
    }
}
```

### 5.2 chain.doFilter()方法的实际作用

此时，在我们的应用程序中执行任何请求时，我们都可以看到响应未返回。原因是我们省略了chain.doFilter()调用，从而破坏了过滤链。如果我们观察控制台，我们会注意到第二个过滤器的日志丢失了：

```text
11:02:35.253 [main] INFO  c.t.taketoday.chaindofilter.FirstFilter - Processing the First Filter
```

要恢复过滤器链，我们应该在第一个过滤器中包含一个方法调用：

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException, ServletException {
    LOG.info("Processing the First Filter");
    chain.doFilter(request, response);
}
```

在这种情况下，第一个过滤器成功地将请求和响应传递给第二个过滤器，并且过滤器链具有连续的流动。

让我们再次观察控制台并确认日志语句是否存在：

```text
11:02:59.330 [main] INFO  c.t.taketoday.chaindofilter.FirstFilter - Processing the First Filter
11:02:59.330 [main] INFO  c.t.taketoday.chaindofilter.SecondFilter - Processing the Second Filter
```

### 5.3 注册过滤器

值得注意的是，我们使用@Component注解来标注过滤器，以创建一个Spring Bean，此步骤非常重要，因为它允许我们在Servlet容器中注册过滤器。

还有[另一种注册过滤器的方法](https://www.baeldung.com/spring-boot-add-filter#1-filter-with-url-pattern)，它提供了更多的控制和自定义功能，可以通过FilterRegistrationBean类来实现。使用这个类，我们可以指定URL模式并定义过滤器的顺序。

## 6. 总结

在本文中，我们探索了过滤器、过滤器链以及chain.doFilter()方法的正确用法。此外，我们还了解了如何在Spring中创建和注册自定义过滤器。

此外，我们讨论了与责任链模式的联系，并强调了过滤器提供的灵活性和关注点分离。