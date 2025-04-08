---
layout: post
title:  使用Reddit OAuth2和Spring Security进行身份验证
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将使用Spring Security OAuth进行Reddit API身份验证。

## 2. Maven配置

首先，为了使用Spring Security OAuth-我们需要在pom.xml中添加以下依赖(当然还包括你可能使用的任何其他Spring依赖)：

```xml
<dependency>
    <groupId>org.springframework.security.oauth</groupId>
    <artifactId>spring-security-oauth2</artifactId>
    <version>2.0.6.RELEASE</version>
</dependency>
```

## 3. 配置OAuth2客户端

接下来，让我们配置OAuth2客户端OAuth2RestTemplate，以及所有与身份验证相关的属性的reddit.properties文件：

```java
@Configuration
@EnableOAuth2Client
@PropertySource("classpath:reddit.properties")
protected static class ResourceConfiguration {

    @Value("${accessTokenUri}")
    private String accessTokenUri;

    @Value("${userAuthorizationUri}")
    private String userAuthorizationUri;

    @Value("${clientID}")
    private String clientID;

    @Value("${clientSecret}")
    private String clientSecret;

    @Bean
    public OAuth2ProtectedResourceDetails reddit() {
        AuthorizationCodeResourceDetails details = new AuthorizationCodeResourceDetails();
        details.setId("reddit");
        details.setClientId(clientID);
        details.setClientSecret(clientSecret);
        details.setAccessTokenUri(accessTokenUri);
        details.setUserAuthorizationUri(userAuthorizationUri);
        details.setTokenName("oauth_token");
        details.setScope(Arrays.asList("identity"));
        details.setPreEstablishedRedirectUri("http://localhost/login");
        details.setUseCurrentUri(false);
        return details;
    }

    @Bean
    public OAuth2RestTemplate redditRestTemplate(OAuth2ClientContext clientContext) {
        OAuth2RestTemplate template = new OAuth2RestTemplate(reddit(), clientContext);
        AccessTokenProvider accessTokenProvider = new AccessTokenProviderChain(
                Arrays.<AccessTokenProvider> asList(
                        new MyAuthorizationCodeAccessTokenProvider(),
                        new ImplicitAccessTokenProvider(),
                        new ResourceOwnerPasswordAccessTokenProvider(),
                        new ClientCredentialsAccessTokenProvider())
        );
        template.setAccessTokenProvider(accessTokenProvider);
        return template;
    }
}
```

以及“reddit.properties”：

```properties
clientID=xxxxxxxx
clientSecret=xxxxxxxx
accessTokenUri=https://www.reddit.com/api/v1/access_token
userAuthorizationUri=https://www.reddit.com/api/v1/authorize
```

你可以通过从[https://www.reddit.com/prefs/apps/](https://www.reddit.com/prefs/apps/)创建Reddit应用程序来获取自己的密钥代码。

我们将使用OAuth2RestTemplate来：

1. 获取访问远程资源所需的访问令牌
2. 获取访问令牌后访问远程资源

还请注意我们如何将范围“identity”添加到 RedditOAuth2ProtectedResourceDetails，以便我们稍后可以检索用户帐户信息。

## 4. 自定义AuthorizationCodeAccessTokenProvider

Reddit OAuth2实现与标准略有不同，因此，我们实际上需要覆盖其中的某些部分，而不是简单地扩展AuthorizationCodeAccessTokenProvider。

Github上有跟踪改进的问题，这些改进将使这不再是必要的，但这些问题尚未完成。

Reddit所做的非标准事情之一是-当我们重定向用户并提示他使用Reddit进行身份验证时，我们需要在重定向URL中包含一些自定义参数。更具体地说，如果我们要求Reddit提供永久访问令牌，我们需要添加一个参数“duration”，其值为“permanent”。

因此，在扩展AuthorizationCodeAccessTokenProvider之后，我们在getRedirectForAuthorization()方法中添加了此参数：

```java
requestParameters.put("duration", "permanent");
```

你可以从[这里](https://github.com/Baeldung/reddit-app/tree/master/reddit-common/src/main/java/org/baeldung/security)查看完整的源代码。

## 5. ServerInitializer

下一步，让我们创建自定义的ServerInitializer。

我们需要添加一个id为oauth2ClientContextFilter的过滤器Bean，这样我们就可以使用它来存储当前上下文：

```java
public class ServletInitializer extends AbstractDispatcherServletInitializer {

    @Override
    protected WebApplicationContext createServletApplicationContext() {
        AnnotationConfigWebApplicationContext context =
                new AnnotationConfigWebApplicationContext();
        context.register(WebConfig.class, SecurityConfig.class);
        return context;
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/" };
    }

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        return null;
    }

    @Override
    public void onStartup(ServletContext servletContext) throws ServletException {
        super.onStartup(servletContext);
        registerProxyFilter(servletContext, "oauth2ClientContextFilter");
        registerProxyFilter(servletContext, "springSecurityFilterChain");
    }

    private void registerProxyFilter(ServletContext servletContext, String name) {
        DelegatingFilterProxy filter = new DelegatingFilterProxy(name);
        filter.setContextAttribute(
                "org.springframework.web.servlet.FrameworkServlet.CONTEXT.dispatcher");
        servletContext.addFilter(name, filter).addMappingForUrlPatterns(null, false, "/*");
    }
}
```

## 6. MVC配置

现在让我们看一下我们的简单Web应用程序的MVC配置：

```java
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = { "cn.tuyucheng.taketoday.web" })
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public static PropertySourcesPlaceholderConfigurer
    propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }

    @Bean
    public ViewResolver viewResolver() {
        InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
        viewResolver.setPrefix("/WEB-INF/jsp/");
        viewResolver.setSuffix(".jsp");
        return viewResolver;
    }

    @Override
    public void configureDefaultServletHandling(
            DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }

    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/resources/**").addResourceLocations("/resources/");
    }

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/home.html");
    }
}
```

## 7. 安全配置

接下来让我们看一下主要的Spring Security配置：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(AuthenticationManagerBuilder auth)
            throws Exception {
        auth.inMemoryAuthentication();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.anonymous().disable()
                .csrf().disable()
                .authorizeRequests()
                .antMatchers("/home.html").hasRole("USER")
                .and()
                .httpBasic()
                .authenticationEntryPoint(oauth2AuthenticationEntryPoint());
    }

    private LoginUrlAuthenticationEntryPoint oauth2AuthenticationEntryPoint() {
        return new LoginUrlAuthenticationEntryPoint("/login");
    }
}
```

注意：我们添加了一个简单的安全配置，重定向到“/login”，从中获取用户信息并加载身份验证，如下一节所述。

## 8. RedditController

现在让我们看一下控制器RedditController。

我们使用方法redditLogin()从Reddit帐户获取用户信息并从中加载Authentication，如下例所示：

```java
@Controller
public class RedditController {

    @Autowired
    private OAuth2RestTemplate redditRestTemplate;

    @RequestMapping("/login")
    public String redditLogin() {
        JsonNode node = redditRestTemplate.getForObject("https://oauth.reddit.com/api/v1/me", JsonNode.class);
        UsernamePasswordAuthenticationToken auth =
                new UsernamePasswordAuthenticationToken(node.get("name").asText(),
                        redditRestTemplate.getAccessToken().getValue(),
                        Arrays.asList(new SimpleGrantedAuthority("ROLE_USER")));

        SecurityContextHolder.getContext().setAuthentication(auth);
        return "redirect:home.html";
    }
}
```

这个看似简单的方法有一个有趣的细节-reddit模板**在执行任何请求之前检查访问令牌是否可用**；如果没有可用的令牌，它会获取一个令牌。

接下来，我们将信息呈现给我们非常简单的前端。

## 9. home.jsp

最后让我们看一下home.jsp-显示从用户的Reddit帐户检索到的信息：

```html
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"%>
<html>
<body>
    <h1>Welcome, <small><sec:authentication property="principal.username" /></small></h1>
</body>
</html>
```

## 10. 总结

在这篇介绍性文章中，我们探讨了如何使用Reddit OAuth2 API进行身份验证并在简单的前端显示一些非常基本的信息。

现在我们已经通过了身份验证，我们将在本系列的下一篇文章中探索使用Reddit API做更多有趣的事情。