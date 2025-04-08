---
layout: post
title:  在Spring Boot中到达控制器之前修改请求主体
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们将学习如何在HTTP请求到达Spring Boot应用程序中的控制器之前对其进行修改。Web应用程序和RESTful Web服务通常使用此技术来解决常见问题，例如在传入的HTTP请求到达实际控制器之前对其进行转换或丰富，这促进了松耦合并大大减少了开发工作量。

## 2. 使用过滤器修改请求

应用程序通常必须执行通用操作，例如身份验证、日志记录、转义HTML字符等。**[过滤器](https://www.baeldung.com/spring-boot-add-filter)是一个绝佳的选择处理在任何Servlet容器中运行的应用程序的这些一般问题**。让我们看看过滤器是如何工作的：

![](/assets/images/2025/springboot/springbootchangerequestbodybeforecontroller01.png)

**在Spring Boot应用程序中，过滤器可以注册为按特定顺序调用，以便**：

- **修改请求**
- **记录请求**
- **检查身份验证请求或一些恶意脚本**
- **决定拒绝或将请求转发到下一个过滤器或控制器**

假设我们想要转义HTTP请求正文中的所有HTML字符以防止[XSS攻击](https://www.baeldung.com/cs/cross-site-scripting-xss-explained)，我们首先定义过滤器：

```java
@Component
@Order(1)
public class EscapeHtmlFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        filterChain.doFilter(new HtmlEscapeRequestWrapper((HttpServletRequest) servletRequest), servletResponse);
    }
}
```

Order注解中的值1表示所有HTTP请求首先通过过滤器EscapeHtmlFilter。我们还可以在Spring Boot配置类中定义的[FilterRegistrationBean](https://www.baeldung.com/spring-boot-add-filter#1-filter-with-url-pattern)的帮助下注册过滤器，通过这个，我们也可以定义过滤器的URL模式。

doFilter()方法将原始ServletRequest包装在自定义包装器EscapeHtmlRequestWrapper中：

```java
public class EscapeHtmlRequestWrapper extends HttpServletRequestWrapper {
    private String body = null;
    public HtmlEscapeRequestWrapper(HttpServletRequest request) throws IOException {
        super(request);
        this.body = this.escapeHtml(request);
    }

    @Override
    public ServletInputStream getInputStream() throws IOException {
        final ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(body.getBytes());
        ServletInputStream servletInputStream = new ServletInputStream() {
            @Override
            public int read() throws IOException {
                return byteArrayInputStream.read();
            }
            //Other implemented methods...
        };
        return servletInputStream;
    }

    @Override
    public BufferedReader getReader() {
        return new BufferedReader(new InputStreamReader(this.getInputStream()));
    }
}
```

**包装器是必要的，因为我们无法修改原始HTTP请求。如果没有这个，Servlet容器将拒绝请求**。

在自定义包装器中，我们重写了方法getInputStream()以返回新的ServletInputStream。基本上，我们在使用escapeHtml()方法转义HTML字符后为其分配了修改后的请求正文。

让我们定义一个UserController类：

```java
@RestController
@RequestMapping("/")
public class UserController {
    @PostMapping(value = "save")
    public ResponseEntity<String> saveUser(@RequestBody String user) {
        logger.info("save user info into database");
        ResponseEntity<String> responseEntity = new ResponseEntity<>(user, HttpStatus.CREATED);
        return responseEntity;
    }
}
```

对于此演示，控制器返回它在端点/save上收到的请求主体user。

让我们看看过滤器是否有效：

```java
@Test
void givenFilter_whenEscapeHtmlFilter_thenEscapeHtml() throws Exception {
    Map<String, String> requestBody = Map.of(
            "name", "James Cameron",
            "email", "<script>alert()</script>james@gmail.com"
    );

    Map<String, String> expectedResponseBody = Map.of(
            "name", "James Cameron",
            "email", "&lt;script&gt;alert()&lt;/script&gt;james@gmail.com"
    );

    ObjectMapper objectMapper = new ObjectMapper();

    mockMvc.perform(MockMvcRequestBuilders.post(URI.create("/save"))
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(requestBody)))
            .andExpect(MockMvcResultMatchers.status().isCreated())
            .andExpect(MockMvcResultMatchers.content().json(objectMapper.writeValueAsString(expectedResponseBody)));
}
```

嗯，过滤器在到达UserController类中定义的URL /save之前成功转义了HTML字符。

## 3. 使用Spring AOP

**RequestBodyAdvice接口以及Spring框架的注解@RestControllerAdvice有助于将全局通知应用于Spring应用程序中的所有REST控制器**，让我们使用它们在HTTP请求到达控制器之前转义其中的HTML字符：

```java
@RestControllerAdvice
public class EscapeHtmlAspect implements RequestBodyAdvice {
    @Override
    public HttpInputMessage beforeBodyRead(HttpInputMessage inputMessage,
                                           MethodParameter parameter, Type targetType, Class<? extends HttpMessageConverter<?>> converterType) throws IOException {
        InputStream inputStream = inputMessage.getBody();
        return new HttpInputMessage() {
            @Override
            public InputStream getBody() throws IOException {
                return new ByteArrayInputStream(escapeHtml(inputStream).getBytes(StandardCharsets.UTF_8));
            }

            @Override
            public HttpHeaders getHeaders() {
                return inputMessage.getHeaders();
            }
        };
    }

    @Override
    public boolean supports(MethodParameter methodParameter,
                            Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {
        return true;
    }

    @Override
    public Object afterBodyRead(Object body, HttpInputMessage inputMessage,
                                MethodParameter parameter, Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {
        return body;
    }

    @Override
    public Object handleEmptyBody(Object body, HttpInputMessage inputMessage,
                                  MethodParameter parameter, Type targetType, Class<? extends HttpMessageConverter<?>> converterType) {
        return body;
    }
}
```

**方法beforeBodyRead()在HTTP请求到达控制器之前被调用**，因此我们要转义其中的HTML字符。**support()方法返回true这意味着它将通知应用于所有REST控制器**。

让我们看看它是否有效：

```java
@Test
void givenAspect_whenEscapeHtmlAspect_thenEscapeHtml() throws Exception {

    Map<String, String> requestBody = Map.of(
            "name", "James Cameron",
            "email", "<script>alert()</script>james@gmail.com"
    );

    Map<String, String> expectedResponseBody = Map.of(
            "name", "James Cameron",
            "email", "&lt;script&gt;alert()&lt;/script&gt;james@gmail.com"
    );

    ObjectMapper objectMapper = new ObjectMapper();

    mockMvc.perform(MockMvcRequestBuilders.post(URI.create("/save"))
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(requestBody)))
            .andExpect(MockMvcResultMatchers.status().isCreated())
            .andExpect(MockMvcResultMatchers.content().json(objectMapper.writeValueAsString(expectedResponseBody)));
}
```

正如预期的那样，所有HTML字符都被转义了。

我们还可以创建[自定义AOP注解](https://www.baeldung.com/spring-aop-annotation)，它可以在控制器方法上使用，以更精细的方式应用通知。

## 4. 使用拦截器修改请求

Spring拦截器是一个可以拦截传入的HTTP请求并在控制器处理它们之前对其进行处理的类。拦截器用于各种目的，例如身份验证、授权、日志记录和缓存。**此外，拦截器特定于Spring MVC框架，它们可以访问Spring ApplicationContext**。

让我们看看拦截器是如何工作的：

![](/assets/images/2025/springboot/springbootchangerequestbodybeforecontroller02.png)

DispatcherServlet将HTTP请求转发到拦截器。此外，拦截器处理后可以将请求转发给控制器或拒绝它。因此，**存在一种普遍的误解，认为拦截器可以更改HTTP请求。然而，我们将证明这个概念是不正确的**。

让我们考虑一下前面部分讨论的从HTTP请求中转义HTML字符的示例，让我们看看是否可以使用Spring MVC拦截器来实现：

```java
public class EscapeHtmlRequestInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HtmlEscapeRequestWrapper htmlEscapeRequestWrapper = new HtmlEscapeRequestWrapper(request);
        return HandlerInterceptor.super.preHandle(htmlEscapeRequestWrapper, response, handler);
    }
}
```

所有拦截器都必须实现HandleInterceptor接口，**在拦截器中，在将请求转发到目标控制器之前会调用preHandle()方法**。因此，我们将HttpServletRequest对象包装在EscapeHtmlRequestWrapper中，它负责转义HTML字符。

此外，我们还必须将拦截器注册到适当的URL模式：

```java
@Configuration
@EnableWebMvc
public class WebMvcConfiguration implements WebMvcConfigurer {
    private static final Logger logger = LoggerFactory.getLogger(WebMvcConfiguration.class);
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        logger.info("addInterceptors() called");
        registry.addInterceptor(new HtmlEscapeRequestInterceptor()).addPathPatterns("/**");

        WebMvcConfigurer.super.addInterceptors(registry);
    }
}
```

我们可以看到，WebMvcConfiguration类实现了WebMvcConfigurer。在类中，我们重写了方法addInterceptors()。在该方法中，我们使用方法addPathPatterns()为所有传入的HTTP请求注册了拦截器EscapeHtmlRequestInterceptor。

令人惊讶的是，**HtmlEscapeRequestInterceptor无法转发修改后的请求正文并调用处理程序/save**：

```java
@Test
void givenInterceptor_whenEscapeHtmlInterceptor_thenEscapeHtml() throws Exception {
    Map<String, String> requestBody = Map.of(
            "name", "James Cameron",
            "email", "<script>alert()</script>james@gmail.com"
    );

    ObjectMapper objectMapper = new ObjectMapper();
    mockMvc.perform(MockMvcRequestBuilders.post(URI.create("/save"))
                    .contentType(MediaType.APPLICATION_JSON)
                    .content(objectMapper.writeValueAsString(requestBody)))
            .andExpect(MockMvcResultMatchers.status().is4xxClientError());
}
```

我们在HTTP请求正文中推送了一些JavaScript字符，意外的是，请求失败并显示HTTP错误码400。因此，**虽然拦截器可以像过滤器一样工作，但它们不适合修改HTTP请求。相反，当我们需要修改Spring应用程序上下文中的对象时，它们非常有用**。

## 5. 总结

在本文中，我们讨论了在Spring Boot应用程序中的HTTP请求正文到达控制器之前修改HTTP请求正文的各种方法。根据普遍的看法，拦截器可以帮助做到这一点，但我们看到它失败了。然而，我们看到了过滤器和AOP如何在HTTP请求正文到达控制器之前成功修改它。