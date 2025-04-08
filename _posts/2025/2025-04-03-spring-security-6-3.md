---
layout: post
title:  Spring Security 6.3 – 新功能
category: springsecurity
copyright: springsecurity
excerpt: Spring Data JPA
---

## 1. 简介

Spring Security 6.3版本在框架中引入了一系列安全增强功能。

在本教程中，我们将讨论一些最显著的功能，重点介绍它们的优点和用途。

## 2. 被动JDK序列化支持

Spring Security 6.3包含**被动JDK序列化支持**。然而，在进一步讨论这个问题之前，让我们先了解一下它的问题和相关背景。

### 2.1 Spring Security序列化设计

**在6.3版本之前，Spring Security对通过JDK序列化在不同版本中序列化和反序列化其类有严格的策略**，此限制是框架为确保安全性和稳定性而做出的刻意设计决定。这样做的理由是防止使用不同版本的Spring Security反序列化在一个版本中序列化的对象时出现不兼容和安全漏洞。

此设计的**一个关键方面是整个Spring Security项目使用全局serialVersionUID**。在Java中，序列化和反序列化过程使用唯一标识符serialVersionUID来验证加载的类是否与序列化对象完全对应。

**通过维护每个Spring Security发布版本独有的全局serialVersionUID，框架可确保一个版本的序列化对象不能使用另一个版本进行反序列化**。这种方法有效地创建了版本屏障，防止反序列化具有不匹配serialVersionUID值的对象。

例如，Spring Security中的SecurityContextImpl类表示安全上下文信息，此类的序列化版本包含特定于该版本的serialVersionUID。当尝试在不同版本的Spring Security中反序列化此对象时，serialVersionUID不匹配会阻止该过程成功。

### 2.2 序列化设计带来的挑战

在优先考虑增强安全性的同时，这种设计策略也带来了一些挑战。开发人员通常将Spring Security与其他Spring库(如[Spring Session](https://www.baeldung.com/spring-security-session))集成，以管理用户登录会话。这些会话包含关键的用户身份验证和安全上下文信息，通常通过Spring Security类实现。此外，为了优化用户体验并增强应用程序的可扩展性，开发人员通常会将这些会话数据存储在各种持久存储解决方案中，包括数据库。

以下是由于序列化设计而产生的一些挑战。**如果Spring Security版本发生变化，通过Canary发布流程升级应用程序可能会导致问题。在这种情况下，持久会话信息无法反序列化，可能需要用户重新登录**。

**另一个问题出现在使用Spring Security的远程方法调用(RMI)的应用程序架构中。例如，如果客户端应用程序在远程方法调用中使用Spring Security类，则必须在客户端序列化它们，并在另一端反序列化它们。如果两个应用程序不共享相同的Spring Security版本，则此调用会失败，从而导致InvalidClassException异常**。

### 2.3 解决方法

解决此问题的典型方法如下，我们可以使用JDK序列化以外的其他序列化库，例如Jackson序列化。这样，我们就不用序列化Spring Security类了，而是获取所需详细信息的JSON表示，然后使用Jackson对其进行序列化。

另一个选择是扩展所需的Spring Security类，例如Authentication，并通过readObject和writeObject方法明确实现自定义序列化支持。

### 2.4 Spring Security 6.3中的序列化变化

在6.3版本中，类序列化会与前一个次要版本进行兼容性检查，这确保升级到较新版本后可以无缝地反序列化Spring Security类。

## 3. 授权

Spring Security 6.3在Spring Security授权中引入了一些值得注意的变化，让我们在本节中探讨这些变化。

### 3.1 注解参数

Spring Security的[方法安全](https://www.baeldung.com/spring-security-method-security)支持元注解，我们可以根据应用程序的用例采用注解并提高其可读性。例如，我们可以将@PreAuthorize("hasRole('USER')")简化为以下内容：

```java
@Target({ ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("hasRole('USER')")
public @interface IsUser {
    String[] value();
}
```

接下来我们就可以在业务代码中使用这个@IsUser注解了：

```java
@Service
public class MessageService {
    @IsUser
    public Message readMessage() {
        return "Message";
    }
}
```

假设我们有另一个角色ADMIN，我们可以为该角色创建一个名为@IsAdmin的注解。但是，这将是多余的，将此元注解用作模板并将角色作为注解参数包含会更合适。Spring Security 6.3引入了定义此类元注解的功能，让我们用一个具体的例子来演示这一点：

要模板化元注解，首先我们需要定义一个Bean PrePostTemplateDefaults：

```java
@Bean
PrePostTemplateDefaults prePostTemplateDefaults() {
    return new PrePostTemplateDefaults();
}
```

模板解析需要这个Bean定义。

接下来，我们将为@PreAuthorize注解定义一个元注解@CustomHasAnyRole，它可以接受USER和ADMIN角色：

```java
@Target({ ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("hasAnyRole({value})")
public @interface CustomHasAnyRole {
    String[] value();
}
```

我们可以通过提供以下角色来使用这个元注解：

```java
@Service
public class MessageService {
    private final List<Message> messages;

    public MessageService() {
        messages = new ArrayList<>();
        messages.add(new Message(1, "Message 1"));
    }

    @CustomHasAnyRole({"'USER'", "'ADMIN'"})
    public Message readMessage(Integer id) {
        return messages.get(0);
    }

    @CustomHasAnyRole("'ADMIN'")
    public String writeMessage(Message message) {
        return "Message Written";
    }

    @CustomHasAnyRole({"'ADMIN'"})
    public String deleteMessage(Integer id) {
        return "Message Deleted";
    }
}
```

在上面的例子中，我们提供了角色值-USER和ADMIN作为注解参数。

### 3.2 确保返回值安全

**Spring Security 6.3中另一个强大的新功能是使用@AuthorizeReturnObject注解保护域对象的能力**，此增强功能通过对方法返回的对象启用授权检查来实现更细粒度的安全性，确保只有授权用户才能访问特定的域对象。

让我们用一个例子来说明这一点，假设我们有以下带有iban和balance字段的Account类，要求只有具有read权限的用户才能检索帐户余额。

```java
public class Account {
    private String iban;
    private Double balance;

    // Constructor

    public String getIban() {
        return iban;
    }

    @PreAuthorize("hasAuthority('read')")
    public Double getBalance() {
        return balance;
    }
}
```

接下来，让我们定义AccountService类，它返回一个帐户实例：

```java
@Service
public class AccountService {
    @AuthorizeReturnObject
    public Optional<Account> getAccountByIban(String iban) {
        return Optional.of(new Account("XX1234567809", 2345.6));
    }
}
```

在上面的代码片段中，我们使用了@AuthorizeReturnObject注解，Spring Security确保只有具有read权限的用户才能访问Account实例。

### 3.3 错误处理

在上一节中，我们讨论了使用@AuthorizeReturnObject注解来保护域对象。一旦启用，未经授权的访问将导致AccessDeniedException。**Spring Security 6.3提供了MethodAuthorizationDeniedHandler接口来处理授权失败**。

让我们用一个例子来说明这一点，我们扩展第3.2节中的示例，并使用read权限保护IBAN。但是，我们打算提供一个屏蔽值，而不是对任何未经授权的访问返回AccessDeniedException。

让我们定义MethodAuthorizationDeniedHandler接口的实现：

```java
@Component
public class MaskMethodAuthorizationDeniedHandler implements MethodAuthorizationDeniedHandler  {
    @Override
    public Object handleDeniedInvocation(MethodInvocation methodInvocation, AuthorizationResult authorizationResult) {
        return "****";
    }
}
```

在上面的代码片段中，如果存在AccessDeniedException，我们将提供一个屏蔽值。此处理程序类可在getIban()方法中使用，如下所示：

```java
@PreAuthorize("hasAuthority('read')")
@HandleAuthorizationDenied(handlerClass=MaskMethodAuthorizationDeniedHandler.class)
public String getIban() {
    return iban;
}
```

## 4. 密码检查破解

**Spring Security 6.3提供了一个用于[检查泄露密码](https://www.baeldung.com/spring-security-detect-compromised-passwords)的实现**，此实现根据泄露密码数据库(pwnedpasswords.com)检查提供的密码。因此，应用程序可以在注册时验证用户提供的密码，以下代码片段演示了用法。

首先定义一个HaveIBeenPwnedRestApiPasswordChecker类的Bean定义：

```java
@Bean
public HaveIBeenPwnedRestApiPasswordChecker passwordChecker() {
    return new HaveIBeenPwnedRestApiPasswordChecker();
}
```

接下来，使用此实现来检查用户提供的密码：

```java
@RestController
@RequestMapping("/register")
public class RegistrationController {
    private final HaveIBeenPwnedRestApiPasswordChecker haveIBeenPwnedRestApiPasswordChecker;

    @Autowired
    public RegistrationController(HaveIBeenPwnedRestApiPasswordChecker haveIBeenPwnedRestApiPasswordChecker) {
        this.haveIBeenPwnedRestApiPasswordChecker = haveIBeenPwnedRestApiPasswordChecker;
    }

    @PostMapping
    public String register(@RequestParam String username, @RequestParam String password) {
        CompromisedPasswordDecision compromisedPasswordDecision = haveIBeenPwnedRestApiPasswordChecker.checkPassword(password);
        if (compromisedPasswordDecision.isCompromised()) {
            throw new IllegalArgumentException("Compromised Password.");
        }

        // ...
        return "User registered successfully";
    }
}
```

## 5. OAuth 2.0令牌交换授权

**Spring Security 6.3还引入了对[OAuth 2.0](https://www.baeldung.com/spring-security-oauth)令牌交换([RFC 8693](https://www.rfc-editor.org/rfc/rfc8693.html))授权的支持，允许客户端在保留用户身份的同时交换令牌**。此功能支持模拟等场景，其中资源服务器可以充当客户端来获取新令牌。让我们通过一个例子来详细说明这一点。

假设我们有一个名为loan-service的资源服务器，它为贷款账户提供各种API。此服务是安全的，客户端需要提供访问令牌，该令牌必须具有贷款服务的受众(aud声明)。

现在让我们假设loan-service需要调用另一个资源服务loan-product-service，该服务公开贷款产品的详细信息。loan-product-service也是安全的，并且需要具有loan-product-service受众的令牌。由于这两个服务的受众不同，因此loan服务的令牌不能用于loan-product-service。

**在这种情况下，资源服务器loan-service应该成为客户端，并将现有令牌交换为保留原始令牌身份的loan-product-service的新令牌**。

Spring Security 6.3为令牌交换授权提供了OAuth2AuthorizedClientProvider类的新实现，名为TokenExchangeOAuthorizedClientProvider。

## 6. 总结

在本文中，我们讨论了Spring Security 6.3 中引入的各种新功能。

显著的变化是授权框架的增强、被动JDK序列化支持和OAuth 2.0令牌交换支持。