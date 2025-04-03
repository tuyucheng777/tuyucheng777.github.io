---
layout: post
title:  在Spring Boot中使用AzureAD对用户进行身份验证
category: spring-security
copyright: spring-security
excerpt: AzureAD
---

## 1. 简介

在本教程中，我们将展示如何轻松地使用AzureAD作为Spring Boot应用程序的身份提供者。

## 2. 概述

Microsoft的AzureAD是一款全面的身份管理产品，全球许多组织都在使用它。它支持多种登录机制和控件，为组织应用程序组合中的用户提供单点登录体验。

**此外，秉承微软的初衷，AzureAD可以与现有的Active Directory安装很好地集成，许多组织已经将其用于企业网络中的[身份和访问管理](https://www.baeldung.com/cs/iam-security)**。这使管理员可以向现有用户授予对应用程序的访问权限，并使用他们已经习惯的相同工具来管理他们的权限。

## 3. 集成AzureAD

**从基于Spring Boot的应用程序角度来看，AzureAD充当符合OIDC的身份提供者**，这意味着我们只需配置所需的属性和依赖即可将其与Spring Security一起使用。

为了说明AzureAD集成，我们将实现一个[机密客户端](https://www.rfc-editor.org/rfc/rfc6749#section-2.1)，其中访问代码交换的授权代码在服务器端进行。**此流程永远不会将访问令牌暴露给用户的浏览器，因此它被认为比公共客户端替代方案更安全**。

## 4. Maven依赖

我们首先添加基于Spring Security的Web MVC应用程序所需的Maven依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-client</artifactId>
    <version>3.1.5</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.1.5</version>
</dependency>
```

这些依赖的最新版本可在Maven Central上找到：

- [spring-boot-stater-oauth2-client](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-oauth2-client)
- [spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)

## 5. 配置属性

接下来，我们将添加用于配置客户端所需的[Spring Security](https://www.baeldung.com/learn-spring-security-course)属性。**一个好的做法是将这些属性放在专用的Spring Profile中，这样随着应用程序的增长，维护会变得更容易一些**。我们将此Profile命名为azuread，这使其用途明确。因此，我们将在application-azuread.yml文件中添加相关属性：

```yaml
spring:
    security:
        oauth2:
            client:
                provider:
                    azure:
                        issuer-uri: https://login.microsoftonline.com/your-tenant-id-comes-here/v2.0
                registration:
                    azure-dev:
                        provider: azure
                        #client-id: externally provided
                        #client-secret: externally provided         
                        scope:
                            - openid
                            - email
                            - profile
```

在provider部分，我们定义一个Azure提供程序。**AzureAD支持OIDC标准端点发现机制，因此我们需要配置的唯一属性是issuer-uri**。

此属性具有双重用途：首先，它是客户端将发现资源名称附加到其上以获取要下载的实际URL的基本URI。其次，它还用于检查[JSON Web Token](https://www.rfc-editor.org/rfc/rfc7519)(JWT)的真实性。例如，身份提供者创建的JWT的iss声明必须与issuer-uri值相同。

对于AzureAD，issuer-uri始终采用https://login.microsoftonline.com/my-tenant-id/v2.0形式，其中my-tenant-id是租户的标识符。

在registration部分，我们定义使用先前定义的提供程序的azure-dev客户端。我们还必须通过client-id和client -secret属性提供客户端凭据，在本文后面介绍如何在Azure中注册此应用程序时，我们将回顾这些属性。

最后，scope属性定义了此客户端将包含在授权请求中的范围集。在这里，我们请求profile范围，这允许此客户端应用程序请求标准[userinfo](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo)端点。此端点返回存储在AzureAD用户目录中的一组可配置信息，这些可能包括用户的首选语言和区域设置数据等。

## 6. 客户端注册

如前所述，**我们需要在AzureAD中注册我们的客户端应用程序以获取所需属性client-id和client-secret的实际值**。假设我们已经有一个Azure帐户，第一步是登录到Web控制台并使用左上角的菜单选择Azure Active Directory服务页面：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers01.png)

在Overview部分中，我们可以获取需要在issuer-uri配置属性中使用的租户标识符。接下来，我们点击App Registrations，它将带我们进入现有应用程序列表，然后点击“New Registration”，它将显示客户端注册表。在这里，我们必须提供三条信息：

- 应用程序名称
- 支持的账户类型
- 重定向URI

让我们详细地介绍一下每一项。

### 6.1 应用程序名称

**我们在此处输入的值将在身份验证过程中显示给最终用户**，因此，我们应该选择一个对目标受众有意义的名称。让我们使用一个非常普通的名字：“Tuyucheng Test App”：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers02.png)

不过，我们不需要太担心名称是否正确。**AzureAD允许我们随时更改它，而不会影响已注册的应用程序**。需要注意的是，虽然这个名字不必是唯一的，但让多个应用程序使用相同的显示名称并不是一个聪明的主意。

### 6.2 支持的账户类型

在这里，我们有几个选项可以根据应用程序的目标受众进行选择。**对于供组织内部使用的应用程序，第一个选项(“accounts in this organizational directory only”)通常是我们想要的**，这意味着即使应用程序可以从互联网访问，也只有组织内的用户才能登录：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers03.png)

其他可用选项增加了接受来自其他AzureAD支持的目录的用户的功能，例如使用Office 365的任何学校或组织以及在Skype和/或Xbox上使用的个人帐户。

虽然不太常见，但我们也可以稍后更改此设置，但正如文档中所述，用户在进行此更改后可能会收到错误消息。

### 6.3 重定向URI

最后，我们需要提供一个或多个重定向URI，这些URI是可接受的授权流程目标。我们必须选择与URI关联的“platform”，该平台对应于我们要注册的应用程序类型：

- Web：授权码与访问令牌交换发生在后端
- SPA：授权码与访问令牌交换发生在前端
- 公共客户端：用于桌面和移动应用程序

在我们的例子中，我们将选择第一个选项，因为这是我们进行用户身份验证所需要的。

至于URI，我们将使用值http://localhost:8080/login/oauth2/code/azure-dev，此值来自Spring Security的OAuth回调控制器使用的路径，默认情况下，该控制器期望响应代码位于/login/oauth2/code/{registration-name}。这里，{registration-name}必须与配置的registration部分中存在的键之一匹配，在我们的例子中是azure-dev。

**同样重要的是，AzureAD要求对这些URI使用HTTPS，但localhost除外**，这样无需设置证书即可进行本地开发。稍后，当我们转移到目标部署环境(例如Kubernetes集群)时，我们可以添加其他URI。

请注意，此密钥的值与AzureAD的注册名称没有直接关系，但使用与其使用位置相关的名称是有意义的。

### 6.4 添加客户端密钥

一旦我们按下初始注册表上的注册按钮，我们就会看到客户的信息页面：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers04.png)

Essentials部分左侧有应用程序ID，与属性文件中的client-id属性相对应。要生成新的客户端机密，我们现在单击“Add a certificate or secret”，这将带我们进入“Certificates & Secrets”页面。接下来，我们选择“Client Secrets”选项卡，然后单击“New client secret”以打开机密创建表单：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers05.png)

在这里，我们将为该机密指定一个描述性名称并定义其到期日期。我们可以从预配置的持续时间中选择一个，也可以选择自定义选项，这样我们就可以定义开始日期和结束日期。

截至撰写本文时，客户端机密最多会在两年后过期，**这意味着我们必须实施机密轮换程序，最好使用Terraform等自动化工具**。两年似乎很长，但在企业环境中，应用程序运行多年后才被替换或更新的情况相当常见。

一旦我们点击Add，新创建的密钥就会出现在客户端凭据列表中：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers06.png)

**我们必须立即将秘密值复制到安全的地方，因为一旦我们离开此页面，它将不会再次显示**。在我们的例子中，我们将直接将值复制到应用程序的属性文件中的client-secret属性下。

无论如何，我们必须记住这是一个敏感值，将应用程序部署到生产环境时，此值通常会通过某种动态机制提供，例如Kubernetes机密。

## 7. 应用程序代码

我们的测试应用程序有一个控制器，它处理对根路径的请求，记录有关传入身份验证的信息，并将请求转发到[Thymeleaf](https://www.baeldung.com/thymeleaf-in-spring-mvc)视图。在那里，它将呈现一个包含有关当前用户信息的页面。

实际控制器的代码很简单：

```java
@Controller
@RequestMapping("/")
public class IndexController {

    @GetMapping
    public String index(Model model, Authentication user) {
        model.addAttribute("user", user);
        return "index";
    }
}
```

视图代码使用user模型属性创建一个漂亮的表，其中包含有关身份验证对象和所有可用声明的信息。

## 8. 运行测试应用程序

所有部分都准备就绪后，我们现在可以运行该应用程序了。**由于我们使用了具有AzureAD属性的特定Profile，因此我们需要激活它**。当通过Spring Boot的Maven插件运行应用程序时，我们可以使用spring-boot.run.profiles属性执行此操作：

```shell
mvn -Dspring-boot.run.profiles=azuread spring-boot:run
```

现在，我们可以打开浏览器并访问http://localhost:8080，Spring Security将检测到此请求尚未经过身份验证，并将我们重定向到AzureAD的通用登录页面：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers07.png)

具体的登录顺序将根据组织的设置而有所不同，但通常包括填写用户名或电子邮件并提供密码。如果已配置，它还可以请求第二个身份验证因素。但是，如果我们当前在同一浏览器中登录到同一AzureAD租户中的另一个应用程序，它将跳过登录顺序-毕竟，这就是单点登录的意义所在。

第一次访问我们的应用程序时，AzureAD还会显示应用程序的同意书：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers08.png)

虽然这里没有介绍，但AzureAD支持自定义登录UI的多个方面，包括特定于语言环境的自定义。此外，可以完全绕过授权表单，这在授权内部应用程序时很有用。

一旦我们授予权限，我们将看到应用程序的主页，部分显示如下：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers09.png)

我们可以看到，我们已经可以访问用户的基本信息，包括他/她的姓名、电子邮件，甚至获取他/她的照片的URL。不过，有一个烦人的细节：Spring 为用户名选择的值不太友好。

让我们看看如何改进这一点。

## 9. 用户名映射

Spring Security使用Authentication接口来表示经过身份验证的Principal，此接口的具体实现必须提供getName()方法，该方法返回一个值，该值通常用作身份验证域中用户的唯一标识符。

**当使用基于JWT的身份验证时，Spring Security将默认使用标准sub声明值作为Principal的名称**。查看声明，我们发现AzureAD使用内部标识符填充此字段，这不适合显示目的。

幸运的是，这种情况有一个简单的解决方法。我们要做的就是选择一个可用的属性，并将其名称放在提供程序的user-name-attribute属性上：

```yaml
spring:
    security:
        oauth2:
            client:
                provider:
                    azure:
                        issuer-uri: https://login.microsoftonline.com/xxxxxxxxxxxxx/v2.0
                        user-name-attribute: name
#... other properties omitted
```

这里，我们选择了name声明，因为它对应于用户的完整姓名。另一个合适的候选者是email属性，如果我们的应用程序需要将其值用作某些数据库查询的一部分，那么这可能是一个不错的选择。

我们现在可以重新启动应用程序并查看此更改的影响：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers10.png)

现在好多了。

## 10. 检索组成员身份

**仔细检查可用的声明后发现，没有关于用户组成员身份的信息**。Authentication中唯一可用的GrantedAuthority值是与请求的范围相关联的值，包含在客户端配置中。

如果我们只需要限制组织成员的访问权限，这可能就足够了。但是，通常我们会根据分配给当前用户的角色授予不同的访问级别。此外，将这些角色映射到AzureAD组允许重用可用流程，例如用户入职和/或重新分配。

**为了实现这一点，我们必须指示AzureAD将组成员身份包含在我们在授权流程中收到的idToken中**。

首先，我们必须转到我们的应用程序页面并在右侧菜单上选择Token Configuration。接下来，我们点击Add groups claim，这将打开一个对话框，我们将在其中定义此声明类型所需的详细信息：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers11.png)

我们将使用常规AzureAD组，因此我们将选择第一个选项(“Security Groups”)，此对话框还针对每种受支持的令牌类型提供了其他配置选项，我们暂时保留默认值。

一旦我们点击“Save”，应用程序的声明列表将显示groups声明：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers12.png)

现在，我们可以回到我们的应用程序来查看此配置的效果：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers13.png)

## 11. 将组映射到Spring权限

组声明包含与用户分配的组相对应的对象标识符列表，但是，Spring不会自动将这些组映射到GrantedAuthority实例。

这样做需要自定义OidcUserService，如Spring Security的[文档](https://docs.spring.io/spring-security/reference/5.7.7/servlet/oauth2/login/advanced.html#oauth2login-advanced-map-authorities-oauth2userservice)中所述。我们的实现([可在线获取](https://github.com/eugenp/tutorials/blob/master/spring-security-modules/spring-security-azuread/src/main/java/com/baeldung/security/azuread/config/JwtAuthorizationConfiguration.java))使用外部映射通过附加权限“enrich”标准OidcUser实现，我们使用@ConfigurationProperties类来放置所需的信息：

- 我们将从中获取组列表的声明名称(“groups”)
- 从此提供商映射的权限前缀
- 对象标识符到GrantedAuthority值的映射

使用组到列表映射策略使我们能够应对想要使用现有组的情况，它还有助于使应用程序的角色集与组分配策略脱钩。

典型配置如下：

```yaml
tuyucheng:
    jwt:
        authorization:
            group-to-authorities:
                "ceef656a-fca9-49b6-821b-xxxxxxxxxxxx": TUYUCHENG_RW
                "eaaecb69-ccbc-4143-b111-xxxxxxxxxxxx": TUYUCHENG_RO,TUYUCHENG_ADMIN
```

对象标识符在“Groups”页面上可用：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers14.png)

一旦完成所有这些映射并重新启动应用程序，我们就可以测试我们的应用程序，这是我们为属于两个组的用户获得的结果：

![](/assets/images/2025/springsecurity/springbootazureadauthenticateusers15.png)

它现在拥有与映射组相对应的三个新权限。

## 12. 总结

在本文中，我们展示了如何使用AzureAD和Spring Security来验证用户身份，包括演示应用程序所需的配置步骤。