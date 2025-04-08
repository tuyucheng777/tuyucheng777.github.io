---
layout: post
title:  Spring Security中的一次性令牌登录指南
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

为网站提供流畅的登录体验需要一种微妙的平衡。一方面，我们希望不同计算机水平的用户都能尽快完成登录。另一方面，我们需要确保访问我们系统的人的身份，否则可能会发生灾难性的安全事故。

**在本教程中，我们将展示如何在基于Spring Boot的应用程序中使用一次性令牌登录**。此机制在易用性和安全性之间取得了良好的平衡，并且从Spring Boot版本3.4开始，在使用[Spring Security 6.4或更高版本](https://docs.spring.io/spring-security/reference/whats-new.html)时即可开箱即用。

## 2. 什么是一次性令牌登录？

在计算机应用程序中识别用户的传统方法是提供一个表单，让用户提供用户名和密码。现在，如果用户忘记了密码怎么办？**常见的方法是提供“忘记密码”按钮**。

当用户点击此按钮时，后端会向用户发送一条消息，其中包含一个限时令牌，允许用户重新定义其密码。

**然而，对于一系列应用程序来说，用户不需要经常访问网站和/或费心保存密码**。在这些情况下，用户往往会不断使用重置密码功能，这会导致用户感到沮丧，有时甚至会导致客户支持电话发怒。以下是属于此类的一些应用程序：

- 社区场所(俱乐部、学校、教堂、游戏场)
- 文件分发/签名服务
- 弹出式营销网站

**相反，一次性令牌登录(简称OTT)机制的工作方式如下**：

1. 用户告知其用户名，通常与其电子邮件地址相对应
2. 系统生成一个有时间限制的令牌，并使用带外机制发送，可以是电子邮件、短信、移动通知或类似方式
3. 用户在电子邮件/消息应用程序中打开消息并点击提供的链接，其中包含一次性令牌
4. 用户的设备浏览器打开该链接，将其带回系统的OTT登录位置
5. 系统检查链接中嵌入的令牌值。如果有效，则授予访问权限，用户可以继续。或者，显示令牌提交表单，提交后，完成登录过程

## 3. 何时应使用OTT？

**在考虑给定应用程序的一次性登录机制之前，最好先了解一下它的优缺点**：

|                 优点                 |                     缺点                     |
| :------------------------------------: | :--------------------------------------------: |
|  无需管理用户密码，这也消除了安全风险  | 基于单因素的身份验证，至少从应用程序的端点开始 |
| 即使不懂技术的用户也可以轻松使用和理解 |                 易受中间人攻击                 |

我们现在可能会想：为什么不使用社交登录？从技术角度来看，社交登录通常基于OAuth2/OIDC，比OTT更安全。

**然而，启用它需要更多的操作工作**(例如，为每个提供商请求和维护客户端ID)，并且考虑到人们对共享个人数据的意识的提高，可能会导致参与度下降。

## 4. 使用Spring Boot和Spring Security实现OTT

让我们创建一个简单的Spring Boot应用程序，该应用程序使用自3.4版本以来提供的OTT支持。**与往常一样，我们首先添加所需的Maven依赖**：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.4.1<version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>3.4.1<version>
</dependency>
```

这些依赖的最新版本可在Maven Central上找到：

- [spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)
- [spring-boot-starter-security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)

## 5. OTT配置

在当前版本中，为应用程序启用OTT需要我们提供SecurityFilterChain Bean：

```java
@Bean
SecurityFilterChain ottSecurityFilterChain(HttpSecurity http) throws Exception {
    return http
            .authorizeHttpRequests(ht -> ht.anyRequest().authenticated())
            .formLogin(withDefaults())
            .oneTimeTokenLogin(withDefaults())
            .build();
}
```

**这里的关键点是使用6.4版本中作为DSL配置的一部分引入的新oneTimeTokenLogin()方法**，与往常一样，此方法允许我们自定义机制的所有方面。但是，在我们的例子中，我们仅使用Customizer.withDefaults()来接收默认值。

另外，请注意，我们在配置中添加了formLogin()。**如果没有它，Spring Security将默认使用基本身份验证，这无法与OTT很好地兼容**。

最后，在authorizeHttpRequests()部分，我们刚刚添加了一个要求对所有请求进行身份验证的配置。

## 6. 发送令牌

**OTT机制没有内置方法来实现向用户实际交付令牌**，如[文档中所述](https://docs.spring.io/spring-security/reference/servlet/authentication/onetimetoken.html#sending-token-to-user)，这是一个经过深思熟虑的设计决定，因为实现此功能的方法实在太多了。

相反，OTT将此责任委托给应用程序代码，**该代码必须公开实现OneTimeTokenGenerationSuccessHandler接口的Bean**。或者，我们可以直接通过配置DSL传递此接口的实现。

此接口只有一个方法handle()，该方法接收当前Servlet请求、响应以及最重要的OneTimeToken对象。后者具有以下属性：

- tokenValue：我们需要发送给用户的生成的令牌
- username：已获知的用户名
- expiresAt：生成的令牌过期的时间

典型的实现将经历以下步骤：

1. 使用提供的用户名作为键来查找所需的配送详情。例如，这些详情可能包括电子邮件地址或电话号码以及用户的区域设置
2. 构建一个URL，将用户引导至OTT登录页面
3. 准备一条带有OTT链接的消息并发送给用户
4. 向客户端发送重定向响应，将浏览器发送到OTT登录页面

在我们的实现中，我们选择将与步骤1到3相关的职责拆分给专用的OttSenderService。

对于步骤 4，我们将重定向详细信息委托给Spring Security的RedirectOneTimeTokenGenerationSuccessHandler。这是最终的实现：

```java
public class OttLoginLinkSuccessHandler implements OneTimeTokenGenerationSuccessHandler {
    private final OttSenderService senderService;
    private final OneTimeTokenGenerationSuccessHandler redirectHandler = new RedirectOneTimeTokenGenerationSuccessHandler("/login/ott");

    // ... constructor omitted

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                       OneTimeToken oneTimeToken) throws IOException, ServletException {
        senderService.sendTokenToUser(oneTimeToken.getUsername(),
                oneTimeToken.getTokenValue(), oneTimeToken.getExpiresAt());
        redirectHandler.handle(request, response, oneTimeToken);
    }
}
```

请注意传递给RedirectOneTimeTokenGenerationSuccessHandler的“/login/ott”构造函数参数，这对应于令牌提交表单的默认位置，可以使用OTT DSL将其配置为其他位置。

至于OttSenderService，我们将使用一个虚假的发送方实现，将令牌存储在由用户名索引的Map中并记录其值：

```java
public class FakeOttSenderService implements OttSenderService {
    private final Map<String,String> lastTokenByUser = new HashMap<>();

    @Override
    public void sendTokenToUser(String username, String token, Instant expiresAt) {
        lastTokenByUser.put(username, token);
        log.info("Sending token to username '{}'. token={}, expiresAt={}", username,token,expiresAt);
    }

    @Override
    public Optional<String> getLastTokenForUser(String username) {
        return Optional.ofNullable(lastTokenByUser.get(username));
    }
}
```

请注意，OttSenderService有一个可选方法，允许我们恢复用户名的令牌。**此方法的主要目的是简化单元测试的实现**，我们将在自动测试部分中看到。

## 7. 手动测试

**让我们通过一个简单的导航测试来检查使用OTT机制的应用程序的行为**，通过IDE或使用mvn spring-boot:run启动它后，使用你选择的浏览器并导航到http://localhost:8080。该应用程序将显示一个登录页面，其中包含接收用户名/密码的标准表单和OTT表单：

![](/assets/images/2025/springsecurity/springsecurityonetimetokenlogin01.png)

由于我们没有提供任何UserDetailsService，Spring Boot的自动配置会创建一个名为“user”的默认用户。一旦我们将其输入到OTT的表单用户名字段中并单击发送令牌按钮，我们就应该进入令牌提交表单：

![](/assets/images/2025/springsecurity/springsecurityonetimetokenlogin02.png)

现在，如果我们查看应用程序日志，我们会看到这样的消息：

```text
c.t.t.s.ott.service.FakeOttSenderService   : Sending token to username 'user'. token=a0e3af73-0366-4e26-b68e-0fdeb23b9bb2, expiresAt=...
```

**要完成登录过程，只需将令牌值复制并粘贴到表单中，然后单击“Sign In”按钮**。结果，我们将得到一个显示当前用户名的欢迎页面：

![](/assets/images/2025/springsecurity/springsecurityonetimetokenlogin03.png)

## 8. 自动化测试

测试OTT登录流程需要浏览一系列页面，因此我们将使用[Jsoup](https://www.baeldung.com/java-with-jsoup)库来帮助我们。

完整代码遵循我们在手动测试中经历的相同步骤，并在此过程中添加检查。

唯一棘手的部分是获取生成的令牌的访问权限，这就是OttSenderService接口上可用的查找方法派上用场的地方。由于我们利用了Spring Boot的测试基础架构，因此我们可以简单地将服务注入到我们的测试类中并使用它来查询令牌：

```java
@Test
void whenLoginWithOtt_thenSuccess() throws Exception {
    // ... Jsoup setup and initial navigation omitted

    var optToken = this.ottSenderService.getLastTokenForUser("user");
    assertTrue(optToken.isPresent());

    var homePage = conn.newRequest(baseUrl + tokenSubmitAction)
            .data("token", optToken.get())
            .data("_csrf",csrfToken)
            .post();

    var username = requireNonNull(homePage.selectFirst("span#current-username")).text();
    assertEquals("user",username);
}
```

## 9. 总结

在本教程中，我们描述了一次性令牌登录机制以及如何将其添加到基于Spring Boot的应用程序中。