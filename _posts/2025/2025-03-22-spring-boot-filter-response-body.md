---
layout: post
title:  在Spring Boot Filter中获取响应主体
category: spring-boot
copyright: spring-boot
excerpt: Spring Boot
---

## 1. 简介

在本文中，我们将探讨如何从Spring Boot过滤器中的ServletResponse中检索响应主体。

本质上，我们将定义问题，然后使用缓存响应主体的解决方案，使其在Spring Boot过滤器中可用。

## 2. 理解问题

首先，让我们了解我们要解决的问题。

使用Spring Boot过滤器时，从ServletResponse访问响应主体比较棘手。**这是因为响应主体不易获得，因为它是在过滤器链完成执行后写入输出流的**。

但是有些操作，比如生成哈希签名，需要先获取响应体的内容，再发送给客户端，所以我们需要想办法读取响应体的内容。

## 3. 在过滤器中使用ContentCachingResponseWrapper

为了解决前面定义的问题，我们将创建一个自定义过滤器并使用Spring框架提供的ContentCachingResponseWrapper类：

```java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
    ContentCachingResponseWrapper responseCacheWrapperObject = new ContentCachingResponseWrapper((HttpServletResponse) servletResponse);
    filterChain.doFilter(servletRequest, responseCacheWrapperObject);
    byte[] responseBody = responseCacheWrapperObject.getContentAsByteArray();
    MessageDigest md5Digest = MessageDigest.getInstance("MD5");
    byte[] md5Hash = md5Digest.digest(responseBody);
    String md5HashString = DatatypeConverter.printHexBinary(md5Hash);
    responseCacheWrapperObject.getResponse().setHeader("Response-Body-MD5", md5HashString);
    // ...
}
```

简而言之，包装类允许我们包装HttpServletResponse来缓存响应主体内容并调用doFilter()将请求传递给下一个过滤器。

请记住，我们一定不能忘记此处的doFilter()调用。否则，传入的请求将不会进入Spring过滤器链中的下一个过滤器，应用程序将无法按我们预期的方式处理该请求。事实上，**不调用doFilter()违反了Servlet规范**。

此外，我们一定不要忘记使用responseCacheWrapperObject调用doFilter()。否则，响应主体将不会被缓存。简而言之，ContentCachingResponseWrapper将过滤器放在响应输出流和发出HTTP请求的客户端之间。因此，在创建响应主体输出流时(在本例中是在doFilter()调用之后)，内容可以在过滤器内部进行处理。

使用包装器后，可以使用getContentAsByteArray()方法在过滤器中获取响应主体，我们使用此方法计算MD5哈希值。

首先，我们使用[MessageDigest](https://www.baeldung.com/java-md5#md5-using-messagedigest-class)类创建响应主体的MD5哈希值。其次，我们将字节数组转换为十六进制字符串。第三，我们使用setHeader()方法将生成的哈希字符串设置为响应对象的标头。

如果需要，我们可以[将字节数组转换为字符串](https://www.baeldung.com/java-string-to-byte-array#decoding)，并使主体的内容更加明确。

最后，在退出doFilter()方法之前调用copyBodyToResponse()将更新后的响应主体复制回原始响应，这一点至关重要：

```java
responseCacheWrapperObject.copyBodyToResponse();
```

**在退出doFilter()方法之前调用copyBodyToResponse()至关重要。否则，客户端将无法收到完整的响应**。

## 4. 配置过滤器

现在，我们需要在Spring Boot中[添加过滤器](https://www.baeldung.com/spring-boot-add-filter)：

```java
@Bean
public FilterRegistrationBean loggingFilter() {
    FilterRegistrationBean registrationBean = new FilterRegistrationBean<>();
    registrationBean.setFilter(new MD5Filter());
    return registrationBean;
}
```

在这里，我们配置创建一个FilterRegistrationBean，并使用我们之前创建的过滤器的实现。

## 5. 测试MD5

最后，我们可以使用[Spring中的集成测试](https://www.baeldung.com/integration-testing-in-spring)来测试一切是否按预期工作：

```java
@Test
void whenExampleApiCallThenResponseHasMd5Header() throws Exception {
    String endpoint = "/api/example";
    String expectedResponse = "Hello, World!";
    String expectedMD5 = getMD5Hash(expectedResponse);

    MvcResult mvcResult = mockMvc.perform(get(endpoint).accept(MediaType.TEXT_PLAIN_VALUE))
            .andExpect(status().isOk())
            .andReturn();

    String md5Header = mvcResult.getResponse()
            .getHeader("Response-Body-MD5");
    assertThat(md5Header).isEqualTo(expectedMD5);
}
```

在这里，我们调用/api/example控制器，该控制器在正文中返回“Hello, World!”文本。我们定义了getMD5Hash()方法，该方法将响应转换为类似于我们在过滤器中使用的MD5：

```java
private String getMD5Hash(String input) throws NoSuchAlgorithmException {
    MessageDigest md5Digest = MessageDigest.getInstance("MD5");
    byte[] md5Hash = md5Digest.digest(input.getBytes(StandardCharsets.UTF_8));
    return DatatypeConverter.printHexBinary(md5Hash);
}
```

## 6. 总结

在本文中，我们学习了如何使用ContentCachingResponseWrapper类从Spring Boot过滤器中的ServletResponse中检索响应主体，我们使用此机制展示了如何在HTTP响应标头中实现主体的MD5编码。