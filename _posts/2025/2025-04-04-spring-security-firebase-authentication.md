---
layout: post
title:  将Firebase身份验证与Spring Security集成
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在现代Web应用程序中，用户身份验证和授权是关键组件，从头开始构建我们的身份验证层是一项具有挑战性且复杂的任务。但是，随着基于云的身份验证服务的兴起，这个过程变得简单得多。

一个这样的例子就是[Firebase Authentication](https://firebase.google.com/docs/auth)，这是[Firebase和Google](https://firebase.google.com/firebase-and-gcp)提供的完全托管的身份验证服务。

**在本教程中，我们将探讨如何将Firebase Authentication与[Spring Security](https://www.baeldung.com/security-spring)集成以创建和验证我们的用户**。我们将介绍必要的配置，实现用户注册和登录功能，并创建自定义身份验证过滤器以验证私有API端点的用户令牌。

## 2. 设置项目

在深入实现之前，我们需要包含SDK依赖并正确配置我们的应用程序。

### 2.1 依赖

让我们首先将[Firebase Admin依赖](https://mvnrepository.com/artifact/com.google.firebase/firebase-admin/latest)添加到项目的pom.xml文件中：

```xml
<dependency>
    <groupId>com.google.firebase</groupId>
    <artifactId>firebase-admin</artifactId>
    <version>9.3.0</version>
</dependency>
```

此依赖为我们提供了从我们的应用程序与Firebase身份验证服务交互所需的类。

### 2.2 定义Firebase配置Bean

现在，要与Firebase Authentication进行交互，我们需要配置[私钥](https://clemfournier.medium.com/how-to-get-my-firebase-service-account-key-file-f0ec97a21620)来验证API请求。

为了演示，我们将在src/main/resources目录中创建private-key.json文件。**但是在生产中，应从环境变量中加载私钥或从机密管理系统中获取私钥以增强安全性**。

我们将使用@Value注解加载我们的私钥并使用它来定义我们的Bean：

```java
@Value("classpath:/private-key.json")
private Resource privateKey;

@Bean
public FirebaseApp firebaseApp() {
    InputStream credentials = new ByteArrayInputStream(privateKey.getContentAsByteArray());
    FirebaseOptions firebaseOptions = FirebaseOptions.builder()
            .setCredentials(GoogleCredentials.fromStream(credentials))
            .build();
    return FirebaseApp.initializeApp(firebaseOptions);
}

@Bean
public FirebaseAuth firebaseAuth(FirebaseApp firebaseApp) {
    return FirebaseAuth.getInstance(firebaseApp);
}
```

我们首先定义FirebaseApp Bean，然后使用它来创建FirebaseAuth Bean，这允许我们在使用多个Firebase服务(例如[Cloud Firestore Database](https://www.baeldung.com/spring-boot-firebase)、[Firebase Messaging](https://www.baeldung.com/spring-fcm)等)时重用FirebaseApp Bean。

FirebaseAuth类是与Firebase Authentication服务交互的主要入口点。

## 3. 在Firebase Authentication中创建用户

现在我们已经定义了FirebaseAuth Bean，让我们创建一个UserService类并引用它来在Firebase Authentication中创建新用户：

```java
private static final String DUPLICATE_ACCOUNT_ERROR = "EMAIL_EXISTS";

public void create(String emailId, String password) {
    CreateRequest request = new CreateRequest();
    request.setEmail(emailId);
    request.setPassword(password);
    request.setEmailVerified(Boolean.TRUE);

    try {
        firebaseAuth.createUser(request);
    } catch (FirebaseAuthException exception) {
        if (exception.getMessage().contains(DUPLICATE_ACCOUNT_ERROR)) {
            throw new AccountAlreadyExistsException("Account with given email-id already exists");
        }
        throw exception;
    }
}
```

在create()方法中，我们使用用户的email和password初始化一个新的CreateRequest对象。**为简单起见，我们还将emailVerified值设置为true，但是，在生产应用程序中执行此操作之前，我们可能希望先实现电子邮件验证流程**。

此外，我们处理具有给定emailId的帐户已存在的情况，抛出自定义的AccountAlreadyExistsException。    

## 4. 实现用户登录功能

现在我们可以创建用户了，我们自然必须允许他们在访问我们的私有API端点之前进行身份验证。**我们将实现用户登录功能，该功能以[JWT](https://www.baeldung.com/java-json-web-tokens-jjwt)的形式返回ID令牌，并在身份验证成功后[返回刷新令牌](https://www.baeldung.com/cs/access-refresh-tokens#tcp-connection)**。

Firebase Admin SDK不支持使用电子邮件/密码凭据进行令牌交换，因为此功能通常由客户端应用程序处理。不过，为了演示，我们将直接从后端应用程序调用[登录REST API](https://firebase.google.com/docs/reference/rest/auth#section-sign-in-email-password)。

首先，我们将声明几个记录来表示请求和响应有效负载：

```java
record FirebaseSignInRequest(String email, String password, boolean returnSecureToken) {}

record FirebaseSignInResponse(String idToken, String refreshToken) {}
```

**要调用Firebase Authentication REST API，我们需要Firebase项目的[Web API密钥](https://firebase.google.com/docs/projects/api-keys#find-api-keys:~:text=the%20current_key%20field.-,Firebase%20Web%20Apps)**。我们将其存储在application.yaml文件中，并使用@Value注解将其注入到新的FirebaseAuthClient类中：

```java
private static final String API_KEY_PARAM = "key";
private static final String INVALID_CREDENTIALS_ERROR = "INVALID_LOGIN_CREDENTIALS";
private static final String SIGN_IN_BASE_URL = "https://identitytoolkit.googleapis.com/v1/accounts:signInWithPassword";

@Value("${com.baeldung.firebase.web-api-key}")
private String webApiKey;

public FirebaseSignInResponse login(String emailId, String password) {
    FirebaseSignInRequest requestBody = new FirebaseSignInRequest(emailId, password, true);
    return sendSignInRequest(requestBody);
}

private FirebaseSignInResponse sendSignInRequest(FirebaseSignInRequest firebaseSignInRequest) {
    try {
        return RestClient.create(SIGN_IN_BASE_URL)
                .post()
                .uri(uriBuilder -> uriBuilder
                        .queryParam(API_KEY_PARAM, webApiKey)
                        .build())
                .body(firebaseSignInRequest)
                .contentType(MediaType.APPLICATION_JSON)
                .retrieve()
                .body(FirebaseSignInResponse.class);
    } catch (HttpClientErrorException exception) {
        if (exception.getResponseBodyAsString().contains(INVALID_CREDENTIALS_ERROR)) {
            throw new InvalidLoginCredentialsException("Invalid login credentials provided");
        }
        throw exception;
    }
}
```

在我们的login()方法中，我们使用用户的email、password创建一个FirebaseSignInRequest，并将returnSecureToken设置为true。然后，我们将此请求传递给我们的私有sendSignInRequest()方法，该方法使用RestClient向Firebase Authentication REST API发送POST请求。

如果请求成功，我们将包含用户idToken和refreshToken的响应返回给调用者。如果登录凭据无效，我们将抛出自定义的InvalidLoginCredentialsException。

**值得注意的是，我们从Firebase收到的idToken的有效期为一小时，并且我们无法更改它**。在下一节中，我们将探讨如何允许我们的客户端应用程序使用返回的refreshToken来获取新的ID令牌。

## 5. 将刷新令牌兑换为新ID令牌

现在我们已经有了登录功能，**让我们看看如何在当前idToken过期时使用refreshToken获取新的idToken**。这允许我们的客户端应用程序让用户长时间保持登录状态，而无需他们重新输入凭据。

我们首先定义记录来表示请求和响应负载：

```java
record RefreshTokenRequest(String grant_type, String refresh_token) {}

record RefreshTokenResponse(String id_token) {}
```

接下来，在我们的FirebaseAuthClient类中，让我们调用[刷新令牌交换REST API](https://firebase.google.com/docs/reference/rest/auth#section-refresh-token)：

```java
private static final String REFRESH_TOKEN_GRANT_TYPE = "refresh_token";
private static final String INVALID_REFRESH_TOKEN_ERROR = "INVALID_REFRESH_TOKEN";
private static final String REFRESH_TOKEN_BASE_URL = "https://securetoken.googleapis.com/v1/token";

public RefreshTokenResponse exchangeRefreshToken(String refreshToken) {
    RefreshTokenRequest requestBody = new RefreshTokenRequest(REFRESH_TOKEN_GRANT_TYPE, refreshToken);
    return sendRefreshTokenRequest(requestBody);
}

private RefreshTokenResponse sendRefreshTokenRequest(RefreshTokenRequest refreshTokenRequest) {
    try {
        return RestClient.create(REFRESH_TOKEN_BASE_URL)
                .post()
                .uri(uriBuilder -> uriBuilder
                        .queryParam(API_KEY_PARAM, webApiKey)
                        .build())
                .body(refreshTokenRequest)
                .contentType(MediaType.APPLICATION_JSON)
                .retrieve()
                .body(RefreshTokenResponse.class);
    } catch (HttpClientErrorException exception) {
        if (exception.getResponseBodyAsString().contains(INVALID_REFRESH_TOKEN_ERROR)) {
            throw new InvalidRefreshTokenException("Invalid refresh token provided");
        }
        throw exception;
    }
}
```

在我们的exchangeRefreshToken()方法中，我们使用refresh_token授权类型和提供的refreshToken创建一个RefreshTokenRequest。然后，我们将此请求传递给我们的私有sendRefreshTokenRequest()方法，该方法将POST请求发送到所需的API端点。

如果请求成功，我们将返回包含新idToken的响应。如果提供的refreshToken无效，我们将抛出自定义的InvalidRefreshTokenException。

**此外，如果我们需要强制用户重新进行身份验证，我们可以撤销他们的刷新令牌**：

```java
firebaseAuth.revokeRefreshTokens(userId);
```

我们调用FirebaseAuth类提供的revokeRefreshTokens()方法，这不仅会使发给用户的所有refreshToken失效，还会使用户的活动idToken失效，从而有效地将其从我们的应用程序中注销。

## 6. 与Spring Security集成

通过实现用户创建和登录功能，让我们将Firebase Authentication与Spring Security集成，以保护我们的私有API端点。

### 6.1 创建自定义身份验证过滤器

首先，我们将创建扩展[OncePerRequestFilter](https://www.baeldung.com/spring-onceperrequestfilter)类的自定义身份验证过滤器：

```java
@Component
class TokenAuthenticationFilter extends OncePerRequestFilter {

    private static final String BEARER_PREFIX = "Bearer ";
    private static final String USER_ID_CLAIM = "user_id";
    private static final String AUTHORIZATION_HEADER = "Authorization";

    private final FirebaseAuth firebaseAuth;
    private final ObjectMapper objectMapper;

    // standard constructor

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
                                    FilterChain filterChain) {
        String authorizationHeader = request.getHeader(AUTHORIZATION_HEADER);

        if (authorizationHeader != null && authorizationHeader.startsWith(BEARER_PREFIX)) {
            String token = authorizationHeader.replace(BEARER_PREFIX, "");
            Optional<String> userId = extractUserIdFromToken(token);

            if (userId.isPresent()) {
                var authentication = new UsernamePasswordAuthenticationToken(userId.get(), null, null);
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authentication);
            } else {
                setAuthErrorDetails(response);
                return;
            }
        }
        filterChain.doFilter(request, response);
    }

    private Optional<String> extractUserIdFromToken(String token) {
        try {
            FirebaseToken firebaseToken = firebaseAuth.verifyIdToken(token, true);
            String userId = String.valueOf(firebaseToken.getClaims().get(USER_ID_CLAIM));
            return Optional.of(userId);
        } catch (FirebaseAuthException exception) {
            return Optional.empty();
        }
    }

    private void setAuthErrorDetails(HttpServletResponse response) {
        HttpStatus unauthorized = HttpStatus.UNAUTHORIZED;
        response.setStatus(unauthorized.value());
        response.setContentType(MediaType.APPLICATION_JSON_VALUE);
        ProblemDetail problemDetail = ProblemDetail.forStatusAndDetail(unauthorized,
                "Authentication failure: Token missing, invalid or expired");
        response.getWriter().write(objectMapper.writeValueAsString(problemDetail));
    }
}
```

在doFilterInternal()方法中，我们从传入的HTTP请求中提取Authorization标头并删除Bearer前缀以获取JWT令牌。

然后，使用我们的私有extractUserIdFromToken()方法，我们验证令牌的真实性并检索其user_id声明。

如果令牌验证失败，我们将创建一个[ProblemDetail](https://www.baeldung.com/spring-boot-return-errors-problemdetail)错误响应，使用ObjectMapper将其转换为JSON，并将其写入HttpServletResponse。

**如果令牌有效，我们将创建一个新的UsernamePasswordAuthenticationToken实例，并使用userId作为Principal，然后将其设置在[SecurityContext](https://www.baeldung.com/security-context-basics)中**。

身份验证成功后，我们可以从服务层中的SecurityContext中检索经过身份验证的用户的userId：

```java
String userId = Optional.ofNullable(SecurityContextHolder.getContext().getAuthentication())
    .map(Authentication::getPrincipal)
    .filter(String.class::isInstance)
    .map(String.class::cast)
    .orElseThrow(IllegalStateException::new);
```

为了遵循单一责任原则，我们可以将上述逻辑放在单独的AuthenticatedUserIdProvider类中，这有助于服务层维护当前经过身份验证的用户与他们执行的操作之间的关系。

### 6.2 配置SecurityFilterChain

最后，让我们配置SecurityFilterChain来使用我们的自定义身份验证过滤器：

```java
private static final String[] WHITELISTED_API_ENDPOINTS = { "/user", "/user/login", "/user/refresh-token" };

private final TokenAuthenticationFilter tokenAuthenticationFilter;

// standard constructor

@Bean
public SecurityFilterChain configure(HttpSecurity http) {
    http.authorizeHttpRequests(authManager -> {
                authManager.requestMatchers(HttpMethod.POST, WHITELISTED_API_ENDPOINTS)
                        .permitAll()
                        .anyRequest()
                        .authenticated();
            })
            .addFilterBefore(tokenAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

    return http.build();
}
```

我们允许未经身份验证的访问/user，/user/login和/user/refresh-token端点，这对应于我们的用户注册，登录和刷新令牌交换功能。

最后，我们在过滤器链中的UsernamePasswordAuthenticationFilter之前添加自定义的TokenAuthenticationFilter。

**此设置确保我们的私有API端点受到保护，并且只有具有有效JWT令牌的请求才被允许访问它们**。

## 7. 总结

在本文中，我们探讨了如何将Firebase Authentication与Spring Security集成。

我们完成了必要的配置，实现了用户注册、登录和刷新令牌交换功能，并创建了一个自定义Spring Security过滤器来保护我们的私有API端点。

通过使用Firebase Authentication，我们可以减轻管理用户凭据和访问的复杂性，从而让我们专注于构建核心功能。