---
layout: post
title:  在JUnit测试中使用JwtDecoder Mock JWT
category: mock
copyright: mock
excerpt: JWT Mock
---

## 1. 概述

在本教程中，我们将探讨如何有效地Mock [JWT](https://datatracker.ietf.org/doc/html/rfc7519#section-3)(JSON Web Token)以对使用JWT身份验证的Spring Security应用程序进行单元测试。**测试受JWT保护的端点通常需要Mock不同的JWT场景，而不依赖于实际的令牌生成或验证**。这种方法使我们能够编写强大的单元测试，而无需在测试期间管理真实的JWT令牌。

**Mock JWT解码在单元测试中非常重要，因为它允许我们将身份验证逻辑与外部依赖项(例如令牌生成服务或第三方身份提供者)隔离开来**。通过Mock不同的JWT场景，我们可以确保我们的应用程序正确处理有效令牌、自定义声明、无效令牌和过期令牌。

我们将学习如何使用[Mockito](https://www.baeldung.com/mockito-annotations) Mock JwtDecoder、创建自定义JWT声明以及测试各种场景。在本教程结束时，我们将能够为基于Spring Security JWT的身份验证逻辑编写全面的单元测试。

## 2. 设置和配置

在开始编写测试之前，让我们先设置具有必要依赖的测试环境。

### 2.1 依赖

我们将使用Spring Security OAuth2、Mockito和JUnit 5进行测试：

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-jose</artifactId>
    <version>6.4.2</version>
</dependency>
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>5.15.2</version>
    <scope>test</scope>
</dependency>
```

[spring-security-oauth2-jose](https://mvnrepository.com/artifact/org.springframework.security/spring-security-oauth2-jose)依赖支持Spring Security中的JWT，包括用于解码和验证JWT的JwtDecoder接口。[mockito-core](https://mvnrepository.com/artifact/org.mockito/mockito-core/latest)依赖允许我们在测试中Mock依赖，确保我们可以将被测单元UserController与外部系统隔离。

### 2.2 创建UserController

接下来，我们将使用@GetMapping(“/user”)端点创建UserController，以根据JWT令牌检索用户信息。它验证令牌、检查过期时间并提取用户的主题：

```java
@GetMapping("/user")
public ResponseEntity<String> getUserInfo(@AuthenticationPrincipal Jwt jwt) {
    if (jwt == null || jwt.getSubject() == null) {
        throw new JwtValidationException("Invalid token", Arrays.asList(new OAuth2Error("invalid_token")));
    }

    Instant expiration = jwt.getExpiresAt();
    if (expiration != null && expiration.isBefore(Instant.now())) {
        throw new JwtValidationException("Token has expired", Arrays.asList(new OAuth2Error("expired_token")));
    }

    return ResponseEntity.ok("Hello, " + jwt.getSubject());
}
```

### 2.3 设置测试类

让我们创建一个测试类MockJwtDecoderJUnitTest并使用Mockito来Mock JwtDecoder，这是初始设置：

```java
@ExtendWith(MockitoExtension.class)
public class MockJwtDecoderJUnitTest {
    @Mock
    private JwtDecoder jwtDecoder;

    @InjectMocks
    private UserController userController;

    @BeforeEach
    void setUp() {
        SecurityContextHolder.clearContext();
    }
}
```

在此设置中，我们使用@ExtendWith(MockitoExtension.class)在JUnit测试中启用Mockito。使用@Mock Mock JwtDecoder，并使用@InjectMocks将Mock的JwtDecoder注入UserController。每次测试之前都会清除SecurityContextHolder，以确保状态干净。

## 3. Mock JWT解码

设置好环境后，我们编写测试来Mock JWT解码，我们首先测试一个有效的JWT令牌。

### 3.1 测试有效令牌

当提供有效令牌时，应用程序应返回用户信息。以下是我们对这一场景的测试方法：

```java
@Test
void whenValidToken_thenReturnsUserInfo() {
    Map<String, Object> claims = new HashMap<>();
    claims.put("sub", "john.doe");

    Jwt jwt = Jwt.withTokenValue("token")
            .header("alg", "none")
            .claims(existingClaims -> existingClaims.putAll(claims))
            .build();

    JwtAuthenticationToken authentication = new JwtAuthenticationToken(jwt);
    SecurityContextHolder.getContext().setAuthentication(authentication);

    ResponseEntity<String> response = userController.getUserInfo(jwt);

    assertEquals("Hello, john.doe", response.getBody());
    assertEquals(HttpStatus.OK, response.getStatusCode());
}
```

在此测试中，我们创建了一个带有sub(主题)声明的Mock JWT，JwtAuthenticationToken用于设置安全上下文，UserController处理该令牌并返回响应。我们使用断言来验证响应。

### 3.2 测试自定义声明

有时，JWT包含自定义声明，例如角色或电子邮件地址。例如，如果UserController使用roles声明来授权访问，则测试应检查控制器是否根据声明的角色按预期运行：

```java
@Test
void whenTokenHasCustomClaims_thenProcessesCorrectly() {
    Map<String, Object> claims = new HashMap<>();
    claims.put("sub", "john.doe");
    claims.put("roles", Arrays.asList("ROLE_USER", "ROLE_ADMIN"));
    claims.put("email", "john.doe@example.com");

    Jwt jwt = Jwt.withTokenValue("token")
            .header("alg", "none")
            .claims(existingClaims -> existingClaims.putAll(claims))
            .build();

    List authorities = ((List) jwt.getClaim("roles"))
            .stream()
            .map(role -> new SimpleGrantedAuthority(role))
            .collect(Collectors.toList());

    JwtAuthenticationToken authentication = new JwtAuthenticationToken(
            jwt,
            authorities,
            jwt.getClaim("sub")
    );

    SecurityContextHolder.getContext().setAuthentication(authentication);

    ResponseEntity response = userController.getUserInfo(jwt);

    assertEquals("Hello, john.doe", response.getBody());
    assertEquals(HttpStatus.OK, response.getStatusCode());

    assertTrue(authentication.getAuthorities().stream()
            .anyMatch(auth -> auth.getAuthority().equals("ROLE_ADMIN")));
}
```

在此测试中，我们验证roles声明是否正确处理以及用户是否具有预期的权限(在本例中为ROLE_ADMIN)。

## 4. 测试其他场景

接下来，我们探讨测试不同的情况。

### 4.1 测试无效令牌

当提供无效令牌时，应用程序应抛出JwtValidationException。让我们编写一个快速测试来验证JwtDecoder在尝试解码无效令牌时是否正确抛出异常：

```java
@Test
void whenInvalidToken_thenThrowsException() {
    Map<String, Object> claims = new HashMap<>();
    claims.put("sub", null);

    Jwt invalidJwt = Jwt.withTokenValue("invalid_token")
            .header("alg", "none")
            .claims(existingClaims -> existingClaims.putAll(claims))
            .build();

    JwtAuthenticationToken authentication = new JwtAuthenticationToken(invalidJwt);
    SecurityContextHolder.getContext()
            .setAuthentication(authentication);

    JwtValidationException exception = assertThrows(JwtValidationException.class, () -> {
        userController.getUserInfo(invalidJwt);
    });

    assertEquals("Invalid token", exception.getMessage());
}
```

在这个测试中，我们Mock JwtDecoder在处理空令牌时抛出JwtValidationException。

测试断言抛出了JwtValidationException异常，并显示消息“Invalid token”。

### 4.2 测试过期的令牌

当提供过期的令牌时，应用程序应抛出JwtValidationException。以下测试验证JwtDecoder在尝试解码过期令牌时是否正确抛出异常：

```java
@Test
void whenExpiredToken_thenThrowsException() throws Exception {
    Map<String, Object> claims = new HashMap<>();
    claims.put("sub", "john.doe");
    claims.put("exp", Instant.now().minus(1, ChronoUnit.DAYS));

    Jwt expiredJwt = Jwt.withTokenValue("expired_token")
            .header("alg", "none")
            .claims(existingClaims -> existingClaims.putAll(claims))
            .build();

    JwtAuthenticationToken authentication = new JwtAuthenticationToken(expiredJwt);
    SecurityContextHolder.getContext()
            .setAuthentication(authentication);
    JwtValidationException exception = assertThrows(JwtValidationException.class, () -> {
        userController.getUserInfo(expiredJwt);
    });

    assertEquals("Token has expired", exception.getMessage());
}
```

在这个测试中，我们将过期时间设置为1天前，以Mock过期的令牌。

测试断言抛出了JwtValidationException，并显示消息“Token has expired”。

## 5. 总结

在本教程中，我们学习了如何使用Mockito在JUnit测试中Mock JWT解码。我们介绍了各种场景，包括使用自定义声明测试有效令牌、处理无效令牌和管理过期令牌。

通过Mock JWT解码，我们可以为Spring Security应用程序编写单元测试，而无需依赖外部令牌生成或验证服务。这种方法确保我们的测试快速、可靠且独立于外部依赖项。