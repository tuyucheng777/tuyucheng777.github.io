---
layout: post
title:  将Passkeys集成到Spring Security
category: springsecurity
copyright: springsecurity
excerpt: Spring Data JPA
---

## 1. 简介

登录表单早已成为(现在仍然是)任何需要身份验证才能提供服务的Web服务的常见功能。然而，随着安全问题开始成为主流，人们清楚地认识到简单的文本密码是一个弱点：它们可能被猜出、拦截或泄露，从而导致安全事故，造成财务和/或声誉损失。

之前尝试用替代解决方案(mTLS、安全卡等)替换密码来解决此问题，但导致用户体验不佳和额外成本。

**在本教程中，我们将探索Passkeys(也称为WebAuthn)，这是一种提供密码安全替代方案的标准**。特别是，我们将演示如何使用Spring Security快速向Spring Boot应用程序添加对此身份验证机制的支持。

## 2. 什么是Passkey？

Passkeys或WebAuthn是[由W3C联盟定义](https://www.w3.org/TR/webauthn-3)的标准API，允许在Web浏览器上运行的应用程序管理公钥并将其注册以供给定服务提供商使用。

典型的注册场景如下：

1. 用户在服务上创建新帐户，初始凭证通常是熟悉的用户名/密码
2. 注册后，用户进入个人资料页面并选择“创建密钥”
3. 系统显示密钥注册表单
4. 用户在表单中填写所需信息(例如，可帮助用户稍后选择正确密钥的密钥标签)，然后提交
5. 系统将密钥保存在其数据库中，并将其与用户帐户关联。同时，此密钥的私有部分将保存在用户的设备上
6. 密钥注册已完成

**一旦密钥注册完成，用户就可以使用存储的密钥访问服务**。根据浏览器和用户设备的安全配置，登录将需要指纹扫描、解锁智能手机或类似操作。

密钥由两部分组成：浏览器发送给服务提供商的公钥和保留在本地设备上的私钥部分。

**此外，客户端API的实现确保给定的密钥只能在注册它的同一站点上使用**。

## 3. 向Spring Boot应用程序添加密钥

让我们创建一个简单的Spring Boot应用程序来测试密钥，**我们的应用程序只有一个欢迎页面，其中显示当前用户的名称和密钥注册页面的链接**。

第一步是向项目添加所需的依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <version>3.4.3</version>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
    <version>3.4.3</version>
</dependency>
<dependency>
    <groupId>com.webauthn4j</groupId>
    <artifactId>webauthn4j-core</artifactId>
    <version>0.28.5.RELEASE</version>
</dependency>
```

这些依赖的最新版本可在Maven Central上找到：

- [spring-boot-starter-web](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-web)
- [spring-boot-starter-security](https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-security)
- [webauthn4j-core](https://mvnrepository.com/artifact/com.webauthn4j/webauthn4j-core)

> **重要提示：WebAuthn支持需要Spring Boot版本3.4.0或更高版本**

## 4. Spring Security配置

从Spring Security 6.4开始(这是通过spring-boot-starter-security依赖包含的默认版本)，配置DSL通过webautn()方法原生支持密码。

```java
@Bean
SecurityFilterChain webauthnFilterChain(HttpSecurity http, WebAuthNProperties webAuthNProperties) {
    return http.authorizeHttpRequests( ht -> ht.anyRequest().authenticated())
            .formLogin(withDefaults())
            .webAuthn(webauth ->
                    webauth.allowedOrigins(webAuthNProperties.getAllowedOrigins())
                            .rpId(webAuthNProperties.getRpId())
                            .rpName(webAuthNProperties.getRpName())
            )
            .build();
}
```

这是我们通过这种配置得到的结果：

- 登录页面上将显示“login with passkey”按钮
- 注册页面位于/webauthn/register

**为了正常运行，我们必须至少向webauthn配置器提供以下配置属性**：

- allowedOrigins：网站的外部URL，必须使用HTTPS，除非使用localhost
- rpId：应用程序标识符，必须是与allowedOrigin属性的主机名部分匹配的有效域名
- rpName：浏览器在注册和/或登录过程中可能使用的用户友好名称

**但是，此配置忽略了密钥支持的一个关键方面：应用程序重新启动后，已注册的密钥将丢失**。这是因为，默认情况下，Spring Security使用基于内存的实现凭据存储，而这不适合生产使用。

稍后我们将看看如何修复这个问题。

## 5. 密钥巡查

密钥配置完成后，我们就可以快速浏览一下我们的应用程序了。使用mvn spring-boot:run或IDE启动应用程序后，我们可以打开浏览器并导航到[http://localhost:8080](http://localhost:8080/)：

![](/assets/images/2025/springsecurity/springsecurityintegratepasskeys01.png)

**Spring应用程序的标准登录页面现在将包含“Sign in with a passkey”按钮**，由于我们尚未注册任何密钥，因此我们必须使用用户名/密码凭据登录，这些凭据已在application.yaml文件中配置：alice/changeit

![](/assets/images/2025/springsecurity/springsecurityintegratepasskeys02.png)

正如预期的那样，我们现在以Alice的身份登录，现在我们可以通过点击Register PassKey链接继续进入注册页面：

![](/assets/images/2025/springsecurity/springsecurityintegratepasskeys03.png)

在这里，我们只需提供一个标签tuyucheng-demo，然后单击“Register”。**接下来发生的事情取决于设备类型(台式机、移动设备、平板电脑)和操作系统(Windows、Linux、Mac、Android)，但最终，它将导致将新密钥添加到列表中**：

![](/assets/images/2025/springsecurity/springsecurityintegratepasskeys04.png)

例如，在Windows上的Chrome中，对话框将提供创建新密钥并将其存储到浏览器的本机密码管理器中或使用操作系统上提供的[Windows Hello](https://support.microsoft.com/en-us/windows/configure-windows-hello-dae28983-8242-bb2a-d3d1-87c9d265a5f0)功能的选择。

接下来，让我们退出应用程序并尝试使用新密钥。首先，我们导航到[http://localhost:8080/logout](http://localhost:8080/logout)并确认要退出。接下来，在登录表单上，我们单击“Sign in with a passkey”，浏览器将显示一个对话框，允许你选择密钥：

![](/assets/images/2025/springsecurity/springsecurityintegratepasskeys05.png)

一旦我们选择其中一个可用密钥，设备将执行额外的身份验证质询。对于“Windows Hello”身份验证，这可以是指纹扫描、面部识别等。

如果身份验证成功，则将使用用户的私钥签署质询并将其发送到服务器，服务器将使用先前存储的公钥对其进行验证。最后，如果一切正常，则登录完成，并将像以前一样显示欢迎页面。

## 6. 密钥存储库

如前所述，Spring Security创建的默认密钥配置不为注册密钥提供持久性。**为了解决这个问题，我们需要提供以下接口的实现**：

- PublicKeyCredentialUserEntityRepository
- UserCredentialRepository

### 6.1 PublicKeyCredentialUserEntityRepository

**此服务管理PublicKeyCredentialUserEntity实例，并将标准[UserDetailsService](https://www.baeldung.com/spring-security-authentication-with-a-database#UserDetailsService)管理的用户帐户映射到用户帐户标识符**。此实体具有以下属性：

- name：账户的用户友好名称标识符
- id：用户账户的不透明标识符
- displayName：帐户名称的替代版本，用于显示目的

值得注意的是，当前的实现假定name和id在给定的身份验证域中都是唯一的。

一般来说，我们可以假设该表中的条目与标准UserDetailsService管理的帐户具有1:1的关系。

该实现[可在线获取](https://github.com/eugenp/tutorials/blob/master/spring-security-modules/spring-security-passkey/src/main/java/com/baeldung/tutorials/passkey/repository/DbPublicKeyCredentialUserEntityRepository.java)，它使用[Spring Data JDBC](https://www.baeldung.com/spring-data-jdbc-intro) Repository将这些字段存储在PASSKEY_USERS表中。

### 6.2 UserCredentialRepository

**管理CredentialRecord实例，该实例存储在注册过程中从浏览器收到的实际公钥**。此实体包括[W3C文档](https://www.w3.org/TR/webauthn-3/#credential-record)中指定的所有推荐属性以及一些其他属性：

-  userEntityUserId：拥有此凭证的PublicKeyCredentialUserEntity的标识符
-  label：此凭证的用户定义标签，在注册时分配
-  lastUsed：此凭证的最后使用日期
-  created：此凭证的创建日期

请注意，CredentialRecord与PublicKeyCredentialUserEntity具有N:1关系，这反映在Repository的方法上。例如，findByUserId()方法返回CredentialRecord实例列表。

[我们的实现](https://github.com/psevestre/tutorials/blob/master/spring-security-modules/spring-security-passkey/src/main/java/com/baeldung/tutorials/passkey/repository/DbUserCredentialRepository.java)考虑到了这一点，并使用PASSKEY_CREDENTIALS表中的外键来确保引用完整性。

## 7. 测试

虽然可以使用模拟请求来测试基于密钥的应用程序，但这些测试的价值有些有限。**大多数故障场景都与客户端问题有关，因此需要使用由自动化工具驱动的真实浏览器进行集成测试**。

这里，我们将使用Selenium实现“快乐路径”场景，以说明该技术。具体来说，我们将使用VirtualAuthenticator功能来配置WebDriver，使我们能够使用此机制模拟注册和登录页面之间的交互。

例如，我们可以这样使用VirtualAuthenticator创建新的驱动程序：

```java
@BeforeEach
void setupTest() {
    VirtualAuthenticatorOptions options = new VirtualAuthenticatorOptions()
            .setIsUserVerified(true)
            .setIsUserConsenting(true)
            .setProtocol(VirtualAuthenticatorOptions.Protocol.CTAP2)
            .setHasUserVerification(true)
            .setHasResidentKey(true);

    driver = new ChromeDriver();
    authenticator = ((HasVirtualAuthenticator) driver).addVirtualAuthenticator(options);
}
```

一旦我们获得了authenticator实例，我们就可以使用它来模拟不同的场景，例如成功或失败的登录、注册等。我们的[实时测试](https://github.com/eugenp/tutorials/blob/master/spring-security-modules/spring-security-passkey/src/test/java/com/baeldung/tutorials/passkey/PassKeyApplicationLiveTest.java)经历了一个完整的周期，包括以下步骤：

- 使用用户名/密码凭证首次登录
- 密钥注册
- 登出
- 使用密钥登录

## 8. 总结

在本教程中，我们展示了如何在Spring Boot Web应用程序中使用Passkeys，包括Spring Security设置和添加实际应用程序所需的密钥持久性支持。

我们还提供了一个如何使用Selenium测试应用程序的基本示例。