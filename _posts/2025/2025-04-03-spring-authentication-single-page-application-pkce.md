---
layout: post
title:  在Spring Authorization Server中使用PKCE进行单页应用程序身份验证
category: springsecurity
copyright: springsecurity
excerpt: Spring Authorization Server
---

## 1. 简介

在本教程中，我们将讨论[OAuth 2.0](https://www.baeldung.com/spring-security-oauth-resource-server)公共客户端的代码交换证明密钥(PKCE)的使用。

## 2. 背景

**OAuth 2.0公共客户端(例如单页应用程序(SPA)或使用授权码授予的移动应用程序)容易受到授权码拦截攻击**，如果客户端-服务器通信发生在不安全的网络上，恶意攻击者可能会从授权端点拦截授权码。

如果攻击者可以访问授权码，则可以使用它来获取访问令牌。一旦攻击者拥有访问令牌，它就可以像合法应用程序用户一样访问受保护的应用程序资源，从而严重危害应用程序。例如，如果访问令牌与金融应用程序相关联，攻击者可能会访问敏感的应用程序信息。

### 2.1 OAuth代码拦截攻击

在本节中，让我们讨论一下Oauth授权代码拦截攻击是如何发生的：

![](/assets/images/2025/springsecurity/springauthenticationsinglepageapplicationpkce01.png)

上图演示了恶意攻击者如何滥用授权授予代码来获取访问令牌的流程：

1. 合法的OAuth应用程序使用其Web浏览器启动OAuth授权请求流程，并提供所有必需的详细信息
2. Web浏览器将请求发送到授权服务器
3. 授权服务器将授权码返回给Web浏览器
4. **在此阶段，如果通信通过不安全的渠道进行，恶意用户可能会访问授权码**
5. 恶意用户交换授权码授权以从授权服务器获取访问令牌
6. 由于授权许可有效，授权服务器向恶意应用程序发出访问令牌。恶意应用程序可以滥用访问令牌，以合法应用程序的名义访问受保护的资源

代码交换的证明密钥是OAuth框架的扩展，旨在减轻这种攻击。

## 3. 使用OAuth的PKCE

PKCE扩展包括OAuth授权码授予流程的以下附加步骤：

- **客户端应用程序在初始授权请求中发送两个额外参数code_challenge和code_challenge_method**
- **客户端在下一步交换授权码以获取访问令牌时，还会发送code_verifier**

首先，支持PKCE的客户端应用程序会选择一个动态创建的加密随机密钥，称为code_verifier。此code_verifier对于每个授权请求都是唯一的，根据[PKCE规范](https://datatracker.ietf.org/doc/html/rfc7636)，code_verifier值的长度必须介于43到128个八位字节之间。

此外，code_verifier只能包含字母数字ASCII字符和一些允许的符号。其次，使用支持的code_challenge_method将code_verifier转换为code_challenge。目前，支持的转换方法是plain和S256。plain是一种无操作转换，使code_challenge值与code_verifier保持一致。S256方法首先生成code_verifier的SHA-256哈希，然后对哈希值执行Base64编码。

### 3.1 防止OAuth代码拦截攻击

下图演示了PKCE扩展如何防止访问令牌被盗：

![](/assets/images/2025/springsecurity/springauthenticationsinglepageapplicationpkce02.png)

1. 合法的OAuth应用程序使用其Web浏览器启动OAuth授权请求流程，**提供所有必需的详细信息以及code_challenge和code_challenge_method参数**
2. Web浏览器将请求发送到授权服务器，**并为客户端应用程序存储code_challenge和code_challenge_method**
3. 授权服务器将授权码返回给Web浏览器
4. 在此阶段，如果通信通过不安全的渠道进行，恶意用户可能会访问授权码
5. **恶意用户尝试交换授权代码授权，以从授权服务器获取访问令牌**。但是，恶意用户不知道需要随请求一起发送的code_verifier，授权服务器拒绝向恶意应用程序发送访问令牌请求
6. 合法应用程序提供code_verifier和授权许可以获得访问令牌，授权服务器根据提供的code_verifier和之前从授权代码授予请求中存储的code_challenge_method计算code_challenge。它将计算出的code_challenge与之前存储的code_challenge进行匹配，这些值始终匹配，并且客户端会获得访问令牌
7. 客户端可以使用此访问令牌访问应用程序资源

## 4. 使用Spring Security的PKCE

**从6.3版本开始，Spring Security支持Servlet和响应式Web应用程序的PKCE**，但是，默认情况下不启用它，因为并非所有身份提供者都支持PKCE扩展。当客户端在不受信任的环境(例如本机应用程序或基于Web浏览器的应用程序)中运行且client_secret为空或未提供且客户端身份验证方法设置为none时，会自动为公共客户端使用PKCE。

### 4.1 Maven配置

[Spring Authorization Server](https://www.baeldung.com/spring-security-oauth-auth-server)支持PKCE扩展，因此，为Spring授权服务器应用程序添加PKCE支持的简单方法是添加spring-boot-starter-oauth2-authorization-server依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-authorization-server</artifactId>
    <version>3.3.0</version>
</dependency>
```

### 4.2 注册公共客户端

接下来，让我们通过在application.yml文件中配置以下属性来注册一个公共的单页应用程序客户端：

```yaml
spring:
    security:
        oauth2:
            authorizationserver:
                client:
                    public-client:
                        registration:
                            client-id: "public-client"
                            client-authentication-methods:
                                - "none"
                            authorization-grant-types:
                                - "authorization_code"
                            redirect-uris:
                                - "http://127.0.0.1:3000/callback"
                            scopes:
                                - "openid"
                                - "profile"
                                - "email"
                        require-authorization-consent: true
                        require-proof-key: true
```

在上面的代码片段中，我们注册了一个客户端，client_id为public-client，client-authentication-methods为none。**require-authorization-consent要求最终用户在成功认证后提供额外的同意才能访问个人资料和电子邮件范围，require-proof-key配置可防止PKCE降级攻击**。

启用require-proof-key配置后，授权服务器将不允许任何恶意尝试绕过没有code_challenge的PKCE流程，其余配置是向授权服务器注册客户端的标准配置。

### 4.3 Spring Security配置

接下来，让我们为授权服务器定义SecurityFileChain配置：

```java
@Bean
@Order(1)
SecurityFilterChain authorizationServerSecurityFilterChain(HttpSecurity http) throws Exception {
    OAuth2AuthorizationServerConfiguration.applyDefaultSecurity(http);
    http.getConfigurer(OAuth2AuthorizationServerConfigurer.class)
            .oidc(Customizer.withDefaults());
    http.exceptionHandling((exceptions) -> exceptions.defaultAuthenticationEntryPointFor(new LoginUrlAuthenticationEntryPoint("/login"), new MediaTypeRequestMatcher(MediaType.TEXT_HTML)))
            .oauth2ResourceServer((oauth2) -> oauth2.jwt(Customizer.withDefaults()));
    return http.cors(Customizer.withDefaults())
            .build();
}
```

在上面的配置中，我们首先应用授权服务器的默认安全设置。然后，我们应用OIDC、CORS和Oauth2资源服务器的Spring Security默认设置，现在让我们定义另一个SecurityFilterChain配置，它将应用于其他HTTP请求，例如登录页面：

```java
@Bean
@Order(2)
SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests((authorize) -> authorize.anyRequest()
                    .authenticated())
            .formLogin(Customizer.withDefaults());
    return http.cors(Customizer.withDefaults())
            .build();
}
```

在此示例中，我们使用一个非常简单的React应用程序作为我们的公共客户端，此应用程序在http://127.0.0.1:3000上运行，授权服务器在不同的端口9000上运行。由于这两个应用程序在不同的域上运行，我们需要提供额外的CORS设置，以便授权服务器允许React应用程序访问它：

```java
@Bean
CorsConfigurationSource corsConfigurationSource() {
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    CorsConfiguration config = new CorsConfiguration();
    config.addAllowedHeader("*");
    config.addAllowedMethod("*");
    config.addAllowedOrigin("http://127.0.0.1:3000");
    config.setAllowCredentials(true);
    source.registerCorsConfiguration("/**", config);
    return source;
}
```

我们定义一个CorsConfigurationSource实例，其中包含允许的来源、标头、方法和其他配置。请注意，在上面的配置中，我们使用IP地址127.0.0.1而不是localhost，因为后者是不允许的。最后，让我们定义一个UserDetailsService实例来在授权服务器中定义用户。

```java
@Bean
UserDetailsService userDetailsService() {
    PasswordEncoder passwordEncoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
    UserDetails userDetails = User.builder()
            .username("john")
            .password("password")
            .passwordEncoder(passwordEncoder::encode)
            .roles("USER")
            .build();

    return new InMemoryUserDetailsManager(userDetails);
}
```

通过以上配置，我们将能够使用用户名john和password作为密码来向授权服务器进行身份验证。

### 4.4 公共客户端应用程序

现在让我们讨论一下公共客户端，为了演示目的，我们使用一个简单的React应用程序作为单页应用程序。此应用程序使用[oidc-client-ts](https://github.com/authts/oidc-client-ts)库来提供客户端OIDC和OAuth2支持，SPA应用程序包含了以下配置：

```javascript
const pkceAuthConfig = {
    authority: 'http://127.0.0.1:9000/',
    client_id: 'public-client',
    redirect_uri: 'http://127.0.0.1:3000/callback',
    response_type: 'code',
    scope: 'openid profile email',
    post_logout_redirect_uri: 'http://127.0.0.1:3000/',
    userinfo_endpoint: 'http://127.0.0.1:9000/userinfo',
    response_mode: 'query',
    code_challenge_method: 'S256',
};

export default pkceAuthConfig;
```

authority配置了Spring授权服务器的地址，即http://127.0.0.1:9000。代码质询方法参数配置为S256。这些配置用于准备UserManager实例，稍后我们将使用它来调用授权服务器。此应用程序有两个端点-“/”用于访问应用程序的登录页面，以及处理来自授权服务器的回调请求的“callback”端点：

```javascript
import React, { useState, useEffect } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Login from './components/LoginHandler';
import CallbackHandler from './components/CallbackHandler';
import pkceAuthConfig from './pkceAuthConfig';
import { UserManager, WebStorageStateStore } from 'oidc-client-ts';

function App() {
    const [authenticated, setAuthenticated] = useState(null);
    const [userInfo, setUserInfo] = useState(null);

    const userManager = new UserManager({
        userStore: new WebStorageStateStore({ store: window.localStorage }),
        ...pkceAuthConfig,
    });

    function doAuthorize() {
        userManager.signinRedirect({state: '6c2a55953db34a86b876e9e40ac2a202',});
    }

    useEffect(() => {
        userManager.getUser().then((user) => {
            if (user) {
                setAuthenticated(true);
            }
            else {
                setAuthenticated(false);
            }
        });
    }, [userManager]);

    return (
        <BrowserRouter>
            <Routes>
                <Route path="/" element={<Login authentication={authenticated} handleLoginRequest={doAuthorize}/>}/>
                <Route path="/callback"
                       element={<CallbackHandler
                           authenticated={authenticated}
                           setAuth={setAuthenticated}
                           userManager={userManager}
                           userInfo={userInfo}
                           setUserInfo={setUserInfo}/>}/>
            </Routes>
        </BrowserRouter>
    );
}

export default App;
```

## 5. 测试

我们将使用启用了OIDC客户端支持的React应用程序来测试流程，要安装所需的依赖，我们需要从应用程序的根目录运行npm install命令。然后，我们将使用npm start命令启动该应用程序。

### 5.1 访问授权码授予应用程序

此客户端应用程序执行以下两个活动：首先，访问http://127.0.0.1:3000上的主页会呈现登录页面，这是我们的SPA应用程序的登录页面：接下来，一旦我们继续登录，SPA应用程序就会使用code_challenge和code_challenge_method调用Spring授权服务器：

![](/assets/images/2025/springsecurity/springauthenticationsinglepageapplicationpkce03.png)

我们可以注意到对Spring授权服务器http://127.0.0.1:9000发出的请求具有以下参数：

```text
http://127.0.0.1:9000/oauth2/authorize?
client_id=public-client&
redirect_uri=http%3A%2F%2F127.0.0.1%3A3000%2Fcallback&
response_type=code&
scope=openid+profile+email&
state=301b4ce8bdaf439990efd840bce1449b&
code_challenge=kjOAp0NLycB6pMChdB7nbL0oGG0IQ4664OwQYUegzF0&
code_challenge_method=S256&
response_mode=query
```

授权服务器将请求重定向到Spring Security登录页面：

![](/assets/images/2025/springsecurity/springauthenticationsinglepageapplicationpkce04.png)

一旦我们提供登录凭据，授权就会请求同意附加的Oauth范围配置文件和电子邮件，这是由于授权服务器中的配置require-authorization-consent为true：

![](/assets/images/2025/springsecurity/springauthenticationsinglepageapplicationpkce05.png)

### 5.2 使用授权码交换访问令牌

如果我们完成登录，授权服务器将返回授权码。随后，SPA向授权服务器请求另一个HTTP以获取访问令牌。**SPA提供上一个请求中获得的授权码以及code_challenge以获取access_token**：

![](/assets/images/2025/springsecurity/springauthenticationsinglepageapplicationpkce06.png)

对于上述请求，Spring授权服务器使用访问令牌进行响应：

![](/assets/images/2025/springsecurity/springauthenticationsinglepageapplicationpkce07.png)

接下来，我们访问授权服务器中的userinfo端点以访问用户详细信息。我们提供带有Authorization HTTP标头的access_token作为Bearer令牌来访问此端点，此用户信息从userinfo详细信息中打印出来：

![](/assets/images/2025/springsecurity/springauthenticationsinglepageapplicationpkce08.png)

## 6. 总结

在本文中，我们演示了如何在使用Spring授权服务器的单页应用程序中使用OAuth 2.0 PKCE扩展，我们从公共客户端对PKCE的需求开始讨论，并探讨了Spring授权服务器中使用PKCE流程的配置。最后，我们利用React应用程序来演示流程。