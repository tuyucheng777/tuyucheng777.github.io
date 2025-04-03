---
layout: post
title:  在Spring授权服务器中的JWT访问令牌中添加权限作为自定义Claims
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

**向[JSON Web Token(JWT)](https://www.baeldung.com/java-jwt-token-decode#structure-of-a-jwt)[声明](https://www.baeldung.com/cs/security-claims-based-authentication#tcp-connection)添加自定义访问令牌在许多情况下都至关重要，自定义Claims允许我们在令牌有效负载中包含附加信息**。

在本教程中，在本教程中，我们将学习如何将资源所有者权限添加到[Spring Authorization Server](https://www.baeldung.com/spring-security-oauth-auth-server)中的JWT访问令牌。

## 2. Spring授权服务器

**Spring授权服务器是Spring生态系统中的一个新项目，旨在为Spring应用程序提供授权服务器支持**。它旨在使用熟悉且灵活的Spring编程模型简化实现OAuth 2.0和OpenID Connect(OIDC)授权服务器的过程。

### 2.1 Maven依赖

让我们首先将[spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)、[spring-boot-starter -security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)、[spring-boot-starter-test](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-test)和[spring- security-oauth2-authorization-server](https://mvnrepository.com/artifact/org.springframework.security/spring-security-oauth2-authorization-server)依赖导入pom.xml：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>2.5.4</version>
</dependency>
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-oauth2-authorization-server</artifactId>
    <version>0.2.0</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>2.5.4</version>
</dependency>
```

或者，我们可以将[spring-boot-starter-oauth2-authorization-server](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-oauth2-authorization-server)依赖添加到我们的pom.xml文件：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-authorization-server</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 2.2 项目设置

让我们设置Spring授权服务器来颁发访问令牌，为了简单起见，我们将使用[Spring Security OAuth授权服务器](https://www.baeldung.com/spring-security-oauth-auth-server)应用程序。

假设我们使用的是授权服务器项目可在[GitHub](https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-authorization-server)上获取。

## 3. 向JWT访问令牌添加基本自定义Claims

**在基于Spring Security OAuth2的应用程序中，我们可以通过自定义授权服务器中的令牌创建过程来向JWT访问令牌添加自定义Claims**。这种类型的Claim可用于将附加信息注入JWT，然后资源服务器或身份验证和授权流程中的其他组件可以使用这些信息。

### 3.1 添加基本自定义Claim

**我们可以使用OAuth2TokenCustomizer<JWTEncodingContext\> Bean将自定义Claims添加到访问令牌**，通过使用它，授权服务器颁发的每个访问令牌都将填充自定义Claims。

让我们在DefaultSecurityConfig类中添加OAuth2TokenCustomizer Bean：

```java
@Bean
@Profile("basic-claim")
public OAuth2TokenCustomizer<JwtEncodingContext> jwtTokenCustomizer() {
    return (context) -> {
        if (OAuth2TokenType.ACCESS_TOKEN.equals(context.getTokenType())) {
            context.getClaims().claims((claims) -> {
                claims.put("claim-1", "value-1");
                claims.put("claim-2", "value-2");
            });
        }
    };
}
```

OAuth2TokenCustomizer接口是Spring Security OAuth2库的一部分，用于自定义OAuth 2.0令牌。在这种情况下，它在编码过程中专门定制JWT令牌。

传递给jwtTokenCustomizer() Bean的Lambda表达式定义自定义逻辑，context参数表示令牌编码过程中的JwtEncodingContext。

首先，我们使用context.getTokenType()方法检查正在处理的令牌是否是访问令牌。然后，我们使用context.getClaims()方法获取与正在构建的JWT关联的Claims。最后，我们向JWT添加自定义Claims。

在此示例中，添加了两个Claims(“claim-1”和“claim-2”)及其相应的值(“value-1”和“value-2”)。

### 3.2 测试自定义Claims

为了进行测试，我们将使用client_credentials授权类型。

首先，我们将AuthorizationServerConfig中的client_credentials授予类型定义为RegisteredClient对象中的授权授予类型：

```java
@Bean
public RegisteredClientRepository registeredClientRepository() {
    RegisteredClient registeredClient = RegisteredClient.withId(UUID.randomUUID().toString())
            .clientId("articles-client")
            .clientSecret("{noop}secret")
            .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
            .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
            .authorizationGrantType(AuthorizationGrantType.AUTHORIZATION_CODE)
            .authorizationGrantType(AuthorizationGrantType.REFRESH_TOKEN)
            .redirectUri("http://127.0.0.1:8080/login/oauth2/code/articles-client-oidc")
            .redirectUri("http://127.0.0.1:8080/authorized")
            .scope(OidcScopes.OPENID)
            .scope("articles.read")
            .build();

    return new InMemoryRegisteredClientRepository(registeredClient);
}
```

然后，我们在CustomClaimsConfigurationTest类中创建一个测试用例：

```java
@ActiveProfiles(value = "basic-claim")
public class CustomClaimsConfigurationTest {

    private static final String ISSUER_URL = "http://localhost:";
    private static final String USERNAME = "articles-client";
    private static final String PASSWORD = "secret";
    private static final String GRANT_TYPE = "client_credentials";

    @Autowired
    private TestRestTemplate restTemplate;

    @LocalServerPort
    private int serverPort;

    @Test
    public void givenAccessToken_whenGetCustomClaim_thenSuccess() throws ParseException {
        String url = ISSUER_URL + serverPort + "/oauth2/token";
        HttpHeaders headers = new HttpHeaders();
        headers.setBasicAuth(USERNAME, PASSWORD);
        MultiValueMap<String, String> params = new LinkedMultiValueMap<>();
        params.add("grant_type", GRANT_TYPE);
        HttpEntity<MultiValueMap<String, String>> requestEntity = new HttpEntity<>(params, headers);
        ResponseEntity<TokenDTO> response = restTemplate.exchange(url, HttpMethod.POST, requestEntity, TokenDTO.class);

        SignedJWT signedJWT = SignedJWT.parse(response.getBody().getAccessToken());
        JWTClaimsSet claimsSet = signedJWT.getJWTClaimsSet();
        Map<String, Object> claims = claimsSet.getClaims();

        assertEquals("value-1", claims.get("claim-1"));
        assertEquals("value-2", claims.get("claim-2"));
    }

    static class TokenDTO {
        @JsonProperty("access_token")
        private String accessToken;
        @JsonProperty("token_type")
        private String tokenType;
        @JsonProperty("expires_in")
        private String expiresIn;
        @JsonProperty("scope")
        private String scope;

        public String getAccessToken() {
            return accessToken;
        }
    }
}
```

让我们回顾一下测试的关键部分，以了解发生了什么：

- 首先构建OAuth2令牌端点的URL
- 从对令牌端点的POST请求中检索包含TokenDTO类的响应，在这里，我们创建一个带有标头(基本身份验证)和参数(授权类型)的HTTP请求实体
- 使用SignedJWT类从响应中解析访问令牌，此外，我们从JWT中提取Claims并将其存储在Map中
- 使用JUnit断言JWT中的特定Claims具有预期值

**此测试确认我们的令牌编码过程正常工作，并且我们的Claims正在按预期生成**。

此外，我们可以使用curl命令获取访问令牌： 

```shell
curl --request POST \
  --url http://localhost:9000/oauth2/token \
  --header 'Authorization: Basic YXJ0aWNsZXMtY2xpZW50OnNlY3JldA==' \
  --header 'Content-Type: application/x-www-form-urlencoded' \
  --data grant_type=client_credentials
```

此处，凭证被编码为客户端ID和客户端机密的[Base64](https://www.baeldung.com/java-base64-encode-and-decode)字符串，并以冒号“:”分隔。

现在，我们可以使用Profile basic-claim运行Spring Boot应用程序。

如果我们获取访问令牌并使用[jwt.io](https://jwt.io/)对其进行解码，我们会在令牌正文中找到测试Claims：

```json
{
    "sub": "articles-client",
    "aud": "articles-client",
    "nbf": 1704517985,
    "scope": [
        "articles.read",
        "openid"
    ],
    "iss": "http://auth-server:9000",
    "exp": 1704518285,
    "claim-1": "value-1",
    "iat": 1704517985,
    "claim-2": "value-2"
}
```

我们可以看到，测试Claims的值符合预期。

在下一节中，我们将讨论将权限添加为访问令牌的Claims。

## 4. 将权限作为自定义Claims添加到JWT访问令牌

**将权限作为自定义Claims添加到JWT访问令牌通常是保护和管理Spring Boot应用程序中的访问的关键方面**。权限通常由Spring Security中的GrantedAuthority对象表示，指示允许用户执行哪些操作或角色。通过将这些权限作为自定义Claims包含在JWT访问令牌中，我们为资源服务器提供了一种方便且标准化的方式来了解用户的权限。

### 4.1 添加权限作为自定义Claims

首先，我们在DefaultSecurityConfig类中使用带有一组权限的简单内存用户配置：

```java
@Bean
UserDetailsService users() {
    UserDetails user = User.withDefaultPasswordEncoder()
            .username("admin")
            .password("password")
            .roles("USER")
            .build();
    return new InMemoryUserDetailsManager(user);
}
```

创建一个用户名为“admin”、密码为“password”、角色为“USER”的用户。

现在，让我们使用这些权限在访问令牌中填充自定义Claims：

```java
@Bean
@Profile("authority-claim")
public OAuth2TokenCustomizer<JwtEncodingContext> tokenCustomizer(@Qualifier("users") UserDetailsService userDetailsService) {
    return (context) -> {
        UserDetails userDetails = userDetailsService.loadUserByUsername(context.getPrincipal().getName());
        Collection<? extends GrantedAuthority> authorities = userDetails.getAuthorities();
        context.getClaims().claims(claims ->
                claims.put("authorities", authorities.stream().map(authority -> authority.getAuthority()).collect(Collectors.toList())));
    };
}
```

首先，我们定义一个实现OAuth2TokenCustomizer<JwtEncodingContext\>接口的Lambda函数，此函数在编码过程中自定义JWT。

然后，我们从注入的UserDetailsService中检索与当前主体(用户)关联的UserDetails对象，主体的名称通常是用户名。

之后，我们检索与用户关联的GrantedAuthority对象的集合。

最后，我们从JwtEncodingContext检索JWT Claims并应用自定义，它包括向JWT添加名为“authorities”的自定义Claims。此外，此Claims还包含从与用户关联的GrantedAuthority对象获取的权限字符串列表。

### 4.2 测试

现在我们已经配置了授权服务器，让我们测试一下。为此，我们将使用[GitHub](https://github.com/Baeldung/spring-security-oauth/tree/master/oauth-authorization-server/client-server)上提供的客户端-服务器项目。

让我们创建一个REST API客户端，它将从访问令牌中获取Claims列表：

```java
@GetMapping(value = "/claims")
public String getClaims(
        @RegisteredOAuth2AuthorizedClient("articles-client-authorization-code") OAuth2AuthorizedClient authorizedClient
) throws ParseException {
    SignedJWT signedJWT = SignedJWT.parse(authorizedClient.getAccessToken().getTokenValue());
    JWTClaimsSet claimsSet = signedJWT.getJWTClaimsSet();
    Map<String, Object> claims = claimsSet.getClaims();
    return claims.get("authorities").toString();
}
```

@RegisteredOAuth2AuthorizedClient注解用于Spring Boot控制器方法中，指示该方法需要注册OAuth 2.0授权客户端指定的客户端ID。在本例中，客户端ID为“articles-client-authorization-code”。

让我们使用Profile authority-claim运行我们的Spring Boot应用程序。

现在，当我们进入浏览器并尝试访问http://127.0.0.1:8080/claims页面时，我们将自动重定向到http://auth-server:9000/login URL下的OAuth服务器登录页面。

**提供正确的用户名和密码后，授权服务器会将我们重定向回所请求的URL，即Claims列表**。

## 5. 总结

总的来说，向JWT访问令牌添加自定义Claims的能力提供了一种强大的机制，可以根据应用程序的特定需求定制令牌，并增强身份验证和授权系统的整体安全性和功能。

在本文中，我们学习了如何向Spring授权服务器中的JWT访问令牌添加自定义Claims和用户权限。