---
layout: post
title:  Spring Security – 攻击OAuth
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 简介

OAuth是委托授权的行业标准框架，在创建构成该标准的各种流程时，我们投入了大量心思和精力。即便如此，它也不是没有漏洞的。

在本系列文章中，我们将从理论的角度讨论针对OAuth的攻击，并描述现有的保护我们应用程序的各种选项。

## 2. 授权码授予

[授权码授予](https://oauth.net/2/grant-types/authorization-code/)流程是大多数实现委托授权的应用程序使用的默认流程。

在该流程开始之前，客户端必须已在授权服务器进行预先注册，并且在此过程中，还必须提供重定向URL-**即授权服务器可以使用授权码回调客户端的URL**。

让我们仔细看看它是如何工作的以及其中一些术语的含义。

在授权码授予流程中，客户端(请求委托授权的应用程序)将资源所有者(用户)重定向到授权服务器(例如，[使用Google登录](https://developers.google.com/identity/sign-in/web/sign-in))。登录后，授权服务器使用授权码重定向回客户端。

接下来，客户端调用授权服务器的端点，通过提供授权码来请求访问令牌。此时，流程结束，客户端可以使用令牌访问授权服务器保护的资源。

现在，**OAuth 2.0框架允许这些客户端公开**，例如在客户端无法安全保存客户端机密的情况下，让我们来看看一些可能针对公共客户端的重定向攻击。

## 3. 重定向攻击

### 3.1 攻击前提条件

重定向攻击依赖于这样一个事实：**OAuth标准没有完全描述必须指定此重定向URL的程度**，这是设计使然。

**这允许OAuth协议的某些实现允许部分重定向URL**。

例如，如果我们针对授权服务器注册一个客户端ID和一个客户端重定向URL，并使用以下基于通配符的匹配：

*.cloudapp.net

这适用于：

app.cloudapp.net

也适用于：

evil.cloudapp.net

我们特意选择了cloudapp.net域，因为这是一个可以托管OAuth支持的应用程序的真实位置。该域是[Microsoft的Windows Azure平台](https://docs.microsoft.com/en-us/archive/blogs/ptsblog/security-consideration-when-using-cloudapp-net-domain-as-production-environment-in-windows-azure)的一部分，允许任何开发人员在其下托管子域来测试应用程序。这本身不是问题，但它是更大漏洞的关键部分。

此漏洞的第二部分是允许在回调URL上进行通配符匹配的授权服务器。

最后，为了实现此漏洞，应用程序开发人员需要向授权服务器注册，以接受主域下的任何URL，格式为*.cloudapp.net。

### 3.2 攻击

当满足这些条件时，攻击者需要诱骗用户从其控制的子域启动页面，例如，向[用户发送一封看似真实的电子邮件](https://www.vadesecure.com/en/blog/5-common-phishing-techniques)，要求他对受OAuth保护的帐户执行某些操作。通常，这看起来像https://evil.cloudapp.net/login。当用户打开此链接并选择登录时，他将被重定向到授权服务器并发送授权请求：

```text
GET /authorize?response_type=code&amp;client_id={apps-client-id}&amp;state={state}&amp;redirect_uri=https%3A%2F%2Fevil.cloudapp.net%2Fcb HTTP/1.1
```

虽然这看起来很典型，但这个URL是恶意的。请看，在这种情况下，授权服务器会收到一个经过篡改的URL，其中包含应用程序的客户端ID和指向恶意应用程序的重定向URL。

然后，授权服务器将验证URL，该URL是指定主域下的子域。由于授权服务器认为请求来自有效来源，因此它将对用户进行身份验证，然后像平常一样征求同意。

完成后，它将重定向回evil.cloudapp.net子域，并将授权码交给攻击者。

由于攻击者现在拥有了授权码，他所需要做的就是使用授权码调用授权服务器的令牌端点来接收令牌，这允许他访问资源所有者的受保护资源。

## 4. Spring OAuth授权服务器漏洞评估

让我们看一个简单的Spring OAuth授权服务器配置：

```java
@Configuration
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    @Override
    public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
        clients.inMemory()
                .withClient("apricot-client-id")
                .authorizedGrantTypes("authorization_code")
                .scopes("scope1", "scope2")
                .redirectUris("https://app.cloudapp.net/oauth");
    }
    // ...
}
```

我们在这里可以看到授权服务器配置一个新客户端，其id为“apricot-client-id”。没有客户端密钥，因此这是一个公共客户端。

对此，我们的安全人员应该提高警惕，因为现在我们已经满足了三个条件中的两个-攻击者可以注册子域名，而且我们正在使用公共客户端。

但是，请注意，**我们也在这里配置了重定向URL**，并且它是绝对的。我们可以通过这样做来缓解漏洞。

### 4.1 严格

默认情况下，Spring OAuth允许重定向URL匹配具有一定程度的灵活性。

例如，DefaultRedirectResolver支持子域名匹配。

我们只使用我们需要的，如果我们可以精确匹配重定向URL，我们应该这样做：

```java
@Configuration
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
    // ...

    @Override
    public void configure(AuthorizationServerEndpointsConfigurer endpoints) {
        endpoints.redirectResolver(new ExactMatchRedirectResolver());
    }
}
```

在这种情况下，我们已切换到使用ExactMatchRedirectResolver来处理重定向URL。此解析器执行精确字符串匹配，而不会以任意方式解析重定向URL，**这使得其行为更加安全和确定**。

### 4.2 宽泛

我们可以在[Spring Security OAuth源](https://github.com/spring-projects/spring-security-oauth/blob/7bfe08d8f95b2fec035de484068f7907851b27d0/spring-security-oauth2/src/main/java/org/springframework/security/oauth2/provider/endpoint/DefaultRedirectResolver.java)中找到处理重定向URL匹配的默认代码：

```java
/**
 Whether the requested redirect URI "matches" the specified redirect URI. For a URL, this implementation tests if
 the user requested redirect starts with the registered redirect, so it would have the same host and root path if
 it is an HTTP URL. The port, userinfo, query params also matched. Request redirect uri path can include
 additional parameters which are ignored for the match
 <p>
 For other (non-URL) cases, such as for some implicit clients, the redirect_uri must be an exact match.
 @param requestedRedirect The requested redirect URI.
 @param redirectUri The registered redirect URI.
 @return Whether the requested redirect URI "matches" the specified redirect URI.
 */
protected boolean redirectMatches(String requestedRedirect, String redirectUri) {
    UriComponents requestedRedirectUri = UriComponentsBuilder.fromUriString(requestedRedirect).build();
    UriComponents registeredRedirectUri = UriComponentsBuilder.fromUriString(redirectUri).build();
    boolean schemeMatch = isEqual(registeredRedirectUri.getScheme(), requestedRedirectUri.getScheme());
    boolean userInfoMatch = isEqual(registeredRedirectUri.getUserInfo(), requestedRedirectUri.getUserInfo());
    boolean hostMatch = hostMatches(registeredRedirectUri.getHost(), requestedRedirectUri.getHost());
    boolean portMatch = matchPorts ? registeredRedirectUri.getPort() == requestedRedirectUri.getPort() : true;
    boolean pathMatch = isEqual(registeredRedirectUri.getPath(),
            StringUtils.cleanPath(requestedRedirectUri.getPath()));
    boolean queryParamMatch = matchQueryParams(registeredRedirectUri.getQueryParams(),
            requestedRedirectUri.getQueryParams());

    return schemeMatch && userInfoMatch && hostMatch && portMatch && pathMatch && queryParamMatch;
}
```

我们可以看到，URL匹配是通过将传入的重定向URL解析为其组成部分来完成的。这非常复杂，因为它有几个特性，**例如端口、子域和查询参数是否应该匹配**。选择允许子域匹配是需要三思而行的。

当然，如果我们需要的话，这种灵活性是存在的-我们只要谨慎使用它即可。

## 5. 隐式流量重定向攻击

需要明确的是，[不推荐使用](https://tools.ietf.org/html/rfc8252#section-8.2)[隐式流程](https://oauth.net/2/grant-types/implicit/)，最好使用授权码授予流程，并由[PKCE](https://tools.ietf.org/html/rfc7636)提供额外的安全性。话虽如此，让我们看看重定向攻击是如何通过隐式流程表现出来的。

针对隐式流的重定向攻击将遵循与我们上面看到的相同的基本步骤，主要区别在于攻击者会立即获得令牌，因为没有授权码交换步骤。

与以前一样，重定向URL的绝对匹配也会减轻此类攻击。

此外，我们还可以发现隐式流还包含另一个相关漏洞，**攻击者可以使用客户端作为开放的重定向器，并让其重新附加片段**。

攻击的开始方式与之前类似，攻击者诱使用户访问攻击者控制的页面，例如https://evil.cloudapp.net/info。该页面旨在像之前一样发起授权请求，但是，它现在包含一个重定向URL：

```text
GET /authorize?response_type=token&client_id=ABCD&state=xyz&redirect_uri=https%3A%2F%2Fapp.cloudapp.net%2Fcb%26redirect_to
%253Dhttps%253A%252F%252Fevil.cloudapp.net%252Fcb HTTP/1.1
```

redirect_to https://evil.cloudapp.net设置授权端点，以将令牌重定向到攻击者控制的域，授权服务器现在将首先重定向到实际的应用程序站点：

```text
Location: https://app.cloudapp.net/cb?redirect_to%3Dhttps%3A%2F%2Fevil.cloudapp.net%2Fcb#access_token=LdKgJIfEWR34aslkf&...
```

当此请求到达开放的重定向器时，它将提取重定向URL evil.cloudapp.net，然后重定向到攻击者的站点：

```text
https://evil.cloudapp.net/cb#access_token=LdKgJIfEWR34aslkf&...
```

绝对URL匹配也会减轻这种攻击。

## 6. 总结

在本文中，我们讨论了一类基于重定向URL的针对OAuth协议的攻击。

虽然这可能会产生严重后果，但在授权服务器上使用绝对URL匹配可以减轻此类攻击。