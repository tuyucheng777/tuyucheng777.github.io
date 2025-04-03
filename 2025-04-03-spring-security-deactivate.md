---
layout: post
title:  禁用Spring Security指南
category: spring-security
copyright: spring-security
excerpt: Spring Data JPA
---

## 1. 概述

[Spring Security](https://www.baeldung.com/security-spring)是一个功能强大、高度可定制的[Java](https://www.baeldung.com/get-started-with-java-series)应用程序身份验证和访问控制框架。我们将概述Spring Security的用途以及可能需要禁用它的一些常见场景，例如在开发、测试期间或使用自定义安全机制时。

本文将指导我们完成在[Spring Boot](https://www.baeldung.com/spring-boot)应用程序中禁用Spring Security的步骤，同时确保配置易于管理和恢复。我们还将在此处提供可在项目中参考的代码示例，并附上单元测试以演示其行为。在本教程中，我们将讨论理解如何在Spring Boot应用程序中禁用Spring Security所需的基本概念。

## 2. 禁用Spring Security

根据我们应用程序的具体要求，我们可以通过多种方式禁用Spring Security，我们将探讨四种常用方法：

- 使用自定义安全配置
- 利用Spring Profile
- 删除Spring Security依赖
- 不包括Spring Security自动配置

在讨论禁用Spring Security的不同策略之前，让我们通过[Spring Initializr](https://start.spring.io/)设置一个简单的Spring Boot应用程序，我们将在整个指南中使用它。我们将创建一个基于Maven的最小Spring Boot项目，其中包含一个控制器，它将作为我们的测试端点：

```java
@RestController
@RequestMapping("/api")
public class PublicController {
    @GetMapping("/endpoint")
    public ResponseEntity<String> publicEndpoint() {
        return ResponseEntity.ok("This is a public endpoint.");
    }
}
```

## 3. 使用自定义安全配置

禁用Spring Security最直接的方法之一是创建自定义安全配置类，此方法涉及定义和配置SecurityFilterChain Bean以允许所有请求而无需身份验证：

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth.anyRequest().permitAll())
            .csrf(AbstractHttpConfigurer::disable);
    return http.build();
}
```

## 4. 利用Spring Profile

Spring Profile允许我们为应用程序配置不同的环境，我们可以使用Profile在某些环境(例如开发或测试)中禁用安全性。创建一个新的特定于Profile的属性文件，例如application-dev.properties，并添加以下行：

```properties
server.port=8080
spring.profiles.active=dev
spring.application.name=spring-security-noauth-profile
```

现在，创建名为DevSecurityConfiguration的新Java类，此类专门为Spring Boot应用程序中的“dev” Profile配置，它通过允许所有请求来允许不受限制地访问所有端点。这在开发阶段很有用，可以简化测试和调试而不受安全限制：

```java
@Profile("dev")
public class DevSecurityConfiguration {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth.anyRequest().permitAll());
        return http.build();
    }
}
```

除了上述配置之外，我们还将定义另一个在dev Profile未处于激活状态时适用的安全配置类，此配置启用身份验证并允许对所有端点进行受限访问：

```java
@Profile("!dev")
public class SecurityConfiguration {
    @Bean
    SecurityFilterChain httpSecurity(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
                .formLogin(withDefaults())
                .httpBasic(withDefaults());

        return http.build();
    }
}
```

当我们使用dev Profile在开发环境中运行应用程序时，所有请求都无需身份验证即可被允许。但是，当应用程序使用除dev之外的任何Profile运行时，我们的应用程序要求对任何类型的请求进行身份验证。

## 5. 删除Spring Security依赖

禁用Spring Security的最简单方法是从项目中删除其依赖，通过这样做，我们将删除Spring Security提供的所有与安全相关的配置和默认值：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>...</version>
</dependency>
```

删除此依赖将会从应用程序中删除所有Spring Security功能。

## 6. 排除Spring Security自动配置

当我们在类路径中包含spring-boot-starter-security时，Spring Boot会自动配置安全性。要禁用它，请通过向application.properties添加以下属性来排除自动配置：

```properties
spring.autoconfigure.exclude=org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration
```

如果我们想完全禁用Spring Security，我们应该使用spring.autoconfigure.exclude，而不创建SecurityConfiguration类。手动配置Spring Security类会覆盖application.properties配置，因此当两者一起使用时，application.properties中的排除不起作用。

## 7. 禁用安全性用于测试

我们可以通过启动应用程序并在本地访问端点来验证是否已禁用安全性，启动并运行Spring Boot应用程序，当我们尝试访问REST端点时，我们将看到以下响应：

![](/assets/images/2025/springsecurity/springsecuritydeactivate01.png)

我们可以通过使用MockMvc库编写JUnit测试来以编程方式验证安全性是否被禁用。

### 7.1 自定义安全配置的单元测试

以下是一个示例单元测试，用于验证安全配置是否允许不受限制的访问：

```java
public class CommonSecurityConfigTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenSecurityIsDisabled_thenAllEndpointsAreAccessible() throws Exception {
        mockMvc.perform(get("/api/endpoint"))
                .andExpect(status().isOk());
    }
}
```

### 7.2 应用程序Profile的单元测试

我们编写了两个单独的测试来验证我们的安全措施是否会根据激活的Profile而有所不同，以下是禁用安全性时对dev Profile的单元测试：

```java
@ActiveProfiles("dev")
public class DevProfileSecurityConfigTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenDevProfileIsActive_thenAccessIsAllowed() throws Exception {
        mockMvc.perform(get("/api/endpoint"))
                .andExpect(status().isOk());
    }
}
```

以下是启用安全性时非dev Profile的单元测试：

```java
public class NonDevProfileSecurityConfigTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void whenNonDevProfileIsActive_thenAccessIsDenied() throws Exception {
        mockMvc.perform(get("/api/endpoint"))
                .andExpect(status().isUnauthorized());
    }
}
```

## 8. 总结

在本文中，我们讨论了在Spring Boot应用程序中禁用Spring Security的各种方法，每种方法都适用于不同的场景。无论我们使用自定义安全配置还是利用Spring Profile，方法都应符合我们的开发和部署需求。通过遵循本文概述的步骤，我们可以确保我们的应用程序保持灵活且易于管理，尤其是在开发和测试阶段。

请务必记住在生产环境中重新启用安全性，以维护应用程序的完整性和安全性。