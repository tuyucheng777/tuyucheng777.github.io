---
layout: post
title:  Spring REST API OAuth2 – 在Angular中处理刷新令牌(旧OAuth堆栈)
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将继续探索在[上一篇文章](https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)中开始整理的OAuth密码流，并重点介绍如何在AngularJS应用程序中处理刷新令牌。

注意：本文使用的是[Spring OAuth旧项目](https://spring.io/projects/spring-authorization-server)，**对于使用新Spring Security 5堆栈的版本，请参阅我们的文章[Spring REST API OAuth2 – 在Angular中处理刷新令牌](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular)**。

## 2. 访问令牌过期

首先，请记住，当用户登录应用程序时，客户端需要获取访问令牌：

```javascript
function obtainAccessToken(params) {
    var req = {
        method: 'POST',
        url: "oauth/token",
        headers: {"Content-type": "application/x-www-form-urlencoded; charset=utf-8"},
        data: $httpParamSerializer(params)
    }
    $http(req).then(
        function(data) {
            $http.defaults.headers.common.Authorization= 'Bearer ' + data.data.access_token;
            var expireDate = new Date (new Date().getTime() + (1000 * data.data.expires_in));
            $cookies.put("access_token", data.data.access_token, {'expires': expireDate});
            window.location.href="index";
        },function() {
            console.log("error");
            window.location.href = "login";
        });
}
```

请注意我们的访问令牌是如何存储在Cookie中的，该令牌的过期时间取决于令牌本身的过期时间。

需要了解的是，**Cookie本身仅用于存储**，不会在OAuth流程中驱动任何其他操作。例如，浏览器永远不会自动将Cookie随请求发送到服务器。

还请注意我们实际上是如何调用这个obtainAccessToken()函数的：

```javascript
$scope.loginData = {
    grant_type:"password", 
    username: "", 
    password: "", 
    client_id: "fooClientIdPassword"
};

$scope.login = function() {   
    obtainAccessToken($scope.loginData);
}
```

## 3. 代理

我们现在将在前端应用程序中运行一个Zuul代理，它基本上位于前端客户端和授权服务器之间。

让我们配置代理的路由：

```yaml
zuul:
    routes:
        oauth:
            path: /oauth/**
            url: http://localhost:8081/spring-security-oauth-server/oauth
```

有趣的是，我们只代理授权服务器的流量，而不代理其他流量。我们真正需要的只是在客户端获取新令牌时才使用代理。

如果你想了解Zuul的基础知识，请快速阅读[主要Zuul文章](https://www.baeldung.com/spring-rest-with-zuul-proxy)。

## 4. 执行基本身份验证的Zuul过滤器

代理的第一次使用很简单，我们不用在JavaScript中透露我们的应用程序“client secret”，而是使用Zuul预过滤器添加Authorization标头来访问令牌请求：

```java
@Component
public class CustomPreZuulFilter extends ZuulFilter {
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        if (ctx.getRequest().getRequestURI().contains("oauth/token")) {
            byte[] encoded;
            try {
                encoded = Base64.encode("fooClientIdPassword:secret".getBytes("UTF-8"));
                ctx.addZuulRequestHeader("Authorization", "Basic " + new String(encoded));
            } catch (UnsupportedEncodingException e) {
                logger.error("Error occured in pre filter", e);
            }
        }
        return null;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public int filterOrder() {
        return -2;
    }

    @Override
    public String filterType() {
        return "pre";
    }
}
```

现在请记住，这不会增加任何额外的安全性，我们这样做的唯一原因是因为令牌端点使用客户端凭据通过基本身份验证进行保护。

从实现的角度来看，过滤器的类型尤其值得注意，我们使用“pre”类型的过滤器在传递请求之前对其进行处理。

## 5. 将刷新令牌放入Cookie中

我们计划让客户端以Cookie的形式获取刷新令牌，这不只是一个普通的Cookie，而是一个安全、仅HTTP使用的Cookie，并且路径非常有限(/oauth/token)。

我们将设置一个Zuul后置过滤器，从响应的JSON主体中提取刷新令牌并将其设置在Cookie中：

```java
@Component
public class CustomPostZuulFilter extends ZuulFilter {
    private ObjectMapper mapper = new ObjectMapper();

    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        try {
            InputStream is = ctx.getResponseDataStream();
            String responseBody = IOUtils.toString(is, "UTF-8");
            if (responseBody.contains("refresh_token")) {
                Map<String, Object> responseMap = mapper.readValue(
                        responseBody, new TypeReference<Map<String, Object>>() {});
                String refreshToken = responseMap.get("refresh_token").toString();
                responseMap.remove("refresh_token");
                responseBody = mapper.writeValueAsString(responseMap);

                Cookie cookie = new Cookie("refreshToken", refreshToken);
                cookie.setHttpOnly(true);
                cookie.setSecure(true);
                cookie.setPath(ctx.getRequest().getContextPath() + "/oauth/token");
                cookie.setMaxAge(2592000); // 30 days
                ctx.getResponse().addCookie(cookie);
            }
            ctx.setResponseBody(responseBody);
        } catch (IOException e) {
            logger.error("Error occured in zuul post filter", e);
        }
        return null;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public int filterOrder() {
        return 10;
    }

    @Override
    public String filterType() {
        return "post";
    }
}
```

这里需要了解一些有趣的事情：

- 我们使用Zuul后置过滤器来读取响应并**提取刷新令牌**
- 我们从JSON响应中删除了refresh_token的值，以确保前端永远无法通过Cookie访问它
- 我们将Cookie的最大期限设置为**30天**-因为这与令牌的过期时间相匹配

为了增加一层额外的保护来抵御CSRF攻击，**我们将在所有Cookie中添加[Same-Site cookie](https://tools.ietf.org/html/draft-ietf-httpbis-cookie-same-site-00)标头**。

为此，我们将创建一个配置类：

```java
@Configuration
public class SameSiteConfig implements WebMvcConfigurer {
    @Bean
    public TomcatContextCustomizer sameSiteCookiesConfig() {
        return context -> {
            final Rfc6265CookieProcessor cookieProcessor = new Rfc6265CookieProcessor();
            cookieProcessor.setSameSiteCookies(SameSiteCookies.STRICT.getValue());
            context.setCookieProcessor(cookieProcessor);
        };
    }
}
```

这里我们将属性设置为strict，以便严格禁止任何跨站点的Cookie传输。

## 6. 从Cookie中获取并使用刷新令牌

现在我们在Cookie中有了刷新令牌，当前端AngularJS应用程序尝试触发令牌刷新时，它将在/oauth/token发送请求，因此浏览器当然会发送该Cookie。

因此我们现在在代理中有另一个过滤器，它将从Cookie中提取刷新令牌并将其作为HTTP参数转发，以便请求有效：

```java
public Object run() {
    RequestContext ctx = RequestContext.getCurrentContext();
    // ...
    HttpServletRequest req = ctx.getRequest();
    String refreshToken = extractRefreshToken(req);
    if (refreshToken != null) {
        Map<String, String[]> param = new HashMap<String, String[]>();
        param.put("refresh_token", new String[] { refreshToken });
        param.put("grant_type", new String[] { "refresh_token" });
        ctx.setRequest(new CustomHttpServletRequest(req, param));
    }
    // ...
}

private String extractRefreshToken(HttpServletRequest req) {
    Cookie[] cookies = req.getCookies();
    if (cookies != null) {
        for (int i = 0; i < cookies.length; i++) {
            if (cookies[i].getName().equalsIgnoreCase("refreshToken")) {
                return cookies[i].getValue();
            }
        }
    }
    return null;
}
```

这是我们的CustomHttpServletRequest，用于**注入刷新令牌参数**：

```java
public class CustomHttpServletRequest extends HttpServletRequestWrapper {
    private Map<String, String[]> additionalParams;
    private HttpServletRequest request;

    public CustomHttpServletRequest(HttpServletRequest request, Map<String, String[]> additionalParams) {
        super(request);
        this.request = request;
        this.additionalParams = additionalParams;
    }

    @Override
    public Map<String, String[]> getParameterMap() {
        Map<String, String[]> map = request.getParameterMap();
        Map<String, String[]> param = new HashMap<String, String[]>();
        param.putAll(map);
        param.putAll(additionalParams);
        return param;
    }
}
```

再次强调，这里有很多重要的实现说明：

- 代理从Cookie中提取刷新令牌
- 然后将其设置到refresh_token参数中
- 它还将grant_type设置为refresh_token
- 如果没有refreshToken Cookie(已过期或首次登录)-则访问令牌请求将被重定向而不进行任何更改

## 7. 从AngularJS刷新访问令牌

最后，让我们修改一下简单的前端应用程序，并实际使用刷新令牌：

这是我们的函数refreshAccessToken()：

```javascript
$scope.refreshAccessToken = function() {
    obtainAccessToken($scope.refreshData);
}
```

这是我们的$scope.refreshData：

```javascript
$scope.refreshData = {grant_type:"refresh_token"};
```

请注意我们只是使用现有的obtainAccessToken函数，并向其传递不同的输入。

还要注意，我们自己不会添加refresh_token–因为这将由Zuul过滤器处理。

## 8. 总结

在本OAuth教程中，我们学习了如何在AngularJS客户端应用程序中存储刷新令牌、如何刷新过期的访问令牌以及如何利用Zuul代理完成所有这些操作。