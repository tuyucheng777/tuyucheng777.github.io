---
layout: post
title:  在OAuth保护的应用程序中注销
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

在此快速教程中，我们将展示如何**向OAuth Spring Security应用程序添加注销功能**。

我们将看到几种方法。首先，我们将看到如何从OAuth应用程序中注销Keycloak用户，如[使用OAuth2创建REST API](https://www.baeldung.com/rest-api-spring-oauth2-angular)中所述，然后使用我们之前看到的[Zuul代理](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular#zuul)。

**我们将在Spring Security 5中使用OAuth堆栈**，如果你想使用Spring Security OAuth旧堆栈，请查看之前的文章：[在OAuth安全应用程序中注销(使用旧堆栈)](https://www.baeldung.com/logout-spring-security-oauth-legacy)。

## 2. 使用前端应用程序注销

**由于访问令牌由授权服务器管理，因此需要在此级别使它们失效**，具体步骤将根据你使用的授权服务器略有不同。

在我们的示例中，根据Keycloak[文档](https://www.keycloak.org/docs/25.0.6/securing_apps/#logout)，要从浏览器应用程序直接从注销，我们可以将浏览器重定向到http://auth-server/auth/realms/{realm-name}/protocol/openid-connect/logout?redirect_uri=encodedRedirectUri。

**除了发送重定向URI之外，我们还需要将id_token_hint传递给Keycloak的[Logout端点](https://www.keycloak.org/docs-api/latest/javadocs/org/keycloak/protocol/oidc/endpoints/LogoutEndpoint.html)**，这应该携带编码的id_token值。

让我们回想一下我们如何保存access_token，我们也将类似地保存id_token：

```javascript
saveToken(token) {
    var expireDate = new Date().getTime() + (1000 * token.expires_in);
    Cookie.set("access_token", token.access_token, expireDate);
    Cookie.set("id_token", token.id_token, expireDate);
    this._router.navigate(['/']);
}
```

**重要的是，为了在授权服务器的响应负载中获取[ID令牌](https://www.oauth.com/oauth2-servers/openid-connect/id-tokens/)，我们应该在[范围参数](https://www.baeldung.com/rest-api-spring-oauth2-angular#app-service)中包含openid**。

现在让我们看看注销过程的具体过程。

我们将修改[应用服务](https://www.baeldung.com/rest-api-spring-oauth2-angular#app-service-1)中的logout函数：

```javascript
logout() {
    let token = Cookie.get('id_token');
    Cookie.delete('access_token');
    Cookie.delete('id_token');
    let logoutURL = "http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/logout?id_token_hint=" + token + "&post_logout_redirect_uri=" + this.redirectUri;

    window.location.href = logoutURL;
}
```

除了重定向之外，我们还需要丢弃从授权服务器获得的访问令牌和ID令牌。

因此，在上面的代码中，我们首先删除令牌，然后将浏览器重定向到Keycloak的logout API。

值得注意的是，我们传递的重定向URI是http://localhost:8089/(我们在整个应用程序中使用的URI)，因此退出后我们将进入登录页面。

当前会话对应的访问令牌、ID令牌和刷新令牌的删除是在授权服务器端进行的，在这种情况下，我们的浏览器应用程序根本没有保存刷新令牌。

## 3. 使用Zuul代理注销

在上一篇关于[处理刷新令牌](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular)的文章中，我们已将应用程序设置为能够使用刷新令牌刷新访问令牌，此实现利用了带有自定义过滤器的Zuul代理。

在这里我们将看到如何向上面添加注销功能。

这次，我们将使用另一个Keycloak API来注销用户。**我们将在logout端点上调用POST以通过非浏览器调用注销会话**，而不是我们在上一节中使用的URL重定向。

### 3.1 定义注销路由

首先，让我们在[application.yml](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular#zuul)中添加另一个到代理的路由：

```yaml
zuul:
    routes:
        //...
        auth/refresh/revoke:
            path: /auth/refresh/revoke/**
            sensitiveHeaders:
            url: http://localhost:8083/auth/realms/baeldung/protocol/openid-connect/logout
        
        //auth/refresh route
```

实际上，我们在已经存在的auth/refresh上添加了一个子路由，**在主路由之前添加子路由非常重要，否则，Zuul将始终映射主路由的URL**。

我们添加了子路由而不是主路由，以便能够访问仅HTTP的refreshToken Cookie，[该Cookie被设置为具有非常有限的路径](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular#extractToken)，如/auth/refresh(及其子路径)。我们将在下一节中看到为什么我们需要该Cookie。

### 3.2 POST到授权服务器的/logout

现在让我们增强CustomPreZuulFilter实现以拦截/auth/refresh/revoke URL并添加要传递给授权服务器的必要信息。

**注销所需的表单参数与[刷新令牌](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular#injectToken)请求的参数类似，只是没有grant_type**：

```java
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    //... 
    @Override
    public Object run() {
        //...
        if (requestURI.contains("auth/refresh/revoke")) {
            String cookieValue = extractCookie(req, "refreshToken");
            String formParams = String.format("client_id=%s&client_secret=%s&refresh_token=%s",
                    CLIENT_ID, CLIENT_SECRET, cookieValue);
            bytes = formParams.getBytes("UTF-8");
        }
        //...
    }
}
```

在这里，我们只是提取了refreshToken Cookie并发送了所需的formParams。

### 3.3 删除刷新令牌

**正如我们之前看到的，当使用logout重定向撤销访问令牌时，与其关联的刷新令牌也会被授权服务器失效**。

但是，在这种情况下，httpOnly Cookie仍会保留在客户端上。鉴于我们无法通过JavaScript将其删除，我们需要从服务器端将其删除。

为此，让我们添加到拦截/auth/refresh/revoke URL的CustomPostZuulFilter实现中，以便在遇到此URL时删除refreshToken Cookie：

```java
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    //...
    @Override
    public Object run() {
        //...
        String requestMethod = ctx.getRequest().getMethod();
        if (requestURI.contains("auth/refresh/revoke")) {
            Cookie cookie = new Cookie("refreshToken", "");
            cookie.setMaxAge(0);
            ctx.getResponse().addCookie(cookie);
        }
        //...
    }
}
```

### 3.4 从Angular客户端中删除访问令牌

除了撤销刷新令牌之外，还需要从客户端删除access_token Cookie。

让我们向Angular控制器添加一个方法，清除access_token Cookie并调用/auth/refresh/revoke POST映射：

```javascript
logout() {
    let headers = new HttpHeaders({'Content-type': 'application/x-www-form-urlencoded; charset=utf-8'});

    this._http.post('auth/refresh/revoke', {}, { headers: headers })
        .subscribe(
            data => {
                Cookie.delete('access_token');
                window.location.href = 'http://localhost:8089/';
            },
            err => alert('Could not logout')
        );
}
```

单击“logout”按钮时将调用此函数：

```javascript
<a class="btn btn-default pull-right"(click)="logout()" href="#">Logout</a>
```

## 4. 总结

在这个快速但深入的教程中，我们展示了如何从OAuth安全应用程序中注销用户并使该用户的令牌无效。