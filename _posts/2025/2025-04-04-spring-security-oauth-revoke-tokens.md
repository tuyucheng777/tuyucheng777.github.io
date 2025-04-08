---
layout: post
title:  Spring Security OAuth2 – 简单令牌撤销(使用Spring Security OAuth遗留堆栈)
category: springsecurity
copyright: springsecurity
excerpt: Spring Security OAuth
---

## 1. 概述

在此快速教程中，我们将说明如何撤销使用Spring Security实现的OAuth授权服务器授予的令牌。

当用户注销时，他们的令牌不会立即从令牌存储中删除；相反，它仍然有效，直到其自行过期。

因此，撤销令牌意味着从令牌存储中删除该令牌。我们将介绍框架中的标准令牌实现，而不是JWT令牌。

注意：本文使用的是[Spring OAuth遗留项目](https://spring.io/projects/spring-authorization-server)。

## 2. TokenStore

首先，让我们设置令牌存储；我们将使用JdbcTokenStore以及随附的数据源：

```java
@Bean
public TokenStore tokenStore() {
    return new JdbcTokenStore(dataSource());
}

@Bean
public DataSource dataSource() {
    DriverManagerDataSource dataSource =  new DriverManagerDataSource();
    dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
    dataSource.setUrl(env.getProperty("jdbc.url"));
    dataSource.setUsername(env.getProperty("jdbc.user"));
    dataSource.setPassword(env.getProperty("jdbc.pass"));
    return dataSource;
}
```

## 3. DefaultTokenServices Bean

处理所有令牌的类是DefaultTokenServices，并且必须在我们的配置中定义为一个Bean：

```java
@Bean
@Primary
public DefaultTokenServices tokenServices() {
    DefaultTokenServices defaultTokenServices = new DefaultTokenServices();
    defaultTokenServices.setTokenStore(tokenStore());
    defaultTokenServices.setSupportRefreshToken(true);
    return defaultTokenServices;
}
```

## 4. 显示令牌列表

为了管理目的，我们还设置一种查看当前有效令牌的方法。

我们将在控制器中访问TokenStore，并检索指定客户端ID的当前存储的令牌：

```java
@Resource(name="tokenStore")
TokenStore tokenStore;

@RequestMapping(method = RequestMethod.GET, value = "/tokens")
@ResponseBody
public List<String> getTokens() {
    List<String> tokenValues = new ArrayList<String>();
    Collection<OAuth2AccessToken> tokens = tokenStore.findTokensByClientId("sampleClientId");
    if (tokens!=null){
        for (OAuth2AccessToken token:tokens){
            tokenValues.add(token.getValue());
        }
    }
    return tokenValues;
}
```

## 5. 撤销访问令牌

为了使令牌无效，我们将使用ConsumerTokenServices接口中的revokeToken() API：

```java
@Resource(name="tokenServices")
ConsumerTokenServices tokenServices;

@RequestMapping(method = RequestMethod.POST, value = "/tokens/revoke/{tokenId:.*}")
@ResponseBody
public String revokeToken(@PathVariable String tokenId) {
    tokenServices.revokeToken(tokenId);
    return tokenId;
}
```

当然，这是一个非常敏感的操作，所以我们要么只在内部使用它，要么我们应该非常小心地在适当的安全措施下公开它。

## 6. 前端

对于我们示例的前端，我们将显示有效令牌列表、发出撤销请求的登录用户当前使用的令牌以及用户可以输入他们希望撤销的令牌的字段：

```javascript
$scope.revokeToken = 
  $resource("http://localhost:8082/spring-security-oauth-resource/tokens/revoke/:tokenId",
  {tokenId:'@tokenId'});
$scope.tokens = $resource("http://localhost:8082/spring-security-oauth-resource/tokens");
    
$scope.getTokens = function(){
    $scope.tokenList = $scope.tokens.query();	
}
	
$scope.revokeAccessToken = function(){
    if ($scope.tokenToRevoke && $scope.tokenToRevoke.length !=0){
        $scope.revokeToken.save({tokenId:$scope.tokenToRevoke});
        $rootScope.message="Token:"+$scope.tokenToRevoke+" was revoked!";
        $scope.tokenToRevoke="";
    }
}
```

如果用户尝试再次使用已撤销的令牌，他们将收到状态码为401的“invalid token”错误。

## 7. 撤销刷新令牌

刷新令牌可用于获取新的访问令牌，每当访问令牌被撤销时，随其收到的刷新令牌将失效。

如果我们想使刷新令牌本身也无效，我们可以使用JdbcTokenStore类的removeRefreshToken()方法，它将从存储中删除刷新令牌：

```java
@RequestMapping(method = RequestMethod.POST, value = "/tokens/revokeRefreshToken/{tokenId:.*}")
@ResponseBody
public String revokeRefreshToken(@PathVariable String tokenId) {
    if (tokenStore instanceof JdbcTokenStore){
        ((JdbcTokenStore) tokenStore).removeRefreshToken(tokenId);
    }
    return tokenId;
}
```

为了测试刷新令牌在撤销后不再有效，我们将编写以下测试，其中我们获取访问令牌，刷新它，然后删除刷新令牌，并尝试再次刷新它。

我们将看到，撤销后，我们将收到响应错误：“invalid refresh token”：

```java
public class TokenRevocationLiveTest {
    private String refreshToken;

    private String obtainAccessToken(String clientId, String username, String password) {
        Map<String, String> params = new HashMap<String, String>();
        params.put("grant_type", "password");
        params.put("client_id", clientId);
        params.put("username", username);
        params.put("password", password);

        Response response = RestAssured.given().auth().
                preemptive().basic(clientId,"secret").and().with().params(params).
                when().post("http://localhost:8081/spring-security-oauth-server/oauth/token");
        refreshToken = response.jsonPath().getString("refresh_token");

        return response.jsonPath().getString("access_token");
    }

    private String obtainRefreshToken(String clientId) {
        Map<String, String> params = new HashMap<String, String>();
        params.put("grant_type", "refresh_token");
        params.put("client_id", clientId);
        params.put("refresh_token", refreshToken);

        Response response = RestAssured.given().auth()
                .preemptive().basic(clientId,"secret").and().with().params(params)
                .when().post("http://localhost:8081/spring-security-oauth-server/oauth/token");

        return response.jsonPath().getString("access_token");
    }

    private void authorizeClient(String clientId) {
        Map<String, String> params = new HashMap<String, String>();
        params.put("response_type", "code");
        params.put("client_id", clientId);
        params.put("scope", "read,write");

        Response response = RestAssured.given().auth().preemptive()
                .basic(clientId,"secret").and().with().params(params).
                when().post("http://localhost:8081/spring-security-oauth-server/oauth/authorize");
    }

    @Test
    public void givenUser_whenRevokeRefreshToken_thenRefreshTokenInvalidError() {
        String accessToken1 = obtainAccessToken("fooClientIdPassword", "john", "123");
        String accessToken2 = obtainAccessToken("fooClientIdPassword", "tom", "111");
        authorizeClient("fooClientIdPassword");

        String accessToken3 = obtainRefreshToken("fooClientIdPassword");
        authorizeClient("fooClientIdPassword");
        Response refreshTokenResponse = RestAssured.given().
                header("Authorization", "Bearer " + accessToken3)
                .get("http://localhost:8082/spring-security-oauth-resource/tokens");
        assertEquals(200, refreshTokenResponse.getStatusCode());

        Response revokeRefreshTokenResponse = RestAssured.given()
                .header("Authorization", "Bearer " + accessToken1)
                .post("http://localhost:8082/spring-security-oauth-resource/tokens/revokeRefreshToken/"+refreshToken);
        assertEquals(200, revokeRefreshTokenResponse.getStatusCode());

        String accessToken4 = obtainRefreshToken("fooClientIdPassword");
        authorizeClient("fooClientIdPassword");
        Response refreshTokenResponse2 = RestAssured.given()
                .header("Authorization", "Bearer " + accessToken4)
                .get("http://localhost:8082/spring-security-oauth-resource/tokens");
        assertEquals(401, refreshTokenResponse2.getStatusCode());
    }
}
```

## 8. 总结

在本教程中，我们演示了如何撤销OAuth访问令牌和OAuth刷新令牌。