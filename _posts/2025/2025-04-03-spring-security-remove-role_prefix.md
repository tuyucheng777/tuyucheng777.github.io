---
layout: post
title:  在Spring Security中删除ROLE_前缀
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

有时，在配置应用程序安全性时，我们的用户详细信息可能不包含[Spring Security](https://www.baeldung.com/category/spring/spring-security)期望的ROLE_前缀。因此，我们遇到“Forbidden”授权错误，无法访问我们的安全端点。

**在本教程中，我们将探讨如何重新配置Spring Security以允许使用没有ROLE_前缀的角色**。

## 2. Spring Security默认行为

**我们首先演示Spring Security角色检查机制的默认行为**，让我们添加一个仅包含一个具有ADMIN角色的用户的InMemoryUserDetailsManager：

```java
@Configuration
public class UserDetailsConfig {
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        UserDetails admin = User.withUsername("admin")
                .password(encoder().encode("password"))
                .authorities(singletonList(new SimpleGrantedAuthority("ADMIN")))
                .build();

        return new InMemoryUserDetailsManager(admin);
    }

    @Bean
    public PasswordEncoder encoder() {
        return new BCryptPasswordEncoder();
    }
}
```

我们创建了UserDetailsConfig配置类，该类生成一个[InMemoryUserDetailsManager](https://docs.spring.io/spring-security/reference/servlet/authentication/passwords/in-memory.html) Bean。在工厂方法中，我们使用了用户详细信息密码所需的[PasswordEncoder](https://www.baeldung.com/spring-security-5-default-password-encoder#springSecurity5)。

接下来，我们将添加想要调用的端点：

```java
@RestController
public class TestSecuredController {

    @GetMapping("/test-resource")
    public ResponseEntity<String> testAdmin() {
        return ResponseEntity.ok("GET request successful");
    }
}
```

我们添加了一个简单的GET端点，它应该返回200状态码。

让我们创建一个安全配置：

```java
@Configuration
@EnableWebSecurity
public class DefaultSecurityJavaConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.authorizeHttpRequests (authorizeRequests -> authorizeRequests
                        .requestMatchers("/test-resource").hasRole("ADMIN"))
                .httpBasic(withDefaults())
                .build();
    }
}
```

这里我们创建了一个[SecurityFilterChain](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/SecurityFilterChain.html) Bean，其中我们指定只有具有ADMIN角色的用户才能访问test-resource端点。

现在，让我们将这些配置添加到我们的测试上下文中并调用我们的安全端点：

```java
@WebMvcTest(controllers = TestSecuredController.class)
@ContextConfiguration(classes = { DefaultSecurityJavaConfig.class, UserDetailsConfig.class,
        TestSecuredController.class })
public class DefaultSecurityFilterChainIntegrationTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @BeforeEach
    void setup() {
        mockMvc =  MockMvcBuilders
            .webAppContextSetup(wac)
            .apply(SecurityMockMvcConfigurers.springSecurity())
            .build();
    }

    @Test
    void givenDefaultSecurityFilterChainConfig_whenCallTheResourceWithAdminRole_thenForbiddenResponseCodeExpected() throws Exception {
        MockHttpServletRequestBuilder with = MockMvcRequestBuilders.get("/test-resource")
                .header("Authorization", basicAuthHeader("admin", "password"));

        ResultActions performed = mockMvc.perform(with);

        MvcResult mvcResult = performed.andReturn();
        assertEquals(403, mvcResult.getResponse().getStatus());
    }
}
```

我们已将用户详细信息配置、安全配置和控制器Bean附加到测试上下文中。然后，我们使用管理员用户凭据调用测试资源，并在基本Authorization标头中发送它们。**但我们没有看到200响应码，而是看到了Forbidden响应码403**。

如果我们深入研究AuthorityAuthorizationManager.hasRole()方法的工作原理，我们会看到以下代码：

```java
public static <T> AuthorityAuthorizationManager<T> hasRole(String role) {
    Assert.notNull(role, "role cannot be null");
    Assert.isTrue(!role.startsWith(ROLE_PREFIX), () -> role + " should not start with " + ROLE_PREFIX + " since "
        + ROLE_PREFIX + " is automatically prepended when using hasRole. Consider using hasAuthority instead.");
    return hasAuthority(ROLE_PREFIX + role);
}
```

**我们可以看到，ROLE_PREFIX在这里是硬编码的，所有角色都应包含它才能通过验证**。当使用[方法安全](https://www.baeldung.com/spring-security-method-security)注解(例如@RolesAllowed)时，我们也会遇到类似的行为。

## 3. 使用权限而不是角色

解决这个问题最简单的方法是使用[权限而不是角色](https://www.baeldung.com/spring-security-granted-authority-vs-role)。**权限不需要预期的前缀，如果我们习惯使用它们，选择权限可以帮助我们避免与前缀相关的问题**。

### 3.1 基于SecurityFilterChain的配置

让我们在UserDetailsConfig类中修改我们的用户详细信息：

```java
@Configuration
public class UserDetailsConfig {
    @Bean
    public InMemoryUserDetailsManager userDetailsService() {
        PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
        UserDetails admin = User.withUsername("admin")
                .password(encoder.encode("password"))
                .authorities(Arrays.asList(new SimpleGrantedAuthority("ADMIN"),
                        new SimpleGrantedAuthority("ADMINISTRATION")))
                .build();

        return new InMemoryUserDetailsManager(admin);
    }
}
```

我们为管理员用户添加了一个名为ADMINISTRATION的权限，现在我们将根据权限访问创建安全配置：

```java
@Configuration
@EnableWebSecurity
public class AuthorityBasedSecurityJavaConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http.authorizeHttpRequests (authorizeRequests -> authorizeRequests
                        .requestMatchers("/test-resource").hasAuthority("ADMINISTRATION"))
                .httpBasic(withDefaults())
                .build();
    }
}
```

在此配置中，我们实现了相同的访问限制概念，但使用了AuthorityAuthorizationManager.hasAuthority()方法。让我们将新的安全配置设置到上下文中并调用我们的安全端点：

```java
@WebMvcTest(controllers = TestSecuredController.class)
@ContextConfiguration(classes = { AuthorityBasedSecurityJavaConfig.class, UserDetailsConfig.class,
        TestSecuredController.class })
public class AuthorityBasedSecurityFilterChainIntegrationTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @BeforeEach
    void setup() {
        mockMvc =  MockMvcBuilders
                .webAppContextSetup(wac)
                .apply(SecurityMockMvcConfigurers.springSecurity())
                .build();
    }

    @Test
    void givenAuthorityBasedSecurityJavaConfig_whenCallTheResourceWithAdminAuthority_thenOkResponseCodeExpected() throws Exception {
        MockHttpServletRequestBuilder with = MockMvcRequestBuilders.get("/test-resource")
                .header("Authorization", basicAuthHeader("admin", "password"));

        ResultActions performed = mockMvc.perform(with);

        MvcResult mvcResult = performed.andReturn();
        assertEquals(200, mvcResult.getResponse().getStatus());
    }
}
```

**我们可以看到，我们可以使用基于权限的安全配置的相同用户访问测试资源**。

### 3.2 基于注解的配置

**要开始使用基于注解的方法，我们首先需要启用方法安全性**。让我们使用@EnableMethodSecurity注解创建一个安全配置：

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(jsr250Enabled = true)
public class MethodSecurityJavaConfig {
}
```

现在，让我们向安全控制器添加一个端点：

```java
@RestController
public class TestSecuredController {

    @PreAuthorize("hasAuthority('ADMINISTRATION')")
    @GetMapping("/test-resource-method-security-with-authorities-resource")
    public ResponseEntity<String> testAdminAuthority() {
        return ResponseEntity.ok("GET request successful");
    }
}
```

在这里，我们使用了带有hasAuthority属性的[@PreAuthorize](https://www.baeldung.com/spring-security-method-security#3-using-preauthorize-and-postauthorize-annotations)注解，指定了我们预期的权限。准备就绪后，我们可以调用我们的安全端点：

```java
@WebMvcTest(controllers = TestSecuredController.class)
@ContextConfiguration(classes = { MethodSecurityJavaConfig.class, UserDetailsConfig.class,
        TestSecuredController.class })
public class AuthorityBasedMethodSecurityIntegrationTest {

    @Autowired
    private WebApplicationContext wac;

    private MockMvc mockMvc;

    @BeforeEach
    void setup() {
        mockMvc =  MockMvcBuilders
                .webAppContextSetup(wac)
                .apply(SecurityMockMvcConfigurers.springSecurity())
                .build();
    }

    @Test
    void givenMethodSecurityJavaConfig_whenCallTheResourceWithAdminAuthority_thenOkResponseCodeExpected() throws Exception {
        MockHttpServletRequestBuilder with = MockMvcRequestBuilders
                .get("/test-resource-method-security-with-authorities-resource")
                .header("Authorization", basicAuthHeader("admin", "password"));

        ResultActions performed = mockMvc.perform(with);

        MvcResult mvcResult = performed.andReturn();
        assertEquals(200, mvcResult.getResponse().getStatus());
    }
}
```

我们已将MethodSecurityJavaConfig和相同的UserDetailsConfig附加到测试上下文。然后，我们调用test-resource-method-security-with-authorities-resource端点并成功访问它。

## 4. SecurityFilterChain的自定义授权管理器

**如果我们需要使用没有ROLE_前缀的角色，我们必须将自定义的[AuthorizationManager](https://www.baeldung.com/spring-security-authorizationmanager)附加到SecurityFilterChain配置中**，此自定义管理器没有硬编码的前缀。

让我们创建这样的实现：

```java
public class CustomAuthorizationManager implements AuthorizationManager<RequestAuthorizationContext> {
    private final Set<String> roles = new HashSet<>();

    public CustomAuthorizationManager withRole(String role) {
        roles.add(role);
        return this;
    }

    @Override
    public AuthorizationDecision check(Supplier<Authentication> authentication,
                                       RequestAuthorizationContext object) {

        for (GrantedAuthority grantedRole : authentication.get().getAuthorities()) {
            if (roles.contains(grantedRole.getAuthority())) {
                return new AuthorizationDecision(true);
            }
        }

        return new AuthorizationDecision(false);
    }
}
```

我们实现了AuthorizationManager接口，在我们的实现中，我们可以指定多个角色，允许调用通过权限验证。在check()方法中，我们正在验证身份验证中的权限是否在我们预期的角色集合中。

现在，让我们将自定义授权管理器附加到SecurityFilterChain：

```java
@Configuration
@EnableWebSecurity
public class CustomAuthorizationManagerSecurityJavaConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
                .authorizeHttpRequests (authorizeRequests -> {
                    hasRole(authorizeRequests.requestMatchers("/test-resource"), "ADMIN");
                })
                .httpBasic(withDefaults());


        return http.build();
    }

    private void hasRole(AuthorizeHttpRequestsConfigurer.AuthorizedUrl authorizedUrl, String role) {
        authorizedUrl.access(new CustomAuthorizationManager().withRole(role));
    }
}
```

**这里我们没有使用AuthorityAuthorizationManager.hasRole()方法，而是使用了AuthorizeHttpRequestsConfigurer.access()，它允许我们使用自定义的AuthorizationManager实现**。

现在让我们配置测试上下文并调用安全端点：

```java
@WebMvcTest(controllers = TestSecuredController.class)
@ContextConfiguration(classes = { CustomAuthorizationManagerSecurityJavaConfig.class,
        TestSecuredController.class, UserDetailsConfig.class })
public class RemovingRolePrefixIntegrationTest {

    @Autowired
    WebApplicationContext wac;

    private MockMvc mockMvc;

    @BeforeEach
    void setup() {
        mockMvc = MockMvcBuilders
                .webAppContextSetup(wac)
                .apply(SecurityMockMvcConfigurers.springSecurity())
                .build();
    }

    @Test
    public void givenCustomAuthorizationManagerSecurityJavaConfig_whenCallTheResourceWithAdminRole_thenOkResponseCodeExpected() throws Exception {
        MockHttpServletRequestBuilder with = MockMvcRequestBuilders.get("/test-resource")
                .header("Authorization", basicAuthHeader("admin", "password"));

        ResultActions performed = mockMvc.perform(with);

        MvcResult mvcResult = performed.andReturn();
        assertEquals(200, mvcResult.getResponse().getStatus());
    }
}
```

我们附加了CustomAuthorizationManagerSecurityJavaConfig并调用了test-resource端点，正如预期的那样，我们收到了200响应码。

## 5. 重写GrantedAuthorityDefaults以确保方法安全

在基于注解的方法中，**我们可以覆盖我们在角色中使用的前缀**。

让我们修改MethodSecurityJavaConfig：

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(jsr250Enabled = true)
public class MethodSecurityJavaConfig {
    @Bean
    GrantedAuthorityDefaults grantedAuthorityDefaults() {
        return new GrantedAuthorityDefaults("");
    }
}
```

我们添加了GrantedAuthorityDefaults Bean，并传递了一个空字符串作为构造函数参数，**此空字符串将用作默认角色前缀**。

对于这个测试用例，我们将创建一个新的安全端点：

```java
@RestController
public class TestSecuredController {

    @RolesAllowed({"ADMIN"})
    @GetMapping("/test-resource-method-security-resource")
    public ResponseEntity<String> testAdminRole() {
        return ResponseEntity.ok("GET request successful");
    }
}
```

我们已将@RolesAllowed({"ADMIN"})添加到此端点，因此只有具有ADMIN角色的用户才能访问它。

我们来调用它并看看响应是什么：

```java
@WebMvcTest(controllers = TestSecuredController.class)
@ContextConfiguration(classes = { MethodSecurityJavaConfig.class, UserDetailsConfig.class,
        TestSecuredController.class })
public class RemovingRolePrefixMethodSecurityIntegrationTest {

    @Autowired
    WebApplicationContext wac;

    private MockMvc mockMvc;

    @BeforeEach
    void setup() {
        mockMvc = MockMvcBuilders
                .webAppContextSetup(wac)
                .apply(SecurityMockMvcConfigurers.springSecurity())
                .build();
    }

    @Test
    public void givenMethodSecurityJavaConfig_whenCallTheResourceWithAdminRole_thenOkResponseCodeExpected() throws Exception {
        MockHttpServletRequestBuilder with = MockMvcRequestBuilders.get("/test-resource-method-security-resource")
                .header("Authorization", basicAuthHeader("admin", "password"));

        ResultActions performed = mockMvc.perform(with);

        MvcResult mvcResult = performed.andReturn();
        assertEquals(200, mvcResult.getResponse().getStatus());
    }
}
```

我们已成功检索到对具有ADMIN角色(没有任何前缀)的用户调用test-resource-method-security-resource的200响应码。

## 6. 总结

在本文中，我们探讨了各种避免Spring Security中ROLE_前缀问题的方法。有些方法需要自定义，而其他方法则使用默认功能。本文介绍的方法可以帮助我们避免在用户详细信息中为角色添加前缀，这有时可能是不可能的。