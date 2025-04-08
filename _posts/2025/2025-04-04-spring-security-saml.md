---
layout: post
title:  使用Spring Boot和Spring Security的SAML
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将使用Spring Boot设置SAML2，SAML是一种长期受信任的实现安全应用程序的技术。设置SAML需要多方配置，因此过程有些复杂。我们必须在服务提供商和身份提供商之间来回切换几次，因此需要耐心按照分步指南进行操作。让我们深入了解创建工作应用程序的每个步骤。

## 2. 设置服务提供商(Sp)

在我们的例子中，Spring Boot应用程序是我们的服务提供者。让我们使用[Spring Security](https://www.baeldung.com/spring-boot-security-autoconfiguration)、Spring MVC和OpenSAML依赖设置一个[Spring Boot](https://www.baeldung.com/spring-boot-start)应用程序。一个关键依赖是Spring Security SAML2，Spring Security框架中的新SAML2支持通过单个依赖提供：

```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-saml2-service-provider</artifactId>
</dependency>
```

### 2.1 SAML配置

现在让我们在application.yml中添加SAML2所需的配置，**最重要的配置是来自身份提供者的元数据**。虽然我们已将metadata-uri添加到配置中以完成配置，但目前它还不可用：

```yaml
spring:
    security:
        saml2:
            relyingparty:
                registration:
                    okta:
                        signing:
                            credentials:
                                - private-key-location: classpath:local.key
                                  certificate-location: classpath:local.crt
                        singlelogout:
                            binding: POST
                            response-url: "{baseUrl}/logout/saml2/slo"
                        assertingparty:
                            metadata-uri: "classpath:metadata/metadata-idp.xml"
```

singlelogout配置定义了成功注销后身份提供者将重定向到的端点。此外，signing credentials配置添加了我们的应用程序将用于向身份提供者签署注销请求的密钥和证书。我们使用[OpenSSL](https://www.baeldung.com/openssl-self-signed-cert)工具生成local.key和local.crt文件：

```shell
openssl req -newkey rsa:2048 -nodes -keyout local.key -x509 -days 365 -out local.crt
```

### 2.2 代码中的安全配置

在此步骤中，**让我们将[安全过滤器](https://www.baeldung.com/spring-security-custom-filter)添加到过滤器链中，此过滤器将身份提供者元数据添加到我们的安全上下文中**。除此之外，我们还在http对象上添加saml2Login()和saml2Logout()方法调用，以分别启用登录和注销：

```java
Saml2MetadataFilter filter = new Saml2MetadataFilter(relyingPartyRegistrationResolver, new OpenSamlMetadataResolver());

http.csrf(AbstractHttpConfigurer::disable).authorizeHttpRequests(authorize -> authorize.anyRequest()
    .authenticated())
    .saml2Login(withDefaults())
    .saml2Logout(withDefaults())
    .addFilterBefore(filter, Saml2WebSsoAuthenticationFilter.class);

return http.build();
```

我们使用withDefaults()方法来配置saml2Login和saml2Logout的默认行为，这就是使用Spring Boot平台的真正威力，只需几行代码即可完成我们所有的SAML2应用程序设置。接下来，我们将在Okta中设置我们的身份提供者。

## 3. 设置身份提供者(IdP)

在此步骤中，让我们将Okta设置为我们的身份提供者。**身份提供者是对我们的用户进行身份验证并生成SAML断言的一方**，然后，将此SAML断言传达回我们的用户代理。用户代理将此SAML断言提供给服务提供商进行身份验证，服务提供商从身份提供者处验证它并允许用户访问其资源。

在[Okta开发者帐户](https://developer.okta.com/signup/)上注册并登录后，我们会看到一个带有左侧边栏的屏幕。在此侧边栏中，让我们导航到“Applications”页面并启动我们的SAML应用程序集成过程：

![](/assets/images/2025/springsecurity/springsecuritysaml01.png)

### 3.1 创建应用集成

接下来，让我们点击“Create App Integration”以打开“Create a new app integration”对话框并选择SAML2.0：

![](/assets/images/2025/springsecurity/springsecuritysaml02.png)

我们将点击“Next”以启动“Create SAML Integration”向导。这是一个三步向导，让我们完成每个步骤以完成我们的设置。

### 3.2 常规设置

我们在此步骤中输入我们的应用程序名称为“Baeldung Spring Security SAML2 App”：

![](/assets/images/2025/springsecurity/springsecuritysaml03.png)

### 3.3 配置SAML

**现在让我们为SAML应用配置最重要的详细信息**，在这里，我们将在身份提供者中注册单点登录URL。因此，身份提供者接收来自此URL的SSO请求。Audience URI是SAML断言接收者的标识符，这将添加到生成并发送回用户代理的SAML断言中：

![](/assets/images/2025/springsecurity/springsecuritysaml04.png)

我们示例中的Audience URI是http://localhost:8080/saml2/service-provider-metadata/okta，而单点登录URL是http://localhost:8080/login/saml2/sso/okta

### 3.4 高级设置和用户属性

现在让我们展开“Show Advanced Settings”部分，为了启用单点注销功能，我们需要在此处上传local.crt证书，**这与我们在服务提供商application.yml中配置的证书相同**，服务提供商应用使用此证书签署任何注销请求。

![](/assets/images/2025/springsecurity/springsecuritysaml05.png)

此外，让我们将“Single Logout URL”配置为http://localhost:8080/logout/saml2/slo。

最后，我们还配置了emailAddress和firstName的属性语句：

> emailAddress -> Unspecified -> user.email
> firstName -> Unspecified -> user.firstName

在进入“Next”之前，让我们使用此步骤底部的“Preview the SAML Assertion”链接预览SAML断言。

![](/assets/images/2025/springsecurity/springsecuritysaml06.png)

### 3.5 最终反馈

在反馈步骤中，让我们选择选项“I’m an Okta customer adding an internal app”。

![](/assets/images/2025/springsecurity/springsecuritysaml07.png)

### 3.6 SAML设置说明

完成反馈步骤后，我们将进入应用程序的“Sign On”选项卡。在此屏幕上，让我们点击右侧边栏底部的“View SAML setup instructions”链接：

![](/assets/images/2025/springsecurity/springsecuritysaml08.png)

这将把我们带到一个包含有关身份提供者的必要信息的页面，让我们转到最后一个包含IdP元数据的字段：

![](/assets/images/2025/springsecurity/springsecuritysaml09.png)

我们复制此元数据并将其保存为服务提供商应用程序resources/metadata文件夹中的metadata-idp-okta.xml，从而满足application.yml中metadata_uri的要求：

![](/assets/images/2025/springsecurity/springsecuritysaml10.png)

这完成了“服务提供商”和“身份提供商”的设置，接下来，我们将创建一个用户并将其分配给Okta开发者帐户中的应用程序。

## 4. 创建用户主体

让我们登录Okta开发者帐户并导航到左侧边栏“Directory”部分下的“People”页面，在这里，我们将填写“Add Person”表格来创建用户。有时，可能需要刷新“People”页面才能在列表中看到新用户：

![](/assets/images/2025/springsecurity/springsecuritysaml11.png)

在这种情况下，我们会自动激活用户。通常，你可能希望发送激活电子邮件或切换开关，让用户在第一次尝试时更改分配的密码。

最后，我们单击“Assign”并按照几个步骤将新用户分配给我们的SAML应用。

![](/assets/images/2025/springsecurity/springsecuritysaml12.png)

## 5. 测试应用程序

现在，我们已准备好测试我们的应用程序，让我们启动Spring Boot应用程序并在http://localhost:8080打开应用程序的默认端点。这将带我们进入登录屏幕：

![](/assets/images/2025/springsecurity/springsecuritysaml13.png)

接下来，我们进入成功登录的页面。除了用户名之外，我们还在此页面上显示获取的用户属性，例如emailAddress和firstName：

![](/assets/images/2025/springsecurity/springsecuritysaml14.png)

至此，设置SAML应用的整个过程就结束了。但在离开之前，我们再检查最后一件事：“Logout”按钮。

首先，我们需要将属性<OKTA-ID\>设置为你的okta标识符(你可以在URL中看到)：

```yaml
spring:
    security:
        saml2:
            relyingparty:
                registration:
                    okta:
                        # ...
                        singlelogout:
                            url: https://dev-<OKTA-ID>.okta.com/app/dev-56617222_springbootsaml_1/exk8b5jr6vYQqVXp45d7/slo/saml
                            binding: POST
                            response-url: "{baseUrl}/logout/saml2/slo"
```

然后，我们将能够从针对已登录用户的所有SAML会话中注销：

![](/assets/images/2025/springsecurity/springsecuritysaml15.png)

## 6. 总结

在本文中，我们了解了Spring Boot Security SAML2支持。尽管SAML2是一项复杂的技术，但它是大型企业的首选。一旦我们了解了SAML2，利用它提供的强大功能就会非常有趣。除了保护我们的应用程序之外，SAML2还允许我们使用SSO，避免记住数十个应用程序的多个用户名和密码。