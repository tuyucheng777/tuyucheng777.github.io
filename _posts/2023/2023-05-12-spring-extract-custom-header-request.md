---
layout: post
title:  从请求中提取自定义标头
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在这个简短的教程中，我们将探讨为Spring应用程序提取请求标头的各种方法。我们将学习如何针对特定端点执行此操作，之后，我们将创建一个HandlerInterceptor来拦截所有传入请求并提取标头。

## 2. 使用HttpServletRequest

**为了能够访问有关HTTP请求的信息，我们可以声明一个HttpServletRequest对象作为我们端点的参数**。这允许我们访问请求详细信息，例如路径、查询参数、cookies和headers。

例如，我们可以使用HttpServletRequest在收到请求时提取自定义标头。要访问某个标头，我们可以通过指定标头的key来使用getHeader()方法：

```java
@RestController
public class FooBarController {

    @GetMapping("foo")
    public String foo(HttpServletRequest request) {
        String operator = request.getHeader("operator");
        return "hello, " + operator;
    }
}
```

我们可以使用[MockMvc](https://www.baeldung.com/integration-testing-in-spring)发送包含自定义标头的GET请求。如果我们将operator标头设置为“John.Doe”，我们将期望响应为“hello, John.Doe”：

```java
@Test
void givenARequestWithOperatorHeader_whenWeCallFooEndpoint_thenOperatorIsExtracted() throws Exception {
    MockHttpServletResponse response = this.mockMvc.perform(get("/foo").header("operator", "John.Doe"))
        .andDo(print())
        .andReturn()
        .getResponse();

    assertThat(response.getContentAsString()).isEqualTo("hello, John.Doe");
}
```

但是，如果我们只需要请求中的一个特定标头，则将整个HttpServletRequest声明为参数可能会被视为违反[接口隔离原则](https://www.baeldung.com/java-interface-segregation)，即SOLID中的“I”。

## 3. 使用@RequestHeader

访问特定端点的请求标头的另一种简单方法是使用@RequestHeader注解：

```java
@GetMapping("bar")
public String bar(@RequestHeader("operator") String operator) {
    return "hello, " + operator;
}
```

**因此，我们的代码不再与整个HttpServletRequest对象耦合，我们的方法现在使用所有传入的数据作为参数**。

让我们为此端点编写一个类似的测试并期望得到相同的结果：

```java
@Test
void givenARequestWithOperatorHeader_whenWeCallBarEndpoint_thenOperatorIsExtracted() throws Exception {
    MockHttpServletResponse response = this.mockMvc.perform(get("/bar").header("operator", "John.Doe"))
        .andDo(print())
        .andReturn()
        .getResponse();

    assertThat(response.getContentAsString()).isEqualTo("hello, John.Doe");
}
```

## 4. 使用HandlerInterceptor

**对于更复杂的用例，我们可以使用HandlerInterceptor对象。优点是它可以拦截所有传入的请求并提取标头的值**。

此外，我们可以将标头的值包装在具有request作用域的Spring bean中，并将其注入到可能需要的不同组件中。

首先，让我们将operator名称包装到一个对象中：

```java
public class OperatorHolder {
    private String operator;
    // getter and setter
}
```

现在，让我们使用@Bean将其声明为一个bean。operator可能因请求而异，因此我们应该将bean作用域设置为SCOPE_REQUEST：

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public OperatorHolder operatorHolder() {
    return new OperatorHolder();
}
```

之后，我们需要创建HandlerInterceptor接口的自定义实现，并覆盖preHandle()方法：

```java
public class OperatorInterceptor implements HandlerInterceptor {
    private final OperatorHolder operatorHolder;

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String operator = request.getHeader("operator");
        operatorHolder.setOperator(operator);
        return true;
    }
    // constructor
}
```

结果，请求被拦截，operator标头被提取，OperatorHolder bean被更新。

最后，我们需要将自定义拦截器添加到Spring MVC的InterceptorRegistry中。我们可以通过实现WebMvcConfigurer并覆盖addInterceptor()的配置类来做到这一点：

```java
@Configuration
public class HeaderInterceptorConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(final InterceptorRegistry registry) {
        registry.addInterceptor(operatorInterceptor());
    }

    @Bean
    public OperatorInterceptor operatorInterceptor() {
        return new OperatorInterceptor(operatorHolder());
    }
}
```

现在要访问operator，我们只需要注入OperatorHolder bean并调用getOperator()方法：

```java
@RestController
public class BuzzController {
    private final OperatorHolder operatorHolder;

    @GetMapping("buzz")
    public String buzz() {
        return "hello, " + operatorHolder.getOperator();
    }
    // constructor
}
```

## 5. 总结

在本文中，我们探讨了访问传入HTTP请求的自定义标头的各种方法。

最初，我们通过HttpServletRequest和@RequestHeader学习了如何针对特定端点执行此操作。之后，我们看到了HandlerInterceptor如何允许我们从所有传入请求中提取标头并提供更通用的解决方案。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-mvc-5)上获得。