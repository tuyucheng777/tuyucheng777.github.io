---
layout: post
title:  Spring Authorization Server中的动态客户端注册
category: springsecurity
copyright: springsecurity
excerpt: Spring Authorization Server
---

## 1. 简介

Spring Authorization Server带有一系列合理的默认值，让我们几乎无需配置即可使用它，这使其成为在测试场景中与客户端应用程序一起使用以及当我们想要完全控制用户登录体验时的绝佳选择。

有一项功能虽然可用，但默认情况下并未启用：动态客户端注册。

在本教程中，我们将展示如何从客户端应用程序启用和使用它。

## 2. 为什么要使用动态注册？

当基于OAuth2的应用程序客户端(或按照OIDC的说法-依赖方(RP))启动身份验证流程时，它会将授权服务器自己的客户端标识符发送给身份提供者。

通常，此标识符使用带外进程发送给客户端，然后将其添加到配置中并在需要时使用。

例如，当使用流行的身份提供商解决方案(如Azure的EntraID或Auth0)时，我们可以使用管理控制台或API来配置新客户端。在此过程中，我们需要告知应用程序名称、授权回调URL、支持的范围等。

一旦提供了所需的信息，我们就会得到一个新的客户端标识符，对于所谓的“secret”客户端，还会得到一个客户端机密。然后，我们将这些添加到应用程序的配置中，然后就可以部署它了。

**现在，当我们只有少量应用程序或始终使用单个身份提供者时，此过程可以正常工作。但是，对于更复杂的场景，注册过程需要动态，这就是[OpenID Connect动态客户端注册](https://openid.net/specs/openid-connect-registration-1_0.html)规范发挥作用的地方**。

对于现实世界的案例，一个很好的例子是英国的[OpenBanking](https://www.openbanking.org.uk/)标准，它使用动态客户端注册作为其核心协议之一。

## 3. 动态注册如何工作？

OpenID Connect标准使用单个注册URL，客户端可使用它来注册自己。此操作通过POST请求完成，请求中包含JSON对象，该对象包含执行注册所需的客户端元数据。

**重要的是，访问注册端点需要身份验证，通常是Bearer令牌。当然，这引出了一个问题：想要成为客户端的人如何获取此操作的令牌**？

不幸的是，答案并不明确。一方面，规范指出端点是受保护的资源，因此需要某种形式的身份验证。另一方面，它还提到了开放注册端点的可能性。

对于Spring Authorization Server，注册需要具有client.create范围的Bearer令牌。要创建此令牌，我们使用常规OAuth2的令牌端点和基本凭据。

这是成功注册的结果序列：

![](/assets/images/2025/springsecurity/springdynamicclientregistration01.png)

一旦客户端成功完成注册，它就可以使用返回的客户端ID和密钥来执行任何标准授权流程。

## 4. 实现动态注册

现在我们了解了所需的步骤，让我们使用两个Spring Boot应用程序创建一个测试场景。一个将托管Spring Authorization Server，另一个将是使用Spring Security OAuth2模块的简单Web MVC应用程序。

后者不会使用客户端的常规静态配置，而是使用动态注册端点在启动时获取客户端标识符和密钥。

让我们从服务器开始。

## 5. 授权服务器实现

我们首先添加所需的Maven依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-authorization-server</artifactId>
    <version>1.3.1</version>
</dependency>
```

最新版本可在[Maven Central](https://mvnrepository.com/artifact/org.springframework.security/spring-security-oauth2-authorization-server)上获取。

对于常规的Spring Authorization Server应用程序，此依赖就是我们所需要的全部。**但是，出于安全原因，默认情况下不启用动态注册**。此外，截至本文撰写时，无法仅使用配置属性来启用它。

这意味着我们必须添加一些代码。

### 5.1 启用动态注册

**OAuth2AuthorizationServerConfigurer是配置授权服务器所有方面的门户，包括注册端点**。此配置应作为创建SecurityFilterChain Bean的一部分完成：

```java
@Configuration
@EnableConfigurationProperties(SecurityConfig.RegistrationProperties.class)
public class SecurityConfig {
    @Bean
    @Order(1)
    public SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
        OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
        http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
                .oidc(oidc -> {
                    oidc.clientRegistrationEndpoint(Customizer.withDefaults());
                });

        http.exceptionHandling((exceptions) -> exceptions
                .defaultAuthenticationEntryPointFor(
                        new LoginUrlAuthenticationEntryPoint("/login"),
                        new MediaTypeRequestMatcher(MediaType.TEXT_HTML)
                )
        );

        http.oauth2ResourceServer((resourceServer) -> resourceServer
                .jwt(Customizer.withDefaults()));

        return http.build();
    }

    // ... other beans omitted
}
```

在这里，我们使用服务器的配置器oidc()方法来访问OidcConfigurer实例，此子配置器具有允许我们控制与OpenID Connect标准相关的端点的方法。要启用注册端点，我们使用具有默认配置的clientRegistrationEndpoint()方法，这将使用Bearer令牌授权在/connect/register路径上启用注册。进一步的配置选项包括：

- 定义自定义身份验证
- 对收到的注册数据进行自定义处理
- 对发送给客户端的响应进行自定义处理

现在，由于我们提供了一个自定义的SecurityFilterChain，Spring Boot的自动配置将会退后一步，让我们负责在配置中添加一些额外的部分。

具体来说，我们需要添加设置表单登录认证的逻辑：

```java
@Bean
@Order(2)
SecurityFilterChain loginFilterChain(HttpSecurity http) throws Exception {
    return http.authorizeHttpRequests(r -> r.anyRequest().authenticated())
            .formLogin(Customizer.withDefaults())
            .build();
}
```

### 5.2 注册客户端配置

如上所述，注册机制本身需要客户端发送Bearer令牌，Spring Authorization Server通过要求客户端使用客户端凭据流来生成此令牌来解决这个先有鸡还是先有蛋的问题。

此令牌请求所需的范围是client.create，并且客户端必须使用服务器支持的身份验证方案之一。在这里，我们将使用[基本凭证](https://datatracker.ietf.org/doc/html/rfc7617)，但在实际场景中，我们可以使用其他方法。

从授权服务器的角度来看，此注册客户端只是另一个客户端。因此，我们将使用RegisteredClient流式API创建它：

```java
@Bean
public RegisteredClientRepository registeredClientRepository(RegistrationProperties props) {
    RegisteredClient registrarClient = RegisteredClient.withId(UUID.randomUUID().toString())
            .clientId(props.getRegistrarClientId())
            .clientSecret(props.getRegistrarClientSecret())
            .clientAuthenticationMethod(ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
            .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
            .clientSettings(ClientSettings.builder()
                    .requireProofKey(false)
                    .requireAuthorizationConsent(false)
                    .build())
            .scope("client.create")
            .scope("client.read")
            .build();

    RegisteredClientRepository delegate = new  InMemoryRegisteredClientRepository(registrarClient);
    return new CustomRegisteredClientRepository(delegate);
}
```

我们使用了@ConfigurationProperties类来允许使用Spring的标准Environment机制配置客户端ID和密钥属性。

此引导注册将是启动时创建的唯一注册，我们将在返回之前将其添加到我们的自定义RegisteredClientRepository中。

### 5.3 自定义RegisteredClientRepository

Spring Authorization Server使用已配置的RegisteredClientRepository实现将所有已注册的客户端存储在服务器中，它开箱即用，带有内存和基于JDBC的实现，涵盖了基本用例。

但是，这些实现在保存注册之前不提供任何自定义注册的功能。在我们的案例中，我们希望修改默认的ClientProperties设置，以便在授权用户时不需要同意或[PKCE](https://www.baeldung.com/spring-security-pkce-secret-clients)。

我们的实现将大多数方法委托给构造时传递的实际Repository，重要的例外是save()方法：

```java
@Override
public void save(RegisteredClient registeredClient) {
    Set<String> scopes = ( registeredClient.getScopes() == null || registeredClient.getScopes().isEmpty())?
            Set.of("openid","email","profile"):
            registeredClient.getScopes();

    // Disable PKCE & Consent
    RegisteredClient modifiedClient = RegisteredClient.from(registeredClient)
            .scopes(s -> s.addAll(scopes))
            .clientSettings(ClientSettings
                    .withSettings(registeredClient.getClientSettings().getSettings())
                    .requireAuthorizationConsent(false)
                    .requireProofKey(false)
                    .build())
            .build();

    delegate.save(modifiedClient);
}
```

**在这里，我们根据收到的RegisteredClient创建一个新注册客户端，并根据需要更改ClientSettings**。然后，此新注册客户端将传递到后端，并在那里存储，直到需要时为止。

至此，服务器实现完毕。现在，让我们继续客户端。

## 6. 动态注册客户端实现

我们的客户端也将是一个标准的[Spring Web MVC](https://www.baeldung.com/spring-mvc-tutorial)应用程序，其中有一个页面显示当前用户信息。Spring Security(或者更具体地说它的OAuth2登录模块)将处理所有安全方面的问题。

让我们从所需的Maven依赖开始：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.3.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
    <version>3.3.2</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
    <version>3.3.2</version>
</dependency>
```

这些依赖的最新版本可在Maven Central上找到：

- [spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)
- [spring-boot-starter-thymeleaf](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-thymeleaf)
- [spring-boot-starter-oauth2-client](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-oauth2-client)

### 6.1 安全配置

默认情况下，Spring Boot的自动配置机制使用来自可用PropertySources的信息来收集所需的数据创建一个或多个ClientRegistration实例，然后将其存储在基于内存的ClientRegistrationRepository中。

例如，给定这个application.yaml：

```yaml
spring:
    security:
        oauth2:
            client:
                provider:
                    spring-auth-server:
                        issuer-uri: http://localhost:8080
                registration:
                    test-client:
                        provider: spring-auth-server
                        client-name: test-client
                        client-id: xxxxx
                        client-secret: yyyy
                        authorization-grant-type:
                            - authorization_code
                            - refresh_token
                            - client_credentials
                        scope:
                            - openid
                            - email
                            - profile
```

Spring将创建一个名为test-client的ClientRegistration并将其传递给Repository。

稍后，当需要启动身份验证流程时，OAuth2引擎会查询此Repository并通过其注册标识符恢复注册-在我们的例子中为test-client。

这里的关键点是授权服务器应该已经知道此时返回的ClientRegistration，**这意味着为了支持动态客户端，我们必须实现一个替代Repository并将其公开为@Bean**。

**这样，Spring Boot的自动配置将自动使用它而不是默认配置**。

### 6.2 动态客户端注册Repository

正如预期的那样，我们的实现必须实现ClientRegistration接口，该接口仅包含一个方法：findByRegistrationId()。这就提出了一个问题：**OAuth2引擎如何知道哪些注册可用？毕竟，它可以在默认登录页面上列出它们**。

事实证明，Spring Security希望Repository也能实现Iterable<ClientRegistration\>，以便它可以枚举可用的客户端：

```java
public class DynamicClientRegistrationRepository implements ClientRegistrationRepository, Iterable<ClientRegistration> {
    private final RegistrationDetails registrationDetails;
    private final Map<String, ClientRegistration> staticClients;
    private final RegistrationRestTemplate registrationClient;
    private final Map<String, ClientRegistration> registrations = new HashMap<>();

    // ... implementation omitted
}
```

我们的类需要一些输入才能有用：

- 包含执行动态注册所需的所有参数的RegistrationDetails记录
- 将动态注册的客户端Map
- 用于访问授权服务器的RestTemplate

**请注意，对于此示例，我们假设所有客户端都将在同一个授权服务器上注册**。

另一个重要的设计决策是定义动态注册何时发生，在这里，我们将采用一种简单的方法，并公开一个公共doRegistrations()方法，该方法将注册所有已知客户端并保存返回的客户端标识符和密钥以供以后使用：

```java
public void doRegistrations() {
    staticClients.forEach((key, value) -> findByRegistrationId(key));
}
```

该实现会针对传递给构造函数的每个静态客户端调用findByRegistrationId()，此方法检查给定标识符是否有有效注册，如果缺少，则触发实际注册过程。

### 6.3 动态注册

doRegistration()函数是实际操作发生的地方：

```java
private ClientRegistration doRegistration(String registrationId) {
    String token = createRegistrationToken();
    var staticRegistration = staticClients.get(registrationId);

    var body = Map.of(
            "client_name", staticRegistration.getClientName(),
            "grant_types", List.of(staticRegistration.getAuthorizationGrantType()),
            "scope", String.join(" ", staticRegistration.getScopes()),
            "redirect_uris", List.of(resolveCallbackUri(staticRegistration)));

    var headers = new HttpHeaders();
    headers.setBearerAuth(token);
    headers.setContentType(MediaType.APPLICATION_JSON);

    var request = new RequestEntity<>(
            body,
            headers,
            HttpMethod.POST,
            registrationDetails.registrationEndpoint());

    var response = registrationClient.exchange(request, ObjectNode.class);
    // ... error handling omitted
    return createClientRegistration(staticRegistration, response.getBody());
}
```

首先，我们必须获取一个注册令牌，以便调用注册端点。**请注意，每次注册尝试时，我们都必须获取一个新令牌，因为如Spring Authorization Server的文档中所述，我们只能使用此令牌一次**。

接下来，我们使用来自静态注册对象的数据构建注册有效负载，添加所需的授权和content-type标头，然后将请求发送到注册端点。

最后，我们使用响应数据来创建最终的ClientRegistration，它将保存在Repository的缓存中并返回给OAuth2引擎。

### 6.4 注册动态Repository @Bean

为了完成我们的客户端，最后一步是将DynamicClientRegistrationRepository公开为@Bean，让我们为此创建一个@Configuration类：

```java
@Bean
ClientRegistrationRepository dynamicClientRegistrationRepository( DynamicClientRegistrationRepository.RegistrationRestTemplate restTemplate) {
    var registrationDetails = new DynamicClientRegistrationRepository.RegistrationDetails(
            registrationProperties.getRegistrationEndpoint(),
            registrationProperties.getRegistrationUsername(),
            registrationProperties.getRegistrationPassword(),
            registrationProperties.getRegistrationScopes(),
            registrationProperties.getGrantTypes(),
            registrationProperties.getRedirectUris(),
            registrationProperties.getTokenEndpoint());

    Map<String,ClientRegistration> staticClients = (new OAuth2ClientPropertiesMapper(clientProperties)).asClientRegistrations();
    var repo =  new DynamicClientRegistrationRepository(registrationDetails, staticClients, restTemplate);
    repo.doRegistrations();
    return repo;
}
```

带有@Bean注解的dynamicClientRegistrationRepository()方法通过首先从可用属性中填充RegistrationDetails记录来创建Repository。

**其次，它利用Spring Boot自动配置模块中提供的OAuth2ClientPropertiesMapper类创建staticClient映射**。这种方法使我们能够以最小的努力快速地从静态客户端切换到动态客户端并切换回来，因为两者的配置结构是相同的。

## 7. 测试

最后，让我们进行一些集成测试。首先，我们启动服务器应用程序，该应用程序配置为监听端口8080：

```text
[ server ] $ mvn spring-boot:run
... lots of messages omitted
[           main] c.t.t.s.s.a.AuthorizationServerApplication : Started AuthorizationServerApplication in 2.222 seconds (process running for 2.454)
[           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state LivenessState changed to CORRECT
[           main] o.s.b.a.ApplicationAvailabilityBean      : Application availability state ReadinessState changed to ACCEPTING_TRAFFIC
```

接下来，在另一个shell中启动客户端：

```text
[client] $ mvn spring-boot:run
// ... lots of messages omitted
[  restartedMain] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
[  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8090 (http) with context path ''
[  restartedMain] d.c.DynamicRegistrationClientApplication : Started DynamicRegistrationClientApplication in 2.063 seconds (process running for 2.425)

```

这两个应用程序都运行了debug属性，因此会产生大量日志消息。**具体来说，我们可以看到对授权服务器的/connect/register端点的调用**：

```text
[nio-8080-exec-3] o.s.security.web.FilterChainProxy        : Securing POST /connect/register
// ... lots of messages omitted
[nio-8080-exec-3] ClientRegistrationAuthenticationProvider : Retrieved authorization with initial access token
[nio-8080-exec-3] ClientRegistrationAuthenticationProvider : Validated client registration request parameters
[nio-8080-exec-3] s.s.a.r.CustomRegisteredClientRepository : Saving registered client: id=30OTlhO1Fb7UF110YdXULEDbFva4Uc8hPBGMfi60Wik, name=test-client
```

在客户端，我们可以看到一条带有注册标识符(test-client)和对应的client_id的消息：

```text
[  restartedMain] s.d.c.c.OAuth2DynamicClientConfiguration : Creating a dynamic client registration repository
[  restartedMain] .c.s.DynamicClientRegistrationRepository : findByRegistrationId: test-client
[  restartedMain] .c.s.DynamicClientRegistrationRepository : doRegistration: registrationId=test-client
[  restartedMain] .c.s.DynamicClientRegistrationRepository : creating ClientRegistration: registrationId=test-client, client_id=30OTlhO1Fb7UF110YdXULEDbFva4Uc8hPBGMfi60Wik
```

如果我们打开浏览器并指向http://localhost:8090，我们将被重定向到登录页面。**请注意，地址栏中的URL已更改为http://localhost:8080，这表明此页面来自授权服务器**。

测试凭证是user1/password，一旦我们将它们填入表单并发送，我们将返回到客户端的主页。由于我们现在已经通过身份验证，我们将看到一个页面，其中包含从授权令牌中提取的一些详细信息。

## 8. 总结

在本教程中，我们展示了如何启用Spring Authorization Server的动态注册功能并从基于Spring Security的客户端应用程序中使用它。