---
layout: post
title:  Apache Shiro简介
category: security
copyright: security
excerpt: Apache Shiro
---

## 1. 概述

在本文中，我们将介绍多功能Java安全框架[Apache Shiro](https://shiro.apache.org/)。

该框架高度可定制和模块化，因为它提供身份验证、授权、加密和会话管理。

## 2. 依赖

Apache Shiro有许多[模块](https://shiro.apache.org/download.html)，但是在本教程中，我们仅使用shiro-core。

让我们将它添加到pom.xml中：

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.4.0</version>
</dependency>
```

可以在[Maven Central](https://mvnrepository.com/artifact/org.apache.shiro)上找到Apache Shiro模块的最新版本。

## 3. 配置安全管理器

SecurityManager是Apache Shiro框架的核心，应用程序通常只运行一个SecurityManager实例。

在本教程中，我们在桌面环境中探索该框架。要配置框架，我们需要在资源文件夹中创建一个shiro.ini文件，其中包含以下内容：

```properties
[users]
user = password, admin
user2 = password2, editor
user3 = password3, author

[roles]
admin = *
editor = articles:*
author = articles:compose,articles:save
```

shiro.ini配置文件的[users\]部分定义了SecurityManager识别的用户凭证，格式为：principal(用户名) = role1,role2, ...,role(密码)。

角色及其相关权限在[roles\]部分中声明，admin角色被授予权限并可以访问应用程序的每个部分，这由通配符(\*)表示。

editor角色拥有与文章相关的所有权限，而author角色只能撰写和保存文章。

SecurityManager用于配置SecurityUtils类，从SecurityUtils中我们可以获取当前与系统交互的用户，并进行认证、授权等操作。

让我们使用IniRealm从shiro.ini文件加载我们的用户和角色定义，然后使用它来配置DefaultSecurityManager对象：

```java
IniRealm iniRealm = new IniRealm("classpath:shiro.ini");
SecurityManager securityManager = new DefaultSecurityManager(iniRealm);

SecurityUtils.setSecurityManager(securityManager);
Subject currentUser = SecurityUtils.getSubject();
```

现在我们有了一个知道shiro.ini文件中定义的用户凭据和角色的SecurityManager，让我们继续进行用户身份验证和授权。

## 4. 身份验证

在Apache Shiro的术语中，主体是与系统交互的任何实体，它可以是人、脚本或REST客户端。

调用SecurityUtils.getSubject()将返回当前Subject的实例，即currentUser。

现在我们有了currentUser对象，我们可以对提供的凭据执行身份验证：

```java
if (!currentUser.isAuthenticated()) {               
    UsernamePasswordToken token = new UsernamePasswordToken("user", "password");
    token.setRememberMe(true);                        
    try {                                             
        currentUser.login(token);                       
    } catch (UnknownAccountException uae) {           
        log.error("Username Not Found!", uae);        
    } catch (IncorrectCredentialsException ice) {     
        log.error("Invalid Credentials!", ice);       
    } catch (LockedAccountException lae) {            
        log.error("Your Account is Locked!", lae);    
    } catch (AuthenticationException ae) {            
        log.error("Unexpected Error!", ae);           
    }                                                 
}
```

首先，我们检查当前用户是否尚未通过身份验证，然后我们使用用户的主体(用户名)和凭据(密码)创建一个身份验证令牌。

接下来，我们尝试使用令牌登录，如果提供的凭据正确，则一切都会顺利。

不同情况有不同的异常，也可以抛出更适合应用程序要求的自定义异常，这可以通过对AccountException类进行子类化来实现。

## 5. 授权

身份验证试图验证用户的身份，而授权试图控制对系统中某些资源的访问。

回想一下，我们在shiro.ini文件中为每个创建的用户分配一个或多个角色。此外，在角色部分，我们为每个角色定义不同的权限或访问级别。

现在让我们看看如何在应用程序中使用它来强制用户访问控制。

在shiro.ini文件中，我们授予admin对系统每个部分的完全访问权限。

editor对文章的所有资源/操作具有完全的访问权，而author仅限于撰写和保存文章。

让我们根据角色处理当前用户：

```java
if (currentUser.hasRole("admin")) {       
    log.info("Welcome Admin");              
} else if(currentUser.hasRole("editor")) {
    log.info("Welcome, Editor!");           
} else if(currentUser.hasRole("author")) {
    log.info("Welcome, Author");            
} else {                                  
    log.info("Welcome, Guest");             
}
```

现在，让我们看看当前用户在系统中被允许做什么：

```java
if(currentUser.isPermitted("articles:compose")) {            
    log.info("You can compose an article");                    
} else {                                                     
    log.info("You are not permitted to compose an article!");
}                                                            
                                                             
if(currentUser.isPermitted("articles:save")) {               
    log.info("You can save articles");                         
} else {                                                     
    log.info("You can not save articles");                   
}                                                            
                                                             
if(currentUser.isPermitted("articles:publish")) {            
    log.info("You can publish articles");                      
} else {                                                     
    log.info("You can not publish articles");                
}
```

## 6. Realm配置

在实际应用中，我们需要一种从数据库而不是shiro.ini文件获取用户凭据的方法，这就是Realm概念发挥作用的地方。

在Apache Shiro的术语中，[Realm](https://shiro.apache.org/realm.html)是一个DAO，指向身份验证和授权所需的用户凭证存储。

要创建一个Realm，我们只需要实现Realm接口。这可能很繁琐；但是，框架附带了我们可以从中子类化的默认实现，其中一个实现是JdbcRealm。

我们创建一个自定义Realm实现，扩展JdbcRealm类并重写以下方法：doGetAuthenticationInfo()，doGetAuthorizationInfo()，getRoleNamesForUser()和getPermissions()。

让我们通过子类化JdbcRealm类来创建一个Realm：

```java
public class MyCustomRealm extends JdbcRealm {
    // ...
}
```

为了简单起见，我们使用java.util.Map来模拟数据库：

```java
private Map<String, String> credentials = new HashMap<>();
private Map<String, Set<String>> roles = new HashMap<>();
private Map<String, Set<String>> perm = new HashMap<>();

{
    credentials.put("user", "password");
    credentials.put("user2", "password2");
    credentials.put("user3", "password3");
                                          
    roles.put("user", new HashSet<>(Arrays.asList("admin")));
    roles.put("user2", new HashSet<>(Arrays.asList("editor")));
    roles.put("user3", new HashSet<>(Arrays.asList("author")));
                                                             
    perm.put("admin", new HashSet<>(Arrays.asList("*")));
    perm.put("editor", new HashSet<>(Arrays.asList("articles:*")));
    perm.put("author", 
        new HashSet<>(Arrays.asList("articles:compose", 
        "articles:save")));
}
```

让我们继续并覆盖doGetAuthenticationInfo()：

```java
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    UsernamePasswordToken uToken = (UsernamePasswordToken) token;

    if(uToken.getUsername() == null
            || uToken.getUsername().isEmpty()
            || !credentials.containsKey(uToken.getUsername())) {
        throw new UnknownAccountException("username not found!");
    }

    return new SimpleAuthenticationInfo(
            uToken.getUsername(),
            credentials.get(uToken.getUsername()),
            getName());
}
```

我们首先将提供的AuthenticationToken转换为UsernamePasswordToken，从uToken中提取用户名(uToken.getUsername())并使用它从数据库中获取用户凭证(密码)。

如果没有找到记录，我们将抛出一个UnknownAccountException，否则我们将使用凭证和用户名来构造从该方法返回的SimpleAuthenticationInfo对象。

如果用户凭证使用盐进行哈希，我们需要返回带有相关盐的SimpleAuthenticationInfo：

```java
return new SimpleAuthenticationInfo(
    uToken.getUsername(), 
    credentials.get(uToken.getUsername()), 
    ByteSource.Util.bytes("salt"), 
    getName()
);
```

我们还需要重写doGetAuthorizationInfo()以及getRoleNamesForUser()和getPermissions()。

最后，让我们将自定义Realm插入securityManager，我们需要做的就是用我们的自定义Realm替换上面的IniRealm，并将其传递给DefaultSecurityManager的构造函数：

```java
Realm realm = new MyCustomRealm();
SecurityManager securityManager = new DefaultSecurityManager(realm);
```

代码的其他部分与之前相同，这就是我们正确配置securityManager和自定义Realm所需的全部内容。

现在的问题是-框架如何匹配凭证？

默认情况下，JdbcRealm使用SimpleCredentialsMatcher，它仅通过比较AuthenticationToken和AuthenticationInfo中的凭证来检查是否相等。

如果我们对密码进行哈希处理，则需要通知框架改用HashedCredentialsMatcher，可以在[此处](https://shiro.apache.org/realm.html#Realm-HashingCredentials)找到具有哈希密码的Realm的INI配置。

## 7. 注销

现在我们已经验证了用户身份，是时候实现注销了。只需调用一个方法即可完成-使用户会话无效并注销用户：

```java
currentUser.logout();
```

## 8. 会话管理

该框架自然带有其会话管理系统，如果在Web环境中使用，则默认为HttpSession实现。

对于独立应用程序，它使用其企业会话管理系统。这样做的好处是，即使在桌面环境中，你也可以像在典型的Web环境中一样使用会话对象。

让我们看一个简单的例子并与当前用户的会话进行交互：

```java
Session session = currentUser.getSession();                
session.setAttribute("key", "value");                      
String value = (String) session.getAttribute("key");       
if (value.equals("value")) {                               
    log.info("Retrieved the correct value! [" + value + "]");
}
```

## 9. 使用Spring开发Web应用程序的Shiro

到目前为止，我们已经概述了Apache Shiro的基本结构，并在桌面环境中实现了它，让我们继续将框架集成到Spring Boot应用程序中。

请注意，这里主要关注的是Shiro，而不是Spring应用程序-我们将只使用它来支持一个简单的示例应用程序。

### 9.1 依赖

首先，我们需要将Spring Boot父依赖项添加到pom.xml中：

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
</parent>
```

接下来，我们必须在同一个pom.xml文件中添加以下依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-freemarker</artifactId>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>${apache-shiro-core-version}</version>
</dependency>
```

### 9.2 配置

将shiro-spring-boot-web-starter依赖添加到pom.xml中将默认配置Apache Shiro应用程序的一些功能，例如SecurityManager。

但是，我们仍然需要配置Realm和Shiro安全过滤器，我们将使用上面定义的相同自定义Realm。

因此，在运行Spring Boot应用程序的主类中，让我们添加以下Bean定义：

```java
@Bean
public Realm realm() {
    return new MyCustomRealm();
}
    
@Bean
public ShiroFilterChainDefinition shiroFilterChainDefinition() {
    DefaultShiroFilterChainDefinition filter = new DefaultShiroFilterChainDefinition();

    filter.addPathDefinition("/secure", "authc");
    filter.addPathDefinition("/**", "anon");

    return filter;
}
```

在ShiroFilterChainDefinition中，我们将authc过滤器应用于/secure路径，并使用Ant模式将anon过滤器应用于其他路径。

authc和anon过滤器都是Web应用程序的默认过滤器，其他默认过滤器可在[此处](https://shiro.apache.org/web.html#Web-DefaultFilters)找到。

如果我们没有定义Realm Bean，ShiroAutoConfiguration将默认提供一个IniRealm实现，该实现期望在src/main/resources或src/main/resources/META-INF中找到一个shiro.ini文件。

如果我们没有定义ShiroFilterChainDefinition Bean，框架将保护所有路径并将登录URL设置为login.jsp。

我们可以通过在application.properties中添加以下条目来更改此默认登录URL和其他默认值：

```properties
shiro.loginUrl = /login
shiro.successUrl = /secure
shiro.unauthorizedUrl = /login
```

现在，authc过滤器已应用于/secure，对该路由的所有请求都将需要表单身份验证。

### 9.3 身份验证和授权

让我们创建一个具有以下路径映射的ShiroSpringController：/index、/login、/logout和/secure。

login()方法就是我们实现上述实际用户身份验证的地方，如果身份验证成功，用户将被重定向到安全页面：

```java
Subject subject = SecurityUtils.getSubject();

if(!subject.isAuthenticated()) {
    UsernamePasswordToken token = new UsernamePasswordToken(cred.getUsername(), cred.getPassword(), cred.isRememberMe());
    try {
        subject.login(token);
    } catch (AuthenticationException ae) {
        ae.printStackTrace();
        attr.addFlashAttribute("error", "Invalid Credentials");
        return "redirect:/login";
    }
}

return "redirect:/secure";
```

现在，在secure()实现中，通过调用SecurityUtils.getSubject()获取currentUser，用户的角色和权限以及用户的主体被传递到安全页面：

```java
Subject currentUser = SecurityUtils.getSubject();
String role = "", permission = "";

if(currentUser.hasRole("admin")) {
    role = role  + "You are an Admin";
} else if(currentUser.hasRole("editor")) {
    role = role + "You are an Editor";
} else if(currentUser.hasRole("author")) {
    role = role + "You are an Author";
}

if(currentUser.isPermitted("articles:compose")) {
    permission = permission + "You can compose an article, ";
} else {
    permission = permission + "You are not permitted to compose an article!, ";
}

if(currentUser.isPermitted("articles:save")) {
    permission = permission + "You can save articles, ";
} else {
    permission = permission + "\nYou can not save articles, ";
}

if(currentUser.isPermitted("articles:publish")) {
    permission = permission  + "\nYou can publish articles";
} else {
    permission = permission + "\nYou can not publish articles";
}

modelMap.addAttribute("username", currentUser.getPrincipal());
modelMap.addAttribute("permission", permission);
modelMap.addAttribute("role", role);

return "secure";
```

这就是我们将Apache Shiro集成到Spring Boot应用程序中的方法。

另请注意，该框架提供了可与过滤器链定义一起使用的附加[注解](https://shiro.apache.org/spring.html)，以保护我们的应用程序。

## 10. JEE集成

将Apache Shiro集成到JEE应用程序中只需配置web.xml文件即可，与往常一样，配置要求shiro.ini位于类路径中。[此处](https://shiro.apache.org/web.html#Web-{{web.xml}})[在此处](https://shiro.apache.org/web.html#Web-JSP%2FGSPTagLibrary)提供了详细的示例配置，此外，还可以找到JSP标签。

## 11. 总结

在本教程中，我们了解了Apache Shiro的身份验证和授权机制，我们还重点介绍了如何定义自定义Realm并将其插入SecurityManager。