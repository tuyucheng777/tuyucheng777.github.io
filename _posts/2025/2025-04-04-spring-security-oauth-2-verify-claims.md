---
layout: post
title:  Spring Security OAuth2中的新功能 – 验证声明
category: spring-security
copyright: spring-security
excerpt: Spring Security OAuth2
---

## 1. 概述

在本快速教程中，我们将使用Spring Security OAuth2实现，并学习如何使用[Spring Security OAuth 2.2.0.RELEASE](https://spring.io/blog/2017/07/28/spring-security-oauth-2-2-released)中引入的新JwtClaimsSetVerifier验证JWT声明。

## 2. Maven配置

首先，我们需要将最新版本的spring-security-oauth2添加到pom.xml中：

```xml
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
```

## 3. TokenStore配置

接下来，我们在资源服务器中配置我们的TokenStore：

```java
@Bean
public TokenStore tokenStore() {
    return new JwtTokenStore(accessTokenConverter());
}

@Bean
public JwtAccessTokenConverter accessTokenConverter() {
    JwtAccessTokenConverter converter = new JwtAccessTokenConverter();
    converter.setSigningKey("123");
    converter.setJwtClaimsSetVerifier(jwtClaimsSetVerifier());
    return converter;
}
```

请注意我们如何将新的验证器添加到我们的JwtAccessTokenConverter。

有关如何配置JwtTokenStore的更多详细信息，请查看有关[使用JWT和Spring Security OAuth](https://www.baeldung.com/spring-security-oauth-jwt)的文章。

现在，在下面的章节中，我们将讨论不同类型的声明验证器以及如何使它们协同工作。

## 4. IssuerClaimVerifier

我们从简单的开始-使用IssuerClaimVerifier验证颁发者的“iss”声明，如下所示：

```java
@Bean
public JwtClaimsSetVerifier issuerClaimVerifier() {
    try {
        return new IssuerClaimVerifier(new URL("http://localhost:8081"));
    } catch (MalformedURLException e) {
        throw new RuntimeException(e);
    }
}
```

在此示例中，我们添加了一个简单的IssuerClaimVerifier来验证我们的颁发者。如果JWT令牌包含颁发者“iss”声明的不同值，则会抛出一个简单的InvalidTokenException。

当然，如果令牌确实包含颁发者“iss”声明，则不会引发异常并且该令牌被视为有效。

## 5. 自定义声明验证器

但是，有趣的是，我们还可以构建自定义声明验证器：

```java
@Bean
public JwtClaimsSetVerifier customJwtClaimVerifier() {
    return new CustomClaimVerifier();
}
```

以下是一个简单的实现-检查user_name声明是否存在于我们的JWT令牌中：

```java
public class CustomClaimVerifier implements JwtClaimsSetVerifier {
    @Override
    public void verify(Map<String, Object> claims) throws InvalidTokenException {
        String username = (String) claims.get("user_name");
        if ((username == null) || (username.length() == 0)) {
            throw new InvalidTokenException("user_name claim is empty");
        }
    }
}
```

请注意，我们在这里只是简单地实现JwtClaimsSetVerifier接口，然后为verify方法提供完全自定义的实现-这为我们所需的任何类型的检查提供了充分的灵活性。

## 6. 结合多个声明验证器

最后，让我们看看如何使用DelegatingJwtClaimsSetVerifier组合多个声明验证器-如下所示：

```java
@Bean
public JwtClaimsSetVerifier jwtClaimsSetVerifier() {
    return new DelegatingJwtClaimsSetVerifier(Arrays.asList(issuerClaimVerifier(), customJwtClaimVerifier()));
}
```

DelegatingJwtClaimsSetVerifier获取JwtClaimsSetVerifier对象列表并将声明验证过程委托给这些验证器。

## 7. 简单集成测试

现在我们已经完成了实现，让我们用一个简单的[集成测试](https://github.com/Baeldung/spring-security-oauth/blob/master/oauth-legacy/oauth-resource-server-legacy-2/src/test/java/com/baeldung/test/JwtClaimsVerifierIntegrationTest.java)来测试我们的声明验证器：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(
        classes = ResourceServerApplication.class,
        webEnvironment = WebEnvironment.RANDOM_PORT)
public class JwtClaimsVerifierIntegrationTest {

    @Autowired
    private JwtTokenStore tokenStore;

    // ...
}
```

我们将从不包含颁发者(但包含user_name)的令牌开始-它应该是有效的：

```java
@Test
public void whenTokenDontContainIssuer_thenSuccess() {
    String tokenValue = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

这样做的原因很简单-只有当令牌中存在颁发者声明时，第一个验证器才会激活。如果该声明不存在，验证器就不会启动。

接下来，让我们看一下包含有效颁发者(http://localhost:8081)和user_name的令牌，这也应该是有效的：

```java
@Test
public void whenTokenContainValidIssuer_thenSuccess() {
    String tokenValue = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

当令牌包含无效的颁发者(http://localhost:8082)时，它将被验证并确定为无效：

```java
@Test(expected = InvalidTokenException.class)
public void whenTokenContainInvalidIssuer_thenException() {
    String tokenValue = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

接下来，当令牌不包含user_name声明时，它将无效：

```java
@Test(expected = InvalidTokenException.class)
public void whenTokenDontContainUsername_thenException() {
    String tokenValue = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

最后，当令牌包含空的user_name声明时，它也是无效的：

```java
@Test(expected = InvalidTokenException.class)
public void whenTokenContainEmptyUsername_thenException() {
    String tokenValue = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9....";
    OAuth2Authentication auth = tokenStore.readAuthentication(tokenValue);

    assertTrue(auth.isAuthenticated());
}
```

## 8. 总结

在这篇简短的文章中，我们了解了Spring Security OAuth中的新验证器功能。