---
layout: post
title:  在Spring Boot中测试CORS
category: spring-test
copyright: spring-test
excerpt: Spring Test
---

## 1. 概述

[跨域资源共享(CORS)](https://www.baeldung.com/cs/cors-preflight-requests)是一种安全机制，允许一个来源的网页访问另一个来源的资源。浏览器强制实施该机制，以防止网站向不同的域发出未经授权的请求。

**在使用Spring Boot构建Web应用程序时，正确测试我们的CORS配置非常重要，以确保我们的应用程序可以安全地与授权来源交互，同时阻止未经授权的来源**。

通常情况下，我们只有在部署应用程序后才会发现CORS问题。**通过尽早测试CORS配置，我们可以在开发过程中发现并修复这些问题，从而节省时间和精力**。

在本教程中，我们将探讨如何使用[MockMvc](https://www.baeldung.com/integration-testing-in-spring)编写有效的测试来验证我们的CORS配置。

## 2. 在Spring Boot中配置CORS

在Spring Boot应用程序中[配置CORS的方法](https://www.baeldung.com/spring-cors)有很多种。在本教程中，我们将使用[Spring Security](https://www.baeldung.com/security-spring)并定义一个CorsConfigurationSource：

```java
private CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration corsConfiguration = new CorsConfiguration();
    corsConfiguration.setAllowedOrigins(List.of("https://tuyucheng.com"));
    corsConfiguration.setAllowedMethods(List.of("GET"));
    corsConfiguration.setAllowedHeaders(List.of("X-Tuyucheng-Key"));
    corsConfiguration.setExposedHeaders(List.of("X-Rate-Limit-Remaining"));

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", corsConfiguration);
    return source;
}
```

**在我们的配置中，我们允许来自https://tuyucheng.com来源的请求，使用GET方法、X-Tuyucheng-Key标头，并在响应中公开X-Rate-Limit-Remaining标头**。

我们已经在配置中对值进行了硬编码，但我们可以使用[@ConfigurationProperties](https://www.baeldung.com/configuration-properties-in-spring-boot)将它们外部化。

接下来，让我们配置SecurityFilterChain Bean来应用我们的CORS配置：

```java
private static final String[] WHITELISTED_API_ENDPOINTS = { "/api/v1/joke" };

@Bean
public SecurityFilterChain configure(HttpSecurity http) {
    http.cors(corsConfigurer -> corsConfigurer.configurationSource(corsConfigurationSource()))
            .authorizeHttpRequests(authManager -> authManager
                    .requestMatchers(WHITELISTED_API_ENDPOINTS)
                    .permitAll()
                    .anyRequest()
                    .authenticated());
    return http.build();
}
```

在这里，我们使用之前定义的corsConfigurationSource()方法配置CORS。

**我们还将/api/v1/joke端点列入白名单，因此无需身份验证即可访问。我们将使用此API端点作为基础来测试我们的CORS配置**：

```java
private static final Faker FAKER = new Faker();

@GetMapping(value = "/api/v1/joke")
public ResponseEntity<JokeResponse> generate() {
    String joke = FAKER.joke().pun();
    String remainingLimit = FAKER.number().digit();

    return ResponseEntity.ok()
            .header("X-Rate-Limit-Remaining", remainingLimit)
            .body(new JokeResponse(joke));
}

record JokeResponse(String joke) {};
```

我们使用[Datafaker](https://www.baeldung.com/java-datafaker)生成一个随机笑joke和一个保留的[速率限制](https://www.baeldung.com/spring-bucket4j)值。然后我们在响应主体中返回joke，并在X-Rate-Limit-Remaining标头中包含生成的值。

## 3. 使用MockMvc测试CORS

现在我们已经在应用程序中配置了CORS，让我们编写一些测试来确保它按预期工作。**我们将使用MockMvc向我们的API端点发送请求并验证响应**。

### 3.1 测试允许的来源

**首先，让我们测试来自我们允许的来源的请求是否成功**：

```java
mockMvc.perform(get("/api/v1/joke")
    .header("Origin", "https://tuyucheng.com"))
    .andExpect(status().isOk())
    .andExpect(header().string("Access-Control-Allow-Origin", "https://tuyucheng.com"));
```

我们还验证响应是否包含来自允许来源的请求的Access-Control-Allow-Origin标头。

接下来，让我们验证来自非允许来源的请求是否被阻止：

```java
mockMvc.perform(get("/api/v1/joke")
    .header("Origin", "https://non-tuyucheng.com"))
    .andExpect(status().isForbidden())
    .andExpect(header().doesNotExist("Access-Control-Allow-Origin"));
```

### 3.2 测试允许的方法

**为了测试允许的方法，我们将使用HTTP OPTIONS方法模拟预检请求**：

```java
mockMvc.perform(options("/api/v1/joke")
    .header("Origin", "https://tuyucheng.com")
    .header("Access-Control-Request-Method", "GET"))
    .andExpect(status().isOk())
    .andExpect(header().string("Access-Control-Allow-Methods", "GET"));
```

我们验证请求是否成功，并且响应中是否存在Access-Control-Allow-Methods标头。

类似地，让我们确保不允许的方法被拒绝：

```java
mockMvc.perform(options("/api/v1/joke")
    .header("Origin", "https://tuyucheng.com")
    .header("Access-Control-Request-Method", "POST"))
    .andExpect(status().isForbidden());
```

### 3.3 测试允许的标头

**现在，我们将通过发送带有Access-Control-Request-Headers标头的预检请求并验证响应中的Access-Control-Allow-Headers来测试允许的标头**：

```java
mockMvc.perform(options("/api/v1/joke")
    .header("Origin", "https://tuyucheng.com")
    .header("Access-Control-Request-Method", "GET")
    .header("Access-Control-Request-Headers", "X-Baeldung-Key"))
    .andExpect(status().isOk())
    .andExpect(header().string("Access-Control-Allow-Headers", "X-Tuyucheng-Key"));
```

让我们验证一下我们的应用程序是否拒绝不允许的标头：

```java
mockMvc.perform(options("/api/v1/joke")
    .header("Origin", "https://tuyucheng.com")
    .header("Access-Control-Request-Method", "GET")
    .header("Access-Control-Request-Headers", "X-Non-Tuyucheng-Key"))
    .andExpect(status().isForbidden());
```

### 3.4 测试暴露的标头

**最后，让我们测试一下公开的标头是否正确包含在允许来源的响应中**：

```java
mockMvc.perform(get("/api/v1/joke")
    .header("Origin", "https://tuyucheng.com"))
    .andExpect(status().isOk())
    .andExpect(header().string("Access-Control-Expose-Headers", "X-Rate-Limit-Remaining"))
    .andExpect(header().exists("X-Rate-Limit-Remaining"));
```

我们验证响应中是否存在Access-Control-Expose-Headers标头，并包含我们公开的标头X-Rate-Limit-Remaining。我们还检查实际的X-Rate-Limit-Remaining标头是否存在。

类似地，让我们确保我们公开的标头不包含在非允许来源的响应中：

```java
mockMvc.perform(get("/api/v1/joke")
    .header("Origin", "https://non-tuyucheng.com"))
    .andExpect(status().isForbidden())
    .andExpect(header().doesNotExist("Access-Control-Expose-Headers"))
    .andExpect(header().doesNotExist("X-Rate-Limit-Remaining"));
```

## 4. 总结

在本文中，**我们讨论了如何使用MockMvc编写有效的测试来验证我们的CORS配置是否正确允许来自授权来源、方法和标头的请求，同时阻止未经授权的请求**。

通过彻底测试我们的CORS配置，我们可以尽早发现错误配置并防止生产中出现意外的CORS错误。