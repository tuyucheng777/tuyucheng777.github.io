---
layout: post
title:  Jakarta EE 8 Security API
category: security
copyright: security
excerpt: Jakarta EE 8 Security
---

## 1. 概述

Jakarta EE 8 Security API是新标准，也是处理Java容器中安全问题的一种可移植方式。

在本文中，**我们将介绍该API的三个核心功能**：

1. **HTTP认证机制**
2. **身份存储**
3. **安全上下文**

我们首先了解如何配置提供的实现，然后了解如何实现自定义的实现。

## 2. Maven依赖

要设置Jakarta EE 8 Security API，我们需要服务器提供的实现或明确的实现。

### 2.1 使用服务器实现

Jakarta EE 8兼容服务器已经为Jakarta EE 8 Security API提供了实现，因此我们只需要[Jakarta EE Web Profile API](https://mvnrepository.com/artifact/javax/javaee-web-api) Maven工件：

```xml
<dependencies>
    <dependency>
        <groupId>javax</groupId>
        <artifactId>javaee-web-api</artifactId>
        <version>8.0.1</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

### 2.2 使用显式实现

首先，我们为[Jakarta EE 8 Security API](https://mvnrepository.com/search?q=javax.security.enterprise-api)指定Maven工件：

```xml
<dependencies>
    <dependency>
        <groupId>javax.security.enterprise</groupId>
        <artifactId>javax.security.enterprise-api</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

然后，我们将添加一个实现，例如[Soteria](https://mvnrepository.com/search?q=javax.security.enterprise)参考实现：

```xml
<dependencies>
    <dependency>
        <groupId>org.glassfish.soteria</groupId>
        <artifactId>javax.security.enterprise</artifactId>
        <version>1.0</version>
    </dependency>
</dependencies>
```

## 3. HTTP认证机制

在Jakarta EE 8之前，我们通过web.xml文件以声明方式配置身份验证机制。

在此版本中，Jakarta EE 8 Security API设计了新的HttpAuthenticationMechanism接口作为替代。因此，Web应用程序现在可以通过提供此接口的实现来配置身份验证机制。

幸运的是，容器已经为Servlet规范定义的三种身份验证方法分别提供了实现：基本HTTP身份验证、基于表单的身份验证和自定义的基于表单的身份验证。

它还提供了注解来触发每个实现：

1. @BasicAuthenticationMechanismDefinition
2. @FormAuthenticationMechanismDefinition
3. @CustomFormAuthenrticationMechanismDefinition

### 3.1 基本HTTP身份验证

**如上所述，Web应用程序只需使用CDI Bean上的@BasicAuthenticationMechanismDefinition注解即可配置基本HTTP身份验证**：

```java
@BasicAuthenticationMechanismDefinition(realmName = "userRealm")
@ApplicationScoped
public class AppConfig{}
```

此时，Servlet容器搜索并实例化提供的HttpAuthenticationMechanism接口的实现。

收到未经授权的请求后，容器将通过WWW-Authenticate响应标头要求客户端提供适当的身份验证信息。

```text
WWW-Authenticate: Basic realm="userRealm"
```

然后，客户端通过Authorization请求标头发送用户名和密码，以冒号“:”分隔并以Base64编码：

```text
//user=tuyucheng, password=tuyucheng
Authorization: Basic YmFlbGR1bmc6YmFlbGR1bmc=
```

请注意，提供凭据的对话框来自浏览器而不是服务器。

### 3.2 基于表单的HTTP身份验证

**@FormAuthenticationMechanismDefinition注解触发Servlet规范定义的基于表单的身份验证**。

然后我们可以选择指定登录和错误页面，或者使用默认的合理的/login和/login-error：

```java
@FormAuthenticationMechanismDefinition(loginToContinue = @LoginToContinue(
        loginPage = "/login.html",
        errorPage = "/login-error.html"))
@ApplicationScoped
public class AppConfig{}
```

调用loginPage的结果是，服务器应该将表单发送给客户端：

```html
<form action="j_security_check" method="post">
    <input name="j_username" type="text"/>
    <input name="j_password" type="password"/>
    <input type="submit">
</form>
```

然后，客户端应该将表单发送到容器提供的预定义支持身份验证过程。

### 3.3 自定义基于表单的HTTP身份验证

Web应用程序可以使用注解@CustomFormAuthenticationMechanismDefinition触发自定义基于表单的身份验证实现：

```java
@CustomFormAuthenticationMechanismDefinition(loginToContinue = @LoginToContinue(loginPage = "/login.xhtml"))
@ApplicationScoped
public class AppConfig {
}
```

但与默认的基于表单的身份验证不同，我们配置自定义登录页面并调用SecurityContext.authenticate()方法作为支持身份验证过程。

让我们看一下支持LoginBean，它包含登录逻辑：

```java
@Named
@RequestScoped
public class LoginBean {

    @Inject
    private SecurityContext securityContext;

    @NotNull private String username;

    @NotNull private String password;

    public void login() {
        Credential credential = new UsernamePasswordCredential(username, new Password(password));
        AuthenticationStatus status = securityContext
                .authenticate(
                        getHttpRequestFromFacesContext(),
                        getHttpResponseFromFacesContext(),
                        withParams().credential(credential));
        // ...
    }

    // ...
}
```

调用自定义login.xhtml页面的结果是，客户端将接收到的表单提交给LoginBean的login()方法：

```html
//...
<input type="submit" value="Login" jsf:action="#{loginBean.login}"/>
```

### 3.4 自定义认证机制

HttpAuthenticationMechanism接口定义了3个方法，**最重要的是validateRequest()**，我们必须提供它的实现。

在大多数情况下，其他两种方法secureResponse()和cleanSubject()的默认行为就足够了。

我们来看一个示例实现：

```java
@ApplicationScoped
public class CustomAuthentication implements HttpAuthenticationMechanism {

    @Override
    public AuthenticationStatus validateRequest(HttpServletRequest request, HttpServletResponse response,
            HttpMessageContext httpMsgContext) throws AuthenticationException {
        String username = request.getParameter("username");
        String password = response.getParameter("password");
        // mocking UserDetail, but in real life, we can obtain it from a database
        UserDetail userDetail = findByUserNameAndPassword(username, password);
        if (userDetail != null) {
            return httpMsgContext.notifyContainerAboutLogin(
                    new CustomPrincipal(userDetail),
                    new HashSet<>(userDetail.getRoles()));
        }
        return httpMsgContext.responseUnauthorized();
    }
    //...
}
```

在这里，实现提供了验证过程的业务逻辑，但在实践中，建议通过调用validate来通过IdentityStoreHandler委托给IdentityStore。

我们还用@ApplicationScoped注解对实现进行了标注，因为我们需要使其支持CDI。

在对凭证进行有效验证并最终检索用户角色之后，**实现应通知容器**：

```java
HttpMessageContext.notifyContainerAboutLogin(Principal principal, Set groups)
```

### 3.5 加强Servlet安全性

**Web应用程序可以通过在Servlet实现上使用@ServletSecurity注解来强制实施安全约束**：

```java
@WebServlet("/secured")
@ServletSecurity(value = @HttpConstraint(rolesAllowed = {"admin_role"}),
        httpMethodConstraints = {
                @HttpMethodConstraint(value = "GET", rolesAllowed = {"user_role"}),
                @HttpMethodConstraint(value = "POST", rolesAllowed = {"admin_role"})
        })
public class SecuredServlet extends HttpServlet {
}
```

此注解有两个属性httpMethodConstraints和value；httpMethodConstraints用于指定一个或多个约束，每个约束表示通过允许的角色列表对HTTP方法的访问控制。

然后，容器将针对每个URL模式和HTTP方法检查连接的用户是否具有访问资源的适当角色。

## 4. 身份存储

**该功能由IdentityStore接口抽象化，用于验证凭证并最终检索组成员身份**。换句话说，它可以提供身份验证、授权或两者兼有的功能。

IdentityStore旨在并鼓励通过名为IdentityStoreHandler的接口由HttpAuthenticationMecanism使用，Servlet容器提供了IdentityStoreHandler的默认实现。 

应用程序可以提供其IdentityStore的实现，或者使用容器为数据库和LDAP提供的两个内置实现之一。

### 4.1 内置身份存储

Jakarta EE兼容服务器应该**为两个身份存储提供实现：数据库和LDAP**。

**通过将配置数据传递给@DataBaseIdentityStoreDefinition注解来初始化数据库IdentityStore实现**：

```java
@DatabaseIdentityStoreDefinition(
        dataSourceLookup = "java:comp/env/jdbc/securityDS",
        callerQuery = "select password from users where username = ?",
        groupsQuery = "select GROUPNAME from groups where username = ?",
        priority=30)
@ApplicationScoped
public class AppConfig {
}
```

作为配置数据，**我们需要一个到外部数据库的JNDI数据源**、两个用于检查调用者及其组的JDBC语句，最后，配置一个在多个存储的情况下使用的优先级参数。

具有高优先级的IdentityStore将稍后由IdentityStoreHandler进行处理。

与数据库类似，**LDAP IdentityStore实现通过传递配置数据通过@LdapIdentityStoreDefinition进行初始化**：

```java
@LdapIdentityStoreDefinition(
        url = "ldap://localhost:10389",
        callerBaseDn = "ou=caller,dc=tuyucheng,dc=com",
        groupSearchBase = "ou=group,dc=tuyucheng,dc=com",
        groupSearchFilter = "(&(member=%s)(objectClass=groupOfNames))")
@ApplicationScoped
public class AppConfig {
}
```

这里我们需要一个外部LDAP服务器的URL，如何在LDAP目录中搜索调用者，以及如何检索他的组。

### 4.2 实现自定义身份存储

**IdentityStore接口定义了4种默认方法**：

```java
default CredentialValidationResult validate(Credential credential)
default Set<String> getCallerGroups(CredentialValidationResult validationResult)
default int priority()
default Set<ValidationType> validationTypes()
```

prioritize()方法返回一个值来表示迭代顺序；此实现由IdentityStoreHandler处理，优先级较低的IdentityStore会被优先处理。 

默认情况下，IdentityStore同时处理凭证验证(ValidationType.VALIDATE)和组检索(ValidationType.PROVIDE_GROUPS)，我们可以覆盖此行为，以便它只提供一项功能。

因此，我们可以将IdentityStore配置为仅用于凭证验证：

```java
@Override
public Set<ValidationType> validationTypes() {
    return EnumSet.of(ValidationType.VALIDATE);
}
```

在这种情况下，我们应该为validate()方法提供一个实现：

```java
@ApplicationScoped
public class InMemoryIdentityStore implements IdentityStore {
    // init from a file or harcoded
    private Map<String, UserDetails> users = new HashMap<>();

    @Override
    public int priority() {
        return 70;
    }

    @Override
    public Set<ValidationType> validationTypes() {
        return EnumSet.of(ValidationType.VALIDATE);
    }

    public CredentialValidationResult validate(UsernamePasswordCredential credential) {
        UserDetails user = users.get(credential.getCaller());
        if (credential.compareTo(user.getLogin(), user.getPassword())) {
            return new CredentialValidationResult(user.getLogin());
        }
        return INVALID_RESULT;
    }
}
```

或者我们可以选择配置IdentityStore，以便它仅用于组检索：

```java
@Override
public Set<ValidationType> validationTypes() {
    return EnumSet.of(ValidationType.PROVIDE_GROUPS);
}
```

然后我们应该为getCallerGroups()方法提供一个实现：

```java
@ApplicationScoped
public class InMemoryIdentityStore implements IdentityStore {
    // init from a file or harcoded
    private Map<String, UserDetails> users = new HashMap<>();

    @Override
    public int priority() {
        return 90;
    }

    @Override
    public Set<ValidationType> validationTypes() {
        return EnumSet.of(ValidationType.PROVIDE_GROUPS);
    }

    @Override
    public Set<String> getCallerGroups(CredentialValidationResult validationResult) {
        UserDetails user = users.get(validationResult.getCallerPrincipal().getName());
        return new HashSet<>(user.getRoles());
    }
}
```

因为IdentityStoreHandler期望实现是一个CDI Bean，所以我们用ApplicationScoped标注来装饰它。

## 5. SecurityContext API

**Jakarta EE 8 Security API通过SecurityContext接口提供编程安全性的访问点**，当容器强制执行的声明式安全模型不够用时，这是一种替代方案。

SecurityContext接口的默认实现应该在运行时作为CDI Bean提供，因此我们需要注入它：

```java
@Inject
SecurityContext securityContext;
```

此时，我们可以通过5种可用方法对用户进行身份验证、检索已经过身份验证的用户、检查其角色成员身份以及授予或拒绝对Web资源的访问。

### 5.1 检索调用者数据

在以前的Jakarta EE版本中，我们会以不同的方式检索Principal或检查每个容器中的角色成员身份。

当我们在Servlet容器中使用HttpServletRequest的getUserPrincipal()和isUserInRole()方法时，在EJB容器中使用EJBContext的类似方法getCallerPrincipal()和isCallerInRole()方法。

**新的Jakarta EE 8 Security API通过SecurityContext接口提供了类似的方法，从而实现了标准化**：

```java
Principal getCallerPrincipal();
boolean isCallerInRole(String role);
<T extends Principal> Set<T> getPrincipalsByType(Class<T> type);
```

getCallerPrincipal()方法返回经过身份验证的调用者的特定于容器的表示，而getPrincipalsByType()方法检索给定类型的所有主体。

如果特定于应用程序的调用者与容器的调用者不同，它会很有用。

### 5.2 测试Web资源访问

首先，我们需要配置一个受保护的资源：

```java
@WebServlet("/protectedServlet")
@ServletSecurity(@HttpConstraint(rolesAllowed = "USER_ROLE"))
public class ProtectedServlet extends HttpServlet {
    // ...
}
```

然后，为了检查对这个受保护资源的访问，我们应该调用hasAccessToWebResource()方法：

```java
securityContext.hasAccessToWebResource("/protectedServlet", "GET");
```

在这种情况下，如果用户处于USER_ROLE角色，则该方法返回true。

### 5.3 通过编程方式验证调用者

应用程序可以通过调用authenticate()以编程方式触发身份验证过程：

```java
AuthenticationStatus authenticate(
    HttpServletRequest request, 
    HttpServletResponse response,
    AuthenticationParameters parameters);
```

然后，容器会收到通知，并依次调用为应用程序配置的身份验证机制；AuthenticationParameters参数为HttpAuthenticationMechanism提供凭证：

```java
withParams().credential(credential)
```

AuthenticationStatus的SUCCESS和SEND_FAILURE值表示身份验证成功和失败，而SEND_CONTINUE表示身份验证过程的进行中状态。

## 6. 运行示例

为了突出显示这些示例，我们使用了支持Jakarta EE 8的[Open Liberty Server](https://openliberty.io/)的最新开发版本，它是通过[liberty-maven-plugin](https://mvnrepository.com/artifact/io.openliberty.tools/liberty-maven-plugin)下载和安装的，它将获取启动服务器所需的所有依赖。

要运行示例，只需访问相应的模块并调用此命令：

```shell
mvn clean package liberty:run
```

因此，Maven将下载服务器、构建、部署和运行应用程序。

## 7. 总结

在本文中，我们介绍了新Jakarta EE 8 Security API的主要功能的配置和实现。

首先，我们展示了如何配置默认的内置身份验证机制以及如何实现自定义身份验证机制。然后，我们了解了如何配置内置身份存储以及如何实现自定义身份存储。最后，我们了解了如何调用SecurityContext的方法。