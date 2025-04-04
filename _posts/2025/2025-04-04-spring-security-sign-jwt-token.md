---
layout: post
title:  创建用于签名JWT令牌的Spring Security密钥
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

[JSON Web Tokens(JWT)](https://www.baeldung.com/java-json-web-tokens-jjwt)是保护无状态应用程序的事实标准，Spring Security框架提供了集成JWT以保护REST API的方法，**生成Token的关键过程之一是应用签名来保证真实性**。

在本教程中，我们将探索一个使用[JWT身份验证](https://www.baeldung.com/spring-security-oauth-jwt)的无状态Spring Boot应用程序，我们将设置必要的组件并创建一个加密SecretKey实例来签署和验证JWT。

## 2. 项目设置

首先，让我们使用Spring Security和JWT令牌启动一个无状态Spring Boot应用程序。**值得注意的是，为了简单起见，我们不会显示完整的设置代码**。

### 2.1 Maven依赖

首先，让我们将[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)、[spring-boot-starter-security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)、[spring-boot-starter-data-jpa](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa)和[h2数据库](https://mvnrepository.com/artifact/com.h2database/h2)依赖添加到pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.3.3</version> 
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId> 
    <version>3.3.3</version> 
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>3.3.3</version>  
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.224</version>
</dependency>
```

Spring Boot Starter Web提供API来构建REST API。此外，Spring Boot Starter Security依赖有助于提供身份验证和授权，我们添加了内存数据库以进行快速原型设计。

接下来，让我们将[jjwt-api](https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-api)，[jjwt-impl](https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-impl)和[jjwt-jackson](https://mvnrepository.com/artifact/io.jsonwebtoken/jjwt-jackson)依赖添加到pom.xml：

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.5</version>
</dependency>
```

这些依赖提供了一个API来生成和签署JWT并将其集成到Spring Security中。

### 2.2 JWT配置

首先，让我们创建一个身份验证入口点：

```java
@Component
class AuthEntryPointJwt implements AuthenticationEntryPoint {
    // ...
}
```

在这里，我们创建一个类来处理使用JWT身份验证的Spring Security应用程序中的授权访问尝试。它充当守门人，确保只有具有有效访问权限的用户才能访问受保护的资源。

然后，让我们创建一个名为AuthTokenFilter的类，它拦截传入的请求，验证JWT令牌，并在存在有效令牌时对用户进行身份验证：

```java
class AuthTokenFilter extends OncePerRequestFilter {
    // ...
}
```

最后，让我们创建JwtUtil类，它提供创建和验证令牌的方法：

```java
@Component
class JwtUtils {
    // ... 
}
```

此类包含使用signWith()方法的逻辑。

### 2.3 安全配置

最后我们来定义SecurityConfiguration类并集成JWT：

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
class SecurityConfiguration {
    // ...

    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http.csrf(AbstractHttpConfigurer::disable)
                .cors(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(req -> req.requestMatchers(WHITE_LIST_URL)
                        .permitAll()
                        .anyRequest()
                        .authenticated())
                .exceptionHandling(ex -> ex.authenticationEntryPoint(unauthorizedHandler))
                .sessionManagement(session -> session.sessionCreationPolicy(STATELESS))
                .authenticationProvider(authenticationProvider())
                .addFilterBefore(
                        authenticationJwtTokenFilter(),
                        UsernamePasswordAuthenticationFilter.class
                );

        return http.build();
    }

    // ...
}
```

在上面的代码中，我们集成了JWT入口点和过滤器来激活JWT身份验证。

## 3. signWith()方法

**JJWT库提供了signWith()方法来帮助使用特定的加密算法和密钥对JWT进行签名**，此签名过程对于确保JWT的完整性和真实性至关重要。

signWith()方法接收Key或SecretKey实例和签名算法作为参数，**[基于哈希的消息认证码(HMAC)算法](https://www.baeldung.com/java-hmac)是最常用的签名算法之一**。

重要的是，该方法需要一个密钥(通常是一个字节数组)用于签名过程，我们可以使用Key或SecretKey实例将密钥字符串转换为密钥。

**值得注意的是，我们可以传递一个普通字符串作为密钥。但是，这缺乏加密Key或SecretKey实例的安全性保证和随机性**。

使用SecretKey实例保证JWT的完整性和真实性。

## 4. 签署JWT

我们可以使用Key和SecretKey实例创建一个强密钥来签署JWT。

### 4.1 使用Key实例

本质上，我们可以将密钥字符串转换为Key实例，然后进一步加密，然后再使用它来签署JWT。

首先，让我们确保密钥字符串是[Base64编码](https://www.baeldung.com/java-base64-encode-and-decode)的：

```java
private String jwtSecret = "4261656C64756E67";
```

接下来让我们创建一个Key对象：

```java
private Key getSigningKey() {
    byte[] keyBytes = Decoders.BASE64.decode(this.jwtSecret);
    return Keys.hmacShaKeyFor(keyBytes);
}
```

在上面的代码中，我们将jwtSecret解码为[字节数组](https://www.baeldung.com/java-string-to-byte-array)。接下来，我们调用hmacShaKeyFor()，它接收keyBytes作为Keys实例的参数，这将根据HMAC算法生成密钥。

如果密钥不是Base64编码的，我们可以对纯字符串调用getByte()方法：

```java
private Key getSigningKey() {
    byte[] keyBytes = this.jwtSecret.getBytes(StandardCharsets.UTF_8);
    return Keys.hmacShaKeyFor(keyBytes);
}
```

但是，我们不建议这样做，因为密钥可能格式不正确，并且字符串可能包含非UTF-8字符。因此，我们必须确保密钥字符串是Base64编码的，然后才能从中生成密钥。

### 4.2 使用SecretKey实例

另外，我们可以使用HMAC-SHA算法形成强密钥来创建SecretKey实例。让我们创建一个返回密钥的SecretKey实例：

```java
SecretKey getSigningKey() {
    return Jwts.SIG.HS256.key().build();
}
```

这里我们直接使用HMAC-SHA算法，而不使用字节数组，这样就形成了一个强签名密钥。接下来，我们可以通过将getSigningKey()作为参数传递来更新signWith()方法。

或者，我们可以从Base16编码的字符串创建一个[SecretKey](https://www.baeldung.com/java-secret-key-to-string#2-string-to-secretkey)实例：

```java
SecretKey getSigningKey() {
    byte[] keyBytes = Decoders.BASE64.decode(jwtSecret);
    return Keys.hmacShaKeyFor(keyBytes);
}
```

这将生成一个强SecretKey类型来签署和验证JWT。

**值得注意的是，建议使用SecretKey实例而不是Key实例，因为用于验证令牌的新方法verifyWith()接收SecretKey类型作为参数**。

### 4.3 应用密钥

现在，让我们使用密钥来签署我们应用程序的JWT：

```java
String generateJwtToken(Authentication authentication) {
    UserDetailsImpl userPrincipal = (UserDetailsImpl) authentication.getPrincipal();

    return Jwts.builder()
        .subject((userPrincipal.getUsername()))
        .issuedAt(new Date())
        .expiration(new Date((new Date()).getTime() + jwtExpirationMs))
        .signWith(key)
        .compact();
}
```

signWith()方法以SecretKey实例作为参数，为令牌附加唯一签名。

## 5. 总结

在本文中，我们学习了如何使用Java Key和SecretKey实例创建密钥。此外，我们还演示了一个无状态的Spring Boot应用程序，它利用JWT令牌来确保令牌完整性，并应用Key或SecretKey实例对其进行签名和验证。