---
layout: post
title:  在Spring Authorization Server中使用社交登录进行身份验证
category: spring-security
copyright: spring-security
excerpt: Spring Authorization Server
---

## 1. 简介

在本教程中，**我们将演示如何设置使用Spring社交登录功能的Web应用程序的后端**。我们将使用[Spring Boot](https://www.baeldung.com/spring-boot)和[OAuth 2.0](https://www.baeldung.com/spring-security-oauth)依赖，我们还将使用Google作为社交登录提供商。

## 2. 通过社交登录提供商注册

### 2.1 同意屏幕配置

在开始项目设置之前，我们需要从社交登录提供商处获取ID和密钥。考虑到我们使用Google作为提供商，让我们转到他们的[API控制台](https://console.developers.google.com/)来启动该过程。

进入[Google](https://www.baeldung.com/google-http-client) API控制台后，我们需要创建一个新项目。一旦我们选择了合适的项目名称，我们将开始获取凭据的过程：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication01.png)

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication02.png)

**接下来，我们需要设置OAuth同意屏幕**。为此，我们需要选择以下选项：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication03.png)

此操作将带来一个包含更多选项的新页面：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication04.png)

在这里，我们将选择“External”选项，以便任何拥有Google帐户的人都可以登录我们的应用程序。接下来，我们将点击“Create”按钮。

下一页“Edit app registration”要求我们介绍一些有关应用程序的信息，在右侧菜单中，我们可以看到一些使用“App Name”的示例。

在这里，我们要使用我们公司的名称。此外，我们可以添加我们公司的徽标，该徽标在示例中使用。最后，我们需要添加“User support email”。这个联系人是想要了解更多有关其同意的人将联系的人：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication05.png)

在继续操作之前，我们需要在屏幕底部添加一个电子邮件地址，此联系人将收到Google发送的有关已创建项目变更的通知(此处未显示)。

为了演示的目的，我们将以下字段留空：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication06.png)

此外，我们将继续执行接下来的步骤，而无需填写任何其他内容。

当我们完成“OAuth同意屏幕”的配置后，我们可以继续进行凭据设置。

### 2.2 凭证设置–Key和密钥

为此，我们需要选择“Credentials”选项(箭头1)。页面中间会出现一个新菜单，在该菜单中，我们将选择“CREATE CREDENTIALS”选项(箭头2)。从下拉列表中，我们将选择“OAuth client ID”选项(箭头3)：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication07.png)

接下来，我们将选择“Web application”选项，这是我们为演示而构建的：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication08.png)

选择后，页面上会出现更多元素。我们必须命名我们的应用程序，在本例中，我们将使用“Spring-Social-Login”。接下来，我们将提供URL。在本例中，我们将使用http://localhost:8080/login/oauth2/code/google：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication09.png)

填写完这些字段后，我们将导航到页面底部并单击“Create”按钮。将出现一个包含密钥和密码的弹出窗口，我们可以下载JSON或将它们保存在本地某处：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication10.png)

我们已经完成了Google的设置，如果我们想使用其他提供商，例如GitHub，我们必须遵循类似的流程。

## 3. Spring Boot项目设置

现在，让我们在添加社交登录功能之前设置项目。如开头所述，我们使用[Spring Boot](https://www.baeldung.com/spring-boot-start)。让我们转到Spring Initializr并设置一个新的[Maven](https://www.baeldung.com/maven)项目，我们将保持简单，使用Spring Web和OAuth2依赖：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication11.png)

在我们最喜欢的IDE中打开项目并启动它之后，主页应该是这样的：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication12.png)

这是Spring Security的默认登录页面，我们将在这里添加社交登录功能。

## 4. 社交登录实现

首先，我们将创建一个HomeController类，它包含两个路由，一个公共，一个私有：

```java
@GetMapping("/")
public String home() {
    return "Hello, public user!";
}

@GetMapping("/secure")
public String secured() {
    return "Hello, logged in user!";
}
```

**接下来，我们需要覆盖默认的安全配置**。为此，我们需要一个新的类SecurityConfig，我们将使用@Configuration和@EnableWebSecurity对其进行标注。在这个配置类中，我们将配置安全过滤器。

**此安全过滤器允许任何人访问主页(auth.requestMatchers("/".permitAll())以及任何其他需要经过身份验证的请求**：

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .authorizeHttpRequests(auth -> {
                    auth.requestMatchers("/").permitAll();
                    auth.anyRequest().authenticated();
                })
                .oauth2Login(Customizer.withDefaults())
                .formLogin(Customizer.withDefaults())
                .build();
    }
}
```

**此外，我们使用formLogin进行用户和密码验证，使用oauth2Login进行社交媒体登录功能**。

接下来，我们需要在applications.properties文件中添加ID和密钥：

```properties
spring.security.oauth2.client.registration.google.client-id = REPLACE_ME
spring.security.oauth2.client.registration.google.client-secret = REPLACE_ME
```

就这样，启动应用程序后，主页会有所不同。默认情况下，它将显示我们在公共端点中配置的消息。当我们尝试访问/secure端点时，我们将被重定向到登录页面，如下所示：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication13.png)

点击Google后我们会重定向到Google登录页面：

![](/assets/images/2025/springsecurity/springauthorizationserversocialloginauthentication14.png)

成功登录后，我们将被重定向到我们之前设置的/secure端点，并显示相应的消息。

## 5. 总结

在本文中，我们演示了如何使用OAuth2社交登录功能设置Spring Boot Maven项目。

我们使用Google API控制台实现了此功能。首先，我们设置了Google项目和应用程序。接下来，我们获取了凭证。之后，我们设置了项目，最后，我们设置了安全配置以使社交登录可用。