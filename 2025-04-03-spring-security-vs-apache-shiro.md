---
layout: post
title:  Spring Security与Apache Shiro
category: spring-security
copyright: spring-security
excerpt: Spring Security
---

## 1. 概述

安全性是应用程序开发领域关注的重点，尤其是在企业Web和移动应用程序领域。

在本快速教程中，**我们将比较两个流行的Java安全框架-[Apache Shiro](https://shiro.apache.org/)和[Spring Security](https://spring.io/projects/spring-security)**。

## 2. 一点背景知识

Apache Shiro于2004年作为JSecurity诞生，并于2008年被Apache基金会接受。到目前为止，它已经发布了许多版本，撰写本文时的最新版本是1.5.3。

Spring Security于2003年以Acegi的形式开始，并于2008年首次公开发布时被纳入Spring框架。自成立以来，它经历了几次迭代，截至撰写本文时的当前GA版本是5.3.2。

**这两种技术都提供身份验证和授权支持以及加密和会话管理解决方案**。此外，Spring Security还提供一流的保护，可抵御CSRF和会话固定等攻击。

在接下来的几节中，我们将看到这两种技术如何处理身份验证和授权的示例。为了简单起见，我们将使用基于Spring Boot的基本MVC应用程序和[FreeMarker模板](https://www.baeldung.com/spring-template-engines#4-freemarker-in-spring-boot)。

## 3. 配置Apache Shiro

首先，让我们看看这两个框架的配置有何不同。

### 3.1 Maven依赖

由于我们将在Spring Boot App中使用Shiro，因此我们需要它的starter和shiro-core模块：

```xml
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-spring-boot-web-starter</artifactId>
    <version>1.5.3</version>
</dependency>
<dependency>
    <groupId>org.apache.shiro</groupId>
    <artifactId>shiro-core</artifactId>
    <version>1.5.3</version>
</dependency>
```

最新版本可以在[Maven Central](https://mvnrepository.com/artifact/org.apache.shiro)上找到。

### 3.2 创建Realm

为了在内存中声明用户及其角色和权限，我们需要创建一个扩展Shiro的JdbcRealm的Realm。我们将定义两个用户-Tom和Jerry，分别具有角色USER和ADMIN：

```java
public class CustomRealm extends JdbcRealm {

    private Map<String, String> credentials = new HashMap<>();
    private Map<String, Set> roles = new HashMap<>();
    private Map<String, Set> permissions = new HashMap<>();

    {
        credentials.put("Tom", "password");
        credentials.put("Jerry", "password");

        roles.put("Jerry", new HashSet<>(Arrays.asList("ADMIN")));
        roles.put("Tom", new HashSet<>(Arrays.asList("USER")));

        permissions.put("ADMIN", new HashSet<>(Arrays.asList("READ", "WRITE")));
        permissions.put("USER", new HashSet<>(Arrays.asList("READ")));
    }
}
```

接下来，为了能够检索此身份验证和授权，我们需要重写一些方法：

```java
@Override
protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
    UsernamePasswordToken userToken = (UsernamePasswordToken) token;

    if (userToken.getUsername() == null || userToken.getUsername().isEmpty() ||
            !credentials.containsKey(userToken.getUsername())) {
        throw new UnknownAccountException("User doesn't exist");
    }
    return new SimpleAuthenticationInfo(userToken.getUsername(),
            credentials.get(userToken.getUsername()), getName());
}

@Override
protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
    Set roles = new HashSet<>();
    Set permissions = new HashSet<>();

    for (Object user : principals) {
        try {
            roles.addAll(getRoleNamesForUser(null, (String) user));
            permissions.addAll(getPermissions(null, null, roles));
        } catch (SQLException e) {
            logger.error(e.getMessage());
        }
    }
    SimpleAuthorizationInfo authInfo = new SimpleAuthorizationInfo(roles);
    authInfo.setStringPermissions(permissions);
    return authInfo;
}
```

方法doGetAuthorizationInfo使用几个辅助方法来获取用户的角色和权限：

```java
@Override
protected Set getRoleNamesForUser(Connection conn, String username) throws SQLException {
    if (!roles.containsKey(username)) {
        throw new SQLException("User doesn't exist");
    }
    return roles.get(username);
}

@Override
protected Set getPermissions(Connection conn, String username, Collection roles) throws SQLException {
    Set userPermissions = new HashSet<>();
    for (String role : roles) {
        if (!permissions.containsKey(role)) {
            throw new SQLException("Role doesn't exist");
        }
        userPermissions.addAll(permissions.get(role));
    }
    return userPermissions;
}
```

接下来，我们需要将这个CustomRealm作为Bean包含在我们的应用程序中：

```java
@Bean
public Realm customRealm() {
    return new CustomRealm();
}
```

此外，为了为我们的端点配置身份验证，我们还需要另一个Bean：

```java
@Bean
public ShiroFilterChainDefinition shiroFilterChainDefinition() {
    DefaultShiroFilterChainDefinition filter = new DefaultShiroFilterChainDefinition();

    filter.addPathDefinition("/home", "authc");
    filter.addPathDefinition("/**", "anon");
    return filter;
}
```

这里，使用DefaultShiroFilterChainDefinition实例，我们指定/home端点只能由经过身份验证的用户访问。

这就是我们所需的全部配置，Shiro会为我们完成剩下的工作。

## 4. 配置Spring Security

现在让我们看看如何在Spring中实现同样的功能。

### 4.1 Maven依赖

首先，依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

最新版本可以在[Maven Central](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter)上找到。

### 4.2 配置类

接下来，我们将在SecurityConfig类中定义Spring Security配置：

```java
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf()
                .disable()
                .authorizeRequests(authorize -> authorize.antMatchers("/index", "/login")
                        .permitAll()
                        .antMatchers("/home", "/logout")
                        .authenticated()
                        .antMatchers("/admin/**")
                        .hasRole("ADMIN"))
                .formLogin(formLogin -> formLogin.loginPage("/login")
                        .failureUrl("/login-error"));
        return http.build();
    }

    @Bean
    public InMemoryUserDetailsManager userDetailsService() throws Exception {
        UserDetails jerry = User.withUsername("Jerry")
                .password(passwordEncoder().encode("password"))
                .authorities("READ", "WRITE")
                .roles("ADMIN")
                .build();
        UserDetails tom = User.withUsername("Tom")
                .password(passwordEncoder().encode("password"))
                .authorities("READ")
                .roles("USER")
                .build();
        return new InMemoryUserDetailsManager(jerry, tom);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

如我们所见，我们构建了一个UserDetails对象来声明我们的用户及其角色和权限。此外，我们使用BCryptPasswordEncoder对密码进行编码。

Spring Security还为我们提供了其HttpSecurity对象以供进一步配置。在我们的示例中，我们允许：

- 每个人都可以访问我们的index和login页面
- 只有经过身份验证的用户才能进入page和logout
- 只有具有ADMIN角色的用户才能访问admin页面

我们还定义了对基于表单的身份验证的支持，以将用户发送到登录端点。如果登录失败，我们的用户将被重定向到/login-error。

## 5. 控制器和端点

现在让我们看一下这两个应用程序的Web控制器映射，虽然它们将使用相同的端点，但一些实现会有所不同。

### 5.1 视图渲染端点

对于渲染视图的端点，实现是相同的：

```java
@GetMapping("/")
public String index() {
    return "index";
}

@GetMapping("/login")
public String showLoginPage() {
    return "login";
}

@GetMapping("/home")
public String getMeHome(Model model) {
    addUserAttributes(model);
    return "home";
}
```

我们的两个控制器实现Shiro和Spring Security都返回根端点上的index.ftl、登录端点上的login.ftl以及主端点上的home.ftl。

但是，两个控制器中/home端点的addUserAttributes方法的定义有所不同，此方法用于检查当前登录用户的属性。

Shiro提供了SecurityUtils#getSubject来检索当前Subject及其角色和权限：

```java
private void addUserAttributes(Model model) {
    Subject currentUser = SecurityUtils.getSubject();
    String permission = "";

    if (currentUser.hasRole("ADMIN")) {
        model.addAttribute("role", "ADMIN");
    } else if (currentUser.hasRole("USER")) {
        model.addAttribute("role", "USER");
    }
    if (currentUser.isPermitted("READ")) {
        permission = permission + " READ";
    }
    if (currentUser.isPermitted("WRITE")) {
        permission = permission + " WRITE";
    }
    model.addAttribute("username", currentUser.getPrincipal());
    model.addAttribute("permission", permission);
}
```

另一方面，Spring Security为此目的从其SecurityContextHolder的上下文中提供了一个Authentication对象：

```java
private void addUserAttributes(Model model) {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    if (auth != null && !auth.getClass().equals(AnonymousAuthenticationToken.class)) {
        User user = (User) auth.getPrincipal();
        model.addAttribute("username", user.getUsername());
        Collection<GrantedAuthority> authorities = user.getAuthorities();

        for (GrantedAuthority authority : authorities) {
            if (authority.getAuthority().contains("USER")) {
                model.addAttribute("role", "USER");
                model.addAttribute("permissions", "READ");
            } else if (authority.getAuthority().contains("ADMIN")) {
                model.addAttribute("role", "ADMIN");
                model.addAttribute("permissions", "READ WRITE");
            }
        }
    }
}
```

### 5.2 POST登录端点

在Shiro中，我们将用户输入的凭证映射到POJO：

```java
public class UserCredentials {

    private String username;
    private String password;

    // getters and setters
}
```

然后我们将创建一个UsernamePasswordToken来登录用户或Subject：

```java
@PostMapping("/login")
public String doLogin(HttpServletRequest req, UserCredentials credentials, RedirectAttributes attr) {

    Subject subject = SecurityUtils.getSubject();
    if (!subject.isAuthenticated()) {
        UsernamePasswordToken token = new UsernamePasswordToken(credentials.getUsername(),
                credentials.getPassword());
        try {
            subject.login(token);
        } catch (AuthenticationException ae) {
            logger.error(ae.getMessage());
            attr.addFlashAttribute("error", "Invalid Credentials");
            return "redirect:/login";
        }
    }
    return "redirect:/home";
}
```

在Spring Security方面，这只是重定向到主页的问题。**Spring的登录过程由其UsernamePasswordAuthenticationFilter处理，对我们来说是透明的**：

```java
@PostMapping("/login")
public String doLogin(HttpServletRequest req) {
    return "redirect:/home";
}
```

### 5.3 仅管理员端点

现在让我们看一个必须执行基于角色的访问的场景，假设我们有一个/admin端点，只有ADMIN角色才可以访问它。

让我们看看如何在Shiro中做到这一点：

```java
@GetMapping("/admin")
public String adminOnly(ModelMap modelMap) {
    addUserAttributes(modelMap);
    Subject currentUser = SecurityUtils.getSubject();
    if (currentUser.hasRole("ADMIN")) {
        modelMap.addAttribute("adminContent", "only admin can view this");
    }
    return "home";
}
```

这里我们提取了当前登录的用户，检查他们是否具有ADMIN角色，并相应地添加内容。

在Spring Security中，无需以编程方式检查角色，我们已经在SecurityConfig中定义了谁可以访问此端点。所以现在，只需添加业务逻辑即可：

```java
@GetMapping("/admin")
public String adminOnly(HttpServletRequest req, Model model) {
    addUserAttributes(model);
    model.addAttribute("adminContent", "only admin can view this");
    return "home";
}
```

### 5.4 注销端点

最后，让我们实现注销端点。

在Shiro中，我们只需调用Subject#logout：

```java
@PostMapping("/logout")
public String logout() {
    Subject subject = SecurityUtils.getSubject();
    subject.logout();
    return "redirect:/";
}
```

对于Spring，我们没有定义任何注销映射。在这种情况下，其默认注销机制将启动，因为我们在配置中创建了SecurityFilterChain Bean，所以该机制会自动应用。

## 6. Apache Shiro与Spring Security

现在我们已经了解了实现方面的差异，让我们看看其他几个方面。

在社区支持方面，**Spring框架通常拥有庞大的开发者社区**，积极参与其开发和使用。由于Spring Security是该框架的一部分，因此它必须享有相同的优势。Shiro虽然很受欢迎，但并没有如此庞大的支持。

就文档而言，Spring再次成为赢家。

但是，Spring Security的学习难度有点大。**而Shiro则比较容易理解**。对于桌面应用程序，通过[shiro.ini](https://shiro.apache.org/configuration.html#Configuration-CreatingaSecurityManagerfromINI)进行配置就更容易了。

但是，正如我们在示例片段中看到的，**Spring Security在保持业务逻辑和安全性分离方面做得很好，并且真正将安全性作为横切关注点**。

## 7. 总结

在本教程中，我们将Apache Shiro与Spring Security进行了比较。

我们只是粗略地了解了这些框架所提供的功能，还有很多内容需要进一步探索。目前有很多替代方案，例如[JAAS](https://docs.oracle.com/en/java/javase/11/security/java-authentication-and-authorization-service-jaas-reference-guide.html)和[OACC](http://oaccframework.org/)。不过，凭借其优势，[Spring Security](https://www.baeldung.com/security-spring)似乎目前胜出。