---
layout: post
title:  OAuth2使用刷新令牌记住我(使用Spring Security OAuth遗留堆栈)
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本文中，我们将利用OAuth 2刷新令牌向OAuth 2安全应用程序添加“记住我”功能。

本文是我们使用OAuth 2保护Spring REST API(可通过AngularJS客户端访问)系列文章的续篇，要设置授权服务器、资源服务器和前端客户端，你可以按照[介绍性文章](https://www.baeldung.com/rest-api-spring-oauth2-angular-legacy)进行操作。

注意：本文使用的是[Spring OAuth遗留项目](https://spring.io/projects/spring-authorization-server)。

## 2. OAuth 2访问令牌和刷新令牌

首先，让我们快速回顾一下OAuth 2令牌及其使用方法。

首次使用password授权类型进行身份验证时，用户需要发送有效的用户名和密码以及客户端ID和密钥。如果身份验证请求成功，服务器将返回以下形式的响应：

```json
{
    "access_token": "2e17505e-1c34-4ea6-a901-40e49ba786fa",
    "token_type": "bearer",
    "refresh_token": "e5f19364-862d-4212-ad14-9d6275ab1a62",
    "expires_in": 59,
    "scope": "read write"
}
```

我们可以看到服务器响应包含访问令牌和[刷新令牌](https://www.baeldung.com/cs/json-web-token-refresh-token)，访问令牌将用于需要身份验证的后续API调用，**而刷新令牌的目的是获取新的有效访问令牌或仅撤销前一个访问令牌**。

要使用refresh_token授予类型接收新的访问令牌，用户不再需要输入他们的凭据，而只需要输入客户端ID、密钥，当然还有刷新令牌。

**使用两种类型的令牌的目的是为了增强用户安全性**。通常，访问令牌的有效期较短，因此如果攻击者获得访问令牌，他们只能在有限的时间内使用它。另一方面，如果刷新令牌被盗用，则这毫无用处，因为还需要客户端ID和密钥。

刷新令牌的另一个好处是，它允许撤销访问令牌，并且如果用户表现出异常行为(例如从新IP登录)，则不会发回另一个令牌。

## 3. 使用刷新令牌实现“记住我”功能

用户通常发现选择保留他们的会话很有用，因为他们不需要在每次访问应用程序时都输入他们的凭据。

由于访问令牌的有效期较短，我们可以使用刷新令牌来生成新的访问令牌，并避免每次访问令牌过期时都必须向用户询问其凭据。

在接下来的部分中，我们将讨论实现此功能的两种方法：

- 首先，拦截任何返回401状态码(表示访问令牌无效)的用户请求。发生这种情况时，如果用户选中了“记住我”选项，我们将使用refresh_token授权类型自动发出新访问令牌的请求，然后再次执行初始请求。
- 其次，我们可以主动刷新访问令牌-我们会在令牌过期前几秒发送刷新请求。

第二种选择的优点是用户的请求不会被延迟。

## 4. 存储刷新令牌

在上一篇关于[刷新令牌](https://www.baeldung.com/spring-security-oauth2-refresh-token-angular-js-legacy)的文章中，我们添加了一个CustomPostZuulFilter，它拦截对OAuth服务器的请求，提取身份验证时发回的刷新令牌，并将其存储在服务器端Cookie中：

```java
@Component
public class CustomPostZuulFilter extends ZuulFilter {

    @Override
    public Object run() {
        // ...
        Cookie cookie = new Cookie("refreshToken", refreshToken);
        cookie.setHttpOnly(true);
        cookie.setPath(ctx.getRequest().getContextPath() + "/oauth/token");
        cookie.setMaxAge(2592000); // 30 days
        ctx.getResponse().addCookie(cookie);
        // ...
    }
}
```

接下来，让我们在登录表单上添加一个复选框，该复选框具有与loginData.remember变量的数据绑定：

```html
<input type="checkbox"  ng-model="loginData.remember" id="remember"/>
<label for="remember">Remeber me</label>
```

我们的登录表单现在将显示一个额外的复选框：

![](/assets/images/2025/springsecurity/springsecurityoauth2rememberme01.png)

loginData对象随身份验证请求一起发送，因此它将包含Remember参数。在发送身份验证请求之前，我们将根据该参数设置一个名为remember的Cookie：

```javascript
function obtainAccessToken(params){
    if (params.username != null){
        if (params.remember != null){
            $cookies.put("remember","yes");
        }
        else {
            $cookies.remove("remember");
        }
    }
    //...
}
```

因此，我们将检查此Cookie以确定是否应该尝试刷新访问令牌，这取决于用户是否希望被记住。

## 5. 通过拦截401响应来刷新令牌

为了拦截返回401响应的请求，让我们修改AngularJS应用程序以添加一个带有responseError函数的拦截器：

```javascript
app.factory('rememberMeInterceptor', ['$q', '$injector', '$httpParamSerializer',
    function($q, $injector, $httpParamSerializer) {
        var interceptor = {
            responseError: function(response) {
                if (response.status == 401){

                    // refresh access token

                    // make the backend call again and chain the request
                    return deferred.promise.then(function() {
                        return $http(response.config);
                    });
                }
                return $q.reject(response);
            }
        };
        return interceptor;
    }]);
```

我们的函数检查状态是否为401(这意味着访问令牌无效)，如果是，则尝试使用刷新令牌来获取新的有效访问令牌。

如果成功，该函数将继续重试导致401错误的初始请求，这可确保为用户提供无缝体验。

让我们仔细看看刷新访问令牌的过程。首先，我们将初始化必要的变量：

```javascript
var $http = $injector.get('$http');
var $cookies = $injector.get('$cookies');
var deferred = $q.defer();

var refreshData = {grant_type:"refresh_token"};
                
var req = {
    method: 'POST',
    url: "oauth/token",
    headers: {"Content-type": "application/x-www-form-urlencoded; charset=utf-8"},
    data: $httpParamSerializer(refreshData)
}
```

你可以看到req变量，我们将使用该变量向/oauth/token端点发送POST请求，并附带参数grant_type=refresh_token。

接下来，让我们使用已注入的$http模块发送请求。如果请求成功，我们将使用新的访问令牌值设置新的Authentication标头，以及为access_token cookie设置新值。如果请求失败(如果刷新令牌最终也过期，则可能会发生这种情况)，则用户将被重定向到登录页面：

```javascript
$http(req).then(
    function(data){
        $http.defaults.headers.common.Authorization= 'Bearer ' + data.data.access_token;
        var expireDate = new Date (new Date().getTime() + (1000 * data.data.expires_in));
        $cookies.put("access_token", data.data.access_token, {'expires': expireDate});
        window.location.href="index";
    },function(){
        console.log("error");
        $cookies.remove("access_token");
        window.location.href = "login";
    }
);
```

Refresh Token由我们在上一篇文章中实现的CustomPreZuulFilter添加到请求中：

```java
@Component
public class CustomPreZuulFilter extends ZuulFilter {

    @Override
    public Object run() {
        //...
        String refreshToken = extractRefreshToken(req);
        if (refreshToken != null) {
            Map<String, String[]> param = new HashMap<String, String[]>();
            param.put("refresh_token", new String[] { refreshToken });
            param.put("grant_type", new String[] { "refresh_token" });

            ctx.setRequest(new CustomHttpServletRequest(req, param));
        }
        //...
    }
}
```

除了定义拦截器之外，我们还需要将其注册到$httpProvider：

```javascript
app.config(['$httpProvider', function($httpProvider) {
    $httpProvider.interceptors.push('rememberMeInterceptor');
}]);
```

## 6. 主动刷新令牌

实现“记住我”功能的另一种方法是在当前访问令牌过期之前请求新的访问令牌。

接收访问令牌时，JSON响应包含一个expires_in值，该值指定令牌有效的秒数。

让我们在每次身份验证时将该值保存在Cookie中：

```javascript
$cookies.put("validity", data.data.expires_in);
```

然后，为了发送刷新请求，让我们使用AngularJS$timeout服务在令牌过期前10秒安排刷新调用：

```javascript
if ($cookies.get("remember") == "yes"){
    var validity = $cookies.get("validity");
    if (validity >10) validity -= 10;
    $timeout( function(){ $scope.refreshAccessToken(); }, validity * 1000);
}
```

## 7. 总结

在本教程中，我们探讨了使用OAuth2应用程序和AngularJS前端实现“记住我”功能的两种方法。