---
layout: post
title:  在OAuth安全应用程序中注销(使用Spring Security OAuth遗留堆栈)
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在此快速教程中，我们将展示如何**向OAuth Spring Security应用程序添加注销功能**。

当然，我们将使用上一篇文章中描述的OAuth应用程序 –[使用OAuth2创建REST API](https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)。

注意：本文使用的是[Spring OAuth旧项目](https://spring.io/projects/spring-authorization-server)，对于使用新Spring Security 5堆栈的版本，请参阅我们的文章[在OAuth安全应用程序中注销](https://www.baeldung.com/logout-spring-security-oauth)。

## 2. 删除访问令牌

简而言之，在OAuth安全的环境中注销会导致用户的访问令牌无效-因此它不再可用。

在基于JdbcTokenStore的实现中，这意味着从TokenStore中删除令牌。

让我们为令牌实现一个删除操作，我们将在这里使用普通的/oauth/token URL结构，并简单地为其引入一个新的DELETE操作。

现在，因为我们实际上在这里使用/oauth/token URI–我们需要小心处理它。我们不能简单地将其添加到任何控制器–因为框架已经将操作映射到该URI–使用POST和GET。

相反，我们需要做的是将其定义为@FrameworkEndpoint，以便它被FrameworkEndpointHandlerMapping而不是标准RequestMappingHandlerMapping拾取和解析，这样我们就不会遇到任何部分匹配，也不会有任何冲突：

```java
@FrameworkEndpoint
public class RevokeTokenEndpoint {

    @Resource(name = "tokenServices")
    ConsumerTokenServices tokenServices;

    @RequestMapping(method = RequestMethod.DELETE, value = "/oauth/token")
    @ResponseBody
    public void revokeToken(HttpServletRequest request) {
        String authorization = request.getHeader("Authorization");
        if (authorization != null && authorization.contains("Bearer")){
            String tokenId = authorization.substring("Bearer".length()+1);
            tokenServices.revokeToken(tokenId);
        }
    }
}
```

请注意我们如何从请求中提取令牌，只需使用标准Authorization标头即可。

## 3. 删除刷新令牌

在上一篇关于[处理刷新令牌](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular-js-legacy)的文章中，我们已将应用程序设置为能够使用刷新令牌刷新访问令牌。此实现利用Zuul代理-使用CustomPostZuulFilter将从授权服务器收到的refresh_token值添加到refreshToken Cookie。

如上一节所示，撤销访问令牌时，与其关联的刷新令牌也会失效。但是，httpOnly Cookie将保留在客户端上，因为我们无法通过JavaScript将其删除-因此我们需要从服务器端将其删除。

让我们增强拦截/oauth/token/revoke URL的CustomPostZuulFilter实现，以便它在遇到此 UR时删除refreshToken Cookie：

```java
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    //...
    @Override
    public Object run() {
        //...
        String requestMethod = ctx.getRequest().getMethod();
        if (requestURI.contains("oauth/token") && requestMethod.equals("DELETE")) {
            Cookie cookie = new Cookie("refreshToken", "");
            cookie.setMaxAge(0);
            cookie.setPath(ctx.getRequest().getContextPath() + "/oauth/token");
            ctx.getResponse().addCookie(cookie);
        }
        //...
    }
}
```

## 4. 从AngularJS客户端中删除访问令牌

除了从令牌存储中撤销访问令牌之外，还需要从客户端删除access_token Cookie。

让我们向AngularJS控制器添加一个方法，清除access_token Cookie并调用/oauth/token/revoke DELETE映射：

```javascript
$scope.logout = function() {
    logout($scope.loginData);
}
function logout(params) {
    var req = {
        method: 'DELETE',
        url: "oauth/token"
    }
    $http(req).then(
        function(data){
            $cookies.remove("access_token");
            window.location.href="login";
        },function(){
            console.log("error");
        }
    );
}
```

单击“logout”链接时将调用此函数：

```html
<a class="btn btn-info" href="#" ng-click="logout()">Logout</a>
```

## 5. 总结

在这个快速但深入的教程中，我们展示了如何从OAuth安全应用程序中注销用户并使该用户的令牌无效。